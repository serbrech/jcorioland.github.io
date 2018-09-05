---
layout: post
title: "[ASP.NET MVC] Authentification et utilisateurs anonymes"
date: 2011-08-22 15:37:00 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: 75211abb-bcdb-4e04-8d39-1b4d5cc8a91f
redirect_from:
  - /archives/aspnet-mvc-authentification-et-utilisateurs-anonymes
  - /Archives/aspnet-mvc-authentification-et-utilisateurs-anonymes
---

Lorsque l’on développe une application web nécessitant de l’authentification, plusieurs stratégies sont possibles, par exemple :

- On autorise l’accès anonyme partout, sauf sur les pages nécessitant que l’utilisateur soit authentifié
- On refuse l’accès anonyme partout sauf sur les pages ne nécessitant pas que l’utilisateur soit authentifié

Depuis ASP.NET MVC 2 il est possible de réaliser la première option très facilement. En effet, il suffit de placer un filtre d’authorisation sur les méthodes des contrôleurs (entendre action) pour lesquelles il faut que l’utilisateur soit authentifié. Il s’agit de l’attribut **AuthorizeAttribute** :

```csharp
[Authorize]
public ActionResult Admin()
{
//code
return View();
}
```

Sur certains projets, seules quelques pages ne nécessitent pas que l’utilisateur soit activée, il devient donc contraigant d’avoir à placer l’attribut Authorize sur TOUTES les méthodes de contrôleur. ASP.NET MVC 3 introduit un nouveau concept : les filtres globaux. Il s’agit en fait de la possibilité d’appliquer des filtres sur toutes les actions de tous les contrôleurs, par exemple.

Pour cela, il suffit de venir ajouter le filtre en question dans la liste des filtres globaux. Ceci se fait dans le fichier Global.asax. Depuis la version 3 d’ASP.NET MVC, une nouvelle méthode est générée : **RegisterGlobalFilters **:

```csharp
public static void RegisterGlobalFilters(GlobalFilterCollection filters)
{
filters.Add(new HandleErrorAttribute());
filters.Add(new AuthorizeAttribute());
}
```

Cette méthode est ensuite appelée dans le **Application_Start** du même fichier :

```csharp
protected void Application_Start()
{
AreaRegistration.RegisterAllAreas();

RegisterGlobalFilters(GlobalFilters.Filters);
RegisterRoutes(RouteTable.Routes);
}
```

Si vous testez ce code, vous verrez que plus rien ne s’affiche, même pas la page d’authentification puisque vous demandez que l’utilisateur soit authentifié partout.

Pour rendre le process un peu plus permissif, il suffit de créer un nouvel attribut AnonymousAttribute qui servira à marquer les actions ne nécessitant pas que l’utilisateur soit authentifié :

```csharp
[AttributeUsage(AttributeTargets.Method, AllowMultiple = false, Inherited = false)]
public class AnonymousAttribute : Attribute
{
}
```

A présent, il est possible de dériver le filtre AuthorizeAttribute et de surcharger la méthode OnAuthorization de celui-ci. Dans cette surcharge, il est possible de vérifier sur le contexte du filtre si l’action possède ou non l’attribut Anonymous décrit ci-dessus :

```csharp
public class AuthenticationRequiredAttribute : AuthorizeAttribute
{
public override void OnAuthorization(AuthorizationContext filterContext)
{
bool anonymousAllowed = filterContext.ActionDescriptor.IsDefined(typeof (AnonymousAttribute), false);

if (!anonymousAllowed)
base.OnAuthorization(filterContext);
}
}
```
Du coup, il suffit d’enregister ce filtre plutôt que le AuthorizeAttribute dans le Global.asax

```csharp
public static void RegisterGlobalFilters(GlobalFilterCollection filters)
{
filters.Add(new HandleErrorAttribute());
filters.Add(new AuthenticationRequiredAttribute());
}
```

Puis de placer l’attribut **Anonymous** partout ou vous souhaitez conserver une accès anonyme (par exemple les actions LogOn) :

```csharp
[Anonymous]
public ActionResult LogOn()
{
return View();
}

[HttpPost]
[Anonymous]
public ActionResult LogOn(LogOnModel model, string returnUrl)
{
//code
}
```

Du coup, le comportement désiré est bien obtenu : toutes les actions nécessitent que l’utilisateur soit authentifié, sauf celles où pour lesquelles l’accès anonyme est autorisé explicitement.

En espérant que cela serve à certains d’entre vous !

A bientôt ![image](/images/aspnet-mvc-authentification-et-utilisateurs-anonymes/ad93fd32-97b6-4147-b081-9089aec7b4c9.jpg)

