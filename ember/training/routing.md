---
layout: ember-training
title: Formation Ember - Routing
permalink:  routing/
prev: ember/training/templates
next: ember/training/ember-data
---

## Routeur

Le routeur est un composant central d'[Ember][ember]. Loin de constituer une pièce rapportée venant compléter un framework existant il en est la pierre angulaire. 
La compréhension et la manipulation du **routeur** et des **routes** est donc capitale dans l'appréhension du développement avec [Ember][ember].

Le routeur [Ember][ember] est unique au sein d'une application, c'est lui qui déclare les différentes **routes** qui seront proposées au sein de celle-ci et qui définit, le cas
échéant, les chemins personnalisés, paramètres, routes imbriquées, etc.

Si l'on examine le routeur de notre application, tel que créé par [Ember CLI][ember-cli], il est vide puisque nous n'avons créé aucune route pour le moment. 

```javascript
// app/router.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {
});

export default Router;
```

Il n'y a pas grand chose à dire sur cet élément pour le moment. Tout juste peut-on noter la récupération, depuis le système de gestion et de configuration des environnements d'[Ember][ember],
d'un paramètre permettant de déterminer la manière dont seront construites ou résolues les URLs de notre application. Se reporter à la 
[documentation](http://emberjs.com/api/classes/Ember.Location.html) pour plus de détails.

Le fait que le routeur soit vide ne signifie pas pour autant qu'aucune route n'existe. Nous avons d'ailleurs pu constater que la route ``application`` (``/``) existait et était personnalisable.
[Ember][ember] génère en effet pour nous des [Routes implicites](#routes-implicites) que nous aborderons plus loin.

<div class="work no-answer">
  {% capture m %}
  {% raw %}
  
On souhaite désormais créer une nouvelle route pour l'affichage et la manipulation de notre liste de ``comics``
  
1. Utiliser le *scaffolding* d'[Ember CLI](http://www.ember-cli.com/) pour déclarer une nouvelle route dans le routeur de notre application.
 
    ```console
    $ ember generate route comics
    
    version: 1.13.8
    installing route
      identical app\routes\comics.js
      identical app\templates\comics.hbs
    updating router
      add route comics
    installing route-test
      identical tests\unit\routes\comics-test.js
    ```
    
    On remarque que plusieurs éléments ont été générés / modifiés :
    * le routeur (``app/router.js``), d'abord, qui déclare désormais notre nouvelle route :
       
        ```javascript
        Router.map(function() {
          this.route('comics');
        });
        ```
    * une nouvelle route ``app/routes/comics.js`` vide.
    
        ```javascript
        export default Ember.Route.extend({
        });
        ```
    * un nouveau template ``app/templates/comics.hbs`` qui ne contient qu'un ``{{outlet}}``. Nous y reviendrons plus tard. (cf. [Routes imbriquées](#routes-imbriquees))
    
        ```html
        {{outlet}}
        ```
    * un nouveau test unitaire ``tests/unit/routes/comics-test.js``. Celui-ci ne contient qu'une simple assertion permettant de vérifier l'existence de la route.
    
        ```javascript
        moduleFor('route:comics', 'Unit | Route | comics', {
          // Specify the other units that are required for this test.
          // needs: ['controller:foo']
        });
        
        test('it exists', function(assert) {
          var route = this.subject();
          assert.ok(route);
        });
        ```
    
        Mais nous reviendrons sur ce test et, plus généralement, sur les tests unitaires par la suite.
  
  {% endraw %}
  {% endcapture %}{{ m | markdownify }}
</div>

## Routes

Le routeur se contente donc de déclarer l'existence d'une route - et donc d'une URL. La logique de cette route et son comportement se trouvent implémentés au sein d'une instance de
``Ember.Route``. Par convention, celle-ci doit se trouver dans un fichier de même nom que celui définit dans le routeur (ici ``comics``) dans le répértoire ``app/routes``. A
noter que dans le cas de [Routes imbriquées](#routes-imbriquees), l'arborescence de ce répertoire suit l'imbrication déclarée dans le routeur.

La route est responsable : 

* du **rendu d'un template** (qu'il soit implicite ou explicite)
* de la **récupération et du chargement d'un modèle** (c'est à dire de la ou des données qui seront fournies à un template)
* de la gestion de l'ensemble des **actions** en lien avec le chargement, la mise à jour d'un modèle ou la **transition** vers une nouvelle route

C'est donc celle-ci qui sera notament chargée d'appeler le **backend** pour récupérer & envoyer des données et mettre ainsi les objets métier (modèle) à jour.

Mais c'est aussi la route qui met en place les différents templates qu'il est nécessaire d'affichier lorsque l'on accède à une URL et l'organisation des routes au sein du routeur et leur
imbrication préside donc à l'organisation des différents templates de notre application.

<div class="work">
  {% capture m %}
  {% raw %}
   
1. Copier le test d'acceptance [02-routing-test.js](https://github.com/bmeurant/ember-training/blob/master/tests/acceptance/02-routing-test.js) dans ``tests/acceptance``.

1. Renommer le test unitaire ``tests/unit/comics-test.js`` en ``tests/unit/02-routing-test.js``

1. Modifier le contenu du template ``app/templates/comics.hbs`` :
    * Déplacer l'affichage (template) et la gestion (route) de la liste de comics pour que cette liste soit gérée totalement au sein de la route ``/comics`` (en conservant
      le titre de premier niveau dans le template ``app/templates/application.hbs``)
    * Ajouter un sous-titre ``Comics list`` juste après l'ouverture de la ``<div class="comics">``
 
    **Test** : *Les modifications doivent permettre de rendre le test [02 - Routing - 01 - Should display second level title](https://github.com/bmeurant/ember-training/blob/master/tests/acceptance/02-routing-test.js#L87) passant.*

    > ```html
    > {{!-- app/templates/application.hbs --}}
    > <div class="container">
    > 
    >   <div class="page-header">
    >     <h1 id="title">Comic books library</h1>
    >   </div>
    > 
    >   {{outlet}}
    > 
    > </div>
    > ``` 
    > 
    > ```html
    > {{!-- app/templates/comics.hbs --}}
    > <div class="row">
    >   <div class="comics">
    >     <h2>Comics list</h2>
    >     <ul>
    >       {{#each model as |comic|}}
    >         <li class="{{if comic.scriptwriter 'comic-with-scriptwriter' 'comic-without-scriptwriter'}}">
    >           {{comic.title}} by {{if comic.scriptwriter comic.scriptwriter "unknown scriptwriter"}}
    >         </li>
    >       {{else}}
    >         Sorry, no comic found
    >       {{/each}}
    >     </ul>
    >   </div>
    > </div>
    > ```
    > 
    > ```javascript
    > // app/routes/application.js
    > export default Ember.Route.extend({
    > });
    > ```
    > 
    > ```javascript
    > // app/routes/comics.js
    > export default Ember.Route.extend({
    > 
    >   model: function () {
    >     // WARN : SOULD NOT BE DONE : We should not affect anything to windows but
    >     // for the exercice, we want to access to comics from console today
    >     window.comics = [{title: "BlackSad"}, {title: "Calvin and Hobbes", scriptwriter: "Bill Watterson"}];
    > 
    >     return window.comics;
    >   }
    > });
    > ```
 
  {% endraw %}
  {% endcapture %}{{ m | markdownify }}
</div>

La nouvelle route ``comics`` affiche désormais la liste de nos comics de la même manière qu'elle était affichée précédemment et est accessible
via l'URL ``/comics``. On note à ce propos que l'URL n'a pas eu à être définie puisque, par convention, [Ember][ember] rend accessible une route
à l'URL définie par son nom qualifié (nom de ses [ancètres](#routes-imbriquees) séparés par des `/` puis nom de la route). Si besoin, il est évidemment possible de
personnaliser l'URL via l'option ``path`` fournie lors de la définition de la route :

```javascript
Router.map(function() {
  this.route('comics', { path: '/livres' });
});
```

## Routes imbriquées

{% raw %}

On a pu remarquer que le template définit au niveau de l'application (``application.hbs``) était toujours affiché (titre principal),
en même temps que le template de la route ``comics``. Ceci est du au fait que la route ``comics`` est, comme toutes les routes d'une application 
[Ember][ember], imbriquée dans la route ``application``.

En effet il s'agit de la route de base de toute l'application. A la manière d'un conteneur, cette route permet classiquement de mettre en place
les éléments communs d'une application : *header*, *menu*, *footer*, ..., ainsi qu'un emplacement pour le contenu même de l'application : c'est
la fonction du *helper* ``{{outlet}}``.

Cette notion est au cœur d'[Ember][ember]. Lorsqu'une route est imbriquée dans une autre,
[Ember][ember] va rechercher les templates de ces deux routes et remplacer la zone `{{outlet}}` de la route mère
avec le rendu de la route fille. Ainsi de suite jusqu'à résolution complète de la route. Lors des transitions entre routes, les
zones des `{{outlet}}` concernées par le changement, **et seulement elles**, sont mises à jour.

Toute route fille (URL : ``mere/fille``) est déclarée dans le routeur en imbriquant sa définition dans celle de sa route mère (URL : ``mere``) 
de la façon suivante : 

```javascript
// app/router.js
...
Router.map(function() {
  this.route('mere', function() {
    this.route('fille');
  });
});
```

La route fille viendra alors se rendre dans la zone définie par l'``{{outlet}}`` de sa route mère, à la manière de poupées russes.

Par convention, les éléments constitutifs des routes filles (template, route, etc.) doivent être définies dans l'arborescence suivante :
``.../<route_mere>/<route_fille>.[hbs|js]``

{% endraw %}

<div class="work">
  {% capture m %}
  {% raw %}
   
1. Créer une route ``index`` accessible à l'URL `/comics/` affichant uniquement le texte *"Please select on comic book for detailled information"*
dans un paragraphe d'id ``no-selected-comic``.
 
    **Test** : *Les modifications doivent permettre de rendre le test [02 - Routing - 02 - Should display text on comics/index](https://github.com/bmeurant/ember-training/blob/master/tests/acceptance/02-routing-test.js#L87) passant.*
 
    **NB :** Dans notre cas, il n'est pas nécessaire de créer de fichier route (``app/routes/comics/index.js``) pour le moment puisque nous
    n'avons aucune logique à y intégrer. Ceci met en évidence les capacités de **génération d'objets** d'[Ember](http://emberjs.com/) déjà 
    évoquées dans le chapitre [Overview - Génération d'objets](../overview/#génération-d'objets). En effet, grâce aux conventions de nommage
    d'[Ember](http://emberjs.com/), le framework génère pour nous dynamiquement les objets de base nécessaires à l'éxécution d'une route.
    Il nous suffit ensuite de définir ces objet pour en fournir notre propre implémentation. En l'occurence,
    l'objet route pour `comics.index` sera généré pour nous.
    
    > ```javascript
    > // app/router.js
    > ...
    > Router.map(function() {
    >   this.route('comics', function() {
    >     this.route('index', {path: '/'});
    >   });
    > });
    > ```
    > 
    > ```html
    > {{!-- app/templates/comics.hbs --}}
    > <div class="row">
    >   <div class="comics">
    >     ...
    >   </div>
    >   {{outlet}}
    > </div>
    > ```
    > 
    > ```html
    > {{!-- app/templates/comics/index.hbs --}}
    > Please select on comic book for detailled information.
    > ```
    > 
    > On note la nécessité de l'``{{outlet}}`` dans la route mère ainsi que les arborescences et les noms utilisés. Enfin, on note également
    > la personalisation de l'URL via l'option `path`.
 
  {% endraw %}
  {% endcapture %}{{ m | markdownify }}
</div>

## Modèle

Comme évoqué plus haut, l'une des responsabilité principales d'une route consiste à assurer la récupération et la gestion d'un modèle (*model*). 
Au sens général un modèle est un objet métier contenant des propriétés. En [Ember][ember], il peut s'agir d'objets javascript natifs ou d'instances de
classes héritant de `Ember.Object`. Cependant, comme on a pu le constater [auparavant](../templates/#bindings), dans le cas où l'on fournit un objet javascript natif à 
[Ember][ember], celui-ci le transforme automatiquement en sous-classe d'`Ember.Object` de manière à être capable d'écouter les changements survenus sur
ce modèle. Principalement pour pouvoir être capable de mettre à jour les template via les *bindings*.

[Ember][ember] propose également une librairie, [Ember Data](https://github.com/emberjs/data), permettant de gérer les modèles, 
leurs propriétés ainsi que leurs relations de manière très poussée. Un peu à la manière d'un ORM. Nous y reviendrons dans un [chapitre ultérieur](../ember-data).

Les modèles peuvent être récupérés en mémoire mais son classiquement chargés depuis un *backend* via une **API REST**. Qu'ils soient ou non gérés au travers 
d'[Ember Data](https://github.com/emberjs/data).

Le rôle de la route consiste donc à récupérer un modèle en méomoire ou à distance. Ce modèle peut être un objet seul ou une collection d'objets et, une fois récupéré par
la route, il est transmis au template associé ce qui permet toutes les opérations de *binding*.
 
De nombreuses opérations et méthodes proposées par les routes [Ember][ember] sont donc relatives à la gestion du modèle :

* ``model()`` : Une des méthodes principale. 
     Cette fonction doit retourner un objet ou une collection d'objet. Elle est automatiquement appelée lorsque la route est activée.
  
     Le modèle ainsi récupéré (en synchrone ou en asynchrone) est ainsi transmis au template associé ainsi qu'aux éventuelles routes filles. La gestion de l'asynchronisme
     est effectuée pour nous par le framework en utilisant des promesses (*promises*). Manipuler un objet ou une collection mémoire ou le retour d'une requête à une API est donc
     strictement équivalent de ce point de vue.

     ```javascript
     model: function () {
       return [{title: "BlackSad"}, {title: "Calvin and Hobbes", scriptwriter: "Bill Watterson"}];
     }
     ```

* ``beforeModel()`` : Cette méthode est appelée au tout début de la résolution de la route et, comme son nom l'indique, avant la récupération du modèle vie la méthode
   ``model()``. 
   
     Elle est courament utilisée pour effectuer une vérification susceptible d'entrainer l'annulation de la transition en cours ou une redirection sans 
     que la résolution du modèle soit préalablement nécessaire. 
     
     Comme elle prend totalement en charge l'aspect asynchrone, cela lui permet d'être aussi très utile lorsqu'il 
     est nécessaire d'effectuer une opération asynchrone avant de pouvoir tenter de récupérer le modèle. En effet, le cycle d'exécution attendra automatiquement la résolution 
     de l'appel asynchrone avant d'exécuter la méthode ``model``.
   
     ```javascript
     beforeModel: function(transition) {
       ...
     }
     ```
   
* ``afterModel()`` : Cette méthode est appelée après la résolution du modèle et est utilisée régulièrement pour déclencher des transitions, des redirections ou toute sorte d'opération
  nécessitant la résolution préalable du modèle.
  
    ```javascript
    afterModel: function(model, transition) {
      ...
    }
    ```
 
Noter que dans ces deux dernières méthodes, un objet ``transition`` est fourni automatiquement. Cet objet permet d'agir sur la transaction en cours et notamment d'annuler
la transition courante (``transition.abort()``) ou de reprendre une transition précédemment annulée (``transition.retry()``).

<div class="work no-answer">
  {% capture m %}
  {% raw %}
  
1. Modifier la route ``comics`` pour passer sur un modèle plus complet : 
   * Créer une classe ``Comic`` étendant ``Ember.Object``
   * Définir les propriétés ``id``, ``title``, ``scriptwriter``, ``illustrator``, ``publisher``
   * Retourner une liste d'au moins deux instances de cette classe 
  
    ```javascript
    // app/routes/comics
    import Ember from 'ember';
    
    let Comic = Ember.Object.extend({
      id: 0,
      title: '',
      scriptwriter: '',
      illustrator: '',
      publisher: ''
    });
    
    let blackSad = Comic.create({
      id: 1,
      title: 'BlackSad',
      scriptwriter: 'Juan Diaz Canales',
      illustrator: 'Juanjo Guarnido',
      publisher: 'Dargaud'
    });
    
    let calvinAndHobbes = Comic.create({
      id: 2,
      title: 'Calvin and Hobbes',
      scriptwriter: 'Bill Watterson',
      illustrator: 'Bill Watterson',
      publisher: 'Andrews McMeel Publishing'
    });
    
    let akira = Comic.create({
      id: 3,
      title: 'Akira',
      scriptwriter: 'Katsuhiro Ôtomo',
      illustrator: 'Katsuhiro Ôtomo',
      publisher: 'Epic Comics'
    });
    
    let comics = [blackSad, calvinAndHobbes, akira]
    
    
    export default Ember.Route.extend({
      model() {
        return comics;
      }
    });
    ```
 
  {% endraw %}
  {% endcapture %}{{ m | markdownify }}
</div>
 
## Routes dynamiques

-> route comic/id

## Liens entre routes

## Routes implicites

On a évoqué plus haut le fait qu'il n'était pas nécessaire de déclarer explicitement la route ``application``, ni dans le routeur ni dans les routes ; 
et que seul le template était nécessaire. La route ``application`` est en effet une route implicite ; créée, connue et gérée nativement par
[Ember][ember]. C'est une route encore plus particulière car il s'agit de la route mère, du conteneur principal de toute application [Ember][ember].

Mais la route ``application`` n'est pas la seule route implicite gérée par [Ember][ember]. On peut le constater immédiatement par une simple manipulation :

<div class="work">
  {% capture m %}
  {% raw %}
 
1. Supprimer la déclaration de la route ``comics.index`` dans le routeur et constater l'absence de changement : la route existe toujours et sont fonctionnement
est strictement équivalent, le template ayant été conservé. 
 
  {% endraw %}
  {% endcapture %}{{ m | markdownify }}
</div>
 
D'autres routes encore sont créées comme le montre la capture ci-dessous pour notre application : 

<p class="text-center">
    <img src="/images/implicit-routes.png" alt="Routes implicites"/>
</p>

En noir apparaissent les routes déclarées explicitement (à l'exception près de la route ``application`` qui apparait en noir alors qu'elle est implicitement créée par
[Ember][ember]). En gris apparaissent l'ensemble des routes créées implicitement par [Ember][ember] et donc disponibles au sein de notre application sans que nous ayons
besoin de nous en occuper.

On constate que plusieurs types de routes sont implicitement créées lors de la déclaration explicite d'une de nos routes applicatives :

* index : 
* loading :
* error : 

-> expliquer model, api backend

A noter que le fonctionnement de ces routes et notamment la gestion de leur imbrication les unes par rapport aux autres fonctionnement rigoureusement de la même manière que toute
route déclarée explicitement. En particulier au niveau du *bubbling* : les évènement *error* et *loading*, notamment, vont remonter la structure d'imbrication des routes les unes
par rapport aux autres jusqu'à trouver une route en capacité de traiter cet évènement. Ce fonctionnement permet donc, au choix :

* de proposer une gestion des erreurs et du chargement commun à toute l'application au travers des routes ``application-loading`` et ``application-error``
* de spécialiser la gestion de ces évènement au niveau local, par exemple pour un sous ensemble fonctionnel devant gérer les mêmes types d'erreurs, etc.

En réalité, la bonne solution se trouve souvent dans un mix des deux avec la fourniture dun traitement général au niveau application spécialisé au besoin au niveau
des routes mères.

On note enfin que, grâce capacités de **génération d'objets** d'[Ember](http://emberjs.com/) déjà évoquées dans le chapitre [Overview - Génération d'objets](../overview/#génération-d'objets), 
les routes n'ont pas été les seuls objets à avoir été implicitement créés. En effet, on remarque que les contrôleurs et templates associés ont été également créés. Ils proposent une implémentation
par défaut totalement vide, bien entendu.

## Redirections et Transitions

rediriger / vers comics

beforeModel / after / redirect & routes imbriquées

transitions et interceptions (plus tard)

...
 
[handlebars]: http://handlebarsjs.com/
[ember-cli]: http://www.ember-cli.com/
[ember]: http://emberjs.com/