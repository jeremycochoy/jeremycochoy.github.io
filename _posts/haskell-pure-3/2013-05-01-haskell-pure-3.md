---
layout: post
title: Le Haskell, un langage au label pure. Troisième partie.
description: Inroduction à haskell (partie 2). On discute
  foncteurs aplicatifs, et monades.
author: Jérémy Cochoy
date: 2013-05-01 + 0100
categories: programming haskell language software
lang: fr
...

Et voici la troisième et dernière partie de notre découverte de ce joli langage. Au programme : des super foncteurs. D'abord des foncteurs applicatifs, puis des monades! Si vous rêviez de savoir pourquoi tant de gens s’enflamment devant ce langage, il est peut-être venue l'heure de la révélation.

La [première partie est ici](http://zenol.fr/site/blog/posts/haskell-pure-1/fr.htm "Le Haskell, un langage au label pure. Première partie."), [la seconde là](http://zenol.fr/site/blog/posts/haskell-pure-2/fr.htm "Le Haskell, un langage au label pure. Seconde partie.").

## Foncteurs applicatifs

Les foncteurs applicatifs sont un enrichissement des foncteurs. C'est donc une classe, définie dans le module `Control.Applicative`, de la façon suivante :
``` haskell
class (Functor f) => Applicative f where
   pure  :: a -> f a
   (<*>) :: f (a -> b) -> f a -> f b
```

Avec un type fonctoriel Fonc, on pouvais :
  * Prendre un élément de type `c` et le transformer en `Fonc c`, c'est à dire le "placer dans un contexte minimal".
  * Prendre une fonction de type `a->b` et la transformer en une fonction de type `Fonc a -> Fonc b` grâce à `fmap`.

Grace à un foncteur applicatif, on peut :
  * Prendre un élément de type <i>c</i> et le transformer en type `Fonc c`, cela se fait grâce à la fonction `pure :: (Applicative f) => c -> f c`.
  * Prendre une fonction dans un foncteur (ie un élément de type `Fonc (a->b)`) et l'appliquer à un élément de type `Fonc a` pour produire un `Fonc b`. L'opérateur permettant une telle chose est noté `<*> :: (Applicative f) => f (a -> b) -> f a -> f b`.

La seconde notion est plus générale, car si vous avez un constructeur de type `Fonc` qui est une instance de `Applicative` (La classe des foncteurs applicatifs), alors c'est un foncteur de la façon suivante :
``` haskell
instance Functor Fonc where
   fmap f e = (pure f) <*> e
```

C'est à dire : Prenez f une fonction de type `a -> b`, alors vous savez comment l'appliquer sur un `Fonc a`. Il suffit de la "placer" elle aussi dans un foncteur, pour se retrouver avec une "fonction dans un contexte" de type `Fonc (a->b)` à appliquer sur un "élément dans un contexte" de type `F a`.

C'est donc pour cela que l'on impose à tout foncteur applicatif d'être d'abord une instance de `Functor`, et que vous avez la contraire `"Functor f"` dans la définition de la classe `Applicative`.

### On se prépare au grand saut avec `Maybe`

Tout ceci est très abstrait, alors regardons ce que cela donne avec un exemple simple. On considère `Maybe` qui est une instance de `Applicative` de la façon suivante :
``` haskell
instance Applicative Maybe where
   pure = Just
   (Just f) <*> (Just x) = Just (f x)
   _        <*> _        = Nothing
```

La méthode `pure` prend un `truc :: a` et le place dans un `Maybe a` par `Just truc`. C'est la façon la plus intuitive de placer notre `truc` dans un `Maybe`, sans perdre d'information. L'opérateur `<*>` se contente lui de récupérer la fonction à sa gauche, la valeur à sa droite, d'effectuer le calcul (l'application de f à x) puis de placer le résultat dans un contexte minimal en utilisant le constructeur `Just`. La dernière ligne s’occupe des cas où il manque la fonction, l'argument, ou bien les deux. Dans ces trois cas, on ne peut effectuer le calcul, et l'on ne peut donc pas produire de résultat.

Comme dit plus haut, si l'on vais une fonction qui prend la racine carré (`sqrt :: Int`) d'un entier, et "peut-être un nombre" (un `v :: Maybe Int`), on pouvais mapper notre fonction avec la ligne `fmap sqrt v`. Avec un foncteur applicatif, on peut aussi écrire :
``` haskell
resultat  = (pure sqrt) <*> v
```

Mais alors, quel est l’intérêt des foncteurs applicatifs? Un premier exemple est l'application d'une fonction prenant trois entiers à trois "peut-être un entier".
``` haskell
--Ici, v1, v2 et v3 sont de type Maybe Int.
--Si vous voulez savoir d'où il viennent, disons que ce sont le résultat
--de la lecture d'une chaine de caractère et de tentative de conversion de
--la chaine en nombres. Si c'était possible, alors on a des valeurs. Sinon, on auras "Nothing".

--On a une superbe fonction :
deepThought :: Int -> Int -> Int -> Answer

--Et on veut l'appliquer à v1, v2, v3.
--Si l'on essaye avec un foncteur :
premiereApplication :: Maybe (Int->Int->Answer)
premiereApplication = fmap deepThought v1

--Maintenant, on est bloquer, on ne sais pas appliquer
-- une fonction qui se trouve dans un maybe...
--... à moins d'utiliser <*> :)
secondeApplication :: Maybe (Int -> Answer)
secondeApplication = premiereApplication <*> v2
derniereApplication :: Maybe Answer
derniereApplication = secondeApplication <*> v3

--En fait, on aurait pu d'abord placer la fonction dans un Just,
-- puis appliquer v1, v2 et v3 avec <*> :
toutEnUn :: Maybe Answer
toutEnUn = (pure deepThought) <*> v1 <*> v2 <*> v3

--Et pour finir, sachez qu'il existe un petit sucre syntaxique
-- pour "(pure f) <*> x" ; l'opérateur <$> :
toutEnUn :: Maybe Answer
toutEnUn' = deepThought <$> v1 <*> v2 <*> v3
```

Mais parce que le haskell détient bien plus de P-P-P-P-Puissance, intéressons nous à une autre instance de `Applicative` : Les listes.

### Premier plongeon avec les listes

Comme nous l'avions vu la dernière fois, les listes sont un excellent exemple de foncteur. Mais pouvons nous en faire des foncteurs applicatifs? Il est facile de placer une valeur dans un contexte minimal : on définira `pure a = [a]`. En fait, il nous faudrait répondre à la question suivante : Si l'on a une liste de fonction `[f1, f2, f3]` et de valeurs `[2, 3, 4]`, comment définir l'opérateur `<*>` ?

La solution retenue par haskell est on ne peut plus simple : on applique toutes les fonctions à toutes les valeurs. Voici la définition de l'instance :
``` haskell
instance Applicative [] where
    pure x = [x]
    fs <*> xs = [f x | f <- fs, x <- xs]
```

Cela donne une façon très facile d'effectuer de nombreux calculs simultanément ; vous disposez d'un ensemble de valeurs, et d'un ensemble de fonctions. Vous voulez connaître TOUS les résultats possible. Il suffit alors d'appliquer la liste des fonctions sur la liste des valeurs avec l'opérateur `<*>`. Un exemple, en partie tiré de_Learn You Haskell for Great Good_ est le problème du parcoure d'un cavalier. Un cavalier est situé sur une case d'un échiquier infini, et vous voulez connaître toute les positions où il peut se trouver après 5 coups. Il suffit décrire une fonction par déplacement possible, puis de construire une liste de ces fonctions,  disons `[u1, u2, l1, l2, r1, r2, d1, d2]`. On applique cette liste à la position d'origine placée dans un contexte : `[(x, y)]` (ou encore `pure (x, y)`). La solution est donné par l'application répété de notre liste de fonction, comme le montre le code suivant.

``` haskell
u1 (l, c) = (l+2,c+1)
u2 (l, c) = (l+2,c-1)
d1 (l, c) = (l-2,c+1)
d2 (l, c) = (l-2,c-1)
l1 (l, c) = (l+1,c-2)
l2 (l, c) = (l-1,c-2)
r1 (l, c) = (l+1,c+2)
r2 (l, c) = (l-1,c+2)

fctListe = [u1, u2, d1, d2, l1, l2, r1, r2]
origine (l, c) = [(l,c)]

etapeSuivante position = fctListe <*> position

solution (l, c) = etapeSuivante . etapeSuivante . etapeSuivante . etapeSuivante . etapeSuivante $ origine (l, c)
```

### En apnée : le foncteur `(->) r` !

On les avais cacher lors de la discussion des foncteurs, car ils sont difficile à cerné. Leur intérêt n'est pas évident au premier abord et leur construction est quelque peu... surprenante. Mais puisqu'ils sont utile comme foncteur applicatif, parlons en! Si la partie la plus abstraite vous échappe, aucune raison de vous inquiéter, l'idée est de survoler les notions pour avoir un aperçu, éveiller la curiosité et inciter à lire des livres/articles qui expliquent en détaille ce qui n'est ici que mentionner. Si tout cela vous intéresse, sautez à la section "Foncteurs applicatifs" de Learn You Haskell for Great Good!

L'opérateur `->` est un constructeur de type, à deux arguments. Vous lui donnez deux types, `a` et `b`, et il vous construiras le type "prend du a et retourne du b". On peut donc écrire `f :: a -> b` ou encore ` f :: (->) a b`. Que signifie alors `(->) r` ? On parle d'un constructeur de type à un argument qui, si vous lui donnez un type `a`, désigneras alors les fonctions de type `a -> r`. Si `r` désigne un type, on peut alors faire de `(->) r` une instance de `Functor` où mapper une fonction `f` de type `a->b` sur une fonction `g` de type `a -> r` signifie appliquer `f` au résultat de l'évaluation de la fonction `g`. Plus précisément :
``` haskell
instance Functor ((->) r) where
    fmap f g = (\x -> f (g x))
```

On peut se demander l’intérêt, puisque `(fmap f g) 42` se simplifie en `f . g $ 42` qui est, à priori, bien plus lisible. Outre sa fantastique capacité à mettre votre esprit à rude épreuve, ce changement de point de vue devient très intéressant avec la classe `Applicative`, puisqu'il donne la possibilité d'avoir un ensemble de paramètres.

Un exemple commenté :
``` haskell
--Notre type "paramètre". On aurais pu construire une sorte de grosse structure
-- avec diverses informations.
--Dans cette exemple, on se contenteras d'un nombre.
type Param = Int

unEntier :: Param -> Int
unEntier = pure 5

unAutreEntier :: Param -> Int
unAutreEntier param = 5 + param

uneFonction :: Param -> Int - > String
uneFonction param arg = show (arg + param)

somme = pure (+) <$> unEntier <*> unAutreEntier
affichage = uneFonction <*> somme

--Vaut 94 :
evaluationDansUnCOntexte = affichage 42
```

### Quelques règles que nul ne doit ignorer

Comme on dit, `dura lex, sed lex`. Les foncteurs devaient respecter certaines règles, et il en est de même des foncteurs applicatifs. Une fois habitué aux foncteurs applicatifs, ces règles semblent découler du bon sens. Ce sont des invariants que DOIVENT respecter vos instances d'`Applicative`. Si vous ne les respectez pas, c'est que ce que vous voulez faire n'est pas un foncteur applicatif, et n'a donc aucune raison d’être instance d'`Applicative`.

Sans trop rentrer dans les détails, les voici, brièvement commentés :
``` haskell
-- Neutre :
pure id <*> v = v
--Cela signifie qu'appliquer la fonction identité
-- (id = (\x -> x) ) a un élément v via <*> le laisse inchangé.
--C'est une sorte de "neutre à gauche".

-- Composition :
pure (.) <*> u <*> v <*> w = u <*> (v <*> w)
--Cela signifie que composer les fonctions à l'intérieur de u et v
-- via l'opérateur .  (C'est la partie pure (.) <*> u <*> v) revient à calculer u sur le résultat de v.
--Comme on compare deux fonctions sur leur valeur, et que l'on parle de résultat, on doit introduire
-- un certain mister w, et on vérifie que quelque soit ce w, on a bien que u calculer sur v calculé sur w donne bien le même résultat que la composé (pure (.) <*> u <*> v) calculé sur w.

-- Morphisme :
pure f <*> pure x = pure (f x)
--Cette égalité garanti que : placer f dans un contexte minimal, placer x dans un contexte
-- minimal, puis "mouliner" donne le même résultat que f x placé dans un contexte minimal. En quelque sorte,
-- on transforme l'opérateur $ :: (a->b) -> a -> b en l'opérateur <*> :: f (a->b) -> f a -> f b.

-- ' Échange '
u <*> pure y = pure ($ y) <*> u

-- C'est une sorte de commutativité du pauvre. On ne peut pas vraiment échanger les arguments à droite et à gauche, car l'un est une fonction, l'autre une valeur. Mais on peut transformer une évaluation "f y" en "$ f y ", ce qui permet de changer l'ordre des arguments.
```

## Monades

Les monades, c'est le cran au dessus. On ne veut plus seulement mapper des fonctions `f: a -> b` à l'intérieur d'un `Fonc a`, ni seulement évaluer des fonctions `Fonc (a -> b)` dans un contexte `F a`. Maintenant, on dispose de fonctions qui travaillent sur une valeur, et produisent un résultat dans un contexte. Des fonctions de type `f :: a -> Fonc b`. Si l'on essayais de les mapper comme des foncteurs sur un `Fonc a`, on se retrouverais avec du `Fonc (Fonc b)`, ce qui n'est pas du tout ce que l'on veut. Il nous faut donc une fonction capable de recoller ces "Fonc Fonc" en "Fonc". C'est ce que fournisse les monades.

Tout de suite, la classe monade :
``` haskell
class Monad m where
  (>>=) :: m a -> (a -> m b) -> m b -- On l’appelle aussi "bind"
  (>>) :: m a -> m b -> m b -- C'est une sorte d'opérateur de concaténation.
-- >> Ignore le premier argument et renvoi la valeur du second.
-- On vera plus tard qu'en fait, c'est extrêmement utile, avec la monade IO.
  return :: a -> m a -- C'est notre bon vieux pure, sous un autre nom.
  fail :: String -> m a -- On ne l'utiliseras pas, et on en parleras pas.
-- Sachez toute fois que ca renvoi moralement un "contexte sans information".
-- Par exemple une liste vide, un Nothing, etc.
```

Les deux opérateurs principaux sont `bind` et `return`. Voyons comment on pourrait, partant d'une monade, la faire instance d'Applicative :
``` haskell
instance Monade Fonc where
  pure = return
  -- L'astuce est de construire une fonction f' :: (a -> m b) que l'on puisse utiliser à la place de f.
  -- On la construit grâce à "pure . f".
  -- Mais comme f est elle même dans un contexte, il faut faire cette transformation dans le contexte.
  mf (<*>) mv = mf >>= (\f -> mv >>= return.f)
-- On donne mf à manger a la grosse fonction de droite. La grosse fonction de droite récupère la fonction f, la transforme en une f' par return.f. On donne donc la valeur v contenue dans mv à manger a la fonction f' grâce à >>=.
```

Bon, si vous avez suivi jusque là, soit vous connaissez déjà le haskell, soit vous êtes des sur-hommes (ou des matheux, auquel cas je ne peux plus rien pour vous). Voyons en pratique ce qu'apportent les monades, et pourquoi est-ce que bien utilisé, elles offres une nouvelle façon de résoudre certains problèmes bien connu du monde impératif.

### Être ou ne pas être?

Dans un "vrai" programme, on n'a pas toujours une valeur a retourner pour une fonction. Que faire si l'on demande le premier élément d'une liste vide? Et si jamais on veut convertir une chaine en un nombre, qui par malheur contient le prénom de votre animal de compagnie? En bref, comment gérer une erreur correspondant à l’absence d'un résultat?

La réponse est la monade maybe. Commençons par des fonctions qui renvoient peut-être une valeur :
``` haskell
maybeHead :: [a] -> Maybe a
maybeHead [] = Nothing
maybeHead (head : tail) = Just head

maybeList :: (Int, Int) -> Maybe [Int]
maybeList (first, last) = if first <= last then [first..last] else Nothing

maybeRange :: Bool -> Maybe (Int, Int)
maybeRange False = Nothing
maybeRange True = (23, 42)
```

On voudrais maintenant récupérer le premier élément de la liste pour les valeur donné par maybeRange, si la liste existe, bien sure. C'est la que les monades interviennent! Grâce au monades, on peut composer les deux fonctions, bien qu'un `Maybe [Int]` ne soit pas un `[Int]`.
``` haskell
resultat :: Maybe Int
resultat choice = maybeRange choice >>= maybeList >>= maybeHead
```

Si l'une des étapes ne produit pas de résultat (un `Nothing`), alors l’absence de résultat seras propagé et on obtiendras un `Nothing`.

Cette méthode a de nombreux avantages par rapport aux deux vielles solutions bien connues :
1) Le "code d'erreur", c'est à dire placer nullptr quand on pointeur n'existe pas, ou encore "-1" ou 0 pour signaler une erreur. L'inconvénient de cette méthode est d'obliger le développeur à vérifier chacune des valeurs de retour avec un if, généralement pour sortir de la fonction, souvent en retournant un nouveau code d’erreur pour signaler que le résultat produit n'est pas "vraiment" un résultat, mais une absence de résultat.

L'utilisation de la monade Maybe permet d'éviter ces testes répété. Si une seul des fonctions ne peut pas fournir de résultat, alors les applications suivantes seront toute ignoré et, bien entendu, ces fonctions ne seront pas évaluées, donc pas de coût en temps de calcul.

2) Les exceptions. Cela consiste à interrompre l'exécution normale du programme pour remonter a travers toute la pile d'appels, en espérant que quelqu'un seras assez gentil pour s'occuper de cette erreur. Cela a un coût en terme de performances, et doit être réserver pour les évènements exceptionnels. L'impossibilité de produire un résultat est rarement exceptionnel, c'est plutôt chose commune.

