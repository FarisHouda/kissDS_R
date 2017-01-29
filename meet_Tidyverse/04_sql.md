SQL Intro
================

SQL Intro
---------

SQL est une référence en terme de traitemant de données tabulaires. C'est le language utiliser pour faire des requètes dans les bases de données relationnelles comme MySQL, Postgres ou SQLLite. Un certain nombre de base de données des système Big Data prpose une syntaxe basée sur SQL comme Hive, Implala ou Spark. Il est possible d'utiliser la syntaxe SQL de façon directe dans R à travers le package `sqldf`.

Il important de faire le lien entre les base du SQL et les fonctionnalités proposées par dplyr pour plusieurs raison:

-   Cette base commune permet de rendre les scripts dplyr assez naturels pour des utilisateurs connaissant SQL.
-   La transition vers un système de production SQL est plus claire.
-   Les fonctionnalités avancées de `dplyr` permettent de se connecter directement à des bases de données SQL d'où la necessité d'en connaître les bases.

Cette section se propose d'établire les liens entre la syntaxe dplyr et SQL et d'introduire la syntaxe dplyr pour les opérations de jointure et des ensembles. Elle n'est cependant pas une initiation à SQL ou aux systèmes de bases de données relationnelles.

**Ajouter des liens pour approfondir le sujet**

La structure d'une requette SQL:
--------------------------------

SQL est un langage pour passer des requettes à une base de données tabulaires. Il support à la fois les opérations sur une table, comme celles combinant plusieurs tables. Voir **Ajouter liens wikipedia**.

Nous allons faire le lien entre les requettes SQL sur une table et les fonctions que nous avons vus jusqu'à présent dans dplyr. Les opérations sur plusieurs tables sont le sujet des sections suivantes.

Une requette SQL simple est typiquement de la forme: `SELECT colonne_1, colonne_2 FROM table WHERE conditions_a_verifier`. Les verbes dplyr permettent la même expressivité si l'on considère la correspondance suivante:

-   La fonction `select()` de dplyr correspond à la clause `SELECT` de SQL.
-   La fonction `filter()` de dplyr correspond à la clause `WHERE` de SQL.
-   Le choix de la table d'entrée du pipe d'analyse est l'équivalent de la clause `FROM` de SQL.
-   La fonction `mutate()` de dplyr correspond aux opérations de création de nouvelles colonnes possibles dans la clause `SELECT` de SQL.

Les opérations de résumé statistiques par la combinaison `data %>% group_by(var1) %>% summarise(col1 = n(), col2 = min(var2))` ont aussi un équivalent SQL assez direct par les requettes du type `SELECT col1 = count(), clo2 = MIN(var2) FROM data GROUP BY var1`

Les jointures
-------------

Une jointure est une opération qui permet de combiner deux tables sur la base d'une ou plusieurs variables communes. Nous allons présenter les différents types de jointures avec un example simple:

Une petite entreprise "babikea" vend des tables et des chaises dont les données sont répertoriées dans la table suivante:

