# Ouvrir une image en plein écran

La visionneuse d&#x27;images Chloros offre une interface dédiée en plein écran pour visualiser, analyser et manipuler vos images multispectrales. Que vous visualisiez des images originales ou des résultats traités, la visionneuse d&#x27;images propose des outils puissants pour l&#x27;inspection et l&#x27;analyse.

## Accéder à la visionneuse d&#x27;images

### À partir du navigateur de fichiers

La méthode la plus courante pour ouvrir une image dans la visionneuse d&#x27;images :

1. Assurez-vous que vous êtes dans l&#x27;onglet **Navigateur de fichiers** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Cliquez sur n&#x27;importe quelle **vignette d&#x27;image** dans la grille d&#x27;images
3. L&#x27;image s&#x27;ouvre dans la **zone d&#x27;aperçu principale** (au centre de l&#x27;écran)
4. L&#x27;image est désormais chargée et prête à être visualisée en plein écran

### Ouverture de l&#x27;onglet Image Viewer

Une fois qu&#x27;une image est chargée dans la zone d&#x27;aperçu :

1. Cliquez sur l&#x27;icône **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> dans la barre latérale gauche
2. L&#x27;onglet Visionneuse d&#x27;images s&#x27;ouvre et affiche l&#x27;image sélectionnée en plein écran
3. Des outils avancés de visualisation et d&#x27;analyse deviennent disponibles dans la barre latérale gauche

***

## Présentation de l&#x27;interface de la visionneuse d&#x27;images

### Zone d&#x27;affichage principale

La plus grande partie de l&#x27;écran affiche votre image :

* **Résolution maximale** : les images s&#x27;affichent en résolution native
* **Zoomable** : utilisez les commandes ou la molette de la souris pour zoomer
* **Panoramique** : cliquez et faites glisser pour vous déplacer lorsque vous êtes en mode zoom
* **Rapport d&#x27;aspect conservé** : les images sont redimensionnées proportionnellement***

## Options d&#x27;affichage

### Navigation de base dans les images

#### Parcourir les images

Naviguez dans votre ensemble d&#x27;images à l&#x27;aide des raccourcis clavier ou des boutons :

* **Image suivante**: cliquez sur le bouton → ou appuyez sur la touche**→** (flèche droite)
* **Image précédente**: cliquez sur le bouton ← ou appuyez sur la touche**←** (flèche gauche)
* **Aller à une image spécifique** : retournez dans le navigateur de fichiers et cliquez sur la vignette souhaitée

#### Commandes de zoom

Ajustez le grossissement pour examiner les détails de l&#x27;image :

**Zoom avant :*** Cliquez sur le bouton **+** (plus)
* Appuyez sur la touche **+**ou**=*** Faites défiler la molette de la souris **vers le haut**

**Zoom arrière :*** Cliquez sur le bouton **−** (moins)
* Appuyez sur la touche **−** (moins)
* Faites défiler la molette de la souris **vers le bas**

#### Panoramique en mode zoom

Lorsque le zoom dépasse la taille de l&#x27;écran :

1. Placez le curseur de la souris sur l&#x27;image
2. Cliquez et **maintenez le bouton gauche de la souris enfoncé**

3.**Faites glisser** pour déplacer l&#x27;image
4. Relâchez pour arrêter le panoramique

**Alternative** : utilisez les touches fléchées pour effectuer un panoramique par petits incréments***

## Inspection des valeurs de pixels

### Affichage des valeurs de pixels au niveau du curseur

Lorsque vous déplacez le curseur de la souris sur l&#x27;image, les valeurs des pixels s&#x27;affichent en temps réel :**Emplacement d&#x27;affichage des valeurs :*** **Chiffre flottant et ligne rouge dans la légende du gradient de la table de conversion (LUT) située à droite*** **Lorsque le zoom est encore plus poussé, valeur flottante près du curseur et du pixel sélectionné*** Affiche les valeurs du pixel **sous le curseur ou sélectionné*** Mise à jour au fur et à mesure que vous déplacez la souris

***

## Types d&#x27;images que vous pouvez visualiser

### JPG

**Images JPG provenant de l&#x27;appareil photo :**

* Affichage des données JPG telles qu&#x27;elles sont prévisualisées
* Affichage des valeurs d&#x27;origine, non corrigées
* Utile pour vérifier la qualité de l&#x27;image avant le traitement

### RAW (Original)

### RAW (Réflectance)

**Après traitement :**

* Vignettage corrigé
* Réflectance calibrée
* Multibande TIFF (Red, Green, NIR, etc.)
* Données scientifiques prêtes à être analysées

