---
title: 'Configurando Serilog contra Application Insights en nuestra aplicación web Asp.Net'
date: 2020-12-30T08:40:44+01:00
draft: false
tags: ['Carlos Cañizares', 'C#', 'Azure', 'Serilog']
---

La monitorización es un aspecto importante de nuestras aplicaciones :bowtie:. Si tienes la insfraestructura en Azure lo más típico sería usar Application Insights / Azure Monitor como "recolector/servicio" de métricas e incluso visor (para visualizar métricas al final del post mencionaré algunas alternativas).

Si usas app services de Azure, tener un site monitorizado en Azure Monitor es cuestión de minutos... lo puedes hacer desde el portal, pulsando añadir application insights y configurando el setting de la instrumentation key. Esto ya hará que puedas consumir ese resource App Insights que has creado para recolectar métricas del site y ver métricas interesantes como número de peticiones, ver agregados de peticiones por códigos de respuesta, tiempo medio respuesta, etc... Ok, pero que pasa si quiero analizar porque se ha producido un 500 o hacer seguimiento de un hilo de peticiones más complejo. Si no configuramos "algo más" en la api vamos un poco a ciegas cuando queremos ver más en detalle.

.Net como framework incorpora su api para logging https://docs.microsoft.com/es-es/aspnet/core/fundamentals/logging/?view=aspnetcore-5.0 aunque solemos configurar algún paquete que nos ayude con la gestión de todo esto en nuestras aplicaciones. Tienes varias opciones como Log4Net, Nlog (...). En nuestro caso usaremos Serilog.

### Como configurar Serilog en Asp.Net 3.1

En primer lugar asegúrate de tener App Insights Telemetry registrado en la api, ya que serilog "volcará" lo que vaya colectando a un registro que podría ser una base de datos, un fichero o en este caso un servicio en el cloud como es Azure Monitor / Application Insights.

Añade el paquete Microsoft.ApplicationInsights.AspNetCore a tu Host.

```xml
  <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.16.0" />
```

Configura el middleware en tu configuración de arranque:

```java
  services.AddApplicationInsightsTelemetry();
```

Para usar Serilog contra AppInsights necesitaremos estos 3 paquetes:

```xml
  <PackageReference Include="Serilog.AspNetCore" Version="3.4.0" />
  <PackageReference Include="Serilog.Settings.Configuration" Version="3.1.0" />
  <PackageReference Include="Serilog.Sinks.ApplicationInsights" Version="3.1.0" />
```

Ahora el siguiente paso sería configurar serilog en el arranque de nuestra aplicación asp.net normalmente en la clase Program.cs. Nos interesa configurar aquí ya Serilog porque normalmente en nuestras apis solemos tener un seed básico de datos y de este modo registraríamos si se podruciese algún fallo en este proceso.

Nuestro código pinta así, lo importante es como nos traemos el logger de la configuración de la api y como luego le decimos que "escriba" los registros en App Insights.

```java
  public static async Task Main(string[] args)
  {
      var host = BuildWebHost(args);
      using (var scope = host.Services.CreateScope())
      {
          var services = scope.ServiceProvider;
          var logger = services.GetRequiredService<ILogger<Program>>();
          try
          {
              var environment = services.GetRequiredService<IWebHostEnvironment>();
              var builder = new ConfigurationBuilder()
                  .SetBasePath(environment.ContentRootPath)
                  .AddJsonFile("appsettings.json", false, true);

              var config = builder.Build();
              await DbInitializer.InitializeDatabaseAsync(config, services, environment);
          }
          catch (Exception ex)
          {
              logger.LogError(ex.GetInnerException());
          }
      }

      host.Run();
  }
```

```java
  public static IWebHost BuildWebHost(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
          ...
          .UseStartup<Startup>()
          .UseSerilog((hostingContext, loggerConfiguration) => loggerConfiguration
                .ReadFrom.Configuration(hostingContext.Configuration)
                .WriteTo.ApplicationInsights(new TelemetryConfiguration {
                    InstrumentationKey = hostingContext.Configuration.GetValue<string>("ApplicationInsights:InstrumentationKey")
                  }, TelemetryConverter.Traces))
          .Build();
```

Por otro lado en nuestro caso solemos configurar un filtro de excepciones a nivel de aplicación de modo que si se genera una excepción no controlada siempre entrará en el filtro... allí también usamos serilog.

```java
  services.AddMvc(options =>
        {
            ...
            options.Filters.Add(typeof(ExceptionFilter));
            ...
        })
```

Y hasta aquí el post básico sobre como configurar serilog en asp.net contra Application Insights. Como disclaimer añadir que por los tiempos que corren quizás deberíamos ir pensando de movernos a .net 5.0... en fin no tardaremos.
