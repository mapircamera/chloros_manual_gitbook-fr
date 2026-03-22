# Sandbox Index/LUT

Le Sandbox Index/LUT est un espace de travail interactif intégré à la visionneuse d&#x27;images Chloros qui vous permet d&#x27;expérimenter en temps réel le calcul d&#x27;indices multispectraux et la visualisation des couleurs. Cet outil puissant vous aide à tester différents indices, à affiner les plages de valeurs et à créer des visualisations prêtes à être publiées sans avoir à retraiter l&#x27;intégralité de votre ensemble de données.

## Qu&#x27;est-ce que le Sandbox Index/LUT ?

### Objectif

Le Sandbox offre :

* **Calcul d&#x27;indices en temps réel** - Appliquez instantanément n&#x27;importe quel indice de végétation
* **Réglage interactif des LUT** - Affinez les dégradés et les plages de couleurs
* **Optimisation du flux de travail** - Déterminez les meilleurs paramètres avant le traitement par lots

### Sandbox vs. Traitement de projet

**Sandbox Index/LUT (interactif) :**

* Une seule image à la fois
* Retour instantané
* Expérimental et itératif
* Aucune modification permanente des fichiers
* Parfait pour l&#x27;exploration et les tests

**Traitement de projet (par lots) :**

* Ensemble de données complet en une seule fois
* Paramètres préconfigurés
* Fichiers de sortie permanents
* Prise de temps
* Idéal lorsque les paramètres sont finalisés

{% hint style="success" %}
**Meilleur flux de travail** : utilisez le Sandbox pour expérimenter et trouver les paramètres d&#x27;indice et de LUT optimaux, puis appliquez ces paramètres lors du traitement de projet pour l&#x27;ensemble de vos données.
{% endhint %}

***

## Utilisation du bac à sable Index/LUT

### Comprendre les indices précalculés

Dans Chloros, les indices peuvent être appliqués pendant le traitement du projet. Pour déterminer les paramètres d&#x27;index et de LUT que vous souhaitez appliquer aux exportations, le plus simple est d&#x27;utiliser le bac à sable de la visionneuse d&#x27;images.

Le bac à sable vous permet de :

* **d&#x27;appliquer de nouveaux indices et dégradés de couleurs (LUT)** pour visualiser les données
* **d&#x27;ajuster les paramètres de visualisation** de manière interactive
* **de visualiser** les images d&#x27;indice déjà calculées
* **d&#x27;inspecter** les valeurs des pixels à tous les niveaux de zoom

### Ouverture de la zone de test

La zone de test Index/LUT est accessible dans l&#x27;onglet **Visionneuse d&#x27;images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Cliquez sur une image dans la grille d&#x27;images du navigateur de fichiers ; elle s&#x27;ouvre dans l&#x27;onglet **Visionneuse d&#x27;images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Cliquez sur l&#x27;onglet **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pour ouvrir la barre latérale contextuelle de gauche si elle n&#x27;est pas déjà ouverte

### Sélectionner une image à laquelle appliquer un index/une LUT

Pour travailler avec un index dans la <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Ouvrez une image** à partir de la grille d&#x27;images principale en cliquant dessus
2. L&#x27;onglet **Visionneuse d&#x27;images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> s&#x27;ouvre alors
3. Cliquez sur le **menu déroulant des calques** (en haut à droite de la visionneuse)
4. Sélectionnez le calque dans le menu déroulant :
   * RAW (Réflectance)

### Appliquer un indice à une image

Une fois que l&#x27;image est en plein écran et que la barre latérale de l&#x27;onglet **Visionneuse d&#x27;images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> est ouverte :

1. Cochez la case « Index » en haut de la barre latérale
2. Choisissez le filtre de votre caméra dans le menu déroulant de gauche
3. Choisissez la formule d&#x27;index souhaitée dans le menu déroulant de droite
4. Faites glisser les cercles de couleur des canaux du filtre vers les emplacements correspondants dans la formule d&#x27;index ci-dessous
5. Une fois la formule valide, l&#x27;image s&#x27;actualisera et affichera les valeurs d&#x27;index
6. Déplacez le curseur de votre souris pour voir les valeurs à l&#x27;emplacement du curseur
7. Zoomez pour voir les pixels individuels et leurs valeurs associées

Chaque indice a une plage de valeurs et une signification spécifiques :

#### Exemple NDVI

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

Pour une documentation complète sur les formules d&#x27;indice, consultez [Formules d&#x27;indice multispectrales](../project-settings/multispectral-index-formulas.md).

***

## Utilisation des LUT (tables de correspondance)

