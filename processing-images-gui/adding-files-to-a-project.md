# Ajouter des fichiers à un projet

Une fois que vous avez créé ou ouvert un projet dans Chloros, l&#x27;étape suivante consiste à ajouter vos images multispectrales afin de commencer le traitement. L&#x27;onglet « File Browser » <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> vous permet d&#x27;importer facilement des images et de gérer votre ensemble de données.

## Accéder au « File Browser »

1. Ouvrez ou créez un projet dans Chloros
2. Cliquez sur l’icône **« File Browser »** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau « File Browser » affichera la liste des fichiers de votre projet

{% hint style="info" %}
**Types de fichiers pris en charge** :

* **Survey3W / Survey3N** : paires RAW+JPG et images JPG (RAW+JPG recommandé)
* **LATTICE** : captures `.tif` / `.tiff` — enregistrées par le contrôle de caméra Chloros ou par un hub LATTICE
* **Données des capteurs de lumière** : enregistrements `.daq` (DAQ-U/M/E) et journaux de rayonnement descendant DAQ-M `.csv` — importés avec les images pour effectuer l’étalonnage de la réflectance
{% endhint %}

***

## Ajouter des images à votre projet

Il existe deux méthodes principales pour ajouter des images à votre projet :

### Méthode 1 : Ajouter des fichiers

Utilisez cette option pour importer des fichiers image individuels ou une petite sélection de fichiers.

1. Cliquez sur le bouton **« Ajouter des fichiers »** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> situé en haut du panneau « Navigateur de fichiers »
2. Accédez au dossier contenant vos images
3. Sélectionnez un ou plusieurs fichiers image (maintenez la touche **Ctrl** enfoncée pour sélectionner plusieurs fichiers)
4. Cliquez sur **« Ouvrir »** pour importer les fichiers sélectionnés

### Méthode 2 : Ajouter un dossier

Utilisez cette option pour importer toutes les images d’un dossier en une seule fois. Vous pouvez sélectionner **plusieurs dossiers** dans une même boîte de dialogue.

1. Cliquez sur le bouton **« Ajouter un dossier »** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> situé en haut du panneau « Navigateur de fichiers »
2. Accédez au(x) dossier(s) contenant les images de votre session de capture et sélectionnez-le(s)
3. Cliquez sur **« Sélectionner le dossier »** pour importer toutes les images prises en charge

{% hint style="info" %}
**Les fichiers qui ne parviennent pas à se charger sont signalés.** Si un dossier contient des fichiers que Chloros reconnaît mais ne parvient pas à charger, un avertissement vous en informe — les images ne disparaissent pas discrètement de la grille.
{% endhint %}

***

## Importation des dossiers de capture LATTICE

