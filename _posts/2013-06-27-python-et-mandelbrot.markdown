---
layout: post
title: "Python et Mandelbrot"
date: 2013-06-27 23:48
header-img: "img/python.jpg"
author: "poulpi"
---

**Disclaimer : Ce billet sera geek. Voilà ça va parler de programmation (pas trop quand même) et de jolies formes 
(on se rapproche de mon élément :p). Je préfère prévenir =)**

##Une belle rencontre

Il y a peu, on m'a demandé de rajouter une nouvelle fonction dans l'outil que je réalise dans mon stage. il s'agit 
d'une reconnaissance de feuille de papier dans une photo prise par un smartphone puis redimensionnement et "correction"
 de la dite feuille. Le tout en embarqué.

Le quidam moyen qui a déjà un peu joué avec ce genre de choses sais qu'il ne faut pas chercher midi à quatorze heure 
et tout est faisable en quelques lignes bien pensées avec [OpenCV](http://opencv.org) qui dispose d'ailleurs d'une version android, le rêve.

Je ne m'étendrai pas sur cette bibliothèque mais en résumé elle est faite pour les gens qui n'ont pas envie de passer 
leur vie à compulser les 2000 articles scientifiques qui dissertent de la meilleure façon de traiter une image. OpenCV 
c'est un truc de mecs qui veulent que ça fonctionne sans se prendre la tête. Period. Mais c'est aussi une librairie poilue en C++. Et ça c'est pas cool.

Parce que faire joujou avec sa webcam c'est sympa. Mais quand il s'agit de le faire avec un programme codé (et compilé) 
en C++, faut se retrousser les manches. Et ça peut prendre la nuit pour juste arriver à faire dire "Pouet" à son écran.
 Par conséquent j'ai pas envie de prototyper avec ça. Mais OpenCV a pensé à nous autres codeurs glandu, 
 il y a un wrapper Python #top#

C'est comme ça que j'ai rencontré [Python](http://fr.wikipedia.org/wiki/Python_\(langage\)). Comme toute les choses 
merveilleuses, ce langage est apparu au début des années 90. Et il est sexy #love#

##Un truc pour les gens qui aiment garder les chose simples (sans sacrifier la puissance)

Python c'est un langage qui a tout ce qui faut à un langage de "sexy codeur" © : il est élégant, possède la notion d'objet, 
permet de faire de grande chose facilement et dispose d'une grande communauté avec plein de librairies trop stylées.

C'est aussi un langage sans typage fort. Ça veut dire que les variables ne te cassent pas les couilles à te demander sans 
cesse de définir qui elles sont, quelle place elles doivent prendre en mémoire, etc. Par exemple en C pour faire un tableau
 d'entier c'est relativement emmerdant. En Python tu mets un tableau dans une variable et elle encaisse. Si deux ligne plus
 tard tu préfères mettre une chaîne de caractère, elle va se transformer pour te permettre de le faire. En C j'en parle même 
 pas, c'est limite si ton PC ne t'insulte pas.

Après quelques recherches le langage a l'air assez populaire pour les scientifiques qui veulent des gros programmes sans se 
briser les neurones (ou toute autre organe de forme sphérique) à le faire en MATLAB : Python est plus souple, plus facilement 
optimisable, beaucoup plus gratuit :p et surtout c'est plus facile de s'y retrouver dès qu'on fait des gros programmes.

##C'est là où on passe aux jolies formes

Quand je commence un langage j'aime bien coder un truc qui sert à rien. Quand j'ai commencé le C sur ma TI-92 au lycée, j'avais 
fait un programme qui dessinait [l'ensemble de Mandelbrot](http://fr.wikipedia.org/wiki/Ensemble_de_Mandelbrot). J'ai réitéré.

Voilà donc je vous offre quelques images avec des jolies couleurs que mon PC il a fait tout seul. Et je vous donne le code python 
aussi. Vous allez voir, c'est pas sorcier. Enfin je connais des gens qui simulent des ballons de rugby avec des triples boucles 
imbriquées en MATLAB, moi je reste au O(n²) :p (une image comme ça prends quand même 20 secondes sur ma bécane ...)

{:.center}
![Le paté le plus connu de l'histoire](/img/mandel1.png)

{:.center}
![Voilà je zoome. J'ose.](/img/mandel2.png) 

{:.center}
![Les détails c'est important !](/img/mandel3.png)

Et le code maintenant. J'ai une classe qui calcule une matrice qui représente l'ensemble et un fichier qui l'affiche.

Le code de génération :

    import numpy as np
    import sys

    def frange(start, stop, step=1.0):
        while start < stop:
            yield start
            start += step
            
    class mandelbrot:

        def __init__(self, rect, w, h, max_iteration):
            self.grid = np.zeros((w, h))
            self.max_iteration = max_iteration
            self.rect = rect
            self.w = w
            self.h = h
            
        def compute(self):

            w = 0
            
            for i in frange(self.rect.left, self.rect.right, float((self.rect.right - self.rect.left)/float(self.w))):

                h = self.h - 1
                
                for j in frange(self.rect.bottom, self.rect.top, float((self.rect.top - self.rect.bottom)/float(self.h))):
                    
                    c = complex(i, j)
                    z = complex(0,0)
                    t = 0
                    
                    while (abs(z) < 2 and t < self.max_iteration):
                        z = z * z + c
                        t = t + 1
                        
                    if w >= self.w:
                        print 'Out of bound : w'
                        continue

                    if h >= self.h:
                        print 'Out of bound : h'
                        continue
                    
                    if(abs(z) >= 2):
                        self.grid[w, h] = t
                        #print 'z : ' + repr(z.norm()) + ' t : ' + repr(t) + ' zx : ' + repr(z.x) + ' zy : ' + repr(z.y)
                    else:
                        self.grid[w, h] = -1

                    #print '  i / j | h= ' + repr(i) + ' ' + repr(j) + ' ' + repr(h)
                    h = h - 1
                    
                if w % 10 == 0:
                    print 'w = ' + repr(w)
                    
                w = w + 1
        
            return self.grid;                           
                              

Comment afficher la matrice :

    import sys
    import pygame

    import Mandelbrot

    class rectangle:
       def __init__(self, bottom, top, left, right):
          self.right = right
          self.left = left
          self.top = top
          self.bottom = bottom
          
    def draw(N):
       w = N.shape[0]
       h = N.shape[1]
       
       pygame.init() 

       #create the screen
       window = pygame.display.set_mode((w, h)) 

       # Fill background
       background = pygame.Surface(window.get_size())
       background = background.convert()
       background.fill((250, 250, 250))

       for i in range(w):
          for j in range(h):
             if N[i, j] != -1:
                t = N[i, j]
                ## C'est là qu'on tweake les couleurs ! (là c'est fait à la zob mais en vrai faudrait prendre
                ##les valeurs max de la matrice pour faire un joli dégradé)
                background.set_at((i, h - j), ((t*32)%255, (t*16)%255, (t*4)%255))

       #save the image to file
       pygame.image.save(background, 'mandel.png')

       #draw it to the screen
       window.blit(background, (0, 0))
       
       pygame.display.flip()

       #input handling (somewhat boilerplate code):
       while True: 
          for event in pygame.event.get(): 
             if event.type == pygame.QUIT: 
                 sys.exit(0) 
             else: 
                 print event 
          
    ## C'est là où on paramètre les intervalles qu'on veut afficher
    ymin = -1
    ymax = 1
    xmin = -2
    xmax = 0.5

    largeur = 800
    iteration = 300

    ratio = float(ymax - ymin) / float(xmax - xmin)
    rect = rectangle(ymin, ymax, xmin, xmax)

    w = int(largeur)
    h = int(ratio * largeur)

    M = Mandelbrot.mandelbrot(rect, w, h, iteration)
    N = M.compute()

    draw(N)

Pour infos j'utilise la super librairie [PyGame](http://www.pygame.org/) pour l'affichage et [Numpy](http://www.numpy.org/) pour faire ma matrice.