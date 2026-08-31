---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formules d&#x27;indices multispectraux

Les formules d&#x27;indices ci-dessous utilisent une combinaison des plages de transmission moyennes du filtre Survey3 :

<table><thead><tr><th align="center">Couleur du filtre Survey3</th><th width="196.199951171875" align="center">Survey3 Nom du filtre</th><th width="159.800048828125" align="center">Plage de transmission (FWHM)</th><th align="center">Transmission moyenne</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Lorsque ces formules sont utilisées, le nom peut se terminer par « \_1 » ou « \_2 », ce qui correspond au filtre utilisé : soit le NIR, soit le NIR1, soit le NIR2.

Pour les caméras LATTICE M3C (triple passe-bande Bayer), le même moteur d’indexation utilise les bandes de filtrage M3C :

| Filtre M3C | Bande 1 (centre/FWHM) | Bande 2 (centre/FWHM) | Bande 3 (centre/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Les caméras LATTICE M3M sont monobandes (un filtre à bande étroite par caméra) ; par conséquent, les indices multibandes ne sont pas calculés pour une image M3M isolée. Pour calculer des indices avec M3M, combinez deux caméras ou plus en une pile multibande alignée et utilisez le moteur d&#x27;indices LATTICE (`chloros-cli lattice index`, ou le calculateur d&#x27;indices en temps réel de l&#x27;interface graphique).

***

## Où chaque nom d’indice fonctionne

Chloros dispose de **trois** surfaces d’indice, et leurs listes prédéfinies ne sont pas identiques. Utilisez cette section pour vérifier si un nom fonctionnera là où vous prévoyez de l’utiliser.

| Où vous vous trouvez | Quelle liste s’applique | Nombre |
| --- | --- | --- |
| Paramètres du projet → Index → Ajouter un index (interface graphique) | Surface 1 | 27 |
| Visionneuse d&#x27;images [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (interface graphique) | Surface 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Surface 2 | 22 |
| SDK `process_folder(indices=[...])` | Surface 2 | 22 |
| `chloros-cli lattice index --preset` | Surface 3 | 22 (un autre 22) |
| Onglet « Caméras » – Calculateur d&#x27;index en temps réel | Surface 3 | 22 (un autre 22) |

Les surfaces 1 et 2 traitent **une image à la fois provenant d&#x27;une seule caméra**, en utilisant les emplacements de symboles `x`/`y`/`z`(/`a`) liés aux canaux de filtre de cette caméra. La surface 3 traite une**pile multibande alignée** — plusieurs caméras LATTICE co-enregistrées dans un seul cube — et fait référence aux canaux par leur nom en minuscules.

### 1. Paramètres du projet dans l’interface graphique / Menu déroulant « Sandbox » de la visionneuse d’images — 27 formules

Le menu déroulant les répertorie dans cet ordre (il s’agit de l’ordre d’insertion, et non de l’ordre alphabétique) :

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

Dans l’interface graphique, vous faites glisser les canaux de filtre de votre caméra vers les emplacements de bande de la formule ; ainsi, n’importe quelle formule peut être utilisée avec n’importe quelle attribution de bande prise en charge par votre caméra. Les formules personnalisées que vous avez enregistrées sont ajoutées à la fin de cette liste.

Les **cinq formules réservées à l’interface graphique** — celles que la liste CLI/SDK `--indices` n&#x27;accepte pas — sont implémentées comme suit :

| Préréglage réservé à l’interface graphique | Formule (telle qu’implémentée) | Emplacements |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (quatre emplacements) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Le mappage prévu pour chacun d’entre eux est indiqué dans une section distincte plus bas sur cette page (par exemple, GARI correspond à x=Green, y = NIR, z = Blue, a = Red). GARI est la seule formule de Chloros qui utilise un quatrième emplacement.

### 2. CLI / SDK : extension de nom `--indices` — 22 préréglages

L&#x27;option `chloros-cli process --indices` (et le paramètre SDK `indices`) accepte les noms de préréglages suivants :

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Les noms d&#x27;index inconnus sont ignorés sans message.** Tout nom ne figurant pas dans cette liste (y compris les cinq formules réservées à l’interface graphique `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`, ainsi que toute formule personnalisée que vous avez enregistrée dans l&#x27;interface graphique) est ignoré avec uniquement une notification dans le journal — l&#x27;exécution se poursuit sans cet index, et l&#x27;exécution elle-même est tout de même considérée comme réussie. La notification s&#x27;affiche comme suit :

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

La correspondance des noms s’effectue sans distinction de majuscules/minuscules après suppression des espaces, de sorte que `ndvi`, `NDVI` et ` NDVI ` correspondent au même préréglage. Un préréglage est également ignoré s’il nécessite une bande que le filtre de votre appareil photo ne fournit pas.
{% endhint %}

Formules exactes telles qu’elles sont implémentées (les symboles `x`/`y`/`z` correspondent aux emplacements de bandes ; le mappage par défaut est indiqué pour chaque préréglage) :

| Préréglage | Formule (telle qu&#x27;implémentée) | Filtre par défaut | Emplacements (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Comment un nom de préréglage se traduit en positions de bande

Lorsque vous transmettez un nom nu tel que `NDVI`, Chloros doit déterminer quel canal de quel fichier chaque symbole lit. Il utilise ce tableau, qui associe un code de filtre à la position dans le tableau de chaque canal :

| Code de filtre | Canal → index du tableau |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` est accepté comme alias de Orange, également 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Le **filtre par défaut** du préréglage (colonne « Filtre par défaut » ci-dessus) est utilisé lorsque le projet contient des images dotées de ce filtre. Si ce n’est pas le cas, Chloros parcourt les filtres effectivement présents dans le projet dans l’ordre `RGN, OCN, NGB, RGB, RE, NIR` et sélectionne le premier capable de fournir tous les canaux requis par le préréglage. Si aucun ne le peut, le préréglage est ignoré pour cette exécution. C&#x27;est pourquoi le jeu de données `NDVI` demandé sur un jeu de données OCNproduit toujours un résultat plausible : il se lie aux positions Orange et NIR de OCN.

Les chaînes de modèle LATTICE M3C comportent le filtre avec un préfixe `F` (`LATT-M3C-L41-FRGN`), mais ce préfixe est supprimé lorsque le code de filtre est lu à partir de l’image ; une caméra FRGN effectue donc la résolution via la ligne `RGN` située au-dessus et ne nécessite aucun traitement particulier.

### 3. Moteur d’indexation LATTICE (`lattice index --preset`, calculateur d’index en temps réel) — 22 préréglages

Le moteur LATTICE fonctionne sur des piles multibandes alignées (matrices en temps réel ou fichiers TIFF multibandes exportés) et utilise des noms de canaux en minuscules (`red`, `green`, `blue`, `red_edge`, `nir`). Sa liste de préréglages diffère des deux précédentes :

| Préréglage | Formule | Canaux |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rouge, NIR |
| GNDVI | `(nir - green) / (nir + green)` | vert, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | bleu, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rouge\_bord, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | bleu, vert, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | rouge, infrarouge |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | rouge, infrarouge |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rouge, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | bleu, rouge, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rouge, NIR |
| CVI | `(nir / green) - (red / green)` | rouge, vert, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rouge, NIR |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | rouge, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | rouge, vert, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rouge, vert, bleu |
| NGRDI | `(green - red) / (green + red)` | rouge, vert |
| VARI | `(green - red) / (green + red - blue)` | rouge, vert, bleu |
| TGI | `green - 0.39*red - 0.61*blue` | rouge, vert, bleu |
| EXG | `2*green - red - blue` | rouge, vert, bleu |
| CIRE | `(nir / red_edge) - 1` | rouge\_bord, infrarouge lointain |
| CIGREEN | `(nir / green) - 1` | vert, infrarouge lointain |
| NDWI | `(green - nir) / (green + nir)` | vert, infrarouge lointain |

Exécutez `chloros-cli lattice index --list-presets` pour imprimer ce tableau à partir de votre version installée, et `--list-gradients` pour les dégradés de couleurs disponibles. Les symboles de canal sont sensibles à la casse et doivent correspondre aux noms en minuscules des préréglages (par exemple : `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Tel qu’implémenté dans l’interface graphique et dans la liste des préréglages CLI/SDK, CVI correspond à la formule du « rapport des rapports » :

$$
CVI = {(z / y) \over (x / y)}
$$

avec le mappage de canaux par défaut RGB : x = Red, y = Green, z = Blue. Dans l&#x27;interface graphique, vous pouvez faire glisser n&#x27;importe quel canal de votre caméra vers les emplacements x/y/z. Notez que le préréglage `CVI` du moteur d’indice LATTICE utilise une formule différente, `(NIR / Green) - (Red / Green)` — consultez les tableaux ci-dessus pour la surface que vous utilisez.

***

## ENDVI - Indice de végétation par différence normalisée amélioré

Cet indice utilise le canal bleu en plus des canaux NIR et vert, et est couramment utilisé avec les caméras filtrées NGB, où la bande bleue remplace la bande rouge.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

La mise en œuvre correspond à la formule symbolique `((x+y)-(2*z))/((x+y)+(2*z))` — attribuez lescanaux NIR et Green aux emplacements x/y et Blue à l’emplacement z (pour une caméra NGB : x=NIR, y=Green, z=Blue).

***

## EVI - Indice de végétation amélioré

Cet indice a été initialement développé pour être utilisé avec les données MODIS afin d’apporter une amélioration par rapport à NDVI en optimisant le signal de végétation dans les zones présentant un indice de surface foliaire élevé (LAI). Il est particulièrement utile dans les régions où l’indice LAI est élevé et où l’indice NDVI peut présenter une saturation. Il utilise la bande de réflectance bleue pour corriger les signaux de fond liés au sol et pour réduire les influences atmosphériques, notamment la diffusion des aérosols.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Les valeurs de EVI doivent être comprises entre 0 et 1 pour les pixels de végétation. Les éléments clairs tels que les nuages et les bâtiments blancs, ainsi que les éléments sombres comme l’eau, peuvent entraîner des valeurs de pixels anormales dans une image EVI. Avant de créer une image EVI, il convient de masquer les nuages et les éléments clairs de l’image de réflectance, et, si vous le souhaitez, d’appliquer un seuil aux valeurs des pixels comprises entre 0 et 1.

_Référence : Huete, A., et al. « Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices ». Remote Sensing of Environment 83 (2002) : 195–213._

***

## FCI1 - Indice de couverture forestière 1

_Uniquement via l’interface graphique — non disponible en tant que préréglage CLI/SDK `--indices`._

Cet indice distingue le couvert forestier des autres types de végétation à l’aide d’images de réflectance multispectrales comprenant une bande « red edge ».

$$
FCI1 = Red * RedEdge
$$

Les zones forestières présenteront des valeurs FCI1 plus faibles en raison de la réflectance plus faible des arbres et de la présence d’ombres au sein du couvert forestier.

_Référence : Becker, Sarah J., Craig S.T. Daughtry et Andrew L. Russ. « Robust forest cover indices for multispectral images ». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018) : 505-512._

***

## FCI2 - Indice de couverture forestière 2

_Interface graphique uniquement — non disponible en tant que préréglage CLI/SDK `--indices`._

Cet indice distingue le couvert forestier des autres types de végétation à l’aide d’images de réflectance multispectrales qui n’incluent pas de bande « red edge ».

$$
FCI2 = Red * NIR
$$

Les zones forestières présenteront des valeurs FCI2 plus faibles en raison de la réflectance plus faible des arbres et de la présence d’ombres au sein de la canopée.

_Référence : Becker, Sarah J., Craig S.T. Daughtry et Andrew L. Russ. « Robust forest cover indices for multispectral images ». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018) : 505-512._

