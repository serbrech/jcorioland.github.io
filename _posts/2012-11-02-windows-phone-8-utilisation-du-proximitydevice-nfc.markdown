---
layout: post
title: "[Windows Phone 8] Utilisation du ProximityDevice NFC"
date: 2012-11-02 2:04:36 +01:00
categories:
- Windows Phone
author: "Julien Corioland"
identifier: 946dbafa-b128-4a1f-b886-4cf5391733ad
redirect_from:
  - /archives/windows-phone-8-utilisation-du-proximitydevice-nfc
  - /Archives/windows-phone-8-utilisation-du-proximitydevice-nfc
---

Windows Phone 8 permet de mettre en place de la communication NFC entre devices, en mode `tap`. Pour commencer à découvrir les APIs, je vous propose un scénario ultra simple : une application Windows Phone 8 ayant deux pages. La première est capable de publier un message via NFC, la seconde est capable de recevoir un message via NFC. Lorsque l’on `tape` deux téléphones entre eux celui sur la page de publication envoi un message qui est reçu sur l’autre téléphone.

### Publier un message via NFC

Pour commencer, il est nécessaire de récupérer une instance de la classe **ProximityDevice**, défini dans <em>Windows.Networking.Proximity</em>.

```csharp
public partial class Publish : PhoneApplicationPage
{
ProximityDevice _proximityDevice;

public Publish()
{
InitializeComponent();
_proximityDevice = ProximityDevice.GetDefault();
}
}
```
A partir du moment ou vous récupérez une instance, c’est que le NFC est supporté sur le device !

Pour être en capacité de publier un message, on utilise la méthode **PublishMessage**, à laquelle on précise un type de message et le contenu du message :

```csharp
private long _messageId = 0;

protected override void OnNavigatedTo(NavigationEventArgs e)
{
if (_proximityDevice != null)//NFC is supported
{
_messageId = _proximityDevice.PublishMessage("Windows.SampleNFC", "Hello dude, Windows Phone 8 Rocks !");
}

base.OnNavigatedTo(e);
}
```
A partir de là, l’application enverra le message à n’importe qu’elle autre application qui écoute ce type particulier de message, ici `<em>Windows.SampleNFC</em>`. Le type de message indique le protocole utilisé (avant le `.`) et le sous type de message. Lorsque l’on utilise la méthode **PublishMessage**, le protocole est obligatoirement `<em>Windows</em>`.

La méthode **PublishMessage** renvoie un id. Celui-ci est très important car votre application ne peut supprimer qu’un seul type de message à la fois. Aussi, avant de pouvoir publier un autre type de message, il faut stopper la publication, à l’aide de la méthode **StopPublishingMessage **:

```csharp
protected override void OnNavigatedFrom(NavigationEventArgs e)
{
if (_proximityDevice != null)
{
_proximityDevice.StopPublishingMessage(_messageId);
}
base.OnNavigatedFrom(e);
}
```
Voilà concernant la partie publication.

### Souscrire à un message NFC

La souscription à un message NFC est assez similaire, puisqu’elle utilise également un **ProximityDevice**, que l’on récupère avec la méthode statique <em>GetDefault</em>, comme dans la partie précédente. Ensuite, on utilise la méthode SubscribeForMessage en précisant le type de message qu’on vont recevoir :

```csharp
protected override void OnNavigatedTo(NavigationEventArgs e)
{
if (_proximityDevice != null)
{
_subscriptionId = _proximityDevice.SubscribeForMessage("Windows.SampleNFC", OnMessageReceived);
}
base.OnNavigatedTo(e);
}
```
Comme pour la publication, on obtient un id de souscription qu’il est nécessaire d’utiliser pour arrêter d’écouter ce type de message :

```csharp
protected override void OnNavigatedFrom(NavigationEventArgs e)
{
if (_proximityDevice != null)
{
_proximityDevice.StopSubscribingForMessage(_subscriptionId);
}
base.OnNavigatedFrom(e);
}
```
Et voilà le handler appelé lors de la réception d’un message :

```csharp
private void OnMessageReceived(ProximityDevice sender, ProximityMessage message)
{
if (message != null)
{
Dispatcher.BeginInvoke(() =>
{
MessageBox.Show(message.DataAsString);
});
}
}
```
Le tour est joué. Voilà ce que ça donne avec deux téléphones :

<div id="scid:5737277B-5D6D-4f48-ABFC-DD9C333F4C5D:d2c09f3e-eb93-4b9a-807c-55bd9f8767f0" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px"><object width="448" height="252"><param name="movie" value="http://www.youtube.com/v/LdK1tR9ARdM?hl=en&hd=1"></param><embed src="http://www.youtube.com/v/LdK1tR9ARdM?hl=en&hd=1" type="application/x-shockwave-flash" width="448" height="252"></embed></object>
A+

Julien

