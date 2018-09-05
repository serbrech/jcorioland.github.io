---
layout: post
title: "Création d’un multipart file stream provider personnalisé avec ASPNET Web API"
date: 2012-12-10 17:05:59 +01:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: 52c2d6ae-c1c8-4aeb-9dfe-04bf43bc8f17
redirect_from:
  - /archives/creation-dun-multipart-file-stream-provider-personnalise-avec-aspnet-web-api
  - /Archives/creation-dun-multipart-file-stream-provider-personnalise-avec-aspnet-web-api
---

La gestion de l’upload de fichier dans un service ASP.NET Web API se fait à l’aide d’un [MultipartFileStreamProvider](http://msdn.microsoft.com/en-us/library/system.net.http.multipartfilestreamprovider(v=vs.108).aspx). Il est possible de créer ses propres providers pour enregistrer un fichier d’une manière particulière, par exemple dans un blob azure ou une base de données SQL Server. C’est le deuxième exemple que j’ai choisi pour illustrer ce post.

### Utilisation basique du MultipartFileStreamProvider

Pour cet exemple, on se place dans un ApiController tout ce qu’il y a de plus classique, dans une méthode appelée en POST :

```csharp
[HttpPost]
public async Task<HttpResponseMessage> Post()
{
if (!Request.Content.IsMimeMultipartContent("form-data"))
return Request.CreateErrorResponse(HttpStatusCode.UnsupportedMediaType, "contenu non supporté");

//le multipart provider va télécharger les fichiers dans le répertoire temporaire
var multipartProvider = new MultipartFileStreamProvider(Path.GetTempPath());
var files = await Request.Content.ReadAsMultipartAsync(multipartProvider)
.ContinueWith(t =>
{
if (t.IsFaulted)
throw t.Exception;

//retourne la liste des fichiers qui ont été téléchargés sur le systeme de fichier local
//dans le répertoire temporaire
return t.Result.FileData.ToList();
});

foreach (var file in files)
{
//chemin du fichier en local :
string path = file.LocalFileName;

//nom du fichier envoyé :
string fileName = file.Headers.ContentDisposition.FileName;

//contentType du fichier
string contentType = file.Headers.ContentType.MediaType;
}

return Request.CreateResponse(HttpStatusCode.OK);
}
```
Il est d’abord nécessaire de vérifier que le POST a bien été fait avec un mime content en form-data. Si ce n’est pas le cas, on retourne en indiquant que le format du média n’est pas supporté.

Dans un second temps, il suffit d’instancier un MultipartFileStreamProvider en lui passant un répertoire de base pour télécharger les fichiers (ici un dossier temporaire) et d’appeler la méthode ReadAsMultipartAsync sur l’objet Request. Il est alors possible de récupérer les différents fichiers, leur chemin d’accès local, leur content type, leur nom… Il est alors ensuite possible d’appliquer n’importe quel traitement sur le fichier pour l’exploiter (envoie dans un blob azure, stockage dans une base de données…). Ce qui va devenir vraiment intéressant est de factoriser ces différents traitements dans des multipart providers `dédiés` à ces tâches spécifiques !

### Personnalisation d’un MultipartFileStreamProvider

Il est possible de dériver la classe **MultipartFileStreamProvider** afin de surcharger la méthode **ExecutePostProcessingAsync**, en charge de traiter les différents fichier qui sont envoyés sur le serveur. Dans le cas présent, cette méthode va être surchargée pour envoyer automatiquement les fichiers dans une base de données SQL Serveur.

Pour commencer, on créé un constructeur qui permet de passer la chaîne de connexion vers la base de données ainsi que le fournisseur à utiliser :

```csharp
public class MultipartSqlFileStreamProvider : MultipartFileStreamProvider
{
private const string SQL = "INSERT INTO Files (FileId, ContentType, FileName, FileContent) VALUES (@FileId, @ContentType, @FileName, @FileContent)";

private readonly string _connectionString;
private readonly DbProviderFactory _dbProviderFactory;

public List<Guid> FileIDs{ get; set; }

public MultipartSqlFileStreamProvider(string connectionString, string providerName)
: base(Path.GetTempPath())
{
_connectionString = connectionString;
FileIDs = new List<Guid>();
_dbProviderFactory = DbProviderFactories.GetFactory(providerName);
}
}
```
Ensuite, on surcharge la méthode et on parcourt la liste de fichier afin de les enregistrer dans la base SQL :

```csharp
public async override Task ExecutePostProcessingAsync()
{
using (var sqlConnection = _dbProviderFactory.CreateConnection())
{
sqlConnection.ConnectionString = _connectionString;
sqlConnection.Open();

foreach (var file in FileData)
{
using (var dbCommand = sqlConnection.CreateCommand())
{
dbCommand.CommandText = SQL;

Guid fileId = Guid.NewGuid();

var fileIdParameter = dbCommand.CreateParameter();
fileIdParameter.ParameterName = "FileId";
fileIdParameter.Value = fileId;
fileIdParameter.DbType = System.Data.DbType.Guid;
dbCommand.Parameters.Add(fileIdParameter);

var contentTypeParameter = dbCommand.CreateParameter();
contentTypeParameter.ParameterName = "ContentType";
contentTypeParameter.Value = file.Headers.ContentType.MediaType;
contentTypeParameter.DbType = System.Data.DbType.StringFixedLength;
dbCommand.Parameters.Add(contentTypeParameter);

var fileNameParameter = dbCommand.CreateParameter();
fileNameParameter.ParameterName = "FileName";
fileNameParameter.Value = file.Headers.ContentDisposition.FileName.Replace("\"","");
fileNameParameter.DbType = System.Data.DbType.StringFixedLength;
dbCommand.Parameters.Add(fileNameParameter);

var fileContentParameter = dbCommand.CreateParameter();
fileContentParameter.ParameterName = "FileContent";
fileContentParameter.Value = File.ReadAllBytes(file.LocalFileName);
fileContentParameter.DbType = System.Data.DbType.Binary;
dbCommand.Parameters.Add(fileContentParameter);

await dbCommand.ExecuteNonQueryAsync();

FileIDs.Add(fileId);
}
}

sqlConnection.Close();
}
}
```
Pour utiliser le multipart provider personnalisé, il suffit de l’instancier et de le passer à la méthode ReadAsMultipartAsync, comme vu en introduction de cet article.

```csharp
var connectionString = WebConfigurationManager.ConnectionStrings["DataConnectionString"];

var multipartSqlProvider = new MultipartSqlFileStreamProvider(connectionString.ConnectionString, connectionString.ProviderName);
var fileIDs = await Request.Content.ReadAsMultipartAsync(multipartSqlProvider)
.ContinueWith(t =>
{
if (t.IsFaulted)
{
throw t.Exception;
}

var provider = t.Result;
return provider.OutputFileIDs.First().ToString();
}
);
```
Dès lors, les fichiers sont automatiquement envoyés dans la base de données et vous récupérez en output la liste des IDs des fichiers !

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Clignement d'œil" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_634F5206.png">

Julien