***

## GEMI - Indice de surveillance environnementale mondiale

_Uniquement en interface graphique — non disponible en préréglage CLI/SDK `--indices`._

Cet indice de végétation non linéaire est utilisé pour la surveillance environnementale mondiale à partir d’images satellites et vise à corriger les effets atmosphériques. Il est similaire à NDVI, mais est moins sensible aux effets atmosphériques. Il est influencé par le sol nu ; par conséquent, son utilisation n’est pas recommandée dans les zones à végétation clairsemée ou modérément dense.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Où :

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Référence : Pinty, B., et M. Verstraete. GEMI : un indice non linéaire pour la surveillance de la végétation mondiale par satellite. Vegetation 101 (1992) : 15-20._

***

## GARI - Green : indice résistant aux variations atmosphériques

_Uniquement via l&#x27;interface graphique — non disponible en tant que préréglage CLI/SDK `--indices`._

Cet indice est plus sensible à une large gamme de concentrations de chlorophylle et moins sensible aux effets atmosphériques que l&#x27;indice NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La constante gamma est une fonction de pondération qui dépend des conditions des aérosols dans l&#x27;atmosphère. ENVI utilise une valeur de 1,7, qui est la valeur recommandée par Gitelson, Kaufman et Merzylak (1996, page 296).