Le chaînage de monade Maybe a l'avantage de ne pas déclencher un erreurs qui pourrait se perdre et aller jusqu'à interrompre le programme. Que les valeurs soient présente ou non, le comportement est toujours "simple" à prédire. Et plus un code est simple, moins il y a de risque qu'une erreurs s'introduisse à l'insu du développeur.


En règle générale, dès que le résultat peut ne pas être fournit, vous devriez utiliser la monade Maybe. Si parfois une certaine fonction f que vous voulez chaîner produit toujours un résultat, alors vous pouvez la placer au milieu d'une chaine de `>>=` en écrivant `return.f`. Vous pouvez aussi une bonne vielle `fmap`, car toute les monades sont des foncteurs applicatifs, donc des foncteurs.

Nb : Peut-être avez vous besoin de conserver une information sur l'origine de l'erreur. Ceci est possible grâce à la monade `Either a` (souvent on utilise `Either String` pour stocker un message).

Pour finir, voici l'instance de cette monade :
``` haskell
instance Monad Maybe where  
        return x = Just x  
        Nothing >>= f = Nothing  
        Just x >>= f  = f x  
        fail _ = Nothing  
```

### Un calcul pas très déterministe

Nous avions vu comment les listes comme foncteurs applicatifs permettent de résoudre élégamment la question du déplacement d'un cavalier. Mais dans un échiquier fini, on ne savais pas trop comment gérer les bords.

