---
layout: post
title: Haskell - Lazy IO
description: On discute des entrées et sorties paresseuses en Haskell et
  des soucis qu'un comportement paresseux peut poser.
author: Jérémy Cochoy
date: 2013/07/16
...

Ces derniers jours, j'écrivais un script haskell qui repère les fichiers présent en double, et propose de ne conserver qu'un exemplaire. Très pratique pour faire un peu de rangement, par exemple parmi une centaine de PDF que je ne lirais jamais.

Je vais ici vous parler de la phase de hachage des fichiers pour les trier et déterminer les doublons. Et oui, on ne vas pas comparer le contenu de tous les fichiers entre eux, ça serait en n^2 par rapport au nombre de fichiers. On ne veux pas non plus attendre une journée.

Je parlerais donc de la lecture du contenu des fichiers pour produire leur hash. Le trie des paires (Nom du fichier, hash) et l'affichage étant trivial et sans intérêt.

Strict IO ?
===========

La première approche est souvent la plus simple. Nous voulons le hash d'un fichier? Et bien il suffit d'utiliser la fonction `hash :: String -> ByteString` du package `Crypto.Hash.SHA1` (SHA1, MD5, MD4, SHA256 ... celons vos gouts). Pour obtenir le contenue du fichier, on peux utiliser `readFile :: FilePath -> IO String`.

Cela nous donne :
``` {.haskell}
getHash :: String -> (String, ByteString)
getHash filename = do
    hashed <- fmap hash $ readFile filename
    return (filename, hashed)
```

Il suffit alors d'appliquer `mapM getHash` sur une liste de nom de fichier pour obtenir une liste de couple Nomdufichier/Hashdufichier.

Si l'on test, cela fonctionne très bien... jusqu'au moment où vous tombez sur un fichier de plus d'1GO. Là, `readFile` veux charger l'intégralité du fichier en mémoire. Et bien-sur, sur ma machine, c'est impossible.

La paresse a la rescousse!
==========================

On vous à toujours dis qu'être paresseux, c'était mal, improductif, et vous mènerais à votre perte? Et bien, ils avaient tort.

On voudrais lire le fichier par morceau, et construire le hash avec ces morceaux (ce que toute bonne fonction de hachage permet).

Pour ce qui est du hachage, on trouve dans `Crypto.Hash.Whatever` les trois fonctions :
``` {.haskell}
init :: Ctx
update :: Ctx -> S.ByteString -> Ctx
finalize :: Ctx -> S.ByteString
```

Il nous faut donc un flux de `ByteString`. Pour ce faire, on dispose d'une version paresseuse de `ByteString`, qui se trouve dans le package `Data.ByteString.Lazy`. Histoire de fixer les notations et de ne pas se perdre entre les `ByteString` strict et les `ByteString` lazy, on parlera respectivement de `S.ByteString` et de `L.ByteString` (Strict/Lazy).

Cela revient a importer les deux types de la façon suivante :
```haskell
import qualified Data.ByteString as S
import qualified Data.ByteString.Lazy as L
```

Le type L.ByteString est en fait une liste de S.ByteString, et chaque bloc ne sera lu depuis le fichier qu'au moment où il sera utilisé. Le module L nous offre donc gentiment les fonctions
```haskell
L.readFile :: FilePath -> IO L.ByteString
L.foldlChunks :: (a -> S.ByteString -> a) -> a -> L.ByteString -> a
```

La première nous donne le contenue de notre fichier. La seconde, nous offre exactement la méthode qui "prend une fonction ajoutant un morceau de fichier au hash", puis "un hash vide", et enfin "une ByteString paresseuse". Cela parais plus évident si l'on spécialise les "a" en "Ctx".

On peux donc utiliser :
```haskell
hashed <- fmap (finalize . foldlChunks update init) $ readFile filename
```

Bon, en fait, il y avais plus simple. Il y a aussi la fonction :
```haskell
hashlazy :: L.ByteString -> S.ByteString
```
qui produit le même résultat que notre pli.

Et là tout fonctionne bien, nos fichier d'1Go sont haché.