_Référence : Gitelson, A., Y. Kaufman et M. Merzylak. « Utilisation d’un canal Green dans la télédétection de la végétation mondiale à partir d’EOS-MODIS. » Remote Sensing of Environment 58 (1996) : 289-298._

***

## GCI - Green Indice de chlorophylle

Cet indice est utilisé pour estimer la teneur en chlorophylle des feuilles chez un large éventail d&#x27;espèces végétales.

$$
GCI = {NIR \over Green} - 1
$$

L’utilisation de larges bandes de longueurs d’onde NIR et vertes permet une meilleure prédiction de la teneur en chlorophylle tout en offrant une plus grande sensibilité et un rapport signal/bruit plus élevé.

_Référence : Gitelson, A., Y. Gritz et M. Merzlyak. « Relations entre la teneur en chlorophylle des feuilles et la réflectance spectrale, et algorithmes pour l&#x27;évaluation non destructive de la chlorophylle dans les feuilles des plantes supérieures ». Journal of Plant Physiology 160 (2003) : 271-282._

***

## GLI - Green Indice foliaire

Cet indice a été initialement conçu pour être utilisé avec une caméra numérique RGB afin de mesurer la couverture de blé, les valeurs numériques (DN) rouges, vertes et bleues étant comprises entre 0 et 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Les valeurs GLI varient de -1 à +1. Les valeurs négatives correspondent au sol et aux éléments non vivants, tandis que les valeurs positives correspondent aux feuilles et aux tiges vertes.

