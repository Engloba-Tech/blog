---
title: 'Depurar mi código en una aplicación hospedada en App Service de Azure'
date: 2020-12-30T08:40:44+01:00
draft: true
tags: ['Carlos Cañizares', 'Azure', 'App Service', 'C#']
---

Depurar tu código C# en Azure App Service es bastante sencillo y ... extraño 😃. Me explico es raro que necesites depurar tu código una vez está desplegado, tiene que ser algo rebuscado sino siempre entiendo que tirarás antes de logs. En fin, veamos como depurar en app service. Para depurar tu código necesitas Visual Studio (está la versión community que es free) y necesitarás publicar los símbolos de tu aplicación. Te recomiendo por eso que sólo publiques símbolos cuando sea necesario... Por ejemplo podrías hacer una release temporal para ver un problema en modo debug y una vez hayas acabado volver a desplegar en modo release.

### Como públicar los símbolos (pdb)

Los símbolos son esos archivos pdb que se generan cuando estamos corriendo nuestra aplicación en modo Debug. Por normal general si corres en modo Debug se publican símbolos y si lanzas modo Release no se publicarán... sin más. Si tienes definida integración continua con Azure Dev Ops, Jenkins o algo de este estilo seguramente en la pipeline tengas definida una variable buildConfiguration (Azure DevOps te la mete por defecto si no estoy equivocado y también te permite configurar si la variable es setteable en run time). Si no tienes CD a Azure y publicas con botón derecho pues selecciona Debug en el asistente ese que sale.

### Como comprobar que los símbolos están en App Service

Hay varios modos de comprobarlo, la más fácil y cutre es conectando el debugger al App Service. Si no hay símbolos se queja... eso quiere decir que no están 🙈.

Ahora en serio, tienes modos de examinar el contenido de tu App Service y comprobar si para cada assembly que necesitas depurar está su correspondiente .pdb. Desde el portal mismo, si entras al App Service y te fijas en la sección Development Tools.

![App Service Development Tools](../../images/AppService-Development-Tools.PNG 'App Service Development Tools')

Está por un lado Advanced Tools también conocido como [Kudu](https://github.com/projectkudu/kudu "Project's Kudu Github Repository") (tiene api 💓) y por otro lado App Service Editor también conocido como Monaco... Usa el que quieras para listar el contenido del directorio wwwRoot que es donde están los assemblies de tu aplicación. Si usas Kudu lo tendrás que hacer desde la opción "Debug Console" y ahí te puedes mover hasta el directorio.. De echo si te quedas un rato en el portal de Kudu verás que si no lo conocías todavía es oro... tienes registros, profilings. De echo para "resolver misterios" en App Service son más útil estas herramientas que no lo que estoy contando originalmente en el post 😆

### Conectar Visual Studio a AppService

Asegúrate que la versión del assembly que quieres depurar es la misma que tienes en local, por ejemplo si el entorno que vas a depurar tiene CD y está vinculado a master pues con situarte en master te valdría.

Para conectar Visual Studio a AppService

![Debug AppService From Visual Studio](../../images/Debug-AppService.PNG 'Debug AppService From Visual Studio')

Esto llevará su ratito... una vez visual studio se conecta al App Service te abre la web en cuestión en el browser... a partir de ahí ya puedes poner breakpoints y depurar lo que necesites. Por cierto si has leído este post te recomiendo leer un post al que llegué por twitter en el que se profundiza más en kudu y estas movidas.

Hasta aquí este post, es muy básico pero me ha servido para vencer finalmente la procastinación y retomar lo de escribir en blogs que ya hace un par de años que no lo hacía y se echa de menos!!
