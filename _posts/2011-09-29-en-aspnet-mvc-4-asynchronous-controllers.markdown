---
layout: post
title: "[EN] [ASP.NET MVC 4] Asynchronous controllers"
date: 2011-09-29 11:03:00 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: c1179f17-28de-4360-9286-6680cf4dc58f
redirect_from:
  - /archives/en-aspnet-mvc-4-asynchronous-controllers
  - /Archives/en-aspnet-mvc-4-asynchronous-controllers
---

Asynchronous execution is the future of Windows development : it has been largely demonstrated during the //Build conference two weeks ago.

In previous versions of ASP.NET MVC it was possible to create asynchronous controllers by inheriting the AsyncController class and using some conventions :

- MyAction**Async** : method that returns void and launches an asynchronous process
- MyAction**Completed** : method that returns an ActionResult (the result of the MVC action `MyAction`, in this case)

To allow the MVC engine to manage asynchronous operations and pass the result to the view engine, developers had to use the propery AsyncManager of the AsyncController. The `completed` method parameters was passed by the MVC engine through this object.

For example, the controller that is defined bellow allows to get a Json-serialized list of movies – asynchronously – from an OData service :

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

//the asynchronous operation is declared
AsyncManager.OutstandingOperations.Increment();

var webClient = new WebClient();
webClient.DownloadStringCompleted += OnWebClientDownloadStringCompleted;
webClient.DownloadStringAsync(new Uri(url));//the asynchronous process is launched
}

private void OnWebClientDownloadStringCompleted(object sender,
DownloadStringCompletedEventArgs e)
{
//the asynchronous process ends
//"movies" result is added to the parameters of the AsyncManager
//NB : it's the name of the parameter that is take by the
//GetJsonMoviesCompleted method
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

//the ends of the asynchronous operation (launches the call of "Action"Completed)
AsyncManager.OutstandingOperations.Decrement();
}

public ActionResult GetJsonMoviesCompleted(List<Movie> movies)
{
//on retourne le résultat Json
return Json(movies, JsonRequestBehavior.AllowGet);
}
}
```

It’s not really complicated to create an asynchronous controller but ASP.NET MVC 4 and C# 5 with the new **async** and **await** keywords will make it easier !

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

As you can see in the previous code snippet, in ASP.NET MVC 4 you always should inherits from AsyncController but there is no more naming conventions, no more Async/Completed methods, no more AsyncManager and the action returns a Task<actionresult> instead of an ActionResult !

It’s awesome !

Hope this helps ![image](/images/en-aspnet-mvc-4-asynchronous-controllers/234129fb-07df-470d-b429-ee7472ebc124.jpg)

