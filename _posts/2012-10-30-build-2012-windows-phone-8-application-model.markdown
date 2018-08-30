---
layout: post
title: "[Build 2012] - Windows Phone 8 Application Model"
date: 2012-10-30 12:42:00 +01:00
categories: Windows Phone, Build 2012
author: "Julien Corioland"
identifier: f3a426c6-a16c-4916-89eb-c4681bb69560
redirect_from:
  - /archives/build-2012-windows-phone-8-application-model
  - /Archives/build-2012-windows-phone-8-application-model
---

Première session de la journée après le keynote qui fut assez riche en démos autour de Windows 8 / Windows Phone 8 et en surprises… <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_77BECD81.png">

![image](/images/build-2012-windows-phone-8-application-model/phone-app-model_16954160.jpg)

Pour cette première session Windows Phone 8 de la semaine, on s’intéresse aux nouveautés concernant le développement d’applications sur la plateforme. Elle est présentée par **Andrew Clinick**, <em>Group Program Manager Windows Phone</em>.

##

### Modèle d’exécution

Il faut garder en tête que tout ce qui fonctionnait sous Windows Phone 7.x continue à tourner sous Phone 8 : pas besoin de recompiler les applications.

Premier point abordé pendant la session : les performances des applications. Celles-ci doivent démarrer vite et être `fast & fluide` sinon, les utilisateurs risquent de ne pas vouloir les utiliser. Pour cela, Windows Phone 8 introduit une nouveauté : `Compile in the cloud`. Lorsque vous soumettez votre XAP dans le centre de développement, celui-ci est automatiquement recompilé en utilisant NGEN (Native Image Generator) et le XAP est mis à jour (au passage, c’est fait pour toutes les apps 7.x du store). L’utilisataion de NGEN est possible puisque désormais les applications s’exécutent dans la CoreCLR (comme sous Windows 8).

Le support des dual core sous Phone 8 est aussi pour beaucoup <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_77BECD81.png">

###

### Fast App Resume

Cette fonctionnalité est une évolution du fast app swtiching de Phone 7.5. Le constat a été fait que trop peu d’utilisateurs utilisent le task switcher (pression longue sur le bouton back). Du coup maintenant l’application est résumée depuis n’importe où : Live Tile, app list, toast, deep link…

### Multitasking

Il est désormais possible d’avoir une application qui utilise le service de localisation en continue. Au lieu d’être endormie, l’application peut continuer à s’exécuter en background pendant que l’utilisateur utilise une autre application.

### Maps

Le contrôle Bing Maps est toujours supporté mais déprécié. Il faut désormais utiliser les nouveaux contrôles Nokia Maps !!

### Intégration avec le téléphone

VOIP

Microsoft propose aux apps de s’intégrer avec la partie téléphone, pour les applications de type VOIP : récupérer des notification pour les appels entrants, activation d’appels vidéos, nouveaux background agent pour les appels voix, amélioration du support des pushs notifications.

Une application peut donc être en attente d’appels (c’est le cas pour Skype par exemple) et être réveillée dès que l’appel se produit (ouverture d’une popup similaire à celle d’un appel entrant GSM).

Une application peut également venir alimenter le hub contact en données (sans passer par les tâches contacts, comme sous 7.x). L’accès aux contacts créés par l’application se fait en lecture/écriture, mais en lecture seule pour les autres contacts (comme sous Mango)

Deep Linking & Partage

Il est possible d’associer une application à un schéma d’url donnée (exemple zune://, fb://) et faire en sorte que l’application se lance lorsque l’utilisateur clique sur une URL de cette forme. Il est désormais possible d’associer une extension de fichier à une application (comme le faisait déjà Adobe Reader ou les apps de la suite office). Du coup, votre application peut être associée à son propre format de fichier.

### Stockage de données

SQL Server et Linq 2 SQL sont toujours supportés sous Windows Phone 8, mais SQLite est maintenant disponible pour la plateforme !!

Windows Phone 8 supporte les cartes SD et des APIs sont disponibles pour lire et écrire sur celles-ci.

### Live Apps

Les `Live Apps` sont les applications qui utilisent des vignettes dynamique (live tiles). Microsoft oriente vraiment son discours autour de la nouvelle home page de Windows Phone, qui supporte désormais 3 tailles de vignettes dynamiques : petite, moyenne et grande. Microsoft encourage tous les développeurs à mettre en place ces vignettes dans leurs apps et propose des modèles de vignettes dans le SDK :

- <em>Flip</em> : comme sous phone 7.x, se retourne aléatoirement
- <em>Iconic</em> : possibilité d’avoir une icône, un compteur mais mieux que sous Phone 7.x avec un style proche des apps système type boîte mail etc…
- <em>Cycle</em> : plusieurs contenus qui défilent de manière cyclique.

Egalement au menu, l’intégration dans l’écran de vérouillage avec la possibilité de changer l’image de fond et d’alimenter l’écran avec trois ligne d’information (comme Outlook pouvait le faire). Le fond d’écran de vérouillage peut être mis à jour depuis l’application en cours d’exécution ou depuis un background agent. Les informations affichées sur l’écran de vérouillage par l’application sont les mêmes que celles de la vignette dynamique (une seule et même API). L’utilisateur choisi quelle (unique) application peut changer le fond d’écran. L’application peut demander à l’utilisateur d’afficher cette fonctionnalité (via API).

### Windows Phone 8 en entreprise

Windows Phone 7 a souvent été taclé sur ce point : sa mauvaise intégration en entreprise. Microsoft le sait, mais avec Windows Phone 8, ce temps est révolu ! <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_77BECD81.png">

Au niveau gestion de flotte mobile, il sera possible d’utiliser Windows Intune (application de police de sécurité, mise à jour d’apps automatique, gestion de parc…)

Au niveau du déploiement d’applications, il sera possible d’utiliser le web, l’email ou encore une application elle-même (un hub d’entreprise, par exemple). Vous devrez fournir votre propre certificat pour signer vos applications. Celles-ci ne seront alors disponibles que pour les collaborateurs de votre entreprise qui auront à enregistrer le certificat depuis leur téléphone (via web, mail…).

### Conclusion

Cette session a proposé une vue d’ensemble de toutes les nouvelles fonctionnalités qui tournent autour de l’exécution des applications sous Windows Phone, mais également de l’intégration de celles-ci avec le téléphone / OS. Nous aurons l’occasion d’aller dans des considérations plus techniques au cours des autres sessions !

Stay tuned <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_42057428.png">

Julien

