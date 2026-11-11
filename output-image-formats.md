---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formats d&#x27;image de sortie

Chloros exporte les produits traités dans quatre formats de fichier. Sélectionnez le format dans les paramètres du projet (interface graphique), à l&#x27;aide de `--format` (CLI) ou de `export_format` (SDK). La CLI et le SDK acceptent les chaînes de caractères exactes ci-dessous.

| Chaîne de format | Extension | Type de pixel | Plage de pixels | Remarques |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(par défaut)* | `.tif` | nombre binaire uint16 | 0 – 65535 | Recommandé pour la photogrammétrie / les SIG. |
| `TIFF (8-bit)` | `.tif` | nombre numérique uint8 | 0 – 255 | Pour toutes les captures en 8 bits, qui ne proposent que les formats 8 bits : Chloros n&#x27;augmente pas la profondeur de bits. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = réflectance de 100 %. Certaines applications ne peuvent pas lire les fichiers TIFF en virgule flottante ; les fichiers sont plus volumineux. |
| `PNG (8-bit)` | `.png` | nombre numérique uint8 | 0 – 255 | Compression sans perte, adaptée à l&#x27;affichage sur le Web et à la visualisation. |
| `JPG (8-bit)` | `.jpg` | nombre numérique uint8 | 0 – 255 | Compression avec perte, fichiers les plus petits. |

## Emplacement des fichiers de sortie

Les fichiers sont enregistrés dans le dossier du projet, regroupés par caméra puis par format de fichier :

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Le dossier de la caméra est `LATT-<sensor>-<lens>-F<filter>` pour LATTICE et `<model>_<filter>` (par exemple `Survey3N_RGN`) pour Survey3. **Chaque produit exporté conserve le nom du fichier source — c’est le dossier qui identifie le produit, et non un suffixe de nom de fichier.** Consultez la section [Emplacement des fichiers de sortie](reference/cli-reference.md) dans la référence de l’CLI pour connaître l’ensemble des règles.

## Produits LATTICE (niveaux de capture et d’exportation)

Une image brute LATTICE est déclinée en tous les produits demandés en un seul passage. Chaque type de produit dispose de ses propres options (cases à cocher dans l’interface graphique, ou commandes CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, tous activés par défaut) :

| Niveau | Contenu | Type de données |
| --- | --- | --- |
| `raw` | Données Bayer provenant directement du capteur (caméras monochromes : la bande unique). Le traitement part toujours des données brutes. | Telles que capturées |
| `debayered` | Démosaïquage linéaire — 3 canaux pour le M3C, 1 canal en niveaux de gris pour le M3M : le décalage de noir du pack d&#x27;étalonnage est supprimé, puis un démosaïquage tenant compte des contours est effectué, sans correction de linéarité, sans correction de champ plat ni démixage. Une exportation `process` génère le même fichier qu’une capture à ce niveau, quel que soit le mode de traitement. | DN linéaire |
| `radiance` | Radiance spectrale absolue issue de la chaîne radiométrique complète, en **W/m²/sr/nm**. Toujours enregistré au format TIFF 32 bits (`tiff32/Radiance_Images/`), quel que soit le format d’exportation sélectionné. | float32 |
| `reflectance` | Réflectance ρ, où **DN 32768 = ρ 1,0 (100 %)** avec une marge jusqu’à ρ 2,0. Compatible Pix4D. | uint16 |
| `preview` | Rendu prêt à l&#x27;affichage : RGB = balance des blancs + gamma ; multispectral = étirement en fausses couleurs. | affichage 8 bits |

### Orientation

Les fichiers `raw` et `debayered` conservent toujours l’orientation **native** du capteur ; les paramètres de retournement/rotation de la caméra ne s’appliquent qu’aux autres produits (aperçu, radiance, réflectance, indice). Les captures de tableaux conservent l’orientation native de chaque fichier composant, sauf si vous spécifiez une orientation (`--orient` dans l’CLI, `orient=True` dans l’SDK). Chaque fichier enregistre l’orientation propre à ses pixels au format XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — et `ChlorosOrientationApplied` indique si ses propres pixels contiennent cette information (`false` pour les fichiers bruts et débayérisés). Chloros Image Viewer modifie les fichiers bruts et débayérisés en fonction de cet enregistrement à des fins d’affichage uniquement ; les fichiers eux-mêmes ne sont jamais réécrits. Un fichier ne comportant pas cet enregistrement (créé avant la version 1.3.0 ou par un autre outil) s’affiche tel qu’il a été enregistré.

## Masques de limitation

Les produits de radiance et de réflectance d’une caméra multispectrale M3C sont soumis à un démixage par bandes 3×3. Lorsqu’un démixage produit une valeur négative dans une bande, cette valeur est limitée à zéro (les visualisations en direct sont limitées exactement de la même manière). Nouveauté de la version 1.3.0 : un produit dans lequel au moins un pixel a été écrêté est accompagné d’un **masque de limitation** qui indique précisément quels pixels et quelles bandes sont concernés :

