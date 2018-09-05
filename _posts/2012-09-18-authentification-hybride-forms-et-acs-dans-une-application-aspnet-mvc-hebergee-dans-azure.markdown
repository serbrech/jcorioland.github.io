---
layout: post
title: "Authentification Hybride Forms et ACS dans une application ASP.NET MVC hébergée dans Azure"
date: 2012-09-18 10:45:14 +02:00
categories:
- ASP.NET MVC
- Microsoft Azure
author: "Julien Corioland"
identifier: 812cb1c0-69dc-49ad-a4c8-24b514a9e71e
redirect_from:
  - /archives/authentification-hybride-forms-et-acs-dans-une-application-aspnet-mvc-hebergee-dans-azure
  - /Archives/authentification-hybride-forms-et-acs-dans-une-application-aspnet-mvc-hebergee-dans-azure
---

Pour un projet, j’ai du mettre en place un mode d’authentification hybride Forms / ACS (Access Control Service) au sein d’une même application ASP.NET MVC, hébergée dans Azure.

Pour la partie Forms Authentication, je me suis basé sur un custom membership provider qui travaille avec le stockage par table de Windows Azure. Vous pouvez également utiliser les Azure Storage providers disponibles sur [CodePlex](http://azureproviders.codeplex.com/) ou NuGet. Je ne détaillerai pas la configuration de ce type d’authentification qui a déjà été traité dans pas mal d’articles sur le net.

Je souhaitais absolument avoir un mécanisme d’authentification qui soit peu intrusif, et surtout éviter d’avoir à gérer deux types d’authentification techniquement distincts dans mon code (si je suis en ACS alors la déconnexion se fait comme cela, si je suis en Forms alors …). Du coup, j’ai décidé d’utiliser ACS uniquement comme mécanisme d’authentification et du coup comme `autorité` d’identification des mes utilisateurs.

Pour se faire, j’ai tout d’abord configuré mon espace de noms ACS sur le portail d’administration Windows Azure. Pour voir comment faire cela pas à pas, je vous invite à lire [ce billet sur le blog de Léo](http://blogs.codes-sources.com/leo/archive/2011/11/14/windows-azure-access-control-service.aspx).

Une fois ACS configuré, il est possible de gérer l’authentification via ACS (et donc Windows Identity Foundation) de plusieurs manières : la première, en utilisant le fichier Web.config et en configurant les bons HTTP module fournis avec le SDK WIF. Lors de la callback d’ACS (après authentification Windows Live, Facebook ou Yahoo…) les modules se chargeront automatiquement de récupérer l’identité de l’utilisateur ainsi que de retourner ses `claims` (propriétés qui identifient l’utilisateur, telles que son identifiant, son email, son nom… en fonction de ce qui a été configuré sur ACS).

L’autre solution consiste à ne pas utiliser la configuration automatique de WIF et de catcher `manuellement` la réponse d’ACS après que l’utilisateur soit authentifié. Pour cela, il suffit d’écrire un peu de code, mais rien de bien méchant !

Avant tout, il est nécessaire d’ajouter deux références au projet : <em>Microsoft.IdentityModel.dll</em> (disponible dans le GAC après avoir installé le SDK WIF) et <em>System.IdentityModel.dll</em>.

Il faut commencer par vérifier que la requête HTTP est bien issue d’une callback d’ACS. Pour se faire, on utilise le module **WSFederationAuthenticationModule** qui fournit des outils pour interpréter les réponses d’ACS en mode WS-Federation :

```csharp
WSFederationAuthenticationModule wsFedAuthModule = new WSFederationAuthenticationModule();
if (!wsFedAuthModule.IsSignInResponse(request))
throw new HttpException(403, "forbidden");
```

Ensuite, il est possible de récupérer la réponse sous la forme d’un objet de type **SignInResponseMessage** :

```csharp
var signInResponseMessage = wsFedAuthModule.GetSignInResponseMessage(System.Web.HttpContext.Current.Request);
```

La récupération des claims renvoyé par le STS (Secure Token Service) d’ACS se fait ensuite à l’aide du **WSFederationSerializer**, qui permet notamment de récupérer le secure token de la réponse :

```csharp
var serializationContext =
new WSTrustSerializationContext(SecurityTokenHandlerCollectionManager.CreateDefaultSecurityTokenHandlerCollectionManager());
var secureTokenResponse = new WSFederationSerializer().CreateResponse(signInResponseMessage, serializationContext);
```

La réponse qui est récupérée contient le token de sécurité renvoyé par ACS. Celui-ci contient les différentes informations relative à l’utilisateur et est chiffré. Il est nécessaire d’utiliser un **Saml2SecurityTokenHandler** afin d’accéder aux informations du token. Avant, il est nécessaire de récupérer l’emprunte du certificat utilisé par ACS, l’espace de nom ACS, ainsi que la liste des URIs autorisées à travailler avec ACS (en fonction de votre configuration, je vous renvoie vers l’article de Léo évoqué plus tôt) :

```csharp
ConfigurationBasedIssuerNameRegistry issuers = new ConfigurationBasedIssuerNameRegistry();
string certificateTumbprint = "<certificat tumbprint>";
var ascNamespace = "https://votrenamespace.accesscontrol.windows.net/";
issuers.AddTrustedIssuer(certificateTumbprint, ascNamespace);

Saml2SecurityTokenHandler tokenHandler = new Saml2SecurityTokenHandler
{
CertificateValidator = X509CertificateValidator.None
};

SecurityTokenHandlerConfiguration config = new SecurityTokenHandlerConfiguration
{
CertificateValidator = X509CertificateValidator.None,
IssuerNameRegistry = issuers
};

config.AudienceRestriction.AllowedAudienceUris.Add(new Uri("http://127.0.0.1"));
tokenHandler.Configuration = config;
```

Maintenant que le token handler est correctement configuré il ne reste qu’à utiliser un simple **XmlReader** pour lire le security token et en extraire la collection de `**ClaimsIdentity**` qu’il transporte :

```csharp
using (var reader = XmlReader.Create(new StringReader(secureTokenResponse.RequestedSecurityToken.SecurityTokenXml.OuterXml)))
{
SecurityToken token = tokenHandler.ReadToken(reader);
ClaimsIdentityCollection claimsIdentity = tokenHandler.ValidateToken(token);
}
```

Chaque **ClaimsIdentity** contenu dans la collection possède alors une collection Claims, retournant chacun des claim associé à l’utilisateur qui s’est connecté. Il ne reste qu’à implémenter votre propre logique pour créer le cookie de session de l’utilisateur (à l’aide de la méthode <em>SetAuthCookie</em> de la classe **FormsAuthentication**, par exemple).

Il est donc possible d’avoir un contrôle vraiment fin sur l’authentification d’un utilisateur via ACS, permettant ici d’inter-opérer simplement avec un autre mécanisme d’authentification !

A bientôt ![image](/images/authentification-hybride-forms-et-acs-dans-une-application-aspnet-mvc-hebergee-dans-azure/ba0b76a6-45ff-45a3-a9d1-28b12ac36d90.jpg)

