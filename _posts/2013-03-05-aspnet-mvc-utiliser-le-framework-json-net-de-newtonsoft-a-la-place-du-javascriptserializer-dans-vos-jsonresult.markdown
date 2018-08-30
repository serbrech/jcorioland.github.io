---
layout: post
title: "[ASPNET MVC] Utiliser le framework Json .NET de Newtonsoft à la place du JavaScriptSerializer dans vos JsonResult"
date: 2013-03-05 13:51:05 +01:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: d32fe820-3e98-4629-970b-e68ab7aa0340
redirect_from:
  - /archives/aspnet-mvc-utiliser-le-framework-json-net-de-newtonsoft-a-la-place-du-javascriptserializer-dans-vos-jsonresult
  - /Archives/aspnet-mvc-utiliser-le-framework-json-net-de-newtonsoft-a-la-place-du-javascriptserializer-dans-vos-jsonresult
---

ASP.NET MVC fait appel au **JavaScriptSerializer** lorsque l’on retourne un JsonResult, contrairement à ASP.NET Web API qui lui utilise nativement le Framework Json.NET de Newtonsoft. Cela peut entraîner des comportements assez étranges dans le cas d’intéropérabilité entre une application MVC et un service Web API.

Concrètement, le framework Json.NET est beaucoup plus riche que le JavaScriptSerializer pour la manipulation d’objets JSON, du coup, il est clairement recommandé de l’utiliser, même en ASP.NET MVC. Pour le récupérer, il suffit de télécharger le package NuGet Json.NET :

![image](/images/aspnet-mvc-utiliser-le-framework-json-net-de-newtonsoft-a-la-place-du-javascriptserializer-dans-vos-jsonresult/image_7285A368.png)

La première étape consiste à créer un **ActionResult** personnalisé qui utilise Json.NET dans la méthode **ExecuteResult** :

```csharp
public class JsonNetResult : JsonResult
{
public override void ExecuteResult(ControllerContext context)
{
if (context == null)
throw new ArgumentNullException("context");

var response = context.HttpContext.Response;

response.ContentType = !String.IsNullOrEmpty(ContentType) ? ContentType : "application/json";

if (ContentEncoding != null)
response.ContentEncoding = ContentEncoding;

if (Data == null)
return;

var serializedObject = JsonConvert.SerializeObject(Data, Formatting.Indented);
response.Write(serializedObject);
}
}
```
Ensuite, pour pouvoir faire en sorte que ce soit ce **JsonNetResult** là qui soit automatiquement renvoyé par la méthode **Json** de vos contrôleurs, il suffit de passer par un contrôleur intermédiaire dans lequel vous pouvez redéfinir les différentes surcharges de la méthode Json, et notamment :

```csharp
protected override JsonResult Json(object data, string contentType, Encoding contentEncoding, JsonRequestBehavior behavior)
{
var result = new JsonNetResult()
{
Data = data,
ContentType = contentType,
ContentEncoding = contentEncoding,
JsonRequestBehavior = behavior
};

return result;
}
```
Désormais, tous les contrôleurs qui dérivent de votre contrôleur de base utiliseront automatiquement Json.NET pour sérialiser des objets en Json. Au passage, c’est toujours une bonne pratique de créer votre propre contrôleur au démarrage d’un projet (ou dans un Framework) pour pouvoir à tout moment influer sur le comportement de tous vos contrôleurs sans impacter votre code !

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_5E8C10D2.png">

Julien

