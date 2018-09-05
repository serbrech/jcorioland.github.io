---
layout: post
title: "Develop a blog using ASPNET vNext, Azure Website, DocumentDB and Search services - Part 2"
date: 2014-10-25 11:17:05 +02:00
categories:
- Microsoft Azure
- ASP.NET MVC
author: "Julien Corioland"
identifier: 2a68383f-dbbc-42c3-a7d6-540d958f7e87
redirect_from:
  - /archives/develop-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-2
  - /Archives/develop-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-2
---

## Introduction

This post is part of a series of articles about developing a blog with ASP.NET vNext, Azure Website, DocumentDB and search services. If you would like to read another article in the series:  - [Develop a blog using ASPNET vNext, Azure Website, DocumentDB and Search services - Part 1](http://www.juliencorioland.net/archives/developing-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-1)- Develop a blog using ASPNET vNext, Azure Website, DocumentDB and Search services - Part 2

In the first article I explained what ASP.NET vNext is and how it is possible to create the Azure resources we’ll need via the preview portal. Today, I’m going to explain what Azure DocumentDB is and start the implementation of the data layer of the blog, using this service. NOTE: the alpha-4 version of ASP.NET vNext has been release with Visual Studio CTP 4 few days ago, so you can switch to this new version! <em>Thanks to [Adrien](https://twitter.com/asiffermann) that reviews this blog post before I publish it !</em>## Creation of the Visual Studio Solution

Before getting into Azure DocumentDB, we are going to create the blog solution in Visual Studio 14 CTP. In the web section, be sure to choose the new ASP.NET vNext project templates:

![image](/images/develop-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-2/image_362CADD1.png)

I chose to create an empty ASP.NET vNext application. Now, add a new ASP.NET vNext class library to the solution. It’s in this library that we will create the domain, queries and commands that work with Document DB. As you can see, both projects have a project.json file with some dependencies configuration. The project.json file defines two kinds of dependencies: external libraries such as MVC, Entity Framework, Document DB and frameworks dependencies.  ![image](/images/develop-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-2/image_65EA5650.png)

aspnet50 represents the full .NET framework and aspnetcore50 represents the small cloud optimized framework that comes with ASP.NET vNext. Referencing these two frameworks ensure that your application will build for both of them but also that all the libraries that you work with are supported by the cloud optimized Framework. In our case, it’s not possible because some of the Azure SDK librairies are not supported by this version, so you should remove the aspnetcore50 line from all project.json files. Now you can add a very simple C# class in the library project, to represent a blog post: ```csharp
namespace MetroBlog.Domain.Entities
{
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;

/// <summary>
/// Represents a blog post
/// </summary>
public class Post
{
/// <summary>
/// Creates a new instance of the class <see cref="MetroBlog.Domain.Entities.Post"/> class;
/// </summary>
public Post()
{
this.Tags = new Collection<string>();
}

/// <summary>
/// Gets or sets the post identifier
/// </summary>
[JsonProperty("id")]
public Guid Id { get; set; }

/// <summary>
/// Gets or sets the publication date
/// </summary>
[JsonProperty("publishDate")]
public DateTime PublishDate { get; set; }

/// <summary>
/// Gets or sets the title
/// </summary>
[JsonProperty("title")]
public string Title { get; set; }

/// <summary>
/// Gets or sets the permalink
/// </summary>
[JsonProperty("permalink")]
public string Permalink { get; set; }

/// <summary>
/// Gets or sets the summary
/// </summary>
[JsonProperty("summary")]
public string Summary { get; set; }

/// <summary>
/// Gets or sets the content of the post
/// </summary>
[JsonProperty("content")]
public string Content { get; set; }

/// <summary>
/// Gets or sets the collection of tags
/// </summary>
[JsonProperty("tags")]
public Collection<string> Tags { get; set; }

/// <summary>
/// Gets or sets a boolean that indicates if the post is published
/// </summary>
[JsonProperty("isPublished")]
public bool IsPublished { get; set; }

/// <summary>
/// Gets or sets the username of the author
/// </summary>
[JsonProperty("username")]
public string Username { get; set; }
}
}
```
## Using Azure DocumentDB to store the blog posts

### Configure the project to use Azure Document DB

###

DocumentDB is a NoSQL database that supports the storage of documents described as JSON. One of the big advantage of this kind of database is that the schema has not to be fixed: it can evolve with the application and data that are pushed inside the database are continuously indexed to provide the best performances. Because the document are stored as JSON format, it is really easy to push and extract data from DocumentDB. Microsoft provides a language that looks like SQL to query the database, so if you are not used to work with document database, don’t be afraid: it’s simple!
To reference the nuget package in your project, open the project.json file and start typing the name of the package (Microsoft.Azure.Documents.Client) in the dependencies section and click on the package in the IntelliSense:

![image](/images/develop-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-2/image_213DBC0F.png)

As soon as you save the file, the package will be downloaded and the project references updated:
![image](/images/develop-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-2/image_19B24CA2.png)

To start using the Azure Document DB with this SDK, you’ll need three information:

- The endpoint of the instance you have created
- The authorization key to access the database
- The id of the database to work with

The two first information are available on the preview Azure portal :

![image](/images/develop-a-blog-using-aspnet-vnext-azure-website-documentdb-and-search-services-part-2/image_02F30519.png)

### Create a base class for commands and queries

I chose to work with queries and commands to read and create post from / to the document database. I also made a base class for all my commands and queries.
To work with Document DB in your application you need to create an instance of the DocumentClient class that comes with the NuGet package and takes the endpoint URL and the authorization key as parameters.
```csharp
public DocumentDbCommandQueryBase(DocumentDbOptions options)
{
this.options = options;
this.DocumentClient = new DocumentClient(new System.Uri(options.EndpointUrl), options.AuthorizationKey);
}
```
After that you can get the database to work with or create it if it’s not exist already :

```csharp
protected async Task<Microsoft.Azure.Documents.Database> GetDatabaseAndCreateIfNotExists()
{
var database = this.DocumentClient.CreateDatabaseQuery()
.Where(d => d.Id == this.options.DatabaseId)
.AsEnumerable()
.FirstOrDefault();

if (database == null)
{
database = await this.DocumentClient.CreateDatabaseAsync(
new Microsoft.Azure.Documents.Database()
{
Id = this.options.DatabaseId
});
}

return database;
}
```
Relational databases like SQL Azure use table to store data. In Document DB, the documents are stored in document collections that you can get or create using the DocumentClient:

```csharp
protected async Task<Microsoft.Azure.Documents.DocumentCollection> GetDocumentCollectionAndCreateIfNotExists(string collectionId)
{
var database = await this.GetDatabaseAndCreateIfNotExists();

var documentCollection = this.DocumentClient.CreateDocumentCollectionQuery(database.SelfLink)
.Where(d => d.Id == collectionId)
.AsEnumerable()
.FirstOrDefault();

if(documentCollection == null)
{
documentCollection = await this.DocumentClient.CreateDocumentCollectionAsync(
database.SelfLink,
new Microsoft.Azure.Documents.DocumentCollection()
{
Id = collectionId
});
}

return documentCollection;
}
```
### Query the document database to list all posts

It’s possible to query the database with a pseudo SQL language or using LINQ. I chose the second one. You can find more information about querying DocumentDb [here](http://azure.microsoft.com/fr-fr/documentation/articles/documentdb-sql-query/).
To create a query you also need to use the DocumentClient class and call the CreateDocumentQuery method:
```csharp
var documentCollection = await this.GetDocumentCollectionAndCreateIfNotExists("Posts");

var postsQuery = this.DocumentClient
.CreateDocumentQuery<Domain.Entities.Post>(documentCollection.DocumentsLink)
.Where(p => p.IsPublished)
.AsQueryable();
```
You can call the extension method AsDocumentQuery on your linq query to get the underlying document query and execute it asynchronously:```csharp
var result = await documentQuery.ExecuteNextAsync<Domain.Entities.Post>();
```
Like the Azure table storage, DocumentDb uses continuation token when returning the documents :

```csharp
var posts = new List<Domain.Entities.Post>();
posts.AddRange(result);
while (documentQuery.HasMoreResults)
{
result = await documentQuery.ExecuteNextAsync<Domain.Entities.Post>();
posts.AddRange(result);
}
```
### Add a post document in the database

Adding a document in the database is pretty simple! You have to get the documents collection on your database and then call the CreateDocumentAsync method with the entity as parameter:

```csharp
var documentCollection = await this.GetDocumentCollectionAndCreateIfNotExists("Posts");
await this.DocumentClient.CreateDocumentAsync(documentCollection.SelfLink, this.post);
```
Cool, isn’t it ?

## Conclusion

In this post we have seen how to use Visual Studio 14 CTP to create new ASP.NET vNext projects, handle the new dependencies file project.json and how to use the DocumentDB SDK to create / read data from / to the documents collection.

The code is now available on [Github](https://github.com/jcorioland/metroblog/), so feel free to browse it if you want.

In the next post we will start to really use ASP.NET vNext to display the post !

Stay tuned

Julien

