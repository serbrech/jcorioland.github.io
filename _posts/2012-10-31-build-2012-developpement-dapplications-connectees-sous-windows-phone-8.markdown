---
layout: post
title: "[Build 2012] Développement d’applications connectées sous Windows Phone 8"
date: 2012-10-31 21:40:06 +01:00
categories: Windows Phone, Build 2012
author: "Julien Corioland"
identifier: 5ba3539d-9074-4ef8-aada-3ccc66629c99
redirect_from:
  - /archives/build-2012-developpement-dapplications-connectees-sous-windows-phone-8
  - /Archives/build-2012-developpement-dapplications-connectees-sous-windows-phone-8
---

Windows Phone 8 apporte un certain nombre d’APIs permettant de développer des applications `connectées` : Socket, NFC, Bluetooth…

![image](/images/build-2012-developpement-dapplications-connectees-sous-windows-phone-8/WP_20121031_024_5AC6B2A5.jpg)

Dans cette session, Tim Laverty, PM Windows Phone nous présente toutes ces nouveautés !

### Les objectifs de la session

- Avoir une bonne vue d’ensemble de toute la stack networking de Windows Phone 8
- Voir quel sont les éléments qui diffèrent entre Phone 7, Phone 8 et Win 8 (et donc ceux qui se rapprochent, pour pouvoir réutiliser un max de code)
- Découvrir les APIs et leur prise en main

### Les APIs de networking

- APIs .NET pour la manipulation des sockets  dans <em>System.Net.Sockets</em>
- APIs WinRT pour la manipulation des sockets dans <em>Windows.Networking.Sockets</em>
- APIs WinRT pour le networking de proximité (NFC / Bluetooth) dans <em>Windows.Networking.Proximity</em>
- APIs native pour la manipuation de socket (Winsock api family)

![image](/images/build-2012-developpement-dapplications-connectees-sous-windows-phone-8/WP_20121031_025_178B0EAB.jpg)

### Communication via Bluetooth

- Couvre des scénarii de communication App to App et App to Device
- Permet d’ouvrir un socket WinRT (StreamSocket)
- Fonctionne dans un rayon de 0 à 100m

### Communication NFC

- Couvre également des scénarii de communication App to App et App to Device, mais pour des volumes de données réduits (bande passant ~424 kbit/s)
- Fonctionne dans un rayon de 2 à 4 cm
- Il est possible d’associer le `contenu` NFC à une application du téléphone : je scan un tag NFC, ça ouvre mon application (on utilise une extension de protocole dans le manifeste de l’application)
- Il est également possible d’envoyer de la données dans un tag NFC (pour l’associer avec votre app, en lui précisant le protocole à utiliser)

### Sockets

Concernant les sockets, on peut donc utiliser plusieurs API : .NET, WinRT ou native. Dans tous les cas, il est possible de faire aussi bien du TCP que de l’UDP.

Concrètement, à partir du moment ou il faut mettre en place des stratégies de code sharing avec les applications Windows Store, il faut partir sur des sockets WinRT, les APIs .NET et native n’étant pas compatibles avec les Windows Store Apps. Les API .NET sont là pour du code portable sous Windows 8 (pas WinRT) et les API natives si vous possédez du code existant utilisant Winsock.

Stay tuned <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_34345DF3.png">

Julien

