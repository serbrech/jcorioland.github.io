---
layout: post
title: "Using ASPNET Web API to stream Windows Azure blobs"
date: 2013-09-06 14:35:00 +02:00
categories:
- Microsoft Azure
- ASP.NET Web API
author: "Julien Corioland"
identifier: d552d7e2-0a86-40af-a161-8a909fe9fa14
redirect_from:
  - /archives/using-aspnet-web-api-to-stream-windows-azure-blobs
  - /Archives/using-aspnet-web-api-to-stream-windows-azure-blobs
---

Few months ago I blogged about [how to use ASP.NET Web API to generate a shared access signature to upload a file from Windows Phone to the Windows Azure blob storage](http://www.juliencorioland.net/archives/using-aspnet-web-api-and-windows-azure-shared-access-signature-to-upload-block-blobs-from-windows-phone). It is also possible to use a Web API to return a shared access key that allows a client to download a blob, directly from the storage.

Today I will discuss on another scenario : the ability to stream a blob through a Web API, without returning a shared access key to the user. It can be really useful when you have to serve a file to a client without giving the shared access key that may be use until it expires (for one shot download, for example).

First, you have to get a reference to the blob to stream against the blob storage :

```csharp
public async Task<HttpResponseMessage> Get(string blobId)
{
//security checks (for example)
if (!User.Identity.IsAuthenticated || !User.IsInRole("Premium"))
return Request.CreateErrorResponse(HttpStatusCode.Unauthorized, "unauthorized");

//call your service to get the blob information
var blobService = new BlobInformationService();
var blobInfo = blobService.GetBlobInfoById(blobId);

if (blobInfo == null)
return Request.CreateErrorResponse(HttpStatusCode.NotFound, "not found");

//create the storage credentials
var storageCredentials = new StorageCredentials(blobInfo.StorageAccountName, blobInfo.StorageAccountKey);

//create the cloud storage account
var cloudStorageAccount = new CloudStorageAccount(storageCredentials, true);

//create the cloud blob client
var cloudBlobClient = cloudStorageAccount.CreateCloudBlobClient();

//get the cloud blob reference
var cloudBlob = await cloudBlobClient.GetBlobReferenceFromServerAsync(new Uri(blobInfo.BlobUrl));
var cloudBlobExists = await cloudBlob.ExistsAsync();

if (!cloudBlobExists)
return Request.CreateErrorResponse(HttpStatusCode.NotFound, "not found");

//next step
}
```
Now that you have a valid blob reference, you can stream it ! To do that, you have to create your own instance of <em>HttpResponseMessage</em> and not use the <em>CreateResponse</em> method of the Request that uses the content negotiation to serialize the output. This http response message uses an instance of <em>StreamContent</em> as content.

The StreamContent accept a stream as constructor parameter. To get a valid stream on the cloud blob reference, you have to call the OpenReadAsync method on it.

To allow the browsers to display all the download’s information to the user (file name, file size, download progress…) you have to set the following headers :

- ContentType : the file type

- ContentLength : the length of the file

- ContentDisposition : the file is an attachment, the file name, the size…

```csharp
HttpResponseMessage message = new HttpResponseMessage(HttpStatusCode.OK);
Stream blobStream = await cloudBlob.OpenReadAsync();

message.Content = new StreamContent(blobStream);
message.Content.Headers.ContentLength = cloudBlob.Properties.Length;
message.Content.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue(cloudBlob.Properties.ContentType);
message.Content.Headers.ContentDisposition = new System.Net.Http.Headers.ContentDispositionHeaderValue("attachment")
{
FileName = blobInfo.FileName,
Size = cloudBlob.Properties.Length
};

return message;
```
Et voilà ! Now you can stream blobs via a Web API instead of sharing access key with any client.

Hope this helps.

Julien <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_008EEB08.png">

