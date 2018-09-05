---
layout: post
title: "[Entity Framework] Continuer à utiliser un EDMX avec EF 4.1"
date: 2011-09-19 10:50:00 +02:00
categories:
- Web Development
author: "Julien Corioland"
identifier: 99aeb394-2351-4cb0-b7ee-1f8707dd9335
redirect_from:
  - /archives/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41
  - /Archives/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41
---

Le titre de ce post vous a peut-être suppris puisque l’une des nouveautés la plus plébicitée de Entity Framework 4.1 est la possibilité de s’affranchir d’un modèle EDMX et d’adopter une stratégie Code First : les entités .NET permettent de générer le modèle de données SQL.

Pourtant, Entity Framework 4.1 apporte de nombreuses autres nouveautés, notamment en terme de gestion de l’état des entités, la mise à jour d’entités dans un contexte state-less (notamment), de requêtage… via des méthodes beaucoup plus simples que précédement.

Ce post à pour but d’expliquer comment utiliser Entity Framework 4.1 tout en continuant d’utiliser un EDMX et une stratégie Database First !

### Importer la librairie Entity Framework 4.1

Pour pouvoir utiliser EF 4.1, le plus simple est de référencer le package NuGet associé. Faites un clic droit sur votre projet Visual Studio puis cliqué sur `Manage NuGet packages…` :

![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/8140a49c-8268-4797-a240-543039f14abd.jpg)

Dans la fenêtre qui s’affiche, recherchez Entity Framework :

![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/aa0a2412-5933-4612-86e8-8de2cd6c5b92.jpg)

Cliquez sur `Install`, acceptez les conditions. La librairie EntityFramework.dll va être téléchargée et référencée dans le projet.

### Télécharger les templates de génération de code (T4) pour EF 4.1

Afin de générer le code propre à EF 4.1 (entités et DbContext) il est possible de télécharger un template d’item via le gestionnaire d’extension de Visual Studio. Placez-vous dans la galerie en ligne et recherchez DbContext :

![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/0c9bd5f7-54dd-4c6f-8d8f-efd5ae66f619.jpg)

Installez l’élément `ADO.NET C# DbContext Generator` (ou VB si vous le souhaitez ![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/c404a2ee-ac89-4d06-a38b-a49f6dcf656c.jpg)).

### Supprimer l’outil de génération de code de votre EDMX

Afin que le code ne soit plus généré par l’EDMX, éditez les propriétés de ce dernier et supprimer l’outil personnalisé :

![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/31e06249-66bd-419b-951b-366ece363d4d.jpg)

***Attention :*** ne faites pas celà dans un projet qui utilise les éléments de code généré par l’EDMX Entity Framework `classique`.

### Ajouter les templates au projet

Maintenant, ajoutez un nouvel item de type `ADO.NET C# DbContext Generator` au projet.

![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/9ab72d8f-27c5-4a79-8f36-a60e47518c30.jpg)

Identifiez la ligne de code suivante dans chacun des templates et modifiez la chaine $edmxInputFile$ par le chemin relatif vers votre fichier EDMX.

![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/6d088850-685e-40d8-9165-aab0eaed284f.jpg)

Dès lors, lorsque vous enregistrez les templates, du code devrait être généré : les entités et le DbContext.

![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/247b9026-a277-4d5e-b405-6a320aec9f9c.jpg)

Et voilà, vous pouvez utilisez le contexte généré pour attaquer votre base de données !

```csharp
using (var dbContext = new BlogContainer()) {
var posts = dbContext.Posts.Where(p => p.IsPublished).ToList();

return View(posts);
}
```

Après modification de l’EDMX, il suffit de regénérer les templates T4 pour les mettre à jour et de recompiler le tout ![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/c404a2ee-ac89-4d06-a38b-a49f6dcf656c.jpg)

A bientôt ![image](/images/entity-framework-continuer-a-utiliser-un-edmx-avec-ef-41/24d6543a-2ef6-4171-925d-1cd87f55ed7c.jpg)