``` r
products <- read_csv("./data/babikea_products.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   product = col_character(),
    ##   type = col_character(),
    ##   price = col_integer()
    ## )

``` r
products
```

    ## # A tibble: 5 <U+00D7> 3
    ##    product  type price
    ##      <chr> <chr> <int>
    ## 1 product1 table    23
    ## 2 product2 chair    13
    ## 3 product3 chair    11
    ## 4 product4 table    20
    ## 5 product5 stool     5

Les ventes du dernier trimestre de 2016 sont enregistrées dans une table qui lie les produits au nom du client, à la date d'achat et à la quantité achetée:

``` r
sales <- read_csv("./data/babikea_product_sales.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   custormer = col_character(),
    ##   product = col_character(),
    ##   date = col_date(format = ""),
    ##   quantity = col_integer()
    ## )

``` r
sales
```

    ## # A tibble: 8 <U+00D7> 4
    ##   custormer         product       date quantity
    ##       <chr>           <chr>     <date>    <int>
    ## 1    Moogle        product1 2016-09-12       20
    ## 2    Moogle        product2 2016-09-12       80
    ## 3       BPP        product1 2016-12-03       15
    ## 4       BPP        product2 2016-12-03       30
    ## 5    Azamon        product3 2016-11-24       16
    ## 6 Macrosoft        product3 2016-12-23       14
    ## 7  Pacebook        product4 2016-12-23        4
    ## 8  Pacebook sofa_sur_mesure 2016-12-23        2

Si nous voulons faire le bilan du résultat des ventes de chaises, nous avons besoin du nombre de chaises vendues et du prix auquel elle ont été vendues. Mais l'information est séparée entre les deux tables. Une jointure nous permet de combiner l'information entre les deux tables. On dispose de plusieurs types de jointures selon l'objectif de l'analyse. Vous pouvez en apprendre plus sur le sujet sur la [page wikipédia](https://fr.wikipedia.org/wiki/Jointure_(informatique)) dédiée à ce sujet:

-   On doit définir les colonnes à utiliser pour réaliser la jointure (clé de la jointure)
-   On combine chaque pair de lignes des deux tables si elle ont les même valeurs des colonnes de jointure
-   Le type de jointure permet de spécifier le comportement au cas où une valeur de jointure est abscente d'une des deux tables.

Les principaux types de jointures sont décrits dans [la page du cours `SQL.sh`](http://sql.sh/cours/jointures).

Nous allons apprendre les jointures de base avec notre examples `babikea` de façon pratique, et en utilisant dplyr pour nous exprimer.

### Inner Join:

Le directeur financier de `babikea` veut connaitre le total des ventes du catalogue sur le dernier trimestre 2016. Pour faire ce calcule, on à besoin de combiner les prix de ventes dans la table `products` avec les quantités vendues par produit de la table `sales`. La colonne qui permet de faire le lien entre les deux tables est la colonne `product`. Nous devons donc ajouter à chaque ligne des ventes, les données de prix de la table produit. Nous avons besoin des informations de prix et de quantité pour faire le calcul. Donc un produit qui n'a pas été vendu, ou la vente d'un produit qui n'est pas dans la catalogue ne sont pas utiles. Nous avons besoin d'un `inner join` qui ne garde que les produit qui sont à la foix dans le catalogue des produits et ont été vendu.

``` r
 products %>%
  inner_join(sales, by="product")
```

    ## # A tibble: 7 <U+00D7> 6
    ##    product  type price custormer       date quantity
    ##      <chr> <chr> <int>     <chr>     <date>    <int>
    ## 1 product1 table    23    Moogle 2016-09-12       20
    ## 2 product1 table    23       BPP 2016-12-03       15
    ## 3 product2 chair    13    Moogle 2016-09-12       80
    ## 4 product2 chair    13       BPP 2016-12-03       30
    ## 5 product3 chair    11    Azamon 2016-11-24       16
    ## 6 product3 chair    11 Macrosoft 2016-12-23       14
    ## 7 product4 table    20  Pacebook 2016-12-23        4

Pour avoir les revenues de chaque opération de vente, on doit multiplier le prix par la quantité, puis sommer sur toute les lignes:

``` r
 products %>%
  inner_join(sales, by="product") %>%
  mutate(revenues = price * quantity) %>%
  summarise(total_revenues = sum(revenues))
```

    ## # A tibble: 1 <U+00D7> 1
    ##   total_revenues
    ##            <int>
    ## 1           2645

Avec un `group_by` on peut avoir les ventes par type de produit:

``` r
 products %>%
  inner_join(sales, by="product") %>%
  mutate(revenues = price * quantity) %>%
  group_by(type) %>%
  summarise(total_revenues = sum(revenues))
```

    ## # A tibble: 2 <U+00D7> 2
    ##    type total_revenues
    ##   <chr>          <int>
    ## 1 chair           1760
    ## 2 table            885

On peut faire la même chose par produit du catalogue:

``` r
catalog_sales <- products %>%
  inner_join(sales, by="product") %>%
  mutate(revenues = price * quantity) %>%
  group_by(product) %>%
  summarise(total_revenues = sum(revenues))

catalog_sales
```

    ## # A tibble: 4 <U+00D7> 2
    ##    product total_revenues
    ##      <chr>          <int>
    ## 1 product1            805
    ## 2 product2           1430
    ## 3 product3            330
    ## 4 product4             80

Cependant, on n'a pas de ligne correspondant au produit `product5` qui ne s'est pas vendu sur la période.

### Left & Right Join:

On utilise le left join pour garder taute les valeurs de la clé de jointure qui sont présente dans la table de gauche (`products` dans notre cas). Si la table de droite (`sales` dans notre example) contient des produits hors catalogue (comme sofa\_sur\_mesure), il ne seront pas retenu dans la jointure. Right join fait la même chose mais en gardant toutes les valeurs de la table de droite.

Pour avoir un tableau des résultats de ventes sur lequel figurent toutes les références du catalogue, on peut utiliser `left_join()` entre `products` et `catalog_sales`:

``` r
products %>%
  left_join(catalog_sales, by="product")
```

    ## # A tibble: 5 <U+00D7> 4
    ##    product  type price total_revenues
    ##      <chr> <chr> <int>          <int>
    ## 1 product1 table    23            805
    ## 2 product2 chair    13           1430
    ## 3 product3 chair    11            330
    ## 4 product4 table    20             80
    ## 5 product5 stool     5             NA

On peut ajouter une étape de traitement pour remplacer les valaurs `NA` avec un 0. On utilise la fonction `is.na()` permet d'identifier les `NA` puis utiliser `ifelse()` pour renplacer par un 0 si la valeur est `NA`:

``` r
products %>%
  left_join(catalog_sales, by="product") %>%
  mutate(total_revenues = ifelse(is.na(total_revenues), 0, total_revenues))
```

    ## # A tibble: 5 <U+00D7> 4
    ##    product  type price total_revenues
    ##      <chr> <chr> <int>          <dbl>
    ## 1 product1 table    23            805
    ## 2 product2 chair    13           1430
    ## 3 product3 chair    11            330
    ## 4 product4 table    20             80
    ## 5 product5 stool     5              0

### Outer Join:

Outer join permet de faire la jointure en gardant toutes les valeurs de clé présentes dans la table de droite et dans la table de gauche. Les valeurs manquantes sont remplacées par des `NA`. On peut voir le résultat de l'utilisation d'un outer join au lieu du inner join dans le premier example que nous avons fait. Dans dplyr la fonction `full_join()` permet de réaliser un outer join:

``` r
 products %>%
  full_join(sales, by="product") 
```

    ## # A tibble: 9 <U+00D7> 6
    ##           product  type price custormer       date quantity
    ##             <chr> <chr> <int>     <chr>     <date>    <int>
    ## 1        product1 table    23    Moogle 2016-09-12       20
    ## 2        product1 table    23       BPP 2016-12-03       15
    ## 3        product2 chair    13    Moogle 2016-09-12       80
    ## 4        product2 chair    13       BPP 2016-12-03       30
    ## 5        product3 chair    11    Azamon 2016-11-24       16
    ## 6        product3 chair    11 Macrosoft 2016-12-23       14
    ## 7        product4 table    20  Pacebook 2016-12-23        4
    ## 8        product5 stool     5      <NA>       <NA>       NA
    ## 9 sofa_sur_mesure  <NA>    NA  Pacebook 2016-12-23        2

Avec cette jointure on peut calculer les volumes de ventes de toutes les produits catalogue et sur mesure. On peut utiliser l'option `na.rm` dans la fonction sum pour ignorer les valeures `NA` dans la somme:

``` r
volumes <- products %>%
  full_join(sales, by="product") %>%
  group_by(product) %>%
  summarise(volumes = sum(quantity, na.rm = T))

volumes
```

    ## # A tibble: 6 <U+00D7> 2
    ##           product volumes
    ##             <chr>   <int>
    ## 1        product1      35
    ## 2        product2     110
    ## 3        product3      30
    ## 4        product4       4
    ## 5        product5       0
    ## 6 sofa_sur_mesure       2

### Semi\_join & Anti\_join:

Le `semi_join()` et `anti_join()` permet de filtrer une table sur la base des valeurs d'une deuxième table:

-   `semi_join()` filtre la première table pour ne garder que les valeurs de clé présentent dans la deuxième table.
-   `anti_join()` filtre la première table pour ne garder que les valeurs de clé qui ne sont pas présentent dans la deuxième table.

On peut utiliser semi\_join pour ne garder que les produits du catalogues:

``` r
catalog_volumes <- volumes %>% 
  semi_join(products, by="product")

catalog_volumes
```

    ## # A tibble: 5 <U+00D7> 2
    ##    product volumes
    ##      <chr>   <int>
    ## 1 product1      35
    ## 2 product2     110
    ## 3 product3      30
    ## 4 product4       4
    ## 5 product5       0

ou que ceux les volumes produits vendus:

``` r
sold_volumes <- products %>%
  full_join(sales, by="product") %>%
  group_by(product) %>%
  summarise(volumes = sum(quantity, na.rm = T)) %>%
  semi_join(sales, by = "product")
  
sold_volumes
```

    ## # A tibble: 5 <U+00D7> 2
    ##           product volumes
    ##             <chr>   <int>
    ## 1        product1      35
    ## 2        product2     110
    ## 3        product3      30
    ## 4        product4       4
    ## 5 sofa_sur_mesure       2

On peut aussi faire la même chose pour les produits hors-catalogue avec la fonction `anti_join()`:

``` r
horscatalog_volumes <- volumes %>% 
  anti_join(products, by="product")

horscatalog_volumes
```

    ## # A tibble: 1 <U+00D7> 2
    ##           product volumes
    ##             <chr>   <int>
    ## 1 sofa_sur_mesure       2

Les opérations d'ensemble
-------------------------

Une autre façon de réaliser des opérations de filtre sur une table en utilisant une autre table sont les opérations d'ensemble. Ces opérations sont utile pour des tables qui contiennent le même type de contenu en terme de colonnes mais des observations plus ou moins différentes:

-   `intersect()` permet de garder que les observations en commun entre les deux tables.
-   `union()` permet de combiner les observations des deux tables mais sans dupliquer les observations communes.
-   `setdiff()` permet de ne garder que les observations présente dans la première table seulement.

On peut construire une tables avec que les produits du catalogue qui ont été vendu sur le dernier trimestre 2016. On utilise pour cela l'intersection de la table des volume des produits vendus `sold_volumes` avec la tables des volumes de vente du catalogues `catalog_volumes`:

``` r
sold_volumes %>%
  intersect(catalog_volumes)
```

    ## # A tibble: 4 <U+00D7> 2
    ##    product volumes
    ##      <chr>   <int>
    ## 1 product1      35
    ## 2 product2     110
    ## 3 product3      30
    ## 4 product4       4

De le même façon on peut reconstruire la table exhaustive des volumes en utilisant l'union des ces deux même tables:

``` r
sold_volumes %>%
  union(catalog_volumes)
```

    ## # A tibble: 6 <U+00D7> 2
    ##           product volumes
    ##             <chr>   <int>
    ## 1        product5       0
    ## 2 sofa_sur_mesure       2
    ## 3        product4       4
    ## 4        product3      30
    ## 5        product2     110
    ## 6        product1      35

La fonction `set_diff()` nous permet d'enlever les références de la première tables qui sont déjà présentent dans la deuxième table. En d'autre termes, on enlève de la table `sold_volumes` les produits du catalogue (donc présents dans la table `catalog_volumes`):

``` r
sold_volumes %>%
  setdiff(catalog_volumes)
```

    ## # A tibble: 1 <U+00D7> 2
    ##           product volumes
    ##             <chr>   <int>
    ## 1 sofa_sur_mesure       2

Ce qui nous donne la tables de volumes des produits vendus hors catalogue.
