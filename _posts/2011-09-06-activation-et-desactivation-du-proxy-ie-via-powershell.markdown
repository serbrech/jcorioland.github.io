---
layout: post
title: "Activation et désactivation du proxy IE via Powershell"
date: 2011-09-06 7:28:00 +02:00
categories: General
author: "Julien Corioland"
identifier: 5ac95852-419c-4618-9755-a5a0c797c6c0
redirect_from:
  - /archives/activation-et-desactivation-du-proxy-ie-via-powershell
  - /Archives/activation-et-desactivation-du-proxy-ie-via-powershell
---

En ce moment je suis chez un client où je dois activer un proxy pour me connecter à Internet. Comme j’en avais assez de devoir reconfigurer mon IE tous les soirs chez moi et tous les matins chez mon client, voilà deux scripts Powershell permettant de faire celà en deux clics :

***EnableProxy.ps1 :***

```ps
set-itemproperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings'
-name ProxyEnable -value 1
```

***DisableProxy.ps1***

```ps
set-itemproperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings'
-name ProxyEnable -value 0
```

Simple et efficace !

A bientôt ![image](/images/activation-et-desactivation-du-proxy-ie-via-powershell/04dced6b-31a0-4857-bb09-873169d48712.jpg)

