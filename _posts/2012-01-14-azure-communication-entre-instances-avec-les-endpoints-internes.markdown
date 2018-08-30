---
layout: post
title: "[Azure] Communication entre instances avec les endpoints internes"
date: 2012-01-14 14:58:00 +01:00
categories: ASP.NET MVC, Windows Azure
author: "Julien Corioland"
identifier: db719690-423d-4332-b24e-4ee5d25685f1
redirect_from:
  - /archives/azure-communication-entre-instances-avec-les-endpoints-internes
  - /Archives/azure-communication-entre-instances-avec-les-endpoints-internes
---

Il y a quelques temps j’ai posté deux billets décrivant des techniques pour réaliser de l’invalidation de cache sur plusieurs instances d’un rôle Windows Azure :

- [Invalidation de settings ou de cache à l’aide d’une clé de configuration](http://www.juliencorioland.net/Archives/azure-invalidation-de-settings-ou-de-cache-a-l-aide-d-une-cle-de-configuration)
- [Multicast dans une application hébergée sur Azure avec AppFabric ServiceBus et NetEventRelayBinding](http://www.juliencorioland.net/Archives/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding)

Dans ce billet, je souhaiterai vous présenter une autre manière de communiquer entres différentes instances d’un même rôle, en utilisant les Endpoints internes.

Les endpoints internes sont des points d’accès sur lesquels chaque instance est en capacité, par exemple, d’hoster un service WCF. Il existe plusieurs avantages dans l’utilisation de ceux-ci :

- Ils sont internes, donc inaccessibles depuis l’extérieur
- Il est possible de communiquer via un binding en net.tcp
- Leur utilisation n’entraîne pas de surcoût sur votre facture Windows Azure

### *Création d’un endpoint interne*

Pour ajouter un endpoint interne à un rôle Windows Azure, rendez-vous dans les propriétés de celui-ci dans votre projet Visual Studio, puis accédez à l’onglet `Endpoints` :

![image](/images/azure-communication-entre-instances-avec-les-endpoints-internes/f4f8676f-261f-41a2-a522-cd3be6276604.jpg)

Il suffit de cliquer sur le bouton Add Endpoint, de le nommer, de mettre son type à Internal, de choisir le protocole et  enfin de définir le port privé sur lequel le endpoint sera accessible :

![image](/images/azure-communication-entre-instances-avec-les-endpoints-internes/8daa9fab-8db5-480b-b485-1e9b33c955c9.jpg)

Toutes les instances du rôles exposeront alors ce endpoint automatiquement !

###

### *Héberger un service WCF sur un endpoint interne*

Il est possible de faire en sorte qu’un service WCF soit hébergé au démarrage de chaque instance. Celui-ci pourra être appelé par n’importe quelle autre instance du rôle. Par exemple, dans le cas d’un scénario d’invalidation de cache, une instance de rôle doit être capable de notifier toutes les autres de réinitialiser leur cache local. Pour cela, il suffit de créer un canal de communication via WCF entre les instances.

Dans le cas présent, le contrat de service WCF est très simple :

```csharp
[ServiceContract]
public interface ICacheInvalidationService
{
[OperationContract]
void InvalideCache();
}
```

Son implémentation aussi :

```csharp
public class CacheInvalidationService : ICacheInvalidationService
{
public void InvalideCache()
{
//invalidation du cache local
}
}
```

Le démarrage du service WCF se fait dans la méthode **OnStart** du WebRole. En premier lieu, il faut récupérer l’adresse du endpoint interne qui a été créé :

```csharp
if (RoleEnvironment.CurrentRoleInstance.InstanceEndpoints
.Any(ip => ip.Key == "CacheInvalidationEndpoint"))
{
IPEndPoint endpoint = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints
.First(ip => ip.Key == "CacheInvalidationEndpoint").Value.IPEndpoint;

}
```

Ensuite, l’hébergement du service WCF se fait très simplement à l’aide d’un **ServiceHost** :

```csharp
public class WebRole : RoleEntryPoint
{
private ServiceHost _serviceHost;

public override bool OnStart()
{
if (RoleEnvironment.CurrentRoleInstance.InstanceEndpoints
.Any(ip => ip.Key == "CacheInvalidationEndpoint"))
{
IPEndPoint endpoint = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints
.First(ip => ip.Key == "CacheInvalidationEndpoint").Value.IPEndpoint;

Uri baseAddress = new Uri(string.Format("net.tcp://{0}", endpoint));
_serviceHost = new ServiceHost(typeof(CacheInvalidationService), baseAddress);

NetTcpBinding binding = new NetTcpBinding(SecurityMode.None);
_serviceHost.AddServiceEndpoint(typeof(ICacheInvalidationService), binding, "CacheInvalidation");

try
{
_serviceHost.Open();
}
catch
{
Trace.WriteLine("Impossible de démarrer le service d'invalidation de cache");
}
}

return base.OnStart();
}
}
```

Il est également possible de surcharger la méthode **OnStop** du rôle pour arrêter le service WCF proprement :

```csharp
public override void OnStop()
{
if (_serviceHost != null)
{
try
{
_serviceHost.Close();
}
catch
{
_serviceHost.Abort();
}
}
base.OnStop();
}
```

A présent, toute instance de ce rôle expose un service WCF permettant l’invalidation de son cache local !

### Contacter toutes les instances

La dernière étape consiste à écrire le code qui permet d’appeler le service d’invalidation de cache sur toutes les instances du rôle.

Pour cela, il est nécessaire de récupérer tous les **IPEndPoint** qui portent le nom `CacheInvalidationEndpoint` :

```csharp
var ipEndpoints =
from instance in RoleEnvironment.CurrentRoleInstance.Role.Instances
from endpoint in instance.InstanceEndpoints
where endpoint.Key == "CacheInvalidationEndpoint"
select endpoint.Value.IPEndpoint;
```

Ensuite, il suffit de parcourir les endpoints pour créer un canal de communication vers chacun d’eux, à l’aide d’une **ChannelFactory** WCF :

```csharp
foreach (var ipEndPoint in ipEndpoints)
{
try
{
NetTcpBinding binding = new NetTcpBinding(SecurityMode.None);

ChannelFactory<ICacheInvalidationService> channelFactory =
new ChannelFactory<ICacheInvalidationService>(binding);

string uri = string.Format("net.tcp://{0}/CacheInvalidation", ipEndPoint);
EndpointAddress address = new EndpointAddress(uri);

ICacheInvalidationService service = channelFactory.CreateChannel(address);
if (service != null)
{
service.InvalideCache();
}
}
catch
{
//log exception
}
}
```

Et voilà, toutes les instances ont maintenant été notifiée de vider leur cache local !

Vraiment simple à mettre en place, et super utile !

Pour tester, récupérez le projet d’exemple et mettez un point d’arrêt dans la méthode **InvalidateCache** du service WCF. Lancez le rôle Azure (configuré pour exécuter deux instances) et appelez ensuite l’URL /Home/InvalidateCache sur l’application MVC. Vous verrez que Visual Studio s’arrête bien deux fois dans le service, une fois pour chaque instance du rôle ![image](/images/azure-communication-entre-instances-avec-les-endpoints-internes/a3056944-f256-4f62-8d8a-04f781bfc30a.jpg)

Le code source est disponible [ici](https://juliencorioland.blob.core.windows.net/publicfiles/ExempleEndpointInterne.zip).

A bientôt !

