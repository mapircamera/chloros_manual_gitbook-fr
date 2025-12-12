# Paramètres du projet

La barre latérale Paramètres du projet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> dans Chloros vous permet de configurer tous les aspects du traitement de l'image, de la détection des cibles d'étalonnage, des calculs de l'indice multispectral et des options d'exportation pour votre projet. Ces paramètres sont enregistrés avec votre projet et peuvent être sauvegardés en tant que modèles pour être réutilisés dans plusieurs projets.

## Accès aux paramètres du projet

Pour accéder aux paramètres du projet :

1. Ouvrez un projet dans Chloros
2. Cliquez sur l'onglet **Paramètres du projet** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau des paramètres affiche toutes les options de configuration disponibles, organisées par catégorie

***

## Détection de la cible

Ces paramètres contrôlent la façon dont Chloros détecte et traite les cibles d'étalonnage dans vos images.

### Zone minimale de l'échantillon d'étalonnage (px)

* **Type** : Nombre
* **Range** : 0 à 10 000 pixels
* **Par défaut** : 25 pixels
* **Description Définit la surface minimale (en pixels) requise pour qu'une région détectée soit considérée comme un échantillon de cible d'étalonnage valide. Des valeurs plus petites permettent de détecter des cibles plus petites, mais peuvent augmenter le nombre de faux positifs. Des valeurs plus élevées nécessitent des régions cibles plus grandes et plus claires pour la détection.
* **Quand ajuster** :
  * Augmentez la valeur si vous obtenez de fausses détections sur de petits artefacts d'image
  * Diminuez la valeur si vos cibles d'étalonnage apparaissent petites dans vos images et ne sont pas détectées

### Regroupement minimal des cibles (0-100)

* **Type** : Nombre
* **Plage** : 0 à 100
* **Par défaut** : 60
* **Description Contrôle le seuil de regroupement des régions de couleur similaire lors de la détection des cibles d'étalonnage. Des valeurs plus élevées nécessitent de regrouper davantage de couleurs similaires, ce qui entraîne une détection plus prudente des cibles. Des valeurs plus faibles permettent une plus grande variation des couleurs au sein d'un groupe de cibles.
* **Quand ajuster** :
  * Augmentez la valeur si les cibles d'étalonnage sont divisées en plusieurs détections
  * Diminuer si les cibles d'étalonnage présentant des variations de couleur ne sont pas entièrement détectées

***

## Traitement

Ces paramètres contrôlent la façon dont Chloros traite et étalonne vos images.

### Correction de la vignette

* **Type** : Case à cocher
* **Défaut** : Activé (coché)
* **Description** : Applique une correction de vignette pour compenser l'assombrissement de l'objectif sur les bords des images. Le vignettage est un phénomène optique courant où les coins et les bords d'une image apparaissent plus sombres que le centre en raison des caractéristiques de l'objectif.
* **Quand désactiver** : Ne désactivez cette fonction que si votre combinaison appareil photo/objectif a déjà appliqué la correction du vignettage, ou si vous souhaitez corriger manuellement le vignettage lors du post-traitement.

### Calibration de la réflectance / balance des blancs

* **Type** : Case à cocher
* **Défaut** : Activé (coché)
* **Description** : Active l'étalonnage automatique de la réflectance en utilisant les cibles d'étalonnage détectées dans vos images. Cela permet de normaliser les valeurs de réflectance dans l'ensemble de vos données et de garantir des mesures cohérentes quelles que soient les conditions d'éclairage.
* **Quand désactiver** : Ne désactivez cette option que si vous souhaitez traiter des images brutes, non calibrées, ou si vous utilisez un flux de travail de calibration différent.

### Méthode Debayer

* **Type** : Sélection déroulante
* **Options** :
  * Haute qualité (plus rapide) - Actuellement la seule option disponible
* **Par défaut** : Haute qualité (plus rapide)
* **Description** : Sélectionne l'algorithme de démosaïquage utilisé pour convertir les données brutes du capteur de la mire de Bayer en images en couleurs. La méthode "Haute qualité (plus rapide)" offre un équilibre optimal entre la vitesse de traitement et la qualité de l'image.
* **Remarque** : D'autres méthodes de démosaïquage pourront être ajoutées dans les versions futures de Chloros.

