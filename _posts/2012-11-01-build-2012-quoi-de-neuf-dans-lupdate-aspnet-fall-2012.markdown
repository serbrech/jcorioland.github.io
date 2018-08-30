---
layout: post
title: "[Build 2012] Quoi de neuf dans l’update ASPNET Fall 2012"
date: 2012-11-01 12:43:47 +01:00
categories: ASP.NET MVC, Build 2012
author: "Julien Corioland"
identifier: 25c61c61-b76c-4e7b-a96e-6f9cf69c7a5e
redirect_from:
  - /archives/build-2012-quoi-de-neuf-dans-lupdate-aspnet-fall-2012
  - /Archives/build-2012-quoi-de-neuf-dans-lupdate-aspnet-fall-2012
---

Le keynote de la deuxième journée était principalement consacrée à Windows Azure (vous pourrez le voir ou revoir [ici](http://channel9.msdn.com/Events/Build/2012/1-002), lorsqu’il sera dispo) mais quelques annonces assez sympa ont été faites autour d’asp.net / asp.net mvc et asp.net web api.

Toutes les nouveautés annoncées sont regroupées sous la forme d’une mise à jour `ASP.NET Fall 2012 Update` qu’il est possible d’installer depuis [cette page](http://www.asp.net/vnext/overview).

![image](/images/build-2012-quoi-de-neuf-dans-lupdate-aspnet-fall-2012/fall-update_422F1D0C.png)

### Tooling

On retrouve quelques nouveautés en terme de publication (même expérience pour les web applications que pour les websites, selective publish qui permet de choisir les fichiers à publier…) et de nouveaux templates de projets : Facebook, Single Page Application et FixedDisplayModes package.

### ASP.NET Web API OData

Il s’agit d’une extension à Web API qui permet de construire des services OData compliant (i.e. avec la possibilité d’appliquer des filtres directement dans l’URL). Cette fonctionnalité était déjà disponible via nuget en preview. Concrètement, c’est super simple à mettre en place, puisqu’il suffit d’exposer des IQueryable dans les ApiController et de précéder les actions qui doivent supporter OData par un attribut `Queryable` :

```csharp
[Queryable]
public IQueryable<Post> Get()
{
return _context.Posts;
}
```
### ASP.NET Web API Tracing

Il s’agit tout simplement de la mise en place par défaut de traces .NET pour les services ASP.NET Web API. On peut exploiter ces données de trace dans l’IntelliTrace.

###

### ASP.NET Web API Help Page

Fonctionnalité assez sympatique, puisqu’elle permet très simplement de générer des pages d’aide à l’utilisation des services ASP.NET Web API.

### Windows Azure Authentication

Il s’agit de la continuité d’une des annonces faites sur Azure lors du keynote : la possibilité de supporter dans une application ASP.NET de l’authentification via Windows Azure Active Directory ou Office 365.

### SignalR

SignalR est un framework permettant de développer des interfaces temps réel en javascript, notamment. Il permet d’optimiser les communications avec le serveur pour connaître (par exemple) l’avancement de certaines tâche en faisant du long polling plutôt que de lancer n requêtes http par secondes. Il est désormais supporté de base depuis cette mise à jour.

###

Voilà pour ce que l’on pouvait dire concernant les nouveautés de cette update ASP.NET. Plein de nouvelles choses sympa en perspective ! Au passage, les démos ont été faites avec une preview d’Entity Framework 6 qui supporte désormais le développement asynchrone basé sur les Task et les mots-clés async/await !! Cette preview est disponible sur nuget : <a title="http://nuget.org/packages/EntityFramework/6.0.0-alpha1" href="http://nuget.org/packages/EntityFramework/6.0.0-alpha1">http://nuget.org/packages/EntityFramework/6.0.0-alpha1</a>

Stay tuned <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_6D270B11.png">

Julien

