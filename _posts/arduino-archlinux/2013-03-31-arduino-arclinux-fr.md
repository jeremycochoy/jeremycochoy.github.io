---
layout: post
title: Arduino Uno sous ArchLinux (3.8.4-1-ARCH)
description: Explique comment contourner un bug avec la version
  2013 de l'environement de développement arduino sous arclinux.
author: Jérémy Cochoy
date: 2013-03-31 + 0100
categories: arduino software arclinux linux
lang: fr
redirect_from:
  - /blog/arduino-archlinux/fr.html
...

Voilà que je ressort mon arduino de mes cartons, motivé à dépoussiérer le code de ma 'boite musicale'. Et quel ne fut pas ma surprise quand le package rxtx refusa de s'installer suite a des erreurs de compilation.

Vous commencez sûrement à me connaître ; je n'ai pas abandonné pour si peu. Voilà donc que je me retrouve a publier ce petit billet, accompagné des instructions pour patcher et installer la dernière version de rxtx sous votre système :)


## Étape 1 : Arduino package

Installez le package arduino en utilisant la commande yaourt :

``` shell
yaourt -S arduino
```

Étape 2 : Télécharger, patcher et installer rxtx
================================================

Telechargez la pre-release depuis http://rxtx.qbang.org/wiki/index.php/Download :

``` shell
wget http://rxtx.qbang.org/pub/rxtx/rxtx-2.2pre2.zip
unzip rxtx-2.2pre2.xip
cd rxtx-2.2pre2
```

Lancez configure avec l'option --disable-lockfile :

``` shell
./configure --disable-lockfile
```


Ici, si vous tentez de compiler, vous aurez probablement une erreur `UTS_RELEASE` est indéfini. Pour corriger ce problème, commencez par trouver le fichier utsrelease.h (`find /usr/ -name 'utsrelease.h'`). Pour ma part, il se trouvait dans `/lib/modules/3.8.4-1-ARCH/build/include/generated/`. Ensuite, incluez le dans config.h(C'est un fichier généré par configure), de facon à ce que la constant soit définie partout.

``` shell
echo "\n#include \"/lib/modules/3.8.4-1-ARCH/build/include/generated/utsrelease.h\"\n" > config.h
```

Maintenant, on doit ajouter ttyACM à la liste des périphériques. On modifit le fichier src/gnu/io/RXTXCommDriver.java en ajoutant une entrée à un tableau (ttyACM). Regardez le fichier diff suivant :


``` shell
--- src/gnu/io/RXTXCommDriver.java.back 2013-03-31 14:14:38.718567087 +0200
+++ src/gnu/io/RXTXCommDriver.java      2013-03-31 14:08:38.728149384 +0200
@@ -577,6 +577,7 @@
                                                "ttyS", // linux Serial Ports
                                                "ttySA", // for the IPAQs
                                                "ttyUSB", // for USB frobs
+                                               "ttyACM",// linux CDC ACM devices
                                                "rfcomm",       // bluetooth serial device
                                                "ttyircomm", // linux IrCommdevices (IrDA serial emu)
                                                };
```

The line marked with a plus is the one added.


Maintenant, compilez et installez!

``` shell
make && sudo make install
```

## Utilisez votre arduino

Vous pouvez maintenant lancer l'IDE arduino (commande arduino) et charger un programme de test. Vous pouvez sélectionner votre carte dans la liste des périphérique, probablement sous le nom "ttyACM0".

## Quelques infos

On désactive l'option `--diseable-lockfile` pour faire disparaître des messages d'erreur parlant d'impossibilité d'écrire les fichiers de lock. On ajouter le bon fichier .h contenant `UTS_RELEASE` pour éviter de stupide erreur de compilation (le fichier dans le quel est définit la macro a changé récemment). Enfin, il est nécessaire de modifier le code de rxtx (lisez les commentaires, vous verez qu'on vous demande explicitement de rajouter les devices manquant, en vous proposant la liste de tous ceux possible, et elle est bien longue!) pour que vous puissiez utiliser ttyACMx. Une autre solution serait d'ajouter un lien symbolique dans /dev/ d'un ttyUSBx vers un ttyACMx, par exemple (ttyUSB figure dans le tableau où nous avons ajouter ttyACM).

Bref, en fin de compte, si vous ne parvenez pas à utiliser votre arduino et que vous avez une erreur du type `processing.app.SerialNotFoundException: Port série « /dev/ttyACM0 » non trouvé`, c'est peut-etre tout simplement que ttyACM ne figure pas dans les périphériques accepté par rxtx.

## Références :
 *  <http://arduino.cc/en/Guide/troubleshooting#toc1> Drivers / Linux.
 *  <https://wiki.archlinux.org/index.php/Arduino>
