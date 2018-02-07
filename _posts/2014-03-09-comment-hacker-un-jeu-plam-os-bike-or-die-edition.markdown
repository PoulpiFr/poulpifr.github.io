---
layout: post
title: "Comment hacker un jeu Palm Os ?"
date: 2014-03-09 23:24
header-img: "img/hack.jpg"
author: "poulpi"
---

## Contexte 
Le soleil est revenu et il n'y a rien de tel qu'un bain de lumière pour faire le plein d'idées =) Ainsi j'ai trouvé mon prochain projet qui devrait m'occuper quelques soirées ^^

En fait je suis récemment retombé sur une pépite de mon adolescence : le jeu **Bike or Die** que j'avais sur ma [**Tapwave Zodiac**](http://en.wikipedia.org/wiki/Tapwave_Zodiac), et il déchirait ! Ce qui faisait la force du jeu, en plus de son gameplay à la fois fun et frustrant, c'était son éditeur de niveaux qui a permis à de nombreux joueurs de créer eux mêmes leurs packs de cartes.

Comme il n'y a pas vraiment à mon sens de jeu sur Android et iOs qui reprend ce concept, je me suis donné le projet d'en faire un. Certes il y a eu un port de Bike or Die par le créateur mais il est vieux, cher et moche, un peu comme les meubles Louis XIV qu'on trouve dans les brocantes ...

L'idée de base serait d'assurer une compatibilité avec les 1700+ niveaux de **Bike or Die**.  Donc pour cela il va déjà me falloir comprendre comment sont stockés les niveaux étant donné que le format de données n'est pas libre ou documenté =) Et ça commence par arriver à lancer le jeu sur mon ordinateur ! 

*\<mylife\>*
Pour cela il faut trouver un émulateur de Palm OS sur PC et y installer le jeu. Personnellement je n'ai réussi qu'à faire fonctionner la version 1.6b sur Palm OS 3.5. En effet je n'ai jamais réussi à installer la version 2 de Bike or Die sur un émulateur de Palm OS 5 ou 6 (il faut au moins la version 5 pour faire tourner la deuxième version du jeu). 
*\</mylife\>*

Mais une fois le jeu installé, il faut le faire passer en version complète en achetant un code spécifique à taper directement dans le jeu. Le seul hic c'est que le site permettant d'acheter les codes ne fonctionne plus. Et donc il va falloir le hacker #devil#

## Comment hacker un jeu Palm OS ? 

Bon vous l'aurez compris, le plus facile est de modifier le jeu pour qu'il accepte n'importe quel code comme étant bon. Et c'est exactement ce qu'on va faire. Cette manipulation peur s'appliquer pour pleins d'autres programmes que juste un jeu Palm OS :) 

