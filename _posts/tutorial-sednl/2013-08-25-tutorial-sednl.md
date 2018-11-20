---
layout: post
title: Tutoriel d'initiation à SEDNL
description: Initiation à la bibliothèque réseau SEDNL.
author: Jérémy Cochoy
date: 2013-08-25 +0100
categories: sednl network programming c++ software
lang: fr
...

Ce court tutoriel vas vous apprendre à utiliser la bibliothèque réseau SedNL.
Grâce à cette bibliothèque facile et rapide à prendre en main, nous réaliseront
un client et serveur de chat, ainsi qu'une application de dessin collaborative.

## Installation de SedNL

### Récupération des sources

Vous pouvez obtenir les sources de la bibliothèque sur Git Hub.

On supposera l'utilisation de git sous linux, ou bien de msysgit sous windows.

Votre client git en main, et une fois dans le répertoire de travail qui
vous convient, récupérez les sources avec les commandes :

``` shell
git clone http://github.com/Zenol/sednl.git
cd sednl
```

Ensuite, utilisez cmake pour générer les makefile de votre projet.

Commençons par créer un répertoire pour la compilation.

``` shell
mkdir build
cd build
```

Pour le moment, VC++ ne supporte pas encore le C++11, et il n'est
malheureusement pas possible de compiler SedNL avec le compilateur de
Microsoft. La procédure pour utiliser MinGW est légèrement plus compliquée.

Sous linux :

``` shell
cmake ..
```

Sous Windows pour utiliser MinGW :

``` shell
cmake -G "MinGW Makefiles" ..
```

Vous pouvez éventuellement personnaliser le build en exécutant `cmake-gui` sous
linux ou `cmake-gui.exe` sous Windows.

Vous pouvez maintenant lancer la compilation avec `make` sous linux,
et `mingw32-make` sous Windows avec mingw.

Une fois la compilation terminé, vous obtenez une bibliothèque statique
`sednl-1.0.a` et une dynamique `sednl-1.0.so`. (.lib et .dll sous windows).

C'est terminé, nous allons pouvoir compiler notre premier programme.

## Service de log d'extraits de code

Pour nos premiers pas avec cette bibliothèque, nous allons réaliser une
application cliente capable d'envoyer un extrait de code lue depuis l'entrée
standard vers un serveur qui loggera tout les extraits reçus, sur sa sortie
standard.

### Le client

Commençons par le client. Il doit recevoir un extrait de code sur son entrée
standard, puis l'envoyer au serveur.

``` cpp
#include <SEDNL/sednl.hpp>
#include <iostream>

using namespace SedNL;

// Read input from stdin
std::string get_input()
{
    std::string line;
    std::string output;

    while (std::getline(std::cin, line))
        output += line;

    return output;
}

int main(int argc, char* argv[])
{
    if (argc < 2)
    {
        std::cout << argv[0] << " listing_name" << std::endl;
        return EXIT_FAILURE;
    }

    try
    {
        //Connect to server port 4280
        TCPClient client(SocketAddress(4280, "localhost"));

        //Read the input
        std::string input = get_input();

        //Send the code to log
        client.send(make_event("log_that", argv[1], input));

        client.disconnect();
    }
    catch(...)
    {
        std::cout << "Failed to connect" << std::endl;
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

Si le fichier se nome « client.cpp », vous pouvez le compiler avec la ligne
`clang++ -std=c++11 -I/usr/local/include/sednl-1.0/ -lsednl-1.0 -lpthread client.cpp`.

La première fonction `get_input` lit l'entrée standard et stocke tout l'extrait de code dans une chaîne qu'elle renvoie.

Par la suite, on demande à ce que le premier argument du programme soit le nom
de l'extrait de code (`argv[0]`).

Enfin, c'est à la ligne 26 que commence vraiment notre programme.
On commence par ouvrir une connexion vers `localhost`. C'est l'ordinateur
qui exécute ce programme. On peux aussi indiquer « 127.0.0.1 ».
Le port sur le quel on cherche à atteindre le serveur est le port « 4280 ».
Un port n'a pas de réalitée matériel, c'est juste une façon de distinguer
les différentes connexions ouverte vers un ordinateur.
Le choix du numéro de port est arbitraire, même si l'on cherchera à
éviter d'utiliser les plus communs (comme 80 pour http,
20 et 21 pour ftp, etc.) ainsi que les valeurs inférieurs à 1024, car
certains système requière des privilèges administrateur pour les utiliser.

La ligne 35 fait deux choses. D'abord, `make_event` fabriquer un objet
de type `Event` qui contient :

1)  Le nom de l'évènement envoyé au serveur. Ici, `"log_that"`.
2)  Le nom sous le quel l'extrait de code sera enregistré. Ici, `argv[1]`.
3)  L'extrait de code à logger. Ici, `input`.

Une fois l'évènement crée, il est envoyé avec par la fonction membre `send`.

Enfin, la connexion est fermée à la ligne 37.

Si le serveur est indisponible, la connexion échoue, et une exception est
levée. Elle est alors attrapée à la ligne 39.

### Le serveur

Le serveur n'est pas beaucoup plus compliqué. Il doit attendre de nouvelle
connections, et écrire sur la sortie standard le texte reçu. Attention,
il faudra bien penser à gérer le cas où deux clients envoie un message
au même moment. Pour ça, on utilisera une mutex.



``` cpp
#include <SEDNL/sednl.hpp>
#include <iostream>
#include <mutex>

using namespace SedNL;

//Locked when writing to stdout
std::mutex log_mutex;

void on_log_that(Connection&, const Event& event)
{
    std::string log_name;
    std::string log_content;

    PacketReader reader = event.get_packet();
    reader >> log_name >> log_content;

    log_mutex.lock();
    std::cout << "--------------------" << std::endl;
    std::cout << ": " << log_name << std::endl;
    std::cout << "--------------------" << std::endl;
    std::cout << log_content << std::endl;
    log_mutex.unlock();
}

int main(int argc, char* argv[])
{
    try
    {
        TCPServer server(SocketAddress(4280));

        EventListener listener(server);

        EventConsumer consumer(listener);
        consumer.bind("log_that").set_function(on_log_that);

        listener.run();
        consumer.run();

        // Sleep
        while (true)
            std::this_thread::sleep_for(std::chrono::milliseconds(1000));
        listener.join();
    }
    catch(...)
    {
        std::cout << "Failed to connect" << std::endl;
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```
