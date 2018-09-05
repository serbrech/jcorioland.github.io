---
layout: post
title: "Execution de librairie 32 bit dans un rôle Windows Azure"
date: 2013-05-14 9:13:00 +02:00
categories:
- Microsoft Azure
author: "Julien Corioland"
identifier: 1da6b4cf-5097-4b0a-bd97-f37b0df00c0f
redirect_from:
  - /archives/execution-de-librairie-32-bit-dans-un-role-windows-azure
  - /Archives/execution-de-librairie-32-bit-dans-un-role-windows-azure
---

Par défaut, il n’est pas possible d’exécuter une librairie en 32 bit dans un web rôle Windows Azure, tout simplement parce que la configuration de l’application pool IIS l’interdit. Cependant, il est possible de modifier cette configuration afin de rendre ce genre de scénario possible !

Pour cela, il faut exécuter un script de configuration de IIS, au sein d’une Startup Task. Les Startup Task Windows Azure permettent d’exécuter un script ou un programme (par exemple un installateur msi) avant que le rôle n’ait vraiment démarrer. Cela permet donc de préparer votre environnement, à chaque démarrage d’une instance d’un rôle.

Dans votre Web Role, commencez par ajouter un script enable32bit.cmd avec la commande ci-dessous :

```plain
%windir%\system32\inetsrv\appcmd set config -section:applicationPools -applicationPoolDefaults.enable32BitAppOnWin64:true
```
Il est important de définir sa `Build Action` à Content et sa propriété `Copy to output directory` à Copy always :

![image](/images/execution-de-librairie-32-bit-dans-un-role-windows-azure/image_5E07659F.png)

Enfin, il faut éditer le fichier de définition de la configuration des rôles de votre projet Cloud, le fichier ServiceDefinition.csdef et d’ajouter un noeud StartupTask sous le Web Role ciblé :

```xml
<Startup>
<Task commandLine="enable32bit.cmd" executionContext="elevated" taskType="simple" />
</Startup>
```
Le fait d’indiquer un executionContext `elevated` indique que l’on souhaite exécuter le script en mode de privilèges élevés.

Désormais, lorsque vous allez publier votre rôle sur Azure, le script sera automatiquement exécuté et vous pourrez exécuter du code 32 bit dans votre applicatif. Si vous vous connectez en bureau à distance à votre instance, vous pourrez d’ailleurs constater dans la console de gestion IIS, propriétés du pool applicatif, que le paramètre `Enable 32-Bit Applications` est à True :

![image](/images/execution-de-librairie-32-bit-dans-un-role-windows-azure/image_3946285B.png)

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_57B06944.png">

Julien

