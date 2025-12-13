---
description: This page lists some multispectral indices that Chloros uses.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---
# Formules d&#x27;indice multispectral

Les formules d&#x27;indice ci-dessous utilisent une combinaison des plages de transmission moyennes du filtre Survey3 :

<table><thead><tr><th align="center">Couleur du filtre Survey3</th><th width="196.199951171875" align="center">Nom du filtre Survey3</th><th width="159.800048828125" align="center">Plage de transmission (FWHM)</th><th align="center">Transmission moyenne</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Lorsque ces formules sont utilisées, le nom peut se terminer par « \_1 » ou « \_2 », ce qui correspond au filtre NIR utilisé, soit NIR1, soit NIR2.

***

## EVI - Indice de végétation amélioré

Cet indice a été initialement développé pour être utilisé avec les données MODIS afin d&#x27;améliorer le NDVI en optimisant le signal de végétation dans les zones à indice de surface foliaire élevé (LAI). Il est particulièrement utile dans les régions à indice LAI élevé où l&#x27;indice NDVI peut être saturé. Il utilise la région de réflectance bleue pour corriger les signaux de fond du sol et réduire les influences atmosphériques, y compris la diffusion des aérosols.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Les valeurs EVI doivent être comprises entre 0 et 1 pour les pixels de végétation. Les éléments clairs tels que les nuages et les bâtiments blancs, ainsi que les éléments sombres tels que l&#x27;eau, peuvent entraîner des valeurs de pixels anormales dans une image EVI. Avant de créer une image EVI, vous devez masquer les nuages et les éléments clairs de l&#x27;image de réflectance, et éventuellement fixer un seuil pour les valeurs de pixels compris entre 0 et 1.

_Référence : Huete, A., et al. « Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices » (Aperçu des performances radiométriques et biophysiques des indices de végétation MODIS). Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indice de couverture forestière 1

Cet indice distingue les canopées forestières des autres types de végétation à l&#x27;aide d&#x27;images de réflectance multispectrales qui incluent une bande rouge.

$$
FCI1 = Red * RedEdge
$$

Les zones forestières auront des valeurs FCI1 plus faibles en raison de la réflectance plus faible des arbres et de la présence d&#x27;ombres dans la canopée.

_Référence : Becker, Sarah J., Craig S.T. Daughtry et Andrew L. Russ. « Robust forest cover indices for multispectral images » (Indices robustes de couverture forestière pour les images multispectrales). Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018) : 505-512._

***

## FCI2 - Indice de couverture forestière 2

Cet indice distingue les couvertures forestières des autres types de végétation à l&#x27;aide d&#x27;images de réflectance multispectrale qui n&#x27;incluent pas de bande rouge.

$$
FCI2 = Red * NIR
$$

Les zones forestières auront des valeurs FCI2 plus faibles en raison de la réflectance plus faible des arbres et de la présence d&#x27;ombres dans la canopée.

_Référence : Becker, Sarah J., Craig S.T. Daughtry et Andrew L. Russ. « Robust forest cover indices for multispectral images » (Indices robustes de couverture forestière pour les images multispectrales). Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018) : 505-512._

***

## GEMI - Indice de surveillance environnementale mondiale

Cet indice de végétation non linéaire est utilisé pour la surveillance environnementale mondiale à partir d&#x27;images satellites et tente de corriger les effets atmosphériques. Il est similaire à NDVI, mais est moins sensible aux effets atmosphériques. Il est affecté par le sol nu ; par conséquent, son utilisation n&#x27;est pas recommandée dans les zones à végétation clairsemée ou modérément dense.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Où :

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Référence : Pinty, B., et M. Verstraete. GEMI : un indice non linéaire pour surveiller la végétation mondiale à partir de satellites. Vegetation 101 (1992) : 15-20._

***

## GARI - Green Indice résistant à l&#x27;atmosphère

