---
layout: post
title: "Using OWIN to test your web api controllers"
date: 2014-04-01 8:42:00 +02:00
categories:
- ASP.NET MVC
- ASP.NET Web API
author: "Julien Corioland"
identifier: ac109bb5-e8ec-4664-87b5-97f439d48716
redirect_from:
  - /archives/using-owin-to-test-your-web-api-controllers
  - /Archives/using-owin-to-test-your-web-api-controllers
---

During the Microsoft Techdays in Paris, I spoke with Simon Ferquel about the newest features that came with ASP.NET MVC 5 and Web API 2. A big part of our session was about OWIN that is (IMO) one of the most exciting feature that has been released.

### Introduction to OWIN middleware programming

###

In short, OWIN (**O**pen **W**eb **I**nterface for .**N**et) is a set of specifications that define how a web application runs on a web server. You can find more about these specifications on [this page](http://owin.org/).

OWIN is based on middlewares to add behaviors to an application. For example, you can use OWIN middlewares to do external authentication with providers like Facebook, Google, Microsoft or Windows Azure Active Directory. You can also use OWIN middlewares as simple technical components of your application (logs, for example). Each OWIN middleware is responsible for its business.

Working with OWIN is very easy : first you have to write your OWIN middleware (or not, if you’re using an existing one provided by Microsoft or another third party).

An OWIN middleware is represented by a class that inherits from **OwinMiddleware**, as it is done for the LogsMiddleware bellow :

```csharp
public class LogMiddleware : OwinMiddleware
{
public LogMiddleware(OwinMiddleware next) : base(next)
{

}

public async override Task Invoke(IOwinContext context)
{
Debug.WriteLine("Request begins: {0} {1}", context.Request.Method, context.Request.Uri);
await Next.Invoke(context);
Debug.WriteLine("Request ends : {0} {1}", context.Request.Method, context.Request.Uri);
}
}
```
As you can see, an OWIN middleware takes the next middleware to invoke as constructor parameter. You should next override the **Invoke** method that will do the middleware stuff and invoke the next middleware.

The sample above is very simple : it logs the begin and the end of each request that is made to the server. Once the middleware is ready, you only have to register it in your application and it will be available for all technologies in your app (mvc, web forms, web api, signalR…) :<em> you write one middleware to rule them all !</em> <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_5503DC4D.png">

To register OWIN middlewares in your app, you have to create an OWIN Startup Class. You can use a Visual Studio template to create this class (or you can use the one that has been created at the project creation if it already exists) and all the structure of the class :

![image](/images/using-owin-to-test-your-web-api-controllers/image_4901EC19.png)

```csharp
using Microsoft.Owin;
using Owin;

[assembly: OwinStartupAttribute(typeof(EbookManager.Web.Startup))]
namespace EbookManager.Web
{
public partial class Startup
{
public void Configuration(IAppBuilder app)
{

}
}
}
```
An OWIN startup class is very simple : it contains a method Configuration that returns void and takes an IAppBuilder instance as parameter. To make the class visible from OWIN, you should use the attribute OwinStartupAttribute at the assembly level and reference the type of the startup class. Now, you can register the log middleware and test your first OWIN middleware :

```csharp
public void Configuration(IAppBuilder app)
{
app.Use(typeof(LogMiddleware));
}
```
And that’s all : your application will now use the log middleware via OWIN.

### OWIN Self-Hosting

One of the other cool exciting feature of OWIN is the ability to self-host code like Web API or Signal-R :

![image](/images/using-owin-to-test-your-web-api-controllers/image_4AD241E0.png)

Microsoft provides different hosts that can run OWIN applications. The first one that everybody knows is IIS server that runs ASP.NET for many years. But Microsoft provides two other hosts : the http listener that allows to self-host a web api or a signal-r hub in a .NET application (for example a windows service or a console application) and the unit test host that allows to unit tests web api controllers without server-deployment.

### Using the unit test host

To work with the OWIN Unit Test host, you have to install the NuGet package Microsoft.Owin.Testing in your test project :

![image](/images/using-owin-to-test-your-web-api-controllers/image_2AB9C5D4.png)

Once the package is installed, it is possible to create a test server that will host your web api without running IIS. To do that, you have to create an OWIN configuration class in the projet. This class is defined in the same way than an OWIN startup class except that you do not have to register it at the assembly level with the OwinStartupAttribute :

```csharp
class OwinTestConf
{
public void Configuration(IAppBuilder app)
{
app.Use(typeof (LogMiddleware));
HttpConfiguration config = new HttpConfiguration();
config.Services.Replace(typeof(IAssembliesResolver), new TestWebApiResolver());
config.MapHttpAttributeRoutes();
app.UseWebApi(config);
}
}

```
Here, the configuration is done in the same way that you are use to configure an api written with Web API 2. The **TestWebApiResolver** is an assembly resolver that allows Web API to resolve controllers that are defined in another assembly than the one that defines unit tests.

Now, it is possible to create a test server based on the OWIN configuration in your unit test and use it like any Web API, for example with an HttpClient :

```csharp
using (var server = TestServer.Create<OwinTestConf>())
{
using (var client = new HttpClient(server.Handler))
{
var response = await client.GetAsync("http://testserver/api/values");
var result = await response.Content.ReadAsAsync<List<string>>();
Assert.IsTrue(result.Any());
}
}

```
As you can see in the sample bellow, the HttpClient takes an custom handler that is provided by the OWIN test server. This is this step that allows us to `bypass` the use of the http protocol and that allows the tests to run without IIS.

### Conclusion

I think that the ability to self-host a Web API in a test server to make unit test is a very cool feature because it really simplifies the process.

In my next post, I’ll discuss about testing a Web API that uses OAuth with OWIN.

If you’re interested in reading source code, you’ll find samples of OWIN usages [in this project on github](https://github.com/simonferquel/techdays-paris-2014-mvc-webapi) (this is the sample application we use at Microsoft Techdays).

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_5503DC4D.png">

