---
layout: post
title: De la résolution d'équations polynomiales de degré 3
description: Implémentation de la résolution d'équations polynomiales
  de degré 3. Peut être appliqué par exemple en raytracing.
author: Jérémy Cochoy
date: 2011-02-25 +0100
categories: maths c programming software
lang: fr
math: true
redirect_from:
  - /blog/fast-exponentiation/fr.html
...

## La recherche de racine pour des polynômes du 3ème degré

$$\Delta = b^2 - 4ac$$, voici une formule que la plupart d'entre vous ont appris très jeunes
et dont la simple lecture évoque immédiatement la résolution de polynômes du second degré.

Mais vous êtes-vous déjà posé la question de la résolution d'équations mettant en jeu des polynômes du troisième degré, ou pire, d'un degré encore plus élevé?

Sachez que pour les degrés strictement supérieurs à 4, les équations ne sont alors plus "solubles par radicaux". Ce que l'on pourrait résumer à "finis les belles formules magiques au dessus du degré 4".

Dans cet article, nous nous intéresserons à la méthode de Tartaglia-Cardan, du nom de ces deux célèbres mathématiciens, respectivement à l'origine de la méthode et de sa publication. (Je vous invite à en apprendre plus sur nos deux protagonistes, ainsi que Ferrari - le secrétaire de Cardan - à qui l'on doit la résolution des équations de degré 4 par radicaux.)

Nous chercherons à adapter la méthode "papier" pour une application informatique, afin d'obtenir une valeur réelle avec une précision convenable. Nous ne chercherons donc pas la réponse "exacte" comme l'on peut la trouver avec un crayon et beaucoup de papier.

Sachez que résoudre des équations de degré 3 peut avoir de nombreuses applications, et que probablement certains de mes lecteurs souhaiteront la mettre en œuvre dans le cadre d'un raytracer d'ici la fin de l'année scolaire. Plus que de donner une implémentation, je souhaite ici détailler la logique qui en est à l'origine.

## Étape 1 ) Changement de variable

La méthode de Cardan ne s'applique qu'à des polynômes de la forme $$Z^3 + pZ + q = 0$$ et nous cherchons une solution pour tout polynôme de la forme $$aX^3 + bX^2 + cX + d = 0$$

On considérera donc deux cas : notre polynôme a son coefficient du second degré nul, et dans ce cas on n'effectue pas de changement de variable, on le rend juste unitaire (diviser par a). Dans le cas contraire, on posera $$X = Z - \frac{b}{3a}$$, ce qui nous amène à calculer $$p = - \frac{b^2}{3a^2} + \frac{c}{a}$$ et $$q = \frac{b}{27a}\left(\frac{2b^2}{a^2}-\frac{9c}{a}\right)+\frac{d}{a}$$ (en développant le polynôme $$a(Z - \frac{b}{3a})^3 + b(Z - \frac{b}{3a})^2 + c(Z - \frac{b}{3a}) + d = 0$$) , et par la suite ajouter $$\frac{b}{3a}$$ à nos solutions.

Voici le code c correspondant :
```c
int cardan(float a, float b, float c, float d, float resultat[3])
// -&gt; resultat est notre tableau de solutions
//Retourne le nombre de racines trouvées
{
  //Nos coefs
  float p;
  float q;
  //Ce que nous ajouterons à nos solutions, si le changement de variable a eu lieu
  float correction = 0;

  //S'il n'y a pas besoin de changement de variable
  if (b == 0)
  {
    p = c / a;
    q = d / a;
  }
  //Sinon
  else
  {
    p = (-b * b / (3 * a) + c) / a;
    q = (b/(27 * a) * (2 * b * b / a - 9 * c) + d) / a;
    correction = -b/(3 * a);
  }
  //... see next block
```

## Étape 2 ) Système solution

