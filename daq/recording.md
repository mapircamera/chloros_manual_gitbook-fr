# Enregistrement et format .daq

Un fichier `.daq` correspond au format d&#x27;enregistrement du capteur de lumière Chloros : une **base de données SQLite** contenant des images spectrales calibrées provenant d&#x27;un capteur DAQ. Enregistrez-en un lors d’une session de capture et le pipeline de réflectance pourra ensuite diviser chaque image par l’irradiance descendante mesurée à ce moment précis.

## Contenu d’un fichier .daq

| Propriété | Valeur |
| --- | --- |
| Conteneur | Base de données SQLite, un fichier par capteur et par enregistrement |
| Nom de fichier | Comprend l’**identifiant du capteur**et un**horodatage**, par exemple `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spectre par image | 135 points, 340–1010 nm par pas de 5 nm, plus le tristimulus CIE XYZ |
| Unités | Irradiance spectrale étalonnée, **W/m²/nm** (ensemble d’étalonnage d’usine + profil de capuchon appliqué) |
| Métadonnées enregistrées | ID du capteur (clé permettant de récupérer l’étalonnage d’usine de cet appareil) et profil de cap en vigueur — voir [Profils de cap et plage étalonnée](caps-and-range.md) |

Le format est identique pour les modèles DAQ-U, DAQ-M et DAQ-E ; le traitement en aval ne tient donc pas compte du périphérique de transport qui a effectué l’enregistrement.

L’enregistrement étalonné nécessite le pack d’étalonnage d’usine du capteur. Pour les DAQ-U et DAQ-M, le backend récupère le ensemble depuis le cloud de MAPIR à l’aide de l’identifiant du capteur (l’enregistrement est refusé s’il n’y parvient pas) ; les appareils DAQ-E font exception car ils conservent leur étalonnage sur l’appareil.

## Enregistrement depuis l’interface graphique

L&#x27;enregistrement via l&#x27;interface graphique nécessite un **projet ouvert** (sinon, les boutons d&#x27;enregistrement sont désactivés) :

* **Enregistrer tout / Arrêter tout** — en haut de la barre latérale « Capteurs de lumière » ; lance ou arrête simultanément un enregistrement `.daq` sur tous les capteurs connectés.
* **Enregistrer / Arrêter l’enregistrement** — par capteur, dans la fenêtre contextuelle des paramètres (icône en forme d’engrenage). Un indicateur « REC » rouge s’affiche dans les lignes d’informations en temps réel du capteur pendant l’enregistrement.

Les fichiers sont enregistrés au format `<project>/light_sensor/`, et lorsqu’un enregistrement s’arrête — que ce soit via « Arrêter », « Tout arrêter » ou la déconnexion d’un capteur d’enregistrement — le fichier `.daq` finalisé est **automatiquement ajouté au projet ouvert**. Il apparaît dans la liste des fichiers du projet sans qu’il soit nécessaire de l’ajouter manuellement, et est déjà prêt pour le traitement de la réflectance.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Enregistrement à partir du CLI

Le fichier CLI s’enregistre via le pool de capteurs du backend (le backend doit être en cours d’exécution — ces commandes sont des clients légers HTTP) :

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Récupérer la valeur `--sensor-id` à partir de `chloros-cli daq pool-list`. Deux valeurs par défaut à connaître :

| Option | Valeur par défaut |
| --- | --- |
| `--duration` | `0` — enregistrer jusqu’à `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` sur le système de fichiers du **backend**, et non sur celui de CLI |

La distinction entre les répertoires de sortie est importante lorsque le fichier CLI cible un backend situé sur une autre machine : le fichier est alors enregistré à l&#x27;emplacement où le backend s&#x27;exécute.

## Enregistrement à partir de Python

`DAQSensorSession` (renvoyé par `chloros_sdk.connect_daq_sensor()`) expose le même enregistrement mis en commun : `record_start(output_dir=None, device_name=None)` renvoie le chemin d’accès au fichier, `record_stop()` renvoie `{path, rows}`. Consultez la [Référence SDK](../reference/sdk-reference.md) pour la session complète API. Les classes matérielles directes de SDK (installations sur ordinateur de bureau uniquement) enregistrent les données par défaut dans `~/Documents/DAQ/` ; pour les versions publiées, le chemin commun indiqué ci-dessus est la méthode prise en charge.

## Utilisation d’un fichier .daq lors du traitement

Pour calculer la réflectance à partir d’images, Chloros a besoin d’une irradiance descendante correspondant à chaque exposition :

* **Conservez le fichier `.daq` avec les images.**Au moment du traitement, le pipeline détermine automatiquement l’**irradiance descendante correspondant à l’horodatage** à partir d’un fichier `.daq` enregistré (quel que soit le modèle de DAQ) — ou à partir d’un fichier `.csv` natif de DAQ-M — se trouvant aux côtés des images. Les enregistrements GUI satisfont automatiquement à cette condition, puisqu’ils sont ajoutés au projet dès qu’ils s’arrêtent.
* **L&#x27;étalonnage est récupéré à la demande.** Si un ensemble d&#x27;étalonnages d&#x27;usine par caméra ou par DAQ n&#x27;est pas déjà mis en cache localement, le fichier Chloros le récupère automatiquement depuis le cloud de MAPIR lors de la première utilisation (connexion Internet requise une seule fois ; mis en cache sous `~/.chloros/`).
* **Les captures en direct créent leur propre fichier d&#x27;accompagnement.** Pour toute image de réflectance capturée en direct, la lecture DAQ effectivement utilisée est enregistrée sous forme de fichier d’accompagnement `.daq` à côté de l’image, ce qui permet de retraiter la capture ultérieurement sans l’enregistrement d’origine.

## Récupération de l’irradiance

Le traitement d’un projet exporte également tous les enregistrements de capteurs de lumière qu’il contient dans un
dossier `Light Sensor/` situé à côté des produits d’imagerie. Cela ne nécessite **pas** d’images : un
capteur de lumière utilisé seul constitue une capture complète, et un dossier ne contenant que des fichiers `.daq`
est une entrée valide. L’exécution indique le nombre de produits de capteur de lumière qu’elle a générés.

| Produit | Description |
| --- | --- |
| `<name>_calibrated.daq` | Une archive rétraitable suivant le même schéma qu’un enregistrement en temps réel, déclarant désormais le jeu d’étalonnage qui l’a produite. Sa réimportation **ne**provoque**pas** un deuxième étalonnage. |
| `<name>_calibrated.csv` | Irradiance spectrale en W/m²/nm sur la grille de longueurs d’onde propre au capteur, une ligne par lecture, plus des colonnes photométriques : puissance totale, lux photopiques et scotopiques, PPFD avec sa répartition bleu/vert/rouge, et longueur d’onde de crête. |

Un DAQ-U ou DAQ-M dont le fichier d’étalonnage ne peut pas être récupéré — vous êtes hors ligne, ou
ce capteur n’a pas d’étalonnage enregistré — est **ignoré avec indication du motif**, et n’est jamais enregistré
sous forme de fichier « étalonné » contenant des comptages bruts. Connectez-vous à Internet et relancez l’opération. Un DAQ-E
dispose de son propre fichier d’étalonnage ; il n’en a donc besoin que lorsque l’appareil n’est pas connecté et que
rien n’est mis en cache localement.

### DAQ-A : comptages bruts, et pourquoi c’est la bonne solution

Le **DAQ-A** est antérieur au système de fichier d’étalonnage par numéro de série et ne dispose d’aucun fichier à
récupérer. Ce n’est pas un oubli : un DAQ-A est étalonné sur le terrain à l’aide d’une
cible de réflectance, et l’étalonnage basé sur une cible ne nécessite que la réponse *relative*
du capteur — ce qui correspond exactement à ses comptages bruts. Chloros s’étalonne aujourd’hui à l’aide de ces derniers.

Ainsi, un enregistrement DAQ-A s’exporte, mais sous un nom différent :

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, et non `_calibrated` — il s’agit d’un nom de fichier différent plutôt que d’un indicateur à l’intérieur du fichier,
car cette information doit rester intacte lorsque le fichier est transmis par e-mail sous forme de nom nu. L’en-tête `.csv`
indique `raw spectral sensor counts (NOT irradiance)` et précise que les valeurs sont
comparables **au sein** du fichier et non entre les capteurs. Les colonnes qui n’ont de sens
que pour l’irradiance réelle — puissance totale, lux, PPFD — sont laissées vides plutôt que d’être
calculées à partir des comptages.

Les anciens enregistrements DAQ-A-SD (schéma v1.01 / v1.02) n&#x27;enregistrent que l&#x27;heure d&#x27;écriture du fichier, et non un
horodatage par lecture. Chloros ne fera pas correspondre les images à ces données — associer une image à un
moment d’écriture serait erroné sans que cela ne semble jamais faux — mais l’exportation les lit correctement et
le fichier CSV indique de quelle horloge il s’agit.

Pour tout savoir sur la réflectance — capteur unique avec une caméra, et double capteur ambiant/objet —, consultez [Workflows de réflectance](reflectance.md).
