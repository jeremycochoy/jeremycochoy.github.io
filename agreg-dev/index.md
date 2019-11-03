---
layout: page
title: Développements algébriques
permalink: /agreg-dev/
---

Disclaimer : Je fait une impasse sur conique et une semi-impasse sur
déterminant. Attendez vous à des recasages abusifs pour cette leçon.


01) Théorème de Lüroth (pseudo-original)
----------------------------------------

C'est à l'origine un résultat de géométrie algébrique.

La démonstration utilise des extensions de corps, des polynômes minimaux et
annulateurs, ainsi que des anneaux. La référence est correcte, mais je me
suis permit de ré-agencer les arguments pour qu'ils paraissent plus naturels.

L'idée est que, sous de bonnes hypothèses, à la droite projective complexe on
peut associer le corps $k(X)$, et à une courbe algébrique $C$ un corps $L$.
On a une correspondance entre les morphismes de courbes et les morphismes de
corps. Un morphisme de $\mathbb{P}^1$ dans $C$ induit un morphisme
$L \rightarrow C$. Le théorème donne alors un inverse pour ce morphismes de
corps, qui montre alors que $C$ est isomorphe à $\mathbb{P}^1$.

Mais bon, mieux vaut garder ces détails pour soit, et éviter d'en parler le
jours de l'agreg.

+ __125 : Extensions de corps. Exemples et applications.__
+ __140 : Corps des fractions rationnelles à une indéterminée sur un corps
          commutatif. Applications.__
+ __142 : Algèbre des polynômes à plusieurs indéterminées. Applications.__

_Ref : Patrice Tauvel - Corps commutatifs et théorie de Galois - p 88, 89._


02) Équation cartésienne d'une courbe rationnelle (original)
------------------------------------------------------------

L'une des applications qui vous fera aimer la leçon résultant si ce n'est pas
déjà le cas.

C'est encore une fois un résultat de géométrie algébrique, ou plutôt une version
édulcoré pour l'agreg. On a une courbe donné par un paramétrage rationnel (de la forme $\{(F(t), R(t)) | t \in k\}$ pour $F, S \in k(X)$) et l'on souhaite
une équation de la forme $P(X, Y) = 0$. Comme le résultat n'est pas très
difficile à obtenir, on montreras aussi un premier résultat sur le résultant.

Ce developpement est plutot simple et pas trop long.

+ __143 : Résultant. Applications.__

_Ref : Goblot pour le premier lemme._


03) Nullstellensat faible par le résultant
------------------------------------------

Un résultat fonamentale de géométrie algébrique.

La démonstration se fait d'abord par un lemme de changement de base qui permet
de rendre verticales les asymptotes dans une direction $X_ n$ pivilégiée.

L'idée, pour montrer qu'une famille de polynomes n'ont pas de zéros communs,
est de prendre le résultant de "toute une famille" $f_ n$, ce que l'on fait en
introduisant un polynome en une indéterminée supplémentaire $U$ dont les
coefficients sont les éléments $f_ n$.

+ __142 : Algèbre des polynômes à plusieurs indéterminées. Applications.__
+ __143 : Résultant. Applications.__

_Ref : Apery._


04) Gauss-Wantzel
-----------------

Détermine les polygones constructible à la règle et au compas.
Le développement est un peu long, et plutot mal référencé, mais
a de bons recasages.

+ __102 : Groupe des nombres complexes de module 1. Sous-groupes des racines
          de l’unité. Applications.__
+ __125 : Extensions de corps. Exemples et applications.__
+ __182 : Applications des nombres complexes à la géométrie.__

_Ref : Patrice Tauvel - Corps commutatifs et théorie de Galois._
_Ref : Carréga._


05) D'Alembert-Gauss algébrique
-------------------------------

Bien que le résultat soit classique, et démontrable en une ligne via Liouville,
tout l'intérêt repose dans la démonstration. Elle utilise les relations
coefficient racines, le théorème fondamentale des polynômes symétriques
élémentaires, et la résolution d'équations de degré 2 dans $\mathbb{C}$.

Mais, dans toute démonstration de D'Alembert-Gauss, on est obligé d'utiliser
la topologie de l'ordre. Ici cela se produit au travers du théorème des valeurs
intermédiaires appliqué aux polynômes de degré impair.

+ __144 : Racines d’un polynôme. Fonctions symétriques élémentaires.
          Exemples et applications.__

_Ref : Pierre Samuel, Theorie Algebrique Des Nombres._

06) Classification des formes quadratiques ($\mathbb{R}, \mathbb{C}, \mathbb{F}_q$)
-----------------------------------------------------------------------------------

On classifie les formes quadratiques sur les corps de la vie de tous les
jours : $\mathbb{R}, \mathbb{C}, \mathbb{F}_q$.

L'élément central est une reccurence sur la dimension, et un lemme de
remplacement de paire de vecteur pour le cas des corps finis.

