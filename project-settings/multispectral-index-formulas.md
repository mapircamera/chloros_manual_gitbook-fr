---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formules de l'indice multispectral

Les formules d'index ci-dessous utilisent une combinaison des plages de transmission moyenne des filtres Survey3 :

<table><thead><tr><th align="center">Survey3 Filter Color</th><th width="196.199951171875" align="center">Survey3 Filter Name</th><th width="159.800048828125" align="center">Transmission Range (FWHM)</th><th align="center">Average Transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td> align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>

Lorsque ces formules sont utilisées, le nom peut se terminer par "\_1" ou "\_2", ce qui correspond au filtre NIR utilisé, soit NIR1, soit NIR2.

***

## EVI - Indice de végétation amélioré

Cet indice a été développé à l'origine pour être utilisé avec les données MODIS comme une amélioration par rapport à NDVI en optimisant le signal de végétation dans les zones où l'indice de surface foliaire est élevé (LAI). Il est particulièrement utile dans les régions à LAI élevé où NDVI risque d'être saturé. Il utilise la région de réflectance bleue pour corriger les signaux de fond du sol et pour réduire les influences atmosphériques, y compris la diffusion des aérosols.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

les valeurs de EVI doivent être comprises entre 0 et 1 pour les pixels de végétation. Les éléments brillants tels que les nuages et les bâtiments blancs, ainsi que les éléments sombres tels que l'eau, peuvent entraîner des valeurs de pixels anormales dans une image EVI. Avant de créer une image EVI, vous devez masquer les nuages et les éléments lumineux de l'image de réflectance, et éventuellement seuiller les valeurs des pixels de 0 à 1.

référence : Huete, A., et al. "Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices" Remote Sensing of Environment 83 (2002):195-213

***

## FCI1 - Indice de couverture forestière 1

Cet indice distingue les couverts forestiers des autres types de végétation à l'aide d'images de réflectance multispectrales comprenant une bande rouge.

$$
FCI1 = Red * RedEdge
$$

Les zones forestières auront des valeurs FCI1 plus faibles en raison de la réflectance plus faible des arbres et de la présence d'ombres à l'intérieur de la canopée.

référence : Becker, Sarah J., Craig S.T. Daughtry, et Andrew L. Russ. "Robust forest cover indices for multispectral images" Photogrammetric Engineering & Remote Sensing 84.8 (2018) : 505-512._

***

## FCI2 - Indice de couverture forestière 2

Cet indice permet de distinguer les couverts forestiers des autres types de végétation à l'aide d'images de réflectance multispectrales qui ne comportent pas de bande rouge.

$$
FCI2 = Red * NIR
$$

Les zones forestières auront des valeurs FCI2 plus faibles en raison de la réflectance plus faible des arbres et de la présence d'ombres à l'intérieur de la canopée.

référence : Becker, Sarah J., Craig S.T. Daughtry, et Andrew L. Russ. "Robust forest cover indices for multispectral images" Photogrammetric Engineering & Remote Sensing 84.8 (2018) : 505-512._

***

## GEMI - Indice de surveillance de l'environnement mondial

Cet indice de végétation non linéaire est utilisé pour la surveillance de l'environnement mondial à partir de l'imagerie satellitaire et tente de corriger les effets atmosphériques. Il est similaire à NDVI mais est moins sensible aux effets atmosphériques. Il est affecté par les sols nus ; il n'est donc pas recommandé de l'utiliser dans les zones où la végétation est clairsemée ou modérément dense.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Où :

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

référence : Pinty, B., et M. Verstraete. GEMI : a Non-Linear Index to Monitor Global Vegetation From Satellites. Vegetation 101 (1992) : 15-20._

***

## GARI - Green Indice de résistance atmosphérique

Cet indice est plus sensible à une large gamme de concentrations de chlorophylle et moins sensible aux effets atmosphériques que NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La constante gamma est une fonction de pondération qui dépend des conditions d'aérosols dans l'atmosphère. ENVI utilise une valeur de 1.7, qui est la valeur recommandée par Gitelson, Kaufman, et Merzylak (1996, page 296).

référence : Gitelson, A., Y. Kaufman, et M. Merzylak. "Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS Remote Sensing of Environment 58 (1996) : 289-298._

***

## GCI - Green Indice de chlorophylle

Cet indice est utilisé pour estimer la teneur en chlorophylle des feuilles d'une large gamme d'espèces végétales.

$$
GCI = {NIR \over Green} - 1
$$

Le fait d'avoir des longueurs d'onde larges et vertes permet une meilleure prédiction de la teneur en chlorophylle tout en permettant une plus grande sensibilité et un rapport signal/bruit plus élevé.

référence : Gitelson, A., Y. Gritz et M. Merzlyak. "Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves" Journal of Plant Physiology 160 (2003) : 271-282._

***

## GLI - Green Indice foliaire

