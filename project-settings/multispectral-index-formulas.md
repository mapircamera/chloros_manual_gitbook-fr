---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formules d&#x27;indices multispectraux

Les formules d&#x27;indices ci-dessous utilisent une combinaison des plages de transmission moyenne du filtre Survey3 :

<table><thead><tr><th align="center">Couleur du filtre Survey3</th><th width="196.199951171875" align="center">Survey3 Nom du filtre</th><th width="159.800048828125" align="center">Plage de transmission (FWHM)</th><th align="center">Transmission moyenne</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Lorsque ces formules sont utilisées, le nom peut se terminer par « \_1 » ou « \_2 », ce qui correspond au filtre NIR utilisé, à savoir NIR1 ou NIR2.

***

## EVI - Indice de végétation amélioré

Cet indice a été initialement développé pour être utilisé avec les données MODIS afin d&#x27;améliorer le NDVI en optimisant le signal de végétation dans les zones à indice de surface foliaire élevé (LAI). Il est particulièrement utile dans les régions à forte valeur LAI où NDVI peut saturer. Il utilise la région de réflectance bleue pour corriger les signaux de fond du sol et réduire les influences atmosphériques, y compris la diffusion des aérosols.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Les valeurs de EVI doivent être comprises entre 0 et 1 pour les pixels de végétation. Les éléments clairs tels que les nuages et les bâtiments blancs, ainsi que les éléments sombres tels que l&#x27;eau, peuvent entraîner des valeurs de pixels anormales dans une image EVI. Avant de créer une image EVI, vous devez masquer les nuages et les éléments clairs de l&#x27;image de réflectance, et éventuellement appliquer un seuil aux valeurs de pixels comprises entre 0 et 1.

_Référence : Huete, A., et al. « Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices ». Remote Sensing of Environment 83 (2002) : 195–213._

***

## FCI1 - Indice de couverture forestière 1

Cet indice distingue la canopée forestière des autres types de végétation à l&#x27;aide d&#x27;images de réflectance multispectrale incluant une bande « red edge ».

$$
FCI1 = Red * RedEdge
$$

Les zones forestières présenteront des valeurs FCI1 plus faibles en raison de la réflectance plus faible des arbres et de la présence d&#x27;ombres au sein de la canopée.

_Référence : Becker, Sarah J., Craig S.T. Daughtry et Andrew L. Russ. « Robust forest cover indices for multispectral images ». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018) : 505-512._

***

## FCI2 - Indice de couverture forestière 2

Cet indice distingue les canopées forestières des autres types de végétation à l&#x27;aide d&#x27;images de réflectance multispectrale qui n&#x27;incluent pas de bande « red edge ».

$$
FCI2 = Red * NIR
$$

Les zones forestières auront des valeurs FCI2 plus faibles en raison de la réflectance plus faible des arbres et de la présence d&#x27;ombres au sein de la canopée.

_Référence : Becker, Sarah J., Craig S.T. Daughtry et Andrew L. Russ. « Robust forest cover indices for multispectral images ». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018) : 505-512._

***

## GEMI - Indice de surveillance environnementale mondiale

Cet indice de végétation non linéaire est utilisé pour la surveillance environnementale mondiale à partir d&#x27;images satellitaires et vise à corriger les effets atmosphériques. Il est similaire à NDVI mais est moins sensible aux effets atmosphériques. Il est influencé par le sol nu ; par conséquent, son utilisation n&#x27;est pas recommandée dans les zones à végétation clairsemée ou modérément dense.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Où :

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Référence : Pinty, B., et M. Verstraete. GEMI : un indice non linéaire pour surveiller la végétation mondiale à partir de satellites. Vegetation 101 (1992) : 15-20._

***

## GARI - Green Indice résistant aux effets atmosphériques