Un développement qui trouve tout particulièrement sa place dans exemples
d'actions de groupes sur les espaces de matrices.

+ __150 : Exemples d’actions de groupes sur les espaces de matrices.__
+ __151 : Dimension d’un espace vectoriel (on se limitera au cas de
          la dimension finie). Rang. Exemples et applications.__
+ __170 : Formes quadratiques sur un espace vectoriel de dimension finie.
          Orthogonalité, isotropie. Applications.__

_Ref : Caldero Germoni - H2G2 - p 151, 153 et p 173, 179._
w

07) L'homémorphisme $O(p,q) \rightarrow O(p)\times O(q)\times \mathbb{R}^{pq}$
------------------------------------------------------------------------------

Ce développement est très bien référencé, plutôt facile à retenir, et joli.
Il utilise la décomposition polaire et l'homéomorphisme
$exp : S_n \rightarrow S_n^{++}$.

Il a l'avantage de se recaser dans les quelques leçons de géométries, alors
qu'il ne s'agit pourtant que d'algèbre.

+ __156 : Exponentielle de matrices. Applications.__
+ __160 : Endomorphismes remarquables d'un espace vectoriel euclidien
          (de dimension finie).__
+ __161 : Isométries d'un espace affine euclidien de dimension finie.
          Applications en dimensions 2 et 3.__

_Ref : Caldero Germoni - H2G2 - p 211, 213._


08) L'isomorphisme exceptionnel $\frac{SU_2}{\pm I_2} \simeq SO_3$
------------------------------------------------------------------

Cette isomorphisme "explique" pourquoi on peux utiliser les quaternions
pour représenter les rotations de $\mathbb{R}^3$ et éviter les problèmes de
"gabal lock".

La construction du morphisme se réalise en remarquant que la conjugaison
stabilise les imaginaires des quaternions. La surjectivitée repose sur
un argument de famille génératrice.

+ __170 : Formes quadratiques sur un espace vectoriel de dimension finie.
          Orthogonalité, isotropie. Applications.__
+ __171 : Formes quadratiques réelles. Exemples et applications.__
+ __161 : Isométries d’un espace affine euclidien de dimension finie.
          Applications en dimensions 2 et 3.__
+ __183 : Utilisation des groupes en géométrie.__

_Ref : Caldero Germoni - H2G2 - p 232, 233._


09) Parallèles et méridiens de $SU_2$ (original)
------------------------------------------------

Un très jolis résultat, qui n'est pas sans rappeler la fibration de Hopf.
On pourra constater dans le Caldéro que "l'équateur" est un stabilisateur, et
que le méridien considéré est en fait une orbite, le tout pour une bonne action
qui permet de construire la fibration de Hopf à travers les actions de
groupes topologiques.

Les arguments mis en jeux sont tous assez élémentaires : polynôme
caractéristique, diagonalisation d'endomorphismes normaux, etc.

Il est peut-être un peu long, mais on peux intégrer la définition de méridien
et parallèle dans l'énoncé, et admettre le premier point comme lemme.

+ __102 : Groupe des nombres complexes de module 1. Sous-groupes des racines
          de l’unité. Applications.__
+ __106 : Groupe linéaire d’un espace vectoriel de dimension finie $E$,
          sous-groupes de $GL(E)$. Applications.__

_Ref : Michael Artin - Algebra - p 272, 275_


10) Formes de Hankel
--------------------

Aussi connus sous le nom de "comptages des racines par une
forme quadratique".

On construit une forme quadratique dont la signature $(p, q)$ compte
le nombre de racines $(p+q)$ et le nombre de racines réels $(p - q)$
distinctes.

On calcul un déterminant de vandermonde pour montrer que certaines
formes linéaires sont indépendantes. Il n'est pas très difficile,
mais la référence que je propose le donne sous forme d'exercice.


+ __144 : Racines d’un polynôme. Fonctions symétriques élémentaires.
          Exemples et applications.__
+ __152 : Déterminant. Exemples et applications.__
+ __159 : Formes linéaires et hyperplans en dimension finie.
          Exemples et applications.__
+ __170 : Formes quadratiques sur un espace vectoriel de dimension finie.
          Orthogonalité, isotropie. Applications.__
+ __171 : Formes quadratiques réelles. Exemples et applications.__

_Ref : Caldero Germoni - H2G2 - p 197._


11) Théorèmes de Sylow
----------------------

Des théorèmes classiques qui figurent dans le plan. C'est bien de
connaitre la démonstration, elle est bien référencée, et intuitive
si l'on retien les actions. Bref, des groupes, et des actions de
groupe, donc très agréable.

+ __101 : Groupe opérant sur un ensemble. Exemples et applications.__
+ __103 : Exemples et applications des notions de sous-groupe distingué
          et de groupe quotient.__
+ __104 : Groupes finis. Exemples et applications.__
+ __121 : Nombres premiers. Applications.__

_Ref : Perrin - Cours d'algèbre._


