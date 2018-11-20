---
layout: post
title: 'SedNL : Simple Event Driven Network Library'
description: Présentation de la bibliothèque C++ SEDNL.
author: Jérémy Cochoy
date: 2013-08-21 +0100
categories: sednl network programming c++ software
lang: fr
...

[SedNL](https://github.com/Zenol/sednl "SedNL repository") est une bibliothèque
réseau écrite en C++11, principalement destinée à la réalisation d’applications
réseau dont le protocole est basé sur la notion d'évènements :
Les clients et le serveur communiquent via des évènements,
de la forme : `"login" { username : "raccoon", password : "leaf" }`.

SedNL est sous [licence ZLib](https://github.com/Zenol/sednl/blob/master/LICENCE),
ce qui autorise une utilisation commerciale / closed source, sans obligation
de redistribution de la licence avec les binaires. Elle autorise aussi la modification
de la bibliothèque pour l'intégrer à vos projets, sans obligation de publier vos
modifications. Il n'y a pas non plus de comportement viral comme celui
de la GPL : la licence est donc compatible avec la plupart
des autres licences.

De nombreux jeux utilisent un système d'évènements pour leur mode réseau.
Par exemple, voici [la documentation d'un SDK de Valve](https://developer.valvesoftware.com/wiki/Networking_Events_%26_Messages)
qui fournit un mécanisme d'évènements.

L'API de SedNL a été conçue pour être __simple__, __concise__, et rapide à
apprendre (moins d'un quart d'heure pour un développeur expérimenté).
Elle convient parfaitement tant à des projets d’envergure moyenne que des
projets de taille plus conséquente.

Un court exemple d'utilisation du mécanisme d'évènement coté serveur :
``` cpp
#include <SEDNL/sednl.hpp>

using namespace SedNL;

void on_disconnect(Connection&);
void on_connect(Connection&);
void on_event(Connection&, const Event&);

TCPServer server(SocketAddress(4242));
EventListener listener(server);
listener.on_connect().set_function(on_connect);

EventConsumer consumer(listener);
consumer.on_disconnect().set_function(on_disconnect);
consumer.bind("login").set_function(on_login);

listener.run();
consumer.run();
```

Déterminer la bibliothèque la plus adaptée à votre projet est
une étape critique : un mauvais choix se répercutera plusieurs mois
plus tard par un coût important, nécessitant souvent le
ré-usinage(refactoring) d'une quantité importante de code, et donc
une énorme perte de temps (sans compter que c'est l'occasion rêvée
pour de nombreux bugs qui voudraient élire domicile dans votre code).
C'est pourquoi la documentation vous met tout de suite en face de
l'architecture type assumée par SedNL : Une communication via des
évènements, procédés de manière asynchrone, avec la garantie que
l'évènement <i>connected</i> sera le premier exécuté à la connexion
d'un client. À partir de cet instant, aucun ordre dans l'appel de
vos callback pour les évènements provenant d'un même utilisateur
ne peut être supposé. En contrepartie, vous pouvez contrôler quels
threads vous allouez à quels évènements.

SedNL fournis aussi des outils légers pour sérialiser et envoyer
de façon fiable les instances de vos classes.

Philosophie particulière de cette bibliothèque : la règle est que même
dans des conditions difficiles (plus de mémoire disponible, limite de
connections ouvertes, exceptions levées par vos callback, dysfonctionnement
du système dans la gestion des threads, ...) la bibliothèque doit faire tout
ce qui est possible pour conserver un comportement normal.
Toutes les exceptions "std::bad_alloc" sont interceptées et traitées
intelligemment afin de prévenir un malencontreux crash.
En cas de file d'évènements pleine ou d'un manque de mémoire,
les évènements sont temporairement ignorés.
Cela vous permet de ne pas perdre les données de vos utilisateurs encore
en mémoire, et de construire des applications serveur fiables à uptime élevé.
C'est, selon moi, ce que l'on doit attendre de toute vraie bibliothèque réseau.

Il est possible que suite à certaines contraintes techniques SedNL ne soit
pas la bibliothèque la plus adaptée à votre projet. Une lecture rapide de
la documentation devrait vous permettre de savoir si SedNL est adapté à
votre projet, ou si vous devez préférer une autre bibliothèque.
Par exemple, SedNL n'est pas du tout adapter [à de la RCP](http://en.wikipedia.org/wiki/Remote_procedure_call).
Même si le mécanisme d'évènement est ce que vous recherchez,
d'autres alternative C basé aussi sur un mécanisme d'évènement
comme libev / libevent offrent un contrôle plus fin,
au coût d'une API plus complexe et moins intuitive.
Gardez aussi à l’esprit que le design encouragé par SedNL devrait
s'adapter à de nombreuses bibliothèques fournissant un mécanisme
d'évènement, et que la licence autorise diverses modifications
pour des besoins spécifiques.
