---
layout: post
title: Comprendre Docker
tags: Docker DevOps
---
Avant, on installait Tomcat (par exemple) en local, en dév, en intégration et en production, 
puis on buildait son application et on la passait d'un environnement à l'autre. 
Et de temps en temps, ce qui marchait en dév ne marchait pas en prod. 
Aujourd'hui on peut, grâce à Docker, livrer son application avec son environnement d'exécution (appli + Tomcat + Linux) 
rapidement sur le serveur de son choix, et ainsi réduire le risque d'avoir des différences entre les environnements.

Docker est donc une solution de virtualisation légère qui fait énormément parler d'elle depuis plus de 2 ans. 
On trouve donc logiquement sur le web une grande quantité d'articles à son sujet. 
Mais les concepts de Docker peuvent prendre un peu de temps à assimiler quand ils ne sont pas bien expliqués. 
C'est pourquoi je vous conseille la lecture de ces 2 articles, qui sont les plus clairs que j'ai trouvés :

[Docker - Présentation](http://blog2dev.blogspot.fr/2014/06/docker-presentation.html)<br/>
[Docker - Premiers pas](http://blog2dev.blogspot.fr/2014/06/docker-mise-en-pratique.html)

Et pour tester Docker, ceux qui sont sous Mac ou Windows (64 bits uniquement) pourront soit 
installer [VirtualBox et Vagrant](http://jffourmond.github.io/2015/08/20/docker-vagrant-virtualbox/), 
soit (plus simple), installer la toute récente [Docker Toolbox](https://www.docker.com/toolbox).