12) Irréductibilité des polynomes cyclotomiques.
------------------------------------------------

Dévelopement cours et plutot trivial. On peut le rallonger
en montrant que si $P=QR$ avec $P \in \mathbb{Z}[X]$ et Q unitaire dans
$\mathbb{Q}$ alors Q et R sont en fait dans $\mathbb{Z}[X]$.

+ __120 : Anneaux $\frac{Z}{nZ}$. Applications.__
+ __141 : Polynômes irréductibles à une indéterminée. Corps de rupture.
          Exemples et applications.__


13) Algorithme de Berlekamp
---------------------------

Un théorème qui doit être connus par tout option C, qui donne une
méthode de factorisation simple dans les corps finis. L'énnoncé
donne la méthode, et la démonstration explique comment celle ci
fonctionne.

Un résultat sympatique et bien référencé.

+ __122 : Anneaux principaux. Exemples et applications.__
+ __123 : Corps finis. Applications.__
+ __151 : Dimension d’un espace vectoriel (on se limitera au cas de
          la dimension finie). Rang. Exemples et applications.__

_Ref : Beck Malick Peyré - Objectif Agrégation._

14) Frobenius - Zolotarev
-------------------------

Certains ne l'aime pas car trop transverse, et sans réel application (il
peut quand même être utiliser pour montrer la réciprocité quadratique),
mais il se recase bien, est simple, et réalise un excellent exemple
d'utilisation des divers outils qui doivent être maitrisé en algèbre
(symbole de legendre, quotient de groupe, famille génératrice, etc..).

+ __103 : Exemples et applications des notions de sous-groupe distingué
          et de groupe quotient.__
+ __105 : Groupe des permutations d’un ensemble fini. Applications.__
+ __106 : Groupe linéaire d’un espace vectoriel de dimension finie $E$,
          sous-groupes de $GL(E)$. Applications.__
+ __121 : Nombres premiers. Applications.__
+ __152 : Déterminant. Exemples et applications.__

_Ref : Beck Malick Peyré - Objectif Agrégation._


15) Loi de réciprocité quadratique par les formes quadratiques
--------------------------------------------------------------

Un très joli développement, qui mélange actions de groupe, corps fini,
et algèbre bilinéaire. Il n'est pas si classique que l'on pourait le croire.

La démonstration consiste à compter de deux façons différentes le nombre de
points dans la sphère unité de $\mathbb{F}_p$ modulo $q$.
Le développement est très bien référencé, mais demande tout de même un petit peu
de travail pour ne pas s'emmêler les pinceaux le jour de l'orale.

+ __120 : Anneaux $\frac{Z}{nZ}$. Applications.__
+ __121 : Nombres premiers. Applications.__
+ __123 : Corps finis. Applications.__
+ __170 : Formes quadratiques sur un espace vectoriel de dimension finie.
          Orthogonalité, isotropie. Applications.__
+ __190 : Méthodes combinatoires, problèmes de dénombrement.__

_Ref : Caldero Germoni - H2G2 - p 185, 186._


16) Surjectivité de l'exponentielle $M_n(\mathbb{C}) \rightarrow GL_n(\mathbb{C})$
----------------------------------------------------------------------------------

Encore une fois, un très jolie développement. Il allie topologie
et algèbre linéaire. On montre que l'exponentielle d'un endomorphisme $A$
est un polynôme en celui çi, et que le groupe multiplicatif des inversibles
$\mathbb{C}[A]^{\times}$ est connexe.
On utilise un petit peu (tout petit peu) de groupes topologiques, notamment le
lemme "un sous groupe ouvert est fermé".

La référence n'est pas une vraie référence, puisqu'il s'agit de la démonstration
plus classique qui utilise la décomposition de jordan. Il faut donc le
connaître. Par contre, il peut être bon de connaître l'image de l'exponentielle
réelle.

+ __153 : Polynômes d’endomorphisme en dimension finie. Réduction d’un
          endomorphisme en dimension finie. Applications.__
+ __156 : Exponentielle de matrices. Applications.__
+ __204 : Connexité. Exemples et applications.__

_Ref : Beck Malick Peyré - Objectif Agrégation - p 213._

17) Somme de deux carrés
------------------------

Un développement simple, facile à retenir, intuitif, agréable.
Le lemme qui figure à la fin est fondamental et _doit_ être connu
par tout agrégatif, même si l'on choisit de ne pas le démontrer
le jours de l'oral.

+ __121 : Nombres premiers. Applications.__
+ __122 : Anneaux principaux. Exemples et applications.__
+ __126 : Exemples d'équations diophantiennes.__

_Ref : Perrin - Cours d'algèbre_


18) Décomposition polaire
-------------------------

Simple, extrèmement bien référencé, facile.

+ __155 : Endomorphismes diagonalisables en dimension finie.__
+ __158 : Matrices symétriques réelles, matrices hermitiennes.__
+ __160 : Endomorphismes remarquables d'un espace vectoriel euclidien
          (de dimension finie).__
