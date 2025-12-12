# Index/LUT Sandbox

L'Index/LUT Sandbox est un espace de travail interactif au sein de la visionneuse d'images Chloros qui vous permet d'expérimenter les calculs d'index multispectraux et les visualisations de couleurs en temps réel. Cet outil puissant vous permet de tester différents indices, d'affiner les plages de valeurs et de créer des visualisations prêtes à être publiées sans avoir à retraiter l'ensemble des données.

## Qu'est-ce que l'Index/LUT Sandbox ?

### Objectif

L'Environnement de travail fournit :

* **Calcul de l'indice en temps réel** - Application instantanée de n'importe quel indice de végétation
* **Ajustement interactif des LUT** - Affinez les gradients et les gammes de couleurs
* **Optimisation du flux de travail** - Déterminez les meilleurs paramètres avant le traitement par lots

### Traitement en bac à sable ou en projet

**Bac à sable Index/LUT (interactif):**

* Une seule image à la fois
* Retour d'information instantané
* Expérimental et itératif
* Pas de modifications permanentes des fichiers
* Parfait pour explorer et tester

**Traitement des projets (par lots):**

* Ensemble de données complet en une seule fois
* Paramètres préconfigurés
* Fichiers de sortie permanents
* Très coûteux en temps
* Meilleur lorsque les paramètres sont finalisés

{% hint style="success" %}
**Best Workflow**: Use the Sandbox to experiment and find optimal index and LUT settings, then apply those settings during Project Processing for your entire dataset.
{% endhint %}

***

## Travailler avec l'Index/LUT Sandbox

### Comprendre les indices précalculés

Dans Chloros, les indices peuvent être appliqués pendant le traitement du projet. Pour déterminer les paramètres d'index et de LUT que vous souhaitez appliquer aux exportations, il est plus facile d'utiliser le bac à sable de la visionneuse d'images.

Le bac à sable vous permet de :

* **appliquer de nouveaux index et gradients de couleur (LUTs)** pour visualiser les données
* **d'ajuster les paramètres de visualisation de manière interactive
* **visualiser les images d'index déjà calculées
* **d'inspecter les valeurs des pixels à tous les niveaux de zoom

### Ouvrir le bac à sable

L'accès à l'Index/LUT Sandbox se fait à partir de l'onglet de la barre latérale **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Cliquez sur une image dans la grille d'images du navigateur de fichiers, elle s'ouvre dans l'onglet **Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Cliquez sur **l'onglet Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pour ouvrir la barre latérale gauche si elle n'est pas déjà ouverte

### Sélection d'une image à laquelle appliquer un index/une table de conversion

Pour travailler avec un index dans le bac à sable de la visionneuse d'images <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Ouvrez une image** à partir de la grille d'images principale en cliquant dessus
2. L'onglet **Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> s'ouvre alors
3. Cliquez sur la liste déroulante **Calque** (en haut à droite de la visionneuse)
4. Sélectionnez le calque dans la liste déroulante :
   * RAW (Réflectance)

### Application d'un index à une image

Une fois que l'image est en plein écran et que la barre latérale de l'onglet **Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> est ouverte :

1. Cochez la case Index en haut de la barre latérale
2. Choisissez le filtre de votre appareil photo dans le menu déroulant de gauche
3. Choisissez la formule d'index souhaitée dans le menu déroulant de droite
4. Faites glisser les cercles de couleur du canal du filtre vers les emplacements de la formule d'indexation ci-dessous
5. Une fois que la formule est valide, l'image se met à jour et affiche les valeurs d'index
6. Déplacez le curseur de votre souris pour voir les valeurs à l'emplacement du curseur
7. Zoomez pour voir les pixels individuels et les valeurs qui leur sont associées

Chaque indice a une plage de valeurs et une signification spécifiques :

#### NDVI Exemple

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Pour une documentation complète sur les formules d'index, voir [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

***

## Travailler avec les LUTs (Look-Up Tables)

### Qu'est-ce qu'une LUT ?

Un **Look-Up Table (LUT)** associe des valeurs d'index numériques à des couleurs à des fins de visualisation :

