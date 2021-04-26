---
layout: post
title:  "scanf() pour les intimes"
categories: [technical, C]
excerpt: À la fonction scanf() on demande de réinterpreter tout ce qui est possible d'écrire avec un clavier en une suite de nombres à bases variables, de caractère, de chaînes de caractères, de pointeurs, le tout potentiellement parsemé d'espaces blancs. De ce fait, lorsqu'on arrive aux limites de l'interprétation, les résultats peuvent être inattendus.
---

«`scanf` marche pas.»

«`scanf` ne saisi pas correctement le nombre que j'ai écrit !»

«`scanf` saisi `-1` alors que j'ai écrit un grand nombre...»

«`scanf` retourne `1` alors que ça ne devrait pas.»

Vous vous reconnaissez ?
Moi aussi.
`scanf` est une fonction «de base» mais très riche en fonctionnalités et qui doit s'acquitter de ses tâches au mieux.
À cette fonction on demande de réinterpreter tout ce qui est possible d'écrire avec un clavier en une suite de nombres à bases variables, de caractère, de chaînes de caractères, de pointeurs, le tout potentiellement parsemé d'espaces blancs.
De ce fait, lorsqu'on arrive aux limites de l'interprétation, les résultats peuvent être inattendus.

Dans vos premiers exercices, on vous demande de saisir des nombres et, en bon professeur, je vous demande de tester vos programmes avec des valeurs attendues, des valeurs extrêmes voire des valeurs invalides.
Mais comment bien prendre tous les cas en compte ?
Comment se protéger de l'utilisateur pathologique qui nous veut du mal ?

  - [Retour sur quelques notions](#retour-sur-quelques-notions)
      - [`int`](#int)
      - [`long int` ou `long`](#long-int-ou-long)
      - [`INT_MAX` et `LONG_MAX`](#int_max-et-long_max)
  - [Faisons une expérience](#faisons-une-expérience)
  - [Que s'est-il passé ?](#que-sest-il-passé-)
  - [Comment `scanf` essaie d'interpréter un nombre](#comment-scanf-essaie-dinterpréter-un-nombre)
  - [«`errno` ? On a pas vu ça en classe...»](#errno--on-a-pas-vu-ça-en-classe)
  - [Mais avec votre programme, `scanf` retourne toujours `1` et dans ce cas-ci `1` ça veux dire «J'ai bien saisi *une* valeur.» C'est du bidon, alors ?](#mais-avec-votre-programme-scanf-retourne-toujours-1-et-dans-ce-cas-ci-1-ça-veux-dire-jai-bien-saisi-une-valeur-cest-du-bidon-alors-)
  - [Et si on change `%d` pour `%ld` ?](#et-si-on-change-d-pour-ld-)
  - [Alors, quel est le code «parfait» pour saisir un nombre ?](#alors-quel-est-le-code-parfait-pour-saisir-un-nombre-)
  - [Rien d'autre ?](#rien-dautre-)

## Retour sur quelques notions

Concentrons-nous sur les nombres, leurs types et leurs définitions en C.

#### `int`

Un nombre entier.
L'architecture de nos ordinateurs modernes nous donne des `int`s d'une taille de 32 bits.
Les valeurs possibles d'un `int` vont donc de `-2147483648` à `2147483647`.

Notons au passage que la valeur maximale se situe entre 10<sup>9</sup> et 10<sup>10</sup>.

#### `long int` ou `long`

On peut qualifier le type `int` avec des «adjectifs».
Le type `int` peut être qualifié avec `long`.
Cette qualification lui confère une taille deux fois plus grande qu'un simple `int` : 64 bits.
`long` est un raccourci que le compilateur nous permet d'écrire à la place de `long int`.
Les deux sont équivalents.
Les valeurs possibles d'un `long int` vont donc de `-9223372036854775808` à `9223372036854775807`.

Notons encore au passage que la valeur maximale se situe entre 10<sup>18</sup> et 10<sup>19</sup>.

#### `INT_MAX` et `LONG_MAX`

Vous n'avez pas à retenir ces valeurs par coeur.
Vous trouverez dans le fichier d'en-tête `<math.h>` des constantes prédéfinies qui les représentent.
Voici comment j'affecterais un `int` de la valeur maximale qu'un `int` puisse repésenter :

```
int y_a_pas_plus = INT_MAX;
```

## Faisons une expérience

Pour saisir un nombre que l'utilisateur de notre programme veuille bien nous donner, on utilise la fonction `scanf`.
Si l'utilisateur est gentil et que l'on nous donne des valeurs normales, tout va bien.
Mais si l'utilisateur est un professeur, c'est une autre histoire.
Que ce passe-t-il dans les cas extrêmes ?

Il n'y a rien de tel que la pratique.
Voici un programme simple qui saisi un nombre entier de type `int` (`%d` pour `scanf`), qui affice ce nombre à l'écran et qui se répète en boucle jusqu'à ce ~~que mort s'ensuive~~ qu'une erreur survienne. (Vous ne savez pas ce qu'est `errno` ? Ne vous inquiétez pas, je vais y revenir.)

À chaque saisie, ce programme affiche aussi le résultat de la fonction `scanf` ainsi que la valeur de `errno` et un message d'erreur (en anglais) qui est associé à la valeur d'`errno`.

```
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main()
{
    int valeur_saisie;
    int resultat_scanf;

    do
    {
        resultat_scanf = scanf("%d", &valeur_saisie);

        printf("resultat_scanf = %d, valeur_saisie = %d, errno = %d, strerr = %s\n", resultat_scanf, valeur_saisie, errno, strerror(errno));
    }
    while(!errno);
    
    return 0;
}
```
Je vais maintenant lancer ce programme à l'invite de commande en lui passant un fichier qui contient déjà plusieurs nombres.
Voici ce fichier que j'appelle `input.txt`, il contient des valeurs qui vont 10<sup>0</sup> à 10<sup>19</sup> :

```
1
10
100
1000
10000
100000
1000000
10000000
100000000
1000000000
10000000000
100000000000
1000000000000
10000000000000
100000000000000
1000000000000000
10000000000000000
100000000000000000
1000000000000000000
10000000000000000000
```

Et voici comment je combine les deux à l'invite de commande (le «`$`» est l'invite de commande sous Linux):

```
$./a.out < input.txt
```

Et voici le résultat (formatté visuellement pour une lecture façile) :

```
resultat_scanf = 1, valeur_saisie = 1,              errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10,             errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100,            errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000,           errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10000,          errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100000,         errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000000,        errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10000000,       errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100000000,      errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000000000,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1410065408,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1215752192,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = -727379968,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1316134912,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 276447232,      errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = -1530494976,    errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1874919424,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1569325056,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = -1486618624,    errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = -1,             errno = 34, strerr = Result too large
```

## Que s'est-il passé ?

Tout s'est bien déroulé jusqu'à un certain point.
Les nombres donnés ont été bien saisis et affichés à l'écran mais ça cafouille à partir de `10000000000` et puis après... «Faites vos jeux. Rien ne va plus !»
Des nombres affichés sans aucun lien avec les nombres donnés.
Et finalement «Results too large».

Déjà, j'attire votre attention sur le fait que le moment où ça part en cacahuètes est justement autour de la valeur de `INT_MAX`.
En effet : `1000000000` < `INT_MAX` < `10000000000`.
Il y a anguille sous roche...

## Comment `scanf` essaie d'interpréter un nombre

Quand un ingénieur ne sait pas, il va à la source.
La source de quoi ?
La source de `scanf` bien sûr !
`scanf` est une fonction comme une autre qui est implémentée avec du code écrit en C.
Ce code, il commence [ici](https://sourceware.org/git?p=glibc.git;a=blob;f=stdio-common/vfscanf-internal.c;h=95b46dcbeb55b1724b396f02a940f3047259b926;hb=HEAD#l275) et se termine à la toute fin du fichier.
Vous en avez eu une crise cardiaque ?
Moi aussi.

Faison plus simple et all ons plutôt voir [la documentation](https://en.cppreference.com/w/c/io/fscanf).
Déjà, il y a moins à digérer.
Quoique... ça reste quand même un peu étourdissant.
Allons à l'essentiel.
Voyez dans la table l'élément `d` de la colonne `Conversion specifier`.
On nous dit dans l'explication que le format du nombre sera le même que celui attendu par la fonction `strtol`.
On comprends qu'en fait, la fonction `scanf` délègue le travail d'interprétation des nombres entiers à la fonction `strtol`.
Fort bien.
Mais... attendez un instant !
«`strtol`» ?
«string to long» ?
On a demandé un `int`, pas un `long` !

Allons voir maintenant [la documentation](https://en.cppreference.com/w/c/string/byte/strtol) de la fonction `strtol`.
On y apprend que cette fonction convertit une chaîne de caractères en `long`.
Il n'y a pas de `int` qui tienne !
Ça sera un `long` un point c'est tout.

On y apprend aussi dans la section `Return value` que la valeur de `errno` sera affectée de `ERANGE` si la valeur convertie se retrouve hors des bornes d'un `long`.
En l'occurence, `LONG_MAX`. Déjà, le mystère commence à s'éclaircir.

Ce qu'il faut en conclure, c'est que la fonction `scanf` demande à la fonction `strtol` d'interpéter ce que l'utilisateur à écrit comme un nombre.
La fonction `strtol` s'exécute et retourne un nombre de type `long`.
C'est tout ce que `strtol` peut faire.
Et il n'existe pas de `strtoi` («string to int») auquel `scanf` pourrait faire appel.
C'est ainsi.
Par la suite, comme nous avons donné à la fonction `scanf` une variable de type `int` (`%d`), elle fait de son mieux.
Mais son mieux dans ce cas ci n'est pas très satisfaisant.
`scanf` ne fait que nous donner la «moitié» du `long`, c'est-à-dire seulement 32 des 64 bits qui le compose.
Eh bien oui, un `int` n'a que 32 bits pour stocker son information.
Du coup, on se retrouve avec un nombre qui est sans lien avec la valeur entrée par l'utilisateur.

## «`errno` ? On a pas vu ça en classe...»

`errno` est un mécanisme par lequel certaines fonctions communiquent qu'une erreur s'est produite.
C'est la cas pour des fonctions qui retourne un valeur d'un type qui ne s'y prête pas.
La fonction `fopen`, par exemple, qui sert à ouvrir un fichier modifie la valeur de `errno` si une erreur est survenue (le fichier n'existe pas, le fichier est interdit d'accès à l'utilisateur, etc.).
Voyez `errno` comme une variable globale, accessible en tout temps dans votre code.

Au début de l'exécution de votre programme, la valeur de `errno` sera toujours de `0`.
Par la suite, vous pouvez tester la valeur d'`errno`.
Si elle n'est pas `0`, c'est qu'une erreur quelconque s'est produite.
Les valeurs possibles que `errno` peut prendre sont décrites [ici](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/errno.h.html).
La documentation de `errno` est [ici](https://en.cppreference.com/w/c/error/errno).

## Mais avec votre programme, `scanf` retourne toujours `1` et dans ce cas-ci `1` ça veux dire «J'ai bien saisi *une* valeur.» C'est du bidon, alors ?

Alors si on regarde encore une fois dans la documentation de la fonction `scanf`, à la section [Return value](https://en.cppreference.com/w/c/io/fscanf#Return_value), on nous dit que la valeur de retour de la fonction reflète le «nombre d'arguments de réception affectés avec succès».

Alors je vais vous avouer que je me serais attendu à ce que `scanf` me retourne `0` dans le cas où le nombre donné dépasse les limites du type de la variable.
Ce serait pourtant facile pour `scanf` de déterminer qu'une situation de ce genre s'est produite.
Il apparaît que `scanf` fera tout son possible pour affecter une valeur, *quel qu'ellle soit*, à la variable donnée.
La variable est numérique, le tampon de saisie contient des données numériques, allez hop, j'affecte la variable avec ce que je peux et on parle plus !
Ce sera au programmeur de vérifier...

Ceci dit, si l'utilisateur nous donne une valeur qui n'est pas numérique *du tout*, alors on peut s'attendre que `scanf` retourne `0`.
Par exemple, si j'écris «banane», le résultat sera `0`.

Si on creuse un peu plus loin, la documentation du [standard ISO](https://web.archive.org/web/20181230041359if_/http://www.open-std.org/jtc1/sc22/wg14/www/abq/c17_updated_proposed_fdis.pdf) n'utilise pas les mots «avec succès».
En fait, tout ce qu'on nous dit c'est si la «conversion» ne peut être exécutée avec succès, le résultat est «non-défini».
Piètre consolation mais voilà où s'arrête les responsabilités de la fonction `scanf`.

## Et si on change `%d` pour `%ld` ?

Avec la fonction `scanf` on peut aussi demander à saisir un `long int` plutôt qu'un `int`.
En fait, c'est probablement ce qu'il y a de mieux à faire étant donné ce que nous avons appris.

Si je change mon programme pour y intégrer ce changment (dans le code, `int` devient `long` et `%d` devient `%ld`), voici les résultats que j'observe :

```
resultat_scanf = 1, valeur_saisie = 1,                      errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10,                     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100,                    errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000,                   errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10000,                  errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100000,                 errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000000,                errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10000000,               errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100000000,              errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000000000,             errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10000000000,            errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100000000000,           errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000000000000,          errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10000000000000,         errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100000000000000,        errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000000000000000,       errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 10000000000000000,      errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 100000000000000000,     errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 1000000000000000000,    errno = 0, strerr = Undefined error: 0
resultat_scanf = 1, valeur_saisie = 9223372036854775807,    errno = 34, strerr = Result too large
```

Déjà on observe de meilleurs résultats jusqu'au «débordement» mais ça, c'était attendu.
Nous avons atteind les limites d'un `long int`.

## Alors, quel est le code «parfait» pour saisir un nombre ?

Je vous ai assez fait languir.
Voici comment j'écrirais aujourd'hui un programme qui saisi une valeur avec `scanf` et qui confirme que cette valeur se situe entre deux bornes données :

```
#include <stdio.h>
#include <errno.h>
#include <stdint.h>
#include <limits.h>

int main()
{
    long valeur_saisie;
    long const minimum_permis = 0;          // Pourrait aussi être une autre valeur de votre choix.
    long const maximum_permis = INT_MAX;    // Pourrait aussi être une autre valeur de votre choix. P. ex. '22', '100', 'LONG_MAX', etc.
    int resultat_scanf;
    int const n_valeurs_attendues = 1;      // Nous demandons à `scanf` de saisir une valeur.

    printf("Saisissez un nombre compris entre %ld et %ld : ", minimum_permis, maximum_permis);
    resultat_scanf = scanf("%ld", &valeur_saisie);
    if(errno == 0 && resultat_scanf == n_valeurs_attendues && valeur_saisie >= minimum_permis && valeur_saisie <= maximum_permis)
    {
        // Tout s'est bien passé. La variable `valeur_saisie` peut être utilisée.
        printf("%ld\n", valeur_saisie);
    }
    else
    {
        // Une erreur quelconque est survenue. On ne peut faire confiance à la valeur de `valeur_saisie`.
        printf("errno = %d, resultat_scanf = %d\n", errno, resultat_scanf);
    }

    return 0;
}
```

## Rien d'autre ?

Considérez utiliser les fonctions `fgets` suivi de `strtol` pour plus de contrôle.

Et je vous laisse avec ça.