+ __171 : Formes quadratiques réelles. Exemples et applications.__

_Ref : Caldero Germoni - H2G2 - p 202, 204._


19) Nombre de solutions d'une équation diophantienne
----------------------------------------------------

Des séries génératrices, de la décomposition en éléments
simples, et une équivalence qui peut facilement etre donnée
comme un développement asymptotique (on a un grand O).

Simple et de bon recasages, bien référencé.

+ __124 : Anneau des série formelles. Applications.__
+ __126 : Exemples d'équations diophantiennes.__
+ __140 : Corps des fractions rationnelles à une indéterminée sur un corps
          commutatif. Applications.__

_Ref : Gourdon Analyse - Exercice 7 p 249._


20) Théorème de Molien
----------------------

C'est un résultat qui lie la dimension d'algèbres invariantes sous une
action aux séries fractions rationelles. C'est un développement
interessant pour la leçon "représentations" puisque l'on montre
que l'opérateur de moyenne est un projecteur sur l'espace fixe.

+ __107 : Représentations et caractères d'un groupe fini sur un
          $\mathbb{C}$-espace vectoriel.__
+ __109 : Représentations de groupes finis de petit cardinal.__
+ __124 : Anneau des série formelles. Applications.__
+ __142 : Algèbre des polynômes à plusieurs indéterminées. Applications.__
+ __151 : Dimension d’un espace vectoriel (on se limitera au cas de
          la dimension finie). Rang. Exemples et applications.__
+ __152 : Déterminant. Exemples et applications.__
+ __154 : Sous-espaces stables par un endomorphisme ou une famille
          d’endomorphismes d’un espace vectoriel de dimension finie.
          Applications.__

_Ref : Gabriel Peyré - Algèbre discrète de la transformé de Fourier._


21) Jordan : Classes de similitudes des nilpotentes via les tableaux de Young
-----------------------------------------------------------------------------

On démontre le théorème de jordan (pour l'unicité, on est ramené à montrer que
deux formes réduites de jordan ne sont pas dans la même orbite, ce qui est
évident, et laissé au lecteur ;) ) en construisant une base où la matrice
est sous forme réduite.
L'idée est de regarder comment se comporte les noyeaux itérés, et fait
aparaitre l'aspect "cyclique" des bloques observés. Un peu long,
mais un développement excellent et qui permet de vraiment comprendre
le théorème de Jordan.

+ __150 : Exemples d’actions de groupes sur les espaces de matrices.__
+ __151 : Dimension d’un espace vectoriel (on se limitera au cas de
          la dimension finie). Rang. Exemples et applications.__
+ __153 : Polynômes d’endomorphisme en dimension finie. Réduction d’un
          endomorphisme en dimension finie. Applications.__
+ __154 : Sous-espaces stables par un endomorphisme ou une famille
          d’endomorphismes d’un espace vectoriel de dimension finie.
          Applications.__
+ __157 : Endomorphismes trigonalisables. Endomorphismes nilpotents.__

_Ref : Caldero Germoni - H2G2 - p 88, 93._


22) Méthodes itératives (système linéaire)
------------------------------------------

On partage une matrice $A \in GL_ n(\mathbb{R})$ en $A = M - N$ avec $M$
inversible (plus facile à inverser que A). On construit alors une
méthode itérative qui converge si et seulement si le rayon spectrale de
$M^{-1}N$ est strictement inférieur à 1.

La démonstration repose sur le lemme qui permet de faire tendre une triangulaire
vers une diagonale par changement de base (celui du classique développement
"classes de similitude").

+ __157 : Endomorphismes trigonalisables. Endomorphismes nilpotents.__
+ __162 : Systèmes d'équations linéaires ; opérations, aspects algorithmiques
          et conséquences théoriques.__
+ __232 : Méthodes d’approximation des solutions d’une équation
          $F(X) = 0$. Exemples.__

_Ref : Dumas Laurent - Modélisation Calcul Scientifique - p 168._


23) Groupe des isométries du cube
---------------------------------

Un jolie développement où l'on utilise des générateurs pour montrer
la surjectivité d'un morphisme. Pas très difficile, et bien référencé,
j'aime beaucoup.

+ __101 : Groupe opérant sur un ensemble. Exemples et applications.__
+ __104 : Groupes finis. Exemples et applications.__
+ __105 : Groupe des permutations d’un ensemble fini. Applications.__
+ __108 : Exemples de parties génératrices d’un groupe. Applications.__
+ __161 : Isométries d’un espace affine euclidien de dimension finie.
          Applications en dimensions 2 et 3.__
+ __181 : Barycentres dans un espace affine réel de dimension finie,
          convexité. Applications.__
+ __183 : Utilisation des groupes en géométrie.__

_Ref : Caldero Germoni - H2G2 - p 364, 365._

24) Lemme de Farkas
---------------------------------