* **Où :**dans un dossier `clamp_masks/`**à côté** des dossiers de produits, jamais à l’intérieur de l’un d’entre eux : `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, et `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Quoi :** un fiTIFF `uint8` à bande unique sur la grille de pixels propre au produit (orientation, ROI, binning et alignement de la matrice inclus). Le bit 0 (`mask & 1`) est activé lorsque la bande du canal R du produit a été écrêtée, le bit 1 (`& 2`) le canal G, le bit 2 (`& 4`) le canal B. La proportion de pixels écrêtés par bande est également indiquée sur le produit sous les codes `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` et `Chloros:BandBUnmixClampFraction`.
* **Comment cela est indiqué sur le produit :** la balise XMP `Chloros:UnmixClampMask` contient le chemin d’accès au masque par rapport au dossier du produit (`../clamp_masks/<product folder>/<name>_clampmask.tif`, toujours séparé par `/`), `none` lorsqu’aucun pixel n’a été rogné (aucun masque n’est écrit), ou `unavailable` lorsqu’un masque n’a pas pu être écrit. Un produit sans balise n’a jamais fait l’objet d’un démixage (caméras RGB et monochromes, niveaux bruts et débayérisés).

Comme aucun dossier de produit ne contient de masque, vous pouvez transmettre un dossier de produit tel que `Reflectance_Calibrated_Images/` à Pix4D, Metashape ou tout autre logiciel de photogrammétrie tel quel, sous-dossiers compris — transmettez-lui le dossier de produit, et non le dossier de format situé au-dessus. L’importation propre à Chloros ignore les masques de limitation. L’interface CLI les comptabilise séparément : `Image products written: N (plus M clamp masks)`.

## Produits d’index

Chaque index est placé dans son propre dossier `<INDEX>_Index_Images/` (par exemple, `NDVI_Index_Images/`).

* **Les formats entiers**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) stockent un**étirement linéaire par image**, et non une correspondance −1…1 : DN 0 correspond à la valeur d’index la plus basse de cette image et l’échelle maximale à la plus haute. Le fichier contient les éléments `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` et `Chloros:IndexMax`, ainsi que `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Comme chaque image possède ses propres limites, comparez les images en termes de valeurs, et non en termes de DN.
* **`TIFF (32-bit, Percent)`** stocke les valeurs d’index elles-mêmes sous forme de float32, sans mise à l’échelle — le meilleur choix pour les travaux quantitatifs portant sur l’ensemble d’un vol.

Nouveauté de la version 1.3.0 : un produit d’index se décrit **lui-même** plutôt que de répéter les balises de la source brute. Ses balises GPS et caméra sont copiées à partir du produit à partir duquel il a été calculé (le produit de réflectance lorsqu’il en existe un), puis :

| Balise | Valeur |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (le fichier n’est donc jamais réimporté en tant que capture) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / la formule utilisée |
| `Chloros:IndexLevel` | `reflectance` ou `radiance` — élément à partir duquel l’indice a été calculé (absent si aucun des deux n’est présent) |
| `Chloros:IlluminationInvariant` | `True` pour un indice calculé à partir de la réflectance, `False` à partir de la radiance ou des valeurs du capteur |
| `Chloros:ChlorosPipelineVersion` | la version d’Chloros qui a généré l’indice |

`Chloros:PixelScale` est supprimé des produits d’index — utilisez `IndexMin` / `IndexMax` ci-dessus. La recette complète de relecture se trouve sous « Une grille à indice entier correspond à un étirement du contraste par image » dans la [Référence CLI](reference/cli-reference.md).

## Lecture des valeurs de réflectance des pixels

La réflectance est stockée sous la forme d’un nombre numérique entier (DN), et **le DN correspondant à ρ = 1,0 (réflectance de 100 %) dépend de la caméra source** :

| Caméra source | ρ = 1,0 correspond à DN | Comment le déterminer |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (plage allant jusqu’à ρ 2,0) | La balise XMP `Chloros:PixelScale=32768` est intégrée au fichier. |
| Survey3 | `65535` (limité à ρ 1,0) | Pas de balise `Chloros:PixelScale` — cette absence est le signe distinctif. (Les produits Survey3 comportent d’autres balises de traitement `Chloros:*`, telles que `Chloros:CalibrationApplied`, dans tous les modes de traitement à partir de la version 1.3.0.) |

**Lisez la balise XMP `Chloros:PixelScale` et divisez par sa valeur** plutôt que de supposer une constante. La balise est définie dans le domaine uint16 ; elle reste donc `32768` dans tous les formats de sortie qui effectuent un redimensionnement — normalisez d’abord le type de données stocké en uint16 (×257 à partir de 8 bits, ×65535 à partir de float32).

{% hint style="warning" %}
**Captures 8 bits.** Les données exportées d’une capture 8 bits sont en 8 bits. Sa réflectance est mise à l’échelle et marquée `Chloros:PixelScale=32768`, ce qui permet de conserver l’échelle ; les produits DN 8 bits, ainsi que les fichiers de réflectance créés avant la version 1.3.0, sont écrêtés entre 0 et 255 et ne comportent aucune balise d’échelle. Si la balise est absente d’un fichier de réflectance LATTICE, ne supposez pas qu’il existe une échelle ; réexportez plutôt en 16 bits ou 32 bits.
{% endhint %}

Pour connaître l’ensemble des règles (y compris les balises compatibles avec MicaSense), consultez la section **« Lecture des pixels de réflectance »** dans la [Référence CLI](reference/cli-reference.md).
