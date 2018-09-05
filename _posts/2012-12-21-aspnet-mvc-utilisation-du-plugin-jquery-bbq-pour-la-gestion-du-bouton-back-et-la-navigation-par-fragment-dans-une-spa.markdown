---
layout: post
title: "[ASPNET MVC] Utilisation du plugin jQuery BBQ pour la gestion du bouton back et la navigation par fragment dans une SPA"
date: 2012-12-21 9:07:05 +01:00
categories:
- ASP.NET MVC
- Web Development
author: "Julien Corioland"
identifier: 14a43d10-551f-4342-8b4b-bcfc6aea0011
redirect_from:
  - /archives/aspnet-mvc-utilisation-du-plugin-jquery-bbq-pour-la-gestion-du-bouton-back-et-la-navigation-par-fragment-dans-une-spa
  - /Archives/aspnet-mvc-utilisation-du-plugin-jquery-bbq-pour-la-gestion-du-bouton-back-et-la-navigation-par-fragment-dans-une-spa
---

Lorsque l’on développe une Single Page Application (SPA) en JavaScript, il est souvent recommandé de mettre en place la gestion du bouton back pour l’historique de navigation, ainsi que de la navigation par fragment, permettant de conserver des URL propres à chaque ressource et permettant ainsi aux utilisateur d’utiliser la SPA comme n’importe qu’elle autre application.

### Comprendre la navigation par fragment

Il s’agit en fait des éléments que l’on rajoute à une URL de page, après le caractère `#`. On parle aussi d’ancre dans la page. Par exemple, le lien suivant [http://www.monsite.com/page1.html#menu](http://www.monsite.com/page1.html#menu), permet de rediriger l’utilisateur directement vers une section de le page.

Ce système existant dans HTML depuis de nombreuses années a été ré-exploité en JavaScript pour de la navigation par fragment. Par exemple, l’équivalent de la page [http://www.monsite.com/customers/32](http://www.monsite.com/customers/32), qui affiche le détail du client ayant 32 pour id, peut se retrouver sous la forme de [http://www.monsite.com/customers/32](http://www.monsite.com/customers/32) afin d’être exploité directement depuis du code JavaScript. Les éléments qui suivent le caractère `#` représente le `hash` de l’URL.

Il suffit donc d’avoir un mécanisme en JavaScript pour détecter les changements de hash et permettre à l’utilisateur de passer de ressource en ressource à l’aide des boutons précédent / suivant du navigateur, de mettre une ressource en favoris, d’envoyer une ressource par mail à un autre utilisateur, etc…

C’est ce que permet de faire le plugin jQuery BBQ : <a title="http://benalman.com/projects/jquery-bbq-plugin/" href="http://benalman.com/projects/jquery-bbq-plugin/">http://benalman.com/projects/jquery-bbq-plugin/</a>

### Mise en place de jQuery BBQ

Commencez par télécharger les sources du plugin [ici](http://github.com/cowboy/jquery-bbq/raw/v1.2.1/jquery.ba-bbq.js) ou [ici](http://github.com/cowboy/jquery-bbq/raw/v1.2.1/jquery.ba-bbq.js) (en version `minified`). Ajoutez ensuite le fichier à votre projet de développement et référencez le dans la/les page(s) dans la(les)quelle(s) vous souhaitez l’utiliser.

```js
<script src="/Scripts/jquery.ba-bbq.js"></script>
```
Une fois ceci fait, il est possible de s’abonner à un événement `hashchange` sur la window. Cet événement sera levé à chaque fois que le hash de l’URL sera modifié :

```js
<div id="menu">

- @Html.ActionLink("Clients", "Index", "Customers", null, new { @class = "menu-link" })

- @Html.ActionLink("Achats", "Index", "Orders", null, new { @class = "menu-link" })

<div id="contentDiv">

<div id="currentFragment">

@section scripts{
<script type="text/javascript">
$(document).ready(function () {
$(window).bind("hashchange", function (e) {
//récupération du fragment
var fragment = e.fragment;
$("#currentFragment").html("" + fragment + "

");
});
});
</script>
}
```
Dès lors, dès que le fragment change dans l’URL, le hash est affiché sur la page, dans le conteneur `currentFragment` :

![image](/images/aspnet-mvc-utilisation-du-plugin-jquery-bbq-pour-la-gestion-du-bouton-back-et-la-navigation-par-fragment-dans-une-spa/image_31DEB6F2.png)

A présent, il suffit de remplacer les liens directs des différentes sections de l’application par des liens fragmentés :

```js
$(".menu-link").each(function(index, item) {
var href = $(item).attr("href");
var newHref = thisUrl + "#" + href;
$(item).attr("href", newHref);
});
```
Et de charger le contenu représenter par le fragment lorsque le hash change :

```js
$(window).bind("hashchange", function (e) {
//récupération du fragment
var fragment = e.fragment;
$("#currentFragment").html("" + fragment + "

");

//chargement du contenu
$("#contentDiv").load(fragment);
});
```
Enfin, pour faire en sorte que le contenu soit également chargé lorsque l’utilisateur accède directement à votre application web via une URL fragmentée, il est nécessaire de forcer l’évènement `hashchange` sur le document ready :

```js
$(window).trigger("hashchange");
```
Et voilà, le tour est joué. Vous pouvez récupérer les sources de l’exemple [ici](http://juliencorioland.blob.core.windows.net/publicfiles/SampleOutputCacheProvider.zip).

Enjoy <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Clignement d'œil" src="https://juliencorioland.blob.core.windows.net/medias/wlEmoticon-winkingsmile_07AF090A.png">

Julien

