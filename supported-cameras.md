---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Caméras prises en charge

Chloros traite les images provenant de deux gammes de caméras MAPIR sur **toutes les plateformes** (Windows, Linux amd64 et Linux arm64/Jetson) :

* **Survey3** — caméras Survey3W (grand angle) et Survey3N (angle étroit). Entrée : `RAW+JPG`.
* **LATTICE**— Modules de caméra multispectrale M3C et M3M. Entrée : captures `.tif`/`.tiff`. Les caméras LATTICE peuvent également être**contrôlées en direct** à partir de Chloros — via l’onglet « Caméras » de l’interface graphique (Windows) ou `chloros-cli lattice` / Python SDK (Windows et Linux) — y compris les réseaux multicaméras synchronisés. Consultez le [guide LATTICE](lattice/).

Le pipeline de traitement accepte également les fichiers d&#x27;entrée `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Fabricant</th><th width="250">Modèle de caméra</th><th width="138">Modèle de filtre</th><th width="187">Type d’image</th></tr></thead><tbody><tr><td><strong>Fabricant</strong></td><td><strong>Modèle de caméra</strong></td><td><strong>Modèle de filtre</strong></td><td><strong>Type d&#x27;image</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

La gamme LATTICE est un système de caméras multispectrales modulaires basé sur le capteur à obturateur global Sony IMX265 (3,1 MP, pixels de 3,45 µm). Chaque caméra stocke son identifiant sous la forme d’une chaîne de caractères :

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros l&#x27;affiche avec le préfixe `LATT-` (par exemple `LATT-M3M-L41-F550`), et cette chaîne de modèle gère tout en aval : le profil du capteur, la disposition des bandes et l&#x27;étalonnage sont déterminés automatiquement ; il n’y a rien à configurer pour chaque caméra. Le numéro d’objectif correspond au **champ de vision horizontal en degrés** : `L41` = étroit 41°, `L87` = large 87°.

Il existe deux configurations de capteur :

| Configuration | Capteur      | Type de filtre                           | Bandes par caméra                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Couleur Bayer | Triple bande passante                       | 3 bandes spectrales à partir d’une seule exposition                                 |
| **M3M**       | Monochrome  | Filtre interférentiel à bande étroite unique | 1 bande étalonnée — combiner plusieurs caméras M3M pour les indices de végétation |

### Options de filtre M3C (Bayer)

| Filtre | Bandes (nom @ centre nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### Catalogue de filtres M3M (mono) — 23 références

Le numéro F correspond à la référence du produit ; la bande mesurée (estampillée sur chaque produit calibré destiné à l’exportation) correspond au scan du filtre par lot :

| Référence    | Centre (nm, mesuré) | FWHM bords (nm) | Largeur (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominal)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominal)           | 867–893         | 26         |
| F905   | — (nominal)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_« Les bords des bandes sont mesurés en largeur à mi-hauteur à partir des balayages par lot de MAPIR — les mêmes valeurs que Chloros appose sur chaque exportation calibrée. »_ « — (nominal) » = aucun balayage par lot n&#x27;a encore été effectué ; pour ces références, le centre indiqué correspond au numéro de référence et la largeur correspond à la valeur fournie par le fabricant.

† « La réflectance F988 est étalonnée à l’aide d’un panneau de réflectance intégré à la scène : la bande se situant au-delà de la plage étalonnée du capteur de lumière DAQ, Chloros applique votre dernière capture du panneau et la conserve entre deux observations du panneau. » Voir [Cibles d’étalonnage](calibration-targets.md).

Pour le contrôle en temps réel de la caméra, les réseaux de capteurs, la configuration réseau et la chaîne de traitement radiométrique, consultez le [guide LATTICE](lattice/).