### Intervalle de recalibrage minimum

* **Type** : Nombre
* **Plage** : 0 à 3 600 secondes
* **Par défaut** : 0 seconde
* **Description Définit l'intervalle de temps minimum (en secondes) entre l'utilisation des cibles d'étalonnage. Lorsqu'il est réglé sur 0, Chloros utilise chaque cible de calibrage détectée. Si la valeur est plus élevée, Chloros n'utilisera que les cibles d'étalonnage séparées d'au moins ce nombre de secondes, ce qui réduit le temps de traitement des ensembles de données avec des captures fréquentes de cibles d'étalonnage.
* **Quand ajuster** :
  * Régler à 0 pour une précision d'étalonnage maximale lorsque les conditions d'éclairage varient
  * Augmentez la valeur (par exemple, entre 60 et 300 secondes) pour accélérer le traitement lorsque l'éclairage est constant et que les images de cibles d'étalonnage sont fréquentes

### Décalage du fuseau horaire du capteur de lumière

* **Type** : Nombre
* **Range** : -12 à +12 heures
* **Par défaut** : 0 heure
* **Description Spécifie le décalage du fuseau horaire (en heures par rapport à UTC) pour les horodatages des données du capteur de lumière. Ce décalage est utilisé lors du traitement des fichiers de données PPK (Post-Processed Kinematic) pour assurer une synchronisation temporelle correcte entre les captures d'images et les données GPS.
* **Quand ajuster** : Réglez ce paramètre sur le décalage de votre fuseau horaire local si vos données PPK utilisent l'heure locale au lieu de l'heure UTC. Par exemple :
  * Heure du Pacifique : -8 ou -7 (en fonction de l'heure d'été)
  * Heure de l'Est : -5 ou -4 (selon l'heure d'été)
  * Heure d'Europe centrale : +1 ou +2 (selon l'heure d'été)

### Appliquer les corrections PPK

* **Type** : Case à cocher
* **Défaut** : Désactivé (non coché)
* **Description** : Active l'utilisation des corrections cinématiques post-traitées (PPK) des enregistreurs DAQ MAPIR contenant un GPS (GNSS). Lorsque cette option est activée, Chloros utilise tous les fichiers journaux .daq contenant des données d'exposition dans le répertoire de votre projet et applique des corrections de géolocalisation précises à vos images.
* **Exigences** : un fichier journal .daq contenant des entrées de pin d'exposition doit être présent dans le répertoire de votre projet
* **Quand l'activer ? Il est recommandé de toujours activer la correction PPK si vous avez des entrées de retour d'exposition dans votre fichier journal .daq.

### Broche d'exposition 1

* **Type** : Sélection déroulante
* **Visibilité** : Visible uniquement lorsque l'option "Appliquer les corrections PPK" est activée ET que des données d'exposition sont disponibles pour la broche 1
* **Options** :
  * Noms des modèles de caméra détectés dans le projet
  * "Ne pas utiliser" - Ignorer cette broche d'exposition
* **Par défaut** : Sélection automatique en fonction de la configuration du projet
* **Description Attribue une caméra spécifique à la broche d'exposition 1 pour la synchronisation temporelle du PPK. La broche d'exposition enregistre le moment exact où l'obturateur de la caméra est déclenché, ce qui est essentiel pour une géolocalisation précise du PPK.
* **Comportement de sélection automatique** :
  * Une seule caméra + une seule broche : Sélection automatique de l'appareil photo
  * Caméra unique + deux broches : La broche 1 est automatiquement attribuée à la caméra
  * Plusieurs caméras : Sélection manuelle nécessaire

### Broche d'exposition 2

* **Type** : Sélection déroulante
* **Visibilité** : Visible uniquement lorsque l'option "Appliquer les corrections PPK" est activée ET que des données d'exposition sont disponibles pour la broche 2
* **Options** :
  * Noms des modèles de caméra détectés dans le projet
  * "Ne pas utiliser" - Ignorer cette broche d'exposition
* **Par défaut** : Sélection automatique en fonction de la configuration du projet
* **Description Attribue une caméra spécifique à la broche d'exposition 2 pour la synchronisation du temps PPK lors de l'utilisation d'une configuration à deux caméras.
* **Comportement de l'auto-sélection** :
  * Une seule caméra + une seule broche : La broche 2 est automatiquement réglée sur "Ne pas utiliser"
  * Caméra unique + deux broches : La broche 2 est automatiquement réglée sur "Ne pas utiliser"
  * Plusieurs caméras : Sélection manuelle requise