Cet indice a été conçu à l'origine pour être utilisé avec un appareil photo numérique RGB pour mesurer la couverture du blé, où les nombres numériques (DN) rouge, vert et bleu vont de 0 à 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI s'échelonnent de -1 à +1. Les valeurs négatives représentent le sol et les éléments non vivants, tandis que les valeurs positives représentent les feuilles et les tiges vertes.

référence : Louhaichi, M., M. Borman, et D. Johnson. "Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat" Geocarto International 16, No. 1 (2001) : 65-70._

***

## GNDVI - Green Indice de végétation par différence normalisée

Cet indice est similaire à NDVI sauf qu'il mesure le spectre vert de 540 à 570 nm au lieu du spectre rouge. Cet indice est plus sensible à la concentration de chlorophylle que NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

référence : Gitelson, A., et M. Merzlyak. "Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves (Télédétection de la concentration de chlorophylle dans les feuilles des plantes supérieures) Advances in Space Research 22 (1998) : 689-692._

***

## GOSAVI - Green Indice de végétation optimisé ajusté au sol

Cet indice a été conçu à l'origine avec la photographie couleur-infrarouge pour prévoir les besoins en azote du maïs. Il est similaire à OSAVI, mais la bande verte est remplacée par la bande rouge.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

référence : Sripada, R., et al. "Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography" Thèse de doctorat, North Carolina State University, 2005

***

## GRVI - Green Indice de rapport de végétation

Cet indice est sensible aux taux de photosynthèse dans les canopées forestières, car les réflectances verte et rouge sont fortement influencées par les changements dans les pigments des feuilles.

$$
GRVI = {NIR \over Green }
$$

référence : Sripada, R., et al. "Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn" Agronomy Journal 98 (2006) : 968-977._

***

## GSAVI - Green Indice de végétation ajusté au sol

Cet indice a été conçu à l'origine avec la photographie couleur-infrarouge pour prédire les besoins en nitrogène du maïs. Il est similaire à SAVI, mais la bande verte est remplacée par la bande rouge.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

référence : Sripada, R., et al. "Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography" Thèse de doctorat, North Carolina State University, 2005

***

## LAI - Indice de surface foliaire

Cet indice est utilisé pour estimer la couverture foliaire et pour prévoir la croissance et le rendement des cultures. ENVI calcule l'indice vert LAI en utilisant la formule empirique suivante, issue de Boegh et al (2002) :

$$
LAI = 3.618 * EVI - 0.118
$$

Où EVI est :

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Les valeurs LAI élevées sont généralement comprises entre 0 et 3,5. Cependant, lorsque la scène contient des nuages et d'autres éléments lumineux qui produisent des pixels saturés, les valeurs LAI peuvent dépasser 3,5. Vous devriez idéalement masquer les nuages et les éléments lumineux de votre scène avant de créer une image LAI.

référence : Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, et A. Thomsen. "Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture (Données multispectrales aéroportées pour quantifier l'indice de surface foliaire, la concentration d'azote et l'efficacité photosynthétique en agriculture) Remote Sensing of Environment 81, no. 2-3 (2002) : 179-193._

***

## LCI - Indice de chlorophylle des feuilles

Cet indice est utilisé pour estimer la teneur en chlorophylle des plantes supérieures, sensible à la variation de la réflectance causée par l'absorption de la chlorophylle.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

référence : Datt, B. "Remote Sensing of Water Content in Eucalyptus Leaves" Journal of Plant Physiology 154, no. 1 (1999) : 30-36._

***

## MNLI - Indice non linéaire modifié

Cet indice est une amélioration de l'Indice Non-Linéaire (INL) qui incorpore l'Indice de Végétation Ajusté au Sol (SAVI) pour prendre en compte l'arrière-plan du sol. ENVI utilise un facteur d'ajustement du couvert végétal (_L_) de 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

référence : Yang, Z., P. Willis, et R. Mueller. "Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy (Impact de l'image AWIFS améliorée par le rapport de bande sur la précision de la classification des cultures) Actes du symposium Pecora 17 sur la télédétection (2008), Denver, CO

***

## MSAVI2 - Indice de végétation ajusté au sol modifié 2

Cet indice est une version plus simple de l'indice MSAVI proposé par Qi, et al (1994), qui améliore l'indice de végétation ajusté au sol (SAVI). Il réduit le bruit du sol et augmente la gamme dynamique du signal de végétation. MSAVI2 est basé sur une méthode inductive qui n'utilise pas une valeur _L_ constante (comme pour SAVI) pour mettre en évidence la végétation saine.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

référence : Qi, J., A. Chehbouni, A. Huete, Y. Kerr, et S. Sorooshian. "A Modified Soil Adjusted Vegetation Index Remote Sensing of Environment 48 (1994) : 119-126._

***

## NDRE- Différence normalisée RedEdge

Cet indice est similaire à NDVI mais compare le contraste entre NIR et RedEdge au lieu de Red, qui détecte souvent le stress de la végétation plus tôt.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indice de végétation par différence normalisée