Cet indice est plus sensible à une large gamme de concentrations de chlorophylle et moins sensible aux effets atmosphériques que l&#x27;indice NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La constante gamma est une fonction de pondération qui dépend des conditions des aérosols dans l&#x27;atmosphère. ENVI utilise une valeur de 1,7, qui est la valeur recommandée par Gitelson, Kaufman et Merzylak (1996, page 296).

_Référence : Gitelson, A., Y. Kaufman et M. Merzylak. « Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS ». Remote Sensing of Environment 58 (1996) : 289-298._

***

## GCI - Green Indice de chlorophylle

Cet indice est utilisé pour estimer la teneur en chlorophylle des feuilles chez un large éventail d&#x27;espèces végétales.

$$
GCI = {NIR \over Green} - 1
$$

L&#x27;utilisation de larges bandes de longueurs d&#x27;onde vertes permet une meilleure prédiction de la teneur en chlorophylle tout en offrant une plus grande sensibilité et un rapport signal/bruit plus élevé.

_Référence : Gitelson, A., Y. Gritz et M. Merzlyak. « Relations entre la teneur en chlorophylle des feuilles et la réflectance spectrale, et algorithmes pour l&#x27;évaluation non destructive de la chlorophylle dans les feuilles des plantes supérieures ». Journal of Plant Physiology 160 (2003) : 271-282._

***

## Indice foliaire GLI - Green

Cet indice a été initialement conçu pour être utilisé avec une caméra numérique RGB afin de mesurer la couverture de blé, où les nombres numériques (DN) rouges, verts et bleus varient de 0 à 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Les valeurs GLI varient de -1 à +1. Les valeurs négatives représentent le sol et les éléments non vivants, tandis que les valeurs positives représentent les feuilles et les tiges vertes.

_Référence : Louhaichi, M., M. Borman et D. Johnson. « Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat. » Geocarto International 16, n° 1 (2001) : 65-70._

***

## GNDVI - Green Indice de végétation par différence normalisée

Cet indice est similaire à NDVI, à l&#x27;exception qu&#x27;il mesure le spectre vert de 540 à 570 nm au lieu du spectre rouge. Cet indice est plus sensible à la concentration en chlorophylle que NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Référence : Gitelson, A., et M. Merzlyak. « Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves ». Advances in Space Research 22 (1998) : 689-692._

***

## GOSAVI - Green Indice de végétation optimisé et ajusté au sol

Cet indice a été initialement conçu à partir de photographies en infrarouge couleur afin de prédire les besoins en azote du maïs. Il est similaire à OSAVI, mais remplace la bande verte par la bande rouge.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Référence : Sripada, R., et al. « Détermination des besoins en azote en cours de saison pour le maïs à l&#x27;aide de la photographie aérienne en infrarouge couleur ». Thèse de doctorat, Université d&#x27;État de Caroline du Nord, 2005._

***

## GRVI - Indice de végétation basé sur le rapport Green

Cet indice est sensible aux taux de photosynthèse dans les canopées forestières, car les réflectances verte et rouge sont fortement influencées par les variations des pigments foliaires.

$$
GRVI = {NIR \over Green }
$$

_Référence : Sripada, R., et al. « Photographie aérienne couleur-infrarouge pour déterminer les besoins en azote en début de saison du maïs ». Agronomy Journal 98 (2006) : 968-977._

***

## GSAVI - Green Indice de végétation ajusté au sol

Cet indice a été initialement conçu à partir de la photographie couleur-infrarouge pour prédire les besoins en azote du maïs. Il est similaire à SAVI, mais remplace la bande verte par la bande rouge.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Référence : Sripada, R., et al. « Détermination des besoins en azote en cours de saison pour le maïs à l&#x27;aide de la photographie aérienne couleur-infrarouge ». Thèse de doctorat, Université d&#x27;État de Caroline du Nord, 2005._

***

## LAI - Indice de surface foliaire

Cet indice est utilisé pour estimer la couverture foliaire et pour prévoir la croissance et le rendement des cultures. ENVI calcule l&#x27;indice LAI vert à l&#x27;aide de la formule empirique suivante, tirée de Boegh et al (2002) :