* **Remarque** : La même caméra ne peut pas être affectée simultanément à la broche 1 et à la broche 2.

***

## Index

Ces paramètres vous permettent de configurer des indices multispectraux pour l'analyse et la visualisation.

### Ajouter un indice

* **Type** : Panneau spécial de configuration d'index
* **Description** : Ouvre un panneau interactif dans lequel vous pouvez sélectionner et configurer des indices de végétation multispectraux (NDVI, NDRE, EVI, etc.) à calculer pendant le traitement de l'image. Vous pouvez ajouter plusieurs indices, chacun avec ses propres paramètres de visualisation.
* **Indices disponibles** : Le système comprend plus de 30 indices multispectraux prédéfinis :
  * NDVI (Indice de végétation par différence normalisée)
  * NDRE (Différence normalisée RedEdge)
  * EVI (Indice de végétation amélioré)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Et bien d'autres encore (voir [Multispectral Index Formulas](multispectral-index-formulas.md) pour une liste complète)
* **Caractéristiques** :
  * Sélection de formules d'index prédéfinies
  * Configurer les gradients de couleur de visualisation (LUT - Look-Up Tables)
  * Définir des valeurs seuils pour l'analyse
  * Créer des formules d'indexation personnalisées

### Formules personnalisées (Chloros+ Fonctionnalité)

* **Type** : Tableau de définitions de formules personnalisées
* **Description Permet de créer et d'enregistrer des formules d'index multispectrales personnalisées à l'aide de la mathématique des bandes. Les formules personnalisées sont enregistrées avec les paramètres de votre projet et peuvent être utilisées comme les indices intégrés.
* **Comment créer** :
  1. Dans le panneau de configuration de l'index, recherchez l'option de formule personnalisée
  2. Définissez votre formule en utilisant des identifiants de bande (par exemple, NIR, Red, Green, Blue)
  3. Enregistrez la formule avec un nom descriptif
* **Syntaxe de la formule** : Les opérations mathématiques standard sont prises en charge, notamment
  * Arithmétique : `+`, `-`, `*`, `/`
  * Parenthèses pour l'ordre des opérations
  * Références de la bande : NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Ces paramètres contrôlent le format et la qualité des images traitées exportées.

### Format de l'image calibrée

* **Type** : Sélection déroulante
* **Options** :
  * **TIFF (16 bits)** - Format 16 bits non compressé
  * **TIFF (32 bits, Pourcentage)** - Format 32 bits à virgule flottante TIFF avec des valeurs de réflectance sous forme de pourcentages
  * **PNG (8 bits)** - Format 8 bits compressé PNG
  * **JPG (8-bit)** - Format 8-bit compressé JPEG (8-bit)** - Format 8-bit compressé JPEG (8-bit)**
* **Par défaut** : TIFF (16 bits)
* **Description** : Sélectionne le format de fichier pour l'enregistrement des images traitées et calibrées.
* **Recommandations de format** :
  * **TIFF (16-bit)**: Recommandé pour les analyses scientifiques et les flux de travail professionnels. Préserve la qualité maximale des données sans artefacts de compression. Idéal pour l'analyse multispectrale et le traitement ultérieur dans un logiciel SIG.
  * **TIFF (32 bits, pourcentage)** : Idéal pour les flux de travail qui nécessitent des valeurs de réflectance sous forme de pourcentages (0-100 %). Offre une précision maximale pour les mesures radiométriques.
  * **PNG (8-bit)**: Bon pour l'affichage sur le web et la visualisation générale. Les fichiers sont plus petits grâce à la compression sans perte, mais la plage dynamique est réduite.
  * **JPG (8 bits)** : Fichier de taille réduite, idéal pour les prévisualisations et l'affichage sur le web. Utilise une compression avec perte qui n'est pas adaptée à l'analyse scientifique.

***

## Enregistrer le modèle de projet

Cette fonction vous permet d'enregistrer les paramètres de votre projet actuel en tant que modèle réutilisable.