Les listes, vu comme monade, nous permettent de combiner des fonctions de type `a -> [b]`. L'idée est que vous disposez de diverse fonctions qui prennent une valeur, et produise divers résultats possible. Vous voulez alors appliquer des fonctions sur chacun de ces résultats. On peut donc parler de calcul non-déterministe : une valeur donne plusieurs résultats possible.

On peut donc réaliser un remake du cavalier, en se servant de ce calcul non déterministe, puisqu'à partir d'une position, on a diverses positions possibles
``` haskell
--Quelques types pour plus de lisibilité,
-- histoire de rappeler que les types sont
-- aussi là pour fournir des informations sémantiques.
type Ligne = Int
type Collone = Int
type Position = (Ligne, Collone)

--Fonction utilisé pour ne garder que les positions dans l'échiquier
dansLechiquier :: Position -> Bool
dansLechiquier (l, c) = l `elem` [1..8] && c `elem` [1..8]

--Produit une liste des positions possibles
deplacerCavalier :: Position -> [Position]
deplacerCavalier (l, c) = filter dansLechiquier liste
  where liste = [(l+2,c-1),(l+2,c+1),(l-2,c-1),(l-2,c+1) 
                ,(l+1,c-2),(l+1,c+2),(l-1,c-2),(l-1,c+2)]

--Donne la liste des positions possible du cavalier, partant de (4, 5), et après trois déplacements.
resultat = return (4, 5) >>= deplacerCavalier >>= deplacerCavalier >>= deplacerCavalier
```


