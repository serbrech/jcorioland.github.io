---
layout: post
title: "[En] Enable or disable IE proxy using Powershell"
date: 2011-09-06 7:46:00 +02:00
categories:
- General
author: "Julien Corioland"
identifier: eec03230-094d-49f9-aaf2-a77fb92ca48f
redirect_from:
  - /archives/en-enable-or-disable-ie-proxy-using-powershell
  - /Archives/en-enable-or-disable-ie-proxy-using-powershell
---

I’m currently working in a company where I have to set up a proxy to connect my laptop to the Internet. Because I'm fed up to enable it each morning and disable it each evening, I made two Powershell scripts to do these operations :

***EnableProxy.ps1 :***

```ps
set-itemproperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings'
-name ProxyEnable -value 1
```

***DisableProxy.ps1 :***

```ps
set-itemproperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings'
-name ProxyEnable -value 0 
```

Et voilà !

Hope this helps ![image](/images/en-enable-or-disable-ie-proxy-using-powershell/e5a900f5-7560-4497-8c39-06708d65f0a2.jpg)