### Qu&#x27;est-ce qu&#x27;une LUT ?

Une **table de correspondance (LUT)** associe des valeurs d&#x27;indice numériques à des couleurs à des fins de visualisation :

* **Entrée** : valeur d&#x27;indice du pixel (par exemple, NDVI 0,65)
* **Sortie** : couleur (par exemple, vert vif)
* **Objectif** : rendre les motifs plus faciles à voir et à interpréter**LUT en niveaux de gris vs LUT couleur :**

* Niveaux de gris : scientifique et neutre, affiche les données brutes
* LUT couleur : intuitive et percutante, met en évidence les motifs et les différences

{% hint style="success" %}
**Puissance de visualisation** : l&#x27;application d&#x27;une table de conversion couleur à une image d&#x27;index en niveaux de gris facilite considérablement l&#x27;identification des motifs, des anomalies et des zones d&#x27;intérêt en un coup d&#x27;œil.
{% endhint %}

### Application d&#x27;une table de conversion à une image d&#x27;index

Une fois que vous disposez d&#x27;une image d&#x27;index affichant

1. Cliquez sur le <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> bouton « +Ajouter une LUT »
2. Sélectionnez le dégradé de couleurs
3. Réglez les points d&#x27;extrémité min/max de la coupure
4. Réglez le mode de coupure
5. Cochez la case Index dans la barre latérale de l&#x27;**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pour appliquer la LUT

### Choisir un dégradé de couleurs

**Sélection d&#x27;un dégradé :**

1. Dans le panneau LUT, repérez la**barre de dégradé de couleurs**

2. Passez votre souris dessus pour afficher les préréglages de dégradé disponibles
3. Sélectionnez le dégradé souhaité
4. L&#x27;image **s&#x27;actualise immédiatement** avec les nouvelles couleurs lorsque la case Index est cochée

{% hint style="success" %}
**Meilleure pratique** : Pour les indices de végétation tels que NDVI, le dégradé Red-Jaune-Green est le plus intuitif car il correspond aux associations de couleurs naturelles (vert = sain, jaune = modéré, rouge = stressé).
{% endhint %}

### Réglage des classes de couleurs

Le **contrôle Classes**détermine le nombre de paliers de couleurs distincts qui apparaissent dans votre dégradé :**Options de nombre de classes :*** **2 à 5 classes** : catégories très larges, zones distinctes
* **6 à 10 classes** : équilibré, idéal pour la classification
* **11 à 20 classes** : dégradés lisses, aspect continu
* **20 classes et plus** : Presque continu, douceur maximale**Comment régler :**

1. Dans le panneau LUT, repérez les**carreaux de couleur sous la barre de dégradé**

2. Ajustez le nombre de classes en ajoutant des classes à l&#x27;aide du bouton +
3. Supprimez des classes en double-cliquant sur un carreau de couleur
4. Le dégradé s&#x27;actualise **en temps réel** sur l&#x27;image**Effet sur la visualisation :*** **Moins de classes** (3-5) : crée des zones distinctes, classification simplifiée, catégories plus faciles à distinguer
* **Nombre moyen de classes** (6-10) : approche équilibrée, convient à la plupart des applications
* **Plus de classes** (15-20) : transitions fluides, variations détaillées, aspect photographique**Quand l&#x27;utiliser :*** **Peu de classes (3-5)** : diapositives de présentation, cartes de classification, rapports simples
* **Nombre moyen de classes (6-10)** : analyse générale, détails équilibrés, rapports standard
* **Nombreuses classes (15-20)** : analyse scientifique, inspection détaillée, résultats de qualité publication

### Réglage fin des plages de valeurs

Les **commandes de plage de valeurs**déterminent quelles valeurs d&#x27;indice correspondent à quelles couleurs dans votre dégradé :**Commandes de plage dans le panneau LUT :*** **Valeur minimale** : limite inférieure de l&#x27;échelle de couleurs
* **Valeur maximale** : limite supérieure de l&#x27;échelle de couleurs
* **Valeurs intermédiaires** : réparties automatiquement entre la valeur minimale et la valeur maximale (en fonction du nombre de classes)

#### Réglage des valeurs minimales et maximales

**Pour ajuster les plages de valeurs :**

1. Dans le panneau LUT, repérez les champs de saisie**Valeur minimale**et**Valeur maximale**

2. Cliquez sur le champ**Valeur minimale**

