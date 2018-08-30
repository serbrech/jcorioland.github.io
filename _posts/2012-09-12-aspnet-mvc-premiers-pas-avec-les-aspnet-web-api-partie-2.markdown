---
layout: post
title: "[ASP.NET MVC] Premiers pas avec les ASP.NET Web API - Partie 2"
date: 2012-09-12 18:56:00 +02:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: eecbc6bd-1251-4595-9d0a-d4007f7ef988
redirect_from:
  - /archives/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-2
  - /Archives/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-2
---

Dans [mon article précédent](http://www.juliencorioland.net/Archives/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api---partie-1), je vous expliquais comment mettre en place les nouveaux contrôleurs `Web API` pour exposer vos données de manière `RESTful`. Je m’étais arrêté à la partie ASP.NET MVC Web API pure. Dans cet article, je souhaite fournir un exemple d’interface graphique venant consommer ces APIs.

J’ai choisi de développer l’interface en HTML/jQuery pur au sein du même projet que celui hébergeant mon API contrôleur. Ce qu’il faut bien comprendre ici, c’est que j’aurai pu faire le choix de n’importe quelle technologie capable d’envoyer une requête HTTP et de traiter du JSON (du texte, au final) : Windows Phone, Windows 8, iPhone, PHP… Bref, n’importe quelle plateforme de développement suffisamment récente (ou presque!).

###

### Définition de l’interface graphique

L’interface graphique que j’ai choisi est ultra simple :

```xml
<div style="margin: 20px">
## Consommation des WEB Api

### Liste des requêtes

<a id="showRequestsLink" title="afficher les requêtes" href="#">afficher les requêtes</a>

<div id="requestsList">

### Ajouter une requête

<div id="addFormContainer">
<form id="addRequestForm">
Titre : <input type="text" name="Title" id="Title" />
Description : <input type="text" name="Description" id="Description" />
<input type="hidden" name="Id" id="Id" value="" />
<input type="button" id="addRequestButton" value="Ajouter" />
</form>

```

Voilà ce que cela donne dans un navigateur :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-2/5a711f69-815d-43ee-94ee-ef266dccf67a.jpg)

Ensuite, tout se passe en ajax (et en HTTP!!).

### Affichage de la liste des requêtes

Lorsque l’utilisateur clique sur le lien pour afficher la liste des requêtes, il est nécessaire de faire un `GET` sur l’URL /api/featurerequest, ce qui aura pour effet d’appeler l’action Get de l’ApiController développé dans [l’article précédent](http://www.juliencorioland.net/Archives/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api---partie-1).

```js
function updateRequests() {
$.get("/api/featurerequests", function (requests) {
if (requests.length == 0) {
$("#requestsList").html("aucune requête dans la base de données

");
}
else {
$("#requestsList").html("");
$("#requestTemplate").tmpl(requests).appendTo("#requestsList");

handleLinksClicks();
}
});
}

$(document).ready(function () {

$("#showRequestsLink").click(function (evt) {
updateRequests();
});
});
```

L’appel en GET de l’url /api/featurerequests retourne directement un tableau d’objet JavaScript qui possède les mêmes propriétés que l’objet server `FeatureRequest`, celui-ci ayant automatiquement été sérialisé en JSON par le moteur Web API. Notez au passage que j’utilise ici le plugin jQuery tmpl, permettant d’effectuer un rendu à partir d’un template HTML. Vous pouvez récupérer directement ce plugin via la console Nuget de Visual Studio :

![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-2/643c3277-3b94-4915-8885-f8ef20b4c811.jpg)

Voilà la structure du template que j’ai défini :

```xml
<script id="requestTemplate" type="text/html">
{{= Title }} - [Détail](#) | [Détail](#) | [Détail](#)

</script>
```

Note : chacune des propriétés de l’objet JSON peut être appelé à l’aide de la notation {{= PropertyName }} au sein du template ! (au passage, il s’agit d’un plugin jQuery développé par Microsoft ![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-2/8a8d04ae-9710-4b08-bf7e-74fad62e6b2e.jpg), on sent l’influence du binding XAML)

###

### Ajouter/Mettre à jour une requête dans la base de données

Pour ajouter une nouvelle requête dans la base de données, il suffit de poster le formulaire sur l’URL /api/featurerequests. La requête sera alors automatiquement traitée par l’action qui lui correspond dans le contrôleur développé. Pour mettre à jour une requête, il suffit d’envoyer une requête HTTP PUT sur la même URL, avec les données du formulaire, sérialisées :

```js
$("#addRequestButton").click(function (evt) {
evt.preventDefault();
var addRequestForm = $("#addRequestForm");
if ($("#Id").val() != "") {
$.ajax({
url: "/api/featurerequests/" + $("#Id").val(),
type: "PUT",
data: addRequestForm.serialize(),
success: function () {
updateRequests();
$("#Id").val("");
$("#Title").val("");
$("#Description").val("");
$("#addRequestButton").val("Ajouter");
},
error: function () {
alert("Erreur lors de la mise à jour");
}
});
}
else {
$.post("/api/featurerequests", addRequestForm.serialize())
.success(function (result) {
$("#Title").val("");
$("#Description").val("");
updateRequests();
})
.error(function () {
alert("Une erreur s'est produite!");
});
}
});
```

###

### Récupérer les informations propres à une requête

Pour récupérer une information propre à une requête, il suffit d’envoyer une requête HTTP GET à l’url /api/featurerequests/id :

```js
$(".request-detail-link").click(function () {
var dataId = $(this).data("id");
var url = "/api/featurerequests/" + dataId;
$.get(url, function (jsonResult) {
alert(JSON.stringify(jsonResult));
});
});
```

### Suppression d’une requête

Cette fonctionnalité est similaire à la récupération d’informations propres à une requête, à la différence près qu’il faut envoyer une requête HTTP DELETE :

```js
$(".request-delete-link").click(function () {
var dataId = $(this).data("id");
if (confirm("Êtes-vous sûr de supprimer cette requête ?")) {
$.ajax({
url: "/api/featurerequests/" + dataId,
type: "DELETE",
success: function () {
alert("La requête a été supprimée");
updateRequests();
}
});
}
});
```

### Conclusion / Résumé

L’important ici n’était pas tant le code JavaScript / HTML, mais plutôt de voir comment consommer très simplement les données à l’aide de requête HTTP et donc potentiellement depuis n’importe quelle plateforme récente ![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-2/3db188c9-eb87-4ef2-9f41-8f240fbbed01.jpg)

Ci-dessous un résumé des différentes requêtes / verbes et de leurs actions respectives dans le contrôleur :

- GET /api/featurerequests : <em>GetFeatureRequests</em> –> retourne la liste des requêtes

- GET /api/featurerequests/id : <em>GetFeatureRequest </em>–> retourne une requête en particulier

- POST /api/featurerequests : <em>PostFeatureRequest </em>–> ajouter une requête dans la base

- PUT /api/featurerequests : <em>PutFeatureRequest</em> –> mise à jour d’une requête dans la base

- DELETE /api/featurerequests/id : <em>DeleteFeatureRequest</em> –> suppression d’une requête de la base

Vous pouvez récupérer le code source [ici](https://skydrive.live.com/redir?resid=739925FC64062118!1042&authkey=!AFhvBvJnMmVVJHo).

A bientôt ! ![image](/images/aspnet-mvc-premiers-pas-avec-les-aspnet-web-api-partie-2/8a8d04ae-9710-4b08-bf7e-74fad62e6b2e.jpg)