Les captures LATTICE sont enregistrées avec **un sous-dossier par niveau d’exportation** — par exemple `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — avec le fichier correspondant de type « downwelling » `.daq` situé à la racine :

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Cochez la case « Ajouter le dossier » à la racine des captures** (`output/` ci-dessus). Lorsque le dossier sélectionné ne contient aucune image mais comporte des sous-dossiers, Chloros les explore automatiquement — les sous-dossiers de ce niveau ainsi que le dossier racine `.daq` sont tous récupérés en une seule fois.**Fonctionnement de l’importation des captures :*** Chaque capture est importée sous forme d’une **seule image**, regroupée par capture (et non pas une entrée par niveau). Les autres niveaux de la même capture apparaissent comme des modes d’affichage de cette image unique.
* **Le traitement commence toujours à partir de l’image brute.** Les autres niveaux sont visibles, mais seul le niveau `raw` est jamais traité par le pipeline — le retraitement d’un produit déjà traité entraînerait une double application des corrections, ce que Chloros refuse. Une exportation réimportée ne peut jamais occuper l’emplacement « brut » d’une capture.
* Un dossier de capture enregistré **sans** importation des données brutes s’affiche normalement, mais le traitement l’ignore et l’indique dans le journal. (L’option CLI `--input-level` permet de forcer un point d’entrée dans ce cas — voir [la référence CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* À partir de la version 1.3.0, un dossier de captures peut également contenir un dossier `clamp_masks/` (masques par pixel pour les niveaux de radiance et de réflectance — voir [Formats d’images de sortie](../output-image-formats.md#clamp-masks)). Les masques ne sont pas des images de la scène, et Chloros les ignore lors de l’importation, où qu’ils se trouvent.

**Les sessions du hub LATTICE** s’importent de la même manière : cliquez sur « Ajouter un dossier » et sélectionnez le dossier de session copié depuis le hub (il contient les fichiers `raw/` et `previews/`), ainsi que tout fichier journal de flux descendant DAQ-M `.csv`. Si l’étalonnage de la caméra ou du DAQ n’est pas encore enregistré dans la mémoire cache de votre ordinateur, Chloros le récupère automatiquement par numéro de série lors de l’importation (une connexion Internet est nécessaire une seule fois).***

## Comprendre le tableau du navigateur de fichiers

Une fois les images importées, elles apparaissent dans un tableau comportant les colonnes suivantes :

### Nom du fichier

* Nom de fichier d’origine provenant de l’appareil photo
* Respecte la convention de nommage de l’appareil photo (par exemple, IMG\_0001.RAW ou capture\_20260816\_101500\_SN213800234\_raw.tif)

### Horodatage

* Date et heure de la prise de vue
* Extrait des métadonnées EXIF de l’image
* Utilisé pour la correspondance des capteurs de lumière, la synchronisation PPK et la planification des cibles d’étalonnage

### Modèle d’appareil photo

* Configuration de l’appareil photo et du filtre détectée automatiquement
* Exemples Survey3 : Survey3W_RGN, Survey3N_OCN, Survey3W_RGB
* Exemples LATTICE : LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Utilisé pour appliquer les profils de traitement appropriés

### Colonne « Cible » (case à cocher)

* Cochez cette case pour les images contenant des cibles d’étalonnage
* Lorsqu’au moins une image est cochée, **seules les images cochées sont analysées** à la recherche de cibles
* Voir [Choix des images cibles](choosing-target-images.md) pour plus de détails

### Affichage des métadonnées des images

Cliquer sur le bouton bascule situé dans le coin supérieur droit, au-dessus du tableau, affiche les métadonnées de l&#x27;image sélectionnée dans la zone de la grille d&#x27;images.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Fichiers de capteurs de lumière dans votre projet

* Les fichiers `.daq` et `.csv` apparaissent dans la liste du navigateur de fichiers mais ne sont pas des images cliquables — ils fournissent l’irradiance descendante pour l’étalonnage de la réflectance.
* Chaque fichier `.daq`/`.csv` importé est répertorié dans **Paramètres du projet → Capteur de lumière DAQ**, où vous pouvez vérifier la correction du capuchon diffuseur appliquée à chaque fichier. Voir [Réglage des paramètres du projet](adjusting-project-settings.md).
* Les enregistrements que vous effectuez dans l’onglet **Capteurs de lumière** sont automatiquement ajoutés au projet ouvert — aucune importation manuelle n’est nécessaire.***

## Gestion des fichiers dans votre projet

### Suppression de fichiers

Pour supprimer des images indésirables de votre projet :

1. Sélectionnez une ou plusieurs images dans le tableau du navigateur de fichiers
2. Cliquez sur le bouton **« Supprimer les éléments sélectionnés »** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Confirmez la suppression (les fichiers ne sont pas supprimés du disque, mais uniquement retirés du projet)

### Tri et filtrage

* **Tri par colonne** : cliquez sur l’en-tête d’une colonne pour trier les images
* **Tri par horodatage** : utile pour organiser des séquences de prises de vue chronologiques
* **Filtre par modèle d’appareil photo** : regroupez les images par type d’appareil photo si vous utilisez plusieurs appareils***

## Aperçu des images

### Affichage de l’image en entier

Cliquez sur n’importe quelle vignette dans le navigateur de fichiers pour l’afficher dans la zone d’aperçu principale :

1. L’image apparaît dans le panneau d’aperçu central
2. Utilisez les commandes de zoom pour examiner les détails de l’image
3. Naviguez entre les images à l’aide des touches fléchées

### Navigation rapide

* **Image précédente** : cliquez sur la flèche gauche ou appuyez sur la touche ←
* **Image suivante** : cliquez sur la flèche droite ou appuyez sur la touche →
* **Zoom avant/arrière** : utilisez la molette de la souris ou les boutons de zoom
* **Panoramique** : cliquez et faites glisser le curseur sur l’image lorsque vous êtes en zoom avant***

## Gestion des fichiers en double

Chloros détecte et ignore automatiquement les fichiers en double :

* Les fichiers dont les noms sont identiques sont ignorés
* Cela évite tout double traitement accidentel
* Un message d’avertissement s’affiche lorsque des doublons sont détectés

{% hint style="warning" %}
**Important** : ne renommez pas et ne modifiez pas vos fichiers image originaux avant l’importation. Chloros s’appuie sur les noms de fichiers et les métadonnées d’origine pour assurer un traitement correct.
{% endhint %}

***

## Ensembles de données provenant de plusieurs caméras

Si votre projet contient des images provenant de plusieurs caméras MAPIR :

1. Chloros détecte automatiquement chaque modèle de caméra — Survey3, LATTICE ou une combinaison des deux
2. Chaque type de caméra est traité avec son profil d’étalonnage approprié
3. L’explorateur de fichiers affiche le modèle de caméra dans la colonne « Modèle de caméra »
4. Chaque caméra dispose de sa propre arborescence de dossiers de sortie une fois traitée

**Exemples de scénarios** : configuration à deux caméras Survey3W RGN + Survey3N OCN, ou un réseau LATTICE avec une caméra principale RGB et plusieurs modules à bande étroite***

## Bonnes pratiques

### Organisation avant l’importation

* Conservez les images de cibles d’étalonnage dans le même dossier que les images de levé
* Conservez les fichiers de capteurs de lumière `.daq` / `.csv` de chaque session de capture avec les images de cette session
* Conservez la structure de dossiers d’origine de votre caméra/carte SD/hub
* Ne mélangez pas les jeux de données provenant de sessions différentes dans un même projet

### Nommage des fichiers

* Conservez les noms de fichiers d’origine de l’appareil photo (IMG\_0001.RAW, capture\_..., etc.)
* Ne renommez pas les fichiers avant l’importation
* Les noms d’origine contiennent des métadonnées importantes

### Images de cibles d’étalonnage

* Incluez toujours 1 à 2 images de cibles d’étalonnage par session (pour Survey3 ; pour LATTICE, un enregistrement DAQ peut les remplacer — voir [Choix des images de cibles](choosing-target-images.md))
* Capturez les cibles avant et après la session de capture
* Placez les cibles dans les mêmes conditions d’éclairage que la zone de capture
* Marquez les images cibles à l’aide de la case à cocher « Cible »

***

## Problèmes courants et solutions

### Les images n’apparaissent pas après l’importation

**Causes possibles :**

* Format de fichier non pris en charge (voir la liste des types pris en charge en haut de cette page)
* Les images proviennent d’appareils photo non MAPIR (voir [Appareils photo pris en charge](../supported-cameras.md))
* Fichier corrompu ou transfert incomplet depuis la carte SD

**Solution** : Vérifiez la compatibilité du format de fichier et du modèle d’appareil photo, puis consultez l’avertissement de chargement des fichiers pour identifier précisément ceux qui ont échoué

### Modèle d’appareil photo non détecté

**Causes possibles :**

* Métadonnées EXIF modifiées
* Images retouchées dans un logiciel externe
* Transfert de fichiers incomplet

**Solution** : réimportez les fichiers originaux non modifiés depuis l’appareil photo ou la carte SD

### Horodatages manquants

**Causes possibles :**

* Horloge de l’appareil photo mal réglée
* Données EXIF supprimées par un logiciel externe

**Solution** : Vérifiez que les réglages de l’heure de l’appareil photo étaient corrects lors de la prise de vue

### Le projet rouvert signale des fichiers manquants

Si des fichiers source ont été déplacés ou supprimés depuis la dernière ouverture du projet, Chloros vous indique **quels**fichiers ont disparu au lieu d’afficher une grille vide. Si le dossier du projet lui-même a été déplacé, Chloros redirige ses chemins d’accès vers le dossier dans lequel se trouve désormais `project.json`, et conserve une sauvegarde (`project.json.before-relink-<date>-<time>`). Lorsque les fichiers ne sont pas trouvés, la fenêtre**Fichiers source introuvables**propose de**Parcourir** le nouvel emplacement, puis de créer un nouveau projet ou de supprimer le projet. Depuis l’CLI, utilisez `chloros-cli project open PATH --relink FOLDER` ; depuis le SDK, `open_project(path, relink=...)`.***

## Étapes suivantes

Une fois vos fichiers importés :

1. **Vérifiez la liste des fichiers** - Assurez-vous que toutes les images ont été chargées correctement
2. **Vérifiez les modèles d&#x27;appareils photo** - Vérifiez que la détection des appareils photo est correcte
3. **Marquez les images cibles** – Voir [Choix des images cibles](choosing-target-images.md)
4. **Ajustez les paramètres** – Configurez les options de traitement dans [Paramètres du projet](adjusting-project-settings.md)
5. **Lancez le traitement** - Voir [Lancer le traitement](starting-the-processing.md)

Pour plus d&#x27;informations sur la configuration du projet, consultez [Réglage des paramètres du projet](adjusting-project-settings.md).