3. Saisissez la valeur minimale souhaitée (par exemple, `0.2`)
4. Appuyez sur **Entrée** ou cliquez en dehors du champ
5. Répétez l&#x27;opération pour le champ **Valeur max** (par exemple, `0.9`)
6. La visualisation **s&#x27;actualise immédiatement**{% hint style="info" %}**Mise à l&#x27;échelle automatique** : Lorsque vous appliquez une table de conversion (LUT) pour la première fois, Chloros définit automatiquement les valeurs min/max en fonction de la plage de données réelle de l&#x27;image. Vous pouvez ensuite réduire cette plage pour vous concentrer sur des plages de valeurs spécifiques qui vous intéressent.
{% endhint %}

**Exemples de réglages de plage NDVI :*** **Plage complète** : de `-1.0` à `1.0` (afficher toutes les valeurs possibles)
* **Axé sur la végétation** : de `0.2` à `0.9` (exclure le sol nu et l&#x27;eau)
* **Végétation saine uniquement** : de `0.5` à `0.9` (mettre en évidence uniquement les plantes vigoureuses)
* **Détection du stress** : de `0.2` à `0.5` (mettre l&#x27;accent sur les zones problématiques)
* **Plage personnalisée** : ajustez en fonction des valeurs de pixels que vous avez observées**Pourquoi ajuster les plages ?*** **Augmenter le contraste** dans votre zone d&#x27;intérêt
* **Exclure les valeurs non pertinentes** (par exemple, les plans d&#x27;eau, le sol nu)
* **Normaliser la visualisation** sur plusieurs images ou dates
* **Mettre en évidence les différences subtiles** au sein d&#x27;une plage de valeurs étroite

### Écarter les valeurs hors plage

Lorsque les valeurs de pixels se situent en dehors de la plage min/max que vous avez définie, vous pouvez contrôler leur affichage à l&#x27;aide des **modes d&#x27;écrêtage**.

#### **Options de mode d&#x27;écrêtage disponibles :**

#### 1. Minimum et maximum

* Pixels **inférieurs au minimum**→ affichage à l&#x27;aide de la**première couleur** du dégradé (par exemple, le rouge)
* Pixels **supérieurs au maximum**→ affichage à l&#x27;aide de la**dernière couleur** du dégradé (par exemple, le vert)
* **Cas d&#x27;utilisation** : mettre en évidence les extrêmes, afficher la plage complète des données avec des couleurs saturées aux limites
* **Exemple** : les valeurs NDVI inférieures à 0,2 apparaissent toutes en rouge, les valeurs supérieures à 0,9 apparaissent toutes en vert

#### 2. Arrière-plan transparent

* Les pixels **en dehors de la plage**deviennent**entièrement transparents*** Seuls les pixels **dans la plage** affichent le dégradé de couleurs
* **Cas d&#x27;utilisation** : superposition SIG, isolation de plages de valeurs spécifiques, mise en évidence des seules zones d&#x27;intérêt
* **Exemple** : afficher uniquement les valeurs NDVI comprises entre 0,4 et 0,7 en couleur, tout le reste en transparent

{% hint style="warning" %}
**Limitation de transparence** : les pixels transparents apparaîtront dans la couleur d&#x27;arrière-plan dans la visionneuse. Lors de l&#x27;exportation pendant le traitement, la transparence est conservée au format PNG mais pas au format JPG.
{% endhint %}

#### 3. Arrière-plan de l&#x27;index

* Les pixels **hors de la plage**s&#x27;affichent en**niveaux de gris** (affichant les valeurs brutes de l&#x27;index)
* Les pixels **dans la plage**affichent un**dégradé de couleurs*** **Cas d&#x27;utilisation** : mise en évidence subtile, conservation du contexte tout en soulignant les zones d&#x27;intérêt
* **Exemple** : Mettre en évidence en couleur la végétation stressée (NDVI 0,3-0,5) tout en affichant les zones saines en gris

#### 4. Arrière-plan d&#x27;origine

* Les pixels **hors de la plage**affichent l&#x27;**image multispectrale d&#x27;origine*** Les pixels **dans la plage**affichent un**dégradé de couleurs*** **Cas d&#x27;utilisation** : Le plus intuitif : combine le contexte naturel de l&#x27;image avec une superposition de couleurs analytiques
* **Exemple** : visualisez l&#x27;aspect réel du champ/de la culture avec les zones de stress superposées et codées par couleur

### Choisir le bon mode de découpage

| Mode de découpage              | Idéal pour                                   | Style de visualisation          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum et maximum**    | Affichage complet des données, analyse scientifique     | Tous les pixels colorés           |
| **Arrière-plan transparent** | Superpositions SIG, isolation de plages spécifiques    | Couleur dans la plage, blanc au-delà |
| **Arrière-plan indexé**       | Mise en évidence subtile, préservation du contexte des données  | Couleur sur la plage, gris au-delà  |
| **Arrière-plan d&#x27;origine**    | Rapports, présentations, analyse intuitive | Couleur sur la plage, photo au-delà |

### Création de couleurs LUT personnalisées

Pour un contrôle total sur votre visualisation, vous pouvez créer des **dégradés de couleurs personnalisés** en modifiant chaque étape de couleur.**Pour créer un dégradé personnalisé :**

1. Dans le panneau LUT, repérez la**barre d&#x27;aperçu du dégradé**

2. Recherchez les**carreaux d&#x27;échantillons de couleur** sous le dégradé
3. **Cliquez sur un arrêt de couleur** pour le sélectionner
4. Un **sélecteur de couleur** s&#x27;ouvre
5. Choisissez une nouvelle couleur à l&#x27;aide de :
   * **Roue chromatique** : sélection visuelle des couleurs
   * **Curseurs RGB/HSV** : contrôle précis des couleurs
   * **Saisie du code hexadécimal** : spécification exacte de la couleur (par exemple, `#FF0000` pour le rouge)
6. Cliquez en dehors du sélecteur de couleurs **pour appliquer la nouvelle couleur**

7. Le dégradé**s&#x27;actualise immédiatement** sur l&#x27;image**Ajouter ou supprimer des arrêts de couleur :*** **Ajouter un arrêt** : cliquez sur l&#x27;icône + pour ajouter un nouvel échantillon à la fin
* **Supprimer un arrêt** : double-cliquez sur le carré de couleur pour supprimer l&#x27;échantillon**Stratégies de personnalisation :*** **Inverser le dégradé** : inversez l&#x27;ordre des couleurs pour inverser la signification (par exemple, vert = faible, rouge = élevé)
* **Couleurs de marque** : Adaptez-vous à la palette de couleurs de votre organisation pour les rapports
* **Adapté aux daltoniens** : Utilisez des combinaisons orange-bleu ou violet-jaune
* **Optimisation de l&#x27;impression** : Choisissez des couleurs qui fonctionnent aussi bien en impression couleur qu&#x27;en niveaux de gris
* **Seuils multiples** : Utilisez des couleurs distinctes à des seuils de valeur spécifiques pour la classification

{% hint style="info" %}
**Enregistrement des dégradés personnalisés** : les dégradés personnalisés peuvent être enregistrés et réutilisés. Cliquez sur l&#x27;icône d&#x27;enregistrement dans le panneau LUT pour conserver vos schémas de couleurs personnalisés en vue d&#x27;une utilisation future.
{% endhint %}

***

## Flux de travail interactif

### Mises à jour en temps réel

Tous les réglages de la LUT dans le bac à sable mettent à jour l&#x27;image **instantanément et de manière interactive** :

* **Changer de calque** → L&#x27;image change immédiatement
* **Sélectionner un dégradé** → Les couleurs sont mises à jour instantanément
* **Ajuster la plage de valeurs** → Le contraste change en temps réel
* **Modifier les classes** → La fluidité du dégradé est mise à jour immédiatement
* **Modifier le détourage** → L&#x27;affichage de l&#x27;arrière-plan change instantanément
* **Modifier les couleurs** → Le dégradé personnalisé s&#x27;applique immédiatement**Pas besoin de bouton « Appliquer »** : toutes les modifications sont en direct et interactives !

{% hint style="success" %}
**Retour en direct** : le retour visuel instantané vous permet d&#x27;expérimenter rapidement différents paramètres jusqu&#x27;à ce que vous trouviez la visualisation optimale pour vos besoins d&#x27;analyse.
{% endhint %}

### Workflow d&#x27;affinement itératif

**Workflow type d&#x27;optimisation de la LUT :**

1.**Sélectionnez le calque d&#x27;index** (par ex., RAW (Réflectance))
2. **Appliquez l&#x27;index** - Choisissez le filtre de l&#x27;appareil photo et la formule d&#x27;index, faites glisser les cercles colorés à l&#x27;emplacement approprié dans la formule d&#x27;index
3. **Appliquez le dégradé de LUT** - Commencez par le préréglage Red-Yellow-Green
4. **Inspectez les valeurs des pixels** - Déplacez le curseur, notez les plages de valeurs
5. **Ajustez les valeurs min/max** - Resserrez la plage pour vous concentrer sur la végétation (par exemple, de 0,2 à 0,9)
6. **Choisissez le découpage** - Essayez « Original Background » pour le contexte
7. **Affinez les couleurs** - Personnalisez le dégradé si nécessaire pour mettre l&#x27;accent sur des éléments spécifiques
8. **Finalisez les paramètres**- Enregistrez les paramètres et copiez-les dans les paramètres du projet pour le traitement d&#x27;exportation

### Inspection des valeurs de pixels

Il est essentiel de comprendre les valeurs réelles des pixels pour définir des plages de LUT efficaces :**Comment inspecter les valeurs :**

1. Les valeurs de pixels s&#x27;affichent lorsque la case**Index**ou les cases**Index**et**LUT** sont cochées.
2. **Déplacez votre curseur** sur différentes zones de l&#x27;image
3. **Observez les valeurs de pixels** affichées dans la légende lorsque vous survolez l&#x27;image
4. Zoomez pour voir les pixels individuels mis en évidence avec une valeur flottante
5. **Notez** les plages de valeurs pour les différentes caractéristiques :
   * **Végétation saine** : par exemple, NDVI 0,55-0,85
   * **Végétation stressée** : par exemple, NDVI 0,30-0,50
   * **Sol nu** : par exemple, NDVI 0,05-0,25
   * **Eau** (si présente) : par exemple, NDVI -0,05 à 0,10**Utilisation des valeurs de pixels pour définir les plages de la table de conversion (LUT) :**Après avoir examiné les valeurs des pixels, ajustez les valeurs min/max de votre LUT en conséquence :**Exemple de scénario :*** **Observation** : Valeurs du sol = 0,05-0,25, Sol stressé = 0,25-0,50, Sol sain = 0,50-0,85
* **Objectif** : Visualiser uniquement la santé des plantes (exclure le sol)
* **Paramètres de la LUT** : Min = `0.25`, Max = `0.85`
* **Écart** : « Arrière-plan d&#x27;origine » pour voir le sol dans sa couleur naturelle
* **Résultat** : le dégradé de couleurs s&#x27;applique uniquement à la végétation, le sol s&#x27;affiche comme sur l&#x27;image d&#x27;origine

{% hint style="info" %}
**Plage dynamique** : les différentes cultures, saisons et stades de croissance auront des plages de valeurs différentes. Vérifiez toujours les valeurs des pixels dans votre ensemble de données spécifique avant de définir les plages de la table de conversion (LUT).
{% endhint %}

***

## Indices personnalisés (Chloros+)

### Création de formules d&#x27;indices personnalisés

{% hint style="info" %}
**Où créer**: les indices personnalisés peuvent être configurés dans les**Paramètres du projet** avant le traitement, ainsi que dans la barre latérale du bac à sable de la visionneuse d&#x27;images.
{% endhint %}

**Pour créer un indice personnalisé :**

1.**Ouvrez les Paramètres du projet** (avant le traitement) ou la barre latérale du bac à sable de la visionneuse d&#x27;images
2. Accédez au **menu déroulant Formule d&#x27;indice**

3. Recherchez l&#x27;option**« Personnalisé »** (vous devez être connecté avec une licence Chloros+)
4. **Définissez votre formule** à l&#x27;aide des variables de bande :
   * Noms des bandes : `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Opérateurs : `+`, `-`, `*`, `/`, `^` (exposant)
   * Fonctions : `sqrt()`, `abs()`, etc. (si prises en charge)
   * Parenthèses : `()` pour l&#x27;ordre des opérations
5. **Nommez votre indice** (par exemple, « MyIndex » ou « CustomNDVI »)
6. **Enregistrez la configuration**

**Exemples de formules personnalisées :**

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
**Validation de la formule** : assurez-vous que votre formule utilise les bandes disponibles sur votre caméra. Par exemple, RedEdge n&#x27;est disponible que sur les caméras équipées d&#x27;un filtre RedEdge.
{% endhint %}

***

## Étapes suivantes

Maintenant que vous comprenez le bac à sable Index/LUT :

* **Appliquez-la au traitement** : utilisez les paramètres découverts dans [Paramètres du projet](../project-settings/project-settings.md)
* **Traitement par lots** : appliquez les indices optimisés à l&#x27;ensemble des données
* **En savoir plus** : consultez [Formules d&#x27;indices multispectraux](../project-settings/multispectral-index-formulas.md)

Documentation associée :

* [**Calques d&#x27;image**](image-layers.md) - Gestion et visualisation des calques
* [**Ouverture d&#x27;une image en plein écran**](opening-an-image-full-screen.md) - Notions de base sur la visionneuse d&#x27;images
* [**Traitement des images (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Flux de travail complet de traitement
