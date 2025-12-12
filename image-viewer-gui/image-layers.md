# Couches d&#x27;image

Le menu déroulant Couches d&#x27;image dans la visionneuse d&#x27;images Chloros vous permet de passer rapidement d&#x27;une version à l&#x27;autre d&#x27;une même image, depuis les captures originales jusqu&#x27;aux sorties de réflectance traitées et aux images d&#x27;indice calculées.

## Que sont les couches d&#x27;image ?

Dans Chloros, les **couches** font référence aux différentes sorties d&#x27;image disponibles pour une seule image source. Lorsque vous traitez des images, Chloros crée plusieurs versions :

* **Images originales** (fichiers JPG et RAW provenant de votre appareil photo)
* Résultats **calibrés en termes de réflectance** (si le calibrage de la réflectance a été activé)
* **Images cibles** (si l&#x27;image contient des cibles de calibrage)
* **Images d&#x27;index** (NDVI, NDRE, GNDVI, etc. si des indices ont été configurés)

Le **menu déroulant Sélecteur de couche** en haut à droite de la visionneuse d&#x27;images vous permet de passer instantanément d&#x27;une version à l&#x27;autre sans quitter la visionneuse.

***

## Types de couches disponibles

### JPG

* L&#x27;image JPG originale de votre appareil photo
* Toujours disponible pour toutes les images
* Non traitée, telle qu&#x27;elle a été capturée par l&#x27;appareil photo
* Chargement et affichage les plus rapides

**Quand l&#x27;afficher :**

* Aperçu rapide de la capture originale
* Vérification de la composition et du cadrage de l&#x27;image
* Vérification de la qualité de la capture avant le traitement

### RAW (original)

* Les données RAW originales du capteur de votre appareil photo
* Débayérisées sans post-traitement appliqué
* Profondeur de bits supérieure à celle du JPG (données du capteur généralement de 12 ou 14 bits)

**Quand l&#x27;afficher :**

* Inspection de la qualité des données originales du capteur
* Vérification des problèmes ou des artefacts du capteur
* Comparaison des résultats avant/après traitement

### RAW (cible)

* N&#x27;apparaît que pour les images identifiées comme contenant des cibles d&#x27;étalonnage
* Affiche l&#x27;image RAW originale avec la cible détectée
* Utilisé pour vérifier que la détection de la cible a réussi

**Quand afficher :**

* Confirmation que les cibles d&#x27;étalonnage ont été détectées correctement
* Vérification de la qualité de l&#x27;image cible
* Dépannage des problèmes d&#x27;étalonnage

{% hint style=&quot;info&quot; %}
**Couche cible** : cette couche n&#x27;apparaît dans le menu déroulant que pour les images contenant des cibles d&#x27;étalonnage. Les images capturées normalement ne disposent pas de cette option.
{% endhint %}

### RAW (réflectance)

* Image de sortie à réflectance étalonnée
* Vignette corrigée (si activée lors du traitement)
* Réflectance calibrée à l&#x27;aide des données cibles (si activée)
* Multibande TIFF avec tous les canaux de la caméra
* Les valeurs de pixels représentent le pourcentage de réflectance (lorsque le mode pourcentage est utilisé)
* Prêt à être manipulé avec le [Sandbox Index/LUT](index-lut-sandbox.md)

**Quand afficher :**

* Inspection des résultats calibrés
* Vérification de la qualité du calibrage
* Vérification de la précision scientifique des valeurs de pixels
* Comparaison avec l&#x27;original pour voir les effets du calibrage

{% hint style=&quot;success&quot; %}
**Recommandé** : utilisez le calque RAW (réflectance) lorsque vous vérifiez les valeurs de pixels pour des mesures et analyses scientifiques.
{% endhint %}

### RAW (NDVI Index)... et similaire

* Image d&#x27;indice de végétation calculée (NDVI dans cet exemple)
* Le nom de l&#x27;indice change en fonction de l&#x27;indice configuré pendant le traitement
* Exemples : RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index), etc.
* Image monochrome à bande unique montrant les résultats du calcul de l&#x27;indice
* Un calque apparaît pour chaque indice configuré dans les paramètres du projet

