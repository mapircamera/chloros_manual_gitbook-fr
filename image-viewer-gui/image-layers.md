# Couches d'images

La liste déroulante des couches d'images de la visionneuse d'images Chloros vous permet de passer rapidement d'une version à l'autre de la même image - des captures originales aux sorties de réflectance traitées et aux images d'index calculées.

## Qu'est-ce qu'une couche d'image ?

Dans Chloros, **les couches** font référence aux différentes sorties d'images disponibles pour une seule image source. Lorsque vous traitez des images, Chloros crée plusieurs versions :

* **Images originales** (fichiers JPG et RAW de votre appareil photo)
* **Sorties calibrées en fonction de la réflectance** (si la calibration de la réflectance a été activée)
* **Images cibles** (si l'image contient des cibles d'étalonnage)
* **Images d'index** (NDVI, NDRE, GNDVI, etc. si les index ont été configurés)

La liste déroulante **Sélecteur de couches** en haut à droite de la visionneuse d'images vous permet de passer instantanément d'une version à l'autre sans quitter la visionneuse.

***

## Types de calques disponibles

### JPG

* L'image JPG originale de votre appareil photo
* Toujours disponible pour toutes les images
* Non traitée, telle qu'elle a été capturée par l'appareil photo
* Plus rapide à charger et à afficher

**Quand regarder:**

* Aperçu rapide de la capture originale
* Vérification de la composition et du cadrage de l'image
* Vérification de la qualité de la capture avant le traitement

### RAW (Original)

* Les données originales du capteur RAW de votre appareil photo
* Décomposées sans post-traitement
* Profondeur de bits supérieure à celle des JPG (généralement 12 ou 14 bits pour les données du capteur)

**Quand regarder:**

* Inspection de la qualité des données originales du capteur
* Vérification des problèmes ou artefacts du capteur
* Comparaison des résultats avant/après traitement

### RAW (cible)

* Apparaît uniquement pour les images identifiées comme contenant des cibles d'étalonnage
* Affiche l'image RAW originale avec la cible détectée
* Utilisé pour vérifier que la détection de la cible a réussi

**Quand visualiser:**

* Confirmation de la détection correcte des cibles d'étalonnage
* Vérification de la qualité de l'image de la cible
* Résolution des problèmes d'étalonnage

{% hint style="info" %}
**Target Layer**: This layer only appears in the dropdown for images that contain calibration targets. Regular capture images will not have this option.
{% endhint %}

### RAW (Réflectance)

* L'image de sortie de la réflectance calibrée
* Vignette corrigée (si activée lors du traitement)
* Réflectance calibrée à l'aide de données cibles (si activée)
* Multibande TIFF avec tous les canaux de la caméra
* Les valeurs des pixels représentent le pourcentage de réflectance (en mode pourcentage)
* Prêt à être manipulé avec le [Index/LUT Sandbox](index-lut-sandbox.md)

**Quand consulter:**

* Inspection des résultats calibrés
* Vérification de la qualité de l'étalonnage
* Vérification de la précision scientifique des valeurs de pixels
* Comparaison avec l'original pour voir les effets de l'étalonnage

{% hint style="success" %}
**Recommended**: Use RAW (Reflectance) layer when checking pixel values for scientific measurements and analysis.
{% endhint %}

### RAW (NDVI Index)... et similaires

* Image de l'indice de végétation calculé (NDVI dans cet exemple)
* Le nom de l'index change en fonction de l'index configuré lors du traitement
* Exemples : RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index), etc.
* Image en niveaux de gris à une bande montrant les résultats du calcul de l'index
* Une couche apparaît pour chaque index configuré dans les paramètres du projet

**Noms d'index possibles:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* Et bien d'autres encore... (voir [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md))

**Quand voir:**

* Examen des résultats du calcul de l'indice
* Vérification des plages de valeurs de l'indice
* Identification des zones d'intérêt
* Vérification des images d'index avant leur utilisation dans un SIG ou une analyse

***

## Utilisation du sélecteur de couches

### Ouverture de la liste déroulante

1. Ouvrez une image en mode plein écran (cliquez sur n'importe quelle vignette dans la visionneuse d'images)
2. Localisez la liste déroulante **couche** dans le coin supérieur droit de la visionneuse
3. La liste déroulante affiche le calque actuellement sélectionné (par exemple, "JPG")
4. Cliquez sur la liste déroulante pour voir tous les calques disponibles

### Passer d'un calque à l'autre

1. Cliquez sur la liste déroulante des couches pour ouvrir la liste
2. Tous les calques disponibles pour l'image en cours sont affichés
3. Cliquez sur le nom d'un calque pour passer à cette version
4. L'image est immédiatement mise à jour pour afficher le calque sélectionné

**Quick switching:**

* La liste déroulante se souvient de votre dernière sélection
* Lorsque vous passez à l'image suivante, Chloros tente d'afficher le même type de calque
* Si ce calque n'existe pas sur l'image suivante, il est affiché par défaut en JPG

