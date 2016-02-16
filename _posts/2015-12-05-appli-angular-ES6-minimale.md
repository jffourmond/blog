---
layout: post
title: Coder une application minimale en Angular 1 et ECMAScript 2015, sans outil de build
tags: ECMAScript ES6 JavaScript Angular Traceur
---
Le standard ECMAScript 2015 (ou ECMAScript 6 ou ES6) définissant la nouvelle version de JavaScript a été adopté en juin, 
mais il faudra encore quelques temps avant qu'il soit implémenté par tous les navigateurs. En attendant, 
on peut utiliser un compilateur comme Babel ou Traceur pour compiler son code ES6 en JavaScript. C'est même 
fortement conseillé par le [Technology Radar](https://www.thoughtworks.com/radar/languages-and-frameworks/ecmascript-6) 
paru le mois dernier.

Récemment au boulot, on m'a donc demandé d'ajouter Babel au build Grunt de notre application AngularJS 1.4, 
puis de migrer un contrôleur JavaScript simple vers ES6 pour valider que la compilation par Babel fonctionnait bien. 
Ça n'a pas posé de problème particulier, mais je me suis demandé si c'était aussi facile de migrer 
toute une application en ES6, et j'ai donc commencé à le faire sur une application perso.

###Babel ou Traceur ?

J'ai d'abord essayé d'utiliser Babel comme un polyfill, pour ne pas avoir à ajouter un outil de build à mon appli toute simple. C'est facile d'inclure le script Babel.js depuis son HTML, mais Babel avait besoin d'autres dépendances et je n'avais pas envie d'ajouter je-ne-sais-combien de fichiers à la main, ni d'utiliser NPM. Je me suis donc tourné vers Traceur, le compilateur de Google, qui n'a pas bonne presse mais qui tient en 2 fichiers. Avec Traceur, il suffit donc d'ajouter les 2 lignes suivantes à son HTML pour pouvoir l'utiliser :

{% highlight html %}
<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
{% endhighlight %}

Ensuite, j'ai migré mon tout code pour avoir des classes et des modules ES6, et bien sûr, ça n'a pas marché du 1er coup. 
Ça n'a même pas marché du tout. On trouve pas mal de posts de blogs sur le sujet, 
mais les exemples sont souvent inutilement compliqués. Au bout d'un certain temps, 
j'ai tout recommencé depuis zéro en partant sur un simple Hello World.

##Ce qui change dans le code

Au final, il y a juste quelques trucs à savoir :

- En plus de Traceur, il faut inclure le chargeur de modules [SystemJS](https://github.com/systemjs/systemjs).

- Il faut inclure le code de son application avec <span style="font-family: Courier New">type="module"</span> 
(et non pas <span style="font-family: Courier New">type="application/javascript"</span>). 
Exemple : 
 
{% highlight html %}
<script type="module" src="app.js"></script>
{% endhighlight %}

- Il faut utiliser la syntaxe "Controller as" introduite en Angular 1.2 
(mais il n'est pas obligatoire de configurer des routes).
Exemple (fichier index.html) : 

{% highlight html %}
<div ng-controller="HelloController as ctrl">
    <button ng-click="ctrl.sayHello()" >
        Say Hello !
    </button>
    {% raw %}{{ctrl.text}} {%endraw %}
</div>
{% endhighlight %}

Le contrôleur étant défini comme suit (fichier HelloController.js) :

{% highlight javascript %}
'use strict';

export default class HelloController {

     constructor(){
       this.text = '';
     }
 
     sayHello(){
       this.text += 'Hello ! ';
     }
}
{% endhighlight %}

Pour savoir comment migrer ses contrôleurs, je conseille ce post qui explique bien la transformation en classes étape par étape : 
[Awesome Angular Controllers with ES6: 6 Easy Steps!](http://essenceofcode.com/2015/08/21/awesome-angular-controllers-with-es6-6-easy-steps/)

- Angular doit être _boostrapé_ depuis le code (pas possible d'utiliser ng-app, les modules étant chargés de façon asynchrone)
Exemple (fichier app.js) : 

{% highlight javascript %}
'use strict';

import HelloController from './HelloController.js';

let applicationName = 'helloApp';
angular.module(applicationName, []).controller('HelloController', HelloController);
angular.bootstrap(document, [applicationName]);
{% endhighlight %}

Et c'est tout ! Les 3 fichiers mentionnés sont disponibles sur [GitHub](https://github.com/jffourmond/angular1-es6). 

Bien sûr, la compilation à la volée dégrade les perfs et il ne faut pas faire ça sur une grosse appli !