_Référence : Louhaichi, M., M. Borman et D. Johnson. « Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat ». Geocarto International 16, n° 1 (2001) : 65-70._

***

## GNDVI - Green Indice de végétation par différence normalisée

Cet indice est similaire à NDVI, à la différence qu’il mesure le spectre vert de 540 à 570 nm au lieu du spectre rouge. Cet indice est plus sensible à la concentration en chlorophylle que NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Référence : Gitelson, A., et M. Merzlyak. « Télédétection de la concentration en chlorophylle dans les feuilles des plantes supérieures ». Advances in Space Research 22 (1998) : 689-692._

***

## GOSAVI - Green Indice de végétation optimisé et ajusté au sol

Cet indice a été initialement conçu à partir de photographies en infrarouge couleur afin de prédire les besoins en azote du maïs. Il est similaire à l&#x27;indice OSAVI, mais remplace la bande verte par la bande rouge.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Référence : Sripada, R., et al. « Détermination des besoins en azote en cours de saison pour le maïs à l’aide de la photographie aérienne couleur-infrarouge ». Thèse de doctorat, Université d’État de Caroline du Nord, 2005._

***

## Indice de végétation GRVI - Green

Cet indice est sensible aux taux de photosynthèse dans les canopées forestières, car les réflectances vertes et rouges sont fortement influencées par les variations des pigments foliaires.

$$
GRVI = {NIR \over Green }
$$

_Référence : Sripada, R., et al. « Photographie aérienne en couleur et infrarouge pour déterminer les besoins en azote précoces du maïs en début de saison ». Agronomy Journal 98 (2006) : 968-977._

***

## GSAVI - Green Indice de végétation ajusté en fonction du sol

Cet indice a été initialement conçu à partir de la photographie couleur-infrarouge pour prédire les besoins en azote du maïs. Il est similaire à l’indice SAVI, mais remplace la bande verte par la bande rouge.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Référence : Sripada, R., et al. « Détermination des besoins en azote en cours de saison pour le maïs à l&#x27;aide de la photographie aérienne en infrarouge couleur ». Thèse de doctorat, Université d&#x27;État de Caroline du Nord, 2005._

***

## LAI - Indice de surface foliaire

