---
layout: post
title: "[ASP.NET MVC] Gestion des erreurs HTTP"
date: 2011-09-27 11:19:00 +02:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: 17322703-6812-401e-b9b1-a7e329961070
redirect_from:
  - /archives/aspnet-mvc-gestion-des-erreurs-http
  - /Archives/aspnet-mvc-gestion-des-erreurs-http
---

Bien que cela soit un sujet qui paraissent essentiel et surtout `basic` lorsque l’on développe une application web (avec ASP.NET MVC ou pas d’ailleurs), ce sujet semble avoir fait couler beaucoup d’encre sur le web ! (si je puis dire…). En effet, il semble qu’il règne une sorte de `flou` autour de ce sujet ou chacun utilise une méthode différente pour gérer les erreurs HTTP. Je ne sais pas si on peut parler ici de `bonne pratique` mais en tout cas, ce post présente un moyen que je juge efficace et suffisament générique pour gérer les Http Exception dans une application ASP.NET MVC ! (n’hésitez pas à me faire par de vos remarques / suggestions en commentaire ![image](/images/aspnet-mvc-gestion-des-erreurs-http/3ce1b146-665b-440a-b9be-fd76de65ecbb.jpg))

### Quel est le réel besoin ?

Au final, pourquoi souhaite-t-on traiter les erreurs HTTP de manière différente des autres erreurs ? A mon humble avis, pour 3 raisons principalement :

<ol>- Le respect des standards (une page qui n’existe pas c’est un 404, c’est tout !)
- La SEO : les moteurs de recherche utilisent tous les codes HTTP (comme c’est standard) pour construire leurs index.
- L’utilisateur : on souhaite en général personnaliser la page d’erreur sur laquel arrive l’utilisateur !
</ol>Ce qui importe donc, c’est avant tout que le code d’erreur soit bien renvoyé : si j’appelle une URL `http://www.monsite.com/Pages/Page-qui-n-existe-pas` je souhaite que ce soit bien cette page là qui me retourne un code de status 404 et non pas un code 302 (trouvé) suivi d’un 200 (ou 404, à la limite, mais non) sur une URL de type `http://www.monsite.com/Errors/Error404`.

### Quand est-ce que des HttpException sont levées en ASP.NET MVC ?

Les exceptions sont levées soit par le moteur ASP.NET MVC, soit dans votre code :

```csharp
throw new HttpException(404, "Not found");
```

Elles peuvent également être levées au niveau du serveur (erreur interne 500, par exemple), mais dans tous les cas, il est possible de les gérer !

### Comment gérer ces exceptions ?

Après pas mal de recherches et de tests, je pense qu’il y a deux endroits où il faut gérer les exceptions :

<ol>
- Dans un contrôleur de base dérivé par tous les contrôleurs d’une application ASP.NET MVC

- Dans le fichier Global.asax

</ol>

*Gestion des erreurs dans un contrôleur de base :*

Au sein du contrôleur, on gèrera les exceptions qui ont été levées intentionnellement par le développeur. Par exemple :

```csharp
public ActionResult Product(int id)
{
var product = _unitOfWork.GetProduct(id);
if(product == null)
throw new HttpException(404, "Le produit est introuvable");

return View(product);
}
```

Lorsque l’on dérive la classe System.Web.Mvc.Controller, il est possible de surcharger une méthode `OnException` : celle-ci est appelée lorsqu’une exception est levée dans le contrôleur.

```csharp
protected override void OnException(ExceptionContext filterContext) {
base.OnException(filterContext);

if (filterContext.Exception != null) {
filterContext.ExceptionHandled = true;

if (filterContext.Exception is HttpException) {
if (!ControllerContext.RouteData.Values.ContainsKey("error")) {
ControllerContext.RouteData.Values.Add("error", filterContext.Exception);
}

var httpException = (HttpException) filterContext.Exception;

switch (httpException.GetHttpCode()) {
case 404:
filterContext.HttpContext.Response.StatusCode = 404;
filterContext.HttpContext.Response.StatusDescription = httpException.Message;
View("Error404", null).ExecuteResult(ControllerContext);
break;
case 500:
filterContext.HttpContext.Response.StatusCode = 500;
filterContext.HttpContext.Response.StatusDescription = httpException.Message;
View("Error500", null).ExecuteResult(ControllerContext);
break;
default:
filterContext.HttpContext.Response.StatusDescription = httpException.Message;
View("GenericError", null).ExecuteResult(ControllerContext);
break;
}
}

//autre traitement si pas HttpException (log par exemple...)
}
}
```

