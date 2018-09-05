---
layout: post
title: "[Azure] Invalidation de settings ou de cache à l’aide d’une clé de configuration"
date: 2011-10-15 11:43:00 +02:00
categories:
- ASP.NET MVC
- Microsoft Azure
author: "Julien Corioland"
identifier: e4010563-0e3a-481e-9fc5-9a3c6852c01a
redirect_from:
  - /archives/azure-invalidation-de-settings-ou-de-cache-a-laide-dune-cle-de-configuration
  - /Archives/azure-invalidation-de-settings-ou-de-cache-a-laide-dune-cle-de-configuration
---

Dans ce post je vais vous expliquer une technique que j’ai utilisée pour invalider le cache / recharger les settings sur toutes les instances du rôle Web qui héberge mon blog (en vrai il n’y en a qu’une seule, mais je trouve l’astuce sympa ![image](/images/azure-invalidation-de-settings-ou-de-cache-a-laide-dune-cle-de-configuration/f484b684-078c-4f0d-9724-edfec42f8b06.jpg))

*Le constat est simple :* mes posts et les settings du blog (email, smtp, etc…) sont persistés dans le stockage table de Windows Azure. Afin d’éviter les aller-retours vers le stockage par table, je monte tout en cache mémoire (le nombre de transaction est facturé et en plus du coup ça envoi du lourd en terme de perf).

*Problème :* si je change un settings dans mon storage, comment est-ce que je demande à mon service de recharger les données ? Idem si pour une raison x ou y je souhaite vider le cache de posts, je dois avoir une solution. Si on se place dans un contexte Windows Azure multi-instances (cas très fréquent) c’est une couche de complexité supplémentaire qui se rajoute car il faut notifier chaque instance !

La solution la plus simple est d’utiliser un setting Windows Azure et de faire en sorte que vos instances écoutent les changements de configuration. Dès lors, il suffit de détecter le changement de la bonne clé de settings, `ConfigVersion` dans mon cas (ou `CléBidon` si vous voulez) et de reinitialiser vos différents services (ici mon service de cache et mon service de settings).

Tout cela se passe dans le fichier WebRole.cs du webRole, via l’événement Changed de RoleEnvironment :

```csharp
public class WebRole : RoleEntryPoint
{
public override bool OnStart()
{
// For information on handling configuration changes
// see the MSDN topic at http://go.microsoft.com/fwlink/?LinkId=166357.
RoleEnvironment.Changed += OnRoleEnvironmentChanged;
return base.OnStart();
}

private void OnRoleEnvironmentChanged(object sender, RoleEnvironmentChangedEventArgs e)
{
if (e.Changes.Any(change => change is RoleEnvironmentConfigurationSettingChange)) {
foreach (var change in e.Changes.OfType<RoleEnvironmentConfigurationSettingChange>()) {
if (change.ConfigurationSettingName == "ConfigVersion")
{
var cacheService = UnityRoot.Container.Resolve<ICacheService>();
cacheService.InvalidateCache();
var settingsManager = UnityRoot.Container.Resolve<ISettingsManager>();
settingsManager.Reload();
}
}
}
}
}
```

A bientôt ![image](/images/azure-invalidation-de-settings-ou-de-cache-a-laide-dune-cle-de-configuration/0db8278d-538f-4246-b4e3-bfc950a5381e.jpg)

<div style="padding-bottom: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; float: none; padding-top: 0px" id="scid:0767317B-992E-4b12-91E0-4F059A8CECA8:a99247b3-3e3e-448b-941e-ad29b50621c4" class="wlWriterEditableSmartContent">Technorati Tags: [windows](http://technorati.com/tags/windows),[windows](http://technorati.com/tags/windows),[windows](http://technorati.com/tags/windows)
