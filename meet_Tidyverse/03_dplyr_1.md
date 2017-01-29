dplyr Introduction
================

Tibble
------

Avant de nous attacker à `dplyr` nous allons d'abord nous interesser au package `tibble`. C'est un package qui isole la definition de la classe de tableau utilisée dans le `tidyverse` et faisait partie du package `dplyr`avant d'en être extrait pour améliorer la lisibilité du `tidyverse`.

`tibble` est une extension de la classe `data.frame` qui vise à ajuster certaines fonctionalités par default afin de les rendre plus adaptée à la fonçon dont on utilise les data frame. En effet, certains choix de fonctionalités de la classe `data.frame` ne sont tout simplement pas pratiques:

-   `tibble` ne change pas le type des données à la création, comparé à `data.frame` qui transforme par defaut les types `character` en `factor`
-   Une impression partielle des données du tibble (les 10 premières lignes) lors d'un `print` comparé à `data.frame` qui imprime tout
-   `tibble` utilise les noms de variables de façon stricte avec l'opérateur `$` et ne fait pas de matching partiel des noms de variables.
-   `tibble` un comportement fixe pour les opérateurs de selection `[` et `[[` qui retournent systématiquement un `tibble` pour le premier et un vecteur pour le second. Dans `data.frame` le type du resultat de ces opérateurs est variable.

Il est possible de passer d'une classe à l'autre en utilisant les fonction `as_tibble()` et `as.data.frame()`.

Dplyr Introduction
------------------

`dplyr` est un package dédié à la manipulation des tableaux (des tibbles pour être précis). Vous pouvez vous réfferer à cette [fiche sythétique](https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf) (en anglais) pour un résumé des fonctionalité et la syntaxe du package. Nous allons dans la suite introduire ces fonctionalités une à une.

`dplyr` donne accès à une collection de verbes pour manipuler les données tabulaires:

-   Ordonner les observations: `arrange()`
-   Selectionner des oservations: `filter()`
-   Selectionner des variables: `select()`
-   Créer de nouvelles variables: `mutate()`
-   Créer des groupes d'observations: `group_by()`
-   Calculer des résumés statistiques par variable: `summarize()`
-   Combiner plusieurs tables: `Join()`

