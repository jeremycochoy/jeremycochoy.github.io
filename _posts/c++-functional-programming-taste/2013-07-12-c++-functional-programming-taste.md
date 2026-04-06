---
layout: post
title: C++ - Un goût de programmation fonctionnelle
description: Une introduction aux nouveaux aspects fonctionnels du C++ 11.
  Il y est question de lambdas et d'applications partielles.
author: Jérémy Cochoy
date: 2013-07-12 +0100
categories: c++ programming functional language
lang: fr
...

Dans ce billet, nous allons aborder quelques-unes des nouvelles fonctionnalités offertes par le C++11. Elles sont clairement inspirées de la vie dans le monde fonctionnel.

## Alpha, Beta, ... Kappa, Lambda!

Bien que portant le même nom, les _lambda_(C++ 11 powered) sont très différentes de leur homologues fonctionnelles, les _lambda fonctions_. Une lambda en C++11, c'est plutôt une intégration au langage des foncteurs.

``` cpp
class AFunctor
{
public:
  int operator (int a) { return a * b; }
  int b;
}

// ...
AFunctor f;
f.b = 42;
std::cout << f(2) << std::endl; //Display 84
```

Ré-écrivons la même chose avec la syntaxe d'une lambda, que l'on détaillera un peu plus loin (auto permet de laisser le compilateur inférer(deviner) le type).
``` cpp
int b = 42;
auto f = [&b](int a){return a * b};
```

