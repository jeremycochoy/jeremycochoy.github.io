---
layout: post
title: Le Haskell, un langage au label pure. Première partie.
description: Inroduction à haskell (partie 1). On discute curryfication, typage, et pattern matching.
author: Jérémy Cochoy
date: 2013-04-07 +0100
categories: programming haskell language software
lang: fr
redirect_from:
  - /haskell-pure-1/fr.htm
  - /haskell-pure-1/
...

Mauvais jeux de mots mit à part, ce très court et succin billet (vous me voyez venir) va traiter du haskell. Haskell, de monsieur Haskell Brook Curry, un monsieur très épicé, est un langage fonctionnel. Comme beaucoup de langages fonctionnel, cela représente un choque culturel que de les appréhender. Mais le haskell semble avoir un petit quelque chose que n'ont pas les autres langages fonctionnels. À travers les trois prochains billets, j'espère vous convaincre que, si un jour, dans votre vie, vous avez l'occasion de creuser un language fonctionnel, alors vous devriez creuser le haskell.

La [seconde partie est disponible ici](http://zenol.fr/site/blog/posts/haskell-pure-2/fr.htm "Le Haskell, un langage au label pure. Seconde partie.").

On y vas tout en douceur. Curryfication et typage, pour ceux qui n'ont pas déjà découvert les joies (les dolipranes?) qu'apportent ces derniers. Si vous avez déjà eu à toucher à des langages fonctionnels, vous souhaiterez sûrement passer directement à la seconde partie où j'aborderais les diverses fonctionnalités original du haskell, comme les fameuses liste infinies dont tout le monde parle, mais dont trop peu se doutent de l'utilité. On parleras de data driven programming, de fonctions sans effets de bord et de parallélisation des calculs en haskell (avec un VRAI exemple!). Enfin, apogée, moment de transe et de plaisir inégale, on abordera la huitième merveille du monde : les monades.

## Ce que tout le monde sais déjà... ou pas.

Faisons un petit retour sur ce que l'on retrouve dans tous les langages fonctionnels, et ce qui les rend si différents.

Histoire de fixer la syntaxe, une fonction qui prend deux nombres et retourne leur produit s'écrit :
```haskell
mult :: (Num n) => n -> n -> n
mult a b = a * b
```
Une fonction qui ajoute 42 à un entier :
```haskell
addOne :: (Num n) => n -> n
addOne a = a + 42
```

La première ligne est le typage de la fonction (souvent facultatif), et la seconde est le corps de la fonction. En C++, ça donnerais :
```c++
template <class T>
T mult(T a, T b)
{
  return a * b;
}

T addOne(T a)
{
  return a + 42;
}
```

En fait, le `n` qui apparaît dans le type peut être remplacé par n'importe quel lettre/mot qui vous plaît. Cela signifie un type quelconque, qui respecte la contrainte "_peut être multiplié_" décrite par `(Num n) =>`.

La présence de `Num` correspond à la nécessiter d'avoir, dans le code c++, une surcharge de l'opérateur *. Remarquez qu'une fonction en haskell correspond naturellement a une fonction template, à moins de forcer les types à être moins "puissant", comme par exemple :
```haskell
mult :: Int -> Int -> Int
mult a b = a * b
```

Qui ne sais plus que multiplier des entiers.

### Curryfication

La curryfication est essentiellement un changement de point de vue. On peux considérer une fonctions prenant deux nombres, et retournant une valeur. Mais on peut aussi changer de point de vue et considérer une fonction prenant un nombre, et renvoyant une nouvelle fonction, prenant un nombre, et retournant une valeur.

Cela explique l'étrange notation pour le type de la fonction mult. En fait, l'opérateur "->" est associatif à droite, c'est à dire qu'il fallait lire :
```haskell
mult :: (Num n) => n -> (n -> n)
```

Où `X -> Y` indique que la fonction prend du type X et retourne du type Y. Ainsi, `n -> n` signifie prend un type n quelconque et renvoi quelque chose du même type. C'est le type d'une fonction. Si l'on veux un exemple plus concret, `Int-> (Float->Double)` signifie une fonction qui prend un entier, puis renvoi une nouvelle fonction. Cette dernière prend un float et le transforme en un double.

Partant de ce point de vue, il est très simple de fixer les premiers arguments d'une fonction curryfiée. L'application d'un élément à un autre étant associatif à gauche, il suffit de l'appeler avec une partie de ses arguments. (Et oui, puisque écrire `mult a b` signifie alors `((mult a) b)`. C'est à dire appliquer `mult` à `a` pour obtenir une fonction qui "_prend un entier et le multiplie par a_", puis évaluer cette fonction sur l'entier b.)

Par exemple, `nouvelle_fonction = mult 42` correspond à fixer la première valeur de la fonction `mul` à `42`.

En C++, il faudrais écrire
```c++
template<class T>
T nouvelle_fonction(T b)
{
  return mult(42, b);
}
```

Pour finir, puisque l'on est en plein dans la manipulation de fonctions, on peut écrire une fonction comme une valeur, et la stocker dans une "variable".
```haskell
-- Pour écrire une fonction "anonyme", qui prend trois arguments et renvoi 42, on peut écrire
-- (\x y z -> 42)
-- Où x y z seront les trois arguments, et ce qui suit -> est la "valeur de retour".
-- On aurais donc pu écrire la fonction mult de la façon suivante :
mult = (\a b -> a * b) :: (Num n) => n -> n -> n

--On peut aussi écrire \a -> \b -> à la place de \a b, grâce à la curryfication.
```
Le typage est bien sur facultatif.

On a alors deux petites fonctions bien pratique pour manipuler des fonctions ; l'identité et la composition :
```haskell
id = \x -> x
a . b = \x -> a (b x)
```

On peut alors composer la multiplication par 2 et l'addition à 3 :
```haskell
add a b = a + b
superFct = mult 2 . add 3
```

Enfin, on aurais pu écrire :
```haskell
mult = (*)
add = (+)
superFct = (2*) . (3+)
```

### Typage

Les types, en langage fonctionnel, disent beaucoup de choses, et sont lourd de sens.
En haskell, une liste d'entiers se note `[Int]`. Si `a` est un type, alors une liste de `a` est de type `[a]`. Les chaînes de caractères sont par exemple des `[Char]`.

Alors, que devrait faire une fonction de type `(a -> b) -> [a] -> [b]`? Et bien, elle prend une fonction _transformant du a en b_ que l'on appellera f, une _liste de a_, et produit une _liste de b_. Le bon sens veux que cette fonction applique f à chaque élément de la _liste de a_, pour produire une _liste de b_.

Encore un, pour la route. Que dire du type `[[a]] -> [a]`? On prend une _liste de liste_, et on produit _une liste_. La première chose qui nous vient à l'esprit est de concaténer toute ces listes les unes à la suite des autres pour n'en produire plus qu'une.

Voyez tout ce qu'on peut deviner sur le comportement d'une fonction grâce aux types.

Pour éviter les noms à rallonge, on peu aussi construire des alias de type. Plutôt que décrire `[Char]`, on peut écrire `String`, définit par :
```haskell
type String = [Char]
```

C'est très utile pour annoter une fonction. Par exemple, on peut imaginer le scénario suivant :
```haskell
type Nom = String
type Prenom = String
type Identifiant = Int

getSomeone :: Nom -> Prenom -> Identifiant
```

Grâce à ces alias, vous avez une idée claire du comportement de la fonction getSomeone simplement en lisant son type.

Mais le meilleur est à venir : les constructeurs de type.
Attention, un constructeur en haskell est tout sauf un constructeur C++. Ça se rapprocherais plutôt de la structure, et encore...

Pou faire bref, les constructeur permettent de fabriquer des instances d'un type. Considérons quelques types :
```haskell
data Personne = ConstructeurDePersonne Nom Prenom Identifiant
data Reponse = Oui | Non
data LaValeur = Entier Int | Flotant Float
data ListeEntier = Element Int Liste | Vide
data ArbreEntier = Noeud Int Arbre Arbre | Vide
```

Le premier type, `Personne`, est une structure à trois champs. Le second, est un type booléen, qui peut soit être construit par le constructeur _Oui_, soit par le constructeur _Non_. Le troisième est une union pouvant contenir un `Int` ou un `Float`, et les derniers sont respectivement un type liste d'entier et arbre binaire d'entier. Le système de types permet de considérer des objets plus proche des données représentée, et de s'abstraire de l'implémentation.

Donnons une correspondance C++ des deux derniers exemples dans leur version "polymorphique", où Liste et Arbre deviennent des constructeurs de type, et prennent donc en paramètre le type qu'ils doivent contenir.
```haskell
data Liste a = Element a Liste | Vide
data Arbre a = Noeud a Arbre Arbre | Vide

type ListeEntier = Liste Int
type ArbreEntier = Arbre Int
```

L'équivalent C++ :
```haskell
template <class T>
struct Liste {
  union {
    struct Element {
      T value;
      struct List *next;
    } firstConstructor;
    struct Vide {
    } secondConstructor;
  };
};

template <class T>
struct Arbre {
  union {
    struct Noeud {
      T value;
      struct Arbre *left;
      struct Arbre *right;
    } firstConstructor;
    struct Vide {
    } secondConstructor;
  };
};
```

Un petit exemple d'utilisation :
```haskell
personnage = ConstructeurDePersonne "Dent" "Arthur" 42
estCeVrai = Oui
valeur = Entier 42
uneListe = Element 1 ( Element 2 ( Element 3 ( Element 4 Vide ) ) )
racine = Noeud 4 (Noeud 2 Vide Vide) (Noeud 6 Vide Vide)
```

### Patern matching

Si l'on peut construire des types, on peut aussi les détruire, où plus précisément, les dès-construire. Partant d'un objet de type liste, on peut vouloir récupérer le première élément, si la liste n'est pas vide. Cela se fait en faisant matcher un pattern sur l'instance d'un type :

```haskell
obtenirNom (ConstructeurDePersonne nom _ _) = nom

obtenirPrenom (ConstructeurDePersonne _ prenom _) =  prenom

obtenirID (ConstructeurDePersonne _ _ id) = id

recupererValeur (Entier v) = v

premierElement (Element v sousListe) = v
premierElement Empty = 0
```

Il est aussi possible de déconstruire un objet au milieu d'une fonction, sous réserve d’être certain que l'objet respecte le pattern spécifié :
```haskell
recupererValeur' truc = let (Entier v) = truc in v
```

__Nb :__ Le caractère ' est un caractère valide comme un autre pour un nom de fonction ou de variable (bien qu'en fait, en haskell, il n'existe pas de variable au sens usuel).

Voilà, ce seras tout pour aujourd'hui :)

Références :
------------

  * Quelques détails sur les constructeurs : <http://www.haskell.org/haskellwiki/Constructor>