Donc on va désassembler le fichier du jeu pour obtenir le code assembleur et essayer de comprendre où a lieu la vérification du code rentré. Pour cela on pourra utiliser le désassembleur [PilotDis](http://www.freewarepalm.com/utilities/pilotdis.shtml). 

Le résultat ressemble à ça :

	[...]
	00042c76   2f03				L1396	MOVE.L	D3,-(A7)
	00042c78   4e4fa170				TRAP	#15,$A170 = sysTrapFrmDeleteForm
	00042c7c   588f					ADDQ.L	#4,A7
	00042c7e   0c44000c				CMPI.W	#12!$c,D4
	00042c82   660c					BNE	L1397
	00042c84   206decd0				MOVEA.L	-4912(A5),A0
	00042c88   d1fc000070dc				ADDA.L	#28892!$70dc,A0
	00042c8e   4e90					JSR	(A0)
	00042c90   4cee0038ffc4			L1397	MOVEM.L	-60(A6),D3-D5
	00042c96   4e5e					UNLK	A6
	00042c98   4e75					RTS
	00042c9a   4e560000			L1398	LINK	A6,#0
	00042c9e   206e0008				MOVEA.L	8(A6),A0
	00042ca2   0c500009				CMPI.W	#9,(A0)
	00042ca6   6634					BNE	L1402
	00042ca8   30280008				MOVE.W	8(A0),D0
	00042cac   0c400014				CMPI.W	#20!$14,D0
	00042cb0   670a					BEQ	L1399
	00042cb2   0c400015				CMPI.W	#21!$15,D0
	00042cb6   6712					BEQ	L1400
	00042cb8   60000022				BRA	L1402
	00042cbc   206deccc			L1399	MOVEA.L	-4916(A5),A0
	00042cc0   d1fc00004084				ADDA.L	#16516!$4084,A0
	00042cc6   6000000c				BRA	L1401
	00042cca   206deccc			L1400	MOVEA.L	-4916(A5),A0
	00042cce   d1fc000037f8				ADDA.L	#14328!$37f8,A0
	00042cd4   4e90				L1401	JSR	(A0)
	00042cd6   7001					MOVEQ	#1,D0
	00042cd8   60000004				BRA	L1403
	00042cdc   4240				L1402	CLR.W	D0
	00042cde   4e5e				L1403	UNLK	A6
	00042ce0   4e75					RTS
	00042ce2   4e56ffac			L1404	LINK	A6,#-84
	00042ce6   48e71c38				MOVEM.L	D3-D5/A2-A4,-(A7)
	00042cea   47edf1b0				LEA	-3664(A5),A3
	00042cee   3a2b0132				MOVE.W	306(A3),D5
	00042cf2   1d7c0005ffac				MOVE.B	#5,-84(A6)
	00042cf8   422effad				CLR.B	-83(A6)
	00042cfc   3d7c001effae				MOVE.W	#30!$1e,-82(A6)
	00042d02   41eb0078				LEA	120(A3),A0
	[...]

Bon sauf qu'il y a pas mal de lignes quand même (105 461 en tout :o), donc on va essayer de cibler la partie qui nous intéresse. Pour cela, jetons un coup d’œil à ce qui se passe quand on rentre un code faux :

{:.center}
![Oui, la HD c'était pas encore trop ça ...](/img/unlock_bik.jpg)

Bon vous l'aurez vu, le bouton passe de "Unlock" à Wrong code", et ça c'est un début ! Donc on recherche "Wrong code" dans le code assembleur et on trouve cette ligne :

	0004312a   57726f6e6720636f6465		L1422	DC.B	'Wrong code'

Alors en langage "humain", cette ligne revient à stocker le texte "Wrong code" dans une variable **L1422** (bon en vrai c'est un peu plus subtil mais on va pas la jouer trop fine). Donc on regarde quand est ce que **L1422** revient dans le code et on trouve :

	00043318   41fafe10			L1431	 LEA	L1422,A0

Donc cette fois ci en gros on charge la variable en mémoire si **L1431** est appelé. Donc on recherche cette fois ci **L1431** dans le code ! Et on trouve :

	00043300   320b				L1430	MOVE.W	A3,D1
	00043302   4a01					TST.B	D1
	00043304   6712					BEQ	L1431
	00043306   397c0001007c				MOVE.W	#1,124(A4)
	0004330c   2f07					MOVE.L	D7,-(A7)
	0004330e   4e4fa170				TRAP	#15,$A170 = sysTrapFrmDeleteForm
	00043312   588f					ADDQ.L	#4,A7
	00043314   6000fe64				BRA	L1426

Donc là on voit qu'il y a un test (**TST.B**) puis que si il y avait égalité dans le test, on part en **L1431** (**BEQ L1431**). Et je crois qu'on a compris que **L1431** c'était vraiment pas là où on voulait aller ^^ Donc on va transformer le **BEQ L1431** en **BNE L1431** (**BNE** c'est l'inverse de **BEQ**). 

Bon, mais comment on fait ? Vous aurez remarqué que à chaque fois que j'ai mis du code, il y avait 3 colonnes et que je ne me suis intéréssé qu'à la troisième, et bien c'est maintenant que ça va changer ! La première colonne donne en fait à chaque fois la position dans le fichier source du code et la seconde colonne l'hexadécimal brut. Et la troisième colonne c'est l'hexadécimal traduit en assembleur.

Donc on va devoir traduire notre code assembleur en hexadécimal et aller le mettre à la bonne adresse dans le fichier en utilisant un éditeur hexadécimal =)

## Assembler à la main une instruction, comment fait-on ?

Pour commencer, je propose d'assembler l'instruction **BEQ L1431** car on connaît déjà sa forme hexadécimal car elle est dans la deuxième colonne de sa ligne (**6712**). 

C'est ici qu'on va utiliser le [super tableau de GoldenCrystal](http://goldencrystal.free.fr/M68kOpcodes.pdf) dans lequel on va s'intéresser à la ligne **Bcc** (oui en fait **BEQ**, **BNE** et consorts c'est la même famille qui s'appelle **Bcc** ^^). Donc on y voit que l'instruction tient sur 4 mots de 4 bits chacuns :

- Le premier est 0b0110 et identifie le fait que c'est une instruction de la famille **Bcc**
- Le second est 0b0111 et c'est celui qui donne le type de la condition (ici c'est **EQ** comme on assemble **BEQ**). La liste des mots de 4 bits représentant les différentes conditions est dans un petit tableau à droite sur le pdf de GoldenCrystal
- Les deux derniers ça va être l'offset de la distance à parcourir pour arriver à la ligne du code à laquelle on aimerait aller si la condition est vérifiée. Donc c'est l'adresse d'arrivée (**L1431**) moins l'adresse juste après notre **BEQ**, càd 0x00043318 - 0x00043306 = 0x12 = 0b0001 0010

Donc au final notre instruction vaut :
0b0110 0111 0001 0010 = 0x6712

On est bien retombé sur notre code hexa pour notre **BEQ** ! Du coup on trouve assez facilement comment faire un **BNE** (il suffit de changer le byte de condition en prenant 0b0110 au lieu de 0b0111 pour passer de **EQ** à **NE**, [cf le pdf](http://goldencrystal.free.fr/M68kOpcodes.pdf)). Du coup notre nouvelle instruction vaut :
0b0110 0110 0001 0010 = 0x6612

Donc vous l'aurez compris, pour hacker le jeu, on va aller changer le 0x6712 à la ligne 00043304 (cf la première colonne du code mis plus haut) pour le remplacer par 0x6612. Pour cela on ouvre n'importe quel éditeur hexadécimal et on charge le fichier du jeu. End of story. 

##Conclusion 

Tout ça pour ça vous allez me dire. Mais en changeant un 7 en 6 au bon endroit, on hacke le système de protection d'un jeu. Moi je trouve ça pas mal ! 

Sur ce je vais continuer à essayer de comprendre le format des cartes de niveaux du jeu pour coder ma propre version, je me laisse 2-3 mois pour avoir une bêta =) 

PS : Il n'y a pas 10 ans ma petite manip de ce post me semblait magique et hors de portée des gens "normaux", c'est marrant comme les temps changent ^^