Mais ... si vous travaillez sur beaucoup de fichiers, vous risquez d'avoir un soucis. Par exemple, si vous testez sur /usr/lib (qui dois probablement contenir beaucoup de fichiers ;) ), vous obtiendrez peut-être une jolie exception vous indiquant qu'il y a trop de filedescriptors utilisés, et que donc le fichier ne peux être ouvert.
Il se trouve que les fichier ne sont pas fermé immédiatement après le hachage, et c'est un vrai problème. Problème que nous allons résoudre.

La solution : withFile
======================

La fonction `withFile :: FilePath -> IOMode -> (Handle -> IO r) -> IO r` est la clef. Cette fonction prend un nom de fichier, le mode d'ouverture (on utilisera ReadMode), une fonction qui travaille sur le fichier, et nous fait suivre le résultat de l'application de cette fonction. Un peu stupide, me direz vous? Et bien, ça le serait, si cette fonction se contenter de ça. En fait, elle vous assure aussi qu'une fois évaluée, le fichier est fermé. En l'utilisant, on est donc certain que le fichier est immédiatement fermé lorsque withFile est évalué en un "IO r".

````haskell
getHash filename = do
    hashed <- withFile filename ReadMode $ \h -> do
        fmap hashLazy $ L.hGetContents h
return (filename, hashed)
```

Le `unefonction $ \h -> do` est une pratique courante pour ces fonctions qui évalue un morceau de code dans un certains contexte. C'est très pratique, et on reconnaît très vite cette idiome avec un peu d'entraînement.

On compile pour vérifier le typage, tout vas bien, et l'on lance donc notre application que l'on s'attend a voir fonctionner. Et là, c'est le drame : "Illegal operation : handle is closed".

Wohw, quel est ce mystérieux message? Et bien, comme je vous l'ai dit, withFile ferme immédiatement le fichier après l'évaluation de son expression. Et que fait son évaluation? Elle retourne le hash du fichier me dite vous? Faut.

La fonction construit un thunk (je parlerais de promesse de calcul, ou plus simplement de promesse) a l'aide de "fmap ...". L’évaluation de withFile retourne alors cette promesse, plutôt que la valeur du hash. Et cette promesse, elle, ne sera évaluée que plus tard, au moment ou vous en aurez vraiment besoin, et seulement si vous en avez besoin.

Il nous faut donc forcer haskell a être strict, et évaluer le hash avant de sortir de la fonction. Il y a différentes façons de le faire. La plus simple, c'est d'utiliser l'opérateur `` `seq` `` qui force l'évaluation de l'expression a sa gauche, puis retourne l'expression a sa droite. On a aussi l’opérateur `` a ($!) b = b `seq` (a b) `` qui est une "application stricte". C'est a dire que ce qui sera a droite de `$!` sera évalué. 

Nb : Il faut faire attention. Quand je dis évalué, je parle bien de _dé-construire_ le premier niveau de l'expression. C'est a dire que si `compute 1 42` est un calcul, ceci sera remplacer par la promesse ou le résultat retournée par compute. Si compute produit une promesse plutôt qu'un résultat, l'opérateur `` `seq` `` n'évaluera pas la promesse. Il existe un opérateur `` `deepSeq` ``, qui lui vas _tout réduire_ en profondeur. Comme vous vous en doutez, c'est très coûteux et évalue des choses dont on n'auras peut-être pas besoin. Le plus souvent, on peux se contenter de `` `seq` `` appliqué au bonne endroit pour obtenir le résultat souhaité.

Dans notre cas, il faudrait forcer la promesse faire par `fmap` a être évaluée, puis la promesse faite par `hashlazy`. Vous allez voir que la solution n'est pas plus compliquée que ce que nous avions déjà écrit :

```haskell
-- getHash ...
    withFile filename ReadMode $ \h -> do
        -- On obtient la promesse faite par hashlazy
        data <- fmap hashlazy $ hGetContents h
        -- On force l'évaluation de hashlazy, qui donc sera forcé d'ouvrir le fichier et de le lire, puisque la valeur retournée est un S.ByteString, une valeur Stricte.
        return $! data
```

Voila, c'est tout, c'est `$!` qui fait tout le travail en demandant à hashlazy de gentiment s'évaluer.

Et la, c'est le bonheur, tout refonctionne, et l'on peux gérer des fichiers arbitrairement grand en nombre arbitrairement grand, le tout en 4 lignes.
