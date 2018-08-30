---
layout: post
title: "[ASPNET MVC] Génération d’URL absolue dans un environnement Windows Azure"
date: 2013-04-09 9:36:00 +02:00
categories: ASP.NET MVC, Windows Azure
author: "Julien Corioland"
identifier: aebe4fe6-04b4-499d-b3f4-c8b61180f145
redirect_from:
  - /archives/aspnet-mvc-generation-durl-absolue-dans-un-environnement-windows-azure
  - /Archives/aspnet-mvc-generation-durl-absolue-dans-un-environnement-windows-azure
---

C’est assez fréquent d’avoir à générer des URLs absolues dans une application ASP.NET MVC.

cela, il est possible de faire appel à l’UrlHelper, disponible sur n’importe quelle page ou n’importe quel contrôleur, via la propriété `Url` :

```csharp
@Url.Action("About", "Home", null, "http")
```
Si vous êtes sur un applicatif ayant un seul frontal web, aucun problème l’url qui sera générée sera complètement exploitable. Par contre, dès lors que vous serez sur un applicatif hébergée dans une ferme derrière un NLB, vous récupèrerez l’adresse de la machine (IP + port) et non le DNS qui va bien pour contacter votre site. Le code ci-dessus executé dans la fabrique Azure locale donne :

![image](/images/aspnet-mvc-generation-durl-absolue-dans-un-environnement-windows-azure/image_6293290C.png)

Bien que l’on appelle l’adresse sur le port 81 (NLB de la fabrique locale) on récupère bien une URL sur le port 82 !

Pour corriger ce problème, il faut générer l’URL relative en utilisant l’UrlHelper, puis se baser sur le Host header de la requête http courante pour récupérer la bonne adresse. Aussi, il est possible de créer des méthodes d’extensions à l’UrlHelper pour faire cela :

```csharp
public static class UrlHelperExtensions
{
public static string ToAzureComplientAbsoluteUrl(this UrlHelper helper, string action, string controller)
{
return helper.ToAzureComplientAbsoluteUrl(action, controller, null);
}

public static string ToAzureComplientAbsoluteUrl(this UrlHelper helper, string action, string controller, object routeValues)
{
var relativeUrl = helper.Action(action, controller, routeValues);
var host = helper.RequestContext.HttpContext.Request.Headers["Host"];
var scheme = helper.RequestContext.HttpContext.Request.Url.Scheme;

return string.Format("{0}://{1}{2}", scheme, host, relativeUrl);
}
}
```
Du coup, il est possible de l’utiliser de la manière suivante :

```csharp
@Url.ToAzureComplientAbsoluteUrl("About", "Home")
```
Et voilà le résultat :

![image](/images/aspnet-mvc-generation-durl-absolue-dans-un-environnement-windows-azure/image_5A2F53B5.png)

A présent, plus de problème !

Enjoy <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_0BBAC14B.png">

Julien