Long, loin d'être simple, garre aux trous de mémoire.
Même si le résultat et la démonstration est sympatique, je suis
content de ne pas avoir eu à le préparer le jour de l'oral.

+ __159 : Formes linéaires et hyperplans en dimension finie.
          Exemples et applications.__
+ __162 : Systèmes d'équations linéaires ; opérations, aspects algorithmiques
          et conséquences théoriques.__
+ __181 : Barycentres dans un espace affine réel de dimension finie,
          convexité. Applications.__

_Ref : Francinou Gianella Nicolas - Algèbre 3._

25) Topologie des classes de similitude
---------------------------------------

Simple, classique, et inintéressant. Mais utile pour boucher les
derniers trous dans les developpements. Si l'on fait "Méthodes itératives",
on fait d'une pierre deux coups.

+ __155 : Endomorphismes diagonalisables en dimension finie.__
+ __157 : Endomorphismes trigonalisables. Endomorphismes nilpotents.__

_Ref : FGN ?._

26) Simplicité par la table des caractères
------------------------------------------

Résultat très utile (c'est grace à ça que l'on savais que le monstre est
simple avant même de savoir qu'il existait), dont la démonstration utilise une
propriétée fondamentale des représentations d'un groupe fini sur $\mathbb{C}$
et illustre le fait de "remonter" une représentation (ce qui sert,
par exemple, à construire la table de $A_4$ à partir de celle
de $\mathbb{Z}/3\mathbb{Z}$).

+ __103 : Exemples et applications des notions de sous-groupe distingué
          et de groupe quotient.__
+ __104 : Groupes finis. Exemples et applications.__
+ __107 : Représentations et caractères d'un groupe fini sur un
          $\mathbb{C}$-espace vectoriel.__
+ __109 : Représentations de groupes finis de petit cardinal.__

_Ref : Gabriel Peyré - Algèbre discrète de la transformé de Fourier._


A) Nullstellensatz (Théorème des zéros de hilbert)
--------------------------------------------------

On démontre l'énnoncé faible puis le fort. La démonstration n'est
valable que sur un corps undénombrable, donc dites adieux aux clotures
algébriques de corps fini. Comme ses recasages sont strictement inclus
dans le nullstellensatz par le résultant, il se retrouve abandonné en annexe.

+ __144 : Racines d’un polynôme. Fonctions symétriques élémentaires.
          Exemples et applications.__

_Ref : Perin - Géométrie Algébrique._

--------------------------------------------------------------------------------


Développements analytiques
==========================

Disclaimer : Je fait une impasse totale sur les probas.

01) Thérorème de Borel et application
-------------------------------------

Un bon développement, même si il faut se souvenir de la majoration.

Plutot simple, et tout est dans le gourdon.

+ __201 : Espaces de fonctions : exemples et applications.__
+ __207 : Prolongement de fonctions. Exemples et applications.__
+ __228 : Continuité et dérivabilité des fonctions réelles d’une variable
          réelle. Exemples et contre-exemple.__
+ __241 : Suites et séries de fonctions. Exemples et contre-exemples.__

_Ref :  Xavier Gourdon - Analyse._


02) Base Hilbertienne de polynômes
----------------------------------

Ce théorème donne une condition suffisante pour que les polynômes
orthogonaux associés à une fonction poids $\rho : I \Rightarrow \mathbb{R}$
soient dense (i.e. forment une base Hilbertienne) dans $L^2(I, \rho)$.

Cela vaut (sur $I = ]-1;1[$) pour les polynômes de Legendre ($\rho = 1$)
et Tchebychev ($\rho = \frac{1}{\sqrt{1-x^2}}$).

Attention à la référence qui est plutot mauvaise.

+ __201 : Espaces de fonctions : exemples et applications.__
+ __202 : Exemples de parties denses et applications.__
+ __207 : Prolongement de fonctions. Exemples et applications.__
+ __209 : Approximation d’une fonction par des polynômes et des polynômes
          trigonométriques. Exemples et applications.__
+ __213 : Espaces de Hilbert. Bases hilbertiennes. Exemples et applications.__
+ __239 : Fonctions définies par une intégrale dépendant d’un paramètre.
          Exemples et applications.__
+ __240 : Produit de convolution, transformation de Fourier. Applications.__
+ __245 : Fonctions holomorphes et méromorphes sur un ouvert de $\mathbb{C}$.
          Exemples et applications.__

_Ref : Beck Malick Peyré - Objectif Agrégation - p 140, 141._

03) Chemin au dessu d'une courbe concave
----------------------------------------

Un développement où l'on ne peux pas détailler en profondeur tout les
points. La seconde partie n'est pas référencée.

+ __216 : Étude métrique des courbes. Exemples.__
+ __229 : Fonctions monotones. Fonctions convexes. Exemples et applications.__

_Ref : Francinou Gianella Nicolas - Algèbre 3._

04) Méthode de Newton
---------------------

Un développement que l'on meure d'envie de qualifier de "trivial" (tous
es outils utilisé sont élémentaires), mais avec un recasage impressionnant.