Nous entrons enfin dans le vif du sujet. Comment allons-nous maintenant trouver les racines de notre polynôme? Pour commencer, nous savons qu'il existe au moins une racine réelle (un polynôme de degré impair admet toujours au moins une racine réelle) que l'on peut écrire sous la forme v + u, v et u deux réels. (car, évidemment, rien ne nous empêche d'écrire 3 comme 2+1, ou 3+0!)

À ce stade, si nous développons notre polynôme avec u+v, nous obtenons $$v^3+u^3+(3uv+p)(u+v)+q=0$$ (modulo une factorisation par (u+v) ).
Puisque nous pouvons "répartir" notre solution "un peu dans u, le reste dans v" comme nous le souhaitons, rien ne nous empêche de fixer $$3uv+p = 0$$ (ou encore $$uv = \frac{-p}{3u}$$

On remarque alors que notre polynôme se simplifie, laissant apparaître $$u^3 + v^3= -q$$

En d'autres termes, notre solution est donc solution du système $$\begin{cases}u^3+v^3&=-q\\ u^3v^3&=-\frac{p^3}{27}\end{cases}$$ (et doivent respecter en plus la condition $$3uv+p = 0$$).

Vous remarquerez alors que connaissant le produit et la somme de deux nombres a et b, nous pouvons nous servir du polynôme du second degré $$x^2 -(a + b)X + ab = 0$$ qui a pour racines a et b. Appliquons ceci à $$u^3$$ et $$v^3$$

On résout donc $$X^2+qX-\frac{p^3}{27}=0$$. Attention, les solutions sont soit complexes, soit réelles, soit réelles doubles (u = v). Dans le cas de solutions pour u et v réelles, leur racine cubique est un réel unique et nous aurons donc l'unique racine du polynôme. Dans le cas de solutions complexes (non nulles) pour u et v, nous aurons alors 3 racines cubiques pour u et 3 pour v. Nous verrons dans le paragraphe suivant comment calculer ces racines et trouver les bonnes combinaisons.

Voici l'implémentation pour les racines réelles :
```c
  // ... next block :
  float delta = q*q + 4 * p*p*p / 27; //On applique delta = b^2+4ac où b=q, a=1, c = -p^3/27
  // Racine double réelle =&gt; 3 solutions réelles, dont une double
  if (delta == 0)
  {
    //En grattant beaucoup de papier, on parvient à calculer directement
    // les racines sans utiliser pow. Nous en profitons donc :
    float z1 = 3.f * q / p;
    float z2 = -3.f / 2.f * q / p;
    resultat[0] = z1 + correction;
    resultat[1] = z2 + correction;
    resultat[2] = resultat[1];
    //Finalement, ce polynôme a 3 racines :
    return 3;
  }
  // Une unique solution réelle
  else if (delta &gt; 0)
  {
    //La racine carré de delta :
    delta = sqrt(delta);
    //Nos solutions u et v :
    float u = (-q - delta) / 2.f;
    float v = (-q + delta) / 2.f;
    //On calcule alors leur racine cubique :
    u = pow(u, 1.f/3.f);
    v = pow(v, 1.f/3.f);
    // notre racine est donc u + v + correction
    resultat[0] = u + v + correction;
    //Finalement, ce polynôme n'a qu'une racine :
    return 1;
  }
  else
  {
    //to be continue
```

## Étape 3 ) Racines complexes : un peu de géométrie

Multiplier un complexe $$a$$ par un complexe $$b$$, c'est obtenir
un complexe $$c$$ dont la norme est le produit $$|a| |b|$$,
et l'argument la somme $$arg(a) + arg(b)$$. D'un point de vue géométrique,
le produit $$a b$$ est une rotation de $$a$$ d'angle $$arg(b)$$ dans le sens trigonométrique,
et une homothétie par le scalaire $$|b|$$.
Où cela nous conduit-il? À la définition d'une racine cubique d'un complexe.
Soit $$a = |a|e^{arg(a)}$$ le complexe d'argument $$arg(a)$$ et de norme $$|a|$$,
on a pour racine cubique : $$z_1 = \sqrt[3]{|a|}e^{arg(a)}$$, $$z_2 = \sqrt[3]{|a|}e^{arg(a)/3 + 2\pi/3}$$
$$z_3 = \sqrt[3]{|a|}e^{arg(a)/3 + 4\pi/3}$$

De façon générale, les racines nièmes d'un complexe d'argument $$arg(a)$$ et de norme $$|a|$$ est
la rotation de $$\frac{arg(a)}{n}$$ et homothétie de scalaire $$\sqrt[3]{|a|}$$ des racines
nièmes complexes de 1 (pouvant elles-mêmes être considérées comme un polygone...)

Bref, de belles façons d'interpréter les choses, mais en quoi cela nous aide-t-il?

Et bien, si nous pouvions connaître l'argument d'un complexe et son module, nous pourrions calculer numériquement l'argument et le module de ses racines, et ainsi trouver les valeurs possibles pour u et v. Considérons donc nos solutions complexes comme les points d'un plan. Le module et l'argument forment alors leurs coordonnées polaires.
Nous avons le système suivant qui lie les coordonnées cartésiennes d'un point à ses coordonnées polaires : \[\begin{cases}x&=r cos(\alpha)\\ y&=r sin(\alpha)\end{cases}\] De plus, nous pouvons calculer $$r$$, en appliquant le théorème de Pythagore.
Si l'on fait alors le quotient de x par y, on obtient $$tan(\alpha)$$. En fait, en étudiant le signe de x et de y, on peut connaître le signe de alpha et donc utiliser la fonction arc tangente (`atan` en C). C'est  la réciproque de tangente,
modulo le bon ensemble de définition.
Pour se simplifier la tâche dans l'implémentation C du calcul de l'arc tangente, on peut utiliser la fonction `atan2(x, y)` qui effectue le quotient et tient compte des signes.

Encore un dernier détail, et nous avons terminé. Les solutions que nous trouverons pour u et v peuvent se classer par paires de conjugués (les racines de u et v correspondant à la même racine cubique de 1). On remarque alors que les solutions seront toutes bien réelles. (La partie imaginaire de chacun des conjugués s'annulant.) Bien, passons à l'implémentation :
```c
  // ... last block :
    //On commence par prendre la valeur absolue de delta
    //Par la suite, on suppose delta &gt; 0.
    delta = -delta;
    //Petit rappel, nos solutions sont de la forme : -q/2 +- i sqrt(delta)/2
    //Donc, de module au carré r^2 = (-q/2)^2 + (sqrt(delta)/2)^2 = q^2/4 + (delta)/4
    //Finalement r = sqrt(q^2 + delta) / sqrt(4)
    //Calculons le module :
    float r = sqrt(q*q + delta) / 2;
    //Maintenant, l'argument de nos complexes. Si deux complexes sont des conjugués, alors
    // ils ont simplement leur argument de signe opposé. Comme on peut indifféremment inverser u et v, nous n'avons
    // pas besoin de tan2.
    // En simplifiant y / x, on trouve +-sqrt (delta) / q
    float alpha = atan(sqrt(delta) / q);
    // Nous calculons l'argument et le module d'une des racines complexes
    // (Celle au plus petit argument, c'est-à-dire Theta + 0PI/3)
    // Par la suite, alpha = theta et r = r^(1/3)
    r = pow(r, 1.f/3.f);
    alpha = alpha / 3.f;
    //Nous pouvons donc calculer nos trois racines réelles, en passant de coordonnées polaires à des coordonnées cartésiennes.
    //De plus, nous simplifions et ne calculons pas la partie imaginaire, qui comme expliqué précédemment est nulle
    resultat[0] = 2 * r * cos(alpha) + correction; //nous avons effectivement r*cos(alpha) + r*cos(alpha)
    resultat[1] = 2 * r * cos(alpha + 2*PI/3) + correction;
    resultat[2] = 2 * r * cos(alpha + 4*PI/3) + correction;
    return 3;
  }
}
//Ouf! C'est terminé!
```

Pour conclure, un petit coup de Wikipédia vous montrera que l'on peut simplifier le calcul du module, et n'utiliser qu'une simple racine carrée. Pensez aussi que si vous pouvez éviter de calculer deux fois la même chose, vous gagnerez en performance. Pensez à factoriser les expressions de manière à ce que les produits/quotients soient résolus à la compilation et cherchez à minimiser les produits/quotients.

Notez que selon le contexte, on préférera considérer une racine double qu'une seule fois, afin d'éviter la redondance des calculs.

## Sur papier?

Sachez que la méthode papier consiste à effectuer le même raisonnement que nous avons
mené jusqu'au calcul du discriminant du polynôme qui nous permet de trouver a et b.
Dans le cas $$\Delta <= 0$$, on calculera la racine cubique et on factorisera le polynôme
pour faire apparaître les racines.
Dans le cas de complexes, on pourra être amené à chercher les racines d'un nouveau
polynôme du 3ème degré pour simplifier l'expression des racines.
Bien que la méthode de Tartaglia-Cardan soit une méthode qui fonctionne parfaitement,
il n'est pas toujours facile de calculer les racines d'un polynôme de degré 3.