Cet indice est utilisé pour estimer le couvert végétal et pour prévoir la croissance et le rendement des cultures. ENVI calcule l’indice vert LAI à l’aide de la formule empirique suivante, tirée de Boegh et al. (2002) :

$$
LAI = 3.618 * EVI - 0.118
$$

Où EVI est :

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Les valeurs élevées de LAI varient généralement entre environ 0 et 3,5. Cependant, lorsque la scène contient des nuages et d’autres éléments clairs produisant des pixels saturés, les valeurs de LAI peuvent dépasser 3,5. Idéalement, vous devriez masquer les nuages et les éléments clairs de votre scène avant de créer une image LAI.

_Référence : Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde et A. Thomsen. « Données multispectrales aériennes pour la quantification de l&#x27;indice de surface foliaire, de la concentration en azote et de l&#x27;efficacité photosynthétique en agriculture.» Remote Sensing of Environment 81, n° 2-3 (2002) : 179-193._

***

## LCI - Indice de chlorophylle foliaire

_Uniquement via l&#x27;interface graphique (GUI) — non disponible en tant que préréglage CLI/SDK `--indices`._

Cet indice est utilisé pour estimer la teneur en chlorophylle des plantes supérieures ; il est sensible aux variations de réflectance causées par l&#x27;absorption de la chlorophylle.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Référence : Datt, B. « Remote Sensing of Water Content in Eucalyptus Leaves ». Journal of Plant Physiology 154, n° 1 (1999) : 30-36._

***

## MNLI - Indice non linéaire modifié

Cet indice est une amélioration de l’indice non linéaire (NLI) qui intègre l’indice de végétation ajusté au sol (SAVI) afin de tenir compte du fond du sol. ENVI utilise une valeur de facteur d’ajustement du fond du couvert végétal (_L_) égale à 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Référence : Yang, Z., P. Willis et R. Mueller. « Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy ». Actes du symposium Pecora 17 sur la télédétection (2008), Denver, CO._

***

## MSAVI2 - Indice de végétation ajusté au sol modifié 2

Cet indice est une version simplifiée de l&#x27;indice MSAVI proposé par Qi et al. (1994), qui constitue une amélioration de l’indice de végétation ajusté au sol (SAVI). Il réduit le bruit lié au sol et augmente la plage dynamique du signal de végétation. Le MSAVI2 repose sur une méthode inductive qui n&#x27;utilise pas de valeur _L_ constante (comme c&#x27;est le cas avec SAVI) pour mettre en évidence la végétation saine.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Référence : Qi, J., A. Chehbouni, A. Huete, Y. Kerr et S. Sorooshian. « A Modified Soil Adjusted Vegetation Index ». Remote Sensing of Environment 48 (1994) : 119-126._

***

## MSR - Rapport simple modifié

Cet indice est une modification du rapport simple NIR/Red, conçu pour linéariser sa relation avec les paramètres biophysiques ; il est plus sensible que le rapport NDVI lorsque la densité de végétation est élevée.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Référence : Chen, J. « Évaluation des indices de végétation et d&#x27;un rapport simple modifié pour des applications boréales ». Canadian Journal of Remote Sensing 22 (1996) : 229-242._

***

## NDRE - Différence normalisée RedEdge

Cet indice est similaire à NDVI, mais compare le contraste entre NIR et RedEdge au lieu de Red, qui détecte souvent plus tôt le stress de la végétation.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indice de végétation par différence normalisée

Cet indice mesure la santé de la végétation verte. La combinaison de sa formulation par différence normalisée et de l’utilisation des régions d’absorption et de réflectance maximales de la chlorophylle lui confère une grande robustesse dans un large éventail de conditions. Il peut toutefois atteindre la saturation dans des conditions de végétation dense lorsque la valeur de LAI devient élevée.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

La valeur de cet indice varie de -1 à 1. La plage habituelle pour la végétation verte est comprise entre 0,2 et 0,8.

_Référence : Rouse, J., R. Haas, J. Schell et D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Troisième symposium ERTS, NASA (1973) : 309-317._

***

## NLI - Indice non linéaire

