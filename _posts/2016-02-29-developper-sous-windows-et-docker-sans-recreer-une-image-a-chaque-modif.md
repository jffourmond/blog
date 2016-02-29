---
layout: post
title: "[Docker pour les nuls] Développer sous Windows sans recréer une image à chaque modif"
tags: Docker DevOps Linux Windows
---
Utiliser la même image Docker (serveur + appli) sur tous ses environnements, c'est bien. 
Mais en phase de développement, recréer une image à chaque modif du code, c'est long. 
Pour éviter ça, il est possible d'ajouter ses sources au démarrage du conteneur Docker plutôt qu'au build de l'image. 

## Sous Linux, c'est facile

Sous Linux, c'est facile grâce à l'option -v qui sert à monter un volume de données. 
Exemple avec un conteneur embarquant un serveur Apache :

{% highlight bash %}
docker run -p 8000:80 -v ~jeff/boulot/angular1-es6/:/usr/local/apache2/htdocs/ -it httpd:2.4
{% endhighlight %}

Si on modifie un fichier HTML et qu'on actualise son navigateur à l'URL http://localhost:8000, on verra tout de suite les changements.

## Sous Windows, ça se complique

A ce jour, Docker tourne ne tourne que sous Linux. Il faut donc passer par une machine virtuelle (VM) Linux quand on est sous Windows. 
Cette couche de complexité supplémentaire est en partie masquée par le Docker Quickstart Terminal 
qui s'occupe de démarrer la VM via VirtualBox. 
Mais pour recharger les sources à chaud sur son conteneur Docker, il va falloir un peu de configuration. 
En effet, comme Docker tourne sur la VM, il ne voit pas nos sources qui, elles, se trouvent sous Windows. 
Il va donc falloir faire transiter ses sources de Windows au conteneur Docker en passant par la VM gérée par VirtualBox. 
Personnellement, je trouve que ce n'est pas évident, et je vais donc illustrer ce post avec quelques captures d'écran.

# Configuration

La configuration se fait en 3 étapes. Dans les captures d'écran ci-dessous, 
on prendra l'exemple d'une appli front nommée angular1-es6 tournant sous Apache, 
et dont les sources sont dans un dossier Windows D:\jeff\boulot\angular1-es6. On n'utilise pas de Dockerfile dans cet exemple.

## 1. Dans VirtualBox

Lancer le Docker Quickstart Terminal, puis VirtualBox (ces deux outils peuvent être installés via la [Docker Toolbox](https://www.docker.com/products/docker-toolbox)). 
Dans la configuration de la VM « default » créée automatiquement dans VirtualBox, cliquer sur « Dossiers partagés » et ajouter un dossier partagé. 
Le chemin est celui du dossier Windows contenant les sources. On peut choisir n'importe quel nom (ici : es6).
Laisser les cases cochées par défaut (Montage automatique, Configuration permanente).

![Configuration de VirtualBox]({{ site.url }}/assets/img/virtualbox.png)

## 2. Dans la VM Linux

Il va falloir monter le dossier partagé que l'on vient d'ajouter dans VirtualBox. 

D'abord, ouvrir un terminal et taper la commande suivante : 
{% highlight bash %}
docker-machine ssh default
{% endhighlight %}
Cette commande va nous connecter à la VM qui fait tourner Docker. L'OS de la VM étant Ubuntu, nous allons ensuite utiliser sudo.

Créer un dossier à l'endroit de son choix. Exemple :
{% highlight bash %}
sudo mkdir /vmWorkDir 
{% endhighlight %}
Puis monter les sources avec la commande mount. Ici, il faut bien faire correspondre le nom du volume à monter avec le nom du dossier qu'on a partagé dans VirtualBox (ici : es6) : 
{% highlight bash %}
sudo mount -t vboxsf es6 /vmWorkDir
{% endhighlight %}


![Configuration de la VM Linux]({{ site.url }}/assets/img/vm.png)
<center><i>Il faudrait penser à mettre à jour l'ascii art...</i></center>

## 3. Dans le Docker Quickstart Terminal
Maintenant que le lien est fait entre Windows et la VM, on peut utiliser lancer son conteneur Docker comme sous Linux, ou presque. 
On utilise toujours l'option -v pour monter ses sources, mais le dossier qu'on va passer en paramètre est le dossier monté sur la VM (ex : /vmWorkDir), pas celui de Windows.

{% highlight bash %}
docker run -p 8000:80 -v /vmWorkDir/:/usr/local/apache2/htdocs/ -it httpd:2.4
{% endhighlight %}

![La commande docker à taper]({{ site.url }}/assets/img/quick-terminal.png)

Et voilà, désormais chaque modif des sources dans le dossier Windows D:\jeff\boulot\angular1-es6 sera propagée dans la VM Linux dans le dossier /vmWorkDir, puis dans le conteneur Docker dans le dossier /user/local/apache2/htdocs. 

Au final, le cheminement des sources ressemble à ça : 
![Configuration de VirtualBox]({{ site.url }}/assets/img/cheminement.png)

# Test

On peut tester en ouvrant un navigateur, non pas sur localhost, mais sur l'IP de la VM qui est par défaut 192.168.99.100.

![Test dans un navigateur]({{ site.url }}/assets/img/navigateur.png)

# Références

* [How to mount a VirtualBox shared folder? (serverfault)](http://serverfault.com/questions/674974/how-to-mount-a-virtualbox-shared-folder)
* [Utiliser Docker pour son environnement de développement (Raphaël Gonçalves)](http://www.raphael-goncalves.fr/blog/utiliser-docker-pour-son-environnement-de-developpement)
* [Manage data in containers (doc officielle Docker)](https://docs.docker.com/engine/userguide/containers/dockervolumes/)