On fera quand même attention à une légère subtilité dans une majoration.

+ __206 : Théorèmes de point fixe. Exemples et applications__
+ __218 : Applications des formules de Taylor.__
+ __223 : Convergence des suites numériques. Exemples et application.__
+ __224 : Comportement asymptotique de suites numériques.
          Rapidité de convergence. Exemples.__
+ __226 : Comportement d’une suite réelle ou vectorielle définie par
          une itération $u_{n+1} = f(u_n)$. Exemples.__
+ __232 : Méthodes d’approximation des solutions d’une équation
          $F(X) = 0$. Exemples.__

_Ref : François Rouvière, Petit guide de calcul différentiel._

05) Lemme de Morse
------------------

Classique, passe par la démonstration d'un lemme de réduction
d'une forme quadratique version différentielle. Le gros
soucis est que l'énoncer et la correction du Rouvière ne
correspondent pas au format Agreg. Il faut donc "retenir" (i.e. savoir
retrouver rapidement avec le livre) l'énoncer du lemme, et
les bon espaces tangent.

+ __171 : Formes quadratiques réelles. Exemples et applications.__
+ __214 : Théorème d’inversion locale, théorème des fonctions implicites.
          Exemples et applications.__
+ __215 : Applications différentiables définies sur un ouvert
          de $\mathbb{R}^n$. Exemples et applications.__
+ __217 : Sous-variétés de $\mathbb{R}^n$. Exemples.__
+ __218 : Applications des formules de Taylor.__

_Ref : François Rouvière, Petit guide de calcul différentiel._


06) Inégalité isopérimétrique
-----------------------------

Il faut admettre Stokes en dimension 2 (Green-Riemann), ou bien
la formule de l'aire enclose par une courbe. On pourra regarder
le Amar-Matheron pour une idée de la démonstration de Stokes.
Ceci mis à part, c'est une magnifique application des séries
des Fourier et de la théorie des courbes paramétrées.

+ __216 : Étude métrique des courbes. Exemples.__
+ __219 : Problèmes d’extremums.__
+ __236 : Illustrer par des exemples quelques méthodes de calcul
          d’intégrales de fonctions d’une ou plusieurs variables réelles.__
+ __246 : Séries de Fourier. Exemples et applications.__

_Ref : Queffélec Zuily - Éléments d'Analyse pour l'Agrég. - Inég. Iso. p 101, 102._
_Ref : Faraut - Calcul Intégral - p 178, 179._

07) L'isomorphisme exceptionnel $PSL_ 2(\mathbb{C}) \simeq SO_ 3(\mathbb{C})$
-----------------------------------------------------------------------------

Du calcul différentiel (inversion local) sur des sous variétés, saupoudré
de quaternions, et de formes quadratiques. La recette miracle pour un développement
d'algèbre qui ne se reccase qu'en analyse. Ce genre de petite merveille est tellement
rare, qu'il faut en profiter.

+ __204 : Connexité. Exemples et applications.__
+ __214 : Théorème d’inversion locale, théorème des fonctions implicites.
          Exemples et applications.__
+ __217 : Sous-variétés de $\mathbb{R}^n$. Exemples.__

_Ref : Caldero Germoni - H2G2._

08) Lacunes de Hadamard
-----------------------

Cette théorème montre que si une serie entière a suffisament de trous dans
ses termes, alors il n'existe pas de prolongement analytique hors du disque de
convergence.

Une démonstration astucieuse et plutot bien rédigé dans le QZ (attention à
l'ordre des auteurs, le QZ et le ZQ sont deux livres très différents).

+ __241 : Suites et séries de fonctions. Exemples et contre-exemples.__
+ __243 : Convergence des séries entières, propriétés de la somme.
          Exemples et applications.__
+ __244 : Fonctions développables en série entière, fonctions analytiques.
          Exemples.__
+ __245 : Fonctions holomorphes et méromorphes sur un ouvert de $\mathbb{C}$.
          Exemples et applications.__

_Ref : Qufélec Zuily - Analyse pour l'agrégation - p 54, 55._


09) Billard Elliptique
----------------------

Une très bonne application des extremas liés pour construire
une trajectoire à trois rebond dans une ellipse.

C'est en fait un résultat valable dans n'importe quel "billard convexe".
L'avantage de cet énoncé est de mettre de coté les subtilités
relatives aux (sous-)variétés à bord.
Le développement appelle la question "où utilisez vous que c'est
une ellipse", et la réponse est "nul part". La convexité est
"facultative", dans le sens où elle ne sert qu'à imposer que la
trajectoire à trois rebond reste dans le billard (un hypothèse
somme toute raisonnable).

+ __215 : Applications différentiables définies sur un ouvert
          de $\mathbb{R}^n$. Exemples et applications.__
+ __217 : Sous-variétés de $\mathbb{R}^n$. Exemples.__
+ __219 : Problèmes d’extremums.__

