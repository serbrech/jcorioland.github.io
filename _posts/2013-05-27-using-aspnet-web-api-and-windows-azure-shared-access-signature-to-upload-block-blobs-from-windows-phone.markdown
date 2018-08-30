---
layout: post
title: "Using ASPNET Web API and Windows Azure Shared Access Signature to upload block blobs from Windows Phone"
date: 2013-05-27 23:42:00 +02:00
categories: Windows Azure, Windows Phone, ASP.NET Web API
author: "Julien Corioland"
identifier: 0fca6b8a-8793-4445-b09d-92096ebae577
redirect_from:
  - /archives/using-aspnet-web-api-and-windows-azure-shared-access-signature-to-upload-block-blobs-from-windows-phone
  - /Archives/using-aspnet-web-api-and-windows-azure-shared-access-signature-to-upload-block-blobs-from-windows-phone
---

One of the best thing when working with the Windows Azure platform is that all APIs are accessible through REST http requests. For one Windows Phone app I’m currently developing I had to upload video file in a blob container. To do that, I choose to use the shared access signatures.

Windows Azure storage accounts are secured with a key that is generated when the account is created. Even if it is possible to regenerate this key at any time, it gives to anyone who owns it admin rights on the storage account. The purpose of using shared access signatures is to allow read, write, delete or list operations on a blob reference or a blob container. Shared access signatures are built from shared access policies that allows to control which kind of permission is given and the expiration time of the signature. For example, you can create an shared access signature that allows any user that owns it to upload blobs in a given container during 20 minutes.

First, I have developed a web API service that is consumed by my Windows Phone application to get a shared access signature to a blob container. To generate the signature, you can use the traditional storage APIs from the Windows Azure SDK (2.0 in this case) :

```csharp
public class SharedAccessSignatureController : ApiController
{
private const string cloudStorageAccountConnectionString = "<your storage connection string>";

public HttpResponseMessage Get()
{
CloudStorageAccount account = CloudStorageAccount.Parse(cloudStorageAccountConnectionString);
CloudBlobClient client = account.CreateCloudBlobClient();
CloudBlobContainer container = client.GetContainerReference("container");
container.CreateIfNotExists();

SharedAccessBlobPolicy policy = new SharedAccessBlobPolicy()
{
Permissions = SharedAccessBlobPermissions.Read|SharedAccessBlobPermissions.Write,
SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5),
SharedAccessExpiryTime = DateTime.UtcNow.AddMinutes(30)
};

string sharedAccessSignature = container.GetSharedAccessSignature(policy);
string containerWithSasUri = string.Format("{0}{1}", container.Uri, sharedAccessSignature);

return Request.CreateResponse(HttpStatusCode.OK, new { ContainerWithSasUrl = containerWithSasUri });
}
}
```
In the previous sample, I get a reference to a blob container using the azure storage sdk. After that I can get a shared access signature for read and write operations. The signature is valid during the next 30 minutes.

Now I can easily get the blob container URI with shared access signature from my Windows Phone application :

```csharp
private async Task<string> GetContainerWithSasUri()
{
var httpClient = new HttpClient();
var requestUrl = "http://mywebapi.com/api/SharedAccessSignature";
var response = await httpClient.GetAsync(requestUrl);
if (response.StatusCode == HttpStatusCode.OK)
{
return await response.Content.ReadAsStringAsync();
}

return string.Empty;
}
```
<em>NB : the HttpClient is now available for Windows Phone 8 applications. You can find it with the Nuget packages manager console, in Visual Studio 2012.</em>

Now that I have the shared access signature in the Windows Phone application I can use it to upload the video file in the container, using the HttpClient and the Windows Azure storage REST APIs.

There are two steps to upload a block blob in a container :

- Upload each block with an id
- Commit the block ids to finalize the upload and consolidate the file in the storage

```csharp
public async Task<string> UploadVideoAsync(string fileName)
{
//get the container uri with SAS
string containerWithSasUri = GetContainerWithSasUri().Replace("\"", "");
//get the query string
string query = new Uri(containerWithSasUri).Query;
//extract the blob container uri, without the shared access signature
string blobContainerUri = containerWithSasUri.Substring(0, containerWithSasUri.Length - query.Length);

//create a list to store block id's
List<string> blocks = new List<string>();

//file block size
int blockSize = 4 * 1024;

//create a buffer
byte[] fileBuffer = new byte[blockSize];

//blobName
string blobName = Guid.NewGuid().ToString();

//create http client
HttpClient httpClient = new HttpClient();

//open the file from iso store
using (var isf = IsolatedStorageFile.GetUserStoreForApplication())
{
if (!isf.FileExists(fileName))
return "";

//open the stream
using (var fileStream = isf.OpenFile(fileName, FileMode.Open, FileAccess.Read))
{
int blockIndex = 0;

//while end of file is not reached
while (fileStream.Read(fileBuffer, 0, blockSize) != 0)
{
//add the block id in the list
string blockId = blockIndex.ToString("d4");
blocks.Add(blockId);

//create the URL to put the file block
string requestUrl = string.Format("{0}/{1}{2}&comp=block&blockid={3}", blobContainerUri, blobName, query,
Convert.ToBase64String(Encoding.UTF8.GetBytes(blockId)));

//PUT the block async
var response = await httpClient.PutAsync(requestUrl, new ByteArrayContent(fileBuffer));

//if the block was not created, return empty string
if (response.StatusCode != HttpStatusCode.Created)
return "";
}
}
}

//consolidate the block list
string xmlBody = @"<?xml version=""1.0"" encoding=""utf-8"" ?><BlockList>{0}</BlockList>";
StringBuilder xmlBodyBuilder = new StringBuilder();
foreach (string blockId in blocks)
{
xmlBodyBuilder.AppendFormat("<Latest>{0}</Latest>", Convert.ToBase64String(Encoding.UTF8.GetBytes(blockId)));
}

xmlBody = string.Format(xmlBody, xmlBodyBuilder.ToString());

//commit the block list
string commitUrl = string.Format("{0}/{1}{2}&comp=blockList", blobContainerUri, blobName, query);
var commitResponse = await httpClient.PutAsync(commitUrl, new ByteArrayContent(Encoding.UTF8.GetBytes(xmlBody)));

if (commitResponse.StatusCode != HttpStatusCode.Created)
return "";

return string.Format("{0}/{1}", blobContainerUri, blobName);
}
```
This code snippet allows to upload a file from the isolated storage to a blob container and get its URI once the upload is done. As you can see, a distinct URL is used to PUT each blob. After that all file blocks have been upload, an XML payload is built to regroup all the block ids. Next, this payload is also put to commit the file upload.

Hope this helps <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_603D72C2.png">

Julien

