---
layout: post
title: "Mandelbrot et Python : Part II (Optimisation)"
date: 2013-07-02 20:48
header-img: "img/thinker.jpg"
author: "poulpi"
---

Alors dans mon billet précédent je vous parlais de ma découverte de Python et je vous ai mis quelques images 
de l'ensemble de Mandelbrot en vous donnant l'algorithme qui produit ces zolies images #love# Mais aujourd'hui 
on va essayer de faire un peu mieux.

Bon je pense que personne n'est allé sur wiki pour voir ce que c'était que cet ensemble, donc je vais le faire 
vite fait. Vous voyez, comme tout les bons blockbusters, on va parler d'une suite, et même d'une suite complexe !

$$
\begin{cases}
Z_0 = 0\\
Z_{n+1} = Z_n^2 + c
\end{cases}
$$

Bon voilà elle fait pas trop peur (en plus elle est en $\LaTeX$ #tritop#) . A moins que vous ayez la phobie des suites 
depuis que vous avez subi Matrix II et III. L'idée de cette suite c'est qu'on va faire varier $c$ en prenant toutes 
les valeurs du plan complexe. Quand la suite ne diverge pas on va dire que c'est dans l'ensemble de Mandelbrot 
(le blanc sur les images). Et quand elle diverge on va lui donner une couleur relative à la vitesse à laquelle elle a
 divergé. Bref, pas de quoi casser 3 nuggets à un canard.
 
Mais malheureusement on a pas de jolie formule pour savoir si la suite diverge ou pas, il faut la calculer et voir 
au bout d'un moment si on est parti en live (le module explose => la suite diverge). Des mathématiciens trop stylés 
(je dis ça parce que j'ai oublié leurs noms dès que je les ai lus) ont montré que si le $\| z\| > 2$ 
à un moment, alors ça va diverger. Facile.

Mais dans la cas où ça diverge pas, on est obligé de calculer toutes les itérations de la suite jusqu'à ce qu'on en ai 
marre. Et la patience d'un informaticien est très limitée (si on passe sa vie à attendre l'ordinateur, alors ça bouffe du
 temps libre qu'on préférerait passer dans son pieu, parole de stagiaire =) ).

Par exemple dans mes images j'ai mis une limite de 300 itérations avant qu'on dise que ça ne diverge pas. Mais ça prend
 un temps fou de calculer jusqu'à $Z_{299}$ à chaque fois, genre 30 secondes par image #eeek#

Donc, comme le rendu est déjà assez long et que j'aimerais bien améliorer un peu l'aspect de nos fractales, on va voir 
comment optimiser tout ça ! Il existe 2 types d'optimisation lorsqu'on fait du code :

- **L'optimisation technique :** ça va consister à essayer de faire exécuter son code le plus vite possible en évitant les pièges
 des langages (genre les boucles en Matlab ou instancier 15 000 objets en java). Ça va aussi consister à changer de langage 
 de programmation si on a atteint les limites de celui qu'on utilise (genre passer au C/C++) ou alors faire du multi-threading.
 On finit bien sûr par la solution la plus prisée des gens qui ont de l'argent mais pas de temps (les boîtes quoi...) : acheter 
 une machine plus puissante. Bref ce genre d'optimisation c'est quand on a plus d'idées, ça permet de faire gagner en performance 
 mais c'est mieux de commencer par le second type d'optimisation ! 

- **L'optimisation fonctionnelle :** là il va s'agir de réfléchir au problème qu'on essaye de résoudre, à comment comment trouver le résultat 
en moins de temps. Donc typiquement ça va être de faire baisser la complexité asymptotique du code (ne pas dépasser le $O(n^{2})$ et encore le 
$O(n)$ c'est quand même bien plus sympa !), trouver des astuces, chasser la bonne idée qui va tout casser. 

Vous l'aurez compris je ne suis pas très intéressé par une optimisation technique de ce code dans un premier temps (sinon j'aurai fait ça en C direct). Pour autant voici les 
pistes naturelles qu'il faudrait suivre :

- **Tout refaire en C/C++**, mais ici le sujet c'est le python donc bof bof... 
- **Utiliser Cython :** un package qui fait sauter le typage faible mais qui accélère pas mal les choses, ça demande de revoir un peu le code 
mais ça se fait... 
- **Utiliser Numba :** ça va compiler le code python à l'exécution, c'est cool mais ça demande LLVM et j'ai pas encore trouvé comment l'installer
 facilement sur Windows :(
- **Me mettre au multi-threading en Python :** ouais on verra ça plus tard %)

###Bon alors, comment on fait pour optimiser Mandelbrot ?

Et bien, des mathématiciens ont montré (oui Mandelbrot c'est un truc de mecs qui s'ennuient, alors forcément y'a des résultats !) que l'ensemble 
de Mandelbrot est connexe #top#. En français ça veut dire que c'est un espace d'un seul tenant, sans trous. En gros c'est une patate.

Et donc une des implications c'est que si je dessine un contour qui appartient à l'ensemble, alors tout ce qu'il y a dans le contour appartient 
à l'ensemble ! Donc en gros je vais couper mon image en pleins de petits rectangles (disons 4 pour commencer) et voir si le contour de ces rectangles
 est dans l'ensemble (si ça diverge quoi !). Si oui je les rempli en blanc, sinon je coupe mon rectangle en pleins de petits rectangles et je m'intéresse
 à ces minis rectangles que je peint en blanc ou que je recoupe en ...., etc.

Forcément il y a un moment où on arrête de couper les rectangles en petits rectangles et où on fait comme avant (on regarde pour tout les points si ça
 diverge ou pas). Mais normalement ça devrait faire gagner pas mal de perfs !

Alors voici 2 images pour montrer comment ça marche maintenant. Sur la première j'arrête assez vite d'essayer de faire des petits carrés alors que sur la seconde je
continue assez longtemps à en faire. Toute la partie en rouge est donc constitué de rectangles que j'ai colorié en rouge parce que leur contour appartient à l'ensemble de Mandelbrot,
donc leur contenu aussi.

Dans un prochain (et sans doute dernier) billet sur le sujet j'essayerai un coup de voir comment on pourrait faire pour faire ça en multi-threading et voir les apports niveau performance
 des différentes approches.

{:.center}
![Hum, quel joli choix de couleurs !](/img/mandel-opt1.png)

{:.center}
![Ceci n'est pas un hot dog.](/img/mandel-opt2.png)

Le code : par rapport à la dernière fois, mandelbrot.py s'est fait passé dessus, et y'a eu une ou deux modifs sur launcher.py.

    import numpy as np
    import sys
    import Rectangle as Rect

    def frange(start, stop, step=1.0):
        while start < stop:
            yield start
            start = start + step
            
    class mandelbrot:

        def __init__(self, w, h, rect, max_iteration, max_small_rect_width):
            self.grid = -2 * np.ones((w, h))
            self.max_iteration = max_iteration
            self.w = w
            self.h = h
            self.rect = rect
            self.wIncrement = float((self.rect.right - self.rect.left)/self.w)
            self.hIncrement = float((self.rect.top - self.rect.bottom)/self.h)
            self.max_small_rect_width = max_small_rect_width

        # Fonction récursive qui coupe en 4 le rectangle qu'on lui donne et regarde si le contour des petits rectangles
        # appartient à l'ensemble; Si c'est le cas on "colorie" l'intérieur du rectangle sinon on appelle la fonction sur chacun des
        # petits rectangles ...
        # Quand on a des rectangles trop petits, on se contente de les calculer (sans les diviser)
        def recCompute(self, w, h, rect):

            if(w > self.max_small_rect_width):

                # Decoupe en 4 du rectangle
                rects = []
                rects.append(Rect.rectangle(abs(rect.top - rect.bottom) / 2. + rect.bottom, rect.top, rect.left, abs(rect.right - rect.left) / 2. + rect.left))
                rects.append(Rect.rectangle(abs(rect.top - rect.bottom) / 2. + rect.bottom, rect.top, abs(rect.right - rect.left) / 2. + rect.left, rect.right))
                rects.append(Rect.rectangle(rect.bottom, abs(rect.top - rect.bottom) / 2. + rect.bottom, rect.left, abs(rect.right - rect.left) / 2. + rect.left))
                rects.append(Rect.rectangle(rect.bottom, abs(rect.top - rect.bottom) / 2. + rect.bottom, abs(rect.right - rect.left) / 2. + rect.left, rect.right))

                for recti in rects:
                    
                    if(self.isInsideMandelbrot(recti)):
                        
                        w0 = (self.rect.left - recti.left) * self.w / (self.rect.left - self.rect.right)
                        w1 = (self.rect.left - recti.right) * self.w / (self.rect.left - self.rect.right)
                        h0 = self.h - (self.rect.top - recti.bottom) * self.h / (self.rect.top - self.rect.bottom) - 1
                        h1 = self.h - (self.rect.top - recti.top) * self.h / (self.rect.top - self.rect.bottom) - 1

                        self.grid[w0:w1, h0:h1] = -3
                        
                    else:
                        self.recCompute(w/2., h/2., recti)

            else:
                offw = abs((self.rect.left - rect.left) * self.w / (self.rect.left - self.rect.right))
                offh = self.h - abs((self.rect.top - rect.top) * self.h / (self.rect.top - self.rect.bottom))
                self.computeRectangle(rect, w, h, offw, offh)
                return

        # Calcule la matrice des divergences des intervalles définis
        def compute(self):

            self.recCompute(self.w, self.h, self.rect)

            return self.grid

        # Regarde si le contour d'un rectangle est dans l'ensemble de Mandelbrot
        def isInsideMandelbrot(self, rect):

            for i in frange(rect.left, rect.right, self.wIncrement):
                if(self.computeOnePoint(i, rect.top) != -1):
                    return False

            for i in frange(rect.left, rect.right, self.wIncrement):
                if(self.computeOnePoint(i, rect.bottom) != -1):
                    return False

            for j in frange(rect.bottom, rect.top, self.hIncrement):
                if(self.computeOnePoint(rect.left, j) != -1):
                    return False

            for j in frange(rect.bottom, rect.top, self.hIncrement):
                if(self.computeOnePoint(rect.right, j) != -1):
                    return False

            return True

        # Calcule la valeur d'un point (retorune le nombre d'irération avant que la suite ne diverge
        # ou -1 si la suite ne diverge pas
        def computeOnePoint(self, i, j):

            c = complex(i, j)
            z = complex(0,0)
            t = 0
                    
            while (abs(z) < 2 and t < self.max_iteration):
                z = z * z + c
                t = t + 1

            if(abs(z) >= 2):
                return t
            else:
                return -1

        # Calcule tout les points d'un rectangle avec les valeurs étant dans l'intervalle du rectangle rect et
        # avec une résolution permettant d'aller dans une matrice de taille (w, h)
        # Les offw et offh sont deux offsets pour mettre les résultats au bon endroit dans la matrice
        def computeRectangle(self, rect, w, h, offw, offh):

            mW = offw
            
            for i in frange(rect.left, rect.right, self.wIncrement):

                mH = offh - 1
                
                for j in frange(rect.bottom, rect.top , self.hIncrement):

                    if mW >= self.w:
                        print 'Out of bound : w = ' + repr(mW) + ' i = ' + repr(i) + ' j = ' + repr(j)
                        continue

                    if mH >= self.h:
                        print 'Out of bound : h = ' + repr(mH) + ' i = ' + repr(i) + ' j = ' + repr(j)
                        continue

                    if self.grid[int(mW), int(2 * offh - h - mH - 2)] == -2:
                        self.grid[int(mW), int(2 * offh - h - mH - 2)] = self.computeOnePoint(i, j)
                            
                    mH = mH - 1
                    
                mW = mW + 1
        
            return self.grid;                           
                               
^

    import sys
    import pygame

    import Mandelbrot
    import Rectangle as Rect
          
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
                if(t != -3):
                   background.set_at((i, h - j), ((t*32)%255, (t*16)%255, (t*4)%255))
                else:
                   background.set_at((i, h - j), (255, 0, 0))

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
    ymin = -1.
    ymax = 1.
    xmin = -2
    xmax = 0.5

    largeur = 1000
    iteration = 500

    ratio = float(ymax - ymin) / float(xmax - xmin)
    rect = Rect.rectangle(ymin, ymax, xmin, xmax)

    w = int(largeur)
    h = int(ratio * largeur)

    M = Mandelbrot.mandelbrot(w, h, rect, iteration, 20)
    N = M.compute()

    draw(N)