_Ref : François Rouvière, Petit guide de calcul différentiel._


10) Formule sommatoire de Poisson
---------------------------------

Un développement que je n'aime pas du tout, mais il faut savoir
faire des sacrifices.

+ __247 : Exemples de problèmes d’interversion de limites.__
+ __230 : Séries de nombres réels ou complexes. Comportement des restes ou
          des sommes partielles des séries numériques. Exemples.__
+ __235 : Suites et séries de fonctions intégrables.
          Exemples et applications.__
+ __239 : Fonctions définies par une intégrale dépendant d’un paramètre.
          Exemples et applications.__
+ __246 : Séries de Fourier. Exemples et applications.__

_Ref : Gourdon._

11) Transformée de Fourier-Plancherel
-------------------------------------

Un développement qui demande du travail, et montre un théorème très utile.
On admet le résultat $L^1$ pour se concentrer sur la démonstration que
la transformée restreinte à $L^1 \cap L^2$ est une isométrie, afin de la
prolonger par uniforme continuité. J'utilise le noyeau gaussien, car
c'est aussi celui que j'utilise pour la transformé de fourier dans
$\mathcal{S}$.

On remarquera que l'on fait une convergence monotone, et que l'on ne
peux pas appliquer une convergence dominé.

+ __207 : Prolongement de fonctions. Exemples et applications.__
+ __208 : Espaces vectoriels normés, applications linéaires continues.
          Exemples.__
+ __234 : Espaces $L^p, 1 \le p \le +\infty$.__
+ __240 : Produit de convolution, transformation de Fourier. Applications.__

_Non référencé._

12) Densité de $\mathcal{D}$ dans $L^p$.
----------------------------------------

Un développement court et plutot simple qui utilise la convolution
(approximation de l'unité), fubini, et hölder. Très bien référencé.

+ __234 : Espaces $L^p, 1 \le p \le +\infty$.__
+ __235 : Suites et séries de fonctions intégrables.
          Exemples et applications.__
+ __240 : Produit de convolution, transformation de Fourier. Applications.__

_Ref : Faraut - Calcul Intégral - p 121, 123._

12) Banach-Steinhauss et fonction dont la série de Fourier diverge
------------------------------------------------------------------

Démonstration du théorème de Banach-Steinhauss (qui utilise Baire) puis
une application à la construction d'une fonction dont la série de
Fourier diverge.
Il faut savoir qu'une légère modification de l'énoncer permet d'avoir un
$G_\delta$ dense de fonctions de ce type. La démonstration étant casi
identique, mieux vaut savoir le faire.

Je n'ai pas de référence pour l'application.

+ __208 : Espaces vectoriels normés, applications linéaires continues.
          Exemples.__
+ __205 : Espaces complets. Exemples et applications.__
+ __201 : Espaces de fonctions : exemples et applications.__
+ __246 : Séries de Fourier. Exemples et applications.__

_Ref : Brezis - p 16._


13) Système de Lotka - Voltera
------------------------------

Un grand classique. Le jury veux voir de l'étude qualitative.
A moins que vous ne comptiez faire
Hadamard-Lévi, c'est la seul étude qualitative non trivial que je connaisse.

On s'intéresse au premier quadrant du plan. On regarde ce qui se produit sur
les bords, on parachute l'intégrale première, on divise le quart de plan en
quatre zone autour du point stable (corespondant aux quatres phases
de la "rotation") et l'on n'étudie qu'un morceau (tous se traitent de la même
façon). On conclut pour la périodicité en regardant le temps de premier retour,
et grace à l'intégrale première.

+ __220 : Équations différentielles $X' = f(t,X)$.
          Exemples d’études qualitatives des solutions.__

_Ref : Francinou Gianella Nicolas - Analyse 4._


14) Résolution d'une equation matricielle grace aux EDL
-------------------------------------------------------

Ne se recase pas très bien, mais c'est un développement facile
pour EDL, et en forçant un peu il se recase dans EDO.

+ __220 : Équations différentielles $X' = f(t,X)$.
          Exemples d’études qualitatives des solutions.__
+ __221 : Équations différentielles linéaires. Systèmes d’équations
          différentielles linéaires. Exemples et applications.__

_Gourdon Analyse - Problème 7 p 384._


15) Théorème Taubérien Fort
---------------------------

Un théorème d'interversion de limite pour une série entière qui
"n'explose pas trop vite en 1".
Long, très long. On peux sûrement gagner du temps dans la rédaction
en posant des notations de la forme $S(\phi)(x) = \sum x^n \phi(x^n)$.

Bien faire le dessin, qui permet à la fois de comprendre ce que l'on fait,
et d'expliquer au jury comment vous construisez vos fonctions continues
à l'oral.

Si vous êtes une fusée, vous pouvez rajouter le lemme 1.

+ __209 : Approximation d’une fonction par des polynômes et des polynômes
          trigonométriques. Exemples et applications.__
