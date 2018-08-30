---
layout: post
title: "[Build 2012] - Windows Phone 8 Native C/C++ Game Development"
date: 2012-10-30 15:54:00 +01:00
categories: Windows Phone, Build 2012
author: "Julien Corioland"
identifier: 9f16824a-92fc-4f47-9b18-4c71a46278a9
redirect_from:
  - /archives/build-2012-windows-phone-8-native-cpp-game-development
  - /Archives/build-2012-windows-phone-8-native-cpp-game-development
---

Ce poste est co-écrit avec [Simon Ferquel](http://www.simonferquel.net/blog).

Le jeu représente un marché non négligeable pour les plateformes mobiles. Une des grandes nouveautés de la plateforme Windows Phone 8 est la possibilité de développer ces jeux en code natif C/C++, en plus de la compatibilité des jeux XNA développés pour Windows 7.x.

![image](/images/build-2012-windows-phone-8-native-cpp-game-development/native-game-development_4C2D68C9.jpg)

La session est présentée par **Sam George**, <em>Principal Group Program Manager Windows Phone</em>.

Avant les démonstrations du moteur Havok par Ross O’Dwyer (Head of developer support, Havok), Sam nous donne les principaux buts qui ont été fixés pour le développement de jeux sur Windows Phone 8 :

Premier point, la réduction des coûts de développement : il faut pouvoir réutiliser du code C++ existant pour d’autres plateformes, mais aussi utiliser des librairies Open Source C++, ou autres librairies comme Cocos2D / SharpDX. Second point, Windows Phone 8 doit supporter des moteurs / frameworks existants comme Havok, Unity, FMOD, ScaleForm, Wwise, Marmalade… Enfin dernier point, dans la stratégie d’unification des devs avec Windows 8, il doit être simple de faire des jeux multi-plateformes utilisant les mêmes APIs Direct3D / XAudio / MediaFoundation. Rappels également des nouveautés concernant l’in-app purchase sous Windows Phone 8, avec notamment la possibilité d’avoir deux types d’items (comme sous Windows 8) : consommables et durables.

Ensuite, nous avons assisté à un retour d’expérience d’Havok : <em>**que du bonheur**</em>. Au niveau des performances ça envoie du paté. Le compilateur est notamment capable d’exploiter des instructions SIMD et le runtime C++ permet de faire tout ce que les gros développeurs de jeux ont l’habitude de faire ! <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_7FED371C.png"> Les démos étaient impressionantes, aussi bien en terme de rendu graphique que de fluidité des animations.

### App Model

Pour le développement, on a deux variantes : soit C#/XAML qui exploite des composants C++, soit uniquement C++ !

C# + XAML et C++

Un peu comme avec Windows 8, on a deux possibilités d’intérop entre XAML et DirectX. La première c’est le nouveau contrôle `DrawingSurface` qui permet d’effectuer un rendu Direct3D et de le copier dans une surface XAML. L’avantage c’est que le rendu Direct3D rentre dans l’arbre de composition XAML et donc que l’on peut lui appliquer des transformations, de l’opacité (etc…) comme n’importe quel autre contrôle XAML. On a aussi accès à un modèle d’événements pour les inputs (chose qu’on a pas sous Windows 8).

La deuxième possibilité d’intérop est d’utiliser un composant `DrawingSurfaceBackgroundGrid` qui permet d’utiliser le rendu DirectX directement comme background de la page et d’éviter la recopie du contenu DirectX dans l’arbre de composition XAML (plus performant).

Petit truc sympa, on peut également créer des composants WinRT C++ appelables en C#.

Uniquement C++

Comme sous Windows 8, on implémente l’interface <em>IFrameworkView</em> qui nous permet d’obtenir une CoreWindow sur laquelle sont disponibles les différents événements d’input, du cycle de vie de l’app (suspended, resumed, launched…) mais aussi qui va nous permettre de créer la swap chain Direct 3D. C’est le mode le plus performant mais l’inconvénient est qu’on a accès à aucun composant .NET (donc pas de Live Tiles, pas de background agent etc…). Au passage, on a accès à une machine à état qui permet de détecter les gestures classiques (swipe, pinch, zoom...).

### Les APIs supportées

Pas de Direct2D/DirectWrite/WIC (pour le moment, mais clairement c’est dans les objectifs pour la prochaine version). Par contre côté Direct3D, support des shaders models 4.0_9.3 (d’ailleurs, la aussi la démo a envoyé du gros paté atomique). Pour l’audio, XAudio 2 est supporté et pour tout ce qui est musique et vidéo, on retrouve Media Foundation.

Côté réseau, on retrouve comme sous Windows 8 des APIs de peer en NFC/Wifi Direct (comme ce qu’on a utilisé pour Fingarock8 <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_7FED371C.png">) et Bluetooth ainsi que les classiques TCP (ssl aussi), UDP, HTTP. Les APIs ont l’air d’être les mêmes que sous Windows 8.

Au niveau du système de fichiers on retrouve la même chose que sous Windows 8 (CreateFile2 et tout ce qui va avec!) ainsi que le FS WinRT.

Enfin support de quelques Launcher & Chooser qui ont portés en WinRT pour l’occasion, car jugés nécessaires pour les jeux (market place, email, etc…).

### Conclusion

Nous avons pu voir dans cette session que Microsoft propose désormais un SDK très riche et complet pour le développement de jeux vidéos sur Windows Phone 8 ! Franchement, ça poutre grave !

Prochaine session, le partage de code entre Windows Phone 8 et Windows 8.

Stay tuned <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_0ED46637.png">

Simon & Julien

