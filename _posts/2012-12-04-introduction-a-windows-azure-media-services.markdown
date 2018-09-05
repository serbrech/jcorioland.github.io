---
layout: post
title: "Introduction à Windows Azure Media Services"
date: 2012-12-04 11:07:48 +01:00
categories:
- Microsoft Azure
author: "Julien Corioland"
identifier: 3c118028-ef11-428d-b61e-b7893245030d
redirect_from:
  - /archives/introduction-a-windows-azure-media-services
  - /Archives/introduction-a-windows-azure-media-services
---

### Windows Azure Media Services

Il s’agit d’une nouvelle brique de Windows Azure qui permet de gérer l’hébergement, l’encodage, la mise à disposition de médias, sur Internet. Il est notamment possible de gérer des workflows de publication de vidéo. Par exemple, vous allez pouvoir choisir d’envoyer un fichier vidéo au format HD dans un espace de stockage Windows Azure (blob), et faire en sorte que ce fichier soit automatiquement encodé dans différents formats (basse résolution etc…) et publié sur Internet.  Pour se faire, Microsoft propose un ensemble d’API dans le SDK Windows Azure Media Services.  ### Création d’un service de média

Connectez-vous sur le site de management Windows Azure et cliquez sur la section « Services de média »  ![image](/images/introduction-a-windows-azure-media-services/image_2B34289F.png)Cliquez ensuite sur le lien « Créer un service de média ». Vous devrez alors fournir trois informations :   - Le nom du service que vous souhaitez créer (chiffres et lettre, tout en minuscules)  - La localisation géographique  - Le stockage Windows Azure qui doit être utilisé pour stocker les fichiers

![image](/images/introduction-a-windows-azure-media-services/clip_image002_29835CCB.jpg)

Patientez ensuite jusqu’à ce que le service soit créé.  Une fois ceci fait, il est possible de récupérer la clé de management qui permettra d’utiliser le SDK pour envoyer des fichiers vidéos sur Azure et les traiter, au sein d’une application .NET. Pour cela, rendez-vous sur le dashboard du service de média que vous venez de créer et cliquez sur le lien « Gérer les clés » :  ![image](/images/introduction-a-windows-azure-media-services/clip_image004_562C16A4.jpg)Copiez alors la clé d’accès primaire que vous allez réutiliser par la suite.  ### Utilisation de l’API .NET

