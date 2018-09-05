---
layout: post
title: "[ASP.NET MVC] Fournisseur de cache de sortie personnalisé"
date: 2011-09-13 7:18:00 +02:00
categories:
- ASP.NET MVC
author: "Julien Corioland"
identifier: 18772593-2789-40c3-a398-c273d5133983
redirect_from:
  - /archives/aspnet-mvc-fournisseur-de-cache-de-sortie-personnalise
  - /Archives/aspnet-mvc-fournisseur-de-cache-de-sortie-personnalise
---

ASP.NET 4.0 apporte de nombreuses nouveautés et notamment la possibilité de développer son propre fournisseur de cache de sortie.

Pour des besoins bien particulier, un de mes clients avait besoin de générer un grand nombre de page dynamique : le problème étant que ces pages faisaient appel à énormément de ressources et donc la génération prenait un temps fou.

Pour résoudre le problème, j’ai proposé de faire un fournisseur de cache de sortie personnalisé permettant de stocker la réponse HTTP sur le FileSystem afin de pouvoir la renvoyer plus rapidement. Cette solution est assez intéressante car le cache reste disponible même si le pool d’application ASP.NET est coupé. En revanche, elle n’est pas optimale dans un cas multi-serveur avec un répartiteur de charge en amont. *Bref, à ne pas utiliser `les yeux fermés`* ![image](/images/aspnet-mvc-fournisseur-de-cache-de-sortie-personnalise/666d8c66-c848-4282-bca8-606428d382cb.jpg)

### Création du fournisseur de cache de sortie personnalisé

Pour créer un fournisseur de cache de sortie personnalisé, il suffit de dériver de la classe abstraite OutputCacheProvider et de redéfinir les méthodes suivantes :

```csharp
public class FileSystemOutputCacheProvider : OutputCacheProvider
{
public override object Add(string key, object entry, DateTime utcExpiry)
{
throw new NotImplementedException();
}

public override object Get(string key)
{
throw new NotImplementedException();
}

public override void Remove(string key)
{
throw new NotImplementedException();
}

public override void Set(string key, object entry, DateTime utcExpiry)
{
throw new NotImplementedException();
}
}
```

Voilà un bref descriptif des méthodes :

- **Add** : permet d’ajouter un élément identifié par une clé unique dans le cache

- **Get** : permet de récupérer un élément dans le cache

- **Remove** : permet de supprimer un élément du cache

- **Set** : permet de mettre à jour un élément dans le cache

Les éléments mis en cache seront de type `CachedItem` :

```csharp
[Serializable]
public class CachedItem
{
public object Item { get; set; }
public DateTime UtcExpiry { get; set; }
}
```

Nous utiliserons également deux méthodes permettant de générer un chemin de fichier à partir de la clé passé par le moteur ASP.NET MVC et de sauvegarder un CachedItem sur le disque :

```csharp
private void SaveCachedItem(CachedItem cachedItem, string filePath)
{
if (File.Exists(filePath))
File.Delete(filePath);

using (var stream = File.OpenWrite(filePath))
{
var binaryFormatter = new BinaryFormatter();
binaryFormatter.Serialize(stream, cachedItem);
}
}

private string GetFilePathFromKey(string key)
{
foreach (var invalidChar in Path.GetInvalidFileNameChars())
key = key.Replace(invalidChar, '_');

return Path.Combine(CacheDirectory, key);
}
```

*Méthode Add :*

```csharp
public override object Add(string key, object entry, DateTime utcExpiry)
{
string filePath = GetFilePathFromKey(key);
var cachedItem = GetCachedItem(filePath);

if (cachedItem != null && cachedItem.UtcExpiry.ToUniversalTime() <= DateTime.UtcNow)
{
Remove(key);
}
else if (cachedItem != null)
{
return cachedItem.Item;
}

SaveCachedItem(new CachedItem()
{
Item = entry,
UtcExpiry = utcExpiry
}, filePath);

return entry;
}
```

*Méthode Get :*

```csharp
public override object Get(string key)
{
string filePath = GetFilePathFromKey(key);
var cachedItem = GetCachedItem(filePath);
if (cachedItem != null)
{
if (cachedItem.UtcExpiry.ToUniversalTime() <= DateTime.UtcNow)
{
Remove(key);
}
else
{
return cachedItem.Item;
}
}

return null;
}
```

*Méthode Remove :*

```csharp
public override void Remove(string key)
{
string filePath = GetFilePathFromKey(key);
if (File.Exists(filePath))
File.Delete(filePath);
}
```

*Méthode Set :*

```csharp
public override void Set(string key, object entry, DateTime utcExpiry) {
string filePath = GetFilePathFromKey(key);
var cachedItem = new CachedItem() {Item = entry, UtcExpiry = utcExpiry};
SaveCachedItem(cachedItem, filePath);
}
```

### Enregistrement du fournisseur de cache de sortie personnalisé

L’enregistrement d’un fournisseur de cache de sortie se fait en deux étapes : dans le Web.config et dans le Global.asax.

*Fichier Web.config :*

```xml
<system.web>

<caching>
<outputCache>
<providers>
<add name="FileSystemOutputCacheProvider"
type="Samples.FileSystemOutputCacheProvider, Samples"/>
</providers>
</outputCache>
</caching>
```

*Global.asax :*

<font size="2">Il faut surcharger la méthode **GetOutputCacheProviderName** qui est en charge de retourner le fournisseur contextuel à la requête. Par exemple, si l’on souhaite utiliser le FileSystemCacheProvider que sur le contrôleur `Products`, on écrira :</font>

```csharp
public override string GetOutputCacheProviderName(HttpContext context)
{
if (context.Request.RawUrl.ToUpper().Contains("PRODUCTS/DETAILS")) {
return "FileSystemOutputCacheProvider";
}
return base.GetOutputCacheProviderName(context);
}
```

<br />

### Utilisation du fournisseur de cache de sortie personnalisé en ASP.NET MVC

Pour utiliser le nouveau fournisseur de cache, il suffit d’utiliser l’attribut "OutputCache` comme à son habitude. C’est le Framework ASP.NET qui se chargera d’instancier le bon fournisseur de cache au bon moment ![image](/images/aspnet-mvc-fournisseur-de-cache-de-sortie-personnalise/666d8c66-c848-4282-bca8-606428d382cb.jpg)

```csharp
public class ProductsController : Controller
{
[OutputCache(Duration = 3600, VaryByParam = "id")]
public ActionResult Details(int id)
{
//long operation
Thread.Sleep(2000);

ViewBag.ProductId = id;
return View();
}
}
```

NB : le fait de préciser ici le paramètre VaryByParam = `id` permet de faire en sorte que le moteur MVC génère une clé différente par id, est donc implicitement que le FileSystemOutputCacheProvider génère un fichier par id !

[Sources de l'exemple](http://juliencorioland.blob.core.windows.net/publicfiles/SampleOutputCacheProvider.zip)

A bientôt ![image](/images/aspnet-mvc-fournisseur-de-cache-de-sortie-personnalise/ded6ef76-614f-4147-b652-66bd87ce42a6.jpg)

