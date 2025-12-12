# Paramètres du projet

La barre latérale Paramètres du projet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> de Chloros vous permettent de configurer tous les aspects du traitement d&#x27;image, de la détection des cibles d&#x27;étalonnage, des calculs d&#x27;indices multispectraux et des options d&#x27;exportation pour votre projet. Ces paramètres sont enregistrés avec votre projet et peuvent être sauvegardés sous forme de modèles afin d&#x27;être réutilisés dans plusieurs projets.

## Accéder aux paramètres du projet

Pour accéder aux paramètres du projet :

1. Ouvrez un projet dans Chloros
2. Cliquez sur l&#x27;onglet **Paramètres du projet**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau des paramètres affiche toutes les options de configuration disponibles, classées par catégorie

***

## Détection des cibles

Ces paramètres contrôlent la manière dont Chloros détecte et traite les cibles d&#x27;étalonnage dans vos images.

### Zone minimale d&#x27;échantillonnage d&#x27;étalonnage (px)

* **Type** : Nombre
* **Plage** : 0 à 10 000 pixels
* **Valeur par défaut** : 25 pixels
* **Description** : Définit la zone minimale (en pixels) requise pour qu&#x27;une région détectée soit considérée comme un échantillon de cible d&#x27;étalonnage valide. Des valeurs plus petites permettront de détecter des cibles plus petites, mais peuvent augmenter le nombre de faux positifs. Des valeurs plus grandes nécessitent des régions cibles plus grandes et plus claires pour la détection.
* **Quand ajuster** :
  * Augmentez si vous obtenez de fausses détections sur de petits artefacts d&#x27;image.
  * Diminuez si vos cibles d&#x27;étalonnage apparaissent petites dans vos images et ne sont pas détectées.

### Regroupement minimal des cibles (0-100)

* **Type** : Nombre
* **Plage** : 0 à 100
* **Par défaut** : 60
* **Description** : contrôle le seuil de regroupement pour regrouper les zones de couleurs similaires lors de la détection des cibles d&#x27;étalonnage. Des valeurs plus élevées nécessitent que des couleurs plus similaires soient regroupées, ce qui se traduit par une détection plus conservatrice des cibles. Des valeurs plus faibles permettent une plus grande variation de couleurs au sein d&#x27;un groupe cible.
* **Quand ajuster** :
  * Augmentez si les cibles d&#x27;étalonnage sont divisées en plusieurs détections.
  * Diminuez si les cibles d&#x27;étalonnage présentant des variations de couleur ne sont pas entièrement détectées.

***

## Traitement

Ces paramètres contrôlent la manière dont Chloros traite et calibre vos images.

### Correction du vignettage

* **Type** : case à cocher
* **Par défaut** : activé (coché)
* **Description** : Applique une correction du vignettage pour compenser l&#x27;assombrissement de l&#x27;objectif sur les bords des images. Le vignettage est un phénomène optique courant où les coins et les bords d&#x27;une image apparaissent plus sombres que le centre en raison des caractéristiques de l&#x27;objectif.
* **Quand désactiver** : ne désactivez cette option que si votre combinaison appareil photo/objectif a déjà appliqué une correction du vignettage ou si vous souhaitez corriger manuellement le vignettage en post-traitement.

### Calibrage de la réflectance / balance des blancs

* **Type** : Case à cocher
* **Par défaut** : Activé (coché)
* **Description** : Active le calibrage automatique de la réflectance à l&#x27;aide des cibles de calibrage détectées dans vos images. Cela normalise les valeurs de réflectance dans l&#x27;ensemble de vos données et garantit des mesures cohérentes, quelles que soient les conditions d&#x27;éclairage.
* **Quand désactiver** : Ne désactivez cette option que si vous souhaitez traiter des images brutes non calibrées ou si vous utilisez un autre flux de travail de calibrage.

### Méthode de débayérisation

* **Type** : Sélection par menu déroulant
* **Options** :
  * Haute qualité (plus rapide) - Actuellement la seule option disponible
* **Par défaut** : Haute qualité (plus rapide)
* **Description** : Sélectionne l&#x27;algorithme de démosaïquage utilisé pour convertir les données brutes du capteur à motif Bayer en images en couleur. La méthode « Haute qualité (plus rapide) » offre un équilibre optimal entre la vitesse de traitement et la qualité d&#x27;image.
* **Remarque** : d&#x27;autres méthodes de débayérisation pourraient être ajoutées dans les prochaines versions de Chloros.

### Intervalle minimum de recalibrage

