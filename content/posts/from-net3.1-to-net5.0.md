---
title: 'Migrando una Api de dotnet 3.1.x a 5.0.x'
date: 2021-02-24T11:10:44+01:00
draft: true
tags: ['Carlos Cañizares', 'C#', 'DotNet']
featureImage: '../../images/Migrating-From-DotNet3.1-To-DotNet5.0.PNG'
---

Hoy me dispongo a migrar una api rest "sencilla" a la versión más reciente de dotnet que en estos momentos es 5.0.
En primer lugar necesitarás instalarte el [SDK](https://dotnet.microsoft.com/download/dotnet/5.0 'net5 sdk download page') en tu máquina local de desarrollo si no lo has hecho ya.

Esta api tiene lo típico, EF, MediatR, Auth, poco más.. Vamos a ello, en los proyectos que componen la solución modificaré el atributo TargetFramework y los paquetes como indican en la guía de migración. Pero la primera duda que me surge es que pasa con los proyectos que apuntan a NetStandard, en esta solución tenemos un par.

Como explican [aquí](https://docs.microsoft.com/es-es/dotnet/standard/net-standard#net-5-and-net-standard 'net 5 and the net standard'), podríamos mantener NetStandar 2.0/2.1 siempre y cuando necesitemos usar esa dll desde una aplicación Net Framework. Realmente si nos paramos a pensar nosotros no tenemos Net Framework por tanto tampoco tiene demasiado sentido apuntar a Net Standard en nuestro caso. En estos casos recomiendan apuntar también estos paquetes a Net 5.

Una vez actualizados los proyectos que componen la solución, actualizaremos también la imagen de aspnet en el docker file.

Y... bien, compila a la primera y desde swagger un GET me devuelve datos del seed. El siguiente paso es correr los test y... bien pasan a la primera.

![Running integration tests in Net 5.0](../../images/Running-integration-tests-in-Net-5.0.PNG 'Running integration tests in Net 5.0')

Por último, quedará ver si es necesario cambiar algo en las pipelines. Usamos un agente própio que es un ubuntu VM en Azure, intuyo que nos tocará conectarnos al agente e instalar el SDK 5.0 como hemos hecho en nuestra máquina local. Una vez hecho funcionan las pipes sin problema, en principio el agente intentará construir las aplicaciones usando el último SDK instalado... es decir intentará compilar con el SDK de 5.0 todas nuestras aplicaciones .NET. En principio debería ser capaz de compilar también 3.1, en nuestro caso siguen funcionando todas las pipes sin problema pero podría darse el caso que necesites compilar una aplicación con un SDK concreto.. en ese caso deberíamos usar el fichero global.json, aunque ya digo que en nuestro caso conviven 5.0/3.1 sin problema y sin necesidad de añadir ficheros global.json.

#### Conclusiones

Esto mismo que estamos haciendo hace 10 años entre versiones de .net, es decir hacer un upgrade de framework de una parte del proyecto sin afectar al resto hubiese sido un reto en mayúsculas. Kudos para Microsoft en general por haber enfocado bien estos últimos años de evolución en sus servicios y lenguajes..

También entiendo que me ha sido todo tan fácil y he tenido 0 problemas porque me he esperado unos meses a pasarme a net 5 y no partimos de la primera minor ni de prereleases... Esperar, si te lo puedes permitir, siempre es recomendable porque te aseguras que está más maduro y por otro lado hay más margen de tiempo para que los fabricantes de librerías saquen el paquete compatible con la nueva versión.