**Noms d&#x27;index possibles :**

* RAW (indice NDVI)
* RAW (indice NDRE)
* RAW (indice GNDVI)
* RAW (indice OSAVI)
* RAW (indice EVI)
* RAW (indice SAVI)
* Et bien d&#x27;autres encore... (voir [Formules d&#x27;indices multispectraux](../project-settings/multispectral-index-formulas.md))

**Quand afficher :**

* Examen des résultats du calcul de l&#x27;indice
* Vérification des plages de valeurs de l&#x27;indice
* Identification des zones d&#x27;intérêt
* Vérification des images de l&#x27;indice avant leur utilisation dans un SIG ou une analyse

***

## Utilisation du sélecteur de couches

### Ouverture du menu déroulant

1. Ouvrez une image en mode plein écran (cliquez sur n&#x27;importe quelle vignette dans la visionneuse d&#x27;images).
2. Localisez le **menu déroulant des couches** dans le coin supérieur droit de la visionneuse.
3. Le menu déroulant affiche le calque actuellement sélectionné (par exemple, « JPG »).
4. Cliquez sur le menu déroulant pour voir tous les calques disponibles.

### Changement de calque

1. Cliquez sur le menu déroulant des calques pour ouvrir la liste.
2. Tous les calques disponibles pour l&#x27;image actuelle s&#x27;affichent.
3. Cliquez sur le nom d&#x27;un calque pour passer à cette version.
4. L&#x27;image est immédiatement mise à jour pour afficher le calque sélectionné.

**Changement rapide :**

* Le menu déroulant mémorise votre dernière sélection.
* Lorsque vous passez à l&#x27;image suivante, Chloros tente d&#x27;afficher le même type de calque.
* Si ce calque n&#x27;existe pas sur l&#x27;image suivante, le format JPG est utilisé par défaut.

### Disponibilité des calques

Tous les calques ne sont pas disponibles pour toutes les images :

**Toujours disponibles :**

* ✅ JPG (chaque image dispose d&#x27;un aperçu JPG)

**Disponible sous certaines conditions :**

* ⚠️ RAW (original) - Uniquement si l&#x27;image a été capturée en mode RAW ou RAW+JPG
* ⚠️ RAW (cible) - Uniquement si l&#x27;image contient des cibles d&#x27;étalonnage détectées
* ⚠️ RAW (réflectance) - Uniquement après traitement avec l&#x27;étalonnage de réflectance activé
* ⚠️ RAW ([Index] Index) - Uniquement après traitement avec les indices configurés

***

## Persistance des calques

### Navigation entre les images

Lorsque vous naviguez vers une autre image (à l&#x27;aide des touches fléchées ou en cliquant sur les vignettes) :

**La préférence de calque est conservée :**

* Si vous affichez « RAW (réflectance) », l&#x27;image suivante affiche « RAW (réflectance) » (si disponible)
* Si vous affichez « RAW (NDVI Index) », l&#x27;image suivante affiche « RAW (NDVI Index) » (si disponible)
* Si le même calque n&#x27;existe pas, le format JPG est utilisé par défaut.

**Exemple de flux de travail :**

1. Ouvrez l&#x27;image 1, passez à RAW (NDVI Index).
2. Appuyez sur → pour afficher l&#x27;image 2.
3. L&#x27;image 2 affiche automatiquement le calque RAW (NDVI Index).
4. Continuez à naviguer : toutes les images affichent le calque NDVI.
5. Très efficace pour examiner les résultats d&#x27;indexation sur plusieurs images.

***

## Flux de travail courants

### Flux de travail 1 : comparaison avant/après

**Objectif** : comparer l&#x27;image originale et l&#x27;image calibrée.

1. Ouvrez l&#x27;image traitée dans Image Viewer.
2. Sélectionnez **RAW (Original)** dans le menu déroulant.
3. Notez le vignettage et les valeurs non calibrées.
4. Passez à **RAW (Réflectance)** dans le menu déroulant.
5. Comparez : le vignettage a été supprimé et les valeurs ont été calibrées.

### Flux de travail 2 : examen de l&#x27;index

**Objectif** : examiner rapidement les résultats NDVI dans l&#x27;ensemble de données.

1. Ouvrez la première image traitée.
2. Sélectionnez **RAW (NDVI Index)** dans le menu déroulant.
3. Utilisez la touche fléchée → pour passer à l&#x27;image suivante.
4. Le calque NDVI persiste automatiquement.
5. Continuez à parcourir toutes les images en vérifiant les motifs NDVI.
6. Passez à **RAW (NDRE Index)** pour comparer.

### Workflow 3 : Vérification des cibles

**Objectif** : Vérifier que toutes les images cibles ont été détectées correctement.

1. Accédez à une image cible.
2. Sélectionnez **RAW (Target)** dans le menu déroulant.
3. Vérifiez que les cibles d&#x27;étalonnage sont clairement visibles et détectées.
4. Accédez à l&#x27;image cible suivante.
5. Répétez la vérification pour toutes les cibles.

### Workflow 4 : Inspection de la valeur des pixels

**Objectif** : Vérifier les valeurs de réflectance pour garantir leur exactitude scientifique.

1. Ouvrez l&#x27;image traitée.
2. Sélectionnez le calque **RAW (Réflectance)**.
3. Activez le mode **Pourcentage de pixels** (bouton dans la barre d&#x27;outils en haut à droite).
4. Déplacez le curseur sur les zones de végétation.
5. Vérifiez que les valeurs des pixels se situent dans les plages attendues (30 à 70 % pour NIR, 5 à 15 % pour Red).
6. Vérifiez que les zones de sol et d&#x27;eau présentent des valeurs appropriées.

***

## Comprendre les valeurs des pixels par couche

Les différentes couches affichent différentes plages de valeurs de pixels :

### Couche JPG

* **Plage** : 0-255 (8 bits)
* **Signification** : valeurs d&#x27;affichage, corrigées en gamma
* **Utilisation** : inspection visuelle uniquement, ne convient pas aux mesures scientifiques

### RAW (original)

* **Plage** : 0-65535 (16 bits)
* **Signification** : nombres numériques bruts du capteur
* **Utilisation** : vérification des performances du capteur, non calibré

### RAW (réflectance)

* **Plage** : 0-65 535 (16 bits TIFF) ou 0,0-1,0 (32 bits pourcentage)
* **Signification** : pourcentage de réflectance calibré
* **Utilisation** : mesures et analyses scientifiques

**Pour 16 bits TIFF :** divisez par 65 535 pour obtenir le pourcentage de réflectance **Pour 32 bits pourcentage :** les valeurs représentent directement le pourcentage (0,5 = 50 % de réflectance)

### RAW (images d&#x27;indice)

* **Plage** : varie selon l&#x27;indice (généralement de -1,0 à +1,0 pour les indices normalisés)
* **Signification** : résultat du calcul de l&#x27;indice
* **Exemples** :
  * NDVI : -1 à +1 (végétation généralement de 0,4 à 0,9)
  * NDRE : -1 à +1 (détection du stress)
  * EVI : 0 à 1 (végétation améliorée)

***

## Conseils et bonnes pratiques

### Changement efficace de couche

* **Raccourcis clavier** : bien qu&#x27;il n&#x27;existe pas de raccourci clavier pour les couches, les flèches de navigation (←/→) fonctionnent sur toutes les couches
* **Workflows cohérents** : sélectionnez une couche (par exemple, NDVI) et examinez l&#x27;ensemble des données avant de passer à une autre
* **Comparaisons rapides** : basculez entre Original et Réflectance pour vérifier la qualité du traitement

### Considérations relatives aux performances

* **Le format JPG est le plus rapide à charger** : utilisez-le pour naviguer rapidement parmi de nombreuses images.
* **Les couches RAW sont plus lentes à charger** : résolution et profondeur de bits plus élevées.
* **Couches d&#x27;index** : vitesse similaire à celle des couches de réflectance.
* **Le premier chargement est le plus lent** : les affichages suivants de la même couche sont mis en cache et plus rapides.

### Vérification de la qualité

* **Vérifiez toujours le format RAW (original)** : Vérifiez la qualité des données sources avant de vous fier aux résultats traités.
* **Comparez les couches** : utilisez le changement de couche pour vérifier que le traitement a fonctionné correctement.
* **Vérifiez les plages d&#x27;index** : utilisez le mode Pourcentage de pixels avec les couches d&#x27;index pour vérifier que les valeurs sont raisonnables.

***

## Dépannage

### Couche indisponible

**Problème** : la couche attendue n&#x27;apparaît pas dans le menu déroulant.

**Causes possibles :**

* L&#x27;image n&#x27;a pas été traitée (seuls les formats JPG et RAW (original) sont disponibles).
* L&#x27;étalonnage de la réflectance a été désactivé pendant le traitement.
* L&#x27;index spécifique n&#x27;a pas été configuré dans les paramètres du projet.
* L&#x27;image est une image cible uniquement (aucun index n&#x27;a été généré pour les cibles).

**Solutions :**

1. Vérifiez que l&#x27;image a été traitée (vérifiez le dossier de sortie pour les fichiers traités).
2. Vérifiez les paramètres du projet pour confirmer que les index ont été configurés.
3. Retraitez l&#x27;image en activant les indices souhaités.

### Couche incorrecte affichée

**Problème** : l&#x27;image s&#x27;ouvre dans une couche inattendue.

**Cause** : les préférences de couche de l&#x27;image précédente ont été conservées, mais cette couche n&#x27;existe pas dans l&#x27;image actuelle.

**Solution** : Chloros revient automatiquement au format JPG lorsque la couche préférée n&#x27;est pas disponible. Il s&#x27;agit d&#x27;un comportement normal.

### Impossible de voir les cibles d&#x27;étalonnage

**Problème** : le calque RAW (cible) n&#x27;affiche pas la détection des cibles.

**Causes possibles :**

* Les cibles n&#x27;ont pas été détectées pendant le traitement.
* L&#x27;image ne contient pas réellement de cibles.
* Les paramètres de détection des cibles sont trop stricts.

**Solutions :**

1. Vérifiez le journal de débogage pour voir s&#x27;il contient des messages « Cible trouvée ».
2. Vérifiez que l&#x27;image contient réellement des cibles d&#x27;étalonnage visibles.
3. Ajustez les paramètres de détection des cibles dans les paramètres du projet.
4. Consultez [Choix des images cibles](../processing-images-gui/choosing-target-images.md).

***

## Fonctionnalités associées

### Outils de la visionneuse d&#x27;images

Lorsque vous affichez un calque, vous pouvez utiliser :

* **Commandes de zoom** : agrandissez pour inspecter les détails.
* **Panoramique** : Cliquez et faites glisser pour vous déplacer dans l&#x27;image agrandie
* **Inspection de la valeur des pixels** : affichez les valeurs à l&#x27;emplacement du curseur
* **Flèches de navigation** : passez d&#x27;une image à l&#x27;autre tout en conservant le calque
* **Mode pourcentage de pixels** : basculez entre l&#x27;affichage DN et l&#x27;affichage en pourcentage

Consultez [Ouverture d&#x27;une image en plein écran](opening-an-image-full-screen.md) pour obtenir la documentation complète sur la visionneuse d&#x27;images.

### Sandbox Index/LUT

Pour tester et visualiser les index de manière interactive :

* **Calcul d&#x27;index en temps réel** : testez différentes formules d&#x27;index.
* **Mappage des couleurs LUT** : appliquez des dégradés de couleurs aux indices en niveaux de gris.
* **Exporter les visualisations** : enregistrez les images d&#x27;index colorées.

Consultez [Sandbox Index/LUT](index-lut-sandbox.md) pour plus de détails.

***

## Étapes suivantes

Maintenant que vous comprenez les couches d&#x27;image :

* [**Ouverture d&#x27;une image en plein écran**](opening-an-image-full-screen.md) - Guide complet de l&#x27;Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualisation interactive des indices
* [**Formules d&#x27;index multispectral**](../project-settings/multispectral-index-formulas.md) - Référence des indices disponibles
* [**Fin du traitement**](../processing-images-gui/finishing-the-processing.md) - Comprendre les résultats traités
