---
layout: post
title: "Using Application Insights in a Windows Store application"
date: 2014-01-28 8:36:00 +01:00
categories:
- DevOps
- Windows 8
author: "Julien Corioland"
identifier: 4f91513d-676f-4454-9dd7-584e4304be3e
redirect_from:
  - /archives/using-application-insights-in-a-windows-store-application
  - /Archives/using-application-insights-in-a-windows-store-application
---

### Introduction

Few months ago Microsoft announced the release of Visual Studio Online (VSO). One of the new feature that comes with VSO is Application Insights. It allows to monitor the performances of your applications and to find out what your users are doing the most or the less in your app. This kind of information may be very useful to optimize users’ experience in your app or propose new feature, based on behavior analysis.

### Declare an application in the Visual Studio Online dashboard

Before using Application Insights, you have to declare the application in your dashboard on the VSO website. Go to your dashboard and click the item `Try Application Insights` :

![image](/images/using-application-insights-in-a-windows-store-application/image_1138CA43.png)

In the Add application section, follow the wizard to declare a Windows Store app

![image](/images/using-application-insights-in-a-windows-store-application/image_37310B4A.png)

Name your application an copy the generated ID.

You are now ready to use application insights in your app.

###

### How do I set up Application Insights in my project ?

Setting up Application Insights in a Windows Store application is very easy. First, you should change the build configuration of your application and choose between x64, x86 and ARM. Any CPU is not supported because application insights is distributed as a winmd native component.

<img alt="image" src="https://infiniteblogs.blob.core.windows.net/medias/2/image_thumb_22FB2F58.png" width="533" height="337">

Once you have changed the build configuration of your projet, you can search for a Nuget package named Application Insights Telemetry SDK for Windows Store Apps :

<img alt="image" src="https://infiniteblogs.blob.core.windows.net/medias/2/image_thumb_3061425E.png">

Do not waste time to look for the reference in your project after you have installed the package, it will be visible only after Visual Studio has been restarted (don’t ask why, it was the case for me… <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_7B07D052.png">).

Build the solution and check you have no error in the console.

###

### Using the Application Insights SDK in your application

The API is really easy to use. In the namespace Microsoft.ApplicationInsights.Telemetry.WindowsStore, you will find a class ClientAnalyticsSession that exposes a property **Default**. There are three interesting methods on this object :

- Start : takes the id of your application insights app and starts the analytics session
- LogEvent : logs a given event in your application (users behavior, most of the time)
- LogPageView : logs a page view

Now, you can implement your own tag plan in your app. You just have to call the Start method when your application launches and it’s ok !

### Application Insights, for what kind of information ?

Application Insights allows to get some performance information about your application, but also monitor usage patterns in the application. For example, you can know what are the versions most used by your users :

<img alt="image" src="https://infiniteblogs.blob.core.windows.net/medias/2/image_thumb_7986EE17.png" width="562" height="279">

You can also check the different resolutions or hardware of your users :

<img alt="image" src="https://infiniteblogs.blob.core.windows.net/medias/2/image_thumb_6DF130D8.png" width="564" height="350">

Or the languages :

<img alt="image" src="https://infiniteblogs.blob.core.windows.net/medias/2/image_thumb_346E20E1.png" width="564" height="443">

And of course the detail of each event that occurs in the application and that you have triggered through the API :

<img alt="image" src="https://infiniteblogs.blob.core.windows.net/medias/2/image_thumb_56C5B65A.png" width="566" height="180">

### Conclusion

As you can see, it is really easy to use Application Insights in a Windows Store application and the dashboard on the VSO website is very cool and very useful to read all the information about your application.

So do not hesitate anymore, use it !

Hope this helps !

Julien <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_68BF0990.png">

