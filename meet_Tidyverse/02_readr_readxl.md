readr & readxl
================

Intro:
------

Le pipe d'analyse de données commence souvent par une étape de lecture des données à partire d'un ou plusieurs (voir une collection nombreuse) de fichiers. Ces fichiers peuent être de plusieurs types dépendant des dommaines d'application et des sources de données utilisées. On peut en citer les fichiers texte du type `.csv`, `.txt` ou des fichiers Excel `.xls` ou `.xlsx`.

Readr:
------

Le package `readr` permet les fichiers texte de plusieurs type. La différence entre ces types est le charactère utiliser les différentes valeurs sur une même ligne:

-   `.csv` utilise `,` ou `;` comme séparateur
-   `.tsv` utilise des tabulations commes séparateur
-   Pour les fichiers `.txt` il faut vérifier au prélable le charactère de séparation. (Certaines fonctions de lecture comme `fread` du package `data.table` détecte automatiquement le charactère séparateur)

`reader` propose une fonction de lecture pour chaque type de fichier:

-   `read_csv()` pour lire les fichiers `.csv` avec un séparateur `,` (csv: comma separated values - "valeurs séparées par une virgule" en français).
-   `read_csv2()` pour lire les fichiers `.csv` avec un séparateur `;` (ce type de csv est notement utilisé par Microsoft Excel par défault).
-   `read_tsv()` pour lire les fichiers texte avec des tabulations comme séparateurs (tsv: tabulation separated values - "valeurs séparées par des tabulations" en français).
-   `read_delim()`pour lire les fichiers texte avec un séparateurs specifique (comme un `|`). Le charactère de séparation doit être donné comme deuxième argument de la `read_delim()`.
-   `read_log()` pour lire des fichier log Apache (des fichiers de serveurs internet). Voir aussi le package `webreadr` pour plus de fonctionalités pour ce type de fichiers.
-   `read_fwf()` pour lire des fichiers de largeur constante (fwf: fixed width files - "fichier à largeur fixe" en français). Il est nécessaire de donner le nombre de charactères dédiés à chanque variable (en spécifiant l'option `fwf_widths()`) ou la position du premier character de chanque variable (en spécifiant l'option `fwf_positions()`)

Nous allons dans la suite nous focaliser sur les fichiers `.csv` sachant que les fonctionalités des autres fonctions sont similaires.

La fonction `read_csv()` prend comme argument le chemin vers le fichier csv. Il est recommender d'uiliser un chemin relatif qui commence par un `.` comme `./le/chemin`. Le `.` repesente le chemin vers le dossier d'éxecusion de R qu'on peut trouver en utilisant le fonction `getwd()` On composer le chemin en ajoutant après le `.` le chemin vers le fichier `.csv` en partant du dossier d'execution.

Dans l'example si dessous le fichier `utilisateur.csv` est dans le dossier `data` qui est contenu dans le dossier principale du projet. Donc le chemin vers ce fichier est `./data/utilisateur.csv`

``` r
read_csv("./data/utilisateur.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Nom = col_character(),
    ##   Prenom = col_character(),
    ##   Age = col_integer(),
    ##   Enregistre_le = col_character()
    ## )

    ## # A tibble: 4 <U+00D7> 4
    ##      Nom  Prenom   Age Enregistre_le
    ##    <chr>   <chr> <int>         <chr>
    ## 1  Marie  Dupond    38    18-03-2017
    ## 2  Amira Kharoub    29    05-02-2017
    ## 3 Claude Jourdan    24    02-04-2017
    ## 4   Joel   Samba    32    30-01-2017

`read_csv()` identifie automatiquement le type de chaque colonne en itilisant les 1000 premières lignes. Le résultat de cette inférence est donnée en sortie de la fonction dans la partie `cols(...)`. On voit dans ce cas que l'inférence du nom, prénom est de type charactère (`col_character()`). Celle de l'age est de type entier (`col_integer()`). L'inférence de la date d'enregistrement sera soit une date (si le format correspond au format par défaut du système) soit un charactère sinon. (la gestion des dates est un sujet délicat, on explorera ce sujet plus en détail avec le package \`lubridate)

On peut fournir le type de chaque colonne dans la commande de lecture à travers l'option `col_types()`. On en profite pour forcer le 4eme champ à être une date. Cette option est très importante pour s'assurer de la reproductibilité des sorties en cas de changement de fichiers.

``` r
read_csv("./data/utilisateur.csv", 
         col_types = list(
           col_character(),
           col_character(),
           col_integer(),
           col_date(format = "%d-%m-%Y")
         ))
