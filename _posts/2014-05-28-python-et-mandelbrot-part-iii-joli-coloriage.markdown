---
layout: post
title: "Python et Mandelbrot : Part III (Joli coloriage)"
date: 2014-05-28 00:34
header-img: "img/coloriage.jpg"
author: "poulpi"
---

## Dans les épisodes précédents ...

Bon alors où en est-on sur l'ensemble de Mandelbrot ? 

- Au début on a découvert python et on a tracé l'ensemble comme des gentils novices. Et c'était moche.
- Ensuite on a parlé d'optimisation et on a utilisé la connexité de l'ensemble pour éviter des calculs.  Et c'était toujours moche ...

Qu'est-ce qui reste à voir ? 

- Colorier ça de manière plus choupie. 
- Faire un antialiasing avec du sur-échantillonnage pour devenir super sexy (dans le prochain billet)
- Faite une version multi-thread pour faire tourner à fond son PC et arrêter d'attendre 30 ans pour une image (dans un prochain billet, je suis un maître du **teasing** dis-donc).

## Le coloriage, version BG

### Colorier l'ensemble de Mandelbrot de manière continue

Comme vous avez pu le voir dans mes billets précédents sur le sujet, le coloriage le plus basique consiste à associer une couleur par "itération d'échappement". En résumé on regardait le nombre d'itérations qu'il fallait avant que le module de la suite soit supérieur à 2 et on donnait associait ce nombre d'itérations au point du plan complexe qui avait servi de $z_{0}$. Ainsi on avait des points qui divergeaient très vite (1, 2 ou 3 itérations par exemple) et d'autres qui divergeaient bien plus tard (plus de 100 itérations).

Mais plus on s'approche de l'ensemble de Mandelbrot et plus les points divergent tard, par conséquent on a un grand nombre de couleurs différentes à la frontière de l'ensemble et c'est assez moche. De plus, on voit bien les changements de couleurs sur le graphique, et c'est pas très esthétique !

