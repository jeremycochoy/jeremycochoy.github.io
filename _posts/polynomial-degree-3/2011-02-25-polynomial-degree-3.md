---
layout: post
title: De la résolution d'équations polynomial de degré 3
description: Implémentation de la résolution d'équation polynomiales
  de degré 3. Peut être appliqué par exemple en raytracing.
author: Jérémy Cochoy
date: 2011-02-25 +0100
categories: maths c programming software
lang: fr
math: true
...

## La recherche de racine pour des polynômes du 3ème degré

$\Delta = b^2 - 4ac$, voici une formule que la plupart d'entre vous on appris très jeunes
et dont la simple lecture évoque immédiatement la résolution de polynômes du second degré.

Mais vous êtes vous déjà pausé la question de la résolution d'équations mettant en jeu des polynômes du troisième degré, ou pire, d'un degré encore plus élevé?

Sachez que pour les dégrées strictement supérieur à 4, les équations ne sont alors plus "soluble par radicaux". Ce que l'on pourrais résumer à "finis les belles formules magiques au dessus du degré 4".

Dans cette article, nous nous intéresserons à la méthode de Tartaglia-Cardan, du nom de ces deux célèbres mathématiciens, respectivement a l'origine de la méthode et de sa publication. (Je vous invite a en apprendre plus sur nos deux protagoniste, ainsi que Ferrari - le secrétaire de cardan - à qui l'on dois la résolution des équations de degré 4 par radicaux.)

Nous chercherons à adapter la méthode "papier" pour une application informatique, affin d'obtenir une valeur réel avec une précision convenable. Nous ne chercherons donc pas la réponse "exacte" comme l'on peut la trouver avec un crayon et beaucoup de papier.

Sachez que résoudre des équations de degré 3 peut avoir de nombreuse applications, et que probablement certains de mes lecteurs souhaiteront la mettre en œuvre dans le cadre d'un raytracer d'ici la fin de l'année scolaire. Plus que de donner une implémentation, je souhaite ici détailler la logique qui en est à l'origine.

## Étape 1 ) Changement de variable

La méthode de cardan ne s'applique qu'à des polynômes de la forme $Z^3 + pZ + q = 0$ et nous cherchons une solution pour tout polynôme de la forme $aX^3 + bX^2 + cX + d = 0$

On considéreras donc deux cas : Notre polynôme a son coefficient du second degré nulle, et dans ce cas on n'effectue pas de changement de variable, on le rend juste unitaire(diviser par a). Dans le cas contraire, on pauseras $X = Z - \frac{b}{3a}$, ce qui nous amène a calculer à calculer $p = - \frac{b^2}{3a^2} + \frac{c}{a}$ et $q = \frac{b}{27a}\left(\frac{2b^2}{a^2}-\frac{9c}{a}\right)+\frac{d}{a}$ (en développant le polynôme $a(Z - \frac{b}{3a})^3 + b(Z - \frac{b}{3a})^2 + c(Z - \frac{b}{3a}) + d = 0$) , et par la suite ajouter $\frac{b}{3a}$ a nos solutions.