Comme vous pouvez le constater, ce code vérifie si l’exception est de type HttpException, si c’est le cas, elle redirige vers une vue d’erreur après avoir renseigner un status code et un status description au niveau de la réponse.

NB : ces vues sont placées dans le dossier `Shared` des vues partagées.

***Premier objectif atteind :*** une erreur HTTP levée dans un contrôleur se traduit par une page d’erreur `user friendly` et une réponse HTTP `SEO/Standards friendly` puisque retournant le bon code HTTP sur la bonne URL !

*Gestion des erreurs HTTP dans le Global.asax :*

Dans le Global.asax, on va gérer toutes les autres erreurs HTTP qui pourraient être levées dans l’application.

Pour se faire, on surcharge d’abord la méthode **Init** afin de s’abonner à l’événement **Error** :

```csharp
public override void Init()
{
base.Init();
this.Error += new EventHandler(MvcApplication_Error);
}
```

Dans la méthode **MvcApplication_Error**, on récupère la dernière erreur du serveur, on vérifie si celle-ci est de type HttpException. Si oui, on instancie un contrôleur spécialisé dans la gestion des erreurs (ici ErrorsController, détaillé plus bas) on exécute le rendu de la vue en lui passant les bonnes RouteData :

```csharp
var routeData = new RouteData();
routeData.Values.Add("controller", "Errors");

var lastException = Server.GetLastError();

if (lastException is HttpException) {
var httpException = (HttpException) lastException;
switch(httpException.GetHttpCode()) {
case 404:
routeData.Values.Add("action", "Error404");
break;
case 500:
routeData.Values.Add("action", "Error500");
break;
default:
routeData.Values.Add("action", "GenericError");
break;
}
}
else
{
routeData.Values.Add("action", "GenericError");
}

routeData.Values.Add("exception", lastException);

Server.ClearError();

IController errorController = _unityContainer.Resolve<ErrorsController>();
errorController.Execute(new RequestContext(
new HttpContextWrapper(Context), routeData));
```

Voilà le code du contrôleur **ErrorsController** :

```csharp
public class ErrorsController : Controller
{
public ActionResult Error404() {
Response.StatusCode = 404;
Exception exception = null;
if(RouteData.Values.ContainsKey("exception")) {
exception = (Exception) RouteData.Values["exception"];
}
return View(exception);
}

public ActionResult Error500() {
Response.StatusCode = 500;
Exception exception = null;
if (RouteData.Values.ContainsKey("exception"))
{
exception = (Exception)RouteData.Values["exception"];
}
return View(exception);
}

public ActionResult GenericError() {
Exception exception = null;
if (RouteData.Values.ContainsKey("exception"))
{
exception = (Exception)RouteData.Values["exception"];
}
return View(exception);
}
}
```

NB : notez que c’est dans chaque action que l’on défini le status code de la réponse HTTP !

Je vous passe le code des vues qui n’a pas grand intérêt (mise à part que l’on peut réutiliser ici les vues du dossier `Shared` évidemment)

***Deuxième objectif atteind :*** une erreur HTTP levée autre part que dans un contrôleur se traduit par une page d’erreur `user friendly` et une réponse HTTP `SEO/Standards friendly` puisque retournant le bon code HTTP sur la bonne URL !

A bientôt ! ![image](/images/aspnet-mvc-gestion-des-erreurs-http/e66b5970-7a8a-4db8-ad7b-abcf794a1208.jpg)

