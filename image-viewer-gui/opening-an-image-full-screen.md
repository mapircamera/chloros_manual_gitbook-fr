# Ouverture d'une image en plein écran

La Chloros visionneuse d'images offre une interface plein écran dédiée à la visualisation, à l'analyse et à la manipulation de vos images multispectrales. Qu'il s'agisse d'images originales ou de sorties traitées, la visionneuse d'images offre des outils puissants pour l'inspection et l'analyse.

## Accès à la visionneuse d'images

### Depuis le navigateur de fichiers

La façon la plus courante d'ouvrir une image dans la visionneuse d'images :

1. Assurez-vous d'être dans l'onglet **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Cliquez sur n'importe quelle **vignette d'image** dans la grille d'images
3. L'image s'ouvre dans la **zone de prévisualisation principale** (au centre de l'écran)
4. L'image est maintenant chargée et prête à être visualisée en plein écran

### Ouverture de l'onglet de la visionneuse d'images

Une fois l'image chargée dans la zone de prévisualisation :

1. Cliquez sur l'icône **Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> dans la barre latérale gauche
2. L'onglet Visionneuse d'images s'ouvre et affiche l'image sélectionnée en plein écran
3. Des outils avancés de visualisation et d'analyse sont disponibles dans la barre latérale gauche

***

## Vue d'ensemble de l'interface de la visionneuse d'images

### Zone d'affichage principale

La plus grande partie de l'écran affiche votre image :

* **Résolution complète** : Images affichées à la résolution native
* **Zoomable** : Utilisez les commandes ou la molette de la souris pour zoomer
* **Possibilité de cliquer et de faire glisser la souris pour se déplacer lorsque l'on fait un zoom Cliquez et faites glisser pour vous déplacer lorsque le zoom est effectué
* **Le rapport d'aspect est maintenu : Les images sont mises à l'échelle de manière proportionnelle

***

## Options de visualisation

### Navigation de base dans les images

#### Parcourir les images

Naviguez dans votre série d'images à l'aide de raccourcis clavier ou de boutons :

* **Image suivante** : Cliquez sur le bouton → ou appuyez sur la touche **→** (flèche droite)
* image précédente** : Cliquez sur le bouton → ou appuyez sur la touche **←** (flèche droite) : Cliquez sur le bouton ← ou appuyez sur la touche **←** (flèche gauche)
* **Sauter à une image spécifique** : Retournez au navigateur de fichiers et cliquez sur la vignette souhaitée

#### Commandes de zoom

Ajuster l'agrandissement pour inspecter les détails de l'image :

**Zoom In:**

* Cliquez sur le bouton **+** (Plus)
* Appuyez sur la touche **+** ou **=**
* Faire défiler la molette de la souris **vers le haut**

**Zoom arrière:**

* Cliquez sur le bouton **-** (Moins)
* Appuyer sur la touche **-** (Moins)
* Faire défiler la molette de la souris **vers le bas**

**Adaptation à l'écran:**

* Cliquez sur le bouton **↔** (Ajuster)
* Appuyez sur la touche **0** (Zéro)
* Double-cliquez sur l'image

#### Panoramique en cas de zoom

Lorsque l'image est agrandie au-delà de la taille de l'écran :

1. Déplacer le curseur de la souris sur l'image
2. Cliquer et **maintenir le bouton gauche de la souris**
3. **Glisser** pour déplacer l'image
4. Relâcher pour arrêter le panoramique

**Alternative** : Utilisez les touches fléchées pour effectuer un panoramique par petits incréments

***

## Inspection de la valeur du pixel

### Visualisation des valeurs de pixels au niveau du curseur

Lorsque vous déplacez le curseur de votre souris sur l'image, les valeurs des pixels s'affichent en temps réel :

**L'emplacement de l'affichage de la valeur:**

* **Nombre flottant et ligne rouge dans la légende du gradient de la LUT de l'index de droite
* **En cas d'agrandissement, valeur flottante près du curseur et du pixel en surbrillance**
* Affiche les valeurs pour le pixel **sous le curseur ou en surbrillance**
* Se met à jour lorsque vous déplacez la souris

***

## Types d'images que vous pouvez visualiser

### Images originales (prétraitement)

**Images RAW + JPG de l'appareil photo:**

* Afficher les données RAW sous forme de prévisualisation
* Afficher les valeurs originales non corrigées
* Utile pour vérifier la qualité de l'image avant le traitement

### Images de réflectance calibrées

**Après traitement:**

* Vignette corrigée
* Réflectance calibrée
* Multi-bandes TIFF (Red, Green, NIR, etc.)
* Données scientifiques prêtes à être analysées

### Index Images

**NDVI, NDRE, GNDVI, etc. (fichiers \_NDVI.tif):**

* Images en niveaux de gris à une bande
* Les valeurs des pixels représentent les résultats du calcul de l'indice
* Plage typique de -1 à +1 pour les indices normalisés
* Possibilité d'appliquer des LUT de couleur pour la visualisation

***

## Application d'indices et de LUT

Appliquer des indices multispectraux et des tables de correspondance des couleurs :

1. Localiser **Index/LUT Sandbox** dans **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebar
2. Sélectionner l'indice de végétation (NDVI, NDRE, etc.)
3. Sélectionnez une formule multispectrale ou créez-en une personnalisée (Chloros+ uniquement)
4. Appliquer le gradient de la LUT de couleur pour la visualisation
5. Ajuster les plages de valeurs et les seuils

Voir [Index/LUT Sandbox](index-lut-sandbox.md) pour des instructions détaillées.

***

## Raccourcis clavier

### Navigation

* **→** (flèche droite) : Image suivante
* **←** (flèche gauche) : Image précédente
* **Home** : Première image de la liste
* **Fin** : Dernière image de la liste

### Zoom

* **+** ou **=** : Zoom avant
* **-** : Zoom arrière
* **0** (Zéro) : Ajuster à l'écran
* **Molette de la souris** : Zoom avant/arrière

### Contrôles d'affichage

* **P** : Basculer le mode pourcentage de pixels
* **L** : Basculer le panneau des calques
* **Esc** : Fermer le plein écran ou revenir au navigateur de fichiers

### Autres

* **Ctrl+S** : Enregistrer l'image actuelle
* **F** : Mode plein écran (si disponible)

***

### Vérification des calculs d'indices

Vérifier que les indices ont été calculés correctement :

1. Ouvrir NDVI ou une autre image d'index
2. Vérifier les zones de végétation :
   * **NDVI**: Doit afficher 0,4-0,9 pour les plantes saines
   * **NDRE**: Des valeurs plus élevées pour une croissance vigoureuse
   * **GNDVI**: Similaire à NDVI mais sensible à la chlorophylle
3. Vérifier la non-végétation :
   * **Sol** : Sol** : proche de 0 ou légèrement négatif
   * **Eau** : Valeurs négatives (-0,5 à 0)

***

## Résolution des problèmes de visualisation

### L'image ne s'ouvre pas

**Causes possibles:**

* Fichier corrompu pendant le traitement
* Format de fichier non pris en charge
* Mémoire insuffisante pour une image de grande taille

**Solutions:**

1. Essayer d'ouvrir l'image dans une visionneuse externe pour vérifier l'intégrité du fichier
2. Vérifier que le format du fichier correspond au type attendu
3. Fermer les autres applications pour libérer de la mémoire
4. Essayer une image plus petite/différente

### Affichage d'une image en noir ou blanc

**Causes possibles:**

* Plage de valeurs en dehors de la capacité d'affichage
* image flottante 32 bits avec des valeurs inhabituelles
* Erreur de calcul de l'index

**Solutions:**

1. Vérifier les valeurs des pixels - si elles sont toutes très basses ou très hautes, ajuster la plage d'affichage
2. Essayer d'ouvrir le fichier dans QGIS ou un logiciel similaire avec ajustement automatique de la plage d'affichage
3. Vérifier le journal de débogage du traitement pour les erreurs

### Les valeurs des pixels semblent erronées

**Causes possibles:**

* Visualisation d'une mauvaise image (originale ou traitée)
* L'étalonnage n'a pas été appliqué correctement
* Les données du capteur de lumière n'ont pas été incluses dans l'entrée
* Le mode pourcentage n'a pas été basculé correctement

**Solutions:**

1. Vérifiez que vous visualisez bien la sortie traitée (vérifiez le suffixe du nom de fichier)
2. Vérifier l'état du bouton du mode pourcentage
3. Comparer avec des images connues du même jeu de données

***

## Prochaines étapes

Vous pouvez maintenant afficher les images en plein écran :

* [**Calques d'images**](image-layers.md) - Découvrez la visualisation multibande
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Appliquez des indices et des cartographies de couleurs personnalisés
* [Formules d'indices multi-spectraux**](../project-settings/multispectral-index-formulas.md) - Comprendre les indices disponibles

Pour le processus de traitement, voir :

* [**Traitement des images (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Guide de traitement complet