En fait, une autre façon de faire serai de mapper la fonction `deplacerCavalier` dans la liste de positions, produisant un `[[Position]]`, puis d'appeler `concat` sur cette liste, la recollant en une `[Position]`. C'est d'ailleurs de cette façon qu'est implémenté l'opérateur `>>=` !

L'utilisation des listes sous leur forme de monade n'est pas limité à cette exemple. Je peux mentionner un second cas qui vous paraîtras peut-être plus concret. Disons que vous voulez enregistrer une image, et que vous disposez d'une fonction qui vous donne les couleurs r, g, b, a sous la forme d'une liste de nombres, c'est à dire `getPixel (x, y) :: [Word8]` (Word8 est un nombre codé sur 8 bits). Sachant que pour enregistrer l'image, vous devez fournir un tableau, qui peut être très facilement construit à partir de la liste des valeurs qu'il vas contenir. (Le compilateur est très malins, et le programme compilé ne s’amusera pas à produire une liste, la construire en mémoire, puis la placer dans le tableau. Pas d'inquiétude, le compilateur est très douer à ce niveau.)
La monade `[]` permet d'écrire en une ligne, de façon très élégante, la création d'un tableau où les valeurs sont bien la succession des valeurs de `getPixel` calculé à chaque coordonné. Bien sur, on aurais pu utiliser `concat` et `map`, mais c'est justement ce que fait l'opérateur `>>=`. Tout ça pour dire que les listes vue comme monade ne sont pas un gadget, mais bien un outil utile à l'implémentation d'applications de la "vie de tout les jouers".