{:.center}
![La drogue, c'est mal.](/img/mandel1.png)

Donc il va falloir trouver mieux ! Pour cela on peut par exemple prendre la formule : $t + 1 - log( log( \|z\| ) / log( N ) )$ où t est le nombre d'itérations après lesquelles le point a fini par diverger (1, 2, 3, ..., 1000, etc.) et N est le critère qui permet de dire si le point diverge en vérifiant $\|z\|>N$. Si vous avez mieux, un petit commentaire ne serait pas de refus ^^

Comme on l'a vu plus tôt (ça fait quelques mois quand même)  $N=2$ suffit à dire qu'un point va diverger, mais ici on prendra $N=\sqrt(6)$ pour que ça soit un peu plus joli (et oui ça prends plus de temps du coup :p).

En utilisant cette formule on obtiendra un "adoucissement" des transitions entre les couleurs, et ça sera plus joli.

### Le coloriage linéaire

Maintenant qu'on a obtenu une formule permettant de ne plus avoir une matrice remplie de nombres entiers mais de nombres répartis de manière continue, il faut voir comment les colorier. Pour cela on va prendre 2 couleurs, en associer une à la valeur la plus basse de la matrice et l'autre à la valeur la plus haute. Et toutes les valeurs entre les deux vont être coloriée de manière linéaire en fonction de leur valeur.

Pour les gens un peu plus matheux, on va en fait faire 3 régression linéaires sur les trois composantes R, G et B des deux couleurs (celle associée à la valeur la plus basse et celle associée à la valeur la plus haute) et donc on va trouver 3 fonctions affines qui vont donner la valeur des composantes R, G et B en fonction de la valeur d'un point.

{:.center}
![J'ai vomi.](/img/mandel_lin_0aa_500.png)

Bon là on est forcé de penser que c'est bien sympa d'avoir adoucit les couleurs mais qu'on voit pas grand chose... D'où cela vient-il ? Et bien en fait sur cette image j'ai choisi un nombre de 500 itérations avant de considérer qu'un point n'ayant pas divergé est bien dans l'ensemble. Ce qui signifie que je vais avoir des valeurs assez élevées à la frontière de l'ensemble (genre 500 en fait :p), mais pour très peu de points ...

Donc notre régression linéaire conduit à une image très moche étant donné que la couleur associée à la valeur maximale (ici le jaune) ne sera donnée qu'à un très petit nombre de points, tous les autres points étant certes beaucoup plus nombreux mais ayant une valeur beaucoup plus petite en comparaison et des variations de valeur minimes entre eux.

On pourrait essayer d'arranger le coup en limitant la variation de valeurs en mettant un nombre d'itérations maximal beaucoup plus faible pour que les inégalités de valeurs entre les points ne soient pas si flagrantes. Par contre on perd beaucoup en précision :( C'est ce qui est fait ici en prenant 50 comme nombre d'itérations maximal :

{:.center}
![I'm blurpy and I like it.](/img/mandel_lin_0aa_50.png)

### Le coloriage par histogramme

Comme on a vu, il y a un problème à cause du petit nombre de points qui ont une très grande valeur (à la frontière). Pour colorier de manière plus équilibrée il va falloir tenir compte du nombre de points dans certains intervalles pour être capable de dire si une variation de valeur doit être répercutée en une grande variation de couleur ou pas.

Je m'explique : on a beaucoup de points ayant une valeur assez faible dont les variations entre elles ne sont pas très grandes en absolu par rapport aux points à côté de la frontière qui sont pourtant eux beaucoup moins nombreux. On va donc pondérer les variations de valeurs avec le volume de points dans l'intervalle pour plus mettre en évidence les petites variations là où il y a beaucoup de points que les grandes variations là où il y a très peu de points.

Pour cela on commence par répartir les valeurs de notre matrice en intervalles de longueur égales pour voir comment elles sont réparties.

{:.center}
![J'y ai au moins passé 1h de R&D alors on se moque pas !](/img/histogram.png)

Donc comme on peut le voir on a beaucoup plus de points ayant une valeur entre 0 et 50 ! Il s'agit maintenant de prendre ça en compte au moment de colorier l'image. L'idée est de pondérer les variations de couleurs par rapport à la proportions de points dans les intervalles.

Donc on calcule la variation globale de la composante R de la couleur par exemple. Ensuite on remarque qu'il y a 90% des points dans l'intervalle [0..50], donc on va dire que ces points là vont être répartis sur 90% de la variation totale de couleur. Donc on calcule les deux couleurs qui correspondent au début et à la fin de la variation recouverte par ces points là et on fait tout simplement une régression linéaire sur cet intervalle entre es deux couleurs pour associer une couleur aux valeurs (si t'as compris alors t'as un sérieux problème).

On passe ensuite à l'intervalle [50..100] qui a seulement 5% des points, donc on calcule les deux couleurs correspondant à une variation de 5% de la couleur et on fait une nouvelle régression linéaire. A la fin on obtient ainsi une loi de correspondance valeur -> couleur qui ressemble à ça :

{:.center}
![Ce graphique rend toute sa crédibilité à ce bog. Enfin au moins j'aurais essayé.](/img/colorsplot.png)

On voit bien que la variation de couleurs entre 0 et 50 est bien plus importante que pour les autres ! Pour un nombre maximal d'itérations de 500 on obtient la figure suivante :

{:.center}
![Still ugly.](/img/mandel_hist_0aa_500.png)

C'est tout pour aujourd'hui ! La prochaine fois on verra comment faire de l’antialiasing pour rendre tout ça plus joli. Petit avant-goût plus bas :p

Ah et je mettrais le code qui correspond aussi ^^

{:.center}
![Aucun animal n'a été blessé au cours de la génération de cette fractale. Enfin officiellement.](/img/mandel_hist_9aa_500.png)