Cet indice est une mesure de la santé et de la verdure de la végétation. La combinaison de la formulation de la différence normalisée et de l'utilisation des régions d'absorption et de réflectance les plus élevées de la chlorophylle le rend robuste dans une large gamme de conditions. Il peut cependant saturer dans des conditions de végétation dense lorsque LAI devient élevé.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

La valeur de cet indice varie de -1 à 1. La fourchette habituelle pour la végétation verte est de 0,2 à 0,8.

référence : Rouse, J., R. Haas, J. Schell et D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Troisième symposium ERTS, NASA (1973) : 309-317._

***

## NLI - Indice non linéaire

Cet indice part du principe que la relation entre de nombreux indices de végétation et les paramètres biophysiques de surface n'est pas linéaire. Il linéarise les relations avec les paramètres de surface qui ont tendance à être non linéaires.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

référence : Goel, N. et W. Qin. "Influences of Canopy Architecture on Relationships Between Various Vegetation Indices and LAI and Fpar : A Computer Simulation." Remote Sensing Reviews 10 (1994) : 309-347._

***

## OSAVI - Indice de végétation optimisé ajusté au sol

Cet indice est basé sur l'indice de végétation ajusté au sol (SAVI). Il utilise une valeur standard de 0,16 pour le facteur d'ajustement de l'arrière-plan de la canopée. Rondeaux (1996) a déterminé que cette valeur fournit une plus grande variation du sol que SAVI pour une faible couverture végétale, tout en démontrant une sensibilité accrue pour une couverture végétale supérieure à 50 %. Cet indice est mieux utilisé dans les zones où la végétation est relativement clairsemée et où le sol est visible à travers la canopée.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

référence : Rondeaux, G., M. Steven, et F. Baret. "Optimisation des indices de végétation ajustés au sol Remote Sensing of Environment 55 (1996) : 95-107._

***

## RDVI - Indice de végétation par différence renormalisée

Cet indice utilise la différence entre les longueurs d'onde du proche infrarouge et du rouge, ainsi que le NDVI, pour mettre en évidence une végétation saine. Il est insensible aux effets du sol et de la géométrie d'observation du soleil.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

référence : Roujean, J., et F. Breon. "Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements (Estimation du PAR absorbé par la végétation à partir de mesures de réflectance bidirectionnelles) Remote Sensing of Environment 51 (1995) : 375-384._

***

## SAVI - Indice de végétation ajusté au sol

Cet indice est similaire à NDVI, mais il supprime les effets des pixels du sol. Il utilise un facteur d'ajustement de l'arrière-plan de la canopée, _L_, qui est fonction de la densité de la végétation et nécessite souvent une connaissance préalable des quantités de végétation. Huete (1988) suggère une valeur optimale de _L_=0,5 pour tenir compte des variations de premier ordre de l'arrière-plan du sol. Cet indice est mieux utilisé dans les zones où la végétation est relativement clairsemée et où le sol est visible à travers la canopée.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

référence : Huete, A. "A Soil-Adjusted Vegetation Index (SAVI)" Remote Sensing of Environment 25 (1988) : 295-309._

***

## TDVI - Indice de végétation par différence transformée

Cet indice est utile pour surveiller la couverture végétale dans les environnements urbains. Il ne sature pas comme NDVI et SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

référence : Bannari, A., H. Asalhi, et P. Teillet. "Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping" In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS '02, IEEE International, Volume 5 (2002)

***

## VARI - Indice de résistance à l'atmosphère visible

Cet indice est basé sur le ARVI et est utilisé pour estimer la fraction de végétation dans une scène peu sensible aux effets atmosphériques.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

référence : Gitelson, A., et al. "Vegetation and Soil Lines in Visible Spectral Space : A Concept and Technique for Remote Estimation of Vegetation Fraction". International Journal of Remote Sensing 23 (2002) : 2537-2562._

***

## WDRVI - Indice de végétation à large gamme dynamique

Cet indice est similaire à l'indice NDVI, mais il utilise un coefficient de pondération (_a_) pour réduire la disparité entre les contributions des signaux proche infrarouge et rouge à l'indice NDVI. Le WDRVI est particulièrement efficace dans les scènes qui ont une densité de végétation modérée à élevée lorsque le NDVI dépasse 0,6. le NDVI tend à se stabiliser lorsque la fraction de végétation et l'indice de surface foliaire (LAI) augmentent, alors que le WDRVI est plus sensible à une gamme plus large de fractions de végétation et aux changements dans LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Le coefficient de pondération (_a_) peut varier de 0,1 à 0,2. Une valeur de 0,2 est recommandée par Henebry, Viña et Gitelson (2004).

références

gitelson, A. "Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation" (Indice de végétation à large gamme dynamique pour la quantification à distance des caractéristiques biophysiques de la végétation) Journal of Plant Physiology 161, No. 2 (2004) : 165-173._

henebry, G., A. Viña, et A. Gitelson. "The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis " (L'indice de végétation à large gamme dynamique et son utilité potentielle pour l'analyse des lacunes) Gap Analysis Bulletin 12 : 50-56