Voici la définition de l'instance pour les curieux :
``` haskell
instance Monad [] where  
    return x = [x]  
    xs >>= f = concat (map f xs)  
    fail _ = []
```

### Dou? Doo? Do, c'est le goût!

La notation do est une sorte de super sucre syntaxique. Seulement, les monades sont tellement amère que vous aurez vraiment besoin de ce sucre, je vous l'assure.
La notation do permet décrire facilement le chaînage d'actions, et le fait de récupérer des valeurs dans un contexte.

Considérons l'exemple suivant tiré de Learn You Haskell for Great Good (Non, je ne suis pas encore sponsorisé par eux.) :
``` haskell
foo :: Maybe String  
foo = Just 3   >>= (\x -> 
      Just "!" >>= (\y -> 
      Just (show x ++ y)))
```
On récupère deux valeurs de monades à travers x et y, puis l'on place le résultat de show x ++ y dans un contexte. C'est lourd a écrire, demande l'imbrication de fonctions... Avec la notation do, cela évident :
``` haskell
foo :: Maybe String  
foo = do  
        x <- Just 3  
        y <- Just "!"  
        Just (show x ++ y)  

-- Ou encore :
foo :: Maybe String  
foo = do  
        x <- Just 3  
        y <- Just "!"  
        return (show x ++ y)  
```

