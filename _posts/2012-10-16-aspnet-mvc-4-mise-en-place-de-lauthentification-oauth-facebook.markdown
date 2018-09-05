---
layout: post
title: "[ASP.NET MVC 4] Mise en place de l’authentification OAuth Facebook"
date: 2012-10-16 12:25:47 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: 1a17cfd0-2f0c-4ce8-b8b0-a2ee5baeb410
redirect_from:
  - /archives/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook
  - /Archives/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook
---

L’une des nouveautés d’ASP.NET MVC 4 est la possibilité de mettre en place très simplement de l’authentification au travers de fournisseurs d’identité tels que Facebook, Microsoft, Google (etc.) dans une application. Ce type d’authentification est d’ailleurs proposée de base dans les nouveaux modèles de projets d’application ASP.NET MVC 4 de Visual Studio. Cependant, afin de bien comprendre comment mettre en place les différentes briques qui permettront cette authentification, je vous propose de partir de zéro et de créer un projet à l’aide du modèle `Basic` :

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/b195631d-1f36-434a-80d8-996089708f9b.jpg)

##

### Déroulement de l’authentification

Le processus d’authentification à l’aide d’un fournisseur d’identité externe se déroule en plusieurs étapes décrites sur le schéma ci-dessous :

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/af0ba7c5-3095-44f7-ba3e-f28617eea35f.jpg)

Dans un premier temps, l’utilisateur choisi le fournisseur d’identité (parmi ceux proposés par l’application) avec lequel il souhaite s’authentifier. Il est alors redirigé vers la page d’authentification du fournisseur choisi. L’utilisateur s’authentifie et est renvoyé sur l’application à l’aide d’une URL de callback qui a été transmise au fournisseur d’identité lors de la venue de l’utilisateur. Dans la dernière étape, l’application valide l’utilisateur et récupère son identité (établie par le fournisseur).

Il est alors possible de récupérer 4 informations :

- L’état de l’authentification (échec ou succès)
- Le nom d’utilisateur (si succès, transmis par le fournisseur)
- L’id de l’utilisateur (si succès, transmis par le fournisseur)
- Le nom du fournisseur

### Ajout des références nécessaires à la mise en place de l’authentification OAuth

La mise en place de l’authentification OAuth requière l’ajout de plusieurs références au projet. Il est possible d’en récupérer la plupart à l’aide du package nuget  [DotNetOpenOAuth extensions for ASP.NET (WebPages)](https://nuget.org/packages/DotNetOpenAuth.AspNet/4.0.4.12182). Une fois le package installé, il ne reste qu’à ajouter une référence vers la librairie<em> Microsoft.Web.WebPages.OAuth.dll</em> depuis la fenêtre d’ajout de référence de Visual Studio.

Voilà les références que vous devriez avoir à ce stade :

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/04e2dc3b-c8bb-4772-969e-e5cdb5007231.jpg)

### Configuration des fournisseurs d’identités

Afin que l’authentification puisse fonctionner, il est nécessaire de configurer les différents fournisseurs d’identités à utiliser dans l’application. Dans le cas présent, nous allons voir comment configurer Facebook.

Pour commencer, créez une nouvelle classe dans le dossier App_Start du projet. Nommez la <em>OAuthConfig</em>. Cette classe possède une méthode statique RegisterOAuthProviders qui sera en charge d’initialiser les différents fournisseurs d’identités au démarrage de l’application :

```csharp
public class OAuthConfig
{
public static void RegisterOAuthProviders()
{
//enregistrement des fournisseurs d'identité
}
}
```

Appelez cette méthode dans le Global.asax :

```csharp
public class MvcApplication : System.Web.HttpApplication
{
protected void Application_Start()
{
AreaRegistration.RegisterAllAreas();

WebApiConfig.Register(GlobalConfiguration.Configuration);
FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
RouteConfig.RegisterRoutes(RouteTable.Routes);
BundleConfig.RegisterBundles(BundleTable.Bundles);

//enregistrement des fournisseurs d'identités
OAuthConfig.RegisterOAuthProviders();
}
}
```

L’enregistrement des différents fournisseurs va se faire à l’aide de la classe <em>OAuthWebSecurity</em> définie dans l’espace de noms <em>Microsoft.Web.WebPages.OAuth</em>. Celle-ci possède un certain nombre de méthodes statiques pour enregistrer les différents fournisseurs supportés nativement (<em>RegisterFacebookClient</em>, <em>RegisterMicrosoftClient</em>…).

Configuration de l’authentification Facebook

Avant de procéder à l’enregistrement du fournisseur, il est nécessaire de se connecter sur <a title="https://developers.facebook.com/apps" href="https://developers.facebook.com/apps">https://developers.facebook.com/apps</a> et de créer une nouvelle application. Pour se faire, cliquez sur le bouton `<em>+ Create New App</em>` en haut à droite. Donnez un nom à l’application et validez en cliquant sur `<em>Continue`</em>.

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/34eea680-d603-459c-aadd-253b252bd441.jpg)

