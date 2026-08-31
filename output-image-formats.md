---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formats d&#x27;image de sortie

Chloros exporte les produits traités dans quatre formats de fichier. Sélectionnez le format dans les paramètres du projet (interface graphique), à l’aide de `--format` (CLI) ou de `export_format` (SDK). Les options CLI et SDK acceptent les chaînes de caractères exactes indiquées ci-dessous.

| Chaîne de format | Extension | Type de pixel | Plage de pixels | Remarques |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(par défaut)* | `.tif` | nombre numérique uint16 | 0 – 65 535 | Recommandé pour la photogrammétrie / les SIG. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = réflectance de 100 %. Certaines applications ne peuvent pas lire les fichiers TIFF à virgule flottante ; les fichiers sont plus volumineux. |
| `PNG (8-bit)` | `.png` | nombre binaire uint8 | 0 – 255 | Compression sans perte, adaptée à l’affichage sur le Web et à la visualisation. |
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

Le dossier de la caméra est `LATT-<sensor>-<lens>-F<filter>` pour LATTICE et `<model>_<filter>` (par exemple `Survey3N_RGN`) pour Survey3. **Chaque produit exporté conserve le nom du fichier source — c’est le dossier qui identifie le produit, et non un suffixe de nom de fichier.** Consultez la section [Emplacement des fichiers de sortie](reference/cli-reference.md) dans la référence CLI pour connaître l’ensemble des règles.

## Produits LATTICE (niveaux de capture et d’exportation)

Une image brute LATTICE est répartie en tous les produits demandés en un seul passage. Chaque type de produit dispose de son propre commutateur (cases à cocher de l’interface graphique, ou CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, tous activés par défaut) :

| Niveau | Contenu | Type de données |
| --- | --- | --- |
| `raw` | Données Bayer provenant directement du capteur (caméras monochromes : la bande unique). Le traitement part toujours des données brutes. | Telles que capturées |
| `debayered` | Démosaïquage linéaire — 3 canaux pour M3C, 1 canal en niveaux de gris pour M3M. | DN linéaire |
| `radiance` | Radiance spectrale absolue issue de la chaîne radiométrique complète, en **W/m²/sr/nm**. Toujours enregistrée au format 32 bits TIFF (`tiff32/Radiance_Images/`), quel que soit le format d’exportation sélectionné. | float32 |
| `reflectance` | Réflectance ρ, où **DN 32768 = ρ 1,0 (100 %)** avec une marge jusqu’à ρ 2,0. Compatible Pix4D. | uint16 |
| `preview` | Rendu prêt à l’affichage : RGB = balance des blancs + gamma ; multispectral = étirement en fausses couleurs. | affichage 8 bits |

## Lecture des valeurs de réflectance des pixels

La réflectance est stockée sous la forme d’un nombre numérique entier, et **le DN correspondant à ρ = 1,0 (réflectance de 100 %) dépend de la caméra source** :

| Caméra source | ρ = 1,0 correspond à DN | Comment le déterminer |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (marge jusqu’à ρ 2,0) | La balise XMP `Chloros:PixelScale=32768` est apposée sur le fichier. |
| Survey3 | `65535` (limité à ρ 1,0) | Aucune balise XMP `Chloros:*` — cette absence est le signe distinctif. |

**Lisez la balise XMP `Chloros:PixelScale` et divisez par cette valeur** plutôt que de supposer une constante. La balise est définie dans le domaine uint16 ; elle reste donc inchangée (`32768`) quel que soit le format de sortie qui effectue une mise à l&#x27;échelle — normalisez d&#x27;abord le type de données stocké en uint16 (×257 à partir de 8 bits, ×65535 à partir de float32).

{% hint style="warning" %}
**Un cas particulier ne comporte pas d’échelle, par conception.** Lorsqu’une capture provenant d’une source 8 bits (BayerRG8) est enregistrée au format 8 bits TIFF, le pipeline écrête la valeur entre 0 et 255 au lieu de la redimensionner ; le fichier n’est donc associé à aucune échelle — Chloros omet délibérément `Chloros:PixelScale` dans ce cas. Si la balise est absente d’un fichier de réflectance LATTICE, ne présumez pas d’une échelle ; réexportez plutôt en 16 bits ou 32 bits.
{% endhint %}

Pour connaître l’ensemble des règles (y compris les balises compatibles avec MicaSense), consultez la section **« Lecture des pixels de réflectance »** dans la [Référence CLI](reference/cli-reference.md).