$$
LAI = 3.618 * EVI - 0.118
$$

Où EVI est :

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Les valeurs élevées de LAI varient généralement entre environ 0 et 3,5. Cependant, lorsque la scène contient des nuages et d&#x27;autres éléments lumineux produisant des pixels saturés, les valeurs de LAI peuvent dépasser 3,5. Idéalement, vous devriez masquer les nuages et les éléments lumineux de votre scène avant de créer une image LAI.

_Référence : Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde et A. Thomsen. « Données multispectrales aéroportées pour la quantification de l&#x27;indice de surface foliaire, de la concentration en azote et de l&#x27;efficacité photosynthétique en agriculture. » Remote Sensing of Environment 81, n° 2-3 (2002) : 179-193._

***

## LCI - Indice de chlorophylle foliaire

Cet indice est utilisé pour estimer la teneur en chlorophylle des plantes supérieures ; il est sensible aux variations de réflectance causées par l&#x27;absorption de la chlorophylle.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Référence : Datt, B. « Télédétection de la teneur en eau des feuilles d&#x27;eucalyptus ». Journal of Plant Physiology 154, n° 1 (1999) : 30-36._

***

## MNLI - Indice non linéaire modifié

Cet indice est une amélioration de l&#x27;indice non linéaire (NLI) qui intègre l&#x27;indice de végétation ajusté au sol (SAVI) afin de tenir compte du fond du sol. ENVI utilise une valeur de facteur d&#x27;ajustement du fond de la canopée (_L_) de 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Référence : Yang, Z., P. Willis et R. Mueller. « Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy ». Actes du symposium Pecora 17 sur la télédétection (2008), Denver, CO._

***

## MSAVI2 - Indice de végétation ajusté au sol modifié 2

Cet indice est une version simplifiée de l&#x27;indice MSAVI proposé par Qi et al. (1994), qui améliore l&#x27;indice de végétation ajusté au sol (SAVI). Il réduit le bruit du sol et augmente la gamme dynamique du signal de végétation. Le MSAVI2 repose sur une méthode inductive qui n&#x27;utilise pas de valeur constante _L_ (comme dans le cas du SAVI) pour mettre en évidence la végétation saine.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Référence : Qi, J., A. Chehbouni, A. Huete, Y. Kerr et S. Sorooshian. « A Modified Soil Adjusted Vegetation Index ». Remote Sensing of Environment 48 (1994) : 119-126._

***

## NDRE - Différence normalisée RedEdge

Cet indice est similaire à NDVI, mais compare le contraste entre NIR et RedEdge au lieu de Red, ce qui permet souvent de détecter plus tôt le stress de la végétation.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indice de végétation par différence normalisée

Cet indice mesure la santé de la végétation verte. La combinaison de sa formulation par différence normalisée et de l&#x27;utilisation des régions d&#x27;absorption et de réflectance les plus élevées de la chlorophylle le rend robuste dans un large éventail de conditions. Il peut toutefois atteindre la saturation dans des conditions de végétation dense lorsque LAI devient élevé.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

La valeur de cet indice varie de -1 à 1. La plage courante pour la végétation verte est comprise entre 0,2 et 0,8.

_Référence : Rouse, J., R. Haas, J. Schell et D. Deering. Surveillance des systèmes de végétation dans les Grandes Plaines avec l&#x27;ERTS. Troisième symposium ERTS, NASA (1973) : 309-317._

***

## NLI - Indice non linéaire

Cet indice part du principe que la relation entre de nombreux indices de végétation et les paramètres biophysiques de surface est non linéaire. Il linéarise les relations avec les paramètres de surface qui ont tendance à être non linéaires.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Référence : Goel, N., et W. Qin. « Influences de l&#x27;architecture du couvert végétal sur les relations entre divers indices de végétation et LAI et Fpar : une simulation informatique ». Remote Sensing Reviews 10 (1994) : 309-347._

