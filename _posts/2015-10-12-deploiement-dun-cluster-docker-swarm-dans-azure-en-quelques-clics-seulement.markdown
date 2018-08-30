---
layout: post
title: "Déploiement d’un cluster Docker Swarm dans Azure"
date: 2015-10-12 15:39:26 +02:00
categories: Microsoft Azure, Docker
author: "Julien Corioland"
identifier: 625f744a-3f4d-4e7b-9f81-d26488d32151
redirect_from:
  - /archives/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement
  - /Archives/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement
---

[Docker Swarm](https://docs.docker.com/swarm/) est un outil qui permet d’orchestrer le déploiement de conteneurs Docker au sein d’un cluster. L’idée étant d’exécuter des commandes Docker comme vous le feriez sur un hôte Docker classique, mais de distribuer automatiquement ces commandes sur les différents nœuds du cluster.

Azure Resource Manager (ARM) permet - entre autre - de déployer une solution composée de plusieurs machines et/ou services Microsoft Azure, depuis le portail Preview à partir d’un fichier JSON qui en décrit la structure. De nombreux [templates sont à votre disposition sur GitHub](http://github.com/Azure/azure-quickstart-templates), pour vous aider à démarrer plus vite ou à construire les vôtres. (Si vous n’êtes pas encore familier avec ARM, je vous invite à lire [templates sont à votre disposition sur GitHub](http://github.com/Azure/azure-quickstart-templates))

Récemment, un template qui permet de déployer un cluster Docker Swarm a été rendu disponible ([ici](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-swarm-cluster)). Il permet de déployer deux groupes de haute disponibilité au sein d’un même réseau virtuel :

- Un pour les masters Swarm (au total 3), derrière un load balancer  - Un pour les nœuds Swarm (nombre de machines à définir au moment du déploiement du template), également derrière un load balancer, mais non exposé sur Internet

Vous trouverez une description complète du cluster qui est déployé par le template sur la page GitHub, directement.

Avant de le déployer, il faut vous assurer de posséder une clé SSH, pour pouvoir vous y connecter ensuite. Si vous êtes sur Linux ou MacOS X, cette étape est un détail. Si vous êtes sous Windows, je vous invite à télécharger [Git for Windows](https://git-scm.com/download/win), qui vous permettra d’installer automatiquement des outils vous permettant la génération de cette clé, puis d’établir ensuite une connexion SSH depuis la ligne de commande.

Une fois installé, il vous suffit de lancer Git Bash, de taper la commande suivante, et de vous laisser guider :

<em>>  ssh-keygen.exe</em>

Une clé privée et une clé publique seront alors générée et placées pour vous dans le dosser `.ssh` de votre répertoire utilisateur. La clé publique vous sera demandée lors de la création du cluster Swarm.

Rendez-vous sur la page du template sur GitHub et cliquez sur le bouton `Deploy to Azure`.

![image](/images/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement/image_41F75C7A.png)

Lorsque cela vous est demandé, authentifiez-vous avec votre compte Azure, et vous devriez alors vous retrouver sur une page vous demandant la saisie de différents paramètres :

![image](/images/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement/image_3F51317A.png)

Concrètement, il vous faut saisir :

- le nom d’un compte de stockage à créer  - le nom d’utilisateur / administrateur qui pourra se connecter au cluster  - votre clé publique SSH (à préfixer avec ssh-rsa <VOTRE CLE PUBLIQUE>)  - Le nombre de nœuds Swarm que vous souhaitez créer

Choisissez / créez un groupe de ressources dans lequel le cluster doit être déployé, acceptez les conditions et validez en cliquant sur le bouton Créer.

Une fois le déploiement terminé, vous devriez voir les différents éléments qui ont été créés pour vous :

![image](/images/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement/image_21067A84.png)

![image](/images/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement/image_42F1DD08.png)

Le template a été configuré de manière à ce que les noeuds du cluster Swarm ne soient pas directement accessibles sur Internet (pas d’IP publique). Pour vous y connecter, il faut donc passer par l’un des master (port 2200, 2201 ou 2202) sur le DNS <nom_dns_renseigné_à_la_création>-manage.<location>.cloudapp.azure.com. Par exemple, si vous avez choisi le nom DNS **swarmcluster** à la création et que vous avez déployé le cluster dans le data center **West Europe**, vous pouvez alors vous connecter au master grâce à la commande :

> ssh -A nom_utilisateur@**swarmcluster**-manage.**westeurope**.cloudapp.azure.com

Vous pouvez alors ensuite exécuter la commande suivante pour récupérer des infos sur les nœuds Docker qui sont disponibles :

> docker -H tcp://localhost:2375 info

![image](/images/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement/image_04F84C4A.png)

Il ne vous reste plus qu’à déployer des conteneurs dans le cluster. A chaque fois que vous allez demander le déploiement d’un conteneur, Swarm va automatiquement sélectionner un nœud du cluster dans lequel le déployer. Par exemple, si vous tapez 2 fois la commande suivante :

> docker -H tcp://localhost:2375 run -d -p 80:80 nginx

Puis :

> docker -H tcp://localhost:2375 ps -a

Vous pourrez alors constater que deux conteneurs basés sur l’image Nginx ont été démarrés, sur deux nœuds différents du cluster:

![image](/images/deploiement-dun-cluster-docker-swarm-dans-azure-en-quelques-clics-seulement/image_747FDB4E.png)

Le tour est joué, votre cluster Docker Swarm est en ligne !

A+

Julien