Cet indice est plus sensible à une large gamme de concentrations de chlorophylle et moins sensible aux effets atmosphériques que NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La constante gamma est une fonction de pondération qui dépend des conditions des aérosols dans l&#x27;atmosphère. ENVI utilise une valeur de 1,7, qui est la valeur recommandée par Gitelson, Kaufman et Merzylak (1996, page 296).

_Référence : Gitelson, A., Y. Kaufman et M. Merzylak. « Utilisation d&#x27;un canal Green dans la télédétection de la végétation mondiale à partir d&#x27;EOS-MODIS. » Remote Sensing of Environment 58 (1996) : 289-298._

***

## GCI - Green Indice de chlorophylle

Cet indice est utilisé pour estimer la teneur en chlorophylle des feuilles d&#x27;un large éventail d&#x27;espèces végétales.

$$
GCI = {NIR \over Green} - 1
$$

Le fait de disposer de longueurs d&#x27;onde NIR et vertes étendues permet de mieux prédire la teneur en chlorophylle tout en offrant une plus grande sensibilité et un rapport signal/bruit plus élevé.

_Référence : Gitelson, A., Y. Gritz et M. Merzlyak. « Relations entre la teneur en chlorophylle des feuilles et la réflectance spectrale et algorithmes pour l&#x27;évaluation non destructive de la chlorophylle dans les feuilles des plantes supérieures ». Journal of Plant Physiology 160 (2003) : 271-282._

***

## GLI - Green Indice foliaire

Cet indice a été initialement conçu pour être utilisé avec un appareil photo numérique RGB afin de mesurer la couverture du blé, où les valeurs numériques (DN) rouge, verte et bleue varient de 0 à 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Les valeurs GLI varient de -1 à +1. Les valeurs négatives représentent le sol et les éléments non vivants, tandis que les valeurs positives représentent les feuilles et les tiges vertes.

_Référence : Louhaichi, M., M. Borman et D. Johnson. « Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat » (Plateforme localisée spatialement et photographie aérienne pour la documentation des impacts du pâturage sur le blé). Geocarto International 16, n° 1 (2001) : 65-70._

***

## GNDVI - Green Indice de végétation par différence normalisée

Cet indice est similaire à NDVI, sauf qu&#x27;il mesure le spectre vert de 540 à 570 nm au lieu du spectre rouge. Cet indice est plus sensible à la concentration en chlorophylle que NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Référence : Gitelson, A., et M. Merzlyak. « Télédétection de la concentration en chlorophylle dans les feuilles des plantes supérieures ». Advances in Space Research 22 (1998) : 689-692._

***

## GOSAVI - Green Indice de végétation optimisé ajusté au sol

Cet indice a été initialement conçu à partir de photographies infrarouges en couleur afin de prédire les besoins en azote du maïs. Il est similaire à OSAVI, mais remplace la bande verte par la bande rouge.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Référence : Sripada, R., et al. « Détermination des besoins en azote du maïs en cours de saison à l&#x27;aide de la photographie aérienne infrarouge couleur ». Thèse de doctorat, Université d&#x27;État de Caroline du Nord, 2005._

***

## GRVI - Green Indice de végétation de ratio

Cet indice est sensible aux taux de photosynthèse dans les canopées forestières, car les réflexions vertes et rouges sont fortement influencées par les changements dans les pigments des feuilles.

$$
GRVI = {NIR \over Green }
$$

_Référence : Sripada, R., et al. « Photographie aérienne infrarouge couleur pour déterminer les besoins en azote du maïs en début de saison ». Agronomy Journal 98 (2006) : 968-977._

***

## GSAVI - Green Indice de végétation ajusté au sol

Cet indice a été initialement conçu à partir de photographies infrarouges couleur afin de prédire les besoins en azote du maïs. Il est similaire à SAVI, mais remplace la bande verte par la bande rouge.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Référence : Sripada, R., et al. « Détermination des besoins en azote du maïs en cours de saison à l&#x27;aide de la photographie aérienne infrarouge couleur ». Thèse de doctorat, Université d&#x27;État de Caroline du Nord, 2005._

***