### RAW (Index)

**NDVI, NDRE, GNDVI, etc. (fichiers \_NDVI.tif) :**

* Images en niveaux de gris à bande unique
* Les valeurs des pixels représentent les résultats du calcul des indices
* Plage généralement comprise entre -1 et +1 pour les indices normalisés
* Possibilité d&#x27;appliquer des tables de conversion de couleurs (LUT) pour la visualisation

***

## Application des indices et des LUT

Appliquer des indices multispectraux et des tables de conversion de couleurs (LUT) :

1. Localisez **Index/LUT Sandbox**dans**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> barre latérale
2. Sélectionnez un indice de végétation (NDVI, NDRE, etc.)
3. Sélectionnez une formule multispectrale ou créez votre propre formule personnalisée (Chloros+ uniquement)
4. Appliquez un dégradé de LUT de couleurs pour la visualisation
5. Ajustez les plages de valeurs et les seuils

Consultez [Index/LUT Sandbox](index-lut-sandbox.md) pour obtenir des instructions détaillées.

***

## Raccourcis clavier

### Navigation

* **→** (Flèche droite) : Image suivante
* **←** (Flèche gauche) : Image précédente
* **Home** : Première image de la liste
* **Fin** : Dernière image de la liste

### Zoom

* **+**ou**=** : Zoom avant
* **−** : Zoom arrière
* **Molette de la souris** : Zoom avant/arrière***

### Vérification des calculs d&#x27;indices

Vérifiez que les indices ont été calculés correctement :

1. Ouvrez NDVI ou une autre image d&#x27;indice
2. Vérifiez les zones de végétation :
   * **NDVI** : Doit afficher une valeur comprise entre 0,4 et 0,9 pour les plantes saines
   * **NDRE** : Valeurs plus élevées pour une croissance vigoureuse
   * **GNDVI** : similaire à NDVI mais sensible à la chlorophylle
3. Vérifiez les zones non végétalisées :
   * **Sol** : proche de 0 ou légèrement négatif
   * **Eau** : valeurs négatives (-0,5 à 0)***

## Dépannage des problèmes d&#x27;affichage

### L&#x27;image ne s&#x27;ouvre pas

**Causes possibles :**

* Fichier corrompu pendant le traitement
* Format de fichier non pris en charge
* Mémoire insuffisante pour une image volumineuse

**Solutions :**

1. Essayez d&#x27;ouvrir le fichier dans une visionneuse externe pour vérifier l&#x27;intégrité du fichier
2. Vérifiez que le format du fichier correspond au type attendu
3. Fermez les autres applications pour libérer de la mémoire
4. Essayez une image plus petite ou différente

### Affichage de l&#x27;image en noir ou en blanc

**Causes possibles :**

* Plage de valeurs hors de la capacité d&#x27;affichage
* Image en 32 bits flottant avec des valeurs inhabituelles
* Erreur de calcul de l&#x27;indice

**Solutions :**

1. Vérifiez les valeurs des pixels : si elles sont toutes très basses ou très élevées, ajustez la plage d&#x27;affichage
2. Essayez d&#x27;ouvrir le fichier dans QGIS ou un logiciel similaire avec réglage automatique de la plage
3. Vérifiez le journal de débogage du traitement pour détecter d&#x27;éventuelles erreurs

### Les valeurs des pixels semblent incorrectes

**Causes possibles :**

* Affichage d&#x27;une image incorrecte (originale vs traitée)
* L&#x27;étalonnage n&#x27;a pas été appliqué correctement
* Les données du capteur de lumière n&#x27;ont pas été incluses dans les données d&#x27;entrée
* Le mode pourcentage a été activé/désactivé de manière incorrecte

**Solutions :**

1. Vérifiez que vous visualisez bien le résultat traité (vérifiez l&#x27;extension du nom de fichier)
2. Vérifiez l&#x27;état du bouton du mode pourcentage
3. Comparez avec des images connues pour être correctes issues du même jeu de données

***

## Étapes suivantes

Maintenant que vous pouvez afficher les images en plein écran :

* [**Calques d&#x27;image**](image-layers.md) - En savoir plus sur la visualisation multibande
* [**Bac à sable Index/LUT**](index-lut-sandbox.md) - Appliquer des indices personnalisés et un mappage des couleurs
* [**Formules d&#x27;indices multispectraux**](../project-settings/multispectral-index-formulas.md) - Comprendre les indices disponibles

Pour le flux de traitement, voir :

* [**Traitement des images (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Guide complet de traitement
