---
layout: post
title: "[ASP.NET MVC] Bonnes pratiques : création d’un Bootstrapper ou comment isoler l’initialisation d’une application"
date: 2012-03-11 11:55:00 +01:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: 47ed9e4b-410c-4491-b954-41f09b403805
redirect_from:
  - /archives/aspnet-mvc-bonnes-pratiques-creation-dun-bootstrapper-ou-comment-isoler-linitialisation-dune-application
  - /Archives/aspnet-mvc-bonnes-pratiques-creation-dun-bootstrapper-ou-comment-isoler-linitialisation-dune-application
---

Dans ce post, je souhaite revenir sur une bonne pratique que [Rui](http://rui.fr) et moi-même avons évoqué lors de notre session sur [Rui](http://rui.fr) le mois dernier : l’isolation de l’initialisation d’une application ASP.NET MVC.

En effet, le template par défaut de création de projet ASP.NET MVC de Visual Studio concentre tout le code dans un seul et même projet ce qui peut poser des problèmes de maintenabilité et/ou d’organisation, notamment sur un projet conséquent.

L’idée est de créer un Bootstrapper, c’est à dire une entité qui soit capable d’initialiser l’application, charger les différentes dépendances via un conteneur IoC (Unity, dans cet exemple), enregistrer les différentes routes, areas (etc…) qui composent l’application.

Voilà l’architecture globale de ma solution d’exemple (sources disponibles en bas de page!) :

![image](/images/aspnet-mvc-bonnes-pratiques-creation-dun-bootstrapper-ou-comment-isoler-linitialisation-dune-application/8f8cbfc8-1515-48ef-a563-71eb36cb012b.jpg)

- **Sample.Bootstrapper** est une bibliothèque de classe qui référence : System.Web, System.Web.Mvc et System.Web.Routing et Microsoft.Practices.Unity, afin d’être en capacité d’initialiser l’application
- **Sample.Services** est une bibliothèque de classes qui défini un service tout simple, représenté par une interface et une implémentation concrète.
- **Sample.Web** est l’application ASP.NET MVC

Vous l’avez surement compris, l’essentiel du travail va se concentrer dans le projet Sample.Bootstrapper.

En premier lieu, le UnityBootstrapper : il s’agit d’une classe qui expose un conteneur Unity au reste de l’application, c’est celui-ci qui est utilisé pour la résolution des dépendances au runtime :

```csharp
public class UnityBootstrapper
{
private UnityBootstrapper()
{

}

private static void InitializeContainer(IUnityContainer container)
{
container.RegisterType<ISampleService, LoremIpsumSampleService>();
}

private static readonly object _syncRoot = new object();
private static IUnityContainer _container;

public static IUnityContainer Container
{
get
{
if (_container == null)
{
lock (_syncRoot)
{
if (_container == null)
{
_container = new UnityContainer();
InitializeContainer(_container);
}
}
}

return _container;
}
}
}
```

On expose ici le conteneur Unity racine de l’application sous la forme d’un Singleton. Les registers sont fait dans le code ou dans le fichier Web.config, selon le besoin.

Afin de pouvoir utiliser l’injection de dépendance dans l’application, il faut configurer le moteur MVC pour utiliser ce conteneur Unity lors de la résolution des contrôleur. Pour cela, on passe par une implémentation de l’interface IControllerActivator, ici la classe UnityControllerActivator :

```csharp
public class UnityControllerActivator : IControllerActivator
{
private readonly IUnityContainer _container;
public UnityControllerActivator(IUnityContainer unityContainer)
{
_container = unityContainer;
}

public IController Create(System.Web.Routing.RequestContext requestContext, Type controllerType)
{
return _container.Resolve(controllerType) as IController;
}
}
```

Il ne reste plus qu’à définir la bonne controller factory afin que le UnityControllerActivator soit utilisé.

Avant cela, j’attire votre attention sur la dernière classe qui se trouve dans le projet Sample.Bootstrapper : MvcApplication. Il s’agit en réalité de la classe d’application (qui dérive de System.Web.HttpApplication) et qui se trouve par défaut liée au fichier Global.asax, lors de la création du projet. Le fait de déporter cette classe dans un projet externe permet de bien isoler le chargement de l’application et d’y définir les différentes routes, areas ou comportement du Framework, par exemple, demander au ControllerBuilder d’aller utiliser notre propre IControllerActivator :

```csharp
public class MvcApplication : HttpApplication
{
public static void RegisterGlobalFilters(GlobalFilterCollection filters)
{
filters.Add(new HandleErrorAttribute());
}

public static void RegisterRoutes(RouteCollection routes)
{
routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

routes.MapRoute(
"Default",
"{controller}/{action}",
new { controller = "Sample", action = "Index" }
);

}

protected void Application_Start()
{
AreaRegistration.RegisterAllAreas();

RegisterGlobalFilters(GlobalFilters.Filters);
RegisterRoutes(RouteTable.Routes);

var controllerFactory = new DefaultControllerFactory(
new UnityControllerActivator(UnityBootstrapper.Container));
ControllerBuilder.Current.SetControllerFactory(controllerFactory);
}
}
```

Il ne reste plus qu’à lier cette classe au fichier Global.asax afin qu’elle soit prise en compte au démarrage de l’application. Pour cela, il suffit d’ouvrir le fichier à l’aide de l’éditeur XML de Visual Studio, et de modifier la définition de la classe d’application :

```xml
<%@ Application Codebehind="Global.asax.cs" Inherits="Sample.Bootstrapper.MvcApplication" Language="C#" %>
```

Et voilà, le tour est joué! Les sources sont disponibles ici : <a title="http://juliencorioland.blob.core.windows.net/publicfiles/Sample-Bootstrapper.zip" href="http://juliencorioland.blob.core.windows.net/publicfiles/Sample-Bootstrapper.zip">http://juliencorioland.blob.core.windows.net/publicfiles/Sample-Bootstrapper.zip</a>

A bientôt ![image](/images/aspnet-mvc-bonnes-pratiques-creation-dun-bootstrapper-ou-comment-isoler-linitialisation-dune-application/42bbc110-92e7-44cc-8552-595c4ba6161f.jpg)

Julien

