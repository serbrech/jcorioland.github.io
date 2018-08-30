---
layout: post
title: "[ASP.NET MVC] Premiers pas avec les ASP.NET Web API - Partie 1"
date: 2012-08-17 16:54:00 +02:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: 82c5c304-a88e-4786-a87b-b8cd1ae8bb7e
redirect_from:
  - /archives/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1
  - /Archives/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1
---

### Introduction

ASP.NET MVC 4 a été rendu disponible par Microsoft il y a peu et apporte son lot de nouveautés. Parmi celles-ci, on constate l’apparition d’une nouvelle brique : ASP.NET Web API. Il s’agit d’un Framework qui permet de développer rapidement et simplement des services HTTP pour la mise à disposition de données cross-plateforme et le développement d’application RESTful !

Dans ce post, je vous propose de créer une application reposant sur ce Framework et permettant de poster des requêtes de fonctionnalités / feedback pour un produit, comme le fait par exemple [Uservoice.com](http://www.uservoice.com/) (en beaucoup plus simple, évidemment).

Avant de commencer, il vous faut installer ASP.NET MVC 4. Pour cela, rendez-vous sur [cette page](http://www.asp.net/mvc/mvc4) et cliquez sur **Install ASP.NET MVC 4**.

<em>*Note :* il est possible de développer des applications ASP.NET MVC 4 / ASP.NET Web API aussi bien dans Visual Studio 2010 que 2012, avec le .NET Framework 4.0 ou 4.5. Pour ma part, j’utiliserai Visual Studio 2012 et le .NET Framework 4.5.</em>

### Création du projet

Dans Visual Studio, créez un nouveau projet Web, de type **ASP.NET MVC 4 Web Application** :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1/b40426c4-4c19-4a0a-8825-45cfc6813b17.jpg)

Dans la fenêtre qui suit, choisissez un modèle de projet de type **Web API**. Gardez Razor comme moteur de vue, et validez la création du projet :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1/893325b2-91b2-4a75-8867-1fb4157da6ea.jpg)

### Analyse de la solution

Avant d’ajouter la moindre ligne de code au projet, nous allons nous intéresser à ce qui a été généré par Visual Studio. S’il s’agit de votre premier projet ASP.NET MVC 4, vous devez remarquer qu’un nouveau dossier App_Start est présent dès la création du projet. Celui-ci contient un ensemble de classes utilisées pour l’initialisation de l’application, dans la méthode Application_Start du fichier Global.asax!

Visual Studio a également créé deux contrôleurs :

**HomeController** : il s’agit d’un contrôleur ASP.NET MVC classique, composé d’une action Index pour renvoyer l’utilisateur sur la page d’accueil lorsqu’il se connecte sur le site. C’est ce contrôleur et cette action qui sont appelés lorsque vous lancez le site web en débogue.

**ValuesController** : il s’agit d’un contrôleur ASP.NET Web API d’exemple. Celui-ci est composé de 5 méthodes permettant d’adresser différentes actions : récupérer une liste de valeur, récupérer une valeur par son id, ajouter une valeur, modifier une valeur, supprimer une valeur. Remarquez qu’à chacune de ces actions est associé un verbe HTTP : **GET** pour la récupération, **POST** pour l’ajout, **PUT** pour la modification et **DELETE** pour la suppression.

```csharp
// GET api/values
public IEnumerable<string> Get()
{
return new string[] { "value1", "value2" };
}

// GET api/values/5
public string Get(int id)
{
return "value";
}

// POST api/values
public void Post([FromBody]string value)
{
}

// PUT api/values/5
public void Put(int id, [FromBody]string value)
{
}

// DELETE api/values/5
public void Delete(int id)
{
}
```

Lancez l’application en débogue en appuyant sur F5.

Dans la barre d’adresse du navigateur, connectez-vous sur [/api/values">http://localhost:<port>/api/values](http://localhost:<port>/api/values) :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1/f53e0ce5-e3a4-45f3-a29c-51381e3b8784.jpg)

Vous pourrez alors constater que votre navigateur vous propose de télécharger un fichier `.json` contenant la liste de valeurs retournées par la méthodes Get du contrôleur ValuesController, au format JSON : ["value1","value2"].

ASP.NET Web API utilise le même principe de routes qu’ASP.NET MVC, par défaut, toutes les URLs contenant /api/quelque chose sont redirigé vers un contrôleur de type ApiController. Ensuite, la résolution de l’action (méthode) et de ses éventuels paramètres fonctionne comme ASP.NET MVC. Le verbe HTTP utilisé est également très important puisque déterminant la méthode à appeler.

La route `api` par défaut est définie dans le fichier **WebApiConfig** du dossier** App_Start** et est initialisée dans le fichier Global.asax, au démarrage de l’application :

```csharp
public static class WebApiConfig
{
public static void Register(HttpConfiguration config)
{
config.Routes.MapHttpRoute(
name: "DefaultApi",
routeTemplate: "api/{controller}/{id}",
defaults: new { id = RouteParameter.Optional }
);
}
}
```

### Création du modèle

Dans cette partie, nous allons créer le modèle chargé de représenter une requête de fonctionnalité dans l’application d’exemple. Faites un clic droit sur le dossier Models dans l’explorateur de solution et ajoutez une classe FeatureRequest :

```csharp
public class FeatureRequest
{
public int Id { get; set; }
public string Title { get; set; }
public string Description { get; set; }
}
```

Compilez simplement l’application afin que les outils ASP.NET MVC puissent détecter cette nouvelle classe dans la suite.

### Ajout d’un nouveau contrôleur Web API

Faites un clic droit sur le dossier Controllers dans l’explorateur de solution et choisissez d’ajouter un nouveau contrôleur. Dans la fenêtre qui s’ouvre, nommez le **FeatureRequestsController**. Dans les options de scaffholding, choisissez le template <em>API controller with read/write actions, using Entity Framework</em>, le Model <em>FeatureRequest </em>et de <em>créer un nouveau DataContext EntityFramework</em> pour accéder aux données :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1/c692f11d-9129-463d-82cf-73c07464b684.jpg)