+ __230 : Séries de nombres réels ou complexes. Comportement des restes ou
          des sommes partielles des séries numériques. Exemples.__
+ __243 : Convergence des séries entières, propriétés de la somme.
          Exemples et applications.__
+ __247 : Exemples de problèmes d’interversion de limites.__

_Gourdon Analyse - Problème 20 p 284, 286._


16) Kakutani-Massera
--------------------

Un théorème de point fixe dans un compacte convexe, suivit d'une application
aux équations différentielles linéaires. Connus et reconnus pour ses recasages.

+ __203 : Utilisation de la notion de compacité.__
+ __206 : Théorèmes de point fixe. Exemples et applications__
+ __221 : Équations différentielles linéaires. Systèmes d’équations
          différentielles linéaires. Exemples et applications.__
+ __253 : Utilisation de la notion de convexité en analyse.__

_Ref : Gonnor et Tosel - Topologie p 77, Calcul Diff. p 121._


17) Optimisation dans un espace de Hilbert
------------------------------------------

Un développement très agréable d'analyse fonctionnelle dans un espace
de espace de Hilbert. Il est question de montrer qu'un minimum existe.

L'idée est que, à défaut de pouvoir faire converger une suite
minimisante dans $H$, on peut injecter la suite dans
le dual topologique $H^*$ via le produit Hermitien.
On la fait alors converger faiblement (sans le dire), puis l'on
revient dans $H$ via le théorème de représentation de Riesz.
On vérifie alors, grâce à la caractérisation de la projection
sur un convexe fermé, que notre candidat convient.

On pouras remarquer que l'essentielle de la démonstration revient
à montrer qu'un convexe fortement fermé est faiblement fermé (si si,
regardez bien les $C_\alpha$).

Attention, la fin n'est pas celle de la référence.

+ __205 : Espaces complets. Exemples et applications.__
+ __213 : Espaces de Hilbert. Bases hilbertiennes. Exemples et applications.__
+ __219 : Problèmes d’extremums.__
+ __223 : Convergence des suites numériques. Exemples et application.__
+ __229 : Fonctions monotones. Fonctions convexes. Exemples et applications.__
+ __253 : Utilisation de la notion de convexité en analyse.__

_Ref : Ciarlet - Intro. Analyse Num., Matri., Opti. - Ch 8, p 176._


18) Inversion de Fourier dans $\mathcal{S}(\mathbb{R})$.
--------------------------------------------------------

C'est, grosso modo, la transformé de fourier $L^1$ (quelques arguments
changent car on considère $\mathcal{S}$). J'utilise le
noyeau gaussien, cf Fourier-Plancherel.

+ __254 : Espaces de Schwartz ($\mathcal{S}(\mathbb{R}^d)$) et distributions
          tempérées. Transformation de Fourier dans $\mathcal{S}(\mathbb{R}^d)$
          et $\mathcal{S}'(\mathbb{R}^d)$.__
+ __255 : Espaces de Schwartz. Distributions. Dérivation au sens des
          distributions.__

_Ref : ???._

19) Equation différentielle sur $\mathcal{S}'$.
-----------------------------------------------

Bon, ne le cachons pas, on sais résoudre cette equation depuis la terminale
(sur $\mathbb{R}+$ et $\mathbb{R}-$, puis on recole). Du coup, on vas le faire
d'une façon inutilement compliquer.

Il faut le voir comme une illustration de la recherche d'une solution élémentaire,
et d'une méthode général des distributions.

Un petit calcul de résidus tout gentil montre le bout de son nez, ce qui nous permet
de recaser le développement dans calculs d'intégrales.

+ __254 : Espaces de Schwartz ($\mathcal{S}(\mathbb{R}^d)$) et distributions
          tempérées. Transformation de Fourier dans $\mathcal{S}(\mathbb{R}^d)$
          et $\mathcal{S}'(\mathbb{R}^d)$.__
+ __255 : Espaces de Schwartz. Distributions. Dérivation au sens des
          distributions.__

_Non référencé._


A) Courbe à courbure croissante.
--------------------------------

Un exercice de calcul diff, qui dit (en gros) que les disques
osculateurs à une courbe dont la courbure croit forment un anneau.

Ce développement a ses recasages strictement inclus dans
"Chemin au dessu d'une courbe concave", d'où son abandon.

+ __216 : Étude métrique des courbes. Exemples.__

_Non référencé._


B) Morgenster (Fonction nul part analytique)
---------------------------------------------

Une application de Baire amusante. Plus simple que les fonctions
non dérivables, mais non référencé.

+ __201 : Espaces de fonctions : exemples et applications.__
+ __202 : Exemples de parties denses et applications.__
+ __205 : Espaces complets. Exemples et applications.__
+ __228 : Continuité et dérivabilité des fonctions réelles d’une variable
          réelle. Exemples et contre-exemple.__
+ __243 : Convergence des séries entières, propriétés de la somme.
          Exemples et applications.__

_Non référencé._
