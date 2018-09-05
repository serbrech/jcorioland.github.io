---
layout: post
title: "[ASPNET] Exécutez du code au pré-démarrage d’une application"
date: 2013-04-10 6:48:47 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: 1080de24-8336-4a0c-9edd-720cab9d11c4
redirect_from:
  - /archives/aspnet-executez-du-code-au-predemarrage-dune-application
  - /Archives/aspnet-executez-du-code-au-predemarrage-dune-application
---

Depuis ASP.NET 4, il existe une fonctionnalité assez intéressante : la possibilité d’exécuter du code au chargement de l’app domain dans lequel s’exécute votre application, c’est à dire avant même que le code de votre application soit chargé.

On parle du `pré-démarrage` de l’application. Ce mécanisme est très simple à mettre en place et peut être réalisé dans n’importe quel assembly chargé dans l’app domain de votre application :

- Création d’une classe statique PreApplicationStart (par exemple)
- Création d’une méthode statique OnStart (par exemple)
- Ajout d’un attribut d’assembly pour indiquer l’emplacement de cette classe

Cela peut être très utile, notamment pour enregistrer des modules http, par exemple !

Vous obtenez donc un code de ce genre là :

```csharp
public static class PreApplicationStart
{
public static void OnStart()
{
//code de pré-démarrage
}
}
```
Ensuite, il suffit de pointer cette classe depuis le fichier Assembly.cs du projet dans lequel elle est définie :

```csharp
[assembly: PreApplicationStartMethod(typeof(PreApplicationStart), "OnStart")]
```
Il ne reste qu’à référencer la librairie dans le projet web, et la méthode OnStart sera automatiquement appelée lors du pré-démarrage.

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_28EC3169.png">

Julien

