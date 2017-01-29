dplyr mini project
================

Le type "factor":
-----------------

Une variable de type `factor` est composée d'un nombre de niveaux (des chaînes de charactères). Les valeurs du vecteur doivent être parmi les niveaux définis, sinon celà se traduit par une valeur `NA`. Par exemple, on peut avoir un vecteur qui contient l'information du genre d'utilisateur d'un service:

``` r
genre <- c("homme", "femme", "homme", "femme", "femme")
genre
```

    ## [1] "homme" "femme" "homme" "femme" "femme"

On peut encoder la même information avec un vector `factor` avec deux niveaux, `homme` et `femme`:

``` r
genre_factor <- factor(genre, levels=c("femme","homme"))
genre_factor
```

    ## [1] homme femme homme femme femme
    ## Levels: femme homme

Le vecteur en lui même contient un entier qui identifie le niveau que contient la case. On peut voir ceci si on on transforme le factor en numeric:

``` r
genre_factor %>% 
  as.numeric
```

    ## [1] 2 1 2 1 1

Si les données encodée est de nature numéric, il faut faire attention à l'application direct de `as.numeric`. On va illuster ce point par l'example d'un vecteur encodant des années:

``` r
years <- c(2015, 2016, 2017, 2016, 2017, 2015, 2016,2015)
years_factor <- factor(years, levels = c(2015, 2016, 2017))
years_factor
```

    ## [1] 2015 2016 2017 2016 2017 2015 2016 2015
    ## Levels: 2015 2016 2017

La transformation direct en format numérique nous donne ceci:

``` r
years_factor %>% 
  as.numeric()
```

    ## [1] 1 2 3 2 3 1 2 1

Pas fraiment ce qu'on voulait, mais c'est cohérent avec le contenu du vecteur `factor` qui encode le niveau par un entier. Comment faire pour avoir les années en format numeric? Il faut combiner `as.character` et `as.numeric` (ou `as.integer` selon le besoin):

``` r
years_factor %>%
  as.character() %>%
  as.numeric()
```

    ## [1] 2015 2016 2017 2016 2017 2015 2016 2015

Modifier l'ordre des facteurs:
------------------------------

Reprenons l'exemple du vecteur `genre` précédent. La construction du vecteur de type `factor` sans spécifier de niveaux utilise l'ordre naturel des valeurs unique du vecteur. Dans ce cas c'est l'ordre alphabétique:

``` r
genre %>%
  factor()
```

    ## [1] homme femme homme femme femme
    ## Levels: femme homme

On peut décider d'organiser les niveaux des facteurs par ordre de première occurance dans les données. Dans ce cas on peut utiliser la fonction `fct_inorder()` du package `forcats` après la définition du vecteur facteur:

``` r
library(forcats)

genre %>%
  factor() %>%
  fct_inorder()
```

    ## [1] homme femme homme femme femme
    ## Levels: homme femme

On voit que dans ce cas, la première valeur du vecteur est `homme`, du coup le premier niveau utilise cette valeur.

On peut aussi classer les niveaux sur la base de la fréquence d'occurance dans le vecteur de donnée avec la fonction `fct_infreq()`.

``` r
genre %>%
  factor() %>%
  fct_infreq()
```

    ## [1] homme femme homme femme femme
    ## Levels: femme homme

On peut aussi choisir un, ou plusieurs niveaux pour être les premiers niveaux du vecteur à travers la fonction `fct_relevel()`.

``` r
years_factor %>%
  fct_relevel(2017)
```

    ## [1] 2015 2016 2017 2016 2017 2015 2016 2015
    ## Levels: 2017 2015 2016

Modifier les niveaux de facteurs:
---------------------------------
