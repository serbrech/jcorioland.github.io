---
layout: post
title: "[ASP.NET MVC] Repository Entity Framework dans un contexte stateless"
date: 2011-07-27 13:44:57 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: c4235b76-4324-4894-a769-2970d39d2272
redirect_from:
  - /archives/aspnet-mvc-repository-entity-framework-dans-un-contexte-stateless
  - /Archives/aspnet-mvc-repository-entity-framework-dans-un-contexte-stateless
---

Les mappeurs objets relationnels comme Entity Framework apportent de nombreux avantages lors du développement d’application utilisant des bases de données comme système de stockage. Au delà d’effectuer un mapping entre le monde objet et le monde relationnel, permettre la génération de requête SQL à partir du langage .NET LINQ, Entity Framework fournit également un système de détection de changement sur les entités afin de générer uniquement le nécessaire lorsque l’on demande un commit sur la base de données.

L’idée est assez simple au final : toute entité chargée depuis un contexte Entity Framework possède un état :

- **Unchanged** : l’entité n’est pas modifiée, aucune requête SQL ne sera générée pour celle-ci lors du commit
- **Added** : l’entité a été ajoutée au contexte, un ordre INSERT sera généré lors du commit
- **Modified** : l’entité a été modifiée depuis qu’elle a été chargée via le contexte, un ordre UPDATE sera généré lors du commit
- **Deleted** : l’entité a été supprimée du contexte, un ordre DELETE sera généré lors du commit

Il existe un dernier état que peut prendre une entité : **Detached**, c’est à dire qu’elle n’a pas été récupérée via le contexte Entity Framework.

L’état d’une entité est maintenue au sein du contexte par l’**ObjectStateManager**. Dès lors que le contexte Entity Framework est disposé, toutes les informations relatives au suivi des changements sont perdues.

ASP.NET MVC de part sa nature stateless force une très courte durée de vie pour un contexte Entity Framework : j’envoie une requête HTTP au serveur, le contexte Entity Framework est créé, la récupération de données est effectuée, le contexte est libéré et la connection à la base de données fermée, impliquant que le suivi des changements soit impossible dans ce cas : comment replacer mon objet dans le contexte lors que je post un formulaire HTML ?

Pour l’exemple, nous allons travailler avec l’entity data model suivant :

![image](/images/aspnet-mvc-repository-entity-framework-dans-un-contexte-stateless/a1f0dc2a-5290-4a3d-ae30-839901b04299.jpg)

Créeons à présent la classe repository qui va permettre d’effectuer les opérations de CRUD sur un item de type Blog :

```csharp
public class BlogRepository : IDisposable
{
private readonly BlogsContainer _blogsContainer = new BlogsContainer();

public void AddBlog(Blog blog)
{
_blogsContainer.Blogs.AddObject(blog);
_blogsContainer.SaveChanges();
}

public Blog GetBlog(int blogId)
{
return _blogsContainer.Blogs.SingleOrDefault(b => b.Id == blogId);
}

public IEnumerable<Blog> GetBlogs()
{
return _blogsContainer.Blogs.ToList();
}

public void DeleteBlog(Blog blog)
{

}

public void UpdateBlog(Blog blog)
{

}

public void Dispose()
{
_blogsContainer.Dispose();
}
}
```

Comme il est possible de le constater dans l’extrait de code ci-dessus, les opérations Add/Get sont réalisées de manière tout à fait classique sur l’object context Entity Framework. Cela va en revanche différer pour les méthodes Delete et Update, volontairement laissées vides pour le moment.

En effet, dans un contexte stateless, il n’est pas possible de bénéficier directement du mécanisme de change tracking proposé par Entity Framework : il faut auparavant attacher l’objet au contexte s'il ne l’est pas déjà.

Pour cela, l’object context expose une propriété **ObjectStateManager** qui va permettre de récupérer une **ObjectStateEntry** associée a une entité : si celle-ci existe, l’élément est connu du contexte, sinon il est inconnu :

```csharp
ObjectStateEntry stateEntry;
if (_blogsContainer.ObjectStateManager.TryGetObjectStateEntry(blog, out stateEntry))
{
//l'objet Blog est connu du change tracker
}
else
{
//l'objet blog est inconnu du change tracker
}
```

Du coup, dans le cas d’un delete, on vérifie si l’objet est connu du change tracker : si oui, pas de souci, sinon on attache l’objet au contexte avant de demander sa suppression :

```csharp
public void DeleteBlog(Blog blog)
{
ObjectStateEntry stateEntry;
if (!_blogsContainer.ObjectStateManager.TryGetObjectStateEntry(blog, out stateEntry))
{
_blogsContainer.Blogs.Attach(blog);
}

_blogsContainer.Blogs.DeleteObject(blog);
_blogsContainer.SaveChanges();
}
```

De la même manière, pour la mise à jour il faut vérifier si l’objet est connu du change tracker. Si oui, on va pouvoir utiliser la méthode `ApplyCurrentValues` de l’object set Blogs pour appliquer les changements et forcer le passage de l’entité dans l’état `Modified`. Sinon, il faut forcer le changement d’état via l’object state manager, après avoir attacher l’entité blog :

```csharp
public void UpdateBlog(Blog blog)
{
ObjectStateEntry stateEntry;
if (!_blogsContainer.ObjectStateManager.TryGetObjectStateEntry(blog, out stateEntry))
{
_blogsContainer.Blogs.Attach(blog);
_blogsContainer.ObjectStateManager.ChangeObjectState(blog, System.Data.EntityState.Modified);
}
else
{
_blogsContainer.Blogs.ApplyCurrentValues(blog);
}

_blogsContainer.SaveChanges();
}
```

Et voilà un repository ultra simple, fonctionnel et adapté au développement d’applications stateless avec ASP.NET MVC.

A bientôt ![image](/images/aspnet-mvc-repository-entity-framework-dans-un-contexte-stateless/fb69f88c-fdd2-4965-8b0d-96165045c90a.jpg)

