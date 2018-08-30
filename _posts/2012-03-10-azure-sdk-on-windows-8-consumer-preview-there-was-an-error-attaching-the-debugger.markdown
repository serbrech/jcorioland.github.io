---
layout: post
title: "Azure SDK on Windows 8 Consumer preview : there was an error attaching the debugger…"
date: 2012-03-10 19:42:00 +01:00
categories: Windows Azure, Windows 8
author: "Julien Corioland"
identifier: 6bbae0a1-dfd5-40cc-8787-e400d7fcbed1
redirect_from:
  - /archives/azure-sdk-on-windows-8-consumer-preview-there-was-an-error-attaching-the-debugger
  - /Archives/azure-sdk-on-windows-8-consumer-preview-there-was-an-error-attaching-the-debugger
---

When I was trying to debug a Windows Azure application in Visual Studio 2010 on Windows 8 Consumer Preview, I was getting the following error :

![image](/images/azure-sdk-on-windows-8-consumer-preview-there-was-an-error-attaching-the-debugger/a3a9bd27-19f4-43ab-af39-a408f0053766.jpg)

If you encounter this error, open the Control Panel, go to the Programs section and click the Turn Windows features on or off link. In the Windows features, check that the **Internet Information Service Hostable Web Core** feature is installed on your machine. If not, install it :

![image](/images/azure-sdk-on-windows-8-consumer-preview-there-was-an-error-attaching-the-debugger/7ce70d08-091f-4227-9dc3-b381aabf3429.jpg)

It solves my problem and now I can debug Windows Azure applications without any error ! It seems that the Web PI 4.0 preview does not detect this dependency while installing the Windows Azure SDK…

Hope this helps ![image](/images/azure-sdk-on-windows-8-consumer-preview-there-was-an-error-attaching-the-debugger/4981911f-27d3-475f-9307-e66d874bfdc4.jpg) !

Julien

