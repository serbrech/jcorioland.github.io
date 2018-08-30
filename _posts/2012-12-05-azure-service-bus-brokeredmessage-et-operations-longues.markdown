---
layout: post
title: "Azure Service Bus, BrokeredMessage et opérations longues"
date: 2012-12-05 13:51:39 +01:00
categories: Windows Azure
author: "Julien Corioland"
identifier: 4abc9fb0-e795-4000-a0f6-c5f4f8b199f6
redirect_from:
  - /archives/azure-service-bus-brokeredmessage-et-operations-longues
  - /Archives/azure-service-bus-brokeredmessage-et-operations-longues
---

Le service de messaging d’Azure Service Bus n’est à l’origine pas prévu pour exécuter des long-running process. Cependant le SDK 1.8 sorti en Octobre dernier apporte une petite nouveauté sur ce point. En effet, la classe BrokeredMessage offre désormais une méthode `RenewLock` qui permet de demander le renouvèlement d’un verrou sur le brokered message afin qu’il ne soit pas remis à disposition dans la file d’attente.

Du coup, en se basant sur la propriété LockedUntilUtc, on peut réussir à exécuter une opération longue sans avoir de problème de perte de verrou sur le message :

```csharp
//création d'un CTS pour lancer un task en charge de renouveller le verrou du message
var brokeredMessageRenewCancellationTokenSource = new CancellationTokenSource();

try
{
//reception du message
var brokeredMessage = _client.Receive();

var brokeredMessageRenew = Task.Factory.StartNew(() =>
{
while (!brokeredMessageRenewCancellationTokenSource.Token.IsCancellationRequested)
{
//on se base sur la propriété LockedUntilUtc pour savoir si le verrou expire bientôt
if (DateTime.UtcNow > brokeredMessage.LockedUntilUtc.AddSeconds(-10))
{
//si oui, on renouvelle le message
brokeredMessage.RenewLock();
}

Thread.Sleep(500);
}
}, brokeredMessageRenewCancellationTokenSource.Token);

//exécution d'une opération longue
RunLongRunningOperation();

//on marque le message comme terminé
brokeredMessage.Complete();
}
catch(MessageLockLostException)
{
//le verrou a expiré
}
catch(Exception)//autre exception
{
//autre erreur : on abandonne le message
brokeredMessage.Abandon();
}
finally
{
//on arrête la task de renouvellement du verrou
brokeredMessageRenewCancellationTokenSource.Cancel();
}
```
Et voilà ! <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Clignement d'œil" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_31C051F0.png">

Julien

