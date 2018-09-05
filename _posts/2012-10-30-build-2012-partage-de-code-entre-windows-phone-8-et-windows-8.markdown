---
layout: post
title: "[Build 2012] Partage de code entre Windows Phone 8 et Windows 8"
date: 2012-10-30 17:37:00 +01:00
categories:
- Windows Phone
- Windows 8
- Build 2012
author: "Julien Corioland"
identifier: f9e3290c-07bb-40a1-8996-b2972c398dfe
redirect_from:
  - /archives/build-2012-partage-de-code-entre-windows-phone-8-et-windows-8
  - /Archives/build-2012-partage-de-code-entre-windows-phone-8-et-windows-8
---

Un post rapide pour vous parler de cette session sur le partage de code entre Windows Phone 8 et Windows 8. Je prendrai le temps de revenir dessus dans un article plus complet une fois que j’aurai un peu plus de recule et fait quelques exemples !

![image](/images/build-2012-partage-de-code-entre-windows-phone-8-et-windows-8/sharecode_503909D8.jpg)

Globalement, ce qu’il faut retenir, c’est que les deux plateformes sont en train de converger, avec l’apparition de WinPRT (Windows Phone Runtime) sous Windows Phone 8 qui est un sous ensemble des APIs WinRT (Windows Runtime) de Windows 8. Concrètement, on retrouve 3 sets d’API distincts sous Windows Phone 8 :

- Framework .NET
- Windows Phone Runtime
- Native API : Direct3D, XAudio2, Media Foundation

Voilà un schéma résumant ces APIs :

![image](/images/build-2012-partage-de-code-entre-windows-phone-8-et-windows-8/WP_000185_6924B450.jpg)

En terme de stratégie, pour favoriser le partage de code entre les deux plateformes, il faut bien évidemment séparer au maximum la logique de l’application de l’UI. Pour ça, MVVM reste le pattern à utiliser en XAML. Pour partager du code .NET entre Windows Phone et Windows, on utilisera les Portable Class Libraries. Il est également possible d’utiliser des fichiers liés (add as link) ou un composants WinRT compilé (pas possible de référencer directement le projet dans Windows Phone 8, il faut aller chercher le .winmd!)

Concernant le code qui est `platform specific`, pas de recette miracle ni grosse nouveauté, on utilise la portable class library pour créer nos interfaces et derrière on fait de l’injection de dépendances dans chaque application.

Voilà ce qu’il faut retenir de cette session à ce stade. Plus d’info sur MSDN, sur [cette page](http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj714089(v=vs.105).aspx).

Stay tuned <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_7BC9883A.png">

Julien