Nommez votre contexte EF dans la fenêtre qui s’affiche :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1/0120fd6a-2cd5-47a7-b4dd-0a6115f5f221.jpg)

Validez en cliquant sur **OK** puis **Add**.

Les outils ASP.NET MVC génèrent alors deux classes pour vous : FeatureRequestController.cs (dossier Controllers) et FeatureRequestWebAPIContext.cs (dossier Models). La première représente le nouveau contrôleur alors que la seconde représente le contexte Entity Framework pour l’accès aux données. Celui-ci est utilisé dans le contrôleurs, dans les différentes méthodes générées pour la récupération de la liste des requêtes, d’une requête en particulier, la création d’une requête, la modification d’une requête et enfin la suppression d’une requête.

```csharp
ublic class FeatureRequestsController : ApiController
{
private FeatureRequestWebAPIContext db = new FeatureRequestWebAPIContext();

// GET api/FeatureRequests
public IEnumerable<FeatureRequest> GetFeatureRequests()
{
return db.FeatureRequests.AsEnumerable();
}

// GET api/FeatureRequests/5
public FeatureRequest GetFeatureRequest(int id)
{
FeatureRequest featurerequest = db.FeatureRequests.Find(id);
if (featurerequest == null)
{
throw new HttpResponseException(Request.CreateResponse(HttpStatusCode.NotFound));
}

return featurerequest;
}

// PUT api/FeatureRequests/5
public HttpResponseMessage PutFeatureRequest(int id, FeatureRequest featurerequest)
{
if (ModelState.IsValid && id == featurerequest.Id)
{
db.Entry(featurerequest).State = EntityState.Modified;

try
{
db.SaveChanges();
}
catch (DbUpdateConcurrencyException)
{
return Request.CreateResponse(HttpStatusCode.NotFound);
}

return Request.CreateResponse(HttpStatusCode.OK);
}
else
{
return Request.CreateResponse(HttpStatusCode.BadRequest);
}
}

// POST api/FeatureRequests
public HttpResponseMessage PostFeatureRequest(FeatureRequest featurerequest)
{
if (ModelState.IsValid)
{
db.FeatureRequests.Add(featurerequest);
db.SaveChanges();

HttpResponseMessage response = Request.CreateResponse(HttpStatusCode.Created, featurerequest);
response.Headers.Location = new Uri(Url.Link("DefaultApi", new { id = featurerequest.Id }));
return response;
}
else
{
return Request.CreateResponse(HttpStatusCode.BadRequest);
}
}

// DELETE api/FeatureRequests/5
public HttpResponseMessage DeleteFeatureRequest(int id)
{
FeatureRequest featurerequest = db.FeatureRequests.Find(id);
if (featurerequest == null)
{
return Request.CreateResponse(HttpStatusCode.NotFound);
}

db.FeatureRequests.Remove(featurerequest);

try
{
db.SaveChanges();
}
catch (DbUpdateConcurrencyException)
{
return Request.CreateResponse(HttpStatusCode.NotFound);
}

return Request.CreateResponse(HttpStatusCode.OK, featurerequest);
}

protected override void Dispose(bool disposing)
{
db.Dispose();
base.Dispose(disposing);
}
}
```

<em>**Note** : remarquez à nouveau que le verbe HTTP utilisé entre en jeu dans la convention de nommage des actions du contrôleur !</em>

Si vous exécutez le site web en débogue et vous connectez à l’adresse [/api/featurerequests">http://localhost:<port>/api/featurerequests](http://localhost:<port>/api/featurerequests), votre navigateur devrait vous proposer de télécharger un fichier JSON contenant la liste des requêtes de fonctionnalités (vide pour le moment, évidemment !) :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1/bfb4863b-8188-451c-ba6a-1ce463c1b445.jpg)

### Conclusion

Dans cette première partie, vous avez mis en place la structure de l’application ASP.NET Web API pour la gestion des requêtes de fonctionnalités.

Dans le prochain article, nous développeront l’interface graphique qui sera en charge de consommer ce service RESTful !

A bientôt ![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-1/32431328-1be7-4fdf-bb2a-49c58e5bb233.jpg)

