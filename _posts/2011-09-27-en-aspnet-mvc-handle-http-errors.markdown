---
layout: post
title: "[EN] [ASP.NET MVC] Handle HTTP errors"
date: 2011-09-27 11:57:00 +02:00
categories: ASP.NET MVC
author: "Julien Corioland"
identifier: 94a09c5d-6d4b-40d4-a769-a7a73d674f8d
redirect_from:
  - /archives/en-aspnet-mvc-handle-http-errors
  - /Archives/en-aspnet-mvc-handle-http-errors
---

This subject seems to be really basic when developing a web application but it seems that it’s very discussed on the web too. In a lot of forum threads or blog posts we can find very various methods to handle HTTP exception within an ASP.NET MVC application. This post exposes a way to do that. I’m not pretending that it’s the best way but I think that it’s an elegant one ![image](/images/en-aspnet-mvc-handle-http-errors/82ea65f8-bfc3-4ca3-a380-27907ed6cd55.jpg) (so don’t hesitate to comment on this post if you’ve remarks or suggestions).

### What’s the real need ?

I think that developers should manage HttpException in an other way than other exceptions for 3 main reasons :

<ol>- Respect of web-standards (a not found error returns a 404 http code, that’s it !)
- SEO : search engines use http codes (because it’s standards) to build their indexes.
- The user : a user-friendly page is already better than a stack trace…
</ol>I think that the most important is that the error code is well returned by the server : for example the URL http://www.mywebsite.com/Pages/some-page-that-does-not-exists is called the server should return a 404 status code and not a 302 (found) followed by a 200 (or 404) from a specific error page like http://www.mywebsite.com/Errors/Error404).

###  When HttpException are thrown in an ASP.NET MVC application ?

HttpException are thrown by the ASP.NET MVC engine or by the application code, directly in the asp.net controllers of your apps :

```csharp
throw new HttpException(404, "Not found");
```

In some cases, they can also be thrown by the server (for internal errors, for example). But all HttpException can be handled in the application.

### How to handle these exceptions ?

After a lot of researches and some tests I think that http exceptions should be handled in two places :

<ol>
- In a base controller inherited by all the controllers of the application

- In the Global.asax file

</ol>

*Handle http exception in the controller :*

Exceptions that are thrown by the developer directly in the application code can be handle in a base controller. For example :

```csharp
public ActionResult Product(int id)
{
var product = _unitOfWork.GetProduct(id);
if(product == null)
throw new HttpException(404, "Le produit est introuvable");

return View(product);
}
```

By inheriting the System.Web.Mvc.Controller class it’s possible to override an `OnException` method. This method is called when an exception occurs in a controller method (action) :

```csharp
protected override void OnException(ExceptionContext filterContext) {
base.OnException(filterContext);

if (filterContext.Exception != null) {
filterContext.ExceptionHandled = true;

if (filterContext.Exception is HttpException) {
if (!ControllerContext.RouteData.Values.ContainsKey("error")) {
ControllerContext.RouteData.Values.Add("error", filterContext.Exception);
}

var httpException = (HttpException) filterContext.Exception;

switch (httpException.GetHttpCode()) {
case 404:
filterContext.HttpContext.Response.StatusCode = 404;
filterContext.HttpContext.Response.StatusDescription = httpException.Message;
View("Error404", null).ExecuteResult(ControllerContext);
break;
case 500:
filterContext.HttpContext.Response.StatusCode = 500;
filterContext.HttpContext.Response.StatusDescription = httpException.Message;
View("Error500", null).ExecuteResult(ControllerContext);
break;
default:
filterContext.HttpContext.Response.StatusDescription = httpException.Message;
View("GenericError", null).ExecuteResult(ControllerContext);
break;
}
}

//autre traitement si pas HttpException (log par exemple...)
}
}
```

As you can see in this code snippet the first step is to check that the exception is an HttpException. If yes, a redirection to an appropriate error page is done just after setting the status code and status description of the http response. The views are in the `Shared` folder of the application.

Now http exceptions that are thrown in a controller return the good status code and redirect to a user friendly page.

*Handle HTTP exceptions in the Global.asax :*

It’s in the Global.asax file that all other http exception are handled. To to that you should subscribe to the Error event of the mvc application (in the Init method override) :

```csharp
public override void Init()
{
base.Init();
this.Error += new EventHandler(MvcApplication_Error);
}
```

In the Error event handler, initialize an errors-dedicated controller and create appropriate route data to work with the error :

```csharp
var routeData = new RouteData();
routeData.Values.Add("controller", "Errors");

var lastException = Server.GetLastError();

if (lastException is HttpException) {
var httpException = (HttpException) lastException;
switch(httpException.GetHttpCode()) {
case 404:
routeData.Values.Add("action", "Error404");
break;
case 500:
routeData.Values.Add("action", "Error500");
break;
default:
routeData.Values.Add("action", "GenericError");
break;
}
}
else
{
routeData.Values.Add("action", "GenericError");
}

routeData.Values.Add("exception", lastException);

Server.ClearError();

IController errorController = _unityContainer.Resolve<ErrorsController>();
errorController.Execute(new RequestContext(
new HttpContextWrapper(Context), routeData));
```

The ErrorsController :

```csharp
public class ErrorsController : Controller
{
public ActionResult Error404() {
Response.StatusCode = 404;
Exception exception = null;
if(RouteData.Values.ContainsKey("exception")) {
exception = (Exception) RouteData.Values["exception"];
}
return View(exception);
}

public ActionResult Error500() {
Response.StatusCode = 500;
Exception exception = null;
if (RouteData.Values.ContainsKey("exception"))
{
exception = (Exception)RouteData.Values["exception"];
}
return View(exception);
}

public ActionResult GenericError() {
Exception exception = null;
if (RouteData.Values.ContainsKey("exception"))
{
exception = (Exception)RouteData.Values["exception"];
}
return View(exception);
}
}
```

As you can see each action defines itself the http status code of the http response.

Now all http exceptions are handled in a user and SEO friendly way in your ASP.NET MVC applications !

Hope this helps ![image](/images/en-aspnet-mvc-handle-http-errors/e89176b7-5f2f-448f-9823-134e65753d2f.jpg)

