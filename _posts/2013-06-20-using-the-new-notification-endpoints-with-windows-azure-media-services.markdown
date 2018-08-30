---
layout: post
title: "Using the new notification endpoints with Windows Azure Media Services"
date: 2013-06-20 16:42:06 +02:00
categories: Windows Azure, Media Services
author: "Julien Corioland"
identifier: 1ef99d02-1733-4b7a-8c05-ab61f80c0a5b
redirect_from:
  - /archives/using-the-new-notification-endpoints-with-windows-azure-media-services
  - /Archives/using-the-new-notification-endpoints-with-windows-azure-media-services
---

Few days ago was released a new version of the Windows Azure Media Services .NET SDK, the version 2.2.0.1 that can be downloaded via the <em>NuGet Packages Manager Console</em> (see [http://nuget.org/packages/windowsazure.mediaservices](http://nuget.org/packages/windowsazure.mediaservices)). One of the top feature in this release is the ability to create notification endpoints to be notified when a job state change occurs !

In the previous versions of the SDK, the only way to track job state changes was to keep a list of all job’ ids (in a sql server table, for example) and pull each job through the **CloudMediaContext** to get its state. This solution works fine but is not really elegant or scalable when dealing with multiple azure worker, for example. Now, it’s possible to create a notification endpoint linked to a Windows Azure Storage Queue and pull this queue instead of the media context. It’s the Windows Azure Media Service backend that manage the messages that are pushed in this queue.

To use the notification endpoints, you need to create a CloudQueue that will be used for notification endpoint(s) :

```csharp
//create the cloud storage account from name and private key
CloudStorageAccount cloudStorageAccount = CloudStorageAccount.Parse(StorageConnectionString);

//create the cloud queue client from the storage connection string
CloudQueueClient cloudQueueClient = cloudStorageAccount.CreateCloudQueueClient();

//get a cloud queue reference
CloudQueue notificationsQueue = cloudQueueClient.GetQueueReference(NotificationQueuePath);

//create the queue if it does not exist
if (!notificationsQueue.Exists())
{
notificationsQueue.Create();
}
```
Now, you can use the **CloudMediaContext** to create an instance of **INotificationEndpoint**, the new interface that enables notifications management :

```csharp
//create the Cloud Media Context
CloudMediaContext mediaContext = new CloudMediaContext("<mediaservicename>", "<mediaservicekey>");

//create a notification endpoint
INotificationEndPoint notificationEndpoint =
mediaContext.NotificationEndPoints
.Create("notificationendpoint", NotificationEndPointType.AzureQueue, NotificationQueuePath);
```
As you can see, the Create method take a **NotificationEndPointType** as second parameter. For now, the only possible value (instead of None) is `AzureQueue`. <em>It may seems that future releases of the SDK will support Azure Service Bus queue or topics ? (I really don’t know, but it will be very cool ! <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Sourire" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_7E0364F0.png">)</em>

Once the notification endpoint has been created, you can create the job and all the tasks it should execute as usual :

```csharp
//create an asset
IAsset asset = mediaContext.Assets.Create("Wildlife HD", AssetCreationOptions.None);

//create an asset file from the sample video file
string fileName = System.IO.Path.GetFileName(VideoTestPath);
IAssetFile assetFile = asset.AssetFiles.Create(fileName);

//upload the asset
assetFile.Upload(VideoTestPath);

//create a media service job
IJob mediaServiceJob = mediaContext.Jobs.Create("Wildlife in HD Adaptive Streaming");

//get the latest Windows Azure Media Encoder
IMediaProcessor mediaProcessor = GetLatestsAzureMediaEncoder(mediaContext);

//create a multi bitrate encoding task
ITask multibitrateTask = mediaServiceJob.Tasks.AddNew("Multibitrate MP4 encoding", mediaProcessor, "H264 Adaptive Bitrate MP4 Set 720p", TaskOptions.None);

//add the asset as input of the task
multibitrateTask.InputAssets.Add(asset);

//create the output asset
multibitrateTask.OutputAssets.AddNew("Wildlife HD Output", AssetCreationOptions.None);
```
Before submitting the job, you have to declare the association with the notification endpoint. To do that, a new property has been added on the IJob interface : JobNotificationsSubscriptions. Call AddNew on this property to link the notification endpoint to the job :

```csharp
mediaServiceJob.JobNotificationSubscriptions.AddNew(NotificationJobState.FinalStatesOnly, notificationEndpoint);
```
Now, you can submit the job :

```csharp
mediaServiceJob.Submit();
```
Et voilà ! To be notified of each job state change you just need to pull the **CloudQueue** :

```csharp
while (continuePull)
{
Thread.Sleep(5000);//sleep for 5 sec

//get a cloud queue message
CloudQueueMessage message = notificationsQueue.GetMessage();
if (message == null)//if null, continue
continue;
}
```
The message content is serialized as json so you can use the DataContractJsonSerializer to get an object that as this prototype :

```csharp
public class EncodingJobMessage
{
public String MessageVersion { get; set; }

public String EventType { get; set; }

public String ETag { get; set; }

public String TimeStamp { get; set; }

public IDictionary<string, object> Properties { get; set; }
}
```
The **EventType** property can take two values :

- <em>JobStateChange</em> : indicates that the notification message is related to a job state change

- <em>NotificationEndpointRegistration</em> : indicates that the notification endpoint has been registered

The **Properties** property is a dictionary that contains different information about the notification event :

- <em>JobId</em> : the id of the job

- <em>NewState</em> : the new state

- <em>OldState</em> : the old state

- <em>NotificationEndpointId</em> : the id of the notification endpoint

- <em>State</em> : the state of the notification endpoint (Registered or Unregistered)

Here is a sample of message deserialization using the DataContractJsonSerializer :

```csharp
//get the bytes
using (MemoryStream ms = new MemoryStream(message.AsBytes))
{
//deserialize the message
DataContractJsonSerializerSettings jsonSerializerSettings = new DataContractJsonSerializerSettings();
jsonSerializerSettings.UseSimpleDictionaryFormat = true;

DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof(EncodingJobMessage), jsonSerializerSettings);
EncodingJobMessage jobMessage = (EncodingJobMessage)serializer.ReadObject(ms);

//if the event type is a state change
if (jobMessage.EventType == "JobStateChange")
{
//try get old and new state
if (jobMessage.Properties.Any(p => p.Key == "OldState") && jobMessage.Properties.Any(p => p.Key == "NewState"))
{
string oldJobState = jobMessage.Properties.First(p => p.Key == "OldState").Value.ToString();
string newJobState = jobMessage.Properties.First(p => p.Key == "NewState").Value.ToString();

Console.WriteLine("job state has changed from {0} to {1}", oldJobState, newJobState);
}
}
}
```
This new feature is really cool and allows more flexible and scalable architecture when dealing with Windows Azure Media Services jobs !

Hope this helps <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Sourire" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_7E0364F0.png">

Julien