Afin de pouvoir utiliser les APIs Windows Azure Media Service, il est nécessaire d’inclure [ce package Nuget](https://nuget.org/packages/windowsazure.mediaservices) dans votre projet.  Le premier objet à instancier est un **CloudMediaContext**. C’est lui qui représente le point d’entrée de l’API Windows Azure Media Services. Il est nécessaire de passer le nom du service ainsi que la clé d’accès primaire au constructeur :```csharp
var context = new CloudMediaContext("nom de votre service Azure Media Services", "votre clé");
```
Via ce context, vous allez pouvoir créer un nouvel asset. Un asset représente une vidéo, dans les différents formats sous lesquels elle sera publié. Pour créer un asset, il faut exécuter le code suivant :

```csharp
string videoFilePath = @"E:\VIDEOS\Test.mp4";
string fileName = Path.GetFileName(videoFilePath);
string assetName = string.Format("{0} - {1}", Path.GetFileNameWithoutExtension(videoFilePath), DateTime.Now);

//création de l'asset sur le CloudMediaContext
IAsset asset = context.Assets.Create(assetName, AssetCreationOptions.StorageEncrypted);
```
L’asset est a présent crée, il est alors possible d’envoyer le fichier vidéo qui est associé :

```csharp
IAssetFile file = asset.AssetFiles.Create(fileName);
file.Upload(videoFilePath);
```
Vous pouvez vous arrêter à ce code si vous ne souhaitez pas ré encoder le fichier derrière pour le publier dans différents formats. D’ailleurs, si vous vous connectez à l’interface de gestion de Windows Azure Media Services, vous devriez voir apparaitre le contenu vidéo qui a été envoyé.

Dans le cas présent, nous allons pousser un peu plus loin puisque nous allons lancer un job d’encodage. Pour se faire, il est nécessaire de récupérer un `media processor`, c’est à dire un outils qui va permettre d’effectuer l’encodage vidéo. Windows Azure Media Services en propose plusieurs :

- Windows Azure Media Encoder
- Windows Azure Media Packager
- Windows Azure Media Encryptor
- Storage Decryption

Pour plus d’info sur les media processors, consultez [cette page](http://msdn.microsoft.com/en-us/library/windowsazure/jj129580.aspx#get_media_processor) de la MSDN.

Ici, c’est le premier de la liste qui va être utilisé. Pour le récupérer, il suffit de faire une simple requête LINQ sur la collection `MediaProcessors` du contexte :

```csharp
var mediaProcessor = context.MediaProcessors.Where(f => f.Name == "Windows Azure Media Encoder").AsEnumerable().FirstOrDefault();
```
Afin de lancer une tâche d’encodage, il est également nécessaire de préciser une configuration à utiliser. Il est possible de récupérer les différentes configuration sur [cette page](http://msdn.microsoft.com/en-us/library/windowsazure/hh973619.aspx), en fonction du media processor utilisé ! Dans le cas présent, c’est le preset `H.264 256k DSL CBR` qui va être utilisé :

```csharp
string encodingConfiguration = "H.264 256k DSL CBR";
```
Il est désormais possible de créer la tâche d’encodage :

```csharp
//création du job
IJob job = context.Jobs.Create(string.Format("job d'encodage de {0}", asset.Name));

//ajout de la tâche d'encodage
ITask task = job.Tasks.AddNew(string.Format("tâche d'encodage de {0}", asset.Name), mediaProcessor, encodingConfiguration, TaskOptions.None);

//ajout de l'asset en input
task.InputAssets.Add(asset);

//création de l'asset d'output
var outputAsset = task.OutputAssets.AddNew(string.Format("{0} (output)", asset.Name), true, AssetCreationOptions.None);

//soumission du job
job.Submit();
```
Pour attendre la fin du job, il faut récupérer une task sur l’objet job à l’aide de la méthode **GetExecutionProgressionTask**. Il est alors possible de tester son état pour savoir s’il a réussi ou échoué :

```csharp
//attente de la fin du job
Task jobProgressionTask = job.GetExecutionProgressTask(CancellationToken.None);
jobProgressionTask.Wait();

//test de l'état du job
if (job.State == JobState.Finished)
{
Console.WriteLine("Le job est terminé");
}
else if (job.State == JobState.Error)
{
Console.WriteLine("Une erreur est survenue pendant l'exécution du job.");
}
```
Il est également possible de s’abonner à l’événement **StateChanged** sur le job afin de récupérer les différentes transitions au fur à mesur de son exécution :

```csharp
static void job_StateChanged(object sender, JobStateChangedEventArgs e)
{
Console.WriteLine("L'état du job vient de passer de {0} à {1}", e.PreviousState, e.CurrentState);
}
```
Et voilà, le tour est joué, vous avez pu envoyer un fichier vidéo directement dans le stockage Azure et lancer différents workflow pour réencoder la vidéo à la volée avant de la publier :

![image](/images/introduction-a-windows-azure-media-services/image_4E347442.png)

Et si vous allez sur l’interface Windows Azure vous pouvez constater que l’output est bien présent et publiable :

![image](/images/introduction-a-windows-azure-media-services/image_3B7F7A8B.png)

Un fois publié, il est possible de lire lire et de récupérer son adresse dans le blob storage Azure :

![image](/images/introduction-a-windows-azure-media-services/image_0BE15BFF.png)

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Clignement d'œil" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_32433F4A.png">

Julien