* **Entrée** : Valeur du pixel d'index (par exemple, NDVI 0.65)
* **Sortie** : RGB couleur (par exemple, vert vif)
* **Objectif** : Faciliter la visualisation et l'interprétation des motifs

**Échelle de gris contre LUT de couleur:**

* Niveaux de gris : Scientifique et neutre, montre les données brutes
* LUT de couleur : Intuitive et percutante, elle met en évidence les modèles et les différences

{% hint style="success" %}
**Visualization Power**: Applying a color LUT to a grayscale index image makes it dramatically easier to identify patterns, anomalies, and areas of interest at a glance.
{% endhint %}

### Application d'une LUT à une image d'index

Une fois que vous avez une image d'index montrant

1. Cliquez sur le bouton <img src="../.gitbook/assets/image.png" alt="" data-size="line"> "+Add LUT"
2. Sélectionnez le dégradé de couleurs
3. Ajustez les points de terminaison min/max de l'écrêtage
4. Ajuster le mode d'écrêtage
5. Cochez la case Index dans la barre latérale de l'onglet **Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pour appliquer la LUT

### Choix d'un dégradé de couleurs

**Sélectionner un dégradé:**

1. Dans le panneau LUT, localisez la **barre de dégradé de couleurs**
2. Passez votre souris dessus pour afficher les préréglages de dégradé disponibles
3. Sélectionnez le dégradé souhaité
4. L'image **se met à jour immédiatement** avec les nouvelles couleurs lorsque la case Index est cochée

{% hint style="success" %}
**Best Practice**: For vegetation indices like NDVI, the Red-Yellow-Green gradient is most intuitive because it aligns with natural color associations (green=healthy, yellow=moderate, red=stressed).
{% endhint %}

### Ajustement des classes de couleurs

La commande **Classes** détermine le nombre de pas de couleur distincts apparaissant dans votre dégradé :

**Options de nombre de classes:**

* **2 à 5 classes** : Catégories très larges, zones distinctes
* **6-10 classes** : Équilibré, bon pour la classification
* **11-20 classes** : Gradients lisses, apparence continue
* **20+ classes** : Quasi-continu, douceur maximale

**Comment ajuster:**

1. Dans le panneau LUT, localisez les **carrés de l'échantillon de couleur sous la barre de gradient**
2. Ajustez le nombre de classes en les ajoutant à l'aide du bouton +
3. Supprimez le nombre de classes en double-cliquant sur un échantillon de couleur
4. Le dégradé se met à jour **en temps réel** sur l'image

**Effet sur la visualisation:**

* **Moins de classes** (3-5) : Crée des zones distinctes, simplifie la classification, permet de distinguer plus facilement les catégories
* **Classes moyennes** (6-10) : Approche équilibrée, bonne pour la plupart des applications
* **Plus de classes** (15-20) : Transitions douces, variations détaillées, aspect photographique

**Quand utiliser:**

* **Peu de classes (3-5)** : Diapositives de présentation, cartes de classification, rapports simples
* **Classes moyennes (6-10)** : Analyse générale, détails équilibrés, rapports standard
* **Nombreuses classes (15-20)** : Analyse scientifique, inspection détaillée, résultats de qualité professionnelle

### Réglage fin des fourchettes de valeurs

Les **règles de valeurs** déterminent quelles valeurs d'index correspondent à quelles couleurs dans votre dégradé :

**Commandes de plage dans le panneau LUT:**

* **Valeur minimale** : Limite inférieure de l'échelle de couleurs
* **Valeur maximale** : Limite supérieure de l'échelle des couleurs
* **Valeurs intermédiaires** : Réparties automatiquement entre min et max (sur la base du nombre de classes)

#### Ajustement des valeurs min/max

**Pour ajuster les plages de valeurs:**

1. Dans le panneau LUT, localisez les champs de saisie **Min Value** et **Max Value**
2. Cliquez sur le champ **Min Value** (valeur minimale)
3. Saisissez la valeur minimale souhaitée (par exemple, `0.2`)
4. Appuyez sur **Entrée** ou cliquez en dehors du champ
5. Répétez l'opération pour le champ **Valeur maximale** (par exemple, `0.9`)
6. La visualisation **se met à jour immédiatement**

