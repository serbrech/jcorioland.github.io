---
layout: post
title: "[Build 2012] Windows Phone 8 – tiles et notifications"
date: 2012-11-01 22:44:53 +01:00
categories: Windows Phone, Build 2012
author: "Julien Corioland"
identifier: 69c82e30-6b23-4a0d-a915-7cc9b11af0a0
redirect_from:
  - /archives/build-2012-windows-phone-8-tiles-et-notifications
  - /Archives/build-2012-windows-phone-8-tiles-et-notifications
---

Allez, on attaque la troisième session de la journée pour voir ce qu'il y a de nouveau du côté te des tiles et des notifications.

<img src="https://juliencorioland.blob.core.windows.net/medias/110112_2244_Build2012Wi1.jpg" alt=""/>

Au niveau des notifications, l'architecture reste similaire à celle de Windows Phone 7.5 avec l'envoie de requêtes http au service de push notifications Microsoft (MPNS). Quelques nouveautés quand même, puisqu'il n'y a plus de limite concernant le nombre d'applications qui peuvent utiliser les notifications sur le device (30 sous WP7).

Il y a également un nouveau type de notification push qui permet de réveiller une application VOIP.

Du côté des vignettes dynamique il y a aussi pas mal de nouveautés, puisqu'il est maintenant possible de créer des tuiles de trois tailles différentes : petite, moyenne, large. A ces trois tailles de vignettes, il est possible d'appliquer un modèle, parmi les suivants :

### Flip tile

Ce modèle est similaire à celui existant sous phone 7.x. On retrouve l'image avant et arrière, le count, le titre, le contenu…

<img src="https://juliencorioland.blob.core.windows.net/medias/110112_2244_Build2012Wi2.jpg" alt=""/>

### Cycle tile

Il s'agit d'un modele permettant de faire defiler plusieurs images, un peu à la manière de la vignette du hub photos.

<img src="https://juliencorioland.blob.core.windows.net/medias/110112_2244_Build2012Wi3.jpg" alt=""/>

### Iconic tile

Ce modèle permet de générer des vignettes qui possèdent un count et un logo, mais en animant le count, comme la tuile Outlook, par exemple.

<img src="https://juliencorioland.blob.core.windows.net/medias/110112_2244_Build2012Wi4.jpg" alt=""/>

Côté local tiles APIs, la ShellTileSchedule peut mettre à jour TOUTES les propriétés de la vignette. Il est désormais possible de passer un payload xml pour alimenter les tiles data, plutôt que d'affecter toutes les propriétés à la main !

Il est également possible de s'intégrer avec l'écran de verrouillage du téléphone afin d'y ajouter des informations (logo, nombre et texte) identiques aux informations de la vignette principale, mais celle-ci ne doit pas obligatoirement être épinglée.

<img src="https://juliencorioland.blob.core.windows.net/medias/110112_2244_Build2012Wi5.jpg" alt=""/>

Les APIs sont toutes plus simple à utiliser les unes que les autres et beaucoup d'incohérences que l'on pouvait avoir ont disparues. Bref, ça poutre bien comme il faut !

Stay tuned <span style="font-family:Segoe UI Symbol">😉
</span>

<span style="font-family:Segoe UI Symbol">Julien</span>

