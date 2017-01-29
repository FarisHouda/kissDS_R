Le tidyverse
================

Le Tidyverse:
-------------

Le Tidyverse est un package qui regroupe une collections de packages utile pour la construction de "pipe" d'analyse de données. Ils ont été dévelopé pour être compatibles et fournir des outils pratiques pour les tâches les plus courantes dans le traitement de données. Dans cette promière rencontre avec le Tidyverse on abordera:

-   Tidyr: Pour rendre nos données Tidy
-   Magrittr: Pour contruire des pipe d'analyse lisibles et composables
-   readr - readxl: Pour lire les fichiers de données
-   tibble - dplyr: Pour manipuler les données tabulaire
-   hms - lubridate: Pour manipuler les données date
-   stringr: Pour manipuler les données textuelles
-   ggplot2: Pour faire des graphiques simples

Nous aurons une deuxième rencontre avec le Tidyverse pour explorer des aspets plus avancés des packages mentionnés ainsi que d'autre packages qui complètent le Tidyverse. On aura aussi une section dédiée à ggplot pour la visualisation avancée.

Le flux de travail que nous voulons construire dans ce cours est de lire les données, les traiter pour les nétoyer et construire les résultats de l'analyse, puis produire des graphiques qui pourrait faire partie d'un dashboard.

Pour commencer il faut installer le tidyverse si ce n'est pas fait puis le charger dans l'environnement de travail:

``` r
# Installer le package s'il n'est pas présent 
if(!require(tidyverse)){
  install.packages(tidyverse)
}
```

    ## Loading required package: tidyverse

    ## Loading tidyverse: ggplot2
    ## Loading tidyverse: tibble
    ## Loading tidyverse: tidyr
    ## Loading tidyverse: readr
    ## Loading tidyverse: purrr
    ## Loading tidyverse: dplyr

    ## Warning: package 'ggplot2' was built under R version 3.3.2

    ## Conflicts with tidy packages ----------------------------------------------

    ## filter(): dplyr, stats
    ## lag():    dplyr, stats

``` r
# Charger le package dans l'environnement
library(tidyverse)
```

Tidy data
---------

Pour commencer l'exploration du Tidyverse, on doit biensûr commencer par l'idée des "tidy data". Cette idée est décrite en détail dans [l'article de Hadley Whickham](http://vita.had.co.nz/papers/tidy-data.html) développeur d'une bonne partie des packages du tidyverse.

Les données sont dites tidy si elles respèctent 3 conditions:

-   Chaque variable constitue une colonne
-   Chaque observation constitue une ligne
-   Chaque valeur dans une seule case

Le package Tidyr permet de transformer des données au format "tidy". On explorera ce package au moment d'introduire les outils tabulaires.

\*\* Ajouter des examples de données non tidy et leur transformation en tidy.\*\*

Magrittr
--------

Le Tidyverse permet aussi d'introduire l'opérateur `%>%` (prononcé "pipe" en anglais) qui est trés utile pour construire des enchainement d'analyse de façon lisible. Cet opérateur est défini dans le package "Margrittr". L'opérateur %&gt;% permet d'enchaîner des fonctions les unes après les autres. Le comportement par défaut est que le résultat de la première fonction est passé comme premier argument à la fonction suivant. Ce résultat est ensuite passé comme premier paramètre à la fonction qui suit jusqu'à la fin de la chaîne de fonctions. donc:

-   `x %>% f` est équivalent à `f(x)`
-   `x %>% f %>% g` est équivalent à `g(f(x))`
-   `x %>% f %>% g %>% h` est équivalent à `h(g(f(x)))`

Si une fonction qui a besoin de plus d'un argument est après un `%>%`, la fonction reçoit sont premier paramètre par la chaîne de `%>%` qui précèdent et doit recevoir les autres paramètres directement. si on veut composer `creer(x)` avec `changer(x, y, z)` puis `modifier(x, a, b)` puis `alterer(x, c, d, f)`, on peut utiliser deux façons, avec ou sans `%>%`:

-   `x %>% creer %>% changer(y, z) %>% modifier(a, b) %>% alterer(c, d, f)`
-   `alterer(modifier(changer(creer(x), y, z), a, b), c, d, f)`

On voit l'avantage que l'apérateur `%>%` permet en terme de lisibilité. En plus, si on veut ajouter ou enlever une étape au milieu de la chaîne, c'est beaucoup plus clair. Il n'y a pas besoin de gérer les parenthèses.

On peut présenter la chaîne avec plus de lisibilité en utilisant des retours à la ligne:

``` r
x %>% 
  creer %>% 
  changer(y, z) %>% 
  modifier(a, b) %>% 
  alterer(c, d, f)
```

Exercice: enlevez l'étape `modifier()` des deux examples (avec et sans `%>%`).

Que faire quand le paramètre que doit recevoir la fonction de la chaîne n'est pas le premier paramètre de la fonction. Plus concrètement on veut former un pipe `f(x)` puis `g(y, x)`, et on veut passer le résultat de `f(x)` comme deuxième argument de `g(y, x)` (donc à l'endroit du x). Dans ce cas il faut invoquer la fonction g avec tous ses paramètres en spécifiant un `.` à la place du paramètre qui doit contenir le résultat de la chaîne jusqu'à ce point.

