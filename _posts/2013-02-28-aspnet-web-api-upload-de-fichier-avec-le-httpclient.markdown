---
layout: post
title: "[ASPNET Web API] Upload de fichier avec le HttpClient"
date: 2013-02-28 11:45:14 +01:00
categories: ASP.NET Web API
author: "Julien Corioland"
identifier: c009f50e-db1a-474d-ac60-2d5b66026861
redirect_from:
  - /archives/aspnet-web-api-upload-de-fichier-avec-le-httpclient
  - /Archives/aspnet-web-api-upload-de-fichier-avec-le-httpclient
---

Il est fréquent qu’un service http est besoin de récupérer un fichier envoyé par l’utilisateur. ASP.NET Web API permet bien entendu de mettre en place ce type de scénario.

### Code côté Serveur

Côté serveur, il s’agit d’un contrôleur ASP.NET Web API tout à fait classique qui va exploiter une multipart provider pour pouvoir télécharger et enregistrer le fichier. Si vous ne connaissez pas les multipart provider, je vous invite à lire [cet article](http://www.juliencorioland.net/archives/creation-dun-multipart-file-stream-provider-personnalise-avec-aspnet-web-api) que j’ai posté il y a quelques temps.

```csharp
public class FileUploadController : ApiController
{
public async Task<HttpResponseMessage> Post()
{
//si la requête n'est pas en multipart/form-data
if (!Request.Content.IsMimeMultipartContent("form-data"))
{
return Request.CreateErrorResponse(HttpStatusCode.UnsupportedMediaType, "non supporté");
}

//création du multipart provider fournit avec Web API et qui permet d'enregistrer les fichiers
//sur le file system (ici dans un répertoire temporaire)
var multipartFileStreamProvider = new MultipartFileStreamProvider(Path.GetTempPath());

//récupération des fichiers
var files = await Request.Content.ReadAsMultipartAsync(multipartFileStreamProvider).ContinueWith(task =>
{
if (task.IsFaulted)
throw task.Exception;

return task.Result.FileData.ToList();
});

//travail sur les fichiers
foreach (var file in files)
{
//chemin du fichier en local :
string path = file.LocalFileName;

//nom du fichier envoyé :
string fileName = file.Headers.ContentDisposition.FileName;
}

return Request.CreateResponse(HttpStatusCode.OK);
}
}
```
###

### Code côté client

Côté client, pour envoyer un fichier, nous allons utiliser une application console et la classe **HttpClient**. Il est nécessaire d’ajouter une référence vers **System.Net.Http.dll** :

![image](/images/aspnet-web-api-upload-de-fichier-avec-le-httpclient/image_1AD2C67F.png)

Ensuite, il faut tout d’abord créer un **FileStream** sur le fichier à Upload, puis un **StreamContent**. Ce StreamContent est alors rajouté à un **MultipartFormDataContent**, qui représente le contenu qui sera envoyé par le client http, en mode multipart/form-data.

Il suffit alors de créer un HttpClient et de poster le contenu à l’adresse adéquate.

```csharp
private static async Task UploadAsync()
{
int bufferSize = 1024;

//création du file stream
using (var fs = new FileStream("Fichier à envoyer.txt", FileMode.Open, FileAccess.Read, FileShare.Read, bufferSize, true))
{
//création d'un stream StreamContent qui représente le fichier
StreamContent streamContent = new StreamContent(fs, bufferSize);

//création d'un MultipartFormDataContent et ajout du fichier
MultipartFormDataContent formDataContent = new MultipartFormDataContent();
formDataContent.Add(streamContent, "fichier", "Fichier à envoyer.txt");

//création du http client
HttpClient client = new HttpClient();

var uri = new Uri("http://localhost:17831/api/fileupload");

//envoie de la requête HTTP POST
HttpResponseMessage response = await client.PostAsync(uri, formDataContent);

//récupération du status
if (response.StatusCode == System.Net.HttpStatusCode.OK)
{
Console.WriteLine("fichier envoyé");
}
else
{
string description = await response.Content.ReadAsStringAsync();
Console.WriteLine("une erreur s'est produite : " + description);
}

Console.ReadLine();
}
}
```
Et voilà, le tour est joué ! Le code source de démonstration est dispo ici : <a title="http://juliencorioland.blob.core.windows.net/publicfiles/UploadHttpSample.zip" href="http://juliencorioland.blob.core.windows.net/publicfiles/UploadHttpSample.zip">http://juliencorioland.blob.core.windows.net/publicfiles/UploadHttpSample.zip</a>

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_17DD61CC.png">

Julien