Voici le code c correspondant :
```c
int cardan(float a, float b, float c, float d, float resultat[3])
// -&gt; resultat est notre tableau de solutions
//Retourne le nombre de racines trouvé
{
  //Nos coefs
  float p;
  float q;
  //Ce que nous ajouterons a nos solution, si le changement de variable a eu lieu
  float correction = 0;

  //Si il n'y a pas besoin de changement de variable
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

Nous entrons enfin dans le vif du sujet. Comment allons nous maintenant trouver les racines de notre polynôme? Pour commencer, nous savons qu'il existe au moins une racine réel (Un polynôme de degré impaire admet toujours au moins une racine réel) que l'on peut écrire sous la forme v + u, v et u deux réels. (Car, évidemment, rien ne nous empêche d'écrire 3 comme 2+1, ou 3+0!)

A ce stade, si nous développons notre polynôme avec u+v, nous obtenons $v^3+u^3+(3uv+p)(u+v)+q=0$ (modulo une factorisation par (u+v) ).
Puisque nous pouvons "répartir" notre solution "un peut dans u, le reste dans v" comme nous le souhaitons, rien ne nous empêche de fixer $3uv+p = 0$ (ou encore $uv = \frac{-p}{3u}$

On remarque alors que notre polynôme se simplifie, laissant apparaître $u^3 + v^3= -q$

En d'autre termes, notre solution est donc solution du système $\begin{cases}u^3+v^3&=-q\\ u^3v^3&=-\frac{p^3}{27}\end{cases}$ (et doivent respecter en plus la condition $3uv+p = 0$).

Vous remarquerez alors que connaissant le produit et la somme de deux nombres a et b, nous pouvons nous servir du polynôme du second dégrée $x^2 -(a + b)X + ab = 0$ qui a pour racines a et b. Appliquons ceci à $u^3$ et $v^3$

On résout donc $X^2+qX-\frac{p^3}{27}=0$. Attention, les solutions sont soit complexe, soit réels, soit réels double (u = v). Dans le cas de solutions pour u et v réels, leur racine cubique est un réel unique et nous aurons donc l'unique racine du polynôme. Dans le cas de solutions complexe(non nulle) pour u et v, nous aurons alors 3 racines cubiques pour u et 3 pour v. Nous verrons dans le paragraphe suivant comment calculer ces racines et trouver les bonnes combinaisons.

Voici l'implémentation pour les racines réels :
```c
  // ... next block :
  float delta = q*q + 4 * p*p*p / 27; //On applique delta = b^2+4ac où b=q, a=1, c = -p^3/27
  // Racine double réel =&gt; 3 solutions réels, dont une double
  if (delta == 0)
  {
    //En grattant beaucoup de papier, on parvient à calculer directement
    // les raine sans utiliser pow. Nous en profitons donc :
    float z1 = 3.f * q / p;
    float z2 = -3.f / 2.f * q / p;
    resultat[0] = z1 + correction;
    resultat[1] = z2 + correction;
    resultat[2] = resultat[1];
    //Finalement, ce polynôme a 3 racines :
    return 3;
  }
  // Une unique solution réel
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

## Étape 3 ) Racines complexe : un peu de géométrie

Multiplier un complexe $a$ par un complexe $b$, c'est obtenir
un complexe $c$ dont la norme est le produit $|a| |b|$,
et l'argument la somme $arg(a) + arg(b)$. D'un point de vue géométrique,
le produit $a b$ est une rotation de $a$ d'angle $arg(b)$ dans le sens trigonométrique,
et une homothétie par le scalaire $|b|$.
Où cela nous conduit-il? A la définition d'une racine cubique d'un complexe.
Soit $a = |a|e^{arg(a)}$ le complexe d'argument $arg(a)$ et de norme $|a|$,
on a pour racine cubique : $z_1 = \sqrt[3]{|a|}e^{arg(a)}$, $z_2 = \sqrt[3]{|a|}e^{arg(a)/3 + 2\pi/3}$
$z_3 = \sqrt[3]{|a|}e^{arg(a)/3 + 4\pi/3}$

De façon générale, les racines nièmes d'un complexe d'argument $arg(a)$ et de norme $|a|$ est
la rotation de $\frac{arg(a)}{n}$ et homothétie de scalaire $\sqrt[3]{|a|}$ des racines
nième complexe de 1 (pouvant elles même être considéré comme un polygone...)

Bref, de belles façons d'interpréter les choses, mais en quoi cela nous aide-il?