### Disponibilité des calques

Tous les calques ne sont pas disponibles pour toutes les images :

**Toujours disponibles:**

* jPG (chaque image a un aperçu JPG)

**Conditionnellement disponible:**

* ⚠️ RAW (Original) - Uniquement si l'image a été capturée en mode RAW ou RAW+JPG
* ⚠️ RAW (Target) - Uniquement si l'image contient des cibles d'étalonnage détectées
* ⚠️ RAW (Reflectance) - Uniquement après traitement avec étalonnage de la réflectance activé
* ⚠️ RAW (\[Index] Index) - Uniquement après le traitement avec les indices configurés

***

## Persistance de la couche

### Naviguer entre les images

Lorsque vous naviguez vers une autre image (en utilisant les touches fléchées ou en cliquant sur les vignettes) :

**La préférence de couche est préservée:**

* Si vous visualisez "RAW (Reflectance)", l'image suivante affiche "RAW (Reflectance)" (si disponible)
* Si vous visualisez "RAW (NDVI Index)", l'image suivante affiche "RAW (NDVI Index)" (si disponible)
* Si le même calque n'existe pas, la valeur par défaut est JPG

**Exemple de flux de travail:**

1. Ouvrez l'image 1, passez en mode RAW (NDVI Index)
2. Appuyez sur → pour afficher l'image 2
3. L'image 2 affiche automatiquement la couche RAW (NDVI Index)
4. Continuez à naviguer - toutes les images affichent la couche NDVI
5. Très efficace pour examiner les résultats de l'index sur de nombreuses images

***

## Flux de travail communs

### Workflow 1 : Comparaison avant/après

**Objectif** : Comparer l'image originale et l'image calibrée

1. Ouvrir l'image traitée dans la visionneuse d'images
2. Sélectionner **RAW (original)** dans la liste déroulante
3. Notez le vignettage et les valeurs non calibrées
4. Sélectionner **RAW (Reflectance)** dans la liste déroulante
5. Comparer - vignettage supprimé, valeurs calibrées

### Flux de travail 2 : Révision de l'index

**Objectif** : Examiner rapidement les résultats NDVI dans l'ensemble du jeu de données

1. Ouvrir la première image traitée
2. Sélectionner **RAW (NDVI Index)** dans la liste déroulante
3. Utiliser la touche fléchée → pour passer à l'image suivante
4. la couche NDVI persiste automatiquement
5. Continuer à parcourir toutes les images, en vérifiant les motifs NDVI
6. Passer à **RAW (NDRE Index)** pour comparer

### Flux de travail 3 : Vérification de la cible

**Objectif** : Vérifier que toutes les images cibles ont été détectées correctement

1. Naviguer vers une image cible
2. Sélectionner **RAW (cible)** dans la liste déroulante
3. Vérifier que les cibles d'étalonnage sont clairement visibles et détectées
4. Naviguer vers l'image de la cible suivante
5. Répéter la vérification pour toutes les cibles

### Flux de travail 4 : Inspection de la valeur des pixels

**Objectif** : Vérifier la précision scientifique des valeurs de réflectance

