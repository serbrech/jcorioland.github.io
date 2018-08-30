---
layout: post
title: "[ASP.NET MVC 4] Contrôleurs asynchrones !"
date: 2011-09-29 10:42:00 +02:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: cbb66151-b0d1-4a43-b374-0360fa562284
redirect_from:
  - /archives/aspnet-mvc-4-controleurs-asynchrones
  - /Archives/aspnet-mvc-4-controleurs-asynchrones
---

On le sait, cela a été largement abordé il y a deux semaines lors de la //Build conférence : le futur du développement est dans l’asynchrone.

Bien entendu, il était déjà possible de réaliser des contrôleur asynchrone dans les versions précédente d’ASP.NET MVC et ce en dérivant nos contrôleurs de la classe AsyncController. Le moteur MVC se basait alors sur des conventions :

- MonAction**Async** : méthode qui retourne void et qui démarre un traitement asynchrone
- MonAction**Completed** : méthode qui retourne un ActionResult (le résultat de l’action MVC `MonAction`, dans ce cas).

Afin de synchroniser le tout, l’AsyncController fournissait une propriété AsyncManager permettant notamment au moteur MVC de connaître le nombre d’opération asynchrone en cours, de gérer celles-ci et de passer les paramètres à la méthode de retour d’action (MonActionCompleted) à la fin du traitement asynchrone, et ce pour retourner le résultat.

Par exemple, le contrôleur ci-dessous expose une action GetJSonMovies asynchrone, qui se charge d’aller récupérer une liste de films via un flux OData :

```csharp
public class MoviesController : AsyncController
{
public ActionResult Index()
{
return View();
}

public void GetJsonMoviesAsync(int? page)
{
const int pageSize = 20;
int skip = pageSize * ((page ?? 1) - 1);
string url = string.Format("http://odata.netflix.com/[…]&$skip={0}&$top={1}",
skip, pageSize);

//on "déclare" l'opération asynchrone
AsyncManager.OutstandingOperations.Increment();

var webClient = new WebClient();
webClient.DownloadStringCompleted += OnWebClientDownloadStringCompleted;
webClient.DownloadStringAsync(new Uri(url));//on lance le traitement asynchrone
}

private void OnWebClientDownloadStringCompleted(object sender,
DownloadStringCompletedEventArgs e)
{
//le traitement asynchrone se termine
//on rajoute le résultat "movies" au paramètre du AsyncManager
//NB : il s'agit du nom du paramètre de la méthode GetJsonMoviesCompleted !!
List<Movie> movies = null;
if (AsyncManager.Parameters.ContainsKey("movies"))
{
movies = (List<Movie>)AsyncManager.Parameters["movies"];
movies.Clear();
}
else
{
movies = new List<Movie>();
AsyncManager.Parameters["movies"] = movies;
}

movies.AddRange(Movie.FromXml(e.Result));

//on indique que l'opération se termine (déclenche l'appel du "Action"Completed)
AsyncManager.OutstandingOperations.Decrement();
}

public ActionResult GetJsonMoviesCompleted(List<Movie> movies)
{
//on retourne le résultat Json
return Json(movies, JsonRequestBehavior.AllowGet);
}
}
```

Nous sommes d’accord, ce code n’est pas extrêment compliqué, mais ASP.NET MVC 4, par le biais des nouveautés de C# 5 et de l’utilisation des mots clés **async** et **await** va grandement le simplifier, comme il est possible de le constater ci-dessous :

```csharp
public class MoviesController : AsyncController
{
public ActionResult Index()
{
return View();
}

public async Task<ActionResult> GetJsonMovies(int? page)
{
const int pageSize = 20;
int skip = pageSize * ((page ?? 1) - 1);
string.Format("http://odata.netflix.com/[…]&$skip={0}&$top={1}",
skip, pageSize);

var webClient = new WebClient();
string xmlResult = await webClient.DownloadStringTaskAsync(url);
return Json(Movie.FromXml(xmlResult), JsonRequestBehavior.AllowGet);
}
}
```

Comme vous pouvez le voir, en ASP.NET MVC 4 on continue à dériver **AsyncController**, en revanche plus besoin de faire appelle à l’AsyncManager ou même d’utiliser des conventions `Action**Async**/Action**Completed**` ! Notez également que la méthode ne retourne plus un ActionResult mais un Task<ActionResult> (induit par l’utilisation de async et await).

Franchement, ça poutre !

A bientôt ![image](/images/aspnet-mvc-4-controleurs-asynchrones/a9dd7904-9ac0-4a5a-be99-fe411038ab4b.jpg)

