---
layout: post
title: "[EN] [Windows Phone] Best practices for periodic tasks with background agents"
date: 2011-10-25 16:51:00 +02:00
categories: Windows Phone
author: "Julien Corioland"
identifier: 03b5bae6-e64f-4886-84d8-6e017ffe49bf
redirect_from:
  - /archives/en-windows-phone-best-practices-for-periodic-tasks-with-background-agents
  - /Archives/en-windows-phone-best-practices-for-periodic-tasks-with-background-agents
---

As you probably already know Windows Phone 7.1 SDK (`Mango`) allows to create background agents to realize some operations when the application is not running.

There are two types of tasks : periodic tasks and long running tasks. This post is about periodic tasks that are executed each 30 minutes by the operating system. There are some limitations with the API that may be used in background agent. For further information please go to [this MSDN page](http://msdn.microsoft.com/en-us/library/hh202962(v=VS.92).aspx).

There are some constraints with background periodic tasks :

- automatically executed by the OS each 30 minutes
- the operation can’t exceed 25 seconds per run
- if the phone switch to battery saver mode the background agent may not be executed
- on some devices only 6 background agents may be planned simultaneously
- agents can’t use more that 6MB of memory
- agents have to be re-planned each 2 weeks
- an agent that crashes two times is automatically disabled by the system

### Avoid exception propagation in background agent

Because two crashes will disable the background agent exception handling is really important in the development. For example, common errors occur when using a WebRequest  : the agent creates an HTTP request, the connection is lost, the callback throws an exception that is uncatched ! To avoid this, you can use the following code :

```csharp
try
{
var httpWebRequest = WebRequest.CreateHttp("http://www.monsite.com/service.aspx");
httpWebRequest.BeginGetRequestStream(asyncResult =>
{
try
{
var responseStream = httpWebRequest.EndGetRequestStream(asyncResult);
//do stuff with the response
}
catch (Exception exception)
{
OnError(exception);
}
}, null);
}
catch (Exception exception)
{
OnError(exception);
}
```

### Schedule a background agent

Background agents are automatically disabled after two weeks : you have to re-schedule them. Windows Phone 7.5 has a centralized interface to manage all background agents that are available on the system so the user can disable an agent without run your application. If you try to schedule a periodic task with a disabled background agent an InvalidOperationException is thrown. You have to take care of this case and catch the exception as demonstrated bellow.

The first step is to get the periodic task by its name :

```csharp
var periodicTask = ScheduledActionService.Find("AgentDemo") as PeriodicTask;
```

Next, delete the periodic task if it exists :

```csharp
if (periodicTask != null)
{
try
{
ScheduledActionService.Remove(PeriodicTaskName);
}
catch (Exception) { }
}
```

Re-create the periodic task and set the description (it’s mandatory) :

```csharp
periodicTask = new PeriodicTask("AgentDemo");
periodicTask.Description = "Agent description";
```

Try to schedule the periodic task :

```csharp
try
{
ScheduledActionService.Add(periodicTask);
}
catch (InvalidOperationException ioe)
{
if (ioe.Message.Contains("BNS Error: The action is disabled"))
{
//agent is disabled for the application
}
}
```

Now the agent can be run by the system if it’s not disabled ![image](/images/en-windows-phone-best-practices-for-periodic-tasks-with-background-agents/626937fa-5522-4b98-93d2-52c5f2ee46fb.jpg)

### Memory usage limit

Background agent can’t use more than 6MB of memory. You can get the current memory usage using the following snippet :

```csharp
var memory = DeviceStatus.ApplicationMemoryUsageLimit
- DeviceStatus.ApplicationCurrentMemoryUsage;
```

Hope this helps ![image](/images/en-windows-phone-best-practices-for-periodic-tasks-with-background-agents/c9223d9f-3ae2-4e26-8309-b0c68a43a80f.jpg)

