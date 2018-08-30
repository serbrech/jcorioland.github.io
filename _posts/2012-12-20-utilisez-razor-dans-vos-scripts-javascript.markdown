---
layout: post
title: "Utilisez Razor dans vos scripts JavaScript"
date: 2012-12-20 9:37:36 +01:00
categories: ASP.NET MVC, Javascript
author: "Julien Corioland"
identifier: fcb2de66-9428-4f5e-b53e-c0f9326dfbae
redirect_from:
  - /archives/utilisez-razor-dans-vos-scripts-javascript
  - /Archives/utilisez-razor-dans-vos-scripts-javascript
---

Dans certains cas, il peut être intéressant de pouvoir formater du JavaScript avec Razor, pour tirer profit des données du modèle directement dans vos scripts JavaScript.

Pour cela, il suffit d’utiliser la notation `<text></text>` comme ci-dessous :

```js
<script type="text/javascript">

//code JavaScript
var customers = new Array();

//code Razor/C#
@foreach(var customer in Model.Customers)
{
//code javascript + razor/c#
<text>
customers.push({
name: "@customer.Name",
firstname: "@customer.FirstName",
birth: "@customer.BirthDate"
});
</text>
}

$(document).ready(function(){
//ect...
});

</script>
```
Et voilà : simple, mais efficace !

A+

Julien