## LAI - Indice de surface foliaire

Cet indice est utilisé pour estimer la couverture foliaire et prévoir la croissance et le rendement des cultures. ENVI calcule le LAI vert à l&#x27;aide de la formule empirique suivante de Boegh et al (2002) :

$$
LAI = 3.618 * EVI - 0.118
$$

Où EVI est :

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Les valeurs élevées de LAI varient généralement entre 0 et 3,5 environ. Cependant, lorsque la scène contient des nuages et d&#x27;autres éléments lumineux qui produisent des pixels saturés, les valeurs de LAI peuvent dépasser 3,5. Idéalement, vous devriez masquer les nuages et les éléments lumineux de votre scène avant de créer une image LAI.

_Référence : Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde et A. Thomsen. « Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture » (Données multispectrales aériennes pour quantifier l&#x27;indice de surface foliaire, la concentration en azote et l&#x27;efficacité photosynthétique dans l&#x27;agriculture). Remote Sensing of Environment 81, n° 2-3 (2002) : 179-193._

***

## LCI - Indice de chlorophylle foliaire

Cet indice est utilisé pour estimer la teneur en chlorophylle des plantes supérieures, sensibles aux variations de réflectance causées par l&#x27;absorption de la chlorophylle.

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

_Référence : Yang, Z., P. Willis et R. Mueller. « Impact de l&#x27;image AWIFS améliorée par rapport de bande sur la précision de la classification des cultures ». Actes du symposium Pecora 17 sur la télédétection (2008), Denver, CO._

***

## MSAVI2 - Indice de végétation ajusté au sol modifié 2

Cet indice est une version simplifiée de l&#x27;indice MSAVI proposé par Qi et al. (1994), qui améliore l&#x27;indice de végétation ajusté au sol (SAVI). Il réduit le bruit du sol et augmente la gamme dynamique du signal de végétation. MSAVI2 est basé sur une méthode inductive qui n&#x27;utilise pas de valeur _L_ constante (comme avec SAVI) pour mettre en évidence une végétation saine.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Référence : Qi, J., A. Chehbouni, A. Huete, Y. Kerr et S. Sorooshian. « A Modified Soil Adjusted Vegetation Index » (Un indice de végétation ajusté au sol modifié). Remote Sensing of Environment 48 (1994) : 119-126.

***

## NDRE - Différence normalisée RedEdge

Cet indice est similaire à NDVI, mais compare le contraste entre NIR et RedEdge au lieu de Red, ce qui permet souvent de détecter plus rapidement le stress végétal.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indice de végétation par différence normalisée

Cet indice mesure la santé et la verdure de la végétation. La combinaison de sa formulation par différence normalisée et de l&#x27;utilisation des régions d&#x27;absorption et de réflectance les plus élevées de la chlorophylle le rend robuste dans un large éventail de conditions. Il peut toutefois saturer dans des conditions de végétation dense lorsque LAI devient élevé.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

La valeur de cet indice varie de -1 à 1. La plage courante pour la végétation verte est comprise entre 0,2 et 0,8.

_Référence : Rouse, J., R. Haas, J. Schell et D. Deering. Surveillance des systèmes de végétation dans les Grandes Plaines avec ERTS. Troisième symposium ERTS, NASA (1973) : 309-317._

***

## NLI - Indice non linéaire

Cet indice part du principe que la relation entre de nombreux indices de végétation et les paramètres biophysiques de surface est non linéaire. Il linéarise les relations avec les paramètres de surface qui ont tendance à être non linéaires.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Référence : Goel, N., et W. Qin. « Influences de l&#x27;architecture du couvert végétal sur les relations entre divers indices de végétation et LAI et Fpar : une simulation informatique ». Remote Sensing Reviews 10 (1994) : 309-347._

***

## OSAVI - Indice de végétation optimisé ajusté au sol

Cet indice est basé sur l&#x27;indice de végétation ajusté au sol (SAVI). Il utilise une valeur standard de 0,16 pour le facteur d&#x27;ajustement de l&#x27;arrière-plan de la canopée. Rondeaux (1996) a déterminé que cette valeur offre une plus grande variation du sol que SAVI pour une couverture végétale faible, tout en démontrant une sensibilité accrue à une couverture végétale supérieure à 50 %. Cet indice est particulièrement adapté aux zones où la végétation est relativement clairsemée et où le sol est visible à travers la canopée.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Référence : Rondeaux, G., M. Steven et F. Baret. « Optimisation des indices de végétation ajustés au sol ». Télédétection de l&#x27;environnement 55 (1996) : 95-107._

***

## RDVI - Indice de végétation par différence renormalisée

Cet indice utilise la différence entre les longueurs d&#x27;onde du proche infrarouge et du rouge, ainsi que le NDVI, pour mettre en évidence une végétation saine. Il est insensible aux effets du sol et à la géométrie d&#x27;observation du soleil.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Référence : Roujean, J., et F. Breon. « Estimation du PAR absorbé par la végétation à partir de mesures de réflectance bidirectionnelle ». Télédétection de l&#x27;environnement 51 (1995) : 375-384._

***

## SAVI - Indice de végétation ajusté au sol

Cet indice est similaire à NDVI, mais il supprime les effets des pixels du sol. Il utilise un facteur d&#x27;ajustement de l&#x27;arrière-plan de la canopée, _L_, qui est une fonction de la densité de la végétation et nécessite souvent une connaissance préalable de la quantité de végétation. Huete (1988) suggère une valeur optimale de _L_=0,5 pour tenir compte des variations de premier ordre de l&#x27;arrière-plan du sol. Cet indice est particulièrement adapté aux zones où la végétation est relativement clairsemée et où le sol est visible à travers la canopée.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Référence : Huete, A. « A Soil-Adjusted Vegetation Index (SAVI) ». Remote Sensing of Environment 25 (1988) : 295-309._

***

## TDVI - Indice de végétation transformé par différence

Cet indice est utile pour surveiller la couverture végétale en milieu urbain. Il ne sature pas comme NDVI et SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Référence : Bannari, A., H. Asalhi et P. Teillet. « Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping » (Indice de végétation transformé (TDVI) pour la cartographie de la couverture végétale) dans Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Indice visible résistant à l&#x27;atmosphère

Cet indice est basé sur le ARVI et est utilisé pour estimer la fraction de végétation dans une scène avec une faible sensibilité aux effets atmosphériques.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Référence : Gitelson, A., et al. « Lignes de végétation et de sol dans l&#x27;espace spectral visible : un concept et une technique pour l&#x27;estimation à distance de la fraction de végétation. International Journal of Remote Sensing 23 (2002) : 2537-2562._

***

## WDRVI - Indice de végétation à large gamme dynamique

Cet indice est similaire à NDVI, mais il utilise un coefficient de pondération (_a_) pour réduire la disparité entre les contributions des signaux proche infrarouge et rouge à NDVI. Le WDRVI est particulièrement efficace dans les scènes présentant une densité de végétation modérée à élevée lorsque le NDVI dépasse 0.6. Le NDVI a tendance à se stabiliser lorsque la fraction de végétation et l&#x27;indice de surface foliaire (LAI) augmentent, tandis que le WDRVI est plus sensible à une plus large gamme de fractions de végétation et aux changements dans le LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Le coefficient de pondération (_a_) peut varier de 0,1 à 0,2. Une valeur de 0,2 est recommandée par Henebry, Viña et Gitelson (2004).

_Références_

_Gitelson, A. « Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation » (Indice de végétation à large gamme dynamique pour la quantification à distance des caractéristiques biophysiques de la végétation). Journal of Plant Physiology 161, n° 2 (2004) : 165-173._

_Henebry, G., A. Viña et A. Gitelson. « The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis » (L&#x27;indice de végétation à large gamme dynamique et son utilité potentielle pour l&#x27;analyse des écarts). Gap Analysis Bulletin 12 : 50-56._