on va utliser un jeu de données sur les ventes de jeux videos pour explorer dplyr. Le jeu de données est disponible sur [Kaggle](https://www.kaggle.com/rush4ratio/video-game-sales-with-ratings).

``` r
# specify colonnes types:
colonnes_types <- cols(
  Name = col_character(),
  Platform = col_character(),
  Year_of_Release = col_integer(),
  Genre = col_character(),
  Publisher = col_character(),
  NA_Sales = col_double(),
  EU_Sales = col_double(),
  JP_Sales = col_double(),
  Other_Sales = col_double(),
  Global_Sales = col_double(),
  Critic_Score = col_integer(),
  Critic_Count = col_integer(),
  User_Score = col_double(),
  User_Count = col_integer(),
  Developer = col_character(),
  Rating = col_character() )

# read data with specified colonnes types
vg_sales <- read_csv("./data/Video_Games_Sales_as_at_22_Dec_2016.csv", 
                     col_type = colonnes_types)
```

    ## Warning: 2694 parsing failures.
    ## row             col   expected actual
    ## 120 User_Score      a double      tbd
    ## 184 Year_of_Release an integer    N/A
    ## 302 User_Score      a double      tbd
    ## 378 Year_of_Release an integer    N/A
    ## 457 Year_of_Release an integer    N/A
    ## ... ............... .......... ......
    ## See problems(...) for more details.

``` r
# show data
vg_sales
```

    ## # A tibble: 16,719 <U+00D7> 16
    ##                         Name Platform Year_of_Release        Genre
    ##                        <chr>    <chr>           <int>        <chr>
    ## 1                 Wii Sports      Wii            2006       Sports
    ## 2          Super Mario Bros.      NES            1985     Platform
    ## 3             Mario Kart Wii      Wii            2008       Racing
    ## 4          Wii Sports Resort      Wii            2009       Sports
    ## 5   Pokemon Red/Pokemon Blue       GB            1996 Role-Playing
    ## 6                     Tetris       GB            1989       Puzzle
    ## 7      New Super Mario Bros.       DS            2006     Platform
    ## 8                   Wii Play      Wii            2006         Misc
    ## 9  New Super Mario Bros. Wii      Wii            2009     Platform
    ## 10                 Duck Hunt      NES            1984      Shooter
    ## # ... with 16,709 more rows, and 12 more variables: Publisher <chr>,
    ## #   NA_Sales <dbl>, EU_Sales <dbl>, JP_Sales <dbl>, Other_Sales <dbl>,
    ## #   Global_Sales <dbl>, Critic_Score <int>, Critic_Count <int>,
    ## #   User_Score <dbl>, User_Count <int>, Developer <chr>, Rating <chr>

### arrange & filter:

Les fonctions `arrange()` et `filter()` opèrent sur les lignes d'un tibble pour ordonner ou selectionner les rangées.

On va utiliser la fonction `arrange()` pour classer les observations autrement. par année (du plus récent au plus vieux) puis par platefrome puis par genre puis par nom:

``` r
vg_sales %>%
  arrange(desc(Year_of_Release), Platform, Genre, Name)
```

    ## # A tibble: 16,719 <U+00D7> 16
    ##                                                Name Platform
    ##                                               <chr>    <chr>
    ## 1                            Imagine: Makeup Artist       DS
    ## 2  Phantasy Star Online 2 Episode 4: Deluxe Package      PS4
    ## 3                  Brothers Conflict: Precious Baby      PSV
    ## 4  Phantasy Star Online 2 Episode 4: Deluxe Package      PSV
    ## 5                  Aikatsu Stars! My Special Appeal      3DS
    ## 6     Ansatsu Kyoushitsu: Assassin Ikusei Keikaku!!      3DS
    ## 7                             Azure Striker Gunvolt      3DS
    ## 8               Azure Striker Gunvolt: Striker Pack      3DS
    ## 9                   Cartoon Network Battle Crashers      3DS
    ## 10                               Disney Art Academy      3DS
    ## # ... with 16,709 more rows, and 14 more variables: Year_of_Release <int>,
    ## #   Genre <chr>, Publisher <chr>, NA_Sales <dbl>, EU_Sales <dbl>,
    ## #   JP_Sales <dbl>, Other_Sales <dbl>, Global_Sales <dbl>,
    ## #   Critic_Score <int>, Critic_Count <int>, User_Score <dbl>,
    ## #   User_Count <int>, Developer <chr>, Rating <chr>

la fonction `desc()` permet de specifier un ordre décroissant, alors que l'order par défault est croissant.

Pour selectionner des observations sur la base de conditions on utilise la fonction `filter` en specifiant des conditions à remplir. Seules les observations pour lesquelles les conditions donnent `TRUE` sont retenues. Nousallons nous interesser aux ventes du Genre Aventure de la plateform playstation (PS3 et PS4), entre les années 2005 et 2015, avec des ventes en region amerique du nord (NA\_Sales) supérieures à 0.1 millions.

``` r
vg_sales %>%
  # select needed data
  filter(Year_of_Release >= 2005, 
         Year_of_Release <= 2015, 
         Platform %in% c("PS3", "PS4"),
         Genre == "Adventure",
         NA_Sales >= 0.1) %>%
  # arrange observations
  arrange(desc(Year_of_Release), Platform, Genre, Name)
```

    ## # A tibble: 22 <U+00D7> 16
    ##                                                Name Platform
    ##                                               <chr>    <chr>
    ## 1                             Minecraft: Story Mode      PS3
    ## 2                      Back to the Future: The Game      PS4
    ## 3                             Minecraft: Story Mode      PS4
    ## 4                                        Until Dawn      PS4
    ## 5  Teenage Mutant Ninja Turtles: Danger of the Ooze      PS3
    ## 6                      The Walking Dead: Season One      PS4
    ## 7                      The Walking Dead: Season Two      PS4
    ## 8                                 The Wolf Among Us      PS4
    ## 9                                 Beyond: Two Souls      PS3
    ## 10          Metal Gear Solid: The Legacy Collection      PS3
    ## # ... with 12 more rows, and 14 more variables: Year_of_Release <int>,
    ## #   Genre <chr>, Publisher <chr>, NA_Sales <dbl>, EU_Sales <dbl>,
    ## #   JP_Sales <dbl>, Other_Sales <dbl>, Global_Sales <dbl>,
    ## #   Critic_Score <int>, Critic_Count <int>, User_Score <dbl>,
    ## #   User_Count <int>, Developer <chr>, Rating <chr>

Pour formuler des conditions plus complexes on peut utiliser les operateur booleens `&` (et), `|` (ou), `!` (negation), `xor`, `any` et `all`.

Il existe d'autre fonctions pour les opérations sur les observations que nous allons lister dans la suite:

-   `distinct()` enlève les observations dupliquées
-   `sample_frac()` selectionne une fraction du nombre d'observations au hasard, avec le ratio specifié. Par example `iris %>% sample_frac(0.1)` va selectionner au hazard 10 % des observations du jeux de données. L'option `replace` permet de choisir si l'echantillonage se fait par remplacement (`TRUE`) ou pas (`FALSE`).
-   `sample_n()` Va selectionner au hazard le nombre specifié d'observations. Par example `iris %>% sample_n(10)` va selectionner au hazard 10 observations. L'option `replace` permet de choisir si l'echantillonage se fait par remplacement (`TRUE`) ou pas (`FALSE`).
-   `slice()` permet de selectionner les observations dans la plage spéficiée. Par example `iris %>% slice(5:10)` retourne les observations 5 à 10 dans le jeux de données.
-   `top_n()` retourne toutes les observations correspondantes aux n valeurs les plus élevées de la colonne specifiée. Par example `iris %>% top_n(4, Petal.Width)`. Les 4 valeurs les plus élevées de Petal.Width sont `2.5, 2.5, 2.5, 2.4`, donc la fonction retourne toutes les observations correspondantes à 2.5 et 2.4 (donc 6 observations et non 4).

### select & mutate:

La fonction `select()`permet de selectionner les colonnes à garder dans le tableau de données. pour voir quelles sont les combinasons de plateformes et de genres présentes dans le jeux de données on va selectionner ces deux colonnes. onn en profite pour utiliser les fonctions `distinct()` et `arrange()` pour éliminer les doublons et classer les résultats:

``` r
vg_sales %>%
  select(Platform, Genre) %>%
  distinct() %>%
  arrange(Platform, Genre)
```

    ## # A tibble: 294 <U+00D7> 2
    ##    Platform      Genre
    ##       <chr>      <chr>
    ## 1      2600     Action
    ## 2      2600  Adventure
    ## 3      2600   Fighting
    ## 4      2600       Misc
    ## 5      2600   Platform
    ## 6      2600     Puzzle
    ## 7      2600     Racing
    ## 8      2600    Shooter
    ## 9      2600 Simulation
    ## 10     2600     Sports
    ## # ... with 284 more rows

Les tableaux de données peuvent contenir un nombre important de colonnes. Les specifier une par une peut vite devenir une tâche pénible. On a à disposision un certain nombre de fonctions pour gérer cet aspect:

-   `contains()` permet de selectionner les colonnes dont le nom contient la chaîne de charactères spécifiée. Par example `iris %>% select(Species, contains("Sepal"))`
-   `starts_with()` permet de selectionner les colonnes dont le nom commence par la chaîne de charactères specifiée. Par example `iris %>% select(Species, starts_with("Sepal"))`
-   `ends_with()` permet de selectionner les colonnes dont le nom se termine par la chaîne de charactères specifiée. Par example `iris %>% select(Species, ends_with("Length"))`
-   `matches()` permet de donner une expression régulière pour identifier les colonnes à garder. Plus sur ce sujet dans le section relative au traitement des chaîne de charactères.
-   `num_range()` permet de selectionner les colonnes dont le nom est composé par un nom avec un suffixe numérique. Par example `num_range("var_", 1:3)` permet de selectionner les colonnes `var_1`, `var_2` et `var_3`.
-   `:` (`Debut:Fin`) permet de determiner une plage de colonnes en spécifiant les noms des première et dernière colonnes. Par example `iris %>% select(Sepal.Length:Petal.Width)`
-   `one_of()` permet de selectionner les colonnes spécifiée par le vecteur de noms donné. Par example `iris %>% select(one_of(c("Species", "Sepal.Width")))`
-   `everything()` selectionne toutes le colonnes.

On a aussi besoin créer de nouvelles colonnes dans la table en utilisant les données existantes. Pour celà on peut utiliser la fonction `mutate()` ou `transmute()`. Le fonction `mutate()` crée la nouvelle variable tout en présérvant les variables initiales qui ont sérvit à la créer.

``` r
vg_sales %>%
  select(Name, NA_Sales) %>%
  mutate(NA_Sales_dlr = NA_Sales * 10**6)
```

    ## # A tibble: 16,719 <U+00D7> 3
    ##                         Name NA_Sales NA_Sales_dlr
    ##                        <chr>    <dbl>        <dbl>
    ## 1                 Wii Sports    41.36     41360000
    ## 2          Super Mario Bros.    29.08     29080000
    ## 3             Mario Kart Wii    15.68     15680000
    ## 4          Wii Sports Resort    15.61     15610000
    ## 5   Pokemon Red/Pokemon Blue    11.27     11270000
    ## 6                     Tetris    23.20     23200000
    ## 7      New Super Mario Bros.    11.28     11280000
    ## 8                   Wii Play    13.96     13960000
    ## 9  New Super Mario Bros. Wii    14.44     14440000
    ## 10                 Duck Hunt    26.93     26930000
    ## # ... with 16,709 more rows

La fonction mutate permet aussi de supprimer une variables en lui assignant la valeur `NULL`.

``` r
vg_sales %>%
  select(Name, NA_Sales) %>%
  mutate(NA_Sales_dlr = NA_Sales * 10**6,
         NA_Sales = NULL)
```

    ## # A tibble: 16,719 <U+00D7> 2
    ##                         Name NA_Sales_dlr
    ##                        <chr>        <dbl>
    ## 1                 Wii Sports     41360000
    ## 2          Super Mario Bros.     29080000
    ## 3             Mario Kart Wii     15680000
    ## 4          Wii Sports Resort     15610000
    ## 5   Pokemon Red/Pokemon Blue     11270000
    ## 6                     Tetris     23200000
    ## 7      New Super Mario Bros.     11280000
    ## 8                   Wii Play     13960000
    ## 9  New Super Mario Bros. Wii     14440000
    ## 10                 Duck Hunt     26930000
    ## # ... with 16,709 more rows

La fonction `transmute()` fait la même chose mais élimine les variables existantes.

``` r
vg_sales %>%
  select(Name, NA_Sales) %>%
  transmute(NA_Sales_dlr = NA_Sales * 10**6)
```

    ## # A tibble: 16,719 <U+00D7> 1
    ##    NA_Sales_dlr
    ##           <dbl>
    ## 1      41360000
    ## 2      29080000
    ## 3      15680000
    ## 4      15610000
    ## 5      11270000
    ## 6      23200000
    ## 7      11280000
    ## 8      13960000
    ## 9      14440000
    ## 10     26930000
    ## # ... with 16,709 more rows

On peut créer plusieurs variables dans la même instance de mutate, et aussi utiliser une variable qui vient d'être définie dans une formule après sa création, tout ça dans le même appel à mutate:

``` r
vg_sales %>%
  select(Name, NA_Sales, EU_Sales, Global_Sales) %>%
  mutate(NA_Sales_ratio = NA_Sales / Global_Sales,
         EU_Sales_ratio = EU_Sales / Global_Sales,
         Wester_Sales_ratio = NA_Sales_ratio + EU_Sales_ratio,
         NA_Sales = NULL,
         EU_Sales = NULL)
```

    ## # A tibble: 16,719 <U+00D7> 5
    ##                         Name Global_Sales NA_Sales_ratio EU_Sales_ratio
    ##                        <chr>        <dbl>          <dbl>          <dbl>
    ## 1                 Wii Sports        82.53      0.5011511     0.35090270
    ## 2          Super Mario Bros.        40.24      0.7226640     0.08896620
    ## 3             Mario Kart Wii        35.52      0.4414414     0.35923423
    ## 4          Wii Sports Resort        32.77      0.4763503     0.33353677
    ## 5   Pokemon Red/Pokemon Blue        31.37      0.3592604     0.28339178
    ## 6                     Tetris        30.26      0.7666887     0.07468605
    ## 7      New Super Mario Bros.        29.80      0.3785235     0.30671141
    ## 8                   Wii Play        28.92      0.4827109     0.31742739
    ## 9  New Super Mario Bros. Wii        28.32      0.5098870     0.24505650
    ## 10                 Duck Hunt        28.31      0.9512540     0.02225362
    ## # ... with 16,709 more rows, and 1 more variables:
    ## #   Wester_Sales_ratio <dbl>

On peut appliquer la même transformation à toute les colonnes en utilisant `mutate_each()`:

``` r
vg_sales %>%
  mutate_each(funs(dlr = . * 10**6), ends_with("Sales"))
```

    ## # A tibble: 16,719 <U+00D7> 21
    ##                         Name Platform Year_of_Release        Genre
    ##                        <chr>    <chr>           <int>        <chr>
    ## 1                 Wii Sports      Wii            2006       Sports
    ## 2          Super Mario Bros.      NES            1985     Platform
    ## 3             Mario Kart Wii      Wii            2008       Racing
    ## 4          Wii Sports Resort      Wii            2009       Sports
    ## 5   Pokemon Red/Pokemon Blue       GB            1996 Role-Playing
    ## 6                     Tetris       GB            1989       Puzzle
    ## 7      New Super Mario Bros.       DS            2006     Platform
    ## 8                   Wii Play      Wii            2006         Misc
    ## 9  New Super Mario Bros. Wii      Wii            2009     Platform
    ## 10                 Duck Hunt      NES            1984      Shooter
    ## # ... with 16,709 more rows, and 17 more variables: Publisher <chr>,
    ## #   NA_Sales <dbl>, EU_Sales <dbl>, JP_Sales <dbl>, Other_Sales <dbl>,
    ## #   Global_Sales <dbl>, Critic_Score <int>, Critic_Count <int>,
    ## #   User_Score <dbl>, User_Count <int>, Developer <chr>, Rating <chr>,
    ## #   NA_Sales_dlr <dbl>, EU_Sales_dlr <dbl>, JP_Sales_dlr <dbl>,
    ## #   Other_Sales_dlr <dbl>, Global_Sales_dlr <dbl>

### group\_by & summarize:

On a souvent besoins de calculer des résumés statistiques par type de observations. Par exemple, on peut vouloir obetenir les revenus moyen par genre de jeux, ou par type de plateforme. Pour calculer ce genre de résultats nous devons utiliser deux fonctions dans un pipe de ce type: `data %>% group_by() %>% summarise()`.

La fonction `group_by()` permet de séparer un tableau en plusieurs tableau sur la base des informations des colonnes spécifiées. Par example si on fait un group by sur la base du genre de jeux, la fonction séparera le tableau en une list de tableaux chacun contenant les données correspondantes à un genre (`Action`, `Adventure`, `Fighting` ...):

``` r
vg_sales %>%
  group_by(Genre)
```

    ## Source: local data frame [16,719 x 16]
    ## Groups: Genre [13]
    ## 
    ##                         Name Platform Year_of_Release        Genre
    ##                        <chr>    <chr>           <int>        <chr>
    ## 1                 Wii Sports      Wii            2006       Sports
    ## 2          Super Mario Bros.      NES            1985     Platform
    ## 3             Mario Kart Wii      Wii            2008       Racing
    ## 4          Wii Sports Resort      Wii            2009       Sports
    ## 5   Pokemon Red/Pokemon Blue       GB            1996 Role-Playing
    ## 6                     Tetris       GB            1989       Puzzle
    ## 7      New Super Mario Bros.       DS            2006     Platform
    ## 8                   Wii Play      Wii            2006         Misc
    ## 9  New Super Mario Bros. Wii      Wii            2009     Platform
    ## 10                 Duck Hunt      NES            1984      Shooter
    ## # ... with 16,709 more rows, and 12 more variables: Publisher <chr>,
    ## #   NA_Sales <dbl>, EU_Sales <dbl>, JP_Sales <dbl>, Other_Sales <dbl>,
    ## #   Global_Sales <dbl>, Critic_Score <int>, Critic_Count <int>,
    ## #   User_Score <dbl>, User_Count <int>, Developer <chr>, Rating <chr>

Lr resultat est la même table avec l'information sur la séparation en groupes. Cette information est accéssible par la fonction `groups()`:

``` r
vg_sales %>%
  group_by(Genre) %>%
  groups
```

    ## [[1]]
    ## Genre

Sur cette base on utilise la fonction `summarise()` pour calculer les statistique sur chaque groupe. D'abords explorons le résultat de l'application de `summarise()` sur le table d'origine:

``` r
vg_sales %>%
  summarise(avg_sales = mean(Global_Sales))
```

    ## # A tibble: 1 <U+00D7> 1
    ##   avg_sales
    ##       <dbl>
    ## 1 0.5335427

On ajouter l'étape `group_by()` on realise la moyenne des ventes par genre de jeux video:

``` r
vg_sales %>%
  group_by(Genre) %>%
  summarise(avg_sales = mean(Global_Sales))
```

    ## # A tibble: 13 <U+00D7> 2
    ##           Genre avg_sales
    ##           <chr>     <dbl>
    ## 1        Action 0.5178843
    ## 2     Adventure 0.1824175
    ## 3      Fighting 0.5270671
    ## 4          Misc 0.4589600
    ## 5      Platform 0.9325225
    ## 6        Puzzle 0.4190000
    ## 7        Racing 0.5835869
    ## 8  Role-Playing 0.6229333
    ## 9       Shooter 0.7958730
    ## 10   Simulation 0.4467048
    ## 11       Sports 0.5672913
    ## 12     Strategy 0.2554905
    ## 13         <NA> 1.2100000

Les jeux de plateforme semblent avoir la moyenne des ventes la plus élevée (en pensant à Mario Bros, ça paraît plausible). Mais il serait interessant d'avoir le nombre de jeux dans chaque categorie et un peu plus d'informations sur les ventes comme la mediane, le min et le max des ventes. pour celà nous allons utiliser la fonction `n()` pour compter les observations par groupe, et les fonctions `median()`, `min()` `max()` pour les statitiques additionnelles:

``` r
vg_sales %>%
  group_by(Genre) %>%
  summarise(avg_sales = mean(Global_Sales),
            med_sales = median(Global_Sales),
            min_sales = min(Global_Sales),
            max_sales = max(Global_Sales),
            nb_games = n())
```

    ## # A tibble: 13 <U+00D7> 6
    ##           Genre avg_sales med_sales min_sales max_sales nb_games
    ##           <chr>     <dbl>     <dbl>     <dbl>     <dbl>    <int>
    ## 1        Action 0.5178843      0.19      0.01     21.04     3370
    ## 2     Adventure 0.1824175      0.05      0.01     11.18     1303
    ## 3      Fighting 0.5270671      0.20      0.01     12.84      849
    ## 4          Misc 0.4589600      0.16      0.01     28.92     1750
    ## 5      Platform 0.9325225      0.27      0.01     40.24      888
    ## 6        Puzzle 0.4190000      0.11      0.01     30.26      580
    ## 7        Racing 0.5835869      0.19      0.01     35.52     1249
    ## 8  Role-Playing 0.6229333      0.18      0.01     31.37     1500
    ## 9       Shooter 0.7958730      0.23      0.01     28.31     1323
    ## 10   Simulation 0.4467048      0.15      0.01     24.67      874
    ## 11       Sports 0.5672913      0.22      0.01     82.53     2348
    ## 12     Strategy 0.2554905      0.09      0.01      5.45      683
    ## 13         <NA> 1.2100000      1.21      0.03      2.39        2

Si on veut former les groupes en utilisant plusieurs variables, il suffit de les spécifier dans l'appel de la fonction `group_by()`. Par example: `vg_sales %>% group_by(Genre, Platform) %>% summarize(nb_games = n())` pour réaliser des groupes à la fois sur `Genre` et `Platform. De façon similaire à`mutate\_each()`, nous avons la fonction`summarise\_each()\` pour réaliser le résumé statistique de plusieurs colonnes avec la même fonction.
