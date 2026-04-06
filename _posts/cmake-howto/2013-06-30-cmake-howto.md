---
layout: post
title: CMake - Mini howto
description: Introduction à CMake.
author: Jérémy Cochoy
date: 2013-06-30 + 0100
categories: language cmake adminsys makefile
lang: fr
...

## Qu'est-ce que CMake ?

CMake est un utilitaire permettant de générer le "build process" d'un projet. C'est un outil du même genre que les autotools (aussi appelés pour des raisons que je tairai 'autohell') en

  * Plus simple à configurer
  * Plus récent (donc moins de façons de procéder obscures)
  * Plus performant (dans le sens où, avec cmake, compiler un projet à l'extérieur du repository est extrêmement simple et agréable!)
  * Vous êtes libre d'organiser vos répertoires/fichiers comme vous le désirez.

Il est bon aussi de savoir que CMake est capable de générer aussi bien des Makefiles que des fichiers de projet VC++ ou XCode (pas mal, hein?).

On a aussi une jolie progression en % de la compilation (et l'on peut toujours utiliser make -j42 pour faire chauffer les cœurs).

##Comment on s'en sert ?

On connaît tous le _./autogen.sh_, _./configure_ puis _make && sudo make install_. Bien entendu, on produit tous les fichiers bien sales dans le dossier des sources, ce qui donne une magnifique liste de fichiers à ajouter au .gitignore de votre projet (ou tout autre fichier équivalent pour svn / mercurial). Avec cmake, on peut reproduire le schéma des autotools via `cmake .` puis `make && sudo make install`. Mais mieux, vous pouvez (par exemple) faire `mkdir build && cd build` puis `cmake ..` et `make && sudo make install`. Dites bonjour à la propreté de votre projet grâce à CMake ;)

##Comment ça fonctionne ?

C'est là qu'est le plus beau. Configurer CMake pour un projet, gérer les dépendances de libs, générer les libs, voire même générer les packages (que ce soit des packages archlinux, debian ...). On prend très vite en main, c'est clair, et la documentation est excellente.

Je ne vais pas détailler comment construire un fichier de config ; le [tutorial CMake](http://www.cmake.org/cmake/help/cmake_tutorial.html) le fait déjà très bien, et l'on trouve tout ce qu'il manque dans le wiki. (Notez que dans un VRAI projet, il vous faudra un peu plus que le tutorial). Je vais aborder les idées générales et les petites astuces qui m'ont servi.

D'abord, on place les instructions dans un fichier CMakeLists.txt à la racine du projet. On évite donc la multiplication des fichiers de configuration. Il est possible d’appeler des scripts d’autres dossiers grâce à `add_subdirectory("${PROJECT_SOURCE_DIR}/your_folder"`). Il est important d'utiliser `${PROJECT_SOURCE_DIR}` pour que tout se place bien quand on souhaite compiler dans un autre dossier.

On peut facilement demander à cmake de trouver des libs. En règle générale, on trouve un FindNomDeLaLib.cmake que l'on placera dans projet/cmake/Modules/, et l'on prendra soin d'ajouter `set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")`.

Une façon pratique pour compiler votre projet est de placer tous vos .c/.cpp dans un dossier src, et de tous les sélectionner (ça évite d'oublier d'ajouter un .cpp à la liste des fichiers du projet). Avec `file(GLOB_RECURSE SOURCE_FILES src/*.cpp SRC)` la liste des fichiers figure dans la variable "SOURCE_FILES". S'il vous faut exclure certains fichiers, vous pouvez utiliser :
`file(GLOB_RECURSE UNWANTED_FILES src/FichierAExclure.cpp SRC)
list (REMOVE_ITEM SOURCE_FILES ${UNWANTED_FILES})`

Si vous êtes fan des tests unitaires, CMake propose une façon agréable de lancer des tests avec :
`add_test (NAME NomDuTest WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/repertoiredexecution" COMMAND "commande a executer")`
Ça va de "lancer un programme et vérifier qu'il EXIT_SUCCESS" à "vérifier que la sortie du programme contient cette expression".

Chose utile, que ne fournissent pas les autotools, c'est une façon unifiée de gérer le numéro de version de l'application ; Toutes les variables d'un CMakeLists peuvent être utilisées pour générer des fichiers à partir de templates (souvent le nom du fichier avec le suffixe .in). En pratique, on définit VERSION_MAJOR et VERSION_MINOR, puis on génère un Version.h, un doxyfile, et on utilise les valeurs pour générer les noms des bibliothèques exportées. C'est simple, intuitif, et très bien expliqué dans le tutoriel. Faire une release se limite alors à changer le numéro de version dans un fichier, et de lancer la compilation pour obtenir les binaires et packages.

CMake dispose aussi d'un système de macros, pour faciliter la vie des maintainers, et l'on comprend donc que de plus en plus de projets tendent à l'utiliser.

Un fichier de configuration emacs est disponible (depuis la doc de cmake) pour ajouter un "cmake-mode", et se charge avec, par exemple, `(load "~/.emacs.d/cmake-mode.el")`.

On peut aussi contrôler la génération d'une documentation avec doxygen. Pour cela, on rajoute un CMakeLists dans un répertoire /doc. On demande à ce que doxygen soit présent via `find_package(Doxygen)`. On écrit un fichier template doxygen.in de configuration, où les valeurs à modifier sont de la forme @NOM_DUN_DEFINE@, et définies dans le fichier de configuration CMake par `set(NOM_DUN_DEFINE pouïc)`. Ensuite, on peut facilement ajouter une règle "make doc" de la façon suivante :
```
if(DOXYGEN_FOUND)
  configure_file(${DOXYGEN_TEMPLATE_FILE}
    ${DOXYGEN_CONFIGURED_FILE}
    @ONLY
    )
  add_custom_target(doc
    ${DOXYGEN_EXECUTABLE}
    ${DOXYGEN_CONFIGURED_FILE}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generating API documentation with Doxygen" VERBATIM
    )
endif(DOXYGEN_FOUND)
```
où le @ONLY signifie que seuls les mots entourés d'@ seront remplacés (Sinon, les ${SOMETHING} sont aussi affectés).

## Conclusion :

Ce qu'il faut retenir, c'est que CMake est simple à configurer, offre de nombreuses fonctionnalités, et laisse la possibilité d'ajouter celles manquantes. Beaucoup de gens sont encore habitués au ./configure, et sont effrayés par cette nouvelle façon d'aborder ce problème. Pourtant, CMake est un réel gain de temps, et l'on voit des gros projets comme KDE changer de build system pour CMake.  On peut même l'utiliser pour de minuscules projets (compiler les .cpp d'un dossier en un exécutable, sans dépendances particulières, se fait avec un fichier de configuration de 3 lignes), et je vous encourage justement à le faire. Trois lignes pour avoir une gestion automatique des dépendances des .h, génération des makefiles, le tout multiplateforme, ce n'est pas cher payé.

### Resources :
  * <http://en.wikipedia.org/wiki/CMake>
  * <http://www.cmake.org/cmake/help/cmake_tutorial.html>