Cet indice part du principe que la relation entre de nombreux indices de végétation et les paramètres biophysiques de surface est non linéaire. Il linéarise les relations avec les paramètres de surface qui ont tendance à être non linéaires.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Référence : Goel, N., et W. Qin. « Influences de l’architecture du couvert végétal sur les relations entre divers indices de végétation et LAI et Fpar : une simulation informatique. » Remote Sensing Reviews 10 (1994) : 309-347._

***

## OSAVI - Indice de végétation ajusté au sol optimisé

Cet indice est basé sur l’indice de végétation ajusté en fonction du sol (SAVI). Il utilise une valeur standard de 0,16 pour le facteur d’ajustement du fond de la canopée. Rondeaux (1996) a déterminé que cette valeur permettait de mieux rendre compte des variations du sol que l’indice SAVI en cas de faible couverture végétale, tout en présentant une sensibilité accrue lorsque la couverture végétale est supérieure à 50 %. Cet indice est particulièrement adapté aux zones à végétation relativement clairsemée où le sol est visible à travers la canopée.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Référence : Rondeaux, G., M. Steven et F. Baret. « Optimization of Soil-Adjusted Vegetation Indices ». Remote Sensing of Environment 55 (1996) : 95-107._

***

## RDVI - Indice de végétation par différence renormalisée

Cet indice utilise la différence entre les longueurs d’onde du proche infrarouge et du rouge, en association avec l’NDVI, pour mettre en évidence une végétation saine. Il est insensible aux effets du sol et à la géométrie d’observation du soleil.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Référence : Roujean, J., et F. Breon. « Estimation du PAR absorbé par la végétation à partir de mesures de réflectance bidirectionnelles ». Remote Sensing of Environment 51 (1995) : 375-384._

***

## SAVI - Indice de végétation corrigé du sol

Cet indice est similaire à NDVI, mais il supprime les effets des pixels de sol. Il utilise un facteur d’ajustement de l’arrière-plan du couvert végétal, _L_, qui est fonction de la densité de la végétation et nécessite souvent une connaissance préalable de la quantité de végétation. Huete (1988) suggère une valeur optimale de _L_ = 0,5 pour tenir compte des variations de fond du sol de premier ordre. Cet indice est particulièrement adapté aux zones à végétation relativement clairsemée où le sol est visible à travers la canopée.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Référence : Huete, A. « A Soil-Adjusted Vegetation Index (SAVI) ». Remote Sensing of Environment 25 (1988) : 295-309._

***

## TDVI - Indice de végétation par différence transformée

Cet indice est utile pour surveiller le couvert végétal en milieu urbain. Il ne présente pas de saturation comme les indices NDVI et SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Référence : Bannari, A., H. Asalhi et P. Teillet. « Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping » dans les Actes du Symposium sur les géosciences et la télédétection, IGARSS &#x27;02, IEEE International, volume 5 (2002)._

***

## VARI - Indice visible résistant aux effets atmosphériques

Cet indice est basé sur l’indice ARVI et sert à estimer la fraction de végétation dans une scène avec une faible sensibilité aux effets atmosphériques.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Référence : Gitelson, A., et al. « Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction ». International Journal of Remote Sensing 23 (2002) : 2537−2562._

***

## WDRVI - Indice de végétation à large plage dynamique

Cet indice est similaire à NDVI, mais il utilise un coefficient de pondération (_a_) afin de réduire la disparité entre les contributions des signaux du proche infrarouge et du rouge à l’indice NDVI. L’indice WDRVI est particulièrement efficace dans les scènes présentant une densité de végétation modérée à élevée lorsque l’indice NDVI dépasse 0,6. L’indice NDVI a tendance à se stabiliser lorsque la fraction de végétation et l’indice de surface foliaire (LAI) augmentent, tandis que l’WDRVI est plus sensible à une plus large gamme de fractions de végétation et aux variations de l’LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Le coefficient de pondération (_a_) peut varier entre 0,1 et 0,2. Une valeur de 0,2 est recommandée par Henebry, Viña et Gitelson (2004).

_Références_

_Gitelson, A. « Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation ». Journal of Plant Physiology 161, n° 2 (2004) : 165-173._

_Henebry, G., A. Viña et A. Gitelson. « L’indice de végétation à large plage dynamique et son utilité potentielle pour l’analyse des lacunes ». Gap Analysis Bulletin 12 : 50-56._
