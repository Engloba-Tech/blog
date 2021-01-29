---
title: 'Configurando CD para mi aplicación React Native a Google Play y Apple Store con App Center'
date: 2020-12-30T08:40:44+01:00
draft: true
tags:
  [
    'Carlos Cañizares',
    'App Center',
    'React Native',
    'Google Play',
    'Android',
    'CD',
  ]
featureImage: '../../images/AppCenter-AppStore-GooglePlay.png'
---

En los últimos 4 o 5 años, los equipos de desarrollo hemos mejorado mucho en cuanto a automatización de deploys y demás. Ha habido un auge devops que ha hecho que la mayoría de developers de bien entiendan y definan una pipeline. Ahora entendemos la importancia de la automatización y como esto va vinculado a entregas rápidas y dinámicas que tanto remarca Agile. A día de hoy no podría hacer un despliegue manual de mis aplicaciones web, de echo, al empezar un proyecto lo primero es montar el circuito de CD con un hello world y a partir de ahí empezamos a montar features y lo que quieras.

Las herramientas de CD/CI también han evolucionado mucho y la adopción del cloud por la mayoría de equipos también tiene a ver en esta tendéncia... En el caso de Engloba Tech a día de hoy usamos Azure Dev Ops para repos y la configuración de pipelines de nuestras aplicaciones web. Para las apps nativas, la parte de build/deploy/distribute la hacemos desde App Center que es el antiguo Hockey App que en su día compró Microsoft y está totalmente especializado en build/deploy/distribute aplicaciones nativas iOS/Android y te da "cosas" OOB que si te las tuvieses que montar en una pipeline de Azure Dev Ops te costaría mucho más. Tienes una capa gratuita que te da 4 horas de agente al mes... Nosotros al final hemos necesitado ampliar (es rollo 40€ mes ahora) pero porque tenemos bastante movimiento.

### CD en aplicaciones nativas

Cuando tu producto incorpora una app nativa, sabemos que lo suyo es hacerla pasar por las correspondientes stores para su distribución. En un servidor web, nadie te va a decir que no puedes subir una web porque el contenido no es super bonito, o no está claro, o porque falta una captura en la descripción de producto o un video explainer (sí sí, apple se gusta un montón pidiendo este tipo de mierdas para aprobarte la publicación). El escenario ideal en nuestro caso sería el mismo que tenemos para las aplicaciones web... un entorno con CD que suele ser nuestro entorno de beta/pruebas que tiene todo lo último... y luego tenemos el paso a producción que depende de que pulsemos el botón de la release correspondiente. Es nuestro escenario, no quiero decir que sea el mejor ni el universal ni nada por el estilo... es una política de despliegues que en nuestro caso funciona y otro equipo podría optar por montar una estrategia de ramas y entornos más elaborado en función de sus necesidades.

### Como planteamos este escenario en Engloba Tech desde App Center

En App Center lo montamos un poco diferente a como solemos hacerlo en web, tenemos una rama "x" que es la que se alinea con nuestro entorno pruebas. Esta rama tiene una build que se distribuye a colaboradores (testers, normalmente nosotros y algún usuario clave de negocio), esta rama por tanto si que está automatizada pero la distrución es mediante App Center y correos... no pasamos por la store lo cual no es una app válida para el usuario en el sentido que tiene que habilitarse movidas en android para poder usar la app (en ios peor, tienes que vincular el device ID al provisioning profile de la app, un follón añadir un tester)... la distribución anterior nos sirve para testear la evolución y una vez ok pasamos a la store.

Te preguntarás como gestionamos la transformación de settings entre un entorno u otro. Lo hacemos jugando con esta secciónd de la build y luego leyendo ese fichero .env desde React Native.

![Managing env variables in App Center build](../../images/AppCenter-Build-Env-Variables.PNG 'Managing env variables in App Center build')

Y en master tenemos configurada la build que va a las stores donde metemos los valores de producción en los settings de entorno. De este modo al hacer una PR y aprobarla de rama "x" a master hará que se lance el proceso de publicación a las correspondientes stores.

### Evitando pasos manuales

¿Es posible automatizar la subida a la store desde App Center?, La respuesta es sí... pero eso no quiero decir que tu aplicación esté disponible desde el momento que acaba la release a la store... tanto Google como Apple tendrán que validar la publicación y puede tardar unos días o incluso que te la deniegen por algún motivo. Esto es un poco jodienda la verdad, tienes que ir con cuidado que si se publican las apis vnext no se rompa la app que consume estas apis... Es posible que te toque cambiar ese V1 que la mayoría solemos meter en nuestras apis por defecto pero que a la hora de la verdad pocas veces te toca mantener versionado en serio en tus apis... Otra opción para mitigar este tema es hacer la publicación en las stores pero no activar la release hasta que está aprobada. Esto puede valerte seguro con Android porque no prueba la release pero has de ir con cuidado con Apple. Si tu app no es compatible con la versión pública de la api podría ser que detecten algun fallo y no te aprueben la publicación (apple prueba la app, te piden credenciales en caso que sean necesarias y te dan feedback).

Veamos como automatizar esto desde App Center, en realidad es muy sencillo si ya te mueves bien por App Center y las correspondientes stores. La primera subida no queda otra que hacerla a mano, ánimo jaja. Una vez ya tienes la primera subida detallo los pasos necesarios para automatizar la publicación a las stores.

### Google Play

Necesitas una developer account, una vez dentro has de crear una cuenta de servicio que pueda acceder a la api. Lo haremos siguiendo los pasos que ves en las capturas.

![Create service account in Google Play Api](../../images/GooglePlay-Api-Access.PNG 'Create service account in Google Play Api')

Una vez pulsas en Crear Cuenta de Servicio te dirige a la página de Google Cloud Platform donde puedes crear cuentas de servicio y vincular "keys" (p12 o json).

![Creating Api Key and Downloading Json](../../images/Create-Service-Account-Json-Key.PNG 'Creating Api Key and Downloading Json')

Descargamos la clave y nos vamos a App Center, vamos a distribute y configuramos el acceso a Google Play subiendo la clave que hemos descargado.

![Upload Google Credentials Json to App Center](../../images/Upload-Json-Key-AppCenter.PNG 'Upload Google Credentials Json to App Center')

Una vez tenemos la store vinculada ya podemos seleccionarla en nuestra build Android/Master.

![Select Distribute To Store in Distribution Section](../../images/Distribute-Store-AppCenter.PNG 'Select Distribute To Store in Distribution Section')

Con esto ya tendríamos automatizada la publicación para Google Play de modo que un commit en master de la app lanza el proceso.

### Apple Store

Para Apple pues vas a App Center a la app de iOS y en Distribute vamos a Store (igual que para Android).

Te pedirá tus credenciales del Apple ID con el que tienes cuenta en Apple developer y has registrado la app la primera vez... Introduces credenciales y como mucho si tienes doble factor de autentiación en tu cuenta (imagino que sí) te hará generar claves para uso de apps que ya verás que el mismo asistente en App Center te guía para hacerlo y es fácil.

Ahora tendrías que hacer exactamente lo mismo que hemos hecho para Android, ir a la build de master y en la sección Distribute seleccionar Store y Production.

También podrías distribuir una release existente de modo manual. Si te vas a la release una vez has configurado la store, te sale como una opción más de distribución.
![Manually distributing a existing release to Apple Store](../../images/Distribute-Apple-Store-AppCenter.PNG 'Manually distributing a existing release to Apple Store')

Con esto ya tendríamos automatizada la publicación en las tiendas!
