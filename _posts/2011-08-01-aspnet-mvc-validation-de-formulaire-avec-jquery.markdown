---
layout: post
title: "[ASP.NET MVC] Validation de formulaire avec jQuery"
date: 2011-08-01 7:32:00 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: bf088c4a-8a0e-416b-b313-64338fc7a204
redirect_from:
  - /archives/aspnet-mvc-validation-de-formulaire-avec-jquery
  - /Archives/aspnet-mvc-validation-de-formulaire-avec-jquery
---

ASP.NET MVC 3 supporte de nouvelle fonctionnalités en terme de validation de formulaire côté client, à l’aide de la bibliothèque de jQuery et de son plugin dédié à la validation.

Dans ce post, nous verrons comment valider un formulaire côté client. Il faut garder à l’esprit que la validation côté client n’est là que pour rendre le formulaire plus ergonomique (il s’agit plutôt d’aide à la saisie) : il est impératif de toujours valider un modèle côté serveur !

Dans le fichier de configuration de votre application ASP.NET MVC 3, vous devriez voir les deux clés suivantes dans la section <appSettings /> :

```xml
<add key="ClientValidationEnabled" value="true" />
<add key="UnobtrusiveJavaScriptEnabled" value="true" />
```

La première, **ClientValidationEnabled**, permet de préciser que vous souhaitez que la validation des formulaires côté client (en Javascript donc) soit activée.

La seconde, **UnobtrusiveJavaScriptEnabled**, permet de préciser que vous souhaitez que du javascript `non intrusif` soit utilisé pour la validation. Le monteur ASP.NET MVC 3 rajoutera alors des attributs supplémentaires lors de la génération des champs d’un formulaire par l’intermédiaire d’un HtmlHelper (TextBox, TextBoxFor, CheckBox, CheckBoxFor…).

Ces attributs pourront ensuite être interprétés par un framework de validation JQuery : **jQuery Validate** (et jQuery Validate Unobtrusive).

Lorsque vous ajoutez une nouvelle vue à votre projet via l’interface Visual Studio prévue à cet effet, il est possible de cocher une case `Reference script libraries` afin que les scripts nécessaires à la mise en place de la validation soit ajoutés à la page :

![image](/images/aspnet-mvc-validation-de-formulaire-avec-jquery/cf2d98cd-a685-4617-8f19-2c722b031922.jpg)

Cela aura pour effet d’ajouter les lignes suivantes à la vue :

```xml
<script src="@Url.Content("~/Scripts/jquery.validate.min.js")" type="text/javascript"></script>
<script src="@Url.Content("~/Scripts/jquery.validate.unobtrusive.min.js")" type="text/javascript"></script>
```

Il s’agit des scripts nécessaires pour faire fonctionner l’API jQuery Validate. L’ajout de ces deux références de scripts ainsi que les deux clés du fichier Web.config suffise à permettre la validation côté client :

![image](/images/aspnet-mvc-validation-de-formulaire-avec-jquery/ff3dfcf6-85a6-433b-97c0-ff86281abe8e.jpg)

NB : pour que cela fonctionne, le modèle doit porter des attributs de validation ! (cf. exemple en bas de page)

Le markup HTML généré lors de l’appel du formulaire est le suivant :

```xml
<div class="editor-label">
<label for="Title">Title</label>

<div class="editor-field">
<input class="text-box single-line" data-val="true" data-val-required="Le champ titre est obligatoire" id="Title" name="Title" type="text" value="" />
<span class="field-validation-valid" data-valmsg-for="Title" data-valmsg-replace="true"></span>

<div class="editor-label">
<label for="Description">Description</label>

<div class="editor-field">
<input class="text-box single-line" data-val="true" data-val-required="Le champ description est obligatoire" id="Description" name="Description" type="text" value="" />
<span class="field-validation-valid" data-valmsg-for="Description" data-valmsg-replace="true"></span>
```

Comme il est possible de le constater, un certain nombre d’attribut / class ont été rajoutés sur les input afin de permettre la valdidation `non intrusive` :

- **data-val** : indique que la validation doit avoir lieu sur cet élément

- **data-val-required** : indique que le champ est requis et fournit le message d’erreur à afficher

- **data-val-message-for** : indique que la balise span affiche le message d’erreur du champ description

Du coup, si JavaScript est activé sur la machine de l’utilisateur, les champs doivent être saisis convenablement avant le post du formulaire.

Il faut cependant valider le modèle côté serveur, dans le contrôleur :

```csharp
if (ModelState.IsValid)
{
using (var repo = new BlogRepository())
{
repo.AddBlog(blog);
return RedirectToAction("Index", "Home");
}
}
```

Si vous désactivez la validation non intrusive (via la clé de configuration **UnobtrusiveJavaScriptEnabled** vue précédemment) vous obtiendrez alors le markup suivant à la suite du formulaire :

```xml
<script type="text/javascript">
//<![CDATA[
if (!window.mvcClientValidationMetadata) { window.mvcClientValidationMetadata = []; }
window.mvcClientValidationMetadata.push({"Fields":[{"FieldName":"Title","ReplaceValidationMessageContents":true,"ValidationMessageId":"Title_validationMessage","ValidationRules":[{"ErrorMessage":"Le champ titre est obligatoire","ValidationParameters":{},"ValidationType":"required"}]},{"FieldName":"Description","ReplaceValidationMessageContents":true,"ValidationMessageId":"Description_validationMessage","ValidationRules":[{"ErrorMessage":"Le champ description est obligatoire","ValidationParameters":{},"ValidationType":"required"}]},{"FieldName":"IsPublished","ReplaceValidationMessageContents":true,"ValidationMessageId":"IsPublished_validationMessage","ValidationRules":[{"ErrorMessage":"The IsPublished field is required.","ValidationParameters":{},"ValidationType":"required"}]},{"FieldName":"PublishDate","ReplaceValidationMessageContents":true,"ValidationMessageId":"PublishDate_validationMessage","ValidationRules":[{"ErrorMessage":"The PublishDate field is required.","ValidationParameters":{},"ValidationType":"required"}]}],"FormId":"form0","ReplaceValidationSummary":false,"ValidationSummaryId":"validationSummary"});
//]]>
</script>
```

Projet web d’exemple : [BlogSample.Mvc.zip](http://juliencorioland.blob.core.windows.net/publicfiles/BlogSample.Mvc.zip)

A bientôt ![image](/images/aspnet-mvc-validation-de-formulaire-avec-jquery/928b01a3-2114-47f2-ba0c-b0b8b4a2e36a.jpg)

