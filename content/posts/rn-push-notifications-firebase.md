---
title: 'Notificationes Push en una app React Native con Firebase'
date: 2020-12-30T08:40:44+01:00
draft: true
tags:
  ['Carlos Cañizares', 'React Native', 'Push Notifications', 'Firebase', 'APNS']
featureImage: '../../images/temp.png'
---

Antes de entrar en el paso a paso de como configurar esto un poco de contexto. Si quieres tener una experiencia "nativa" de notificaciones tienes que pasar por APNS para iOS y por Google Cloud Messaging (Firebase) para Android.

Antes de optar por las librerías React Native Firebase (pesan mucho) estudié hacerlo con esta librería, pero tiene contrapartidas... La integración con APNS no es sencilla, en nuestro caso nuestro servidor está en C#, finalmente conseguimos comunicar nuestro servidor con APNS pero hay más pegas... Al arrancar la app tienes que registrar el device token en un servidor y esto tendríamos que programarlo en Swift o ObjetiveC. Y la pega más importante, que esta librería no proporciona nada para Android. Existen otros servicios de terceros como Microsoft Push Center o X pero es una tontería usarlos porque vas a necesitar configurar una aplicación de firebase igualmente para notificaciones en Android... tanto por tanto ya usamos firebase ¿para que querríamos otra capa más de servicio?

Firebase te permite centralizar iOS y Android en la configuración. Para ello tienes que configurar APNS, descargarte unas claves y subirlas a la configuración de tu app en firebase. A partir de ahí usando la librerías de RN Firebase ya tienes push para ambas plataformas.

### Crear una aplicación en la consola de firebase

#### Configurar iOS

### Configurar RN Firebase

#### Android

#### iOS

### Enviar mensaje de prueba

### Enviar mensaje vía Api de Firebase