***

## OSAVI - Indice de végétation ajusté au sol optimisé

Cet indice est basé sur l&#x27;indice de végétation ajusté au sol (SAVI). Il utilise une valeur standard de 0,16 pour le facteur d&#x27;ajustement du fond de la canopée. Rondeaux (1996) a déterminé que cette valeur offre une plus grande variation du sol que SAVI pour une faible couverture végétale, tout en démontrant une sensibilité accrue à une couverture végétale supérieure à 50 %. Cet indice est particulièrement adapté aux zones à végétation relativement clairsemée où le sol est visible à travers la canopée.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Référence : Rondeaux, G., M. Steven et F. Baret. « Optimization of Soil-Adjusted Vegetation Indices ». Remote Sensing of Environment 55 (1996) : 95-107._

***

## RDVI - Indice de végétation par différence renormalisée

Cet indice utilise la différence entre les longueurs d&#x27;onde du proche infrarouge et du rouge, en association avec l&#x27;NDVI, pour mettre en évidence la végétation saine. Il est insensible aux effets du sol et de la géométrie d&#x27;observation du soleil.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Référence : Roujean, J., et F. Breon. « Estimation de la PAR absorbée par la végétation à partir de mesures de réflectance bidirectionnelle ». Remote Sensing of Environment 51 (1995) : 375-384._

***

## SAVI - Indice de végétation ajusté au sol

Cet indice est similaire à NDVI, mais il supprime les effets des pixels de sol. Il utilise un facteur d&#x27;ajustement du fond de la canopée, _L_, qui est fonction de la densité de la végétation et nécessite souvent une connaissance préalable de la quantité de végétation. Huete (1988) suggère une valeur optimale de _L_=0,5 pour tenir compte des variations de fond du sol de premier ordre. Cet indice est particulièrement adapté aux zones à végétation relativement clairsemée où le sol est visible à travers la canopée.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Référence : Huete, A. « A Soil-Adjusted Vegetation Index (SAVI) ». Remote Sensing of Environment 25 (1988) : 295-309._

***

## TDVI - Indice de végétation par différence transformée

Cet indice est utile pour surveiller le couvert végétal en milieu urbain. Il ne sature pas comme les indices NDVI et SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Référence : Bannari, A., H. Asalhi et P. Teillet. « Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping » Dans les Actes du Symposium sur les géosciences et la télédétection, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Indice visible résistant aux effets atmosphériques

Cet indice est basé sur l&#x27;ARVI et sert à estimer la fraction de végétation dans une scène avec une faible sensibilité aux effets atmosphériques.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Référence : Gitelson, A., et al. « Lignes de végétation et de sol dans l&#x27;espace spectral visible : un concept et une technique pour l&#x27;estimation à distance de la fraction de végétation. International Journal of Remote Sensing 23 (2002) : 2537−2562._

***

## WDRVI - Indice de végétation à large plage dynamique

Cet indice est similaire à l&#x27;NDVI, mais il utilise un coefficient de pondération (_a_) pour réduire la disparité entre les contributions des signaux du proche infrarouge et du rouge à l&#x27;NDVI. L&#x27;WDRVI est particulièrement efficace dans les scènes présentant une densité de végétation modérée à élevée lorsque l&#x27;NDVI dépasse 0,6. NDVI a tendance à se stabiliser lorsque la fraction de végétation et l&#x27;indice de surface foliaire (LAI) augmentent, tandis que le WDRVI est plus sensible à une plus large gamme de fractions de végétation et aux variations de LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Le coefficient de pondération (_a_) peut varier de 0,1 à 0,2. Une valeur de 0,2 est recommandée par Henebry, Viña et Gitelson (2004).

_Références_

_Gitelson, A. « Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation ». Journal of Plant Physiology 161, n° 2 (2004) : 165-173._

_Henebry, G., A. Viña et A. Gitelson. « The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis ». Gap Analysis Bulletin 12 : 50-56._
