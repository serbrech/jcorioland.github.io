---
layout: post
title: "[Windows Phone] Bonnes pratiques dans une tâche périodique avec un background agent"
date: 2011-10-25 10:14:00 +02:00
categories:
- Windows Phone
author: "Julien Corioland"
identifier: 965f2204-947d-428d-89a8-482f91d2ed6d
redirect_from:
  - /archives/windows-phone-bonnes-pratiques-dans-une-tache-periodique-avec-un-background-agent
  - /Archives/windows-phone-bonnes-pratiques-dans-une-tache-periodique-avec-un-background-agent
---

Vous le savez certainement déjà, la version 7.1 du SDK Windows Phone (`Mango`) offre la possibilité de créer des Background Agent, c’est à dire des tâches de fond exécutées par l’OS à interval régulier (en moyenne, selon certains paramètres, cf. plus bas).

Il existe deux types de tâches : périodique et traitement long. Dans ce post, nous nous intéresserons uniquement au premier type.

Dans ces tâches, il n’est pas possible de faire tout et n’importe quoi, seules certaines API sont autorisées (cf. [cette page de la MSDN](http://msdn.microsoft.com/en-us/library/hh202962(v=VS.92).aspx)). Cependant des scénarios assez sympa sont rendus possibles, notamment la mise à jour des Live Tile.

Quelques éléments importants sur les backgrounds agents :

- ils sont exécutés toutes les 30 minutes  - il est possible d’y exécuter du code pendant 25 secondes, pas plus!  - si le téléphone passe en mode d’économie de batterie, ils sont succeptibles de ne pas se lancer  - sur certain device, seuls 6 background agents peuvent être actif en simultanés  - ils ne doivent pas utiliser plus de 6MB de mémoire  - ils doivent être replannifié toutes les deux semaines  - un agent qui crash 2 fois est désactivé par l’OS

Cela fait pas mal de règles assez contraignantes mais il est vraiment possible de tirer profit de ces background agents en respectant quelques bonnes pratiques.

### Eviter la propagation d’exception dans un Background Agent

Comme vu plus tôt, un background agent qui crash 2 fois est automatiquement désactivé par l’OS. Vous devez à tout prix être attentif à ne pas laisser passer d’exception. Un scénario de crash assez courant et la mauvaise gestion des connexions et appels réseaux.

Par exemple, votre background agent lance une requête HTTP vers un service, la connexion est coupée, l’exception n’est pas catchée dans la callback => l’agent plante (il vous reste une vie!).

```csharp
try
{
var httpWebRequest = WebRequest.CreateHttp("http://www.monsite.com/service.aspx");
httpWebRequest.BeginGetRequestStream(asyncResult =>
{
try
{
var responseStream = httpWebRequest.EndGetRequestStream(asyncResult);
//gestion de la réponse
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
Comme il est possible de le constater dans l’extrait de code ci-dessus, il est important d’intercépter les exceptions lors de la création de la requête HTTP, mais également lors de la récupération de la réponse, dans la callback !

### Plannifier un Background Agent

Comme vu précédemment, les background agents sont automatiquement désactivés au bout de 2 semaines. Il est donc nécessaire de les replannifier fréquemment.

Lorsque l’on plannifie un background agent, il est possible que celui-ci ait été désactivé par l’utilisateur depuis l’interface du téléphone prévue à cet effet (dans les paramètres du téléphone). Dans ce cas une InvalidOperationException est levée lors de la tentative de plannification => il faut penser à catcher cette exception et éventuellement sauvegarder le fait que le background agent soit désactivé dans un fichier de configuration, par exemple.

La première étape consiste à récupérer une tâche périodique via son nom :

```csharp
var periodicTask = ScheduledActionService.Find("AgentDemo") as PeriodicTask;
```
Ensuite, on la supprime si elle existe (il est préférable de la supprimer et de la replannifier, afin de palier à la désactivation au bout de 2 semaines!) :

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
Il faut ensuite recréer la tâche et lui passer une description (obligatoire!) :

```csharp
periodicTask = new PeriodicTask("AgentDemo");
periodicTask.Description = "Description de l'agent";
```
Ensuite on tente la plannification (gare à l’InvalidOperationException si l’utilisateur a désactivé le background agent dans les paramètres de Windows Phone!) :

```csharp
try
{
ScheduledActionService.Add(periodicTask);
}
catch (InvalidOperationException ioe)
{
if (ioe.Message.Contains("BNS Error: The action is disabled"))
{
//mise à jour conf
}
}
```
Enfin, pour déboguer, il est possible de demander le lancement de l’agent après un temps donné (lorsque l’application est quittée via le bouton démarrer, pour concerver le débogueur attaché.)

```csharp
ScheduledActionService.LaunchForTest("DemoAgent", TimeSpan.FromSeconds(10));
```
Votre agent est prêt et fonctionnel !!

### Eviter de dépasser la limite mémoire autorisée

Une des règles à respecter dans un background agent est de ne pas dépasser 6MB de mémoire ! Si vous dépassez cette limite, l’agent est intérrompu et il est considéré comme planté, donc désactivé au bout de 2 fois. Tout au long du traitement, il est possible de tester la mémoire consommée par l’agent à l’aide de l’instruction :

```csharp
var memory = DeviceStatus.ApplicationMemoryUsageLimit
- DeviceStatus.ApplicationCurrentMemoryUsage;
```
Voilà quelques règles à respecter pour faire des background agents qui resteront actifs !

A bientôt <img style="border-bottom-style: none; border-left-style: none; border-top-style: none; border-right-style: none" class="wlEmoticon wlEmoticon-winkingsmile" alt="Winking smile" src="https://juliencorioland.blob.core.windows.net/medias/c3f8d0ab-c7b3-4f1a-9897-6df77a11f923">