{% hint style="info" %}
**Auto-Scaling**: When you first apply a LUT, Chloros automatically sets the min/max to the actual data range in the image. You can then narrow this range to focus on specific value ranges of interest.
{% endhint %}

**Exemple NDVI d'ajustements de gamme:**

* **Toute la gamme** : `-1.0` à `1.0` (afficher toutes les valeurs possibles)
* **Focalisation sur la végétation** : `0.2` à `0.9` (exclure le sol nu et l'eau)
* **Végétation saine uniquement** : `0.5` à `0.9` (ne met en évidence que les plantes vigoureuses)
* **Détection de stress** : `0.2` à `0.5` (souligne les zones à problèmes)
* **Plage personnalisée** : Ajustez en fonction des valeurs de pixels observées

**Pourquoi ajuster les plages ?

* **Augmenter le contraste** dans votre zone d'intérêt
* **Exclure les valeurs non pertinentes** (par exemple, les masses d'eau, les sols nus)
* **Standardiser la visualisation** sur plusieurs images ou dates
* **Accentuer les différences subtiles** à l'intérieur d'une plage de valeurs étroite

### Ecrêtage des valeurs hors plage

Lorsque les valeurs des pixels se situent en dehors de la plage min/max définie, vous pouvez contrôler la façon dont elles sont affichées à l'aide des **modes d'écrêtage**.

#### **Options de mode d'écrêtage disponibles:**

#### 1. Minimum et maximum

* Pixels **en dessous du minimum** → affichage en utilisant la **première couleur** du dégradé (par exemple, rouge)
* Pixels **au-dessus du maximum** → affichage en utilisant la **dernière couleur** du dégradé (par exemple, le vert)
* **Cas d'utilisation** : Mettre l'accent sur les extrêmes, montrer toute la gamme des données avec des couleurs saturées aux limites
* **Exemple** : NDVI les valeurs inférieures à 0,2 apparaissent toutes en rouge, les valeurs supérieures à 0,9 apparaissent toutes en vert

#### 2. Arrière-plan transparent

* Les pixels **hors de la plage** deviennent **complètement transparents**
* Seuls les pixels **à l'intérieur de la plage** affichent un dégradé de couleurs
* **Cas d'utilisation** : Superposition de SIG, isolation de plages de valeurs spécifiques, mise en évidence de zones d'intérêt uniquement
* **Exemple** : Afficher seulement NDVI 0.4-0.7 en couleur, tout le reste étant transparent

{% hint style="warning" %}
**Transparency Limitation**: Transparent pixels will appear as the background color in the viewer. When exported during processing, transparency is preserved in PNG format but not in JPG.
{% endhint %}

#### 3. Fond de l'index

* Les pixels **hors de la plage** s'affichent en **échelle de gris** (montrant les valeurs brutes de l'index)
* Les pixels **à l'intérieur de la plage** affichent un **gradient de couleur**
* **Cas d'utilisation** : Mise en évidence subtile, maintien du contexte tout en soulignant les zones d'intérêt
* **Exemple** : Mise en évidence par la couleur de la végétation stressée (NDVI 0.3-0.5) tout en montrant les zones saines en gris

#### 4. Arrière-plan original

* Les pixels **hors de la plage** affichent l'**image multispectrale originale**
* Les pixels **à l'intérieur de la plage** affichent **le gradient de couleur**
* **Cas d'utilisation** : Le plus intuitif - combine le contexte naturel de l'image avec une superposition analytique des couleurs
* **Exemple** : Voir l'apparence réelle du champ/de la culture avec les zones de stress codées en couleur

### Choisir le bon mode d'écrêtage

| Mode d'écrêtage - Meilleur pour - Style de visualisation - Mode d'écrêtage - Meilleur pour - Style de visualisation - Mode de visualisation
| -------------------------- | ------------------------------------------ | ---------------------------- |
| Affichage complet des données, analyse scientifique | Tous les pixels sont colorés
| Les données sont affichées sur un fond transparent, ce qui permet d'isoler des plages spécifiques
| Les données sont affichées dans leur intégralité, dans le cadre d'une analyse scientifique, et tous les pixels sont colorés
| Couleur sur la plage, photo au-delà | **Arrière-plan original** | Rapports, présentations, analyse intuitive | Couleur sur la plage, photo au-delà |

### Création de couleurs LUT personnalisées

Pour un contrôle total de votre visualisation, vous pouvez créer des **gradients de couleur personnalisés** en modifiant les arrêts de couleur individuels.

**Pour créer un dégradé personnalisé:**

1. Dans le panneau LUT, localisez la **barre de prévisualisation des dégradés**
2. Recherchez les **carrés d'échantillons de couleur** sous le dégradé
3. **Cliquez sur un arrêt de couleur** pour le sélectionner
4. Un **choix de couleur** s'ouvre
5. Choisissez une nouvelle couleur en utilisant :
   * **la roue des couleurs** : Sélection visuelle des couleurs
   * **RGB/HSV sliders**: Contrôle précis de la couleur
   * **Saisie du code hexadécimal** : Spécification exacte de la couleur (par exemple, `#FF0000` pour le rouge)
6. Cliquez sur le sélecteur de couleurs **pour appliquer la nouvelle couleur**
7. Le dégradé **s'actualise immédiatement** sur l'image

**Ajouter ou supprimer des arrêts de couleur:**

* **Ajouter un arrêt** : Cliquez sur l'icône + pour ajouter un nouvel échantillon à la fin
* **Supprimer un arrêt** : Double-cliquez sur le carré de couleur pour supprimer l'échantillon

**Stratégies de personnalisation:**

* **Inverser le dégradé** : Inverser l'ordre des couleurs pour inverser la signification (par exemple, vert=bas, rouge=haut)
* **Couleurs de la marque** : Correspondre à la palette de couleurs de votre organisation pour les rapports
* **Adapté aux daltoniens** : Utilisez des combinaisons orange-bleu ou violet-jaune
* **Optimisation de l'impression** : Optimisation de l'impression** : choisissez des couleurs qui fonctionnent à la fois en couleur et en niveaux de gris
* **Multi-seuils** : Utilisez des couleurs distinctes à des seuils de valeur spécifiques pour la classification

{% hint style="info" %}
**Saving Custom Gradients**: Custom gradients can be saved and reused. Click the save icon in the LUT panel to preserve your custom color schemes for future use.
{% endhint %}

***

## Flux de travail interactif

### Mises à jour en temps réel

Tous les ajustements de LUT dans le bac à sable mettent à jour l'image **instantanément et interactivement** :

* **Changer de calque** → L'image change immédiatement
* **Sélection d'un dégradé** → Les couleurs sont mises à jour instantanément
* **Ajuster la plage de valeurs** → le contraste change en temps réel
* **Changer de classe** → Lissage du dégradé mis à jour immédiatement
* **Modifier l'écrêtage** → L'affichage de l'arrière-plan change instantanément
* **Modifier les couleurs** → Le dégradé personnalisé s'applique immédiatement

**Aucun bouton "Appliquer" n'est nécessaire** - tous les changements sont en direct et interactifs !

{% hint style="success" %}
**Live Feedback**: The instant visual feedback allows you to rapidly experiment with different settings until you find the optimal visualization for your analysis needs.
{% endhint %}

### Flux de travail de l'affinage itératif

**Flux de travail typique pour l'optimisation de la LUT:**

1. **Sélection de la couche d'index** (par exemple, RAW (Reflectance))
2. **Appliquer l'index** - Choisir le filtre de la caméra et la formule d'index, faire glisser les cercles colorés à l'endroit approprié dans la formule d'index
3. **Appliquer le gradient LUT** - Commencez par le préréglage Red-Jaune-Green
4. **Inspecter les valeurs des pixels** - Déplacer le curseur, noter les plages de valeurs
5. **Ajuster min/max** - Réduire pour se concentrer sur la végétation (par exemple, 0,2 à 0,9)
6. **Choisir l'écrêtage** - Essayer "Arrière-plan original" pour le contexte
7. **Affiner les couleurs** - Personnaliser le dégradé si nécessaire pour une mise en valeur spécifique
8. **Finaliser les paramètres** - Documenter les paramètres et les copier dans les paramètres du projet pour le traitement de l'exportation

### Inspection de la valeur des pixels

Il est essentiel de comprendre les valeurs réelles des pixels pour définir des plages de LUT efficaces :

**Comment inspecter les valeurs:**

1. Les valeurs des pixels s'affichent lorsque les cases Index, Index et LUT de l'image sont cochées**.
2. **Déplacez votre curseur** sur différentes zones de l'image
3. **Observez les valeurs en pixels** affichées dans la légende lorsque vous survolez l'image
4. Zoomer pour voir les pixels individuels mis en évidence par une valeur flottante
5. **Prenez note** des fourchettes de valeurs pour les différentes caractéristiques :
   * **Végétation saine** : par exemple, NDVI 0,55-0,85
   * **Végétation stressée** : par exemple, NDVI 0,30-0,50
   * **Sol nu** : par exemple, NDVI 0,05-0,25
   * **Eau** (si présente) : par exemple, NDVI -0,05 à 0,10

**Utilisation des valeurs de pixels pour définir les plages de LUT:**

Après avoir inspecté les valeurs des pixels, réglez les valeurs min/max de votre LUT en conséquence :

**Exemple de scénario:**

* **Observation** : Valeurs du sol = 0,05-0,25, stressées = 0,25-0,50, saines = 0,50-0,85
* **Objectif** : Visualiser uniquement la santé des plantes (exclure le sol)
* **Paramètres LUT** : Min = `0.25`, Max = `0.85`
* **Clipping** : "Original Background" pour voir le sol dans sa couleur naturelle
* **Résultat** : Le gradient de couleur ne s'applique qu'à la végétation, le sol apparaît comme l'image originale

{% hint style="info" %}
**Dynamic Range**: Different crops, seasons, and growth stages will have different value ranges. Always inspect pixel values in your specific dataset before setting LUT ranges.
{% endhint %}

***

## Indices personnalisés (Chloros+)

### Création de formules d'indices personnalisés

{% hint style="info" %}
**Where to Create**: Custom indices can be configured in **Project Settings** before processing, as well as in the Image Viewer sandbox sidebar.
{% endhint %}

**Pour créer un index personnalisé:**

1. **Ouvrez les paramètres du projet** (avant le traitement) ou la barre latérale de la visionneuse d'images
2. Naviguez jusqu'à la liste déroulante **Formule d'index**
3. Recherchez l'option **"Personnalisée "** (vous devez être connecté avec une licence Chloros+)
4. **Définissez votre formule** en utilisant des variables de bande :
   * Nom des bandes : `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Opérateurs : `+`, `-`, `*`, `/`, `^` (exposant)
   * Fonctions : `sqrt()`, `abs()`, etc
   * Parenthèses : `()` pour l'ordre des opérations
5. **Nommez votre index** (par exemple, "MyIndex" ou "CustomNDVI")
6. **Sauvegarder la configuration

**Exemples de formules personnalisées:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Formula Validation**: Ensure your formula uses bands available in your camera. For example, RedEdge is only available on cameras with a RedEdge filter.
{% endhint %}

***

## Prochaines étapes

Maintenant que vous avez compris ce qu'est l'Index/LUT Sandbox :

* **Appliquer au traitement** : Utilisez les paramètres découverts dans [Project Settings](../project-settings/project-settings.md)
* **Traitement par lots** : Appliquer les indices optimisés à des ensembles de données complets
* **Pour en savoir plus Lire [Formules de l'indice multispectral](../project-settings/multispectral-index-formulas.md)

Documentation connexe :

* [**Calques d'image**](image-layers.md) - Gestion et visualisation des calques
* [**Ouverture d'une image en plein écran**](opening-an-image-full-screen.md) - Principes de base de la visionneuse d'images
* [**Traitement des images (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Flux de travail complet pour le traitement des images
