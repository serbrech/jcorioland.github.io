---
layout: post
title: "Run background tasks in aspnet applications with dotnet Framework 4.5.2"
date: 2014-05-13 8:30:00 +02:00
categories:
- ASP.NET MVC
- ASP.NET Web API
author: "Julien Corioland"
identifier: 2350ebad-fe86-415c-9658-48f2a7108ef2
redirect_from:
  - /archives/run-background-tasks-in-aspnet-applications-with-dotnet-framework-452
  - /Archives/run-background-tasks-in-aspnet-applications-with-dotnet-framework-452
---

Few days ago Microsoft has announced the availability of the .NET Framework 4.5.2. This new version brings some new features and improvements to ASP.NET. It is now very easy to run some background tasks from an ASP.NET Web app, using the new HostingEnvironment API. It allows to enqueue background tasks (as simple as working with the thread pool) and avoids IIS app pools shutdown until the tracked tasks are completed.

For a complete description of this new .NET Framework version, go to [this page](http://msdn.microsoft.com/en-us/library/ms171868(v=vs.110).aspx). You can find the runtime and developer pack installers [this page](http://msdn.microsoft.com/en-us/library/ms171868(v=vs.110).aspx).

Once you’ve installed .NET Framework 4.5.2 and the developer tools, open Visual Studio and create a new web application project that targets the .NET Framework 4.5.2 :

![image](/images/run-background-tasks-in-aspnet-applications-with-dotnet-framework-452/image_46BAC70C.png)

In this post, I have choose to use the HostingEnvironment api to queue thumbnail generation when the user uploads a picture (really simple scenario). The HostingEnvironment class defines a method QueueBackgroundWorkItem that takes an Action<CancellationToken> or a Func<CancellationToken,Task> as parameters.

To generate the thumbnail I have used the System.Drawing API and created the following simple helper :

```csharp
public class ThumbnailHelper
{
public static Task CreateThumbnail(string filePath)
{
return Task.Factory.StartNew(() =>
{
Image originalImage = Bitmap.FromFile(filePath);

int thumbWidth, thumbHeight;

thumbWidth = 250;
thumbHeight = (originalImage.Height * 250) / originalImage.Width;

Image thumbnail = originalImage.GetThumbnailImage(thumbWidth, thumbHeight, null, IntPtr.Zero);

string thumbFileName = string.Format("{0}_thumb{1}", Path.GetFileNameWithoutExtension(filePath), Path.GetExtension(filePath));
string thumbFilePath = Path.Combine(Path.GetDirectoryName(filePath), thumbFileName);

thumbnail.Save(thumbFilePath);
});
}
}
```
The cshtml code :

```csharp
@{
ViewBag.Title = "Upload a new picture";
}

## @ViewBag.Title

@using(Html.BeginForm("Upload", "Picture", FormMethod.Post, new{ enctype = "multipart/form-data"}))
{
@Html.AntiForgeryToken()
<div class="form-group">
<label for="file">Choose a picture</label>
<input type="file" name="file" id="file" />

<div class="form-group">
<input type="submit" value="Upload and create thumbnail" class="btn btn-primary"/>

}

```
and the controller code :

```csharp
public class PictureController : Controller
{
public ActionResult Upload()
{
return View();
}

[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Upload(HttpPostedFileBase file)
{
var appDataPath = Server.MapPath("~/App_Data");
string filePath = Path.Combine(appDataPath, file.FileName);
file.SaveAs(filePath);

HostingEnvironment.QueueBackgroundWorkItem(cancellationToken =>
{
return ThumbnailHelper.CreateThumbnail(filePath);
});

return RedirectToAction("Upload");
}
}
```
As you can see in the code above, just after that the picture is saved in the App_Data folder, a task is queued using the HostingEnvironment api. Now, you can be sure that the app pool of your application will not stop until all the thumbnails are generated.

Hope this helps <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_666C3185.png">

Julien

