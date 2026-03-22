# Calques d&#x27;image

Le menu déroulant « Calques d&#x27;image » de la visionneuse d&#x27;images Chloros vous permet de passer rapidement d&#x27;une version à l&#x27;autre d&#x27;une même image, depuis les captures originales jusqu&#x27;aux résultats de réflectance traités et aux images d&#x27;indice calculées.

## Que sont les calques d&#x27;image ?

Dans Chloros, les **calques** désignent les différents résultats d&#x27;image disponibles pour une même image source. Lorsque vous traitez des images, Chloros crée plusieurs versions :

* **Images originales** (fichiers JPG et RAW provenant de votre appareil photo)
* **Sorties avec réflectance calibrée** (si le calibrage de la réflectance a été activé)
* **Images cibles** (si l&#x27;image contient des cibles de calibrage)
* **Images d&#x27;index** (NDVI, NDRE, GNDVI, etc. si des indices ont été configurés)

Le **menu déroulant Sélecteur de calque** en haut à droite de la visionneuse d&#x27;images vous permet de basculer instantanément entre ces versions sans quitter la visionneuse.***

## Types de calques disponibles

### JPG

* L&#x27;image d&#x27;aperçu JPG originale provenant de votre appareil photo
* Toujours disponible pour toutes les images
* Non traitée, telle qu&#x27;elle a été capturée par l&#x27;appareil photo
* La plus rapide à charger et à afficher

**Quand l&#x27;afficher :**

* Aperçu rapide de la capture d&#x27;origine
* Vérification de la composition et du cadrage de l&#x27;image
* Vérification de la qualité de la capture avant traitement

### RAW (Original)

* Les données RAW d&#x27;origine du capteur de votre appareil photo
* Dénoyautées sans post-traitement appliqué
* Profondeur de bits supérieure à celle du JPG (généralement des données de capteur 12 bits ou 14 bits)

**Quand l&#x27;afficher :**

* Inspection de la qualité des données brutes du capteur
* Vérification des problèmes de capteur ou des artefacts
* Comparaison des résultats avant/après traitement

### RAW (Cible)

* N&#x27;apparaît que pour les images identifiées comme contenant des cibles d&#x27;étalonnage
* Affiche l&#x27;image RAW originale avec la cible détectée
* Utilisé pour vérifier que la détection de la cible a réussi

**Quand l&#x27;afficher :**

* Confirmation que les cibles d&#x27;étalonnage ont été détectées correctement
* Vérification de la qualité de l&#x27;image de la cible
* Dépannage des problèmes d&#x27;étalonnage

{% hint style="info" %}
**Couche de cible** : cette couche n&#x27;apparaît dans le menu déroulant que pour les images contenant des cibles d&#x27;étalonnage. Les images de capture standard ne disposeront pas de cette option.
{% endhint %}

### RAW (Réflectance)

* L&#x27;image de sortie de réflectance étalonnée
* Vignettage corrigé (si activé lors du traitement)
* Réflectance calibrée à l&#x27;aide des données des cibles (si activé)
* Multibande TIFF avec tous les canaux de la caméra
* Les valeurs des pixels représentent le pourcentage de réflectance (en mode pourcentage)
* Prêt à être manipulé avec le [Index/LUT Sandbox](index-lut-sandbox.md)

**Quand l&#x27;afficher :**

* Inspection des résultats calibrés
* Vérification de la qualité du calibrage
* Vérification de l&#x27;exactitude scientifique des valeurs de pixels
* Comparaison avec l&#x27;original pour observer les effets du calibrage

{% hint style="success" %}
**Recommandé** : utilisez le calque RAW (réflectance) lors de la vérification des valeurs de pixels pour les mesures et analyses scientifiques.
{% endhint %}

### RAW (NDVI Index)... et similaires

* Image de l&#x27;indice de végétation calculé (NDVI dans cet exemple)
* Le nom de l&#x27;indice change en fonction de l&#x27;indice configuré lors du traitement
* Exemples : RAW (Indice NDVI), RAW (Indice NDRE), RAW (Indice GNDVI), etc.
* Image en niveaux de gris à bande unique affichant les résultats du calcul de l&#x27;indice
* Un calque apparaît pour chaque indice configuré dans les paramètres du projet

**Noms d&#x27;index possibles :**

* RAW (Index NDVI)
* RAW (Index NDRE)
* RAW (Index GNDVI)
* RAW (Index OSAVI)
* RAW (Indice EVI)
* RAW (Indice SAVI)
* Et bien d&#x27;autres encore... (voir [Formules d&#x27;indices multispectraux](../project-settings/multispectral-index-formulas.md))

**Quand les consulter :**

* Examiner les résultats du calcul des indices
* Vérifier les plages de valeurs des indices
* Identifier les zones d&#x27;intérêt
* Vérifier les images d&#x27;indices avant de les utiliser dans un SIG ou pour une analyse

***

## Utilisation du sélecteur de couches

### Ouvrir le menu déroulant

1. Ouvrez une image en mode plein écran (cliquez sur n&#x27;importe quelle vignette dans la visionneuse d&#x27;images)
2. Repérez le **menu déroulant des couches** dans le coin supérieur droit de la visionneuse
3. Le menu déroulant affiche la couche actuellement sélectionnée (par exemple, « JPG »)
4. Cliquez sur le menu déroulant pour voir toutes les couches disponibles

### Changer de couche

1. Cliquez sur le menu déroulant des couches pour ouvrir la liste
2. Toutes les couches disponibles pour l&#x27;image actuelle s&#x27;affichent
3. Cliquez sur le nom d&#x27;une couche pour passer à cette version
4. L&#x27;image s&#x27;actualise immédiatement pour afficher la couche sélectionnée

**Changement rapide :**

* Le menu déroulant mémorise votre dernière sélection
* Lorsque vous passez à l&#x27;image suivante, Chloros tente d&#x27;afficher le même type de couche
* Si cette couche n&#x27;existe pas sur l&#x27;image suivante, le format JPG est utilisé par défaut

### Disponibilité des couches

Toutes les couches ne sont pas disponibles pour chaque image :

**Toujours disponibles :*** ✅ JPG (chaque image dispose d&#x27;un aperçu JPG)

**Disponibles sous certaines conditions :**

* ⚠️ RAW (Original) - Uniquement si l&#x27;image a été capturée en mode RAW ou RAW+JPG
* ⚠️ RAW (Cible) - Uniquement si l&#x27;image contient des cibles d&#x27;étalonnage détectées
* ⚠️ RAW (Réflectance) - Uniquement après traitement avec l&#x27;étalonnage de réflectance activé
* ⚠️ RAW (\[Index] Index) - Uniquement après traitement avec indices configurés

***

## Persistance des calques

### Navigation entre les images

Lorsque vous passez à une autre image (à l&#x27;aide des touches fléchées ou en cliquant sur les vignettes) :**La préférence de calque est conservée :**

* Si vous visualisez « RAW (Réflectance) », l&#x27;image suivante affiche « RAW (Réflectance) » (si disponible)
* Si vous visualisez « RAW (NDVI Index) », l&#x27;image suivante affiche « RAW (NDVI Index) » (si disponible)
* Si le même calque n&#x27;existe pas, le format par défaut est JPG

**Exemple de workflow :**

1. Ouvrez l&#x27;image 1, passez en mode RAW (NDVI Index)
2. Appuyez sur → pour afficher l&#x27;image 2
3. L&#x27;image 2 affiche automatiquement le calque RAW (NDVI Index)
4. Continuez à naviguer : toutes les images affichent le calque NDVI
5. Très efficace pour examiner les résultats de l&#x27;index sur de nombreuses images

***

## Workflows courants

### Workflow 1 : Comparaison avant/après

**Objectif** : Comparer l&#x27;image originale et l&#x27;image calibrée

1. Ouvrez l&#x27;image traitée dans la visionneuse d&#x27;images
2. Sélectionnez **RAW (Original)** dans le menu déroulant
3. Notez le vignettage et les valeurs non calibrées
4. Passez à **RAW (Réflectance)** dans le menu déroulant
5. Comparez : le vignettage est supprimé, les valeurs sont calibrées

### Workflow 2 : Examen de l&#x27;index

**Objectif** : Examiner rapidement les résultats NDVI sur l&#x27;ensemble des données

1. Ouvrez la première image traitée
2. Sélectionnez **RAW (NDVI Index)** dans le menu déroulant
3. Utilisez la touche fléchée → pour passer à l&#x27;image suivante
4. Le calque NDVI s&#x27;affiche automatiquement
5. Passez en revue toutes les images en vérifiant les motifs NDVI
6. Passez à **RAW (NDRE Index)** pour comparer

### Workflow 3 : Vérification des cibles

**Objectif** : Vérifier que toutes les images cibles ont été détectées correctement

1. Accédez à une image cible
2. Sélectionnez **RAW (Target)** dans le menu déroulant
3. Vérifiez que les cibles d&#x27;étalonnage sont clairement visibles et détectées
4. Accédez à l&#x27;image cible suivante
5. Répétez la vérification pour toutes les cibles

### Workflow 4 : Inspection des valeurs de pixels

**Objectif** : Vérifier l&#x27;exactitude scientifique des valeurs de réflectance

1. Ouvrez l&#x27;image traitée
2. Sélectionnez le calque **RAW (Réflectance)**

3. Activez le mode**Pourcentage de pixels** (bouton dans la barre d&#x27;outils en haut à droite)
4. Déplacez le curseur sur les zones de végétation
5. Vérifiez que les valeurs des pixels se situent dans les plages attendues (30-70 % pour NIR, 5-15 % pour Red)
6. Vérifiez que les zones de sol et d&#x27;eau présentent des valeurs appropriées

***

## Comprendre les valeurs des pixels par couche

Les différentes couches présentent des plages de valeurs de pixels différentes :

### Couche JPG

* **Plage** : 0-255 (8 bits)
* **Signification** : valeurs d&#x27;affichage, corrigées en gamma
* **Utilisation** : inspection visuelle uniquement, ne convient pas aux mesures scientifiques

### RAW (Original)

* **Plage** : 0-65535 (16 bits)
* **Signification** : Chiffres numériques bruts du capteur
* **Utilisation** : Vérification des performances du capteur, non calibré

### RAW (réflectance)

* **Plage** : 0-65 535 (16 bits TIFF) ou 0,0-1,0 (32 bits en pourcentage)
* **Signification** : Pourcentage de réflectance calibré
* **Utilisation** : Mesures et analyses scientifiques**Pour le format 16 bits TIFF :**Diviser par 65 535 pour obtenir le pourcentage de réflectance**Pour le format 32 bits (pourcentage) :** Les valeurs représentent directement le pourcentage (0,5 = 50 % de réflectance)

### BRUT (Images d&#x27;indice)

* **Plage** : Varie selon l&#x27;indice (généralement de -1,0 à +1,0 pour les indices normalisés)
* **Signification** : Résultat du calcul de l&#x27;indice
* **Exemples** :
  * NDVI : -1 à +1 (végétation généralement de 0,4 à 0,9)
  * NDRE : -1 à +1 (détection du stress)
  * EVI : 0 à 1 (végétation améliorée)

***

## Conseils et bonnes pratiques

### Changement efficace de calque

* **Raccourcis clavier** : bien qu&#x27;il n&#x27;y ait pas de raccourcis clavier pour les couches, les flèches de navigation (←/→) fonctionnent sur toutes les couches
* **Flux de travail cohérents** : choisissez une couche (par exemple, NDVI) et examinez l&#x27;ensemble du jeu de données avant de passer à une autre
* **Comparaisons rapides** : basculez entre « Original » et « Reflectance » pour vérifier la qualité du traitement

### Considérations relatives aux performances

* **Le format JPG se charge le plus rapidement** : à utiliser pour une navigation rapide parmi de nombreuses images
* **Les couches RAW se chargent plus lentement** : résolution et profondeur de bits plus élevées
* **Couches d&#x27;index** : vitesse similaire à celle des couches de réflectance
* **Le premier chargement est le plus lent** : les affichages suivants de la même couche sont mis en cache et donc plus rapides

### Vérification de la qualité

* **Vérifiez toujours la couche RAW (Original)** : vérifiez la qualité des données sources avant de vous fier aux résultats traités
* **Comparez les couches** : utilisez le changement de couche pour valider que le traitement a fonctionné correctement
* **Vérifiez les plages d&#x27;index** : utilisez le mode Pourcentage de pixels avec les couches d&#x27;index pour vérifier que les valeurs sont raisonnables***

## Dépannage

### Couche indisponible

**Problème** : la couche attendue n&#x27;apparaît pas dans le menu déroulant**Causes possibles :**

* L&#x27;image n&#x27;a pas été traitée (seuls les formats JPG et RAW (original) sont disponibles)
* L&#x27;étalonnage de la réflectance a été désactivé pendant le traitement
* L&#x27;indice spécifique n&#x27;a pas été configuré dans les paramètres du projet
* L&#x27;image est une image de cibles uniquement (aucun indice n&#x27;est généré pour les cibles)

**Solutions :**

1. Vérifiez que l&#x27;image a bien été traitée (vérifiez le dossier de sortie pour les fichiers traités)
2. Vérifiez les paramètres du projet pour confirmer que les indices ont été configurés
3. Relancez le traitement en activant les indices souhaités

### Mauvaise couche affichée

**Problème** : L&#x27;image s&#x27;ouvre dans une couche inattendue**Cause** : La préférence de couche de l&#x27;image précédente a été conservée, mais cette couche n&#x27;existe pas sur l&#x27;image actuelle**Solution** : Chloros revient automatiquement au format JPG lorsque la couche préférée n&#x27;est pas disponible - il s&#x27;agit d&#x27;un comportement normal

### Impossible de voir les cibles d&#x27;étalonnage

**Problème** : le calque RAW (Cible) n&#x27;affiche pas la détection des cibles**Causes possibles :**

* Les cibles n&#x27;ont pas été détectées pendant le traitement
* L&#x27;image ne contient pas réellement de cibles
* Les paramètres de détection des cibles sont trop stricts

**Solutions :**

1. Vérifiez le journal de débogage pour voir s&#x27;il contient des messages « Cible trouvée »
2. Vérifiez que l&#x27;image contient effectivement des cibles d&#x27;étalonnage visibles
3. Ajustez les paramètres de détection des cibles dans les paramètres du projet
4. Consultez [Choix des images cibles](../processing-images-gui/choosing-target-images.md)

***

## Fonctionnalités associées

### Outils de la visionneuse d&#x27;images

Lors de la visualisation d&#x27;un calque, vous pouvez utiliser :

* **Commandes de zoom** : agrandissez pour inspecter les détails
* **Panoramique** : Cliquez et faites glisser pour vous déplacer dans l&#x27;image agrandie
* **Inspection de la valeur des pixels** : affichez les valeurs à l&#x27;emplacement du curseur
* **Flèches de navigation** : passez d&#x27;une image à l&#x27;autre tout en conservant le calque
* **Mode pourcentage de pixels** : basculez entre l&#x27;affichage en DN et en pourcentage

Consultez [Ouverture d&#x27;une image en plein écran](opening-an-image-full-screen.md) pour obtenir la documentation complète sur la visionneuse d&#x27;images.

### Sandbox Index/LUT

Pour tester et visualiser les indices de manière interactive :

* **Calcul d&#x27;indice en temps réel** : Testez différentes formules d&#x27;indice
* **Mappage de couleurs LUT** : Appliquez des dégradés de couleurs aux indices en niveaux de gris
* **Exporter les visualisations** : Enregistrez les images d&#x27;indice colorées

Consultez [Sandbox Index/LUT](index-lut-sandbox.md) pour plus de détails.

***

## Étapes suivantes

Maintenant que vous comprenez les couches d&#x27;image :

* [**Ouverture d&#x27;une image en plein écran**](opening-an-image-full-screen.md) - Guide complet de la visionneuse d&#x27;images
* [**Bac à sable Index/LUT**](index-lut-sandbox.md) - Visualisation interactive des indices
* [**Formules d&#x27;indices multispectraux**](../project-settings/multispectral-index-formulas.md) - Référence des indices disponibles
* [**Finalisation du traitement**](../processing-images-gui/finishing-the-processing.md) - Comprendre les résultats du traitement
