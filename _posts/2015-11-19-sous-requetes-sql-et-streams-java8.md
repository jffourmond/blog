---
layout: post
title: Simuler des sous requêtes SQL avec les streams de Java 8
tags: Java SQL Stream
---
Peut-on faire des requêtes de type SQL en utilisant les streams de Java 8 ? En utilisant des fonctions comme MIN ou MAX ? Si on manipule par exemple des instances de la classe Footballeur ci-dessous, comment fait-on pour trouver le joueur ayant marqué le plus de buts ?

{% highlight java %}
class Footballeur {
  String nom;
  Integer nbButs;
}
{% endhighlight %}

En SQL, c'est facile, on ferait une sous requête :
{% highlight sql %}
SELECT nom FROM footballeur
WHERE nbButs = (SELECT MAX(nbButs) FROM footballeur);
{% endhighlight %}

En Java, on trouve beaucoup d'exemples permettant de récupérer le plus grand élément d'une liste, 
mais moins pour récupérer les propriétés de l'objet associé. 
J'ai eu ce genre de problématique cette semaine, et après un long quart d'heure de recherches infructueuses, 
j'ai fini par me lancer dans le deep web des développeurs, c'est à dire la page 2 de Google. 
J'ai trouvé ma réponse sur ce site plein d'exemples intéressants : 
[Java 8 – Collection enhancements leveraging Lambda Expressions – or: How Java emulates SQL](https://technology.amis.nl/2013/10/05/java-8-collection-enhancements-leveraging-lambda-expressions-or-how-java-emulates-sql/)

La solution est de passer à la fonction max() une lambda implémentant l'interface Comparator. 
Dans notre cas, ça donne donc :

{% highlight java %}
  Footballeur meilleurButeur = getFootballeurs().stream().
    		max((f1, f2) -> f1.nbButs - f2.nbButs);
{% endhighlight %}  

Tout simplement !

Le code source de l'exemple et son test unitaire sont disponibles sur [GitHub](https://github.com/jffourmond/java8-streams).

