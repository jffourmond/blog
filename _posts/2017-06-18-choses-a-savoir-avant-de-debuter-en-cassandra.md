---
layout: post
title: 5 choses que j'aurais aimé savoir avant de débuter en Cassandra
tags: NoSQL Cassandra 
---

Cassandra est une excellente base de données pour gérer de très gros volumes de données sans downtime. 
Mais c'est une technologie plus complexe qu'une base de données relationnelle ou qu'un MongoDB, 
et il est facile de faire des erreurs de conception quand on est habitué aux bases relationnelles. 
Avant de me mettre à Cassandra, j'avais regardé pas mal de vidéos sur le sujet et malgré ça, 
je suis passé à côté de certaines particularités. 
Voici 5 choses que j'aurais aimé savoir quand j'ai débuté sur le sujet : 

### 1. Le vocabulaire n'est pas clair

*Cluster, clustering key, row, partition, noeud, table, column family...* en découvrant Cassandra, vous allez rencontrer de nouveaux mots clés dont la signification n'est pas toujours très claire. 

Pour commencer, il faut savoir que le vocabulaire a évolué avec le temps. 
Vous trouverez sans doute d'anciens articles qui parlent de column families et d'autres plus récents qui parlent de tables. 
Une **column family** n'est rien d'autre qu'une **table**. C'est juste le nom qui a changé.

Une **partition** est un sous-ensemble des données d'une table, constitué de **rows** partageant une même clé (nommée **partition key**). 
Au niveau logique, une partition peut être vue comme un mini-shard, dont la distribution et la réplication dans le cluster 
sont gérées automatiquement par Cassandra.

Le mot cluster est associé à plusieurs concepts dans Cassandra. Au niveau de l'architecture, un **cluster** constitue 
l'ensemble des serveurs Cassandra, répartis en un ou plusieurs datacenters et appelés **nodes**. 
Au niveau d'une table, une **clustering key** permet d'ordonner et de filtrer les rows au sein d'une partition, sur un noeud donné. 
Rien à voir avec le cluster donc. 

### 2. INSERT = UPDATE

En CQL, les requêtes INSERT et UPDATE sont équivalentes. Si on fait deux INSERT sur la même primary key (partition key + clustering keys), les données du 1er INSERT seront écrasées. Inversement, si on exécute un UPDATE sur une primary key qui n'existe pas, les données seront insérées comme avec un INSERT. 

### 3. On ne peut pas renommer une table

Dans le cadre d'une migration de données, j'ai du créer une nouvelle table légèrement différente de la table d'origine. 
J'ai nommé cette nouvelle table *ma_table_v2*, en pensant la renommer en *ma_table* une fois que la migration serait terminée 
et que l'ancienne *ma_table* serait supprimée. Cette opération qu'on a l'habitude de faire en relationnel n'est pas possible avec Cassandra. 
Il faut donc toujours faire attention à toujours bien nommer ses tables. 

### 4. Les requêtes ne se font pas forcément dans l'ordre

Pour avoir de bonnes performances, on choisit généralement d'exécuter les écritures de manière asynchrone. 
Ce qui veut dire que les requêtes ne s'exécutent pas forcément dans l'ordre défini dans le code. 
Quand on exécute des DELETE suivis d'INSERT sur une même partition key, ça peut poser des problèmes : la requête de de DELETE peut être exécutée après l'INSERT, ce qui fait qu'on ne voit pas la ligne qu'on vient d'insérer. Ce cas est plutôt rare, ce qui le rend d'autant plus difficile à débugger. Mieux vaut donc le savoir avant.

### 5. Il faut toujours tester ses requêtes de lecture

En relationnel, on crée une table de manière à ce qu’elle reflète au mieux le concept métier à modéliser. Les requêtes d’interrogation de cette table peuvent être variées et ne sont pas toutes connues au moment de la conception. La table doit pouvoir évoluer et l’ajout de colonnes supplémentaires n’a généralement pas d’impact sur les requêtes existantes.

En Cassandra, tous les articles d’introduction mettent bien l’accent sur le fait qu’il faut modéliser ses tables en fonction des requêtes de lecture. Mais on ne dira jamais assez qu’il faut re-tester ses requêtes de lecture dès qu'on modifie une table, chaque modification pouvant invalider les requêtes de lecture existantes.

Quand on a l’habitude de faire du SQL, il faut penser à tenir compte des limitations du CQL. On peut par exemple considérer comme acquis certains mots clés comme OR ou IN. Mais il n’y a pas de OR en CQL, et le IN a des restrictions : il ne peut porter que sur la dernière clustering key, et il est à ce jour incompatible avec les tables qui ont des colonnes de type collection.

Il est donc indispensable de vérifier que toutes les requêtes de lecture fonctionnent quand on crée ou qu'on modifie une table, et ce même quand on pense qu’il n’y aura pas de problème. Les tests unitaires sont normalement là pour ça, mais avant de se lancer dans des modifications du code, il est préférable de faire une vérification rapide en exécutant ses requêtes à l’aide de cqlsh.
