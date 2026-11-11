# Enregistrement et format .daq

Un fichier `.daq` correspond au format d'enregistrement des capteurs de lumière d'Chloros : une **base de données SQLite** contenant des images spectrales calibrées provenant d'un capteur DAQ. Enregistrez-en un lors d’une session de capture et le pipeline de réflectance pourra ensuite diviser chaque image par l’irradiance descendante mesurée à ce moment précis.

## Contenu d’un fichier .daq

| Propriété | Valeur |
| --- | --- |
| Conteneur | Base de données SQLite, un fichier par capteur et par enregistrement |
| Nom de fichier | Comprend l’**identifiant du capteur**et un**horodatage**, par exemple `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spectre par image | 135 points, 340–1010 nm par pas de 5 nm, plus le tristimulus CIE XYZ |
| Unités | Irradiance spectrale étalonnée, **W/m²/nm** (ensemble d’étalonnage d’usine + profil de capuchon appliqué) |
| Métadonnées enregistrées | ID du capteur (clé permettant de récupérer l’étalonnage d’usine de cet appareil) et profil de capuchon en vigueur — voir [Profils de capuchon et plage étalonnée](caps-and-range.md) |

Le format est identique pour les modèles DAQ-U, DAQ-M, DAQ-E et DAQ-E-S ; le traitement en aval ne tient donc pas compte du périphérique de transport qui a effectué l’enregistrement. Un enregistrement DAQ-E-S stocke également l’inclinaison, le roulis et le tangage de l’appareil à côté de chaque spectre.

**Les enregistrements survivent à une coupure de courant (1.3.0).**Un enregistrement en direct est écrit avec un journal et une synchronisation complète du disque à chaque validation ; ainsi, une coupure de courant ou l’arrêt forcé d’un processus ne fait perdre que les 3 dernières secondes, et non le fichier. Un enregistrement interrompu en cours d’écriture peut laisser un fichier `<name>.daq-journal` à côté du fichier `.daq` :**conservez et copiez les deux ensemble.** Chloros effectue un retour en arrière du journal lors de la première lecture du fichier. Un disque saturé retarde l’écriture du fichier, mais jamais les mesures du capteur.

**Un DAQ-E qui cesse de transmettre est reconnecté (1.3.0).** Lorsqu’un flux DAQ-E s’interrompt (redémarrage de son commutateur, perte de connexion), le backend le reconnecte avec ses paramètres précédents, et l’enregistrement se poursuit dans le même fichier `.daq`, l’interruption étant signalée dans le fichier.

L’enregistrement calibré nécessite le pack d’étalonnage d’usine du capteur. Pour les DAQ-U et DAQ-M, le backend récupère le pack depuis le cloud d’MAPIR à l’aide de l’identifiant du capteur (l’enregistrement est refusé s’il n’y parvient pas) ; les unités DAQ-E font exception car elles conservent leur étalonnage sur l’appareil.

## Enregistrement depuis l’interface graphique

L’enregistrement via l’interface graphique nécessite un **projet ouvert** (sinon, les boutons d’enregistrement sont désactivés) :

* **Enregistrer tout / Arrêter tout** — en haut de la barre latérale des capteurs de lumière ; lance ou arrête simultanément un enregistrement `.daq` sur tous les capteurs connectés.
* **Enregistrer / Arrêter l’enregistrement** — par capteur, dans la fenêtre modale des paramètres (icône en forme d’engrenage). Un indicateur « REC » rouge s’affiche dans les lignes d’informations en temps réel du capteur pendant l’enregistrement.

Les fichiers sont enregistrés au format `<project>/light_sensor/`, et lorsqu’un enregistrement s’arrête — que ce soit via Arrêter, Tout arrêter ou en déconnectant un capteur d’enregistrement —, le fichier `.daq` finalisé est **automatiquement ajouté au projet ouvert**. Il apparaît dans la liste des fichiers du projet sans aucune étape d’ajout manuel, déjà prêt pour le traitement de la réflectance.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Enregistrement depuis l’CLI



L’CLI effectue l’enregistrement via le pool de capteurs du backend (le backend doit être en cours d’exécution — ces commandes sont des clients HTTP légers) :

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

Récupère la valeur `--sensor-id` à partir de `chloros-cli daq pool-list`. Deux valeurs par défaut à connaître :

| Option | Valeur par défaut |
| --- | --- |
| `--duration` | `0` — enregistrer jusqu’à `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` sur le système de fichiers du **backend**, et non celui de l’CLI |

La distinction entre les répertoires de sortie est importante lorsque la CLI cible un backend situé sur une autre machine : le fichier est enregistré à l’emplacement où le backend s’exécute. Lorsque le backend se trouve sur le même ordinateur, un chemin relatif `-o` (tel que `./out` ci-dessus) est résolu par rapport au dossier dans lequel vous avez exécuté la commande.

## Enregistrement depuis Python

`DAQSensorSession` (renvoyé par `chloros_sdk.connect_daq_sensor()`) expose le même enregistrement mis en commun : `record_start(output_dir=None, device_name=None)` renvoie le chemin d’accès au fichier, `record_stop()` renvoie `{path, rows}`. Consultez la [Référence du SDK](../reference/sdk-reference.md) pour l’API complète de la session. Les classes de gestion directe du matériel du SDK (installations sur ordinateur de bureau uniquement) écrivent les enregistrements dans `~/Documents/DAQ/` par défaut ; pour les versions publiées, le chemin de stockage groupé ci-dessus est la méthode prise en charge.

## Utilisation d’un fichier .daq lors du traitement

Pour calculer la réflectance à partir d’images, Chloros a besoin d’une irradiance descendante correspondant à chaque exposition :

* **Conservez le fichier `.daq` avec les images.**Au moment du traitement, le pipeline détermine automatiquement l’**irradiance descendante correspondant à l’horodatage** à partir d’un fichier `.daq` enregistré (quel que soit le modèle de DAQ) — ou à partir d’un fichier `.csv` natif de DAQ-M — se trouvant à côté des images. Les enregistrements via l’interface graphique satisfont automatiquement à cette condition, puisqu’ils sont ajoutés au projet dès qu’ils s’arrêtent.
* **L’étalonnage est récupéré à la demande.** Si un ensemble d’étalonnages d’usine par caméra ou par DAQ n’est pas déjà mis en cache localement, Chloros le récupère automatiquement depuis le cloud d’MAPIR lors de la première utilisation (connexion Internet requise une seule fois ; mis en cache sous `~/.chloros/`).
* **Les captures en direct créent leur propre fichier sidecar.** Pour toute image de réflectance capturée en direct, la valeur DAQ effectivement utilisée est enregistrée sous la forme d’un fichier `.daq` dans le dossier `daq/` des captures, ce qui permet de retraiter la capture ultérieurement sans l’enregistrement d’origine.
* **Enregistrements provenant des enregistreurs `chloros_scripts` (1.3.0).**Un fichier `.daq` généré par les enregistreurs publics [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) est répertorié comme le modèle DAQ à l’origine de cet enregistrement, et sa limite peut être définie dans**Paramètres du projet → Capteur de lumière DAQ**, comme pour tout autre enregistrement. Ces enregistreurs consignent les comptages bruts du capteur ; Chloros récupère l’étalonnage d’usine du capteur via la liaison série et l’applique.
* **À qui appartient cette mesure (1.3.0).**À côté de chaque capture `.daq`, un `.daq.cameras.json` identifie la ou les caméras dont les images correspondent à cette lecture, et le traitement l'associe**uniquement aux images provenant de ces caméras**. Une caméra autonome ne disposant pas de son propre capteur de luminosité n’emprunte donc plus la mesure d’une autre caméra : il faut lui associer un capteur ou en enregistrer une. Un enregistrement réalisé avec le DAQ lui-même (Enregistrement via l’interface graphique, `daq pool-record`), ainsi que tout fichier `.daq` ne contenant pas ce fichier (y compris les captures plus anciennes), continue de servir toutes les caméras.

## Récupérer l’irradiance

Le traitement d’un projet exporte également tous les enregistrements de capteurs de lumière qu’il contient dans un
dossier `Light Sensor/` à côté des produits d’image. Cela ne nécessite **pas** d’images : un
capteur de lumière enregistré seul constitue une capture complète, et un dossier ne contenant que des fichiers `.daq`
est une entrée valide. Un projet ne contenant que des enregistrements de capteurs de lumière les répertorie et
les traite aussi bien depuis l’application que depuis l’CLI, et ils restent répertoriés après l’exécution. L’exécution
indique le nombre de produits de capteurs de lumière qu’elle a générés.

| Produit | Description |
| --- | --- |
| `<name>_calibrated.daq` | Une archive rétraitable suivant le même schéma qu’un enregistrement en temps réel, déclarant désormais le jeu d’étalonnage qui l’a produite. Sa réimportation **ne**provoque**pas** un deuxième étalonnage. |
| `<name>_calibrated.csv` | Irradiance spectrale en W/m²/nm sur la grille de longueurs d’onde propre au capteur, une ligne par lecture, plus des colonnes photométriques : puissance totale, lux photopiques et scotopiques, PPFD avec sa répartition bleu/vert/rouge, et longueur d’onde de crête. |

Un DAQ-U ou DAQ-M dont le fichier d’étalonnage ne peut pas être récupéré — vous êtes hors ligne, ou
ce capteur n’a pas d’étalonnage enregistré — est **ignoré avec indication du motif**, et n’est jamais enregistré
sous forme de fichier « étalonné » contenant des comptages bruts. Connectez-vous à Internet et relancez l’opération. Un DAQ-E
dispose de son propre fichier d’étalonnage ; il n’en a donc besoin que lorsque l’appareil n’est pas connecté et que
rien n’est mis en cache localement.

### DAQ-A : compteurs bruts, et pourquoi c’est la bonne réponse

Le **DAQ-A** est antérieur au système de fichier d’étalonnage par numéro de série et ne dispose d’aucun fichier à
récupérer. Ce n’est pas un oubli : un DAQ-A est étalonné sur le terrain à l’aide d’une
cible de réflectance, et l’étalonnage basé sur une cible ne nécessite que la réponse *relative*
du capteur — ce qui correspond exactement à ses comptages bruts. Chloros procède aujourd’hui à l’étalonnage à l’aide de ces derniers.

Ainsi, un enregistrement DAQ-A s’exporte, mais sous un nom différent :

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, et non `_calibrated` — il s’agit d’un nom de fichier différent plutôt que d’un indicateur à l’intérieur du fichier,
car cette information doit rester intacte lorsque le fichier est transféré par e-mail sous forme de nom nu. L’en-tête `.csv`
indique `raw spectral sensor counts (NOT irradiance)` et précise que les valeurs sont
comparables **au sein** du fichier et non entre les capteurs. Les colonnes qui n’ont de sens
que pour l’irradiance réelle — puissance totale, lux, PPFD — sont laissées vides plutôt que d’être
calculées à partir des comptages.

Les anciens enregistrements DAQ-A-SD (schéma v1.01 / v1.02) n’enregistrent que l’heure d’écriture du fichier, et non un
horodatage par lecture. Chloros ne fera pas correspondre les images à ces données — associer une image à une
heure d’écriture serait erroné sans pour autant paraître faux — mais l’exportation les lit correctement et
le fichier CSV indique de quelle horloge il s’agit.

Pour tout savoir sur la réflectance — capteur unique avec une caméra, et double capteur ambiant/objet —, consultez [Workflows de réflectance](reflectance.md).