```

    ## # A tibble: 4 <U+00D7> 4
    ##      Nom  Prenom   Age Enregistre_le
    ##    <chr>   <chr> <int>        <date>
    ## 1  Marie  Dupond    38    2017-03-18
    ## 2  Amira Kharoub    29    2017-02-05
    ## 3 Claude Jourdan    24    2017-04-02
    ## 4   Joel   Samba    32    2017-01-30

Le format de la date est construit sur la base des blocks suivant: `%d` jour, `%m` mois en chiffres, `%Y` les années en 4 chiffres

Pour poursuivre nos testes on va donner à la fonction `read_csv()` une chaîne de charactères aui represente notre fichier d'entrée:

``` r
read_csv(
  "Nom, Prenom, Age, Enregistre_le
Marie, Dupond, 38, 18-03-2017
Amira, Kharoub, 29, 05-02-2017"
)
```

    ## # A tibble: 2 <U+00D7> 4
    ##     Nom  Prenom   Age Enregistre_le
    ##   <chr>   <chr> <int>         <chr>
    ## 1 Marie  Dupond    38    18-03-2017
    ## 2 Amira Kharoub    29    05-02-2017

Si les données d'entrée contiennent des lignes de additionnelles avant le tableau, on utilise l'option skip pour specifier le nombre de lignes à sauter avant de commencer la lecture:

``` r
read_csv(
  "Ce fichier est super important
le tableau est juste après

Nom, Prenom, Age, Enregistre_le
Marie, Dupond, 38, 18-03-2017
Amira, Kharoub, 29, 05-02-2017",
skip = 3
)
```

    ## # A tibble: 2 <U+00D7> 4
    ##     Nom  Prenom   Age Enregistre_le
    ##   <chr>   <chr> <int>         <chr>
    ## 1 Marie  Dupond    38    18-03-2017
    ## 2 Amira Kharoub    29    05-02-2017

Le fichier peut contenir des lignes de commentaires précédées par un character distinctif (\# par example). En specifiant l'option `comment` la fonction peut prendre en compte cette information et évite de lire ces lignes.

``` r
read_csv(
  "# Ceci est un commentaire!!
Nom, Prenom, Age, Enregistre_le
Marie, Dupond, 38, 18-03-2017
# un autre commentaire à propos de cet utilisateur
Amira, Kharoub, 29, 05-02-2017",
comment="#"
)
```

    ## # A tibble: 2 <U+00D7> 4
    ##     Nom  Prenom   Age Enregistre_le
    ##   <chr>   <chr> <int>         <chr>
    ## 1 Marie  Dupond    38    18-03-2017
    ## 2 Amira Kharoub    29    05-02-2017

`read_csv()` suppose que la première ligne lue contient les labels des colonnes. So cette information n'est pas dans le fichier, on peut dire à la fonction de lire directement les données en ajustant l'option `col_names`:

``` r
read_csv(
  "Marie, Dupond, 38, 18-03-2017
Amira, Kharoub, 29, 05-02-2017",
col_names = FALSE
)
```

    ## # A tibble: 2 <U+00D7> 4
    ##      X1      X2    X3         X4
    ##   <chr>   <chr> <int>      <chr>
    ## 1 Marie  Dupond    38 18-03-2017
    ## 2 Amira Kharoub    29 05-02-2017

Cette option permet aussi de spécifier des noms à utiliser pour les colonnes, et qui marche en présence et en abscence de noms de colonnes dans le fichier:

cas sans labels:

``` r
read_csv(
  "Marie, Dupond, 38, 18-03-2017, 
Amira, Kharoub, 29, 05-02-2017",
col_names = c("name", "surname", "age", "subscription_date")
)
```

    ## Warning: 1 parsing failure.
    ## row col  expected    actual
    ##   1  -- 4 columns 5 columns

    ## # A tibble: 2 <U+00D7> 4
    ##    name surname   age subscription_date
    ##   <chr>   <chr> <int>             <chr>
    ## 1 Marie  Dupond    38        18-03-2017
    ## 2 Amira Kharoub    29        05-02-2017

cas avec labels:

``` r
read_csv(
  "Nom, Prenom, Age, Enregistre_le
Marie, Dupond, 38, 18-03-2017
Amira, Kharoub, 29, 05-02-2017",
col_names = c("name", "surname", "age", "subscription_date")
)
```

    ## # A tibble: 3 <U+00D7> 4
    ##    name surname   age subscription_date
    ##   <chr>   <chr> <chr>             <chr>
    ## 1   Nom  Prenom   Age     Enregistre_le
    ## 2 Marie  Dupond    38        18-03-2017
    ## 3 Amira Kharoub    29        05-02-2017

Le package `readr` contient aussi des fonctions d'écriture des données de R vers des fichiers plats. Pour écrire des csv on utilise la fonction `write_csv(x, path)` qui prend comme paramètres obligatoires le nom du tableau à écrire (x) et le chemin vers le fichier où écrire la donnée (path). On liste les fonctions d'écriture ci-dessous:

-   `write_csv()` pour écrire un tableau dans un fichier csv
-   `write_excel_csv()` pour écrire un tableau dans un fichier csv selon le formatage excel (séparateur `;`).
-   `write_tsv()` pour écrire un tableau dans un fichier tsv (tabulations comme séparateur)
-   `write_delim()` pour écrire un tableau dans un fichier texte avec un séparateur spécifié avec l'option `delim`. Le séparateur par défault est un espace.

readxl
------

Le package `readxl` contient la fonction `read_excel` pour lire les données de fichiers de type `.xls` ou `.xlsx`. Cette fonction à l'avantage de donner des fichiers dans le format des tableaux du tidyverse directement (que nous allons explorer dans les sections suivante). Pour des indications sur d'autre packages avec des fonctionalités similaires, voir la fin de cette section.

Les options de la fonction sont similaires à la fonction `read_csv()`:

-   `sheet` permet de specifier la feuille de calcule à lire dans le fichier excel, soit par numéro ou par nom. La valaur par default est la première feuille.
-   `skip` specifie le nombre de ligne à sauter avant de commencer la lecture du fichier
-   `col_names` is `TRUE` permet de lire les labels des colonnes de la première ligne du fichier. Specifier comme `FALSE` pour specifier des noms pas défault (X1, X2, ... etc )
-   `col_type` si speciié comme `NULL`la fonction infère les types de colonnes. Sinon, il faut lui donners un vecteur des types au nombre des colonnes du fichier excel composé de `"blanck", "numeric", "date" ou "text"`

Avant d'utiliser la fonction il est nécessaire de charger le package `readxl` car il n'est pas chargé automatiquement avec le tidyverse.

Example d'utilisation:

``` r
library(readxl)
read_excel("./data/utilisateur.xlsx")
```

    ## # A tibble: 4 <U+00D7> 4
    ##      Nom  Prenom   Age Enregistre_le
    ##    <chr>   <chr> <dbl>         <chr>
    ## 1  Marie  Dupond    38    18-03-2017
    ## 2  Amira Kharoub    29    05-02-2017
    ## 3 Claude Jourdan    24    02-04-2017
    ## 4   Joel   Samba    32    30-01-2017

Pour plus de fonctionalité avec des fichiers excel (pour l'écriture par example) voir les packages [`openxlsx`](https://github.com/awalker89/openxlsx), [`xlsx`](https://cran.r-project.org/web/packages/xlsx/index.html) (utilise java), [XLConnect](https://cran.r-project.org/web/packages/XLConnect/) (utilise java).

Rio
---

Sur ce même thème de l'acquisition de données vous pouvez explorez [le package `rio`](https://github.com/leeper/rio) qui propose une fonction `ìmport` pour lire toute une collection de types de fichiers différents ainsi qu'une fonction `export` pour l'écriture.

Tidyr:
------

Une fois les données sont lues, elles doivent être transformées selon les critères du tidy data.

-   Une variable par colonne
-   Une observation par ligne
-   CHaque valeur doit avoir sa propre case

La declinaison de cette definition sur des données réelles dépend à la fois des données et des questions auquelles on veut répondre. Pour introduire le concept de façon pratique et developper une intuition à ce sujet, on va traiter des cas de données non tidy et les transformations necessaires pour les rendre tidy.

Le package `tidyr` introduit des fonctions de transformations pour passer des differents types de données non-tidy. On va utiliser ces fonctions pour traiter 5 examples representatifs de la manière dont les données peuvent être non-tidy. Plus d'informations à ce sujet sont disponibles en anglais dans la documentation du package (par example en tappant `vignette("tidy-data")` dans une console R.)

"Toutes les familles heureuses se ressemblent. Chaque famille malheureuse, au contraire, l'est à sa façon." L. Tolstoï

"Touts les jeux de données tidy de ressemblent. Chaque jeu de données non-tidy, au contraire, l'est à sa façon." H. Wickham

Des manières dont les données peuvent être non-tidy on peut identifier:

-   De l'information dans les noms de colonnes
-   Plusieurs variables dans une même colonne
-   Des variables enregistrée à la fois dans les lignes et les colonnes
-   Plusieurs types de données dans la même table
-   Un type de données dans différentes tables

Dans la suite on va utiliser `tidyr` pour traiter des examples des deux premiers points. Pour les trois suivants, nous aurons besoin d'utiliser le package `dplyr` pour les opérations sur les tableaux.

### De l'information dans les noms de colonnes:

Un formatage assez courant de données tabulaire consiste à utiliser un les valeurs d'une variable comme nom des colonnes contenant les données. Par example:

``` r
sales_per_year <- read_csv("./data/sales_01.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Zone = col_character(),
    ##   Pays = col_character(),
    ##   Canal = col_character(),
    ##   `2014` = col_double(),
    ##   `2015` = col_double(),
    ##   `2016` = col_double()
    ## )

``` r
sales_per_year
```

    ## # A tibble: 8 <U+00D7> 6
    ##    Zone   Pays  Canal `2014` `2015` `2016`
    ##   <chr>  <chr>  <chr>  <dbl>  <dbl>  <dbl>
    ## 1   NAM    USA retail   2.38   2.45   2.78
    ## 2   NAM    USA online   0.98   1.34   1.89
    ## 3   NAM Canada retail   1.20   1.32   1.35
    ## 4   NAM Canada online   0.30   0.56   0.78
    ## 5    EU France retail   2.32   2.78   2.99
    ## 6    EU France online   0.40   0.78   1.20
    ## 7    EU     UK retail   1.78   2.02   2.23
    ## 8    EU     UK online   0.23   0.98   2.50

L'information de la variable utilisée comme noms de colonnes n'est donc pas dans sa propre colonne. En plus chaque ligne contient l'information de plusieurs observations. On utilse dans ce cas la fonction `gather` pour la transformer en format tidy.

La fonction `gather` prend comme paramètres:

-   Le tableau de données
-   `key`: le nom à donner à la colonne qui va contenir l'information des noms des colonnes
-   `value`: le nom à donner à la colonne qui va contenir les valeurs des colonnes selectionnées
-   les colonnes à inclure (ou à exclure) dans la selection

On utilise `gather` pour créer une colonne specific pour les années, et une colonne pour les chiffres de ventes.

``` r
sales_tidy <- sales_per_year %>%
  gather(key = annee, value = ventes, `2014`, `2015`, `2016`, convert=TRUE)
sales_tidy
```

    ## # A tibble: 24 <U+00D7> 5
    ##     Zone   Pays  Canal annee ventes
    ##    <chr>  <chr>  <chr> <int>  <dbl>
    ## 1    NAM    USA retail  2014   2.38
    ## 2    NAM    USA online  2014   0.98
    ## 3    NAM Canada retail  2014   1.20
    ## 4    NAM Canada online  2014   0.30
    ## 5     EU France retail  2014   2.32
    ## 6     EU France online  2014   0.40
    ## 7     EU     UK retail  2014   1.78
    ## 8     EU     UK online  2014   0.23
    ## 9    NAM    USA retail  2015   2.45
    ## 10   NAM    USA online  2015   1.34
    ## # ... with 14 more rows

A partir du tableau tidy on peut utiliser l'operation inverse à `gather` à travers la fonction `spread`. Cette fonction prend comme arguments:

-   Tableau de données
-   key: la colonnes à utiliser pour former les noms de colonnes à ajouter
-   value: la colonnes contenant les valeurs pour former les colonnes.

On peut par example former un tableau avec les données de ventes partagées entre une colonnes `retail` et une colonne `online`:

``` r
sales_tidy %>%
  spread(Canal, ventes)
```

    ## # A tibble: 12 <U+00D7> 5
    ##     Zone   Pays annee online retail
    ## *  <chr>  <chr> <int>  <dbl>  <dbl>
    ## 1     EU France  2014   0.40   2.32
    ## 2     EU France  2015   0.78   2.78
    ## 3     EU France  2016   1.20   2.99
    ## 4     EU     UK  2014   0.23   1.78
    ## 5     EU     UK  2015   0.98   2.02
    ## 6     EU     UK  2016   2.50   2.23
    ## 7    NAM Canada  2014   0.30   1.20
    ## 8    NAM Canada  2015   0.56   1.32
    ## 9    NAM Canada  2016   0.78   1.35
    ## 10   NAM    USA  2014   0.98   2.38
    ## 11   NAM    USA  2015   1.34   2.45
    ## 12   NAM    USA  2016   1.89   2.78

ou par pays:

``` r
sales_tidy %>%
  spread(Pays, ventes)
```

    ## # A tibble: 12 <U+00D7> 7
    ##     Zone  Canal annee Canada France    UK   USA
    ## *  <chr>  <chr> <int>  <dbl>  <dbl> <dbl> <dbl>
    ## 1     EU online  2014     NA   0.40  0.23    NA
    ## 2     EU online  2015     NA   0.78  0.98    NA
    ## 3     EU online  2016     NA   1.20  2.50    NA
    ## 4     EU retail  2014     NA   2.32  1.78    NA
    ## 5     EU retail  2015     NA   2.78  2.02    NA
    ## 6     EU retail  2016     NA   2.99  2.23    NA
    ## 7    NAM online  2014   0.30     NA    NA  0.98
    ## 8    NAM online  2015   0.56     NA    NA  1.34
    ## 9    NAM online  2016   0.78     NA    NA  1.89
    ## 10   NAM retail  2014   1.20     NA    NA  2.38
    ## 11   NAM retail  2015   1.32     NA    NA  2.45
    ## 12   NAM retail  2016   1.35     NA    NA  2.78

### Plusieurs variables dans une même colonne:

Certaines colonnes peuvent contenir plusieurs informations combinées. L'information peut être combinée dans la colonnes de plusieurs façons:

-   Cas 1: Chaque case contient une information, mais la colonne contient des informations différentes. Une deuxième colonne contient l'information sur le type de données de la ligne.
-   Cas 2: Chaque case contient plusieurs informations combinées dans la même case.
-   Des combinaisons des deux cas!

Prenons des exemples:

#### Cas 1:

``` r
sales_volumes <- read_csv("./data/sales_volumes.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Pays = col_character(),
    ##   Annee = col_integer(),
    ##   Type_info = col_character(),
    ##   valeur = col_double()
    ## )

``` r
sales_volumes
```

    ## # A tibble: 6 <U+00D7> 4
    ##    Pays Annee Type_info valeur
    ##   <chr> <int>     <chr>  <dbl>
    ## 1   USA  2007    ventes   1.32
    ## 2   USA  2007   volumes 378.00
    ## 3   USA  2008    ventes   1.47
    ## 4   USA  2008   volumes 406.00
    ## 5    UK  2008    ventes   0.67
    ## 6    UK  2008   volumes 108.00

La colonne valeur contient deux types de données, les volumes de produits et les résultats de ventes.

On peut utiliser la fonction `spread` pour créer deux colonnes: Une pour les ventes et une pour les volumes.

``` r
sales_volumes %>%
  spread(Type_info, valeur)
```

    ## # A tibble: 3 <U+00D7> 4
    ##    Pays Annee ventes volumes
    ## * <chr> <int>  <dbl>   <dbl>
    ## 1    UK  2008   0.67     108
    ## 2   USA  2007   1.32     378
    ## 3   USA  2008   1.47     406

#### Cas 2:

``` r
sales_per_semestre <- read_csv("./data/sales_02.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Zone = col_character(),
    ##   Pays = col_character(),
    ##   Canal = col_character(),
    ##   S2.2015 = col_double(),
    ##   S1.2016 = col_double(),
    ##   S2.2016 = col_double()
    ## )

``` r
sales_per_semestre
```

    ## # A tibble: 8 <U+00D7> 6
    ##    Zone   Pays  Canal S2.2015 S1.2016 S2.2016
    ##   <chr>  <chr>  <chr>   <dbl>   <dbl>   <dbl>
    ## 1   NAM    USA retail    0.63    1.32    1.46
    ## 2   NAM    USA online    0.61    0.79    1.10
    ## 3   NAM Canada retail    0.45    0.60    0.75
    ## 4   NAM Canada online    0.45    0.44    0.34
    ## 5    EU France retail    0.99    1.29    1.70
    ## 6    EU France online    0.65    0.72    0.48
    ## 7    EU     UK retail    1.02    1.33    0.90
    ## 8    EU     UK online    0.99    1.82    0.68

on utilise la fonction gather pour combiner les données de ventes en une colonne:

``` r
sales_per_semestre_tidy <- sales_per_semestre %>%
  gather(key = "semestre", value = "ventes", S2.2015, S1.2016, S2.2016)
sales_per_semestre_tidy
```

    ## # A tibble: 24 <U+00D7> 5
    ##     Zone   Pays  Canal semestre ventes
    ##    <chr>  <chr>  <chr>    <chr>  <dbl>
    ## 1    NAM    USA retail  S2.2015   0.63
    ## 2    NAM    USA online  S2.2015   0.61
    ## 3    NAM Canada retail  S2.2015   0.45
    ## 4    NAM Canada online  S2.2015   0.45
    ## 5     EU France retail  S2.2015   0.99
    ## 6     EU France online  S2.2015   0.65
    ## 7     EU     UK retail  S2.2015   1.02
    ## 8     EU     UK online  S2.2015   0.99
    ## 9    NAM    USA retail  S1.2016   1.32
    ## 10   NAM    USA online  S1.2016   0.79
    ## # ... with 14 more rows

La colonne `semestre` contient l'information sur le semestre et sur l'année. On peut utiliser la fonction `separate()` pour former deux colonnes avec ces deux informations et l'ajouter au pipe danalyse au dessus:

``` r
sales_per_semestre_tidy <- sales_per_semestre %>%
  gather(key = "semestre", value = "ventes", S2.2015, S1.2016, S2.2016) %>%
  separate(semestre, c("semestre", "annee"))
sales_per_semestre_tidy
```

    ## # A tibble: 24 <U+00D7> 6
    ##     Zone   Pays  Canal semestre annee ventes
    ## *  <chr>  <chr>  <chr>    <chr> <chr>  <dbl>
    ## 1    NAM    USA retail       S2  2015   0.63
    ## 2    NAM    USA online       S2  2015   0.61
    ## 3    NAM Canada retail       S2  2015   0.45
    ## 4    NAM Canada online       S2  2015   0.45
    ## 5     EU France retail       S2  2015   0.99
    ## 6     EU France online       S2  2015   0.65
    ## 7     EU     UK retail       S2  2015   1.02
    ## 8     EU     UK online       S2  2015   0.99
    ## 9    NAM    USA retail       S1  2016   1.32
    ## 10   NAM    USA online       S1  2016   0.79
    ## # ... with 14 more rows

La fonction `separate()` essaye d'identifier un charactère non-alphanumérique (qui ne sont ni un chiffre ni une lettre) pour séparer la donnée d'une case. Dans l'example précédent la fonction identifie le `.` comme charactère de separation. On peut spécifier le charactère de séparation à travers l'option `sep`. L'exemple précédent devient:

``` r
sales_per_semestre_tidy_1 <- sales_per_semestre %>%
  gather(key = "semestre", value = "ventes", S2.2015, S1.2016, S2.2016) %>%
  separate(semestre, c("semestre", "annee"), sep="\\.")
sales_per_semestre_tidy_1
```

    ## # A tibble: 24 <U+00D7> 6
    ##     Zone   Pays  Canal semestre annee ventes
    ## *  <chr>  <chr>  <chr>    <chr> <chr>  <dbl>
    ## 1    NAM    USA retail       S2  2015   0.63
    ## 2    NAM    USA online       S2  2015   0.61
    ## 3    NAM Canada retail       S2  2015   0.45
    ## 4    NAM Canada online       S2  2015   0.45
    ## 5     EU France retail       S2  2015   0.99
    ## 6     EU France online       S2  2015   0.65
    ## 7     EU     UK retail       S2  2015   1.02
    ## 8     EU     UK online       S2  2015   0.99
    ## 9    NAM    USA retail       S1  2016   1.32
    ## 10   NAM    USA online       S1  2016   0.79
    ## # ... with 14 more rows

On besoin d'utiliser "\\" pour specifier le charactère point car l'option `sep` prend comme entrée une expression régulière. Nous étudierons se sujet dans la section dédiée au traitement de texte.

Il est possible d'effecture l'opération inverse avec la fonction `unite()`. Cette fonction prend comme entrée un tableau de données, le nom de la nouvelle colonne, les noms des colonnes à combiner, et un separateur specifié par l'option `sep`. On va ajouter une étape pour créer une colonne qui combine l'information canal, année, semestre, séparé par un "\_":

``` r
sales_per_semestre %>%
  gather(key = "semestre", value = "ventes", S2.2015, S1.2016, S2.2016) %>%
  separate(semestre, c("semestre", "annee")) %>%
  unite(tag, Canal, annee, semestre, sep="_")
```

    ## # A tibble: 24 <U+00D7> 4
    ##     Zone   Pays            tag ventes
    ## *  <chr>  <chr>          <chr>  <dbl>
    ## 1    NAM    USA retail_2015_S2   0.63
    ## 2    NAM    USA online_2015_S2   0.61
    ## 3    NAM Canada retail_2015_S2   0.45
    ## 4    NAM Canada online_2015_S2   0.45
    ## 5     EU France retail_2015_S2   0.99
    ## 6     EU France online_2015_S2   0.65
    ## 7     EU     UK retail_2015_S2   1.02
    ## 8     EU     UK online_2015_S2   0.99
    ## 9    NAM    USA retail_2016_S1   1.32
    ## 10   NAM    USA online_2016_S1   0.79
    ## # ... with 14 more rows

Pour les cas plus complexes il faut combiner ces différentes fonctions pour créer un pipe d'analyse adapté. Nous allons revenir aux tronsformations tidy après avoir fait connaissance avec `dplyr` juste après!
