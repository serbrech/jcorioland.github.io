---
layout: post
title: "[Windows Phone 8] Ecrire un tag NFC et lancement d’une application via NFC"
date: 2012-11-02 17:27:45 +01:00
categories: Windows Phone
author: "Julien Corioland"
identifier: 1ba78eff-2e97-4a0d-aa99-4b66a159dd49
redirect_from:
  - /archives/windows-phone-8-ecrire-un-tag-nfc-et-lancement-dune-application-via-nfc
  - /Archives/windows-phone-8-ecrire-un-tag-nfc-et-lancement-dune-application-via-nfc
---

Dans mon article précédent, je vous montrais comment utiliser l’API de proximity device pour échanger une information entre deux téléphones lors d’un `tap` NFC. Dans ce nouvel article, je vais vous montrer comment écrire du contenu dans un tag NFC afin de le réexploiter pour qu’il puisse fournir une URL ou encore qu’il permette de lancer une application !

### Ecriture d’un tag

L’écriture d’un tag se fait très simplement. Tout d’abord, Il faut récupérer une instance de ProximityDevice grace à la méthode statique GetDefault de cette même classe :

```csharp
public partial class MainPage : PhoneApplicationPage
{
private readonly ProximityDevice _proximityDevice;
// Constructor
public MainPage()
{
InitializeComponent();
_proximityDevice = ProximityDevice.GetDefault();
}
}
```
Une fois récupéré, le ProximityDevice permet d’écouter les messages NFC apportés par un tag sur lequel il est possible d’écrire. Pour celà, il suffit de souscrire aux messages de type `<em>Writeable</em>` :

```csharp
private long subId = 0;
protected override void OnNavigatedTo(NavigationEventArgs e)
{
if (_proximityDevice == null)
{
Dispatcher.BeginInvoke(() =>
{
MessageBox.Show("Le NFC n'est pas supporté sur ce téléphone", "Pas de NFC", MessageBoxButton.OK);
});
}
else
{
subId = _proximityDevice.SubscribeForMessage("WriteableTag", OnWriteableTagArrived);
}

base.OnNavigatedTo(e);
}
```
Dans le handler OnWriteableTagArrived, on récupère le ProximityDevice qui lève l’événement, et on peut alors écrire une information à l’aide de la méthode PublishBinaryMessage :

```csharp
private long pubId = 0;
private void OnWriteableTagArrived(ProximityDevice sender, ProximityMessage message)
{
var dataWriter = new DataWriter();
dataWriter.UnicodeEncoding = Windows.Storage.Streams.UnicodeEncoding.Utf16LE;
dataWriter.WriteString("http://www.juliencorioland.net");
pubId = sender.PublishBinaryMessage("WindowsUri:WriteTag", dataWriter.DetachBuffer());
}
```
Comme vous pouvez le constater, j’utilise ici le protocole WindowsUri:WriteTag, qui me permet d’écrire une information de type URL dans le tag NFC (ici il s’agit d’une carte de dev offerte par Nokia à la //build/), en l’occurrence ici l’adresse de mon blog. Du coup, lorsque je passe mon téléphone devant le tag NFC, celui ci affiche me propose d’afficher mon blog :

![image](/images/windows-phone-8-ecrire-un-tag-nfc-et-lancement-dune-application-via-nfc/image003_490040B8.jpg)

Lorsque j’accepte, le navigateur s’ouvre sur mon blog !

### Lancer son application à partir d’un tag NFC

Un des cas d’utilisation de NFC prévu par Microsoft sous Windows Phone 8 est la possibilité d’associer une application à un type de tag, permettant ainsi de lancer une application par contact NFC. Par exemple, vous pouvez imaginer une borne intéractive à l’entrée d’un magasin, sur laquelle vous poser votre téléphone qui lance alors l’application pour ce magasin en particulier, avec du contenu mis en avant !

Association de protocole

Dans un premier temps, il faut créer une association de protocole pour votre application. La première étape consiste à ajouter une extension dans la manifest de l’application :

```xml
<Extensions>
<Protocol Name="samplenfc" TaskID="_default" NavUriFragment="encodedLaunchUri=%s" />
</Extensions>
```
Ici, on associe le protocole `samplenfc:` à l’application (les deux autres paramètres sont les valeurs à renseigner obligatoirement, cf. <a title="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx#BKMK_Protocolassociations" href="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx#BKMK_Protocolassociations">http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx#BKMK_Protocolassociations</a>).

Ensuite, il faut créer un UriMapper capable de traiter l’association de protocole. Pour cela on créé une classe qui dérive de UriMapperBase :

```xml
public class AssociationUriMapper : UriMapperBase
{
public override Uri MapUri(Uri uri)
{
string url = HttpUtility.UrlDecode(uri.ToString());

if (url.Contains("samplenfc:MainPage"))
{
int paramIndex = url.IndexOf("source=") + 7;
string paramValue = url.Substring(paramIndex);

return new Uri("/MainPage.xaml?source=" + paramValue, UriKind.Relative);
}

return uri;
}
}
```
Le code est assez simple ici : je récupère l’URL `samplenfc:qqchose` et je la converti en URL XAML.

Ecriture du tag NFC

Pour l’écriture du tag NFC, cela se passe à peu près comme dans la première partie de l’article, sauf que cette fois, le protocole n’est plus http, mais samplenfc :

```xml
private long pubId = 0;
private void OnWriteableTagArrived(ProximityDevice sender, ProximityMessage message)
{
var dataWriter = new DataWriter();
dataWriter.UnicodeEncoding = Windows.Storage.Streams.UnicodeEncoding.Utf16LE;
//dataWriter.WriteString("http://www.juliencorioland.net");
//pubId = sender.PublishBinaryMessage("WindowsUri:WriteTag", dataWriter.DetachBuffer());

string appLauncher = string.Format(@"samplenfc:MainPage?source=NFCWakeUp");
dataWriter.WriteString(appLauncher);
pubId = sender.PublishBinaryMessage("WindowsUri:WriteTag", dataWriter.DetachBuffer());
}
```
Du coup, après avoir écrit mon tag sur la carte de développement, lorsque j’approche mon téléphone de celle-ci, j’obtiens bien le résultat attendu : le téléphone me propose de lancer mon application :

<div id="scid:5737277B-5D6D-4f48-ABFC-DD9C333F4C5D:1ce46c89-3a33-4e17-9e12-79104d45d30f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px"><object width="448" height="252"><param name="movie" value="http://www.youtube.com/v/1ORC-3a7KM8?hl=en&hd=1"></param><embed src="http://www.youtube.com/v/1ORC-3a7KM8?hl=en&hd=1" type="application/x-shockwave-flash" width="448" height="252"></embed></object>
NFC Rocks !

Julien