* **Type** : Entrée de texte + bouton Enregistrer
* **Description** : Saisissez un nom descriptif pour votre modèle de paramètres et cliquez sur l'icône d'enregistrement. Le modèle stocke tous les paramètres actuels du projet (détection de la cible, options de traitement, indices et format d'exportation) pour faciliter leur réutilisation dans les projets futurs.
* **Cas d'utilisation** :
  * Créez des modèles pour différents systèmes de caméra (RGB, multispectral, NIR)
  * Sauvegarder des configurations standard pour des types de cultures ou des flux de travail d'analyse spécifiques
  * Partager des paramètres cohérents au sein d'une équipe
* **Comment l'utiliser ?
  1. Configurez tous les paramètres de votre projet
  2. Entrez un nom de modèle (par exemple, "RedEdge Survey3 NDVI Standard")
  3. Cliquez sur l'icône de sauvegarde
  4. Le modèle peut maintenant être chargé lors de la création de nouveaux projets

***

## Enregistrer le dossier du projet

Ce paramètre indique où les nouveaux projets sont enregistrés par défaut.

* **Type** : Affichage du chemin d'accès au répertoire + bouton Modifier
* **Par défaut** : `C:\Users\[Username]\Chloros Projects`
* **Description** : Affiche le répertoire par défaut dans lequel les nouveaux projets Chloros sont créés. Cliquez sur l'icône de modification pour sélectionner un autre répertoire.
* **Quand changer** :
  * Définir un lecteur réseau pour la collaboration entre les membres de l'équipe
  * Changer pour un lecteur avec plus d'espace de stockage pour les grands ensembles de données
  * Organiser les projets par année, par client ou par type de projet dans des dossiers différents
* **Remarque** : La modification de ce paramètre n'affecte que les NOUVEAUX projets. Les projets existants conservent leur emplacement d'origine.

***

## Persistance des paramètres

Tous les paramètres du projet sont automatiquement enregistrés avec votre fichier de projet (`.mapir` format de projet). Lorsque vous rouvrez un projet, tous les paramètres sont restaurés exactement comme vous les avez laissés.

### Hiérarchie des paramètres

Les paramètres sont appliqués dans l'ordre suivant :

1. **Valeurs par défaut du système** - Valeurs par défaut intégrées définies par Chloros
2. **Paramètres du modèle** - Si vous chargez un modèle lors de la création d'un projet
3. **Paramètres du projet enregistrés** - Paramètres enregistrés avec le fichier de projet
4. **Ajustements manuels** - Toutes les modifications que vous apportez pendant la session en cours

### Paramètres et traitement des images

La plupart des changements de paramètres (en particulier dans les catégories Traitement et Exportation) entraîneront un retraitement des images pour refléter les nouveaux paramètres. Cependant, certains paramètres sont "exportés uniquement" et ne nécessitent pas de retraitement immédiat :

* Sauvegarder le modèle de projet
* Répertoire de travail
* Format d'image calibré (s'applique lors de l'exportation)

***

## Bonnes pratiques

1. **Commencez par les paramètres par défaut** : Les paramètres par défaut fonctionnent bien pour la plupart des systèmes de caméras et des flux de travail typiques.
2. **Créer des modèles** : Une fois que vous avez optimisé les paramètres pour un flux de travail ou un appareil photo spécifique, enregistrez-les en tant que modèle pour garantir la cohérence entre les projets.
3. **Testez avant le traitement complet** : Lorsque vous expérimentez de nouveaux paramètres, testez-les sur un petit sous-ensemble d'images avant de traiter l'ensemble de vos données.
4. **Documentez vos paramètres** : Utilisez des noms de modèles descriptifs qui indiquent le système de caméra, le type de traitement et l'utilisation prévue (par exemple, "Survey3\_RGB\_NDVI\_Agriculture").
5. **Sélection du format d'exportation** : Choisissez votre format d'exportation en fonction de votre utilisation finale :
   * Analyse scientifique → TIFF (16 bits ou 32 bits)
   * Traitement SIG → TIFF (16 bits)
   * Visualisation rapide → PNG (8 bits)
   * Partage sur le web → JPG (8 bits)

***

Pour plus d'informations sur les indices multispectraux dans Chloros, voir la page [Multispectral Index Formulas](multispectral-index-formulas.md).
