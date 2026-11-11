# Calques d’image

Le **menu déroulant des calques** situé en haut à droite de la visionneuse d’images permet de basculer entre toutes les versions de l’image que vous consultez — depuis la capture source jusqu’aux images d’index calculées, en passant par chaque produit traité — sans quitter la visionneuse.

## Que sont les couches d’images ?

Dans Chloros, une « couche » correspond à un **fichier de produit**associé à une image source. L’importation vous fournit les fichiers sources ; le traitement ajoute une couche pour chaque produit généré par la session. Les fichiers exportés conservent le nom du fichier source : c’est le**dossier** qui identifie le produit, et le nom du calque correspond à l’étiquette attribuée par Chloros à ce dossier.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## La liste des calques

### Toujours présents

| Calque | Description |
| --- | --- |
| **JPG**(ou**PNG**/**TIFF**) | Le fichier de base issu de la capture. Survey3 importe un fichier `.JPG` à côté de chaque `.RAW` ; les captures LATTICE fournissent un aperçu d’affichage au format PNG ou TIFF. Étiqueté en fonction de ce qui a été réellement importé |
| **RAW (Original)** | L’image brute source, débayérisée pour l’affichage sans aucune correction appliquée. Disponible dès l’importation — elle ne nécessite aucun traitement |

Une capture LATTICE dont le fichier de base **est** son image brute ne dispose pas d’entrée de base distincte : `RAW (Original)` la couvre déjà.

### Produits de traitement d’Survey3



| Couche | Enregistrée dans | Existe lorsque |
| --- | --- | --- |
| **RAW (Cible)** | — | L’image a été identifiée comme contenant une cible d’étalonnage |
| **RAW (Réflectance)** | `Reflectance_Calibrated_Images/` | L’étalonnage de la réflectance s’est déroulé avec succès sur cette image |
| **Correction de la vignette**| `Vignette_Corrected_Images/` | L'image n'a pas pu faire l'objet d'un étalonnage de réflectance**et** la *correction de la vignette* était activée |
| **Réponse du capteur**| `Sensor_Response_Images/` | L'image n'a pas pu être étalonnée en réflectance**et** la *correction de vignettage* était désactivée |
| **Balance des blancs** | `White_Balanced_Images/` | Un produit avec balance des blancs a été généré |

{% hint style="info" %}
**La correction de vignettage et la réponse du capteur sont des options alternatives, jamais les deux à la fois.** Il existe exactement un produit de secours non calibré par exécution, pour chaque modèle de caméra, et le commutateur *Correction de vignettage* choisit lequel. Voir [Paramètres du projet](../project-settings/project-settings.md).
{% endhint %}

### Niveaux LATTICE

LATTICE capture le fan-out dans ces niveaux en un seul passage de traitement. Ceux qui existent dépendent des options d’exportation par produit dans les Paramètres du projet et de ce qui s’applique à l’appareil photo.

| Calque | Enregistré dans | S'applique à |
| --- | --- | --- |
| **RAW (débayérisé)** | `Debayered_Images/` | RGB et multispectral |
| **RAW (Aperçu)** | `Preview_Images/` | Multispectrales (étirement en fausses couleurs) |
| **Balance des blancs** | `Preview_Images/` | Caméras principales RGB — l’aperçu RGB est enregistré sous ce nom afin de correspondre à la couche Survey3 du même nom |
| **RAW (radiance)** | `Radiance_Images/` | Multispectral uniquement |
| **RAW (réflectance)** | `Reflectance_Calibrated_Images/` | Multispectral uniquement, et uniquement lorsqu’un enregistrement descendant `.daq` correspondant ou une cible intégrée à l’image ayant passé le contrôle qualité couvre l’image |

Les caméras principales RGB ne disposent pas de radiométrie par bande ; par conséquent, la radiance et la réflectance sont ignorées pour celles-ci car **non applicables** — le journal l'indique clairement plutôt que de générer une erreur silencieuse.

{% hint style="info" %}
**Les couches brutes et débayérisées s’alignent avec les produits (nouveauté de la version 1.3.0).**Les fichiers bruts et débayérisés LATTICE sont enregistrés dans l’orientation native du capteur et consignent dans le fichier XMP l’orientation qui leur est propre ; les autres produits sont enregistrés avec cette orientation déjà appliquée. La visionneuse et la grille d’images basent l’affichage des couches**RAW (Original)**et**RAW (Débayérisé)** sur cette information, de sorte que l’image ne se retourne plus lorsque vous passez d’une couche de produit à une couche brute. Il s’agit uniquement d’un affichage — les fichiers ne sont jamais réécrits. Les fichiers sans enregistrement d’orientation (capturés avant la version 1.3.0 ou créés par d’autres logiciels) sont affichés tels qu’ils ont été enregistrés. Voir [Formats d’image de sortie](../output-image-formats.md#orientation).
{% endhint %}

Les masques de limitation (`*_clampmask.tif` dans un dossier `clamp_masks/`) ne sont pas des calques : Chloros les ignore lors de l’importation. Voir [Formats d’image de sortie](../output-image-formats.md#clamp-masks).

### Calques d’index, de LUT et de zone de test

| Type de calque | Exemple | Origine |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Un par index configuré dans les paramètres du projet, calculé lors du traitement |
| **`<INDEX>` LUT** | `NDVI LUT` | La version avec mappage des couleurs d’un index |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Un par exécution d'exportation [Index/LUT Sandbox](index-lut-sandbox.md) |

Si le même nom d’index est configuré plusieurs fois avec des paramètres différents, le deuxième et les suivants se voient attribuer un numéro dans leur nom (`RAW (NDVI2 Index)`) afin que les calques restent identifiables.

***

## Utilisation du sélecteur de calque

1. Ouvrez une image en plein écran en cliquant sur une vignette dans la grille
2. Cliquez sur le **menu déroulant des calques** en haut à droite de la visionneuse
3. Choisissez un calque — l’image s’actualise immédiatement

Le menu déroulant affiche en premier **JPG, RAW (Original), RAW (Cible), RAW (Réflectance)**, dans cet ordre, puis répertorie tous les autres calques dans l’ordre d’enregistrement des produits.

### Préférence de calque lors de la navigation

Appuyer sur **←**/**→** permet de passer à l’image suivante tout en essayant de vous maintenir sur le même calque :

1. **Correspondance exacte en priorité** — si l’image suivante comporte un calque portant le même nom, c’est celui-ci qui s’affiche. C’est ce qui vous permet de rester sur `RAW (NDVI Index)` lorsque vous parcourez un ensemble complet
2. **Puis correspondance par type** — un calque d’index recherche n’importe quel calque d’index, une LUT n’importe quelle LUT, un calque de réflectance un calque de réflectance, un calque cible un calque cible, un calque original un calque original, un calque de base un calque de base
3. **Ensuite, pour les calques d’exportation uniquement** — le nom est conservé même si la liste des calques n’est pas encore à jour, car le fichier existe déjà sur le disque. C’est ce qui vous permet de prévisualiser les produits pendant qu’un traitement est encore en cours d’écriture
4. **Sinon** — le premier calque disponible, qui est normalement l’image de base

Les fichiers sidecar `.daq` et `.csv` du projet sont ignorés lors de la navigation à l’aide des touches fléchées ; ainsi, le défilement des images ne s’arrête jamais sur un enregistrement du capteur de lumière.

Le zoom et le panoramique s’appliquent également d’une image à l’autre, ce qui facilite la comparaison avant/après d’une même position sur le champ.

***

## Comprendre les valeurs des pixels par couche

Le [panneau « Valeurs du curseur »](opening-an-image-full-screen.md#cursor-values) indique la valeur réelle par canal sous votre curseur, dans l’unité dans laquelle la couche est enregistrée. Ses colonnes varient en fonction de la couche :

| Couche | Unité indiquée | Remarques |
| --- | --- | --- |
| Base (aperçu JPG / PNG / TIFF) | DN, 0–255 | Valeurs d’affichage, corrigées en gamma sur le RGB. À des fins d’inspection visuelle uniquement |
| RAW (original) | DN | Valeurs numériques brutes du capteur. L’axe de l’histogramme indique la profondeur : 255 (8 bits), 4 095 (12 bits) ou 65 535 (16 bits) |
| RAW (débayérisé) | DN | Linéaire, sans étirement à l’affichage |
| RAW (aperçu) / Balance des blancs | DN | Produit d’affichage — étiré ou corrigé en gamma. Ne convient pas à la mesure |
| RAW (radiance) | **W/m²/sr/nm** | Radiance physique en Float32. Pas de colonne DN |
| RAW (réflectance) | DN **et %** | Pourcentage calculé selon l'échelle propre au fichier — voir ci-dessous |
| Exportations d’index / LUT / sandbox | Valeur d’index, ou composantes RGB | Un fichier d’index monocanal indique la valeur d’index ; un fichier LUT avec mappage des couleurs indique les composantes Red /Green /Blue |

### Réflectance : l’échelle est propre à chaque fichier

{% hint style="warning" %}
**« Diviser par 65 535 » n’est correct que pour Survey3.** La réflectance LATTICE est stockée à une échelle différente, et mélanger les deux diviseurs est le moyen le plus courant d’obtenir des valeurs de réflectance qui correspondent exactement à la moitié de ce qu’elles devraient être.
{% endhint %}

| Source | DN correspondant à une réflectance de 1,0 | Identifié par |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | La balise XMP « `Chloros:PixelScale=32768` » intégrée à chaque exportation de réflectance LATTICE. La marge de 2× permet de représenter des valeurs de ρ supérieures à 1,0 au lieu de les écrêter |
| **Survey3**| **65535** | Pas de balise d'échelle XMP Chloros — l'étalonnage Survey3 enregistre ρ × dtype-max et écrête à 1,0 |

Pour les SIG et les scripts : lisez la valeur `Chloros:PixelScale` dans le fichier et divisez par celle-ci. Si la balise est absente, le fichier est à l’échelle Survey3 (65535). La visionneuse, le bac à sable d’index/LUT et l’exportation d’index interprètent tous l’échelle de cette même manière ; ainsi, la valeur affichée au niveau du curseur correspond à celle utilisée par les calculs de l’index.

Stockage spécifique au format en plus de cette échelle :

* **Le format TIFF** (32 bits, pourcentage) stocke la valeur DN / 65 535 sous forme de nombre à virgule flottante
* **Les formats PNG**(8 bits) et **JPG** (8 bits) stockent la valeur DN × 255 / 65 535
* Un **produit DN 8 bits issu d’une acquisition 8 bits**, ou un fichier de réflectance créé avant la version 1.3.0, est écrêté entre 0 et 255 et ne comporte aucune balise d’échelle. Le panneau affiche uniquement la valeur DN pour ces fichiers, sans colonne « pourcentage »

### Plages de valeurs des indices

| Famille d’indices | Plage typique | Valeur |
| --- | --- | --- |
| Différence normalisée (NDVI, GNDVI, NDRE, ENDVI…) | de −1 à +1 | Végétation saine généralement comprise entre 0,4 et 0,9 ; sol nu proche de 0 ; valeur négative pour l’eau |
| Ajustés au sol (SAVI, OSAVI, MSAVI2…) | environ de −1 à +1,5 | Valeurs similaires à celles du NDVI, mais avec suppression du fond lié au sol |
| Rapports (GRVI, GCI, MSR, CIRE…) | illimités vers le haut | Les rapports augmentent sans limite lorsque la bande du dénominateur tend vers zéro |
| EVI / LAI | de 0 à ~1, de 0 à ~3,5 | Les nuages et autres pixels saturés font sortir ces deux indices de leur plage — il faut d’abord les masquer |

Consultez [Formules des indices multispectraux](../project-settings/multispectral-index-formulas.md) pour connaître la formule exacte derrière chaque préréglage.

***

## Flux de travail courants

### Comparaison avant/après

1. Sélectionnez **RAW (Original)** et notez le vignetage ainsi que les valeurs non calibrées
2. Passez à **RAW (réflectance)**

3. Comparez : le vignettage a disparu, les valeurs sont calibrées. Le zoom et le panoramique restent fixes, vous observez donc la même zone au sol

### Examinez un indice sur l’ensemble d’une série

1. Ouvrez la première image traitée et sélectionnez le calque d’indice
2. Appuyez plusieurs fois sur **→** — le calque d’indice suit votre sélection d’image en image
3. Observez l’histogramme dans la barre latérale au fur et à mesure : une image dont la distribution présente un écart mérite un examen plus approfondi

### Vérification des cibles d’étalonnage

1. Sélectionnez **RAW (Cible)** sur une image cible
2. Vérifiez que la cible est clairement visible et détectée
3. Passez à l’image cible suivante — le calque des cibles suit

### Vérifier l’exactitude des valeurs de réflectance

1. Sélectionnez **RAW (Réflectance)**

2. Lisez la colonne**%** dans le panneau « Valeurs du curseur » — elle est déjà correctement mise à l’échelle pour ce fichier
3. Vérifiez la cohérence par rapport aux matériaux connus présents dans l’image : une végétation saine présente des valeurs élevées en NIR et faibles en rouge ; une cible d’étalonnage devrait afficher une valeur proche de sa réflectance publiée

***

## Dépannage

### Une couche attendue n’apparaît pas dans la liste déroulante

**Causes possibles**

* L’image n’a jamais été traitée — seules les couches « base » et « `RAW (Original)` » existent
* L’option d’exportation du produit n’est pas cochée dans les paramètres du projet
* Le produit ne s’applique pas à cette caméra (radiance et réflectance sur une caméra maître RGB ; tout indice sur une caméra mono M3M à bande unique)
* L’étalonnage de la réflectance n’avait aucune donnée sur laquelle s’appuyer — aucune couverture descendante `.daq` et aucune cible dans l’image ayant passé le contrôle qualité — ; l’image est donc revenue au mode « Vignette Corrected » (correction de la vignette) ou « Sensor Response » (réponse du capteur)

**Que faire ?**

1. Vérifiez le journal de l’exécution : Chloros indique quand un produit d’exportation demandé était impossible à générer et pourquoi
2. Vérifiez les cases à cocher d’exportation par produit dans [Paramètres du projet](../project-settings/project-settings.md)
3. Vérifiez que le dossier du produit existe bien dans l’arborescence de sortie du projet
4. Relancez le traitement en activant le produit

### La liste des couches semble obsolète

Pendant qu’une exécution est en cours, Chloros réanalyse les dossiers de produits du projet et corrige les enregistrements de couches manquants à partir de ce qui se trouve réellement sur le disque ; ainsi, une couche dont l’exportation s’est terminée normalement apparaît d’elle-même lors d’une interrogation. Quitter puis revenir sur l’image force une nouvelle résolution.

### Les valeurs de réflectance semblent être la moitié de ce qu’elles devraient être

Vous divisez très certainement un fichier LATTICE par 65 535. Utilisez `Chloros:PixelScale` (32 768), ou consultez la colonne **%**, qui a déjà appliqué ce facteur.

### La couche d’index existe mais l’image est vierge

L’index nécessite des bandes que votre couche ne possède pas — par exemple, un index qui lit un troisième canal appliqué à un fichier à un ou deux canaux. Passez à une couche multibande (réflectance ou débayérisée), ou choisissez un index adapté au filtre de la caméra.

***

## Étapes suivantes

* [**Ouverture d’une image en plein écran**](opening-an-image-full-screen.md) — affichage du curseur, histogramme et contrôle du GSD
* [**Bac à sable pour indices/LUT**](index-lut-sandbox.md) — visualisation interactive des indices et exportation
* [**Formules d’indices multispectraux**](../project-settings/multispectral-index-formulas.md) — la référence en matière d’indices
* [**Finalisation du traitement**](../processing-images-gui/finishing-the-processing.md) — l’arborescence du dossier de sortie vers laquelle ces calques pointent
