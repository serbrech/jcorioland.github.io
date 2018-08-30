---
layout: post
title: "[Build 2012] - Windows Phone 8 In App Purchase"
date: 2012-10-31 1:38:48 +01:00
categories: Windows Phone, Build 2012
author: "Julien Corioland"
identifier: b4499673-3f16-450f-a6ef-581f60db58bc
redirect_from:
  - /archives/build-2012-windows-phone-8-in-app-purchase
  - /Archives/build-2012-windows-phone-8-in-app-purchase
---

Dernière session de la journée, on s’intéresse à l’in-app purchase sous Windows Phone 8 ! <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_7DCB8BB0.png">

![image](/images/build-2012-windows-phone-8-in-app-purchase/in-app-purchase_554CA99C.jpg)

L’in-app purchase est possible depuis la première version de Windows Phone, mais avec Windows Phone 8, Microsoft apporte un support complet sur la mise en place de celui ci !

L’in-app purchase est une des sources de revenus les plus importantes sur les plateformes mobiles, c’est pourquoi Microsoft fait le choix de fournir des outils pour le mettre en place simplement et proposer aux utilisateurs un service qui soit unifié et pratique, avec notamment la possibilité de payer directement via la facture opérateur! (au passage, l’in app purchase est disponible dans les 191 pays où il est possible de publier des apps, donc le marché est vraiment important !)

Concrètement, il est possible de vendre un peu ce que l’on veut dans son application, mais on distingue deux types d’éléments:

- Consommables : il s’agit d’éléments qui seront utilisables une seule fois par l’utilisateur, par exemple un mode d’invincibilité dans un jeu, la possibilité de sauter un niveau…
- Durables : il s’agit d’éléments que l’utilisateur possèdent et pourra utiliser tant qu’il possède l’application (extension de jeu, ebook, service digital…)

En terme d’expérience utilisateur, le scénario est le suivant :

<ol> - L’utilisateur visualise la liste des éléments qu’il peut acheter au sein de l’application
- L’utilisateur choisi d’acheter un élément, il est redirigé sur le Wallet Windows Phone, rentre son code pin, choisi son mode de paiement (opérateur, carte bleu, paypal…) et valide
- L’utilisateur retourne dans l’application, l’item qu’il a acheté est disponible.
</ol> Du point de vue du développeur de l’application, voilà comment les choses se passent :

Design

Tout d’abord, il faut réfléchir aux items qui vont être mis en vente. Un produit est représenté par :

- Un ID unique sur le store (son `code barre`)
- Un type : consommable ou durable
- Un mot clé pour le catégoriser
- Des métadonnées : titre, description, icône…

Les éléments sont ensuite saisis via le Windows Phone developer center (au niveau du dashboard de l’application)

Développement

Côté développement, une API permet de travailler avec les produits disponibles en in app purchase. Tous les outils nécessaires sont dans l’espace de noms Windows.ApplicationModel.Store. Concrètement, il est possible de :

- Récupérer la liste des produits (par id, par mot clé…) afin d’afficher la liste des éléments disponibles dans l’application (l’affichage reste de la responsabilité du développeur, ce qui permet de vraiment intégrer l’expérience d’achat à l’application)
- De lancer l’action d’achat d’un produit pour rediriger l’utilisateur vers le wallet / le store
- De récupérer la liste des produits que possède l’utilisateur

Tests

Pour tester l’in app purchase, Microsoft propose deux solutions : via l’émulateur, à l’aide d’un mock ou alors via le programme de publication d’application en beta (un store de produit beta est disponible)

Publication

Une fois le design, le développement et les tests terminés, l’application et le catalogue de produits qui l’accompagnent peuvent être publiés sur le store.

Les speakers ont une démo de mise en place de l’in app purchase dans une application existante, c’est vraiment ultra simple. Les APIs sont là et elles sont facile à prendre en main. Comme on dit, y’a plus qu’à !

Stay tuned <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_10705F9B.png">

Julien