1. Ouvrir l'image traitée
2. Sélectionner la couche **RAW (Réflectance)**
3. Activer le mode **Pourcentage de pixels** (bouton dans la barre d'outils en haut à droite)
4. Déplacer le curseur sur les zones de végétation
5. Vérifier que les valeurs des pixels se situent dans les fourchettes attendues (30-70% pour NIR, 5-15% pour Red)
6. Vérifier que les valeurs des zones de sol et d'eau sont appropriées

***

## Comprendre les valeurs des pixels par couche

Des couches différentes affichent des plages de valeurs de pixels différentes :

### Couche JPG

* **Plage** : 0-255 (8 bits)
* **Signification Valeurs d'affichage, corrigées en fonction du gamma
* **Utilisation** : Inspection visuelle uniquement, pas de mesure scientifique

### RAW (Original)

* **Plage** : 0-65535 (16 bits)
* **Signification Numéros numériques bruts du capteur
* **Utilisation Vérification des performances du capteur, non calibré

### RAW (Réflectance)

* **Plage** : 0-65 535 (16 bits TIFF) ou 0,0-1,0 (32 bits - pourcentage)
* **Signification Pourcentage de réflectance calibrée
* **Utilisation** : Mesures et analyses scientifiques

**Pour 16-bit TIFF:** Diviser par 65,535 pour obtenir le pourcentage de réflectance **Pour 32-bit Percent:** Les valeurs représentent directement le pourcentage (0.5 = 50% de réflectance)

### RAW (images d'index)

* **Plage** : Varie en fonction de l'indice (généralement de -1,0 à +1,0 pour les indices normalisés)
* **Signification Résultat du calcul de l'indice
* **Exemples** :
  * NDVI : -1 à +1 (végétation typiquement 0,4 à 0,9)
  * NDRE : -1 à +1 (détection du stress)
  * EVI : 0 à 1 (végétation améliorée)

***

## Conseils et bonnes pratiques

### Commutation de couche efficace

* **Sensibilisation aux raccourcis clavier** : Bien qu'il n'y ait pas de raccourci clavier pour les calques, les flèches de navigation (←/→) fonctionnent sur tous les calques
* **Des flux de travail cohérents** : Choisissez une couche (par exemple, NDVI) et examinez l'ensemble du jeu de données avant de passer à une autre couche
* **Comparaisons rapides** : Comparaisons rapides** : basculez entre l'original et la réflectance pour vérifier la qualité du traitement

### Considérations sur les performances

* **Le JPG se charge le plus rapidement** : A utiliser pour une navigation rapide dans de nombreuses images
* **Les couchesRAW se chargent plus lentement** : Résolution et profondeur de bits plus élevées
* **Calques d'index** : Vitesse similaire à celle des calques de réflectance
* **Le premier chargement est le plus lent** : Les vues suivantes d'une même couche sont mises en cache et plus rapides

### Vérification de la qualité

* **Toujours vérifier le RAW (original)** : Vérifiez la qualité des données sources avant de vous fier aux résultats traités
* **Comparer les couches** : Comparer les couches** : utiliser le changement de couche pour vérifier que le traitement a fonctionné correctement
* **Vérifiez les plages d'index** : Vérifier les plages d'index** : utiliser le mode "Pourcentage de pixels" avec les couches d'index pour vérifier que les valeurs sont raisonnables

***

## Dépannage

### Couche non disponible

**Problème** : Le calque attendu n'apparaît pas dans la liste déroulante

**Causes possibles:**

* L'image n'a pas été traitée (seules les images JPG et RAW (Original) sont disponibles)
* L'étalonnage de la réflectance a été désactivé pendant le traitement
* L'index spécifique n'a pas été configuré dans les paramètres du projet
* L'image est une image de cible uniquement (aucun index n'est généré pour les cibles)

**Solutions:**

1. Vérifier que l'image a été traitée (vérifier le dossier de sortie pour les fichiers traités)
2. Vérifier les paramètres du projet pour confirmer que les indices ont été configurés
3. Retraiter avec les indices souhaités activés

### Mauvaise couche affichée

**Problème** : L'image s'ouvre dans un calque inattendu

**Cause** : La préférence de calque de l'image précédente a été reportée, mais ce calque n'existe pas sur l'image actuelle

**Solution** : Chloros revient automatiquement au format JPG lorsque le calque préféré n'est pas disponible - il s'agit d'un comportement normal

### Impossible de voir les cibles d'étalonnage

**Problème** : La couche RAW (cible) n'affiche pas la détection de la cible

**Causes possibles:**

* Les cibles n'ont pas été détectées pendant le traitement
* L'image ne contient pas réellement de cibles
* Les paramètres de détection des cibles sont trop stricts

**Solutions:**

1. Vérifier le journal de débogage pour les messages "Cible trouvée"
2. Vérifier que l'image contient effectivement des cibles d'étalonnage visibles
3. Ajuster les paramètres de détection des cibles dans les paramètres du projet
4. Voir [Choosing Target Images](../processing-images-gui/choosing-target-images.md)

***

## Fonctionnalités connexes

### Outils de visualisation d'images

Lors de la visualisation d'un calque, vous pouvez utiliser :

* **Commandes de zoom** : Agrandir pour inspecter les détails
* **Pan** : Cliquer et faire glisser pour se déplacer autour de l'image zoomée
* **Inspection de la valeur des pixels** : Voir les valeurs à l'emplacement du curseur
* **Flèches de navigation** : Flèches de navigation** : Déplacez-vous entre les images tout en maintenant la couche
* **Mode pourcentage de pixels** : Basculer entre l'affichage des DN et des pourcentages

Voir [Opening an Image Full Screen](opening-an-image-full-screen.md) pour la documentation complète de la visionneuse d'images.

### Index/LUT Sandbox

Pour les tests interactifs d'index et la visualisation :

* **Calcul d'indice en temps réel** : Testez différentes formules d'index
* **Mappage des couleurs LUT** : Appliquez des gradients de couleur à des indices en niveaux de gris
* **Exporter des visualisations** : Sauvegarde des images d'index colorés

Voir [Index/LUT Sandbox](index-lut-sandbox.md) pour plus de détails.

***

## Prochaines étapes

Maintenant que vous comprenez ce que sont les calques d'image :

* [**Opening an Image Full Screen**](opening-an-image-full-screen.md) - Guide complet de la visionneuse d'images
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualisation interactive des index
* [**Multispectral Index Formulas**](../project-settings/multispectral-index-formulas.md) - Référence des indices disponibles
* [**Finishing the Processing**](../processing-images-gui/finishing-the-processing.md) - Comprendre les sorties traitées