* **Type** : nombre
* **Plage** : 0 à 3 600 secondes
* **Par défaut** : 0 seconde
* **Description** : Définit l&#x27;intervalle de temps minimum (en secondes) entre l&#x27;utilisation des cibles d&#x27;étalonnage. Lorsqu&#x27;il est défini sur 0, Chloros utilisera toutes les cibles d&#x27;étalonnage détectées. Lorsqu&#x27;il est défini sur une valeur plus élevée, Chloros n&#x27;utilisera que les cibles d&#x27;étalonnage séparées par au moins ce nombre de secondes, ce qui réduira le temps de traitement des ensembles de données avec des captures fréquentes de cibles d&#x27;étalonnage.
* **Quand ajuster** :
  * Réglez sur 0 pour une précision d&#x27;étalonnage maximale lorsque les conditions d&#x27;éclairage varient.
  * Augmentez (par exemple, à 60-300 secondes) pour un traitement plus rapide lorsque l&#x27;éclairage est constant et que vous avez des images de cibles d&#x27;étalonnage fréquentes.

### Décalage horaire du capteur de lumière

* **Type** : Nombre
* **Plage** : -12 à +12 heures
* **Par défaut** : 0 heure
* **Description** : Spécifie le décalage horaire (en heures par rapport à l&#x27;UTC) pour les horodatages des données du capteur de lumière. Ce paramètre est utilisé lors du traitement des fichiers de données PPK (Post-Processed Kinematic) afin de garantir une synchronisation horaire correcte entre les captures d&#x27;images et les données GPS.
* **Quand ajuster** : Réglez ce paramètre sur le décalage horaire de votre fuseau horaire local si vos données PPK utilisent l&#x27;heure locale au lieu de l&#x27;UTC. Par exemple :
  * Heure du Pacifique : -8 ou -7 (selon l&#x27;heure d&#x27;été)
  * Heure de l&#x27;Est : -5 ou -4 (selon l&#x27;heure d&#x27;été)
  * Heure d&#x27;Europe centrale : +1 ou +2 (selon l&#x27;heure d&#x27;été)

### Appliquer les corrections PPK

* **Type** : case à cocher
* **Par défaut** : désactivé (non coché)
* **Description** : active l&#x27;utilisation des corrections cinématiques post-traitées (PPK) provenant des enregistreurs MAPIR DAQ contenant un GPS (GNSS). Lorsque cette option est activée, Chloros utilise tous les fichiers journaux .daq contenant des données d&#x27;exposition dans le répertoire de votre projet et applique des corrections de géolocalisation précises à vos images.
* **Exigence** : un fichier journal .daq contenant des entrées d&#x27;exposition doit être présent dans le répertoire de votre projet
* **Quand activer** : il est recommandé de toujours activer la correction PPK si votre fichier journal .daq contient des entrées de retour d&#x27;exposition.

### Broche d&#x27;exposition 1

* **Type** : sélection dans un menu déroulant
* **Visibilité** : visible uniquement lorsque « Appliquer les corrections PPK » est activé ET que les données d&#x27;exposition sont disponibles pour la broche 1
* **Options** :
  * Noms des modèles d&#x27;appareils photo détectés dans le projet
  * « Ne pas utiliser » : ignorez cette broche d&#x27;exposition
* **Par défaut** : sélection automatique en fonction de la configuration du projet
* **Description** : Attribue une caméra spécifique à la broche d&#x27;exposition 1 pour la synchronisation temporelle PPK. La broche d&#x27;exposition enregistre le moment exact où l&#x27;obturateur de la caméra est déclenché, ce qui est essentiel pour une géolocalisation PPK précise.
* **Comportement de sélection automatique** :
  * Caméra unique + broche unique : sélectionne automatiquement la caméra
  * Caméra unique + deux broches : la broche 1 est automatiquement attribuée à la caméra
  * Plusieurs caméras : sélection manuelle requise

### Broche d&#x27;exposition 2

* **Type** : sélection dans un menu déroulant
* **Visibilité** : visible uniquement lorsque « Appliquer les corrections PPK » est activé ET que les données d&#x27;exposition sont disponibles pour la broche 2
* **Options** :
  * Noms des modèles de caméra détectés dans le projet
  * « Ne pas utiliser » : ignore cette broche d&#x27;exposition
* **Par défaut** : sélection automatique en fonction de la configuration du projet
* **Description** : attribue une caméra spécifique à la broche d&#x27;exposition 2 pour la synchronisation temporelle PPK lors de l&#x27;utilisation d&#x27;une configuration à deux caméras.
* **Comportement de sélection automatique** :
  * Caméra unique + broche unique : la broche 2 est automatiquement définie sur « Ne pas utiliser »
  * Caméra unique + deux broches : la broche 2 est automatiquement réglée sur « Ne pas utiliser »
  * Plusieurs caméras : sélection manuelle requise
* **Remarque** : la même caméra ne peut pas être attribuée simultanément à la broche 1 et à la broche 2.

***

## Index

Ces paramètres vous permettent de configurer des indices multispectraux pour l&#x27;analyse et la visualisation.

### Ajouter un indice

* **Type** : panneau de configuration d&#x27;indice spécial
* **Description** : ouvre un panneau interactif dans lequel vous pouvez sélectionner et configurer des indices de végétation multispectraux (NDVI, NDRE, EVI, etc.) à calculer pendant le traitement de l&#x27;image. Vous pouvez ajouter plusieurs indices, chacun avec ses propres paramètres de visualisation.
* **Indices disponibles** : le système comprend plus de 30 indices multispectraux prédéfinis, notamment :
  * NDVI (indice de végétation par différence normalisée)
  * NDRE (différence normalisée RedEdge)
  * EVI (indice de végétation amélioré)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Et bien d&#x27;autres encore (voir [Formules d&#x27;indices multispectraux](multispectral-index-formulas.md) pour la liste complète)
* **Caractéristiques** :
  * Sélectionnez parmi les formules d&#x27;indice prédéfinies.
  * Configurez les dégradés de couleurs de visualisation (LUT - Look-Up Tables).
  * Définissez les valeurs seuils pour l&#x27;analyse.
  * Créez des formules d&#x27;indice personnalisées.

### Formules personnalisées (fonctionnalité Chloros+)

* **Type** : tableau de définitions de formules personnalisées
* **Description** : vous permet de créer et d&#x27;enregistrer des formules d&#x27;indice multispectral personnalisées à l&#x27;aide de calculs mathématiques sur les bandes. Les formules personnalisées sont enregistrées avec les paramètres de votre projet et peuvent être utilisées comme des indices intégrés.
* **Comment créer** :
  1. Dans le panneau de configuration de l&#x27;indice, recherchez l&#x27;option de formule personnalisée
  2. Définissez votre formule à l&#x27;aide des identifiants de bande (par exemple, NIR, Red, Green, Blue)
  3. Enregistrez la formule avec un nom descriptif
* **Syntaxe de la formule** : les opérations mathématiques standard sont prises en charge, notamment :
  * Arithmétique : `+`, `-`, `*`, `/`
  * Parenthèses pour l&#x27;ordre des opérations
  * Références de bande : NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exportation

Ces paramètres contrôlent le format et la qualité des images traitées exportées.

### Format d&#x27;image calibré

* **Type** : sélection dans le menu déroulant
* **Options** :
  * **TIFF (16 bits)** - Format TIFF 16 bits non compressé
  * **TIFF (32 bits, pourcentage)** - TIFF 32 bits à virgule flottante avec valeurs de réflectance exprimées en pourcentage
  * **PNG (8 bits)** - Format PNG compressé 8 bits
  * **JPG (8 bits)** - Format JPEG 8 bits compressé
* **Par défaut** : TIFF (16 bits)
* **Description** : sélectionne le format de fichier pour enregistrer les images traitées et calibrées.
* **Recommandations de format** :
  * **TIFF (16 bits)** : recommandé pour les analyses scientifiques et les flux de travail professionnels. Préserve une qualité maximale des données sans artefacts de compression. Idéal pour l&#x27;analyse multispectrale et le traitement ultérieur dans un logiciel SIG.
  * **TIFF (32 bits, pourcentage)** : idéal pour les flux de travail qui nécessitent des valeurs de réflectance en pourcentage (0-100 %). Offre une précision maximale pour les mesures radiométriques.
  * **PNG (8 bits)** : idéal pour la consultation sur le Web et la visualisation générale. Fichiers de taille réduite grâce à une compression sans perte, mais plage dynamique réduite.
  * **JPG (8 bits)** : fichiers de taille minimale, idéal pour les aperçus et l&#x27;affichage sur le Web uniquement. Utilise une compression avec perte qui ne convient pas à l&#x27;analyse scientifique.

***

## Enregistrer le modèle de projet

Cette fonctionnalité vous permet d&#x27;enregistrer les paramètres de votre projet actuel sous forme de modèle réutilisable.

* **Type** : saisie de texte + bouton Enregistrer
* **Description** : entrez un nom descriptif pour votre modèle de paramètres et cliquez sur l&#x27;icône Enregistrer. Le modèle stockera tous les paramètres actuels de votre projet (détection de cible, options de traitement, indices et format d&#x27;exportation) afin de pouvoir les réutiliser facilement dans de futurs projets.
* **Cas d&#x27;utilisation** :
  * Créer des modèles pour différents systèmes de caméras (RGB, multispectral, NIR)
  * Enregistrer des configurations standard pour des types de cultures ou des workflows d&#x27;analyse spécifiques
  * Partager des paramètres cohérents au sein d&#x27;une équipe
* **Mode d&#x27;emploi** :
  1. Configurez tous les paramètres de projet souhaités
  2. Entrez un nom de modèle (par exemple, « RedEdge Survey3 NDVI Standard »).
  3. Cliquez sur l&#x27;icône Enregistrer.
  4. Le modèle peut désormais être chargé lors de la création de nouveaux projets.

***

## Enregistrer le dossier du projet

Ce paramètre spécifie l&#x27;emplacement par défaut où les nouveaux projets sont enregistrés.

* **Type** : Affichage du chemin d&#x27;accès au répertoire + bouton Modifier
* **Par défaut** : `C:\Users\[Username]\Chloros Projects`
* **Description** : Affiche le répertoire par défaut actuel où les nouveaux projets Chloros sont créés. Cliquez sur l&#x27;icône Modifier pour sélectionner un autre répertoire.
* **Quand modifier** :
  * Définissez un lecteur réseau pour la collaboration en équipe.
  * Passez à un lecteur offrant plus d&#x27;espace de stockage pour les ensembles de données volumineux.
  * Organisez les projets par année, client ou type de projet dans différents dossiers.
* **Remarque** : la modification de ce paramètre n&#x27;affecte que les NOUVEAUX projets. Les projets existants restent à leur emplacement d&#x27;origine.

***

## Persistance des paramètres

Tous les paramètres du projet sont automatiquement enregistrés avec votre fichier de projet (format de projet `.mapir`). Lorsque vous rouvrez un projet, tous les paramètres sont restaurés exactement tels que vous les avez laissés.

### Hiérarchie des paramètres

Les paramètres sont appliqués dans l&#x27;ordre suivant :

1. **Paramètres par défaut du système** - Paramètres par défaut intégrés définis par Chloros
2. **Paramètres du modèle** - Si vous chargez un modèle lors de la création d&#x27;un projet
3. **Paramètres du projet enregistrés** - Paramètres enregistrés avec le fichier de projet
4. **Ajustements manuels** - Toutes les modifications que vous apportez pendant la session en cours

### Paramètres et traitement des images

La plupart des modifications de paramètres (en particulier dans les catégories Traitement et Exportation) déclenchent un nouveau traitement des images afin de refléter les nouveaux paramètres. Cependant, certains paramètres sont « réservés à l&#x27;exportation » et ne nécessitent pas de nouveau traitement immédiat :

* Enregistrer le modèle de projet
* Répertoire de travail
* Format d&#x27;image calibré (s&#x27;applique lors de l&#x27;exportation)

***

## Meilleures pratiques

1. **Commencez avec les paramètres par défaut** : les paramètres par défaut fonctionnent bien pour la plupart des systèmes de caméras MAPIR et des flux de travail classiques.
2. **Créez des modèles** : une fois que vous avez optimisé les paramètres pour un flux de travail ou une caméra spécifique, enregistrez-les sous forme de modèle afin de garantir la cohérence entre les projets.
3. **Testez avant le traitement complet** : lorsque vous testez de nouveaux paramètres, effectuez des essais sur un petit sous-ensemble d&#x27;images avant de traiter l&#x27;ensemble de vos données.
4. **Documentez vos paramètres** : utilisez des noms de modèles descriptifs qui indiquent le système de caméra, le type de traitement et l&#x27;utilisation prévue (par exemple, « Survey3\_RGB\_NDVI\_Agriculture »).
5. **Sélection du format d&#x27;exportation** : choisissez votre format d&#x27;exportation en fonction de votre utilisation finale :
   * Analyse scientifique → TIFF (16 bits ou 32 bits)
   * Traitement SIG → TIFF (16 bits)
   * Visualisation rapide → PNG (8 bits)
   * Partage Web → JPG (8 bits)

***

Pour plus d&#x27;informations sur les indices multispectraux dans Chloros, consultez la page [Formules d&#x27;indices multispectraux](multispectral-index-formulas.md).
