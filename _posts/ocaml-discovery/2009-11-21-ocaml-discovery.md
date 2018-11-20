---
layout: post
title: Découverte d'OCAML
description: Une courte introduction au language de programmation OCAML.
author: Jérémy Cochoy
date: 2009-11-21 +0100
categories: programming functional ocaml language
lang: fr
...

## Découverte d'OCAML

Dans ce petit article, je compte vous parler d'un langage de programmation assez particulier :  (O)CAML. Tout d'abord, si j'ajoute une parenthèse devant la première lettre du sigle, c'est que je ne compte pas aborder toute la partie OBJET de langage, ni la partie impérative. N'espérez donc pas voir une seul structure conditionnel (_if_) ou itérative (_while_/_for_) dans ce texte.

## Propositions : définitions et expressions :

Tout comme en mathématique chaque phrase est une proposition, en ocaml chaque "ligne" est une proposition. Elle commence avec le premier caractère et se termine avec le symbole ';;'

Il y a deux type de propositions :
 *  Les expressions : De la forme $$3*7$$ ou encore $$\sqrt[7]{\frac{2^9}{3^{43}}}$$
 *  Les définitions : De la forme $$q := \sqrt{2}$$ qui déclare q comme un alias de $$\sqrt{2}$$

En ocaml, la syntaxe est la suivante :
 *  Pour un expression : 
    ``` ocaml
    3 * 7;;
    ```
    ou encore (pour $$e^{log(\frac{exp(log(2) * 9)}{exp (log(3) * 43)}) / 7}$$)
    ``` ocaml
    exp (log (exp(log 2. *. 9.) /. exp(log 3. *. 43.) ) /. 7.);;
    ```
 *  Enfin, une définition :
    ``` ocaml
	  let q = sqrt 2;;
	  ```

## L'aspect fonctionnel

Mais comment un tel langage peut-il exister, et surtout, être 'utilisable' sans pour autant faire usage des structures des langages impératif comme le C ou le Python? (Je m'adresse bien-sur ici a ceux qui n'ont pas encore été initier aux joies des langages fonctionnel)

Tout d'abord, définissons ce qu'est un langage fonctionnel. Par analogie à un langage objet où les éléments sont des objets(Dans certains cas, TOUS les éléments), dans un langage fonctionnel, les éléments sont des fonctions, au sens mathématique du terme.

Si l'on considère la fonction suivante : $$\begin{array}{ccccc} f & : & \mathbb{Z} & \to & \mathbb{Z}^2 \\ & & x & \mapsto & (x, x) \end{array}$$

C'est la fonction qui associe à un entier $$x$$ son couple situer sur sa diagonal(injection diagonal) $$(x,x) \in \Delta\mathbb{Z} \subset \mathbb{Z}^2$$

Il est alors très aisé de définir f en ocaml :
```{.ocaml}
let f x = (x, x);
```
Cette fonction est de type `int -> int * int`, c'est a dire qu'elle prend un entier $$int \approx \mathbb{Z}$$ et retourne un couple $$(x, y) \in \mathbb{Z} \times \mathbb{Z}$$.

On peut bien-sur définir des fonctions de fonction. Observons la fonction "composition"(rond) : $$\begin{array}{ccccc} o & : & (\mathbb{B} \Rightarrow \mathbb{C} ) \times (\mathbb{A} \Rightarrow \mathbb{B} ) & \longrightarrow & (\mathbb{A} \to \mathbb{C} ) \\ & & (f, g) & \longmapsto & (x \mapsto f(g(x)) \end{array}$$.

En OCAML, on aurais :
``` ocaml
let o f g = f g;;
(* val o : ('a -> 'b) -> 'a -> 'b = <fun> *)
```

## Les filtres

Je vous ai parler un peut plus tôt de l'absence de structures conditionnel et itératif, je vais ici vous exposer les solutions a votre disposition.

Commençons avec les blocs conditionnel. En OCAML, nous disposons d'une fonction-alitée très intéressante : les filtres.
Si l'on considère la fonction continue suivante : $$\begin{array}{ccccc} f & : & \mathbb{Z} & \to & \mathbb{Z} \\ & & x & \mapsto &
\left\{
\begin{array}{ccc}
x < 0 & \Rightarrow & -2x \\ x \ge 0 &  \Rightarrow & x^2
\end{array}
\right.
\end{array}$$

On constate que l'on a bien deux condition, une première qui regroupe les cas des x négatif, et une deuxième qui s'intéresse aux x restant. En OCAML, on défini de la même façon :
``` ocaml
let f = function
  | x when x < 0 -> -2 * x
  | x when x >= 0 -> x*x
  ;;
```
Ou bien, en omettant la deuxième condition et en considérons que la dernière condition matcheras pour "tous les cas restant"
``` ocaml
let f = function
  | x when x < 0 -> -2 * x
  | x -> x*x
  ;;
```

Dans le cas où l'on a pas besoin de nommer x, on peut utiliser _. Ainsi, un dernier cas `_ -> smtg` correspondrais un peut à 'default' d'un switch.
On pourras aussi utiliser l'instruction match {variable} with {filtre}.

## La récursivité terminal

Maintenant, considérons les itérations. Nous allons utiliser la récursivités terminal(tail-rec pour les intimes) affin de construire une boucle. La récursivitée terminal est une forme de récursivité ou l'unique action qui succède à l'appelle a la fonction récursive est un retour de valeur.

C'est plutôt élémentaire : 
``` ocaml
let rec aff_n = function
  | n when n < 1 -> ();
  | n -> print_endline (string_of_int n);
           aff_n (n - 1);;
```
() représente le type 'unit', c'est a dire que notre fonction ne retourne rien, tout comme print_endl. Nous obtenons une boucle de 10 à 1, et nous affichons les valeurs a l'écran.
Vous vous demandez alors, pourquoi de la tail rec et non une simple récursivités? La réponse est que l'interpréteur/compilateur optimise par une itération, ce qui est bien plus pratique pour nos machines qui on une mémoire limiter et ne peuvent supporter qu'un nombre restreins d'appelles récursif.

## Pour conclure

OCAML est un langage fonctionnel, qui bon nombre d'aspects et concepts qui sont parfois inconnus des programmeurs aillant pratiquer uniquement des langages impératif, et constitue de par son existence une forme de culture dont il peut-être bon d'avoir connaissance.

### Références :

Si vous êtes curieux et désirez en savoir plus, voici quelques liens :

 *  [Présentation d'ocaml](http://www.iie.cnam.fr/~dubois/presentation_caml.pdf)
 *  [Notes on OCaml](http://www.csc.villanova.edu/~dmatusze/resources/ocaml/ocaml.html)
 *  [Wikipedia](http://fr.wikipedia.org/wiki/Objective_Caml)
 *  [Google](http://www.google.fr/search?&q=Tuto+Ocaml)
