---
layout: post
title: "SynCount, une question de design"
date: 2013-11-29 09:22
header-img: "img/design.jpg"
author: "poulpi"
---

Comme je l'ai dit dans le message précédent, je ne suis pas vraiment une bête en matière de design ... Pour autant j'aime 
bien les belles choses et cela va commencer par les applications que j'installe sur mon téléphone.

Donc au moment de faire SynCount, je me suis bien entendu posé la question de la réalisation d'une interface assez 
agréable à la vue. Comme les bonnes idées ne viennent jamais seules, j'ai commencé par étudier les applications que je trouvais *sexy*.

### Et alors ? Qu'est ce qui en ressort ?

Et bien, il en ressort qu'il y a pas mal d'apps très moches. Mais Google a fait un effort ces derniers temps en trouvant 
2 éléments de design que je trouve très bien :

- **L'interface Holo avec l'ActionBar et la navigation par "Tabs" :** l'ActionBar c'est cette barre en haut des application 
où vous aller trouver le nom de l'écran et des icônes à droite qui vont vous permettre de réaliser les actions principales. 
Et bien c'est géré en natif depuis Android 4.0+, et ça tombe bien je ne veut pas m'intérésser aux versions antérieures d'Android 
(GingerBread est à moins de 30% de parts de marché au moment où j'écris ces lignes).

{:.center}
![Oui je l'ai piqué sur internet, et alors ? T'es toujours fier de ce que tu fais toi ? Hein ?](/img/action_bar_tabs.png)

- **L'affichage par "cards" :** il s'agit du nouveau look des applis google, il y a des cartes partout ! Je prendrais l'exemple 
de l'interface de Google Play ou de Google Now. On met le contenu dans une jolie boite qu'on appelle "card" et on épure à fond.
Ça permet d'avoir des super listes avec des miniatures et des options tout en restant joli !

{:.center}
![Google, il est plus fort que toi](/img/gplay_cards.png)

Bon, en suivant ces deux patterns, ce n'est pas trop dur de faire une interface potable. Regardons un peu comment faire techniquement :

## L'actionBar et les Tabs

Ce qui est pratique avec ces composants, c'est qu'ils sont natifs à Android. Ainsi on a naturellement une ActionBar quand on 
débute un nouveau projet et le menu vient s'insérer à sa droite sous forme d'icônes sitôt qu'on indique lesquelles ont veut. 
Du coup, comme tout le monde a déjà une ActionBar, il s'agit de personnaliser la sienne !

### Comment faire ?

Alors pour le choix des icônes, il faut rester très classique parce qu'il faudrait choisir celles que l'utilisateur a l'habitude de voir et 
d'utiliser dans le reste de ses applications pour éviter qu'il ne soit perdu. Ainsi on ira en priorité puiser dans 
l'[ActionBar icon pack](http://developer.android.com/design/downloads/index.html).

Bon ok, c'est encore moins personalisé qu'avant du coup. Il ne nous reste plus qu'à jouer sur un élément : **la couleur** ! =)

### Ah mais comment on fait pour trouver un thème de couleur sympa à l’œil ?

Et bien personnellement je suis assez nul pour trouver des associations de couleur qui vont bien. Mais c'est pas très grave, internet
 est là pour ça ! Donc si vous cherchez une belle palette, [Kuler](https://kuler.adobe.com) est là pour vous.

Après quelques recherches dans la liste des palettes proposées par les membres on finit assez vite sur une palette sympatoche. 
Dans mon cas ce fut celle-ci : [PIE PARTY FOR ALL KULERIST!!!](https://kuler.adobe.com/PIE-PARTY-FOR-ALL-KULERIST!!!-color-theme-1895756/) (no comment, j'aimais bien le titre ...).

Une fois qu'on a les couleurs, il suffit de les rentrer dans ce superbe [outil](http://jgilfelt.github.io/android-actionbarstylegenerator/) 
et on obtient les fichiers à mettre dans le répertoire "/res/" de notre projet, et TADAAA !

### Les Tabs, kézaquo ?

Alors si vous avez vu une des captures que j'ai déjà posté, vous avez pu voir que j'ai mis des Tabs dans la navigation. C'est une extension 
de l'ActionBar et il y a un tas de tutoriaux sur le net pour vous montrer comment faire (comment vous croyez que j'ai appris à le faire :p). 
Toujours est il que ça va de paire avec l'utilisation d'un [ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html) 
et donc de Fragments (spoilers) !

## Les Cards

Bon alors maintenant qu'on a une superbe ActionBar et des Tabs, il faudrait mettre des Cards pour avoir un peu de contenu. Alors malheureusement 
les Cards ne sont pas (encore ?) une brique de base d'Android. Google se les garde pour ses applis.

### Oh, mais il faut tout coder à la main, c'est super long/compliqué !

Hehe, heureusement quelqu'un l'a déjà fait ! Il existe une librairie qui s'appelle [CardsLib](https://github.com/gabrielemariotti/cardslib) et 
qui permet très facilement de construire ses propres Card et de les mettre dans une liste. Par contre c'est compatible uniquement sur du Android >4.0+.
Donc ça c'est le coup de grâce pour la compatibilité avec Gingerbread :(

Cette librairie est assez jeune (la première version utilisable doit avoir 2 mois) mais bien pratique et permet du coup de faire une belle UI 
même en étant mauvais :p

Dans mon application j'ai dû en coder 7 :

- **EventCard** : Carte qui affiche un événement.
- **SpendingCard** : Carte qui affiche une dépense
- **FriendCard** : Carte qui affiche le nom d'un ami
- **CumulCard** : Carte qui affiche le cumul des dépenses
- **BalanceCard** : Carte qui affiche le solde de tous les participants par rapport aux autres
- **SingleLineCard** : Carte qui permet d'afficher la dépense moyenne et la somme de toutes les dépenses
- **SolutionCard** : Carte qui affiche un plan de remboursement

## L'icône / Logo

Alors pour faire une appli Android, il lui faut une icône et un logo. Et je suis vraiment une quiche en conception graphique ... Alors 
j'ai décidé de partir sur des motifs assez simples en vectoriel ([Inkscape](http://inkscape.org/?lang=fr) est votre ami pour ceux qui connaissent).

Pour le motif, j'ai décidé de prendre un nuage parce qu'on envoie ses dépenses dans le cloud pour les synchroniser. Et j'aime bien les nuages.
 Pour trouver un nuage en 
svg : [cloud icon svg](https://www.google.fr/search?q=nuage+icon+svg) dans Google Images et paf on prend un nuage. Easy.

Ensuite on met les couleur de notre (magnifique) palette Kuler qu'on a choisi. Et enfin tu mets les initiales de ton application pour essayer de donner du 
cachet au tout. Dans le cas particulier, c'est SC que j'ai transformé en $€ avec la super police Calibri =).

Au final on obtient ça :

{:.center}
![I'm sexy and I know it ...](/img/sc_logo.png)

Bon je fais un appel aux graphistes si vous avez un truc mieux à proposer, moi j'ai déjà atteint mes limites :p

Dans un prochain épisode on verra comment faire pour synchroniser des dépenses dans le cloud !

