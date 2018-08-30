---
layout: post
title: "[ASP.NET MVC] Utilisation des préfixes pour la génération des noms et id des champs"
date: 2013-02-06 9:58:00 +01:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: fd856000-5b8d-40f1-8a5e-75a1622dc6bb
redirect_from:
  - /archives/aspnet-mvc-utilisation-des-prefixes-pour-la-generation-des-noms-et-id-des-champs
  - /Archives/aspnet-mvc-utilisation-des-prefixes-pour-la-generation-des-noms-et-id-des-champs
---

Il n’est pas rare d’avoir plusieurs formulaires sur une même page HTML, et d’autant plus lorsque l’on compose une vue dynamiquement à partir de vues partielles. Du coup, il est vite possible de se retrouver avec des inputs qui possèdent les mêmes noms et mêmes id. Dans ce cas, c’est l’arrachage de cheveux assuré lors du debug de la manipulation du DOM avec jQuery <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_4068C163.png">

Heuresement, <strike>la nature</strike> ASP.NET MVC est bien fait ! Il est en effet possible de préciser un préfixe à appliquer lors de la génération des inputs à l’aide des html helpers. Au sein d’une action de contrôleur, il suffit d’affecter la propriété **<em>HtmlFieldPrefix</em>** de la propriété **<em>TemplateInfo</em>** du ViewData, comme suivant :

```csharp
public class ClientController : Controller
{
public ActionResult Ajouter()
{
ViewData.TemplateInfo.HtmlFieldPrefix = "AjouterClient";
return View(new Client());
}
}
```
Du coup, en utilisant correctement les Html Helpers pour générer les champs de formulaires (TextBox, CheckBox, DropDownList, Editor etc…), ASP.NET MVC tient compte de ce préfixe pour nommer les champs.

```xml
@model MvcApplication2.Models.Client

@{
ViewBag.Title = "Ajouter un client";
}

## Ajouter un client

@using (Html.BeginForm()) {

<div class="editor-label">
@Html.LabelFor(model => model.Prenom)

<div class="editor-field">
@Html.EditorFor(model => model.Prenom)

<div class="editor-label">
@Html.LabelFor(model => model.Nom)

<div class="editor-field">
@Html.EditorFor(model => model.Nom)

<div class="editor-label">
@Html.LabelFor(model => model.Adresse)

<div class="editor-field">
@Html.EditorFor(model => model.Adresse)

<input type="submit" value="Create" />

}

@section Scripts {
@Scripts.Render("~/bundles/jqueryval")
}
```
Le template razor précédent génère alors le markup HTML qui suit :

```xml
<div class="editor-field">
<input id="AjouterClient_Prenom" name="AjouterClient.Prenom" type="text" value="" />

<div class="editor-label">
<label for="AjouterClient_Nom">Nom :</label>

<div class="editor-field">
<input id="AjouterClient_Nom" name="AjouterClient.Nom" type="text" value="" />

<div class="editor-label">
<label for="AjouterClient_Adresse">Adresse :</label>

<div class="editor-field">
<input id="AjouterClient_Adresse" name="AjouterClient.Adresse" type="text" value="" />

```
Comme vous pouvez le remarquer, les attributs name et id ont bien été préfixés.

Se pose maintenant le problème de la résolution du modèle par le ModelBinder d’ASP.NET MVC lorsque le formulaire sera posté. En effet, par défaut le ModelBinder va chercher dans les champs postés des propriétés nommées exactement comme les propriétés de l’objet que l’on récupère en paramètre de l’action (ici un Client). Du coup, il est nécessaire d’indiquer au ModelBinder qu’un préfixe a été utilisé. Ceci se fait à l’aide de l’attribut **<em>Bind</em>**, comme le montre le code ci-dessous :

```xml
[HttpPost]
public ActionResult Ajouter([Bind(Prefix="AjouterClient")]Client client)
{
//ajout du client
return View("Index");
}
```
Et voilà, le tour est joué, le ModelBinder retombe sur ses pattes :

![image](/images/aspnet-mvc-utilisation-des-prefixes-pour-la-generation-des-noms-et-id-des-champs/sample-binder-prefix_26948B34.png)

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_060D4B82.png">

Julien

