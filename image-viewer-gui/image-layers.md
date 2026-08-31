# Calques d&#x27;image

Le **menu déroulant des calques** situé en haut à droite de la visionneuse d&#x27;images vous permet de basculer entre toutes les versions de l&#x27;image que vous consultez — depuis la capture source jusqu&#x27;aux images indexées calculées, en passant par chaque produit traité — sans quitter la visionneuse.

## Que sont les calques d’image ?

Dans Chloros, un « calque » correspond à un **fichier de produit**associé à une image source. L’importation vous fournit les fichiers sources ; le traitement ajoute un calque pour chaque produit généré par la session. Les fichiers exportés conservent le nom du fichier source — c’est le**dossier** qui identifie le produit, et le nom de la couche correspond à l’étiquette attribuée par Chloros à ce dossier.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## La liste des couches

### Toujours présente

| Calque | Description |
| --- | --- |
| **JPG**(ou**PNG**/**TIFF**) | Le fichier de base fourni avec la capture. Survey3 importe un fichier `.JPG` à côté de chaque fichier `.RAW` ; Les captures LATTICE fournissent un aperçu d’affichage PNG ou TIFF. Étiqueté en fonction de ce qui a été réellement importé |
| **RAW (Original)** | L’image brute source, débayérisée pour l’affichage sans aucune correction appliquée. Disponible dès l’importation — ne nécessite aucun traitement |

Une capture LATTICE dont le fichier de base **est** son image brute ne dispose pas d’entrée de base distincte : `RAW (Original)` la couvre déjà.

### Produits de traitement Survey3

| Calque | Enregistré dans | Existe lorsque |
| --- | --- | --- |
| **RAW (Cible)** | — | L&#x27;image a été identifiée comme contenant une cible d&#x27;étalonnage |
| **RAW (Réflectance)** | `Reflectance_Calibrated_Images/` | L&#x27;étalonnage de la réflectance s&#x27;est déroulé avec succès sur cette image |
| **Correction de la vignettage**| `Vignette_Corrected_Images/` | L&#x27;image n&#x27;a pas pu faire l&#x27;objet d&#x27;un étalonnage de réflectance**et** la *correction de la vignettage* était activée |
| **Réponse du capteur**| `Sensor_Response_Images/` | L&#x27;image n&#x27;a pas pu être étalonnée en réflectance**et** la *correction de vignettage* était désactivée |
| **Balance des blancs** | `White_Balanced_Images/` | Un produit avec balance des blancs a été généré |

{% hint style="info" %}
**La correction de vignettage et la réponse du capteur sont des options alternatives, jamais les deux à la fois.** Il existe exactement un produit de secours non calibré par exécution, pour chaque modèle de caméra, et le commutateur *Correction de vignettage* permet de choisir lequel. Voir [Paramètres du projet](../project-settings/project-settings.md).
{% endhint %}

### Niveaux LATTICE

LATTICE capture le fan-out dans ces niveaux en un seul passage de traitement. Ceux qui existent dépendent des options d&#x27;exportation par produit dans les Paramètres du projet et de ce qui s&#x27;applique à l&#x27;appareil photo.

| Calque | Enregistré dans | S&#x27;applique à |
| --- | --- | --- |
| **RAW (débayérisé)** | `Debayered_Images/` | RGB et multispectral |
| **RAW (Aperçu)** | `Preview_Images/` | Multispectraux (étirement en fausses couleurs) |
| **Balance des blancs** | `Preview_Images/` | Caméras principales RGB — l&#x27;aperçu RGB est enregistré sous ce nom afin de correspondre au calque Survey3 du même nom |
| **RAW (radiance)** | `Radiance_Images/` | Multispectral uniquement |
| **RAW (radiance)** | `Reflectance_Calibrated_Images/` | Multispectral uniquement, et uniquement lorsqu&#x27;un enregistrement descendant `.daq` correspondant ou une cible dans le cadre ayant passé le contrôle qualité couvre le cadre |

Les caméras principales RGB ne disposent pas de radiométrie par bande ; par conséquent, la radiance et la réflectance sont ignorées pour celles-ci car **non applicables** — le journal l&#x27;indique clairement plutôt que de signaler une erreur de manière silencieuse.

### Couches d’index, de LUT et de sandbox

| Type de couche | Exemple | Origine |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Une par index configuré dans les paramètres du projet, calculée lors du traitement |
| **`<INDEX>` LUT** | `NDVI LUT` | La version avec mappage des couleurs d’un index |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Un par cycle d&#x27;exportation [Index/LUT Sandbox](index-lut-sandbox.md) |

Si le même nom d’index est configuré plusieurs fois avec des paramètres différents, le deuxième et les suivants sont identifiés par un numéro dans le nom (`RAW (NDVI2 Index)`) afin que les calques restent identifiables.

***

## Utilisation du sélecteur de calques

1. Ouvrez une image en plein écran en cliquant sur une vignette dans la grille
2. Cliquez sur le **menu déroulant des calques** en haut à droite de la visionneuse
3. Sélectionnez un calque — l&#x27;image s&#x27;actualise immédiatement

Le menu déroulant affiche en premier lieu **JPG, RAW (Original), RAW (Cible), RAW (Réflectance)**, dans cet ordre, puis répertorie tous les autres calques dans l&#x27;ordre d&#x27;enregistrement des produits.

### Priorité des couches lors de la navigation

Appuyer sur **←**/**→** permet de passer à l’image suivante tout en essayant de rester sur la même couche :

1. **Correspondance exacte en priorité** — si l’image suivante comporte un calque portant le même nom, c’est celui-ci qui s’affiche. C’est ce qui vous permet de rester sur le calque `RAW (NDVI Index)` lorsque vous parcourez l’ensemble des images
2. **Puis correspondance par type** — un calque d’index recherche n’importe quel calque d’index, une LUT n’importe quelle LUT, une réflectance une réflectance, une cible une cible, un original un original, une base une base
3. **Ensuite, pour les calques d’exportation uniquement** — le nom est conservé même si la liste des calques n’est pas encore à jour, car le fichier existe déjà sur le disque. C’est ce qui vous permet de prévisualiser les produits pendant qu’un traitement est encore en cours d’écriture
4. **Sinon** — le premier calque disponible, qui est normalement l’image de base

Les fichiers sidecar `.daq` et `.csv` du projet sont ignorés lors de la navigation à l’aide des touches fléchées ; ainsi, le défilement des images ne s’arrête jamais sur un enregistrement du capteur de lumière.

Le zoom et le panoramique s’appliquent également d’une image à l’autre, ce qui facilite la comparaison avant/après d’une même position dans le champ.

***

## Comprendre les valeurs des pixels par couche

Le [panneau « Valeurs du curseur »](opening-an-image-full-screen.md#cursor-values) affiche la valeur réelle par canal sous votre curseur, dans l’unité dans laquelle le calque est enregistré. Ses colonnes varient en fonction du calque :

| Calque | Unité affichée | Remarques |
| --- | --- | --- |
| Base (JPG / PNG / aperçu TIFF) | DN, 0–255 | Valeurs d’affichage, corrigées en gamma sur RGB. Inspection visuelle uniquement |
| RAW (Original) | DN | Valeurs numériques brutes du capteur. L’axe de l’histogramme indique la profondeur : 255 (8 bits), 4 095 (12 bits) ou 65 535 (16 bits) |
| RAW (débayérisé) | DN | Linéaire, sans étirement à l’affichage |
| RAW (Aperçu) / Balance des blancs | DN | Produit d’affichage — étiré ou corrigé en gamma. Ne convient pas à la mesure |
| RAW (Radiance) | **W/m²/sr/nm** | Radiance physique en Float32. Pas de colonne DN |
| RAW (réflectance) | DN **et %** | Pourcentage calculé selon l’échelle propre au fichier — voir ci-dessous |
| Exportations d’index / LUT / sandbox | Valeur d’index, ou composantes RGB | Un fichier d’index monocanal indique la valeur d’index ; un fichier LUT avec mappage de couleurs indique les composants Red/Green/Blue |

### Réflectance : l’échelle est propre à chaque fichier

{% hint style="warning" %}
**« Diviser par 65 535 » n’est correct que pour Survey3.** La réflectance LATTICE est stockée à une échelle différente, et mélanger les deux diviseurs est le moyen le plus courant d’obtenir des valeurs de réflectance qui correspondent exactement à la moitié de ce qu’elles devraient être.
{% endhint %}

| Source | DN correspondant à une réflectance de 1,0 | Identifié par |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | La balise XMP `Chloros:PixelScale=32768` intégrée à chaque exportation de réflectance LATTICE. La marge de 2× signifie que les valeurs de ρ supérieures à 1,0 sont représentables plutôt que tronquées |
| **Survey3**|**65535** | En l&#x27;absence de balise d&#x27;échelle XMP Chloros, l&#x27;étalonnage Survey3 écrit ρ × dtype-max et effectue un écrêtage à 1,0 |

Pour les SIG et les scripts : lisez la valeur `Chloros:PixelScale` dans le fichier et divisez par celle-ci. Si la balise est absente, le fichier est à l’échelle Survey3 (65535). La visionneuse, le bac à sable d’index/LUT et l’exportation d’index déterminent tous l’échelle de cette même manière ; ainsi, le nombre affiché au niveau du curseur correspond à celui utilisé par les calculs de l’index.

Stockage spécifique au format en plus de cette échelle :

* **TIFF (32 bits, pourcentage)** stocke DN / 65 535 sous forme de nombre à virgule flottante
* **PNG (8 bits)**et**JPG (8 bits)** stockent DN × 255 / 65 535
* Une **exportation**TIFF**8 bits d’une capture provenant d’une source 8 bits** est écrêtée à la plage 0–255 plutôt que redimensionnée, et ne comporte délibérément aucune balise d’échelle. Le panneau affiche uniquement la valeur DN pour ces fichiers, sans colonne « pourcentage »

### Plages de valeurs d’indice

| Famille d’indices | Plage typique | Lecture |
| --- | --- | --- |
| Différence normalisée (NDVI, GNDVI, NDRE, ENDVI…) | de −1 à +1 | Végétation saine généralement comprise entre 0,4 et 0,9 ; sol nu proche de 0 ; valeur négative pour l’eau |
| Ajustée au sol (SAVI, OSAVI, MSAVI2…) | environ de −1 à +1,5 | Valeur similaire à celle de NDVI, avec suppression du bruit de fond lié au sol |
| Rapport (GRVI, GCI, MSR, CIRE…) | sans limite vers le haut | Les rapports augmentent sans limite à mesure que la bande du dénominateur tend vers zéro |
| EVI / LAI | de 0 à ~1, de 0 à ~3,5 | Les nuages et autres pixels saturés font sortir les deux valeurs de leur plage — il faut d’abord les masquer |

Consultez [Formules des indices multispectraux](../project-settings/multispectral-index-formulas.md) pour connaître la formule exacte derrière chaque préréglage.

***

## Flux de travail courants

### Comparaison avant / après

1. Sélectionnez **RAW (Original)** et notez le vignetage ainsi que les valeurs non calibrées
2. Passez à **RAW (Réflectance)**

3. Comparez : le vignetage a disparu, les valeurs sont calibrées. Le zoom et le panoramique restent fixes, vous observez donc la même zone au sol

### Examiner un indice sur l’ensemble d’une série

1. Ouvrez la première image traitée et sélectionnez le calque d’indice
2. Appuyez plusieurs fois sur **→** — le calque d’indice suit votre navigation d’image en image
3. Observez l’histogramme dans la barre latérale au fur et à mesure : une image dont la distribution présente un écart mérite un examen plus approfondi

### Vérifier les cibles d’étalonnage

1. Sélectionnez **RAW (Cible)** sur une image cible
2. Vérifiez que la cible est clairement visible et détectée
3. Passez à l’image cible suivante — le calque de cibles vous suit

### Vérifier l’exactitude des valeurs de réflectance

1. Sélectionnez **RAW (Réflectance)**

2. Consultez la colonne**%** dans le panneau « Valeurs du curseur » : elle est déjà correctement mise à l’échelle pour ce fichier
3. Vérifiez la cohérence par rapport aux matériaux connus présents dans l’image : une végétation saine présente une valeur élevée en NIR et une faible valeur dans le rouge ; une cible d’étalonnage devrait afficher une valeur proche de sa réflectance publiée

***

## Dépannage

### Un calque attendu n&#x27;apparaît pas dans la liste déroulante

**Causes possibles**

* L&#x27;image n&#x27;a jamais été traitée — seuls le calque de base et le calque `RAW (Original)` existent
* L’option d’exportation du produit n’est pas cochée dans les paramètres du projet
* Le produit ne s’applique pas à cette caméra (radiance et réflectance sur un maître RGB ; tout indice sur une caméra mono M3M à bande unique)
* L’étalonnage de la réflectance n’avait aucune donnée sur laquelle s’appuyer — absence de couverture descendante `.daq` et absence de cible dans l’image ayant passé le contrôle qualité — ; l’image a donc été traitée par défaut avec la correction de vignettage ou la réponse du capteur

**Que faire ?**

1. Vérifier le journal de la session : Chloros indique quand l’exportation d’un produit demandé était impossible et pourquoi
2. Vérifier les options d’exportation par produit dans [Paramètres du projet](../project-settings/project-settings.md)
3. Vérifiez que le dossier du produit existe bien dans l’arborescence de sortie du projet
4. Relancez le traitement en activant le produit

### La liste des couches semble obsolète

Chloros réanalyse les dossiers de produits du projet pendant qu’une exécution est en cours et corrige les enregistrements de couches manquants à partir de ce qui se trouve réellement sur le disque ; ainsi, une couche dont l’exportation s’est terminée normalement apparaît d’elle-même dans un sondage. Quitter puis revenir sur l’image force une nouvelle résolution.

### Les valeurs de réflectance semblent être la moitié de ce qu’elles devraient être

Vous divisez très certainement un fichier LATTICE par 65 535. Utilisez `Chloros:PixelScale` (32 768), ou consultez la colonne **%**, qui a déjà appliqué ce facteur.

### La couche d’index existe mais l’image est vierge

L’index nécessite des bandes que votre couche ne possède pas — par exemple, un index lisant un troisième canal appliqué à un fichier à une ou deux bandes. Passez à une couche multibande (réflectance ou débayérisée), ou choisissez un index adapté au filtre de la caméra.

***

## Étapes suivantes

* [**Ouverture d&#x27;une image en plein écran**](opening-an-image-full-screen.md) — affichage du curseur, histogramme et contrôle du GSD
* [**Bac à sable pour indices/LUT**](index-lut-sandbox.md) — visualisation interactive des indices et exportation
* [**Formules d’indices multispectraux**](../project-settings/multispectral-index-formulas.md) — référence des indices
* [**Finalisation du traitement**](../processing-images-gui/finishing-the-processing.md) — arborescence du dossier de sortie vers lequel ces calques pointent
