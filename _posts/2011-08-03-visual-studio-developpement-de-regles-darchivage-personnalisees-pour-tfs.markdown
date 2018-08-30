---
layout: post
title: "[Visual Studio] Développement de règles d'archivage personnalisées pour TFS"
date: 2011-08-03 13:30:00 +02:00
categories: Visual Studio, TFS 2010
author: "Julien Corioland"
identifier: 2100a636-e34b-4c1f-8688-5d350dff6881
redirect_from:
  - /archives/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs
  - /Archives/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs
---

Visual Studio permet d’exécuter du code avant chaque opération d’archivage afin de valider qu’un certain nombre de conditions soit réunies pour autoriser (ou non) le développeur à archiver son code : c’est ce qu’on appel des règles d’archivage (ou checkin policies).

Il existe déjà des règles `out of the box` qu’il est possible d’activer en faisant un clic droit sur le projet d’équipe pour lequel vous souhaitez activer une règle dans le Team Explorer, en sélectionnant l’entrée de menu **Team Project Settings** puis **Source Control…**

![image](/images/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs/bdbc514f-c1f2-4c55-bbcf-97b1707c748b.jpg)

Il est également possible de développer ses propres règles d’archivage. Pour cela, il vous faut installer le SDK Visual Studio (disponible via l’extension manager), si ce n’est pas déjà fait :

![image](/images/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs/09e4c37f-a9a7-4b50-8749-c15a3a9476d9.jpg)

Créez une projet de type Class Library et ajoutez-y les références suivantes :

- Microsoft.TeamFoundation.VersionControl.Client
- Microsoft.TeamFoundation.WorkItemTracking.Client

Une checkin policy est en fait une classe qui dérive de **PolicyBase **et qui soit sérialisable. Pour l’exemple, nous allons créer une règle vérifiant qu’un work item ait bien été associé au changeset (déjà existante) mais en plus que l’état du work item n’est pas `Closed` :

```csharp
[Serializable]
public class SampleCheckinPolicy : PolicyBase
{
public override string Description
{
get { return "Cette règle d'archivage vérifie qu'aucun work item associé au changeset n'est en état Closed."; }
}

public override bool Edit(IPolicyEditArgs policyEditArgs)
{
return true;
}

public override PolicyFailure[] Evaluate()
{
throw new NotImplementedException();
}

public override string Type
{
get { return "SampleCheckinPolicy"; }
}

public override string TypeDescription
{
get { return "Cette règle d'archivage vérifie qu'aucun work item associé au changeset n'est en état Closed."; }
}
}
```

**PolicyBase** est une classe abstraite, voilà les méthodes / propriété à redéfinir :

- **Description** / Type Description : information à propos de la règle (pour affichage dans les interfaces de VS)

- **Type** : nom de la règle (pour affichage également)

- **Edit** : permet d’afficher une fenêtre dans le cas où l’on souhaite que la règle soit configurable

- **Evaluate** : vérifie les conditions de la règles et retourne une liste d’erreur, si la règle n’est pas validée

La classe PolicyBase expose une propriété PendingChecking sur laquelle il est possible de récupérer les work items associées, les notes d’archivage… Il est donc possible de vérifier que des work items sont associés et qu’aucun n’est en état Closed :

```csharp
public override PolicyFailure[] Evaluate()
{
var failures = new List<PolicyFailure>();

if (PendingCheckin.WorkItems.CheckedWorkItems.Any())
{
foreach (var workItemCheckinInfo in PendingCheckin.WorkItems.CheckedWorkItems)
{
if(workItemCheckinInfo.WorkItem.State == "Closed")
{
string message = String.Format("Le workitem #{0} est en état Closed. Archivage impossible",
workItemCheckinInfo.WorkItem.Id);

failures.Add(new PolicyFailure(message,this));
}
}
}
else
{
failures.Add(new PolicyFailure("Aucun work item n'est associé au changeset !", this));
}

return failures.ToArray();
}
```

Tant que la méthode **Evaluate** retourne des erreurs, Visual Studio empêchera l’opération d’archivage (sauf si l’utilisateur contourne explicitement la règle).

La règle est prête, il ne reste plus qu’à la packager !

Pour cela ajoutez un projet d’extensibilité VSIX à la solution :

![image](/images/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs/01b3bc1d-607a-452b-a255-ece4bac497fd.jpg)

Ajoutez une référence vers la librairie contenant la règle à ce projet et vérifiez bien que la valeur `Copy Local` de cette référence est à vrai.

Il faut maintenant ajouter une fichier pkgdef au projet VSIX dans lequel sera renseigné l’enregistrement de la checkin policy dans le registre Windows. Ce fichier doit porter le nom du projet VSIX (SampleCheckinPolicyPackage.pkgdef, dans mon cas). Placez la propriété Include in VSIX à vrai dans les propriétés du fichier.

```
[$RootKey$\TeamFoundation\SourceControl\Checkin Policies]
"SampleCheckinPolicy"="$PackageFolder$\SampleCheckinPolicy.dll"
```

**Attention** : Le nom de la clé doit correspondre à celui de la dll !

Il ne reste plus qu’à lancer le projet VSIX en debug dans l’instance expérimentale de Visual Studio ou d’installer le VSIX produit lors de la génération du projet.

Rendez-vous alors dans les paramètres du source contrôles pour le projet d’équipe désiré : vous devriez pouvoir ajouter la règle :

![image](/images/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs/c4425d3c-1756-4319-8221-1eab1c37e878.jpg)

Tentez un archivage sans associer de Work Item ou en associant un Work Item en état `Closed` sur ce projet. Vous devriez voir apparaître les messages d’erreur de la règle dans la fenêtre Pending Changes :

![image](/images/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs/32c35f33-fcec-41ce-a9f2-5e675b7e0ab3.jpg)   ![image](/images/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs/32c35f33-fcec-41ce-a9f2-5e675b7e0ab3.jpg)

Téléchargez le code d’exemple : [SampleCheckinPolicy.zip](http://juliencorioland.blob.core.windows.net/publicfiles/SampleCheckinPolicy.zip)

A bientôt ![image](/images/visual-studio-developpement-de-regles-darchivage-personnalisees-pour-tfs/94ebabfb-a870-4864-ac59-4fe875949036.jpg)

