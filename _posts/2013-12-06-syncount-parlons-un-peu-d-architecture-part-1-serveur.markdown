---
layout: post
title: "SynCount, parlons un peu d\'architecture - Serveur"
date: 2013-12-06 15:28
header-img: "img/architecture.jpg"
author: "poulpi"
---

Dans mes deux billets précédents j'ai abordé le placement fonctionnel de SynCount et j'ai parlé de son design que je voulais 
être un des points différenciant par rapport à des applications existantes bien implantées.

Mais un autre point essentiel dans mon positionnement c'est la synchronisation temps réel de SynCount et son fonctionnement 
aussi bien en mode Hors Ligne que connecté.

Pour tout ça il a fallu aborder pas mal de points et je vais en parler dans cet article.

## Introduction

La synchronisation c'est plutôt tendu, alors j'ai cherché des grosses briques déjà faites. Et si on gagne énormément en qualité et en 
temps de dév en prenant des bouts déjà fait, il faut quand même prendre du temps pour :

- Identifier les bons bouts de code qui vont être assez modulables pour faire ce qu'on veut tout en restant assez simple et performant
- S'arracher les cheveux à faire fonctionner toutes ces librairies ensemble sans perdre toutes ses dents en mordant son clavier

Donc ce qui est exposé ici est une architecture qui marche mais il n'est pas dit que ce soit la meilleure ^^

## De quoi avons nous besoin pour synchroniser des comptes partagés ?

Et bien il faut des clients (les téléphones des clients) et un/des serveur(s) pour enregistrer tout ce qui se passe sur les smartphones.

### Un serveur ? Quelles caractéristiques ?

Alors on peut choisir tout un tas de critères quand on choisi le bon serveur mais ce qui m'a parût important c'est d'avoir quelque chose
 qui respecte ces caractéristiques :

- Pas cher : Je suis pas riche et la monétisation de mon service n'est pas du tout assuré. Comme la location d'un serveur est un coût 
fixe, mieux vaut en prendre un pas cher.
- Evolutif : sait-on jamais si mon application marche et a besoin d'un boost serveur (la meilleure chose qui pourrait m'arriver parce 
que je serais super riche \o/), il faut que je sois capable de monter facilement en puissance.
- Permettant de faire tourner n'importe quoi dessus (donc pas du Google App Engine :( )
- Disponible quasiment tout le temps : ça serais con qu'il se coupe de temps en temps ...

**Mon choix :** Une instance Amazon EC2. C'est une machine virtuelle qui tourne dans un des datacenters d'Amazon, qui n'est pas trop 
chère. La plus petite (que j'ai pour l'instant) coûte 8€ par mois et est offerte la première année.

Et surtout c'est très facilement upgradeable ! Donc si j'ai besoin de plus, je monte en puissance en 3 clics !

### Bon et sur le serveur il faut mettre quoi ?

Alors il va falloir coder une petite application qui va recevoir des enregistrements assez divers (Amis, Transactions, Comptes, etc.) 
et qui va être capable de les mettre à disposition !

Bon là je dois avouer que j'ai un peu craqué sur la techno : j'ai choisi [Node.js](http://nodejs.org/)

Pourquoi ? Parce que c'est cool. Et c'est très orienté I/O temps réel. Et parait que c'est léger (quand c'est lourd faut 
un plus gros serveur, et ça coûte plus cher :( ). Par contre c'est en JavaScript et je suis pas vraiment un expert :p

Bon et à ça il va falloir coller une petite base de données pour enregistrer toutes ces dépenses. Et bien j'ai choisi 
[MongoDB](http://www.mongodb.org/) qui est une base [NoSQL](http://fr.wikipedia.org/wiki/NoSQL).

En gros une base de données SQL c'est pleins de tableaux : genre une dépense va avoir une colonne "montant", une colonne "titre",
 etc. Et une base de donnée NoSQL ça va avoir que 2 colonne : une où on a le numéro de l'entrée du tableau, et l'autre où on a le contenu. Par contre du coup le contenu il faut le mettre en forme comme ça :


    {
      "__v": 0,
      "_id": "5297ac700b6adcc37800000c",
      "passkey": "84ujac",
      "status": "Pending",
      "title": "WE Amiens ",
      "id": "34555b11-031c-4e47-bf6c-6ca3875f25c0",
      "currency": "€",
      "modified": "2013-11-28T21:51:44.877Z",
      "created": "2013-11-28T20:49:52.104Z",
      "friends": [
        "5297ac700b6adcc378000007",
        "5297ac700b6adcc378000008",
        "5297ac700b6adcc378000009",
        "5297ac700b6adcc37800000a",
        "5297ac700b6adcc37800000b"
      ]
    }


Ah oui, c'est pas super jojo à manipuler ça :p

Mais il y a une superbe librairie pour NodeJS qui permet de faire tout ça : [Mongoose](http://mongoosejs.com/index.html). 
Ça permet de gérer ces écritures savantes en objets JavaScript et du coup tout devient plus facile. On peut faire des recherches 
dans la base de données et demander à "peupler" les données : par exemple là on voit que les "friends" c'est pas vraiment ça ... 
En fait les numéros ce sont les identifiants de structures "friend" qui sont dans la base de donnée. On peut demander au serveur 
d'aller chercher ces "friend" et de les afficher directement dans la structure "event" en une ligne, c'est #top#.

Pour info mon serveur fait au final moins de 500 lignes !

### Mais comment on va faire pour faire communiquer ton appli avec ce serveur ? Ils vont parler quoi comme langue entre eux pour se comprendre ?

Bon alors là, il va falloir utiliser un truc que j'ai découvert l'année dernière : les webservices. L'idée c'est qu'on appelle une URL 
(en envoyant des données ou pas) et le serveur répond en envoyant des données dans un format prédéfini (XML ou JSON le plus souvent) ou 
le serveur fait juste une action (genre supprimer un ami) et dit s'il a réussi ou pas.

Pour NodeJS, le plus facile c'est encore d'utiliser le super module [Express](http://expressjs.com/) qui simplifie la vie en 
permettant très facilement de comprendre quelle URL a été appelée et avec quelles données avec.

Pour renvoyer les données, il faut savoir que MongoDB renvoie directement du JSON donc c'est nickel, on fera comme ça !

Pour le formalisme des codes d'erreurs, d'envoi de données, etc on va utiliser les bases du protocole 
[HTTP](http://fr.wikipedia.org/wiki/Hypertext_Transfer_Protocol) avec des requêtes en GET, en POST et en DELETE.

Pour en savoir plus sur le sujet, il y a ce [type](http://coenraets.org/blog/2012/10/creating-a-rest-api-using-node-js-express-and-mongodb/) 
qui donne pas mal de code super sympa !

Donc on a :

- Une instance EC2 chez Amazon
- Un serveur NodeJS dessus (moins de 500 lignes) qui utilise Mongoose et Express.
- Une base de données MongoDB.
==> Un petit serveur répondant du JSON aux webservices qui vont bien !

Niveau de geekerie = ~75% (parce que c'est des technos en plein boom quand même :p).

La prochaine fois on va voir comment on code une application android pour faire tourner tout ça !