`x %>% f %>% g(y, .)` est équivalent à `g(y, f(x))`

Un example pratique:

``` r
iris %>% 
  summary %>%
  print
```

    ##   Sepal.Length    Sepal.Width     Petal.Length    Petal.Width   
    ##  Min.   :4.300   Min.   :2.000   Min.   :1.000   Min.   :0.100  
    ##  1st Qu.:5.100   1st Qu.:2.800   1st Qu.:1.600   1st Qu.:0.300  
    ##  Median :5.800   Median :3.000   Median :4.350   Median :1.300  
    ##  Mean   :5.843   Mean   :3.057   Mean   :3.758   Mean   :1.199  
    ##  3rd Qu.:6.400   3rd Qu.:3.300   3rd Qu.:5.100   3rd Qu.:1.800  
    ##  Max.   :7.900   Max.   :4.400   Max.   :6.900   Max.   :2.500  
    ##        Species  
    ##  setosa    :50  
    ##  versicolor:50  
    ##  virginica :50  
    ##                 
    ##                 
    ## 

Je peux ajouter une étape au milieur

``` r
iris %>% 
  subset(Species != "setosa") %>%
  summary %>%
  print
```

    ##   Sepal.Length    Sepal.Width     Petal.Length    Petal.Width   
    ##  Min.   :4.900   Min.   :2.000   Min.   :3.000   Min.   :1.000  
    ##  1st Qu.:5.800   1st Qu.:2.700   1st Qu.:4.375   1st Qu.:1.300  
    ##  Median :6.300   Median :2.900   Median :4.900   Median :1.600  
    ##  Mean   :6.262   Mean   :2.872   Mean   :4.906   Mean   :1.676  
    ##  3rd Qu.:6.700   3rd Qu.:3.025   3rd Qu.:5.525   3rd Qu.:2.000  
    ##  Max.   :7.900   Max.   :3.800   Max.   :6.900   Max.   :2.500  
    ##        Species  
    ##  setosa    : 0  
    ##  versicolor:50  
    ##  virginica :50  
    ##                 
    ##                 
    ## 

Un example avec la syntaxe du `.`

``` r
# créer une list des premiers du mois de l'année 2017
1:12 %>%
  paste("2017", ., "01", sep="-") %>%
  as.Date 
```

    ##  [1] "2017-01-01" "2017-02-01" "2017-03-01" "2017-04-01" "2017-05-01"
    ##  [6] "2017-06-01" "2017-07-01" "2017-08-01" "2017-09-01" "2017-10-01"
    ## [11] "2017-11-01" "2017-12-01"
