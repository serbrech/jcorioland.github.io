---
layout: post
title: "[En][ASP.NET MVC] Remote attribute and asynchronous validation"
date: 2011-09-05 16:26:00 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: 826210ee-4192-4a1e-84ae-1f13a5979dc7
redirect_from:
  - /archives/enaspnet-mvc-remote-attribute-and-asynchronous-validation
  - /Archives/enaspnet-mvc-remote-attribute-and-asynchronous-validation
---

As you probably know ASP.NET MVC supports .NET Framework 4 Data Annotations to validate user’s inputs (Required, StringLength, Range, RegularExpression…)

Another attribute exists and allows asynchronous client side validation. It’s the **Remote** attribute. For example, it can be used to validate e-mail and username in a registration form and alert the user before the form is posted.

Here is a sample model for a registration form :

```csharp
public class UserModel
{
[Required]
[Remote("CheckUsername", "RemoteValidation", ErrorMessage = "This username is already used.")]
public string Username { get; set; }

[Required]
[Remote("CheckEmail", "RemoteValidation", ErrorMessage = "This e-mail is already used.")]
public string Email { get; set; }

[StringLength(80)]
public string FirstName { get; set; }

[StringLength(80)]
public string LastName { get; set; }

public DateTime BirthDate { get; set; }
}
```

As you can see in the previous code snippet, the **Remote** attribute takes two parameters :

- the MVC action that should validate the input

- the controller that defines the action

*The controller :*

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

These actions are very simple : they take the input to validate as a string parameter and return a Json-serialized Boolean that indicates if the input is valid or not. For this article I’ve used two arrays to represent users and e-mails databases but it’s possible to execute any code here.

*The view :*

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

To activate the client side validation the following jQuery plugins should be loaded :

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

The validation actions are automatically called (asynchronously of course) when the user is completing the register form :

![image](/images/enaspnet-mvc-remote-attribute-and-asynchronous-validation/4f231217-6729-4233-81ea-fb4da3539a26.jpg)

Hope this helps ![image](/images/enaspnet-mvc-remote-attribute-and-asynchronous-validation/aa6b240f-4829-4081-b8ea-7a62900604c6.jpg)

