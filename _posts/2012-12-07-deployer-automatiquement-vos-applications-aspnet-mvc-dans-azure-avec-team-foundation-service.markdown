---
layout: post
title: "Déployer automatiquement vos applications ASPNET MVC dans Azure avec Team Foundation Service"
date: 2012-12-07 9:03:41 +01:00
categories:
- ASP.NET MVC
- Microsoft Azure
author: "Julien Corioland"
identifier: e8fdd5e5-4b10-4fc6-bf90-4b1c6cd3ba1f
redirect_from:
  - /archives/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service
  - /Archives/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service
---

<em>Team Foundation **Service**</em> (anciennement TFS Preview) est la version hébergée de <em>Team Foundation **Server**</em>. Pour en savoir plus sur ce service proposé par Microsoft, je vous invite à lire [cet article](http://blogs.codes-sources.com/etienne/archive/2012/10/31/tfs-preview-n-est-plus-une-preview-tfs-visualstudio-com.aspx) et [cet article](http://blogs.codes-sources.com/etienne/archive/2012/10/31/tfs-preview-n-est-plus-une-preview-tfs-visualstudio-com.aspx) sur le blog d’[cet article](http://blogs.codes-sources.com/etienne/archive/2012/10/31/tfs-preview-n-est-plus-une-preview-tfs-visualstudio-com.aspx) !

Une des fonctionnalité très sympa lorsque l’on développe des applications destinées à être hébergées dans <em>Windows Azure</em>, est la possibilité d’associer automatiquement un projet d’équipe <em>Team Foundation Service</em> à un <em>Cloud Service</em> Azure. Cela permet de faire en sorte qu’à chaque check-in depuis Visual Studio, le code soit compilé et automatiquement déployé dans Azure.

Pour associer un projet d’équipe à un service de compute Azure, il faut se connecter sur l’interface de management Windows Azure puis se rendre sur le tableau de bord du service en question :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_17E41F5B.png)

Cliquez sur **Configurer la publication TFS**. La popup qui s’ouvre vous permet soit d’utiliser un compte TFService déjà existant, soit de créer un compte sur visualstudio.com :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_6A82898A.png)

Saisissez le nom de votre compte de service si vous en avez déjà un. Si ce n’est pas le cas, cliquez sur **Créez un compte TFS maintenant**. La popup qui s’ouvre vous permet de créer un compte TFS Service :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_42DC0D60.png)

Créez votre compte. Pour les utilisateur existant, cliquez sur le lien **Autorisez maintenant**.

Une fenêtre s’ouvre alors pour vous demander d’autoriser votre service Windows Azure à accéder à TFS, un peu comme cela pourrait être pour n’importe quel membre de l’équipe :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_2D122503.png)

Si vous êtes d’accord, acceptez <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Sourire" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-smile_5307D559.png"> Vous pouvez à présent choisir le projet d’équipe à associer :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_0BB27F67.png)

Patientez pendant la liaison du projet d’équipe à votre service Windows Azure :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_4B1032F7.png)

Une fois ceci fait, ouvrez Visual Studio et connectez- vous à votre projet d’équipe. Dans le Team Explorer, affichez les définitions Builds configurées. Comme vous pouvez le constater, une nouvelle Build est présente. Elle porte le nom de votre service Cloud Azure, suffixé par `_CD` pour Continuous Deployment :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_7F445C3D.png)

Si vous ouvrez la définition de la build, vous pourrez voir que celle ci est configurée pour être exécuté à chaque check-in (Continuous Integration), dans l’onglet Trigger. Si vous allez dans l’onglet Process, vous constaterez que le process template utilisé est un spécifique à Windows Azure : **AzureContinuousDeployment.11.xaml**. En ouvrant le workflow de build, et en fouillant un peu dans les différentes activités, vous finirez par trouver la partie qui concerne le déploiement dans Azure :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_3782D356.png)

Concrètement, vous n’avez rien à modifier pour que ça marche <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Clignement d'œil" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_3DC9A9E4.png">

Afin d’avoir la main sur le profil de publication qui doit être utilisé par la build lors du déploiement, vous devez en créer un (ou utiliser un existant, si vous en avez déjà un). Pour se faire, faites un clic droit sur votre projet Cloud dans Visual Studio et cliquez sur **Publish…**

Dans la fenêtre qui s’ouvre, choisissez votre souscription Azure :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_0F702437.png)

Configurez ensuite les paramètres en choisissez votre Cloud Service, l’environnement sur lequel vous souhaitez déployer (Production / Staging), la build configuration et la service configuration à utiliser :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_3C18DE10.png)

Sur la page de résumé, enregistrez le profile, mais ne cliquez pas sur Publish, cliquez sur Cancel pour quitter :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_3B407826.png)

Visual Studio a créé un fichier XML qui représente ce profil, dans le dossier `Profiles` du projet Windows Azure. Retournez dans la définition de la build, dans l’onglet **Process** et déroulez la section** 5. Publishing – Azure Cloud Service. **Modifiez le paramètre Alternate Publish Profile pour le faire pointer sur le fichier de profil que vous venez de créer :

![image](/images/deployer-automatiquement-vos-applications-aspnet-mvc-dans-azure-avec-team-foundation-service/image_41874EB4.png)

Enregistrez les paramètres, et lancez la build. Le tour est joué.

***Note :*** pour le moment le SDK 1.8 de Windows Azure n’est pas pris en charge par l’agent de build hosté sur Team Foundation Service, mais cela devrait arriver. En attendant, il est possible d’indiquer que la build doit s’exécuter sur un agent de build configuré on-premise, par vos soins.

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Clignement d'œil" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_3DC9A9E4.png">

Julien

