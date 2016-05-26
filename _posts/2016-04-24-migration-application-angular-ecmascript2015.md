---
layout: post
title: Migrer une application AngularJS 1.x de JavaScript vers ECMAScript 2015 avec npm
tags: ECMAScript ES2015 ES6 JavaScript Angular Browserify Babel Lebab Jasmine Gulp npm Docker
---
[Dans un article précédent](https://jffourmond.github.io/2015/12/05/appli-angular-ES6-minimale/), 
j'ai essayé de décrire comment créer une application AngularJS minimale en ECMAScript 2015 sans outil de build. 
La compilation ES2015 vers ES5 (JavaScript) se faisant à la volée, mon petit exemple mettait un certain temps à s'initialiser. 
Sur une vraie application, cette solution n'est pas envisageable et il est donc nécessaire de faire la compilation lors du build.

Dans cet article, nous allons nous attaquer à la migration d'une application Angular 1.X de JavaScript vers ES2015. 
Et comme cobaye, on utilisera mon appli [Freelance Impôt](http://www.freelance-impot.fr) écrite il y a quelques mois.

L'appli doit être iso-fonctionnelle une fois la migration terminée : 

- rendu identique
- exécution des tests Jasmine toujours possible depuis la page [http://freelance-impot.fr/specRunner.html](http://www.freelance-impot.fr/specRunner.html)
- debuggage possible du code source d'origine dans le navigateur (autrement dit : sourceMaps)

## Utilisation des nouveautés d'ECMAScript 2015

Les contrôleurs et les services deviennent des classes. Les dépendances sont passées en paramètre des constructeurs :

{% highlight javascript %} 
export default class CourbeController {

    constructor(calculService, nombreEntierFilter) {
        this.calculService = calculService;
        this.nombreEntierFilter = nombreEntierFilter;
    }
}    
{% endhighlight %} 

Le service et le filtre étant ici injectés, on ne manipule pas leur classe, et il n'est donc pas nécessaire de faire des import ici.
Les imports se font au moment de la définition du contrôleur dans Angular, dans le fichier principal de l'application (voir plus bas).

Les filtres peuvent être implémentés avec des *arrow functions*. Exemple : 
{% highlight javascript %} 
export default function nombreEntierFilter() {

    return (nombreAvecVirgules) => {
        return ...
    }
}
{% endhighlight %} 

Par contre je n'ai pas d'exemples de directives/components dans mon appli, ça sera donc à vous de chercher, désolé !

Pour le bootstrap de l'application et l'injection de dépendances, ça se passe donc dans le fichier principal de l'application : 
{% highlight javascript %} 
import nombreEntierFilter from './nombreEntierFilter';
import CalculService from './CalculService';
import CalculController from './CalculController';
import CourbeController from './CourbeController';
import ContactController from './ContactController';

const applicationName = 'freelance-impot';
const app = angular.module(applicationName, ['ngRoute', 'nvd3ChartDirectives']);
angular.module(applicationName).filter('nombreEntier', nombreEntierFilter);
angular.module(applicationName).service("CalculService", CalculService);
angular.module(applicationName).controller("CalculController", ["CalculService", CalculController]);
angular.module(applicationName).controller('CourbeController', ['CalculService', 'nombreEntierFilter', CourbeController]);
angular.module(applicationName).controller("ContactController", ContactController);

...

angular.bootstrap(document, [applicationName]);
{% endhighlight %} 

### Dans les tests

Dans les tests, on instancie les classes avec "new". Il faut donc les importer dans chaque spec.
{% highlight javascript %} 
import CalculService from './CalculService';
import CalculController from './CalculController';

describe("CalculController", calculControllerSpec);

function calculControllerSpec() {
    let ctrl;

    beforeEach(() => {
        const calculService = new CalculService();
        ctrl = new CalculController(calculService);
    });

    it('devrait être initialisé', () => {
        expect(ctrl.remuneration).toBe(0);
    });
}    
{% endhighlight %} 

### Lebab

Il y aura certainement des parties de votre code que vous ne penserez pas à réécrire à la sauce ES2015. 
Pour vous aider à utiliser toutes les nouvelles fonctionnalités d'ES2015, il y a [Lebab](http://lebab.io/) (Babel à l'envers). 
Lebab, c'est un outil en ligne de commande qui s'installe avec npm et qui réécrit les bouts de votre code ES5 ou ES2015 qui pourraient être améliorés. 
Une fois la migration de mon application terminée, j'ai passé Lebab sur mes fichiers source, et il restait encore pas mal de bouts de code à l'ancienne... 
('use strict' inutiles, constantes déclarées avec let au lieu de const, fonctions anonymes, concaténation de String...). 
Bref, Lebab est un compagnon idéal pour perdre ses vieux réflexes de JavaScript, en cas de migration comme en cas de nouveau projet.

## Ajout d'un outil de build

L'ancienne version de l'appli était déployée "nue", sans modification des fichiers source HTML, CSS et JS. 
Les tests étant exécutés depuis une page web, l'appli n'avait pas besoin d'outil de build. 
Pour la nouvelle version, j'avais donc le choix de l'outil à utiliser. 
Je savais juste que cette fois, j'allais utiliser Babel pour compiler (ou transpiler) le code ES2015 en ES5.

### Babel...ify, via Browserify

J'ai d'abord pensé que Babel suffirait, mais j'avais oublié le cas particulier des modules. 
Comme il n'y a pas d'équivalent aux modules ES2015 en JavaScript, Babel doit compiler les modules ES2015 vers un format tiers comme AMD ou CommonJS (le choix par défaut). 
OK, mais CommonJS ça ne marche pas dans le navigateur. Il faut donc utiliser un outil supplémentaire pour rendre les modules CommonJS générés compatibles avec le navigateur. 
Cet outil s'appelle Browserify. On pourrait aussi générer des modules AMD, mais il faudrait alors utiliser RequireJS, ce qui n'est sans doute pas mieux. 
Donc on compile le code ES2015 en JavaScript+CommonJS avec Babel, puis on rend le tout lisible par les navigateurs avec Browserify ? En réalité, ça ne se passe par dans cet ordre là. 
[Le point d'entrée, c'est Browserify](http://codeutopia.net/blog/2016/01/25/getting-started-with-npm-and-browserify-in-a-react-project/), 
Babel étant relégué à une étape de "transformation" pour Browserify nommée Babelify. 
Bon, et maintenant, qui va lancer Browserify ?

### Tentative avec Gulp

Travaillant chez mon client sur un projet buildé avec Gulp, je me suis d'abord tourné vers cet outil en pensant gagner du temps. 
Après des heures perdues à installer des tas de plugins Gulp (mais [pas le plugin gulp-browserify qui, lui, est blacklisté](https://github.com/gulpjs/plugins/issues/47)) 
et à copier/coller des bouts de code sans vraiment comprendre ce qu'ils faisaient, mon appli a fini par builder sous Windows. 
Mais en mettant mon build dans Docker, ça a donné ça (sans plus de précisions) : 
{% highlight bash %} 
gulp returned a non-zero code: 1 
{% endhighlight %}
Pfff... Je n'ai pas réussi à en savoir plus, et il ne m'en fallait pas plus : poubelle. De toute façon, [Gulp, c'est nul](https://www.tildedave.com/2015/01/07/i-find-gulp-extremely-frustrating.html). 
Franchement, pourquoi se compliquer la vie en ajoutant une couche de complexité superflue quand on peut juste utiliser npm ?

## NPM comme outil de build

Ça faisait un moment que j'avais en tête de tester npm comme outil de build, et dès que je l'ai utilisé, c'est devenu une évidence. 
Au lieu de devoir apprendre à utiliser des tas d'éphémères et obscurs plugins, on utilise simplement les commandes Linux de toujours. 
Le fichier de build est beaucoup plus concis qu'avec Gulp, et quand le build échoue, on a des messages d'erreur out-of-the-box, incroyable !

Donc pour résumer, notre build va lancer npm, qui va lancer Browserify, qui va créer un fichier unique contenant les classes ES2015 transformées en JavaScript par babelify 
(pas facile de citer des technos JS sans se planter dans la casse...). 

Et voici donc la description du build de l'application (extrait du fichier package.json lu par npm) : 

{% highlight json %} 
{
  "scripts": {
      "clean": "rm -rf dist && mkdir dist",
      "specs": "browserify src/specRunner.js -o dist/freelance-impot-specs.js -d -t [ babelify --presets [ es2015 ] ]",
      "scripts": "browserify src/app.js -o dist/freelance-impot.js -d -t [ babelify --presets [ es2015 ] ]",
      "build": "npm run clean && npm run scripts && npm run specs"
  }
}
{% endhighlight %}  

5 lignes donc, soit à peu près 10 fois moins que dans mon gulpfile.

L'option -d (debug) sert à dire à Browserify de générer les sourceMaps (qui permettent de débugger le code source d'origine malgré la compilation ES2015 -> ES5).

Pour builder l'appli, il suffit de taper la commande suivante dans son terminal : 
{% highlight bash %} 
npm run build
{% endhighlight %} 

Pour approfondir le sujet sur npm, je conseille la lecture de cet article : [How to Use npm as a Build Tool](http://blog.keithcirkel.co.uk/how-to-use-npm-as-a-build-tool/).

En réalité, dans ce build nous lançons 2 fois la commande browserify. Une fois pour créer le fichiers contenant l'application, et une deuxième fois pour créer un fichier de specs exécutable depuis le navigateur. 
Et ce qui est beau, c'est que nos tests Jasmine peuvent se passer d'Angular et de son module angular-mocks. 
En regardant le fichier specRunner.html, on voit qu'il n'y a plus aucune référence à Angular : 

{% highlight html %} 
<html>
  <head>
      <meta charset="utf-8">
      <title>Jasmine Spec Runner v2.3.4</title>

      <!-- Jasmine -->
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.3.4/jasmine.css">
      <script src="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.3.4/jasmine.js"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.3.4/jasmine-html.js"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.3.4/boot.js"></script>

      <!-- Freelance Impôt -->
      <script src="dist/freelance-impot-specs.js"></script>
  </head>
</html>
{% endhighlight %}

### Précisions sur Browserify

Une chose à savoir pour ceux qui n'ont jamais utilisé Browserify : la commande browserify prend 1 fichier en entrée et produit 1 fichier en sortie. 
Ça ne sert à rien d'essayer de concaténer ses fichiers ES2015 avant de les passer à Browserify. Celui-ci va aller chercher les fichiers dont l'appli a besoin en suivant les "import" ES2015. 
D'où l'existence du fichier specRunner.js, dont le seul but est de servir de point d'entrée à browserify pour générer le fichier exécuté lors des tests : 

{% highlight javascript %} 
import calculControllerSpec from './CalculController.spec';
import calculServiceSpec from './CalculService.spec';
import contactControllerSpec from './ContactController.spec';
import nombreEntierFilterSpec from './nombreEntierFilter.spec';
{% endhighlight %}

## Conclusion

Quelque soit l'outil de build que vous utilisez, il n'y a pas de raison de se priver des nouveautés d'ECMAScript 2015 qui rendront votre application bien plus maintenable. 
Et si vous démarrez un nouveau projet, c'est l'occasion de se débarrasser de Grunt et Gulp et d'utiliser npm à la place.

Les sources des deux versions de l'application sont disponibles sur GitHub : 

- [V1 (JavaScript)](https://github.com/jffourmond/freelance-impot/tree/1.0.0-2015-js)
- [V2 (ECMAScript 2015)](https://github.com/jffourmond/freelance-impot/tree/2.0.0-2015-es6)

## Références

- [Coder une application minimale en Angular 1 et ECMAScript 2015, sans outil de build](https://jffourmond.github.io/2015/12/05/appli-angular-ES6-minimale/)
- [Learn ES2015 (Babel)](http://babeljs.io/docs/learn-es2015/)
- [I Find Gulp.js Extremely Frustrating](https://www.tildedave.com/2015/01/07/i-find-gulp-extremely-frustrating.html)
- [How to Use npm as a Build Tool](http://blog.keithcirkel.co.uk/how-to-use-npm-as-a-build-tool/)
- [Getting started with npm and Browserify in a React project](http://codeutopia.net/blog/2016/01/25/getting-started-with-npm-and-browserify-in-a-react-project/)
- [ES6+ maintenant ! (vidéo)](https://youtu.be/KJzlllc7Jq8?list=PLCWW2DoKbvS9sUT2i2cS-3yG83P88eZt7)
