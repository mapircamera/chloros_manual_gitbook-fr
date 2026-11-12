# Fin du traitement

Une fois le traitement terminé dans Chloros, il est temps d’examiner vos résultats, de vérifier la qualité du rendu et de préparer vos images traitées en vue de leur utilisation dans votre flux de travail. Cette page vous guide à travers les dernières étapes et les actions à entreprendre.

## Indicateurs de fin de traitement

Lorsque le traitement s’achève avec succès, plusieurs indicateurs s’affichent :

* ✅ **Barre de progression** : atteint 100 %
* ✅ **Journal de débogage** : affiche les dernières lignes `[RUN-SUMMARY]` avec les nombres correspondants (images, groupes de caméras, cibles, images calibrées, fichiers enregistrés)
* ✅ **Bouton Démarrer** : redevient actif (prêt pour le prochain traitement)
* ✅ **Fichiers de sortie** : toutes les images traitées sont enregistrées dans l’arborescence de sortie du projet (ci-dessous)

{% hint style="warning" %}
**Un cycle qui n’écrit aucune image est considéré comme un échec.** Si vous avez demandé des produits d’image et que le cycle n’en a écrit aucun, Chloros le signale comme un échec — le nom du journal `[RUN-SUMMARY]` indique la cause probable (rien n’a été importé, aucune cible détectée, ou tous les produits demandés ont été ignorés car inapplicables). L’équivalent en CLI se termine avec un code de sortie non nul. Une exécution délibérée ne portant que sur les métadonnées (tous les produits d’exportation désactivés, pas d’index) est tout de même considérée comme réussie. Voir [la référence CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localisation de vos images traitées

### Ouvrir le dossier de sortie

1. Cliquez sur l’icône **Menu principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(en haut à gauche)
2. Sélectionnez **« Ouvrir le dossier du projet »**

3. Votre explorateur de fichiers s’ouvre sur le répertoire du projet
4. Localisez votre projet par son nom

### L'arborescence de sortie

Les fichiers sont enregistrés **dans le dossier du projet, regroupés par appareil photo puis par format de fichier** :

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Dossier de l'appareil photo** : `LATT-<sensor>-<lens>-F<filter>` pour LATTICE (correspondant aux données EXIF de la capture `Model`), `<model>_<filter>` pour Survey3 (par exemple `Survey3N_RGN`). Deux caméras partageant un capteur et un filtre mais dont l'objectif diffère conservent des arborescences distinctes — la vignettage, le champ de vision et la distorsion diffèrent.
* **Dossier de format** : suit votre paramètre de format d’exportation — `tiff16`, `tiff8`, `png8`, `jpg8`, ou `tiff32` pour le format TIFF (32 bits, pourcentage). La radiance est toujours au format float32 et se trouve toujours sous `tiff32`.
* **Dossiers de produits** :
  * `Reflectance_Calibrated_Images/` — réflectance calibrée
  * `Debayered_Images/` — débayérisation linéaire (LATTICE) : décalage sombre supprimé et débayérisation effectuée, sans champ plat ni démixage, dans l’orientation native du capteur
  * `Preview_Images/` — aperçu à l’écran (LATTICE)
  * `Radiance_Images/` — radiance spectrale en float32, W/m²/sr/nm (multispectral LATTICE)
  * `Vignette_Corrected_Images/` **ou** `Sensor_Response_Images/` — solution de secours non calibrée pour les images sans référence de réflectance ; il n’en existe qu’une seule des deux par série, choisie par le paramètre de correction de vignettage
  * `<INDEX>_Index_Images/` — un dossier par index sélectionné (par ex. `NDVI_Index_Images`)
* **`clamp_masks/`** (nouveauté de la version 1.3.0, caméras multispectrales M3C de LATTICE uniquement) : apparaît dans un dossier de format, à côté des dossiers de produits, lorsque le démixage des bandes a écrêté les valeurs négatives à zéro dans un produit de radiance ou de réflectance. Il contient un masque par pixel pour chaque produit concerné, par exemple `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Les masques ne se trouvent jamais à l'intérieur d'un dossier de produit ; un dossier de produit peut donc être transmis tel quel à un logiciel de photogrammétrie. Voir [Masques de limitation](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Chaque produit exporté conserve le nom du fichier SOURCE.**Une exportation de radiance de `capture_..._raw.tif` s’appelle toujours `capture_..._raw.tif` — elle se trouve simplement dans `tiff32/Radiance_Images/`.**C’est le dossier qui identifie le produit, et non le nom de fichier** ; par conséquent, une recherche sur « `*radiance*.tif` » ne donnera aucun résultat ; il faut plutôt effectuer une recherche sur le répertoire.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Combien de fichiers devrait-il y avoir ?

Ne comptez pas à l’aide d’une formule : le nombre de fichiers de sortie dépend des produits qui ont été activés et de ceux qui s’appliquent à chaque caméra (par exemple, les caméras RGB ne génèrent pas de fichiers de radiance/réflectance). Le nombre faisant autorité se trouve dans le journal : la dernière ligne `[RUN-SUMMARY]` indique exactement combien de fichiers ont été créés, et des lignes d’explication précisent tout ce qui a été ignoré.

***

## Vérification des images traitées

### Aperçu rapide dans l’Explorateur de fichiers

**Aperçu intégré à Windows :**

1. Accédez à un dossier de produits (par exemple, `tiff16/Reflectance_Calibrated_Images/`)
2. Sélectionnez un fichier image
3. L'aperçu s'affiche dans le volet d'aperçu de l'Explorateur deWindows

4. Utilisez les touches fléchées pour parcourir les images

### Aperçu dans des visionneuses d'images externes

**Visionneuses recommandées :*** **QGIS** - Logiciel SIG gratuit (idéal pour l'analyse multispectrale géoréférencée)
* **IrfanView** - Visionneuse d’images rapide et légère (prend en charge le format TIFF)
* **Adobe Photoshop** - Édition professionnelle (prise en charge du format TIFF)
* **GIMP** - Alternative gratuite à Photoshop
* **Photos de Windows** - Visualisation basique (peut ne pas prendre en charge le format TIFF 16 bits)

### Aperçu dans la visionneuse d'images d'Chloros


Utilisez la visionneuse d'images intégrée à Chloros pour une visualisation avancée :

1. Cliquez sur une vignette d'image dans l'explorateur de fichiers
2. L'image s'ouvre dans la zone d'aperçu principale
3. Cliquez sur l'onglet **Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
dans la barre latérale gauche
4. Utilisez [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) pour une analyse interactive

Consultez la page [Visionneuse d’images](../image-viewer-gui/opening-an-image-full-screen.md) pour obtenir des instructions détaillées.

***

## Lecture des valeurs de réflectance des pixels (SIG / Pix4D / Scripts)

La réflectance est stockée sous forme de nombre entier DN, et **la valeur DN correspondant à ρ = 1,0 dépend de la caméra source** :

| Source          | ρ = 1,0 correspond à | Comment le déterminer                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (marge jusqu’à ρ 2,0) | La balise XMP `Chloros:PixelScale=32768` est apposée sur le fichier |
| Survey3          | **65535** (limité à ρ 1,0)     | Pas de balise `Chloros:PixelScale` — cette absence est le signe distinctif (les produits Survey3 comportent d’autres balises de traitement `Chloros:*`) |

**Lisez la balise `Chloros:PixelScale` et divisez par ce chiffre** plutôt que de supposer systématiquement une valeur de 65535 — diviser la réflectance LATTICE par 65535 réduit silencieusement de moitié chaque valeur. Un fichier de réflectance antérieur à la version 1.3.0, enregistré en 8 bits, a été écrêté, et non redimensionné, et ne comporte pas de balise d’échelle : réexportez-le en 16 bits ou 32 bits au lieu de le diviser. Voir [Formats d’image de sortie](../output-image-formats.md) pour plus de détails.

***

## Métadonnées conservées lors des exportations

Chaque produit conserve le **bloc GPS**de la capture source et son**sous-IFD EXIF** ; ainsi, une
exportation inclut les fichiers `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` et
`CameraSerialNumber`, ainsi que le géoréférencement.

{% hint style="warning" %}
**Si une orthomosaïque s'affiche à une échelle absurde, vérifiez d'abord `FocalLength`.**
Pix4D calcule la distance d’échantillonnage au sol à partir de la distance focale et de l’altitude. Sans cette balise, il
revient à une échelle complètement erronée — lors d’un vol mesuré de 49 prises de vue, une
a été reconstruite comme mesurant 47,8 km × 13 km, produisant une ortho de 455 mégapixels composée principalement
d’espace vide. Un mosaïquage lent et un fichier d’une taille inattendue sont les symptômes de ce problème, et non des
problèmes distincts.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Toutes les balises ne sont pas copiées. Les balises structurelles de l’IFD0 sont délibérément laissées de côté (leur copie
corrompt la sortie LATTICE), et les balises `ExifImageWidth` / `ExifImageHeight` sont exclues
car elles décrivent la capture d’origine — une exportation redimensionnée revendiquerait sinon
des dimensions contredites par sa propre trame.

Chloros écrit également ses propres balises XMP `Chloros:*` décrivant les opérations effectuées sur chaque produit
(par exemple `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). À partir de la version 1.3.0, les produits d’Survey3 les intègrent dans chaque ; les produits d’index décrivent l’index lui-même (`Chloros:ProcessingLevel=index`, la
formule et s’il a été calculé à partir de la réflectance ou de la radiance) plutôt que de répéter les
balises des données brutes sources, et les produits situés dans des dossiers dont le chemin d’accès dépasse 260 caractères conservent également leurs métadonnées. Voir [Formats d’image de sortie](../output-image-formats.md#index-products).

***

## Consultation du journal de débogage

### Vérification des avertissements ou des erreurs

1. Ouvrez l’onglet **Journal de débogage** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Faites défiler les messages
3. Recherchez les avertissements jaunes ou les erreurs rouges
4. Lisez les lignes `[RUN-SUMMARY]` et les éventuels conseils
5. Contactez le support MAPIR pour obtenir de l’aide

### Enregistrement du journal

Pour conserver une trace du traitement ou pour l'envoyer au support MAPIR :

1. Cliquez sur le bouton **« Copier »**ou**« Télécharger »**

2. Enregistrez-le sous forme de fichier texte dans le dossier du projet
3. Joignez-le à la documentation du projet
4. Envoyez-le au support MAPIR si vous rencontrez des problèmes

Pour signaler un problème directement à MAPIR, utilisez l’icône en forme d’insecte (**Signaler un bug**) en haut de l’onglet « Journal de débogage » — voir [Signaler un bug depuis l’onglet « Journal de débogage »](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Problèmes courants liés aux résultats et solutions

### Problème : fichiers de sortie manquants

**Causes possibles :**

* Le produit ne s’applique pas à cette caméra (par exemple, radiance/réflectance pour les caméras RGB — le journal l’indique)
* Une référence obligatoire manquait (par exemple, réflectance sans cible et sans rayonnement descendant `.daq`)
* La case à cocher d’exportation du produit était désactivée dans les paramètres du projet
* L’espace disque a été épuisé pendant l’exportation

**Solutions :**

1. Vérifiez les indications `[RUN-SUMMARY]` et les lignes `[EXPORT-CHECK]` dans le journal de débogage — elles expliquent les omissions par caméra
2. Vérifiez les cases à cocher d’exportation des produits dans [Paramètres du projet](adjusting-project-settings.md)
3. Vérifiez que l’espace disque était suffisant
4. Relancez le traitement après avoir corrigé la cause

### Problème : bords sombres ou clairs (vignettage toujours visible)

**Causes possibles :**

* Correction du vignetage désactivée
* Caméra/objectif non répertorié(e) dans la base de données des profils d’Chloros

* Vignetage extrême dépassant les capacités de correction

**Solutions :**

1. Vérifiez que la correction du vignetage est activée dans les Paramètres du projet
2. Vérifiez que le modèle de caméra est correctement détecté
3. Contactez le support MAPIR si le vignetage persiste

### Problème : couleurs ou valeurs incorrectes

**Causes possibles :**

* Aucune cible d’étalonnage détectée
* Modèle de cible d’étalonnage incorrect sélectionné
* Étalonnage de la réflectance désactivé
* Images des cibles de mauvaise qualité

**Solutions :**

1. Vérifiez que l’étalonnage de la réflectance a été activé
2. Vérifiez les messages « Cible trouvée » dans le journal de débogage
3. Vérifiez la qualité des images des cibles
4. Relancez le traitement en marquant les cibles appropriées

### Problème : les valeurs NDVI semblent erronées

**Plages NDVI attendues :*** **Eau, roches, sol** : -0,1 à 0,2
* **Végétation clairsemée/en mauvaise santé** : 0,2 à 0,4
* **Végétation modérée** : 0,4 à 0,6
* **Végétation saine et dense** : 0,6 à 0,9**Si les valeurs se situent en dehors de ces plages :**

1. Vérifiez que l'étalonnage de la réflectance a bien été appliqué
2. Vérifiez que le journal du capteur de lumière a bien été inclus
3. Vérifiez que les cibles d'étalonnage ont bien été détectées
4. Assurez-vous que le modèle de caméra correct a bien été détecté
5. Vérifiez les horaires et les conditions de capture des images cibles
6. Si vous calculez vous-même les indices à partir des fichiers de réflectance, assurez-vous d’avoir divisé par la valeur `Chloros:PixelScale` du fichier (voir ci-dessus)

***

## Utilisation de vos images traitées

### Pour la photogrammétrie / la création d’orthomosaïques

**Flux de travail recommandé :**

1.**Importez les images de réflectance calibrées** dans un logiciel de photogrammétrie :
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conservez les métadonnées EXIF** : assurez-vous que les données GPS sont préservées pour le géomarquage
3. **Flux de travail calibrés** : utilisez des images de réflectance pour une précision scientifique — la réflectance LATTICE contient les balises de calibrage XMP lues par Pix4D
4. **Traitez les mosaïques d’indices** : créez des orthomosaïques NDVI à partir d’images d’indices individuelles
5. **Exporter des fichiers GeoTIFF géoréférencés** : pour une utilisation dans des applications SIG

### Pour l’analyse SIG

**Flux de travail recommandé :**

1.**Importer dans QGIS, ArcGIS ou un logiciel similaire**

2.**Utilisez des images de réflectance au format TIFF 16 bits** pour l’analyse multibande (divisez par la valeur `Chloros:PixelScale` du fichier)
3. **Utilisez des images d’indice** (NDVI, NDRE) comme couches de végétation prêtes à l’emploi
4. **Calculateur raster** : combinez les bandes pour une analyse personnalisée
5. **Exportation** : créez des cartes de classification, détectez les changements, générez des cartes de santé de la végétation

### Pour l’analyse directe / la création de rapports

**Procédure recommandée :**

1.**Utilisez des images d’indice avec des couleurs LUT** pour les rapports visuels
2. **Extraire des statistiques** : NDVI moyen par champ/parcelle
3. **Séries chronologiques** : comparer les indices entre plusieurs sessions
4. **Générer des rapports** : inclure des cartes, des statistiques et des visualisations***

## Archivage et sauvegarde

### Stratégie de sauvegarde recommandée

**Éléments à sauvegarder :*** ✅ **Images RAW/JPG d’origine ou captures brutes LATTICE** - À archiver sur un disque dur séparé ou dans le cloud ; les données brutes constituent la source du pipeline et tout le reste peut être régénéré à partir de celles-ci
* ✅ **Fichiers du capteur de lumière `.daq` / `.csv`** – Nécessaires pour recalculer la réflectance ultérieurement
* ✅ **Résultats traités** – Conserver les images calibrées et les indices
* ✅ **Dossier du projet** (`project.json` et fichiers associés) - Contient tous les paramètres nécessaires à un nouveau traitement si besoin
* ✅ **Journal de débogage** - Documente les détails du traitement
* ✅ **Images cibles d’étalonnage** - Pour la vérification et le nouveau traitement**Recommandations de stockage :*** **Sauvegarde immédiate** : disque dur externe
* **Archivage à long terme** : stockage dans le cloud (Google Drive, Dropbox, etc.)
* **Données critiques** : conserver 2 à 3 copies à des emplacements différents***

## Prochains cycles de traitement

### Réutilisation des paramètres du projet

Si vous devez traiter des jeux de données similaires à l’avenir :

1. **Enregistrez le modèle de projet** (si ce n’est pas déjà fait)
2. **Créez un nouveau projet** à l’aide du modèle enregistré
3. **Importez les nouvelles images**

4.**Traitez**avec des paramètres identiques pour garantir la cohérence

### Traitement par lots de plusieurs sessions

Pour plusieurs sessions/ensembles de données :**Option 1 : Interface graphique (GUI) - Projets multiples**

* Créez un projet distinct pour chaque session
* Utilisez des paramètres de modèle cohérents
* Traitez-les une par une

**Option 2 : Chloros CLI (Chloros+ uniquement)**

* Automatiser le traitement par lots
* Traiter plusieurs dossiers à l’aide de scripts
* Consultez la [Documentation de l’CLI](../CLI.md) et la [Référence de l’CLI](../reference/cli-reference.md)

**Option 3 : Python SDK (Chloros+ uniquement)**

* Contrôle programmatique
* Intégration aux pipelines d’analyse
* Consultez la [documentation de l’API](../api-python-sdk.md) et la [référence du SDK](../reference/sdk-reference.md)

***

## Dépannage du post-traitement

### Nouveau traitement avec des paramètres différents

Si les résultats ne sont pas satisfaisants :

1. Conservez les images d'origine (ne les supprimez jamais)
2. Ouvrez le même projet dans Chloros
3. Modifiez les paramètres dans le panneau « Paramètres du projet »
4. Relancez le traitement — les résultats sont enregistrés dans les mêmes dossiers de produits ; les fichiers portant le même nom que lors de l'exécution précédente sont donc remplacés

### Traitement d’un sous-ensemble d’images

Pour retraiter uniquement certaines images :

1. Créez un nouveau projet
2. Importez uniquement les images nécessitant un nouveau traitement
3. Utilisez le même modèle de paramètres
4. Traitez un ensemble de données plus restreint

### Obtenir de l’aide

Si vous rencontrez des problèmes :

* 📧 **E-mail** : info@mapir.camera (joignez le journal de débogage)
* 🌐 **Assistance** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ** : [Foire aux questions](../faq.md)
* 📖 **Documentation** : [Manuel Chloros](../)

***

## Résumé : workflow complet

Vous avez désormais terminé l'intégralité du workflow de traitement Chloros :

1. ✅ **Projet créé** - Voir [Projets](../projects.md)
2. ✅ **Ajout de fichiers** - Voir [Ajout de fichiers](adding-files-to-a-project.md)
3. ✅ **Ajustement des paramètres** - Voir [Ajustement des paramètres du projet](adjusting-project-settings.md)
4. ✅ **Marquage des cibles** - Voir [Choix des images cibles](choosing-target-images.md)
5. ✅ **Lancement du traitement** - Voir [Lancement du traitement](starting-the-processing.md)
6. ✅ **Suivi de la progression** - Voir [Suivi du traitement](monitoring-the-processing.md)
7. ✅ **Résultats examinés** - Cette page**Vos images multispectrales calibrées et corrigées en réflectance sont prêtes à être analysées !**

***

## Ressources supplémentaires

### Fonctionnalités avancées

* [**Visionneuse d’images**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualisation et analyse interactives
* [**Bac à sable d'indices/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Tests d'indices personnalisés
* [**Formules d'indices multispectraux**](../project-settings/multispectral-index-formulas.md) - Référence complète des indices

### Automatisation et intégration

* [**Documentation CLI**](../CLI.md) - Traitement par lots en ligne de commande
* [**SDK Python pour CLI**](../api-python-sdk.md) - Automatisation par programmation
* [**Fonctionnalités de Chloros+**](../#chloros) - Capacités de traitement avancées

### Assistance et formation

* [**FAQ**](../faq.md) - Réponses aux questions courantes
* [**Cibles d'étalonnage**](../calibration-targets.md) - Comprendre l'étalonnage de la réflectance
* [**Caméras prises en charge**](../supported-cameras.md) - Matériel compatible
