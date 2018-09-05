---
layout: post
title: "[ASP.NET MVC] Multicast dans une application hébergée sur Azure avec AppFabric ServiceBus et NetEventRelayBinding"
date: 2011-11-04 17:24:00 +01:00
categories:
- ASP.NET MVC
- Microsoft Azure
author: "Julien Corioland"
identifier: 59bacedc-f2c3-4c14-942c-4c897da258bf
redirect_from:
  - /archives/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding
  - /Archives/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding
---

Il y a quelques semaines j’ai posté une astuce pour invalider un cache mémoire sur un [changement de configuration d’instance Windows Azure](http://www.juliencorioland.net/Archives/azure-invalidation-de-settings-ou-de-cache-a-l-aide-d-une-cle-de-configuration). Bien que cette technique fonctionne, elle nécessite une intervention humaine, puisqu’à chaque fois que l’on souhaite invalider le cache, il faut modifier la configuration du service hébergé sur Azure.

Afin d’éviter de faire cela, je me suis penché sur l’utilisation d’AppFabric ServiceBus dans Azure et notamment du **NetEventRelayBinding** pour faire en sorte que lorsque je publie / supprime ou modifie un post, toutes les instances de mon blog soit notifiées et invalident leur cache mémoire.

Pour faire simple, j’expose un endpoint sur le service bus Azure, sur lequel toutes les instances vont s’abonner. Dès que l’une d’entre elle contact ce endpoint, toutes les autres sont notifiées.

Le bus des services permet de mettre en place un relai entre un client et un host de service WCF. Par exemple, il est possible d’avoir un service WCF qui soit hébergé sur un serveur IIS dans votre infrastructure et que ce service soit consommé depuis Azure ou une application totalement externe via le bus des services, évitant ainsi des problèmes de sécurité et de configuration de pare-feu.

L’autre scénario (mis en place ici) est le multicast ou le bus des services sert également de relai vers un host WCF, mais en mode multicast.

Pour mettre cela en place, il faut procéder en différentes étapes, décrites ci-après.

##

### 1. Activer un domaine sur le bus des services Windows Azure

Connectez-vous au portail d’administration Windows Azure sur [http://windows.azure.com](http://windows.azure.com) puis rendez-vous dans la section **Bus des services, Contrôle d’accès et Cache** via le menu de gauche :

![image](/images/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding/e0ac8082-83b9-49f9-9107-1e2353a402e5.jpg)

Sélectionnez ensuite **Bus des services** dans le noeud **AppFabric**, puis cliquez sur **Nouveau** dans le ruban. Une popup s’ouvre alors, vous permettant d’ajouter un nouveau domaine sur lequel vous pourrez exposer vos services via AppFabric :

![image](/images/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding/adbba4ff-15d3-4af7-9f39-f7ba042ce86b.jpg)

Choisissez votre espace de nom, la région dans laquelle exposer le bus des services, l’abonnement sur lequel vous souhaitez l’ajouter… puis valider la popup. Patientez jusqu’à ce que le Bus des services ajouté soit marqué comme `Actif` dans la liste.

![image](/images/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding/dea831f3-d634-42c2-b9bc-4d313d36684c.jpg)

### 2. Création du contrat de service et de l’implémentation du service

Comme AppFabric Service Bus sert de relais entre clients et host WCF, on utilise les mêmes mécanismes que WCF classique pour créer contrat de services et implémentations de services.

Dans mon cas, le contrat est ultra simple :

```csharp
[ServiceContract(Name = "IRelayCacheInvalidationEventContract", Namespace = "http://www.juliencorioland.net/Services/")]
public interface IRelayCacheInvalidationEventContract
{
[OperationContract(IsOneWay = true)]
void InvalidateCache();
}
```

<em>Attention toutefois à bien marquer toutes les **OperationContract** comme étant en mode `One-Way`, une condition nécessaire à l’utilisation du contrat sur le bus des services Azure.</em>

J’ai également prévu une interface pour représenter le canal de communication WCF qui sera utilisé pour invalider le cache :

```csharp
public interface IRelayCacheInvalidationEventChannel : IRelayCacheInvalidationEventContract, IClientChannel
{
}
```

A présent l’implémentation du service, une fois encore très simple ici puisqu’il s’agit d’implémenter l’interface **IRelayCacheInvalidationEventContract** :

```csharp
public class CacheInvalidator : IRelayCacheInvalidationEventContract
{
public void InvalidateCache()
{
ICacheService cacheService = ServiceLocator.Current.GetInstance<ICacheService>();
if(cacheService != null)
{
cacheService.InvalidateCache();
}
}
}
```

###

###

### 3. Ajout du code pour hoster le service dans le Global.asax du rôle web ASP.NET MVC

Dans cette étape, il s’agit d’ajouter le code de création de l’host WCF qui sera exécuté par toutes les instances lors du démarrage de l’application web.

Pour cela, il faut dans un premier temps récupérer 3 informations : le domaine du bus des services, le nom l’émetteur par défaut (issuer name) et la clé partagée (shared secret). Pour récupérer ces informations, sélectionnez votre bus des services dans l’interface d’administration et cliquez sur le bouton **Affichage** de la zone** Clé par défaut**, à droite. Une popup s’affiche alors :

![image](/images/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding/4769893e-2415-47b9-a93a-ac7e6c2e0ca5.jpg)

Personnellement, j’ai choisi de rajouter ces éléments au fichier de configuration du mon instance Azure, ce qui me permet de les récupérer de la sorte :

```csharp
string serviceNamespace = RoleEnvironment.GetConfigurationSettingValue("CacheInvalidationRelayDomainNamespace");
string issuerName = RoleEnvironment.GetConfigurationSettingValue("CacheInvalidationRelayIssuerName");
string issuerSecret = RoleEnvironment.GetConfigurationSettingValue("CacheInvalidationRelayIssuerSecret");
```

Ensuite, vous devez créer une **EndpointBehavior** pour identifier l’appel sur le bus des services. Cela s’effectue à l’aide d’un **TokenProvider** et d’une instance de la classe **TransportClientEndpointBehavior** :

```csharp
TokenProvider tokenProvider = TokenProvider.CreateSharedSecretTokenProvider(issuerName, issuerSecret);
TransportClientEndpointBehavior relayCredentials = new TransportClientEndpointBehavior(tokenProvider);
```

<em>Pour avoir accès à ces classes, il vous faut référencer la librairie Microsoft.ServiceBus.dll.</em>

A présent, utilisez l’api du service bus pour créer l’Uri qui hostera le relais vers le service :

```csharp
Uri relayAddress = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "/CacheInvalidation/");
```

Puis comme vous l’auriez fait en WCF, crééez le ServiceHost et ajoutez lui l’endpoint behavior comme cela :

```csharp
_serviceHost = new ServiceHost(typeof(CacheInvalidator), relayAddress);
foreach (var endpoint in _serviceHost.Description.Endpoints)
{
endpoint.Behaviors.Add(relayCredentials);
}
_serviceHost.Open();
```

###

###

<em>Remarquez que l’endpoint behavior est rajoutée sur tous les endpoints configurés pour le service CacheInvalidator.</em>

A présent, toutes les instances peuvent être atteintes via du multicast et donc pourront invalider leur cache mémoire !

### 4. Ajout du code pour récupérer un canal WCF et lancer l’invalidation de cache

Comme en WCF, on utilise un **ChannelFactory** pour récupérer un canal de communication. Pour pouvoir accéder au bus des services, il faut récupérer les mêmes informations que précédemment à savoir espace de noms, émetteur et clé partagé. Il faut également créer la behavior permettant d’authentifier l’appel

```csharp
string serviceNamespace = RoleEnvironment.GetConfigurationSettingValue("CacheInvalidationRelayDomainNamespace");
string issuerName = RoleEnvironment.GetConfigurationSettingValue("CacheInvalidationRelayIssuerName");
string issuerSecret = RoleEnvironment.GetConfigurationSettingValue("CacheInvalidationRelayIssuerSecret");

TokenProvider tokenProvider = TokenProvider.CreateSharedSecretTokenProvider(issuerName, issuerSecret);
TransportClientEndpointBehavior relayCredentials = new TransportClientEndpointBehavior(tokenProvider);
Uri relayAddress = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "/CacheInvalidation/");
```

A partir de là, il est possible de créer la **ChannelFactory** et de récupérer un canal de communication de type **IRelayCacheInvalidationEventChannel** pour invalider le cache :

```csharp
ChannelFactory<IRelayCacheInvalidationEventChannel> channelFactory =
new ChannelFactory<IRelayCacheInvalidationEventChannel>("CacheInvalidationEndpoint",
new EndpointAddress(relayAddress));
channelFactory.Endpoint.Behaviors.Add(relayCredentials);
_cacheInvalidationChannel = channelFactory.CreateChannel();

_cacheInvalidationChannel.Open();
_cacheInvalidationChannel.InvalidateCache();
```

Pensez à bien fermer le canal lorsque vous n’en n’avez plus besoin :

```csharp
_cacheInvalidationChannel.Close();
```

Le code étant en place, il ne reste plus qu’à configurer le service WCF, dans le Web.config du rôle web ASP.NET MVC !

###

### 5. Configuration du service WCF

Comme toujours la configuration d’un service WCF se passe dans la section system.serviceModel du fichier de configuration de l’application : ici le Web.config du rôle web !

Tout d’abord, il faut ajouter les extensions propres à l’utilisation du bus des services :

```xml
<system.serviceModel>
<extensions>
<bindingExtensions>
<add name="netEventRelayBinding" type="Microsoft.ServiceBus.Configuration.NetEventRelayBindingCollectionElement, Microsoft.ServiceBus, Version=1.5.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" />
</bindingExtensions>
</extensions>
</system.serviceModel>
```

Ensuite il faut ajouter la définition du binding netRelayEventBinding (toujours dans system.serviceModel) :

```xml
<bindings>
<netEventRelayBinding>
<binding name="default" />
</netEventRelayBinding>
</bindings>
```

Puis le endpoint du service :

```xml
<services>
<service name="MetroBlog.Bll.CacheInvalidator">
<endpoint name="CacheInvalidationEndpoint"
contract="MetroBlog.Interfaces.IRelayCacheInvalidationEventContract"
binding="netEventRelayBinding"
bindingConfiguration="default" />
</service>
</services>
```

Pour terminer par la configuration du client :

```xml
<client>
<endpoint name="CacheInvalidationEndpoint"
contract="MetroBlog.Interfaces.IRelayCacheInvalidationEventContract"
binding="netEventRelayBinding"
bindingConfiguration="default"
/>
</client>
```

Et voilà ! Si vous avez bien suivi toutes les étapes, vos instances sont toutes capables d’invalider leur cache grâce au multicast de AppFabric Service Bus !

A bientôt ![image](/images/aspnet-mvc-multicast-dans-une-application-hebergee-sur-azure-avec-appfabric-servicebus-et-neteventrelaybinding/bb93d462-5fc6-45b9-9bc2-2600d3c5b99f.jpg)

