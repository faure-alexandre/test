# Readme

- [Readme](#readme)
  - [Description](#description)
  - [Fonctionnement du script par défaut:](#fonctionnement-du-script-par-défaut)
  - [Utilisation du script en ligne de commandes](#utilisation-du-script-en-ligne-de-commandes)
    - [Exemple d'execution du script:](#exemple-dexecution-du-script)
  - [Description des paramètres du fichier json](#description-des-paramètres-du-fichier-json)
  - [Structure du code](#structure-du-code)
    - [Classe Files:](#classe-files)
    - [Classe ShipResampler:](#classe-shipresampler)
    - [resampler.py:](#resamplerpy)
  

## Description
Ce projet a pour but de pré-traiter les données de capteurs afin de les rendres accessible à Numerical twin.

## Fonctionnement du script par défaut:
1. Supprime les colonnes de chaque feuille excel contenant trop de NaNs.
2. Sélectionne certaines variables.
3. Calcul de la variable *'mean_draft'*.
4. Renomme les variables de chaque feuille (en ajoutant le préfixe de le feuille devant le nom de chaque variable).
5. Concatène toutes les feuilles dans un seul dataframe en prenant en compte les variables réparties dans plusieurs feuilles.
6. Convertit les coordonnées spatiales en ‘float’ (degré decimal).
7. Convertit les données angulaires sans discontinuités.
8. Réalise un ré-échantillonnage, une agrégation et une interpolation (avec possibilitée de personnalisation pour chaque variable).
9. Supprime les lignes contenants des NaNs.
10. Comparaison de la distance entres coordonnées spatiales de `“BonVoyage”` et `“GPS”` et suppression des lignes dont la distance est supérieure à un certain seuil.
11. Reconversion des données angulaires dans le format souhaité par l’utilisateur.
12. Reconversion des coordonnées spatiales dans le format souhaité par l’utilisateur.
13. Creation d'un fichier log contenant diverses informations concernant l'execution du script.
    

## Utilisation du script en ligne de commandes
Liste des arguments disponibles en ligne de commande:
* `--version` : permet d'afficher la version du script.
* `--verbose` ou `-v` : affiche les détails de l'execution du script dans la console.
* `--help` ou `-h` : affiche l'aide.

```cmd
Parameters_File
```
Chemin vers le fichier json contenant les paramètres.

### Exemple d'execution du script:
```cmd
python resampler.py -v params01.json
```

## Description des paramètres du fichier json

```python
captors_files: {"C:/Users/user/Documents/Digital Lab/Projet 1/data/Onboard sensors/2021-01-12 16_19_30_CMA-CGM-Magellan.xlsx":
["BonVoyage", "Bridge Sensor", "Motion Sensor", "NMEA GPS","NAVIGATION_GPS"]}
```
Dictionnaire contenant en clé le chemin vers le fichier contenant les données de type capteurs et en valeur le nom des feuilles à charger lors de l'import. 

<u>Remarque concernant le mutli-imports:</u> <br>
Il est possible de charger plusieurs fichiers, cependant pour qu'une variable contenue dans 2 fichiers différents soit reconnue comme identique, il faut que celle-ci porte le même nom et soit dans une feuille de même nom. Toute variable portant un nom différent et/ou dans une feuille de nom différent sera traitée comme une variable nouvelle. 

-----------
```python
vrs_files : ["C:/Users/user/Documents/Digital Lab/Projet 1/data/Onboard sensors/VRS MAGELLAN 2019.xlsx"]
```
Liste vers les fichiers contenant les données de type VRS.
On notera que les fichiers de type 'VRS' chargent par défaut seulement la première feuille.

-----------
```python    
output_file : "C:/Users/user/Documents/GitHub/projet1-gilles-alexandre/data/test.xlsx"
```
Chemin vers le fichier contenant les données traitées.

2 formats sont possibles:
* Soit le nom du fichier est explicite, par exemple :<br>
  `"C:/Users/user/Documents/GitHub/projet1-gilles-alexandre/data/test.xlsx"`.
* Soit on veut que le script nomme par défaut le fichier de sortie en fonction des inputs, il faut nommer le fichier de la manière suivante:<br>
    `"C:/Users/user/Documents/GitHub/Projet Syroco/data/xlsx"`.

A noter que les 2 formats pris en charge sont `.xlsx` et `.csv`.

Si un fichier du même nom est déjà présent x fois dans le répertoire, on ajoute le suffixe '(x)' au nom du fichier de sortie.

-----------
```python
name : "CMA CGM MAGELLAN"
```
Nom du bateau (correspondant à la variable _'Vessel Name'_ de _VRS_)

-----------
```python
resampling_windows : "15m"
```
Fenêtre de ré-échantillonage.

Exemples: 
* `"1s"` pour 1 seconde.
* `"1m"` pour 1 minute.
* `"1h"` pour 1 heure.
* `"1d"` pour 1 jour.
* `"1w"` pour 1 semaine.
* `"3h 1m 2s"` pour 03:01:02.

-----------
```python
interpolation_method: "linear"
```
Méthode par défaut pour interpoler les données numériques ie approximation dans le cas ou une
fenêtre de temps de contient aucune valeur pour une variable donnée.

Les différentes méthodes sont disponibles dans la [documentation pandas de la fonction interpolate](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.interpolate.html) (paramètre _'method'_).

-----------
```python
interpolation_limit_time : "18h"
```
Durée de temps limite pour l'interpolation.

Le format est le même que celui de la variable _'resampling_windows'_.

-----------
```python
NaNs_percentage : 0.2
```
Proportion de NaNs max toléré par variable.

-----------
```python
selected_variables : {'BonVoyage':['Date/Time', 'Latitude', 'Longitude', 'Current Direction [deg]']}
```
Pour sélectionner uniquement certaines variables (dictionnaire contenant en clé le nom de la feuille et en valeur une liste contenant le nom des variables à garder).

-----------
```python
agregation_function : "np.nanmean"
```
Fonction d'agregation par defaut pour la gestion des données numériques
qui apparaissent plusieurs fois dans la même fenêtre de temps.

<u>Attention:</u> il faut veillez à utiliser des fonctions qui ne prennent pas en compte les NaNs.

Exemples:
* `np.nanmean` pour calculer la moyenne.
* `np.nanmin` pour prendre le minimum.
* `np.nanmax` pour prendre le maximum.
* `percentile_agregation = lambda a: np.nanpercentile(a=a, q=70)` pour calculer le percentile d'ordre 70.


-----------
```python
set_custom_agg_function : {'BonVoyage_Current Direction [deg]': 'np.nanmin'}
```
Permet de choisir une fonction d'agrégation personalisée pour une variable donnée avec la syntaxe nom des variables en clé et fonction d'agrégation personalisée en valeur.

Pour sélectionner toutes les variables de _VRS_ uniquement, utiliser la clé `'VRS'`.

-----------
```python
set_custom_inter_method : {}
```
Permet de choisir une méthode d'interpolation personalisée pour une variable donnée avec la syntaxe 
nom des variables en clé et méthode d'interpolation personalisée en valeur.

Pour sélectionner toutes les variables de _VRS_ uniquement, utiliser la clé `'VRS'`.

-----------
```python
set_custom_inter_limit_time : {"VRS": "1d"}
```
Permet de choisir une limite d'interpolation personalisée pour une variable donnée avec la syntaxe
nom des variables en clé et temps limite d'interpolation en valeur.

Pour sélectionner toutes les variables de _VRS_ uniquement, utiliser la clé `'VRS'`.

-----------
```python
geospatial_format : "dd"
```
Format des données géospatiales en sortie.
Formats disponibles:
* `"dd"` pour la notation degré-decimal.
* `"dm"` pour la notation degré-minute.
* `"dms"` pour la notation degré-minute-seconde.

-----------
```python
deg_format : "initial"
```
Format des données de type angle en sortie.

Formats disponibles:
* `"continuous"` pour utiliser une notation continue (unwrap).
* `"0:360"` pour la notation comprise dans [0:360].
* `"-180:180"` pour la notation comprise dans [-180:180].
* `"initial"` pour utiliser la même notation que celle des données brutes (utile lorsque l'on veut passer du format continu après interpolation au format des données d'origines).

-----------
```python
dist_limit : 50
```
Distance maximale tolérée en km entre les données prédites et les données réelles.

-----------
```python
agregation_string_function : "first"
```
Methode d'agregation par defaut pour la gestion des données de type string (données _VRS_).

-----------
```python
fill_method : "ffill"
```
Methode pour compléter les NaNs dans le cas de données de type string.

Cette méthode utilise la fonction [fillna](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.fillna.html) de `pandas.DataFrame`.

3 paramètres sont disponibles:
* `"ffill"` propage la derniere chaine de caractère observée.
* `"bfill"` propage la prochaine chaine de caractère observée.
* Remplace les NaNs par une valeur définie par l'utilisateur.

-----------
```python
same_sheets : [["NMEA Gyro","NAVIGATION_GYRO"], ["NMEA GPS","NAVIGATION_GPS"],["Bridge Sensor","Motion Sensor"]]
```
Feuilles contenants des variables identiques.

<u>Attention:</u> les variables qui ne sont pas présentes dans toutes les feuilles d'une même
paire sont supprimées

## Structure du code

### Classe Files:
Classe générique d'un fichier de capteurs.

Attributs:
* `path` : string contenant le chemin d'accès au fichier source .xlsx.
* `filename` : string contenant le nom du fichier.
* `extension` : string contenant l'extension du fichier source.
* `content` : dictionnaire de DataFrames chaquue clé correspondant à un nom de feuille et chaque valeur correspondant au données contenues dans cette feuille.
* `sheets_name` : liste contenant les nom de feuilles du fichier.
* `var_names` : dictionnaire contenant en clé un nom de feuille et en valeur une liste cotenant le nom des variables contenues dans cette feuille.
* `time_var` : nom de la variable temporelle.

Méthodes:
* `__init__`: Fonction qui charge les données dans un dictionnaire de dataframes,
chaque clé correspondant à une feuille
* `set_date_variable` : Fonction qui charge la variable temporelle et la renomme.
* `add_sheet_prefix` : Fonction qui permet d'ajouter en prefixe de chaque variable avant le nom de la feuille.
* `merge_all_sheets` : Fonction qui permet de mettre toutes les feuilles dans un seul DataFrame.
* `print_file_infos` : Fonction qui permet d'afficher le contenu de notre fichier.
* `rename_sheets` : Fonction qui permet de renommer certaines feuilles.
* `get_sheets_stats` : Fonction qui permet de caclucler les statistiques de base (moyenne, minimum, ecart-type...)
pour chaque variable de chaque feuille.
* `load_columns_types` : Fonction qui permet de créer un attribut contenant pour chaque feuille
le type de chaque variable.
* `var_with_too_much_nans_filter` : Fonction qui permet de supprimer les variables contenant trop de NaNs.
* `save_file` : Fonction qui permet de sauvegarder l'objet dans un fichier .xlsx.

### Classe ShipResampler:
Classe représentant un bateau et les paramètres de ré-échantillonages.

Attributs:
* `files` : liste contenant toutes les fichiers VRS et de capteurs correspondant au bateau.
* `name` : nom du bateau.
* `NaNs_percentage` : pourcentage de NaNs max toléré.
* `resampling_windows` : fenêtre de ré-échantillonage.
* `agregation_function` : fonction d'agregation par defaut pour la gestion des données numériques
qui apparaissent plusieurs fois dans la même fenêtre de temps.
* `interpolation_method` : méthode par défaut pour interpoler les données numériques ie approximation dans le cas ou une
fenêtre de temps de contient aucune valeur pour une variable donnée.
* `interpolation_limit_time` : durée de temps limite pour l'interpolation.
* `agregation_string_function` : methode d'agregation par defaut pour la gestion des données de type string.
* `fill_method` : methode pour compléter les NaNs dans le cas de données de type string.
* `same_sheets` : feuilles contenants des variables identiques
nb: les variables qui ne sont pas présentes dans toutes les feuilles d'une même paire sont supprimées.
* `set_custom_agg_function` : nom des variables en clé et fonction d'agrégation personalisée en valeur.
* `set_custom_inter_method` : nom des variables en clé et méthode d'interpolation personalisée en valeur.
* `set_custom_inter_limit_time` : nom des variables en clé et temps limite d'interpolation en valeur.
* `geospatial_format` : format des données géospatiales en sortie.
* `dist_limit` : distance maximale en km entre données prédites et données réelles.
* `preprocessed_data` : DataFrame contenant toutes les données concaténées et ré-échantillonnées.

Méthodes:
Les méthodes implémentées dans l'objet ShipResampler permettent d'appliquer différentes transformations sur les données initiales:

* `__init__` : Fonction qui charge les fichiers de type `Files` dans l'attribut `files` en sélectionnant le bon bateau quand il s'agit de fichiers de type *VRS*.
* `set_date_index` : Fonction qui permet de mettre la variable temporelle en index dans self.preprocessed_data.
* `set_columns_types` : Fonction qui permet de créer des attributs contenants le type de chaque variable présente dans `preprocessed_data`.
* `nans_filter` : Fonction qui permet de supprimer toutes les fenêtres de temps contenants des NaNs dans `preprocessed_data`.
* `resample_and_interpolate` : Fonction qui permet de resampler, aggréger et interpoler les données concaténées dans `preprocessed_data`.
* `change_spatial_format` : Fonction qui permet de changer le format des données géospatiales.
* `change_deg_format` : Fonction qui permet de convertir les données angulaire.
* `distance_limit_filter` : Fonction qui permet de filtrer les fenêtres de temps lorsque les données réelles et prédites diffèrent trop.
* `save_preprocessed_data` : Fonction qui permet de sauvegarder le dataframe `preprocessed_data` dans un fichier .xlsx ou .csv.


### resampler.py:
Script qui permet de récupérer les paramètres entrés en ligne de commande et dans le fichier `.json` puis de créer des objets de types Files et ShipResampler avant d'appliquer toutes les transformations.

Il est possible de modifier les transformations appliquées en retirant ou en ajoutant des méthodes de transformations sur l'objet `ShipResampler`.
Par exemple, si on veut ne pas faire de modifications sur les données angulaires, on peut modifier le code de la manière suivante:

```python
# Application des transformations sur les données
magellan_ship.concat_and_delete_NaNs(prefix_sheet=True)\
        .change_spatial_format(geospatial_format='dd')\
        #.change_deg_format(auto_detect_deg_var=True,        auto_detect_deg_format=True, imposed_columns=[], deg_format='continuous', include_geo_var=True)\
        .resample_and_interpolate()\
        .nans_filter()\
        .distance_limit_filter()\
        #.change_deg_format(auto_detect_deg_var=True, auto_detect_deg_format=False, imposed_columns=[], deg_format=deg_format, include_geo_var=True)\
        .change_spatial_format(geospatial_format=geospatial_format)
```


