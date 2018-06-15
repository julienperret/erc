**__! Les dernières modifications sur prepare.py obligent à recharger la donnée locale depuis le SFTP !__**  

## ./prepare.py  
Ce script doit être lancé en ligne de commande avec au moins 2 arguments :  
    1 : répertoire des données globales  
    2 : n° de département dans lequel la zone d'étude est située  
    3 : répertoire des données locales  
    4 : répertoire des résultats (créé si besoin)  
    5 : chaîne de paramètres séparés d'un espace, dans n'importe quel ordre (optionnel)  

*Paramètres disponibles :*  

| Processus                                  | Variable ou fichier              | Minimum | Maximum | Unité | Valeur par défaut | Description                                                                                              |
|--------------------------------------------|----------------------------------|---------|---------|-------|-------------------|----------------------------------------------------------------------------------------------------------|
| Extraction des données                     | bufferDistance                   | 500     | 2000    | m     | 1000              | Taille de tampon utilisée pour extraire les données (dépend des distances max pour les raster d’intérêts |
| Création de la grille                      | pixRes                           | 20      | 200     | m     | 50                | Résolution de la maille de la grille / des rasters                                                       |
| Estimation de la population dans la grille | levelHeight                      | 2       | 3       | m     | 3                 | Hauteur utilisée pour estimer le nombre d’étages                                                         |
|                                            | minSurf                          | 50      | 100     | m²    | 50                | Surface minimale pour un bâtiment habitable                                                              |
|                                            | maxSurf                          | 5000    | 10000   | m²    | 10000             | Surface maximale pour un bâtiment habitable                                                              |
|                                            | useTxrp                          | False   | True    | bool  | True              | Utilisation du taux de résidences principales pour réduire la surface plancher estimée                   |
| Création des rasters de distance           | roadDist                         | 50      | 300     | m     | 200               | Distance maximale aux routes                                                                             |
|                                            | transDist                        | 100     | 500     | m     | 300               | Distance maximale aux arrêts de transport                                                                |
| Création du raster de restriction          | maxSlope                         | ?       | 30      | %     | 30                | Seuil de pente pour interdiction à la construction                                                       |
|                                            | maxOverlapRatio                  | 0       | 1       | float | 0,2               | Seuil de chevauchement max entre une cellule et une couche (cimetières, surfaces en eau…) pour exclusion |
| Création des rasters de densité SIRENE     | global_data/sirene/poids.csv     | 1       | +       | int   | 1                 | Poids de chaque raster de densité de points SIRENE                                                       |
|                                            | global_data/sirene/distances.csv | 100     | 1000    | m     | bufferDistance    | Distances maximales de recherche pour chaque raster de densité de points SIRENE                          |

*Mots magiques :*  

* force = suppression du répertoire de sortie si il existe  
* speed = utilisation de plusieurs threads (peut coûter cher en RAM !)  
* truth = écriture des .tif directement dans le répertoire de sortie sans conserver les données intermédiaires  
* silent = aucun 'print' durant l'exécution  

En cas de problème avec les "distance_trucmuche.tif", vérifier qu'il y a un header dans les fichiers csv.  

Usage :  
python3 prepare.py ./global_data/ 34 ./mtp/ ./results/ "pixRes=50 speed"  

Dépendances pour python3 :  
    PyQt5, qgis, gdal, numpy  

## ./simulate.py  
Deux paramètres au minimum :  
    1 : répertoire contenant la donnée  
    2 : répertoire des résultats (créé si besoin)  
    3 : le taux annuel d'évolution de la population (en %), -1 pour utiliser le taux moyen 2009 - 2014  
    4 : chaîne de paramètres séparés d'un espace, dans n'importe quel ordre (optionnelle)  

*Paramètres disponibles :*  

| Variable ou fichier  | Minimum   | Maximum    | Unité  | Valeur par défaut | Description                                                                                                           |
|----------------------|-----------|------------|--------|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| rate                 | 0         | 3          | %      | non               | Taux d’évolution annuel de la population                                                                              |
| scenario             | reduction | tendanciel | string | tendanciel        | Scénario de consommation d’espace (réduction, stable, tendanciel)                                                     |
| pluPriority          | False     | True       | bool   | True              | Utilisation du PLU pour peupler les ZAU en priorité                                                                   |
| buildNonRes          | False     | True       | bool   | True              | Pour simuler la construction au sol de bâtiments non résidentiels (en utilisant un taux de résidentiel par IRIS)      |
| densifyGround        | False     | True       | bool   | False             | Pour autoriser à densifier au sol des cellules déjà construite (si la capacité au sol le permet - voir maxBuiltRatio) |
| maxBuiltRatio        | 50        | 100        | %      | 80                | Taux maximal de la surface bâtie au sol d’une cellule                                                                 |
| densifyOld           | False     | True       | bool   | False             | Pour autoriser à augmenter la surface plancher dans des cellules urbanisées avant le début de la simulation           |
| maximumDensifty      | False     | True       | bool   | False             | Pour utiliser le maximum de la surface autorisée dans chaque cellule - au sol ou en plancher                          |
| winSize              | 3         | 9          | pixel  | 3                 | Taille en pixels du côté de la fenêtre glissante pour calcul de la somme ou de la moyenne des valeurs voisines        |
| minContig            | 0         | 3          | pixel  | 1                 | Nombre minimal de cellules urbanisées contiguës pour urbanisation d’une cellule vide                                  |
| maxContig            | 0         | 8          | pixel  | 5                 | Nombre maximal de cellules urbanisées contiguës pour urbanisation d’une cellule vide                                  |
| local_data/poids.csv | 1         | +          | int    |                   | Poids de chaque raster d’aménités pour la création du raster final interet                                            |

Usage :  
    ./simulate.py ./workspace/mtp/simulation_50m/ ./results/ 0.5 'scenario=tendanciel buildNonRes=True'

Dépendances pour python3 :  
    gdal, numpy  

### Commandes CARE qui semblent marcher :  

care -o ./prepare.tgz.bin  -p ./mtp -p ./global_data ./prepare.py ./global_data/ 34  ./mtp/ ./results/ "pixRes=50 useTxrp=True levelHeight=3 force"  

care -o /my/care/output/dir/simulation.tgz.bin -p /my/global/data/ -p /my/local/data/ -p /my/prepared/data/ ./simulation.py /my/prepared/data/ /my/output/dir/ 50 0.5 "mode=souple saturateFirst=True pluPriority=False"  

ATTENTION : derrière -p : mettre les chemins en absolu

## Outils  

### ./toolbox.py   
Contient les fonctions communes; à déplacer avec tout script sorti du dépôt.  

### ./insee_to_csv.py   
Convertir les données XLS de l'INSEE en CSV en supprimant les champs inutiles, à lancer une seule fois pour toute la région  
Dépendances pour python3 :  
    pandas + xlrd (pour manipuler les .xls)  

Usage :  
./insee_to_csv.py ../global_data/insee/  

### ./tif_to_gif.py  
Génère un GIF à partir des tifs de population générés pour chaque année de la simulation.  
Trois paramètres au minimum:  
    1 : dossier contenant les images pour chaque année  
    2 : dossier de sortie
    3 : type de donnée des images (byte, uint16, uint32, float32)  
    4 : chaîne contenant la durée du GIF et la valeur max à utiliser (delay=n , maxValue=n)  

Usage :  
./tif_to_gif.py ./results/snapshots/surface_sol ./output/ uint16

### ./magic.py  
Convertit les fichiers positionnels MAJIC III en CSV avec création de tables PSQL  