Les lambda permettent de faire la même chose de façon plus légère, et ajoutent la sémantique de fonction (i.e. on ne peut pas confondre une lambda et un objet en lisant du code, alors qu'on '''pourrait''' avec un foncteur et un objet). Les lambda sont aussi plus proches d'une fonction anonyme, puisque certaines fonctions (constructeur, opérateur =), implicitement déclarées dans l'exemple ci-dessus (on peut faire f = g avec f et g deux AFunctor) n'existent pas (sont explicitement supprimées) pour les lambda.


Par exemple, le constructeur du type d'une lambda (on rappelle qu'en c++11, on peut obtenir le type de `f` avec `decltype(f)`. Par exemple `decltype(3.5f)` ou `std::vector v; decltype(v)`) n'existe pas.

Si f est une lambda, le code `decltype(f) g;` ne compilera pas. Pourtant, si `f` est un `AFunctor`, le code `decltype(f) g;` compilera et correspond à `AFunctor g;`.

Bon, qu'on se rassure, on peut quand même faire une copie d'une lambda :
``` cpp
auto f = [](){return 42};

auto        g1 = f;
//Or
auto        g2(f);
//Or
decltype(f) g3 = f;
//Or
decltype(f) g4(f);
```

## Comment fonctionne une lambda?

En fait, c'est très simple, et tout est décrit sur la page "Lambda" du site "CPPReference".

``` cpp
[ capture ] ( params ) mutable exception attribute -> ret { body }
```

Dans capture on trouve la façon dont les variables extérieures à la lambda sont capturées. Il y a deux modes de capture : par valeur, et par référence. Par défaut, [] signifie [=] qui veut dire "tout est récupéré par valeur", et le comportement est identique à une copie des variables (pour les objets comme `std::string`, c'est plutôt un `const std::string&` que vous recevez). On peut aussi spécifier [&] et toutes les variables sont récupérées par référence (et peuvent donc être modifiées depuis la lambda). Enfin, pour ceux qui apprécient la finesse, on peut expliciter le comportement pour chacune des variables, par exemple :
``` cpp
int a = 42;
std::vector<int> v;
std::string msg = "Hello";
//Take a by value
auto f1 = [](){return a;}
//Same, but tell explicitely the return type
auto f2 = []() -> int {return a;}
//Always the same
auto f3 = [a](){return a;}

//Take v by ref and a by value (return type is void)
auto f4 = [&b, a]() {v.push_back(a);}

//Take everybody by ref
auto f5 [&]() {v.push_back(a); a++; msg.push_back('!'); std::cout << msg << std::endl;}
```

La partie "exception" correspond aux spécifications du genre `throw (std::bad_alloc, MyExceptionType)` ou encore `noexcept` (no throw exception safety).

Si vous voulez modifier un objet obtenu par valeur, il vous faudra rajouter "mutable". Cela peut être très utile, si vous voulez appeler des méthodes non const sur une copie d'un objet dans le scope.
``` cpp
std::vector<int> v;
auto f = [v]() mutable {v.push_back(42); std::cout << v[0] << std::endl;}
```

Petite astuce parfois utile : si une lambda ne capture aucune variable, alors elle peut être convertie en pointeur de fonction.

## std::function :

Les std::function représentent des fonctions. Ils sont basés sur les templates variadiques (l'un des ajouts les plus puissants au langage), que l'on peut espérer disponibles sous VisualStudio d'ici 2039 (si l'équipe de microsoft ne prend pas de retard). Le constructeur des std::function autorise de les construire avec plus ou moins n'importe quoi (pointeur de fonction, pointeur de fonction membre, lambda, foncteur, ...).

En code :
``` cpp
//Lambda
std::function<int(int, int)> f = [](int a, int b) = {return a + b;}

//Function
void print(int a)
{
  std::cout << a << std::endl;
}
std::function<void(int)> f = print;

//Functor
struct Functor
{
    int b;
    int operator (int a) {return a + b;};
};

Functor func;
func.b = 3;    
std::function<int(int)> f = func;

//Member function
struct St
{
  int b;
  int sum(int a) { return a + b; };
}
std::function<void(const St&, int)> f = &St::sum;

```

## Application partielle.

Tout ça, pour en venir à vous parler de ```std::bind```. Quand on travaille avec des langages fonctionnels, on peut appeler une fonction avec seulement une partie de ses arguments. On parle d'_application partielle_. `std::bind` permet de reproduire ce comportement. Prenons une innocente fonction :
``` cpp
void display(int a, int b, int c)
{
  std::cout << "a : " << a << " - b :" << b << " - c : " << c << std::endl;
}
```

On peut alors construire, grâce à std::bind, différentes spécialisations de cette fonction :
``` cpp
//On fixe les trois arguments
std::function<void()> f = std::bind(display, 5, 6, 7);
f(); // Affiche a : 5 - b : 6 - c : 7
//On fixe les trois, et on en rajoute un qui sera ignoré
std::function<void(int)> f = std::bind(display, 5, 6, 7);
f(42); // Affiche a : 5 - b : 6 - c : 7

//Placeholders::_i désigne le i-ième argument lors de l’appel de f
std::function<void(int)> f = std::bind(display, 5, 6, std::placeholders::_1);
f(42); //  Affiche a : 5 - b : 6 - c : 42

//Ne fixe que le premier argument
std::function<void(int, int)> f = std::bind(display, 5, std::placeholders::_1, std::placeholders::_2);
f(10, 20); //  Affiche a : 5 - b : 10 - c : 20

//On peut changer l'ordre :
std::function<void(int, int)> f = std::bind(display, 5, std::placeholders::_2, std::placeholders::_1);
f(10, 20); //  Affiche a : 5 - b : 20 - c : 10

```

Bien entendu, on peut aussi faire des choses plus complexes (passage des arguments par référence avec `std::ref` et `std::cref` dans les arguments de bind, pointeurs de fonction membre, pointeurs vers membres, etc.).

Si vous vous demandez à quoi ça peut bien servir, eh bien dites-vous que là où on attend une callback avec une certaine signature (c'est le cas avec beaucoup d'outils de `<algorithm>`) vous avez maintenant la possibilité de spécialiser vos fonctions.

Pour ce qui est du coût, il est faible (selon les cas beaucoup de choses peuvent être optimisées lors de la compilation), et n'est un argument recevable que dans certains cas particuliers. Donc, à moins de faire du temps réel et de faire ce genre de manipulation dans les parties critiques, vous pouvez vous lâcher.

Voilà, j'espère vous avoir donné un petit aperçu de l'apport du c++11 en matière de manipulation des fonctions.

## Références :

* <http://en.cppreference.com/w/cpp/language/lambda>
* <http://en.cppreference.com/w/cpp/utility/functional/function>