Et bien, si nous pouvions connaître l'argument d'un complexe et son module, nous pourrions calculer numériquement l'argument et le module de ses racines, et ainsi trouver les valeurs possible pour u et v. Considérons donc nos solutions complexe comme les points d'un plan. Le module et l'argument forment alors leur coordonnées polaire.
Nous avons le système suivant qui lit les coordonnées cartésiennes d'un point à ses coordonnées polaire : $$\begin{cases}x&=r cos(\alpha)\\ y&=r sin(\alpha)\end{cases}$$ De plus, nous pouvons calculer $r$, en appliquant le théorème de Pythagore.
Si l'on fait alors le quotient de x par y, on obtient $tan(\alpha)$. En fait, en étudiant le signe de x et de y, on peut connaitre le signe de alpha et donc utiliser la fonction arc tangente (`atan` en C). C'est  la réciproque de tangente,
modulo le bonne ensemble de définition.
Pour se simplifier la tache dans l'implémentation C du calcul de l'arc tangente, on peut utiliser la fonction `atan2(x, y)` qui effectue le quotient et tien compte des signes.

Encore un dernier détaille, et nous avons terminé. Les solutions que nous trouverons pour u et v peuvent se classer par paire de conjugué (les racines de u et v correspondant a la même racine cubique de 1). On remarque alors que les solutions seront toute bien réel. (La partie imaginaire de chacun des conjugué s'annulant.) Bien, passons à l'implémentation :
```c
  // ... last block :
    //On commence par prendre la valeur absolue de delta
    //Par la suite, on suppose delta &gt; 0.
    delta = -delta;
    //Petit rappelle, nos solutions sont de la forme : -q/2 +- i sqrt(delta)/2
    //Donc, de module au carré r^2 = (-q/2)^2 + (sqrt(delta)/2)^2 = q^2/4 + (delta)/4
    //FInalement r = sqrt(q^2 + delta) / sqrt(4)
    //Calculons le module :
    float r = sqrt(q*q + delta) / 2;
    //Maintenant, l'argument de nos complexe. Si deux complexe sont des conjugué, alors
    // ils on simplement leur argument de signe opposé. Comme on peux indifféremment inverser u et v, nous n'avons
    // pas besoin de tan2.
    // En simplifiant y / x, on trouve +-sqrt (delta) / q
    float alpha = atan(sqrt(delta) / q);
    // Nous calculons l'argument et le module d'une des racine complexe
    // (Celle au plus petit argument, c'est a dire Theta + 0PI/3)
    // Par la suite, alpha = theta et r = r^(1/3)
    r = pow(r, 1.f/3.f);
    alpha = alpha / 3.f;
    //Nous pouvons donc calculer nos trois racines réels, en passant de coordonnée polaire à des coordonnées cartésiennes.
    //De plus, nous simplifions et ne calculons pas la partie imaginaire, qui comme expliqué précédemment est nulle
    resultat[0] = 2 * r * cos(alpha) + correction; //nous avons effectivement r*cos(alpha) + r*cos(alpha)
    resultat[1] = 2 * r * cos(alpha + 2*PI/3) + correction;
    resultat[2] = 2 * r * cos(alpha + 4*PI/3) + correction;
    return 3;
  }
}
//Ouf! C'est terminé!
```

Pour conclure, un petit coup de wikipedia vous montreras que l'on peut simplifier le calcule du module, et n'utiliser qu'une simple racine carrée. Pensez aussi que si vous pouvez éviter de calculer deux fois la même chose, vous gagnerez en performance. Pensez a factoriser les expression de manière a ce que les produits/quotient soient résolut à la compilation et cherchez a minimiser les produits/quotient.

Notez que celons le contexte, on préférera considérer une racine double qu'une seule fois, affin d'éviter la redondance des calcules.

## Sur papier?

Sachez que la méthode papier consiste à effectuer le même raisonnement que nous avons
mené jusqu'au calcule du discriminant du polynôme qui nous permet de trouver a et b.
Dans le cas $\Delta <= 0$, on calculeras la racine cubique et on factoriseras le polynôme
pour faire apparaître les racines.
Dans le cas de complexes, on pourras être amener a chercher les racines d'un nouveau
polynôme du 3ème degré pour simplifier l'expression des racines.
Bien que la méthode de Tartaglia-Cardan soit une méthode qui fonctionne parfaitement,
il n'est pas toujours facile de calculer les racines d'un polynôme de degré 3.
