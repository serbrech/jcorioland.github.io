---
layout: post
title: "[ASPNET Web API] Forcer le content type pour une action donnée"
date: 2012-10-17 10:42:00 +02:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: 23a19cd5-989e-46dc-9e7a-b1fd2fae8b64
redirect_from:
  - /archives/aspnet-web-api-forcer-le-content-type-pour-une-action-donnee
  - /Archives/aspnet-web-api-forcer-le-content-type-pour-une-action-donnee
---

Avec ASP.NET Web API, la content négociation se fait à l’aide du header <em>Accept</em> de la requête HTTP. Lorsque le type demandé n’est pas pris en charge, la requête est alors traitée avec le formateur par défaut, en l’occurrence le formateur JSON.

Cependant, dans certains cas il est nécessaire de forcer  le format de retour de la requête. Par exemple pour retourner un flux RSS.

Considérons le code suivant :

```csharp
public class RssController : ApiController
{
private readonly IBlogService _blogService;
public RssController()
{
var dbContext = new MetroBlogDbContext();
_blogService = new BlogService(dbContext);
}

public IEnumerable<Post> Get()
{
return _blogService.GetLatestPosts(true, 0, 20).ToList();
}
}
```

Il s’agit d’un simple contrôleur Web API qui retourne une liste d’objet Post (la définition n’a pas d’importance ici). Pour être capable de retourner un format de flux RSS, j’ai écrit un custom formatter pour RSS/Atom (cf. <a title="http://www.strathweb.com/2012/04/rss-atom-mediatypeformatter-for-asp-net-webapi/" href="http://www.strathweb.com/2012/04/rss-atom-mediatypeformatter-for-asp-net-webapi/">http://www.strathweb.com/2012/04/rss-atom-mediatypeformatter-for-asp-net-webapi/</a>).

Une fois le formateur développé, il suffit de l’enregistrer au démarrage de l’application :

```csharp
GlobalConfiguration.Configuration.Formatters.Add(new SyndicationFeedFormatter());
```

Cette solution permet de faire en sorte que lorsque le header <em>Accept</em> de la requête HTTP est à `<em>application/rss+xml</em>` ou `<em>application/atom+xml</em>`, le formateur <em>SyndicationFeedFormatter</em> soit utilisé et donc que le format en sortie soit un flux RSS. Ce n’est cependant pas suffisant pour que l’action Get renvoie toujours un flux RSS!

Pour le forcer, il est nécessaire de modifier le code de l’action Get et de créer nous même l’HttpResponseMessage qui sera renvoyé :

```csharp
public HttpResponseMessage Get()
{
try
{
var posts = _blogService.GetLatestPosts(true, 0, 20).ToList();
var responseMessage = new HttpResponseMessage(HttpStatusCode.OK);
responseMessage.Content = new ObjectContent(typeof(IEnumerable<Post>), posts, new SyndicationFeedFormatter());
responseMessage.Content.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue("application/rss+xml");
return responseMessage;
}
catch(Exception ex)
{
throw new HttpResponseException(HttpStatusCode.InternalServerError);
}
}
```

A partir de là, quelque soit le header <em>Accept</em> qui est envoyé, le format retourné sera `<em>application/rss+xml</em>`.

Il est également possible de filtrer certains type en se basant sur le header <em>Accept</em> de la requête afin de forcer le format que dans certains cas :

```csharp
public HttpResponseMessage Get()
{
try
{
var posts = _blogService.GetLatestPosts(true, 0, 20).ToList();
var responseMessage = new HttpResponseMessage(HttpStatusCode.OK);

if (Request.Headers.Accept.Any(a => a.MediaType == "application/json"))
{
responseMessage.Content = new ObjectContent(typeof(IEnumerable<Post>), posts, new JsonMediaTypeFormatter());
responseMessage.Content.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue("application/json");
}
else
{
responseMessage.Content = new ObjectContent(typeof(IEnumerable<Post>), posts, new SyndicationFeedFormatter());
responseMessage.Content.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue("application/rss+xml");
}

return responseMessage;
}
catch(Exception ex)
{
throw new HttpResponseException(HttpStatusCode.InternalServerError);
}
}
```

Dans le cas présent, si le header <em>Accept</em> de la requête contient le media type `<em>application/json</em>`, on renvoie un retour en JSON, sinon on renvoie le flux RSS formaté.

A bientôt !

