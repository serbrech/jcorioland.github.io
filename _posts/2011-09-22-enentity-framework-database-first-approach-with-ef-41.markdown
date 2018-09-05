---
layout: post
title: "[EN][Entity Framework] Database first approach with EF 4.1"
date: 2011-09-22 11:48:00 +02:00
categories:
- Web Development
author: "Julien Corioland"
identifier: 72df1873-197c-4dae-ae51-838994966730
redirect_from:
  - /archives/enentity-framework-database-first-approach-with-ef-41
  - /Archives/enentity-framework-database-first-approach-with-ef-41
---

As you probably already know the last version of Entity Framework (4.1) supports a new development scenario called `Code first` (also called code only in some cases). It consists to develop the .NET entities (POCOs) before the SQL relational model, create the database, map these entities to table and columns at runtime according the class definitions. To do that, there is no need of an EDMX file, metadata and mapping are based on conventions and attributes.

Entity Framework 4.1 provides a lot of new functionalities that may be used even if the `code first` approach is not applied for many reasons (existing database, database created and maintained by a DBA that doesn’t know .NET or Entity Framework…). To be able to use the new features of Entity Framework 4.1 it’s possible to continue with the database first approach and use an EDMX file to do the mapping between .NET objects (POCOs) and SQL tables and columns. This post explains how to do that !

### Import the Entity Framework 4.1 library

To use EF 4.1 it’s recommended to reference the related NuGet package using the NuGet management console in Visual Studio. Right click on the project that should reference the library and choose Manage NuGet packages…

![image](/images/enentity-framework-database-first-approach-with-ef-41/8140a49c-8268-4797-a240-543039f14abd.jpg)

In the NuGet management console, search for `Entity Framework` and click Install to download and reference the EntityFramework.dll library.

![image](/images/enentity-framework-database-first-approach-with-ef-41/aa0a2412-5933-4612-86e8-8de2cd6c5b92.jpg)

### Download the T4 code generation templates for EF 4.1

To generate the DbContext and the .NET entities from the EDMX definition it’s possible to download two code generation templates from the Visual Studio Extensions manager. Open it and search for `DbContext` in the online gallery :

![image](/images/enentity-framework-database-first-approach-with-ef-41/0c9bd5f7-54dd-4c6f-8d8f-efd5ae66f619.jpg)

Install the `ADO.NET C# DbContext Generator` item (or VB.NET).

### Delete the code generation tool from EDMX properties

When an ADO.NET entity data model is added to a project, it uses the default code generation template. To avoid this, you have to remove this tool in the property window of the EDMX file :

![image](/images/enentity-framework-database-first-approach-with-ef-41/31e06249-66bd-419b-951b-366ece363d4d.jpg)

Be careful if you’re working on an existing project where generated code may be in use !

### Add the templates to the project

Now you’ve to add the two generation code templates to the project. To do that add a new item of type `ADO.NET C# DbContext Generator` from the add item menu :

![image](/images/enentity-framework-database-first-approach-with-ef-41/9ab72d8f-27c5-4a79-8f36-a60e47518c30.jpg)

In these two files, find the text $edmxInputFile and replace it by the relative path to the EDMX file.

![image](/images/enentity-framework-database-first-approach-with-ef-41/6d088850-685e-40d8-9165-aab0eaed284f.jpg)

Save the files. Automatically the DbContext and the entities are generated :

![image](/images/enentity-framework-database-first-approach-with-ef-41/247b9026-a277-4d5e-b405-6a320aec9f9c.jpg)

Now you are ready to use the DbContext and query the database through LINQ to Entites !

```csharp
using (var dbContext = new BlogContainer()) {
var posts = dbContext.Posts.Where(p => p.IsPublished).ToList();

return View(posts);
}
```

After each change on the edmx file just run the custom tool on the two T4 templates to regenerate the DbContext and the entities.

Hope this helps ![image](/images/enentity-framework-database-first-approach-with-ef-41/dddea8f5-5156-45bc-bbb4-62d90d6d517b.jpg)

