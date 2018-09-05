---
layout: post
title: "[ASP.NET MVC] Attribut Remote et validation asynchrone"
date: 2011-09-05 11:35:00 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: 11b2cdbc-c32d-4e9b-b88c-db80cd9175e7
redirect_from:
  - /archives/aspnet-mvc-attribut-remote-et-validation-asynchrone
  - /Archives/aspnet-mvc-attribut-remote-et-validation-asynchrone
---

Comme vous le savez peut-être déjà, ASP.NET MVC 3 supporte les DataAnnotations du .NET Framework 4 pour tout ce qui est validation des entrées utilisateurs : Required, StringLength, Range, RegularExpression…

Il existe un attribut un peu moins connu : **Remote**. Celui-ci permet de lancer une validation asynchrone côté client. Par exemple, sur un formulaire d’inscription il est fréquent de vouloir valider de manière asynchrone qu’un nom d’utilisateur / e-mail n’existe pas déjà en base utilisateur.

Voilà un exemple de modèle pour parvenir à cela :

```csharp
public class UserModel
{
[Required]
[Remote("CheckUsername", "RemoteValidation", ErrorMessage = "Ce nom d'utilisateur est déjà utilisé")]
public string Username { get; set; }

[Required]
[Remote("CheckEmail", "RemoteValidation", ErrorMessage = "Cet e-mail est déjà utilisé")]
public string Email { get; set; }

[StringLength(80)]
public string FirstName { get; set; }

[StringLength(80)]
public string LastName { get; set; }

public DateTime BirthDate { get; set; }
}
```

Comme il est possible de le constater dans l’extrait de code ci-dessus, l’attribut remote prend deux paramètres :

- L’action MVC en charge de la validation

- Le contrôlleur dans lequel l’action est défini (ici, RemoteValidationController)

Voilà le code de ce contrôlleur :

```csharp
public class RemoteValidationController : Controller
{
private readonly string[] _existingUsernames = {"beedoo", "julien", "jcorioland"};
private readonly string[] _existingEmails = { "contact@juliencorioland.net" };

public JsonResult CheckUsername(string username) {
bool userIsAvailable = !_existingUsers.Contains(username);
return Json(userIsAvailable, JsonRequestBehavior.AllowGet);
}

public JsonResult CheckEmail(string email)
{
bool emailIsAvailable = !_existingEmails.Contains(email);
return Json(emailIsAvailable, JsonRequestBehavior.AllowGet);
}
}
```

Les actions sont ultra simples : elles prennent en paramètre le terme à valider et retourne un booléean, sérialisé en Json.

NB : ici la source d’emails/usernames existants a été simplifiée, mais un appel à une base de données est tout à fait possible.

La vue pour tester :

```xml
@using (Html.BeginForm())
{
<div class="editor-label">@Html.LabelFor(m => m.Username)
<div class="editor-field">
@Html.TextBoxFor(m => m.Username)
@Html.ValidationMessageFor(m => m.Username)

<div class="editor-label">@Html.LabelFor(m => m.Email)
<div class="editor-field">
@Html.TextBoxFor(m => m.Email)
@Html.ValidationMessageFor(m => m.Email)

<div class="editor-label">@Html.LabelFor(m => m.FirstName)
<div class="editor-field">
@Html.TextBoxFor(m => m.FirstName)
@Html.ValidationMessageFor(m => m.FirstName)

<div class="editor-label">@Html.LabelFor(m => m.LastName)
<div class="editor-field">
@Html.TextBoxFor(m => m.LastName)
@Html.ValidationMessageFor(m => m.LastName)

<div class="editor-label">@Html.LabelFor(m => m.BirthDate)
<div class="editor-field">
@Html.TextBoxFor(m => m.BirthDate)
@Html.ValidationMessageFor(m => m.BirthDate)

<input type="submit" value="S'enregistrer" />
}
```

Enfin, assurez vous de charger les plugins jQuery UI et jQuery validate pour que cela fonctionne :

```xml
<head>
<title>@ViewBag.Title</title>
<link href="@Url.Content("~/Content/Site.css")" rel="stylesheet" type="text/css" />
<script src="@Url.Content("~/Scripts/jquery-1.5.1.min.js")" type="text/javascript"></script>
<script src="@Url.Content("~/Scripts/jquery-ui-1.8.11.min.js")" type="text/javascript"></script>
<script src="@Url.Content("~/Scripts/jquery.unobtrusive-ajax.min.js")" type="text/javascript"></script>
<script src="@Url.Content("~/Scripts/jquery.validate.min.js")" type="text/javascript"></script>
<script src="@Url.Content("~/Scripts/jquery.validate.unobtrusive.min.js")" type="text/javascript"></script>
</head>
```

Dès lors, les actions de validation sont appelées en asynchrone lors de la saisie du formulaire, permettant ainsi une aide à la saisie avancée pour l’utilisateur !

![image](/images/aspnet-mvc-attribut-remote-et-validation-asynchrone/f5563f95-e071-431a-894d-31e4099d6554.jpg)

NB : et comme toujours, la validation côté client ne doit absolument pas remplacer la validation côté serveur !

A bientôt ![image](/images/aspnet-mvc-attribut-remote-et-validation-asynchrone/77f00b6e-819d-41f5-b67e-4992baf52650.jpg)