Entrez le code de sécurité pour terminer la création. Pour que l’authentification fonctionne, il est nécessaire de cocher `Website with Facebook Login` en bas de page et de renseigner l’URL du site web qui va utiliser l’application Facebook pour l’authentification. En l’occurrence ici votre URL de débogue dans Visual Studio :

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/59ca7598-4e6b-449f-8afb-66ce1c29b4ef.jpg)

Validez en cliquant sur <em>Save Changes</em>.

Il vous faut alors récupérer l’App ID et l’App Secret en haut de la page, afin d’enregistrer le fournisseur dans Visual Studio.

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/83aa0cb2-b715-4b91-8978-931af2865a7f.jpg)

Pour enregistrer le fournisseur, il suffit d’appeler la méthode <em>RegisterFacebookClient</em> de la classe <em>OAuthWebSecurity</em> :

```csharp
OAuthWebSecurity.RegisterFacebookClient(
appId:"[appId]",
appSecret:"[appSecret]",
displayName: "Facebook"
);
```

### Création de la page de Login

Ajoutez un contrôleur `<em>Account</em>` et créez une action **Login**. Le modèle passé à la vue correspond à la liste des fournisseurs configurés (ici Facebook), représentée par la propriété <em>RegisteredProviders</em> sur la classe <em>OAuthWebSecurity</em> :

```csharp
public class AccountController : Controller
{
public ActionResult Login()
{
return View(OAuthWebSecurity.RegisteredClientData);
}
}
```

La page affiche alors un bouton pour chaque fournisseur :

```csharp
@model ICollection<AuthenticationClientData>

@{
ViewBag.Title = "Authentification";
}

## Choisissez un fournisseur d'identité pour vous identifier

@using (Html.BeginForm("Login", "Account", FormMethod.Post))
{
foreach (var provider in Model)
{
<input type="submit" name="provider" value="@provider.DisplayName" />

}
}
```

Le code appelé lors du POST du formulaire effectue la redirection vers le fournisseur sélectionné, en lui passant la callback URL (appelée après l’authentification par le fournisseur d’identité) :

```csharp
[HttpPost]
public ActionResult Login(string provider)
{
string callbackUrl = Url.Action("ProviderCallback", "Account");
OAuthWebSecurity.RequestAuthentication(provider);
return new EmptyResult();
}
```

Lancez l’application, rendez-vous sur la page /Account/Login et vérifiez qu’un bouton Facebook apparait bien :

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/37404d47-86a1-4e09-b556-f1892eec027d.jpg)

En cliquant sur le bouton, vous devriez être redirigé sur Facebook. Authentifiez-vous, acceptez d’utiliser l’application. Vous devriez alors être renvoyé sur votre site (sur l’action ProviderCallback). Pour le moment, vous devez obtenir une erreur 404.

![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/59573716-7840-441a-b8cd-902954295dad.jpg)

Dans l’action de callback, il s’agit de vérifier que l’authentification a bien fonctionné. Pour cela, il faut utiliser la méthode VerifyAuthentication sur la classe OAuthWebSecurity. Si le résultat est valide, vous pouvez alors récupérer le nom et l’id de l’utilisateur et créer une session (à l’aide de la classe FormsAuthentication, par exemple) :

```csharp
public ActionResult ProviderCallback()
{
var result = OAuthWebSecurity.VerifyAuthentication();
if (result.IsSuccessful)
{
string userName = result.UserName;
string userId = result.ProviderUserId;
FormsAuthentication.SetAuthCookie(userName, false);
return RedirectToAction("Index", "Home");
}
else
{
return RedirectToAction("Login", "Account");
}
}
```

Et voilà, le tour est joué, votre utilisateur est authentifié via Facebook.

Le mécanisme d’authentification est ultra générique ici, puisqu’il suffit de rajouter des fournisseurs (Microsoft, Google, Twitter, Yahoo…) dans la méthode RegisterOAuthProviders de la classe OAuthConfig créée au début de cet article !

Le code source de l’application d’exemple est disponible ici : <a title="https://juliencorioland.blob.core.windows.net/publicfiles/MVC4.ArticleOAuth.zip" href="https://juliencorioland.blob.core.windows.net/publicfiles/MVC4.ArticleOAuth.zip">https://juliencorioland.blob.core.windows.net/publicfiles/MVC4.ArticleOAuth.zip</a>

A bientôt ![image](/images/aspnet-mvc-4-mise-en-place-de-lauthentification-oauth-facebook/3c376f03-8d13-4260-95e1-ed16dab5d470.jpg)

