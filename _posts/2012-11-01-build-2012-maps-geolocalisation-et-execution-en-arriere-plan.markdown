---
layout: post
title: "[Build 2012] Maps, géolocalisation et exécution en arrière plan."
date: 2012-11-01 16:28:53 +01:00
categories: Windows Phone, Build 2012
author: "Julien Corioland"
identifier: f5267c0f-83ca-4cb0-9b6d-c056ec9be94c
redirect_from:
  - /archives/build-2012-maps-geolocalisation-et-execution-en-arriere-plan
  - /Archives/build-2012-maps-geolocalisation-et-execution-en-arriere-plan
---

Première session Windows Phone 8 de cette 3ème journée de //build/ 2012 (pas de keynote aujourd’hui!), on s’intéresse aux nouveautés concernant les maps, la géolocalisation et l'exécution en arrière plan !

![image](/images/build-2012-maps-geolocalisation-et-execution-en-arriere-plan/WP_20121101_003_7B653BCF.jpg)

### Location Services APIs

Les APIs de géolocalisation ont été complètement repensées sous Windows Phone 8. On conserve l’ancienne API .NET avec le GeoCoordinateWatcher pour la compatibilité ascendante, mais une nouvelle API WinRT est disponible et beaucoup plus complète !

Côté WinRT, on retrouve une nouvelle classe, Geolocator pour localiser un utilisateur. Il y a plus de contrôle sur la précision souhaitée (en mètres) et il est notamment possible de récupérer la position de l’utilisateur sans la tracker en continue (Single Location Request), ce qui permet d’économiser la batterie de l’utilisateur. Comme beaucoup d’API WinRT, celle-ci est totalement asynchrone (async/await). On retrouve aussi la possibilité de contrôler la mise en cache et les timeout.

En terme de bonnes pratiques, il faut utiliser au maximum le Single Location Request, quand c’est possible, car celà permet vraiment de réduire la consommation de la batterie !

En bonus, un bout de code sur l’utilisation de l’API Geolocator :

![image](/images/build-2012-maps-geolocalisation-et-execution-en-arriere-plan/WP_20121101_005_690C4F40.jpg)

###

### Map Control et service de géolocalisation

Autre grosse nouveauté de la plateforme Windows Phone 8 côté cartes, l’utilisation des maps Nokia et des services de géolocalisation Nokia. On utilise un nouveau contrôle Maps qui permet notamment à l’utilisateur d’avoir ses cartes en mode offline pour ne plus consommer sa connexion de données et augmenter la fluidité ! Au passage, un nouveau launcher permet de proposer à l’utilisateur de télécharger une carte : MapsDownloader.

Le contrôle maps est capable d’afficher des cartes en 2D/3D, en vectoriel, ainsi que des itinéraires.

Avec cette carte, de nouveaux services arrivent pour effectuer directement des opérations de geocoding, géocoding inversé et de calcul d’itinéraire (il ne s’agit plus des services Bing, mais Nokia !)

Le BingMaps contrôle existe toujours pour la compatibilité des applications 7.x, mais il est déprécié !

L’utilisation du contrôle Maps et des services Nokia est gratuite, il faut juste associer son compte de développeur pour pouvoir récupérer un token.

Le maps toolkit extensions ([http://phone.codeplex.com](http://phone.codeplex.com)) permet d’avoir des fonctionnalités supplémentaires pour le contrôle Maps (pushpin etc..)

### Géolocalisation en arrière plan

Sous Windows Phone 8, une application peut s’exécuter en arrière plan ET récupérer la position de l’utilisateur en continue (une seule application peut le faire). C’est uniquement possible pour les applications XAML (XAML, XAML+Direct3D, mais pas pur Direct3D).

En mode background, l’application n’a accès qu’à des ressources limitées et qu’à un sous ensemble des APIs :

![image](/images/build-2012-maps-geolocalisation-et-execution-en-arriere-plan/WP_20121101_008_77D332C0.jpg)

Comme pour les background agent sous Windows Phone 7.5, l’utilisateur peux désactiver l’application dans les paramètres du téléphone.

L’application peut être notifiée (event) et notifier l’utilisateur lorsque la géolocalisation est désactivée au niveau du téléphone.

Au niveau du cycle de vie de l’application, on a un nouvel événement : RunningBackground (en plus de Launching, Deactivated, Activated et Close) qui permet de savoir que l’application s’exécute en arrière plan ! Ce nouvel event remplace le deactivated pour les applications de géolocalisation en arrière plan :

![image](/images/build-2012-maps-geolocalisation-et-execution-en-arriere-plan/WP_20121101_009_393DBF4D.jpg)

### Fast Resume

Le fast resume est la capacité des applications Windows Phone 8 à reprendre là où elles étaient, même si l’utilisateur les relance depuis son écran d’accueil ou la liste des applications (contrairement à Windows Phone 7 ou l’application était totalement relancée!). Pour permettre à une application de faire du fast resume, il faut éditer le fichier manifest, au niveau de la définition de la DefaultTask (par défaut, le comportement est le même que sous Phone 7, `Replace`) :

![image](/images/build-2012-maps-geolocalisation-et-execution-en-arriere-plan/fast-app-resume_7D6E00CC.png)

Lorsque l’application est `résumée`, le mode de navigation dans la méthode OnNavigatedTo est à `**Reset**`, ce qui permet par exemple de vider la stack de navigation tout en conservant une ancienne instance de l’application (optimisation de la mémoire du téléphone et de la rapidité du chargement de l’application). Toutes les explications ici : <a title="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj735579(v=vs.105).aspx" href="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj735579(v=vs.105).aspx">http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj735579(v=vs.105).aspx</a>

### Conclusion

Beaucoup de nouveautés autour de la géolocalisation, avec le nouveau contrôle Maps de Noka, les APIs de Nokia pour le géocoding et surtout la possibilité d’avoir une application qui track la position de l’utilisateur en continue et en background !

Stay tuned <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_07064936.png">

Julien