Dans les deux premières lignes, on récupère `x` et `y` depuis `Just 3` et `Just "!"`, puis on fait notre traitement.

Hey! Mais ça ressemble a des listes en compréhension tout ça! Reproduisons le même exercice mais avec des listes :
``` haskell
foo :: Maybe String  
foo = [1, 2, 3]   >>= (\x -> 
          [".", "!", "?"] >>= (\y -> 
          [show x ++ y]))
```

Le résultat produit est toute les façons de coller l'un des signes de ponctuation après l'un des nombres. C'est exactement la même chose que :
``` haskell
foo :: Maybe String  
foo = do
          x <- [1, 2, 3]
          y <- [".", "!", "?"]
          return (show x ++ y)

-- Ou encore :
foo = [show x ++ y | x <- [1, 2, 3] | y <- [".", "!", "?"]]
```

Voilà donc l'origine des liste en compréhension! Et oui, il nous aura fallu arriver jusqu'ici pour pouvoir enfin expliquer ce que sont les liste en compréhension. C'est simplement un sucre syntaxique spécifique au liste d'un bloque do. C'est donc de la manipulation de monade que vous faites, à chaque fois que vous écrivez une liste en compréhension. Si c'est si pratique avec les listes, vous vous doutez bien que pouvoir le faire avec diverses structures (des arbres par exemple), est tout aussi pratique.

Vous vous demandez alors comment ajouter les conditions, comme dans `[x^2 | x <- [1..20], x `mod` 2 == 0]` ? Et bien vous pouvez utilisez la fonction `guard`, qui produiras une liste vide si la condition n'est pas vérifiée :
``` haskell
foo = do
      x <- [1..20]
      guard (\x -> x `mod` 2 == 0)
      return x^2
```

### Le retour de Jafar, aussi connu sous le nom de `(->) r`.

Nous avions utilisé le foncteur applicatif `(->) r` pour représenter des calculs qui dépendent d'un contexte. On savais donc appliquer des fonctions `r -> (a -> b)` sur des valeurs `r -> a`. Seulement, il est plus commun de partir d'une valeur, et produire un résultat qui dépend du contexte. On voudrais donc une monade, pour pouvoir combiner des fonctions de type `a -> (r -> b)` (Les parenthèses sont là pour faire ressortir que l'on considère (->) r comme un foncteur / une monade, mais bien sur facultatives).

Comme `(->) r` est l'une des monades les plus abstraites, regardons un exemple concret, où le contexte est la position d'une caméra dans un raytracer.

``` haskell
-- On définit une caméra.
-- Une caméra contient la position depuis la quelle
-- les rayons sont lancé, la distance à la quel se trouve
-- le plan, et la taille de celui ci.
type Position = (Double, Double, Double)
type Direction = (Double, Double, Double)
type Distance = Double
type Coordonnee = Int
type Largeur = Coordonnee
type Hauteur = Coordonnee
type Plan = (Largeur, Hauteur)
data Camera = Camera Position Distance Plan

getRay :: Cooronnee -> (Camera -> Direction)
getRay (i, j) cam = let (Camera (x, y, z) _ _ _) = cam in normalize (i - x, j - y, -z)
  where
    normalize (x, y, z) = let norme = sqrt(x^2 + y^2 + z^2) in (x / norm, y / norm, z / norm)

rayTraceScene :: Scene -> Direction -> (Camera -> (Object, Distance))
rayTraceScene = -- Imaginons que l'on fait le nécessaire pour raytracer une scène.

computeColor :: (Object, Distance) -> Camera -> [Word8]
computeColor = -- Calcule la couleur en tenant compte de l'angle de la caméra, etc.

--Pour getPixel, on préférera souvent spécialiser d'abord la Caméra,
-- puis appeler cette spécialisation sur toute les coordonnées de l'image.
-- C'est pourquoi le type est "Camera -> Coordonnee -> [Word8]" plutôt
-- que "Coordonnee -> Camera -> [Word8]". On perd donc la possibilité
-- d'utiliser getPixel avec la monade "(->) Camera".
getPixel :: Camera -> Coordonee -> [Word8])
getPixel cam coords = (return coords >>= getRay >>= (rayTraceScene uneScene) >>= computeColor) cam
```

### Point culture

Comprenez bien que les monades ne sont pas indispensable, et que l'on faisait des monade avant la première apparition de la classe Monade. C'est simplement de nouveaux outils à votre disposition pour résoudre des problèmes, et ils permettent parfois de vieux problèmes de façon très élégante et concise (ce qui est un excellent pour l'évolutivité d'un code).

Sachez que les monades aussi doivent respecter certaines règles, tous comme les foncteurs et les foncteurs applicatifs. Cela assure qu'une monade seras bien un foncteur (applicatif), de la façon décrite plus haut. Les voici :
``` haskell
-- Neutre à gauche
return a >>= f = f a
-- Neutre à droite
m >>= return = m
-- Associativité
(m >>= f) >>= g = m >>= (\x -> f x >>= g)
```

Vous trouverez des liens dans les références pour plus de détails.

On n'a pas aborder la monade IO. Cette monade permet de ramener les actions propre a l'impératif, les "effets de bord", dans le langage haskell. L'astuce diabolique est la suivante : Puisque qu'un programme haskell ne peut produire d'effet de bord, un programme haskell décriras comment composer, jusxtaposer, et transformer les résultats produits par des programmes impératif. Utiliser la monade IO, c'est jouer avec la composition et la juxtaposition de programmes. C'est alors que l'opérateur `>>` prend tout son sens! Il permet de juxtaposer deux programmes impératifs et ne retenir le résultat que du second, par exemple :
``` haskell
afficherMessage = putStrLn "Bonjour," >> putStrLn "monde!"
```

En dire plus sur cette monade n'a d'intérêts que pour les personne voulant écrire des programmes en haskell. Si c'est votre cas, je vous invite à lire, au choix, Learn You Haskell for Great Good ou (plus technique et plus ... "professionnel") Real World Haskell, chez O'Reilly.

### Références :

  * [Les lois respectés par les monades][monades-laws]
  * La monade [Either][either]
  * Le moteur de recherche haskell [Hoogle][hoogle]. Permet de rechercher une fonction à partir de son type.

[monades-laws]: http://www.haskell.org/haskellwiki/Monad_laws
[either]: http://hackage.haskell.org/packages/archive/category-extras/0.53.1/doc/html/Control-Monad-Either.html
[hoogle]: http://www.haskell.org/hoogle/