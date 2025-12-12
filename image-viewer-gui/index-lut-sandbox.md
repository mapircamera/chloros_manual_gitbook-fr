# Sandbox Index/LUT

La Sandbox Index/LUT est un espace de travail interactif intégré à la visionneuse d&#x27;images Chloros qui vous permet d&#x27;expérimenter en temps réel des calculs d&#x27;indices multispectraux et des visualisations en couleur. Cet outil puissant vous aide à tester différents indices, à affiner les plages de valeurs et à créer des visualisations prêtes à être publiées sans avoir à retraiter l&#x27;ensemble de votre jeu de données.

## Qu&#x27;est-ce que le bac à sable Index/LUT ?

### Objectif

Le bac à sable offre :

* **Calcul d&#x27;indice en temps réel** - Appliquez instantanément n&#x27;importe quel indice de végétation
* **Ajustement LUT interactif** - Affinez les dégradés et les plages de couleurs
* **Optimisation du flux de travail** - Déterminez les meilleurs paramètres avant le traitement par lots

### Sandbox vs traitement de projet

**Sandbox Index/LUT (interactif) :**

* Une seule image à la fois
* Réaction instantanée
* Expérimental et itératif
* Aucune modification permanente des fichiers
* Parfait pour explorer et tester

**Traitement de projet (par lots) :**

* Ensemble de données complet à la fois
* Paramètres préconfigurés
* Fichiers de sortie permanents
* Prend beaucoup de temps
* Idéal lorsque les paramètres sont finalisés

{% hint style=&quot;success&quot; %}
**Meilleur flux de travail** : utilisez le bac à sable pour expérimenter et trouver les paramètres d&#x27;index et de LUT optimaux, puis appliquez ces paramètres lors du traitement du projet à l&#x27;ensemble de votre ensemble de données.
{% endhint %}

***

## Utilisation du bac à sable Index/LUT

### Comprendre les indices précalculés

Dans Chloros, les indices peuvent être appliqués pendant le traitement du projet. Pour déterminer les paramètres d&#x27;index et de LUT que vous souhaitez appliquer aux exportations, le plus simple est d&#x27;utiliser le bac à sable de la visionneuse d&#x27;images.

Le bac à sable vous permet de :

* **Appliquer de nouveaux indices et dégradés de couleurs (LUT)** pour visualiser les données
* **Ajuster les paramètres de visualisation** de manière interactive
* **Afficher** les images d&#x27;index déjà calculées
* **Inspecter** les valeurs de pixels à tous les niveaux de zoom

### Ouverture du bac à sable

Le bac à sable Index/LUT est accessible dans l&#x27;onglet **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Cliquez sur une image dans la grille d&#x27;images du navigateur de fichiers pour l&#x27;ouvrir dans l&#x27;onglet **Visionneuse d&#x27;images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Cliquez sur l&#x27;onglet **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pour ouvrir la barre latérale gauche si elle n&#x27;est pas déjà ouverte

### Sélectionner une image à laquelle appliquer un index/LUT

Pour travailler avec un index dans la visionneuse d&#x27;images <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Ouvrez une image** à partir de la grille d&#x27;images principale en cliquant dessus
2. L&#x27;onglet **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> s&#x27;ouvre alors.
3. Cliquez sur le **menu déroulant Layer** (en haut à droite de la visionneuse).
4. Sélectionnez le calque dans le menu déroulant :
   * RAW (réflectance)

### Appliquer un index à une image

Une fois que l&#x27;image est en plein écran et que la barre latérale **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> est ouverte :

1. Cochez la case Index en haut de la barre latérale
2. Choisissez le filtre de votre appareil photo dans le menu déroulant de gauche
3. Choisissez la formule d&#x27;index souhaitée dans le menu déroulant de droite
4. Faites glisser les cercles de couleur du canal de filtre vers les emplacements de la formule d&#x27;index ci-dessous
5. Une fois la formule validée, l&#x27;image sera mise à jour et affichera les valeurs d&#x27;index
6. Déplacez le curseur de votre souris pour voir les valeurs à l&#x27;emplacement du curseur
7. Zoomez pour voir les pixels individuels et leurs valeurs associées.

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

Pour une documentation complète sur les formules d&#x27;indice, consultez [Formules d&#x27;indice multispectral](../project-settings/multispectral-index-formulas.md).

***

## Utilisation des tables de correspondance (LUT)

### Qu&#x27;est-ce qu&#x27;une table de correspondance ?

Une **table de correspondance (LUT)** associe des valeurs d&#x27;indice numériques à des couleurs à des fins de visualisation :

* **Entrée** : valeur de pixel de l&#x27;indice (par exemple, NDVI 0,65)
* **Sortie** : couleur RGB (par exemple, vert vif)
* **Objectif** : faciliter la visualisation et l&#x27;interprétation des modèles

**Table de correspondance en niveaux de gris ou en couleurs :**

* Niveaux de gris : scientifique et neutre, affiche les données brutes
* Table de correspondance en couleurs : intuitive et percutante, met en évidence les modèles et les différences

{% hint style=&quot;success&quot; %}
**Puissance de visualisation** : l&#x27;application d&#x27;une table de correspondance des couleurs à une image indexée en niveaux de gris facilite considérablement l&#x27;identification des modèles, des anomalies et des zones d&#x27;intérêt en un coup d&#x27;œil.
{% endhint %}

### Application d&#x27;une table de conversion à une image indexée

Une fois que vous disposez d&#x27;une image indexée affichant

1. Cliquez sur le <img src="../.gitbook/assets/image.png" alt="" data-size="line"> bouton « +Ajouter une table de conversion »
2. Sélectionnez le dégradé de couleurs
3. Ajustez les points d&#x27;extrémité min/max de l&#x27;écrêtage
4. Ajustez le mode d&#x27;écrêtage
5. Cochez la case Index dans la barre latérale **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pour appliquer la LUT.

### Choisir un dégradé de couleurs

**Sélectionner un dégradé :**

1. Dans le panneau LUT, localisez la **barre de dégradé coloré**.
2. Passez votre souris dessus pour afficher les préréglages de dégradé disponibles.
3. Sélectionnez le dégradé souhaité.
4. L&#x27;image **s&#x27;actualise immédiatement** avec les nouvelles couleurs lorsque la case Index est cochée.

{% hint style=&quot;success&quot; %}
**Meilleure pratique** : pour les indices de végétation tels que NDVI, le dégradé Red-Yellow-Green est le plus intuitif, car il correspond aux associations de couleurs naturelles (vert = sain, jaune = modéré, rouge = stressé).
{% endhint %}

### Réglage des classes de couleurs

Le **contrôle Classes** détermine le nombre d&#x27;étapes de couleurs distinctes qui apparaissent dans votre dégradé :

**Options de nombre de classes :**

* **2 à 5 classes** : catégories très larges, zones distinctes
* **6 à 10 classes** : équilibré, idéal pour la classification
* **11 à 20 classes** : dégradés lisses, apparence continue
* **Plus de 20 classes** : quasi continu, fluidité maximale

**Comment ajuster :**

1. Dans le panneau LUT, localisez les **carrés d&#x27;échantillons de couleurs sous la barre de dégradé**.
2. Ajustez le nombre de classes en ajoutant avec le bouton +.
3. Supprimez le nombre de classes en double-cliquant sur un échantillon de couleur.
4. Le dégradé se met à jour **en temps réel** sur l&#x27;image.

**Effet sur la visualisation :**

* **Moins de classes** (3-5) : Crée des zones distinctes, une classification simplifiée, des catégories plus faciles à distinguer.
* **Classes moyennes** (6-10) : approche équilibrée, adaptée à la plupart des applications.
* **Plus de classes** (15-20) : transitions fluides, variations détaillées, aspect photographique.

**Quand l&#x27;utiliser :**

* **Peu de classes (3-5)** : diapositives de présentation, cartes de classification, rapports simples.
* **Classes moyennes (6-10)** : analyse générale, détails équilibrés, rapports standard
* **Nombreuses classes (15-20)** : analyse scientifique, inspection détaillée, résultats de qualité publication

### Réglage fin des plages de valeurs

Les **commandes de plage de valeurs** déterminent quelles valeurs d&#x27;index correspondent à quelles couleurs dans votre dégradé :

**Commandes de plage dans le panneau LUT :**

* **Valeur minimale** : limite inférieure de l&#x27;échelle de couleurs
* **Valeur maximale** : limite supérieure de l&#x27;échelle de couleurs
* **Valeurs intermédiaires** : réparties automatiquement entre la valeur minimale et la valeur maximale (en fonction du nombre de classes)

#### Réglage des valeurs minimales/maximales

**Pour régler les plages de valeurs :**

1. Dans le panneau LUT, localisez les champs de saisie **Valeur minimale** et **Valeur maximale**
2. Cliquez sur le champ **Valeur minimale**
3. Saisissez la valeur minimale souhaitée (par exemple, `0.2`)
4. Appuyez sur **Entrée** ou cliquez en dehors du champ
5. Répétez l&#x27;opération pour le champ **Valeur maximale** (par exemple, `0.9`)
6. La visualisation **s&#x27;actualise immédiatement**

{% hint style=&quot;info&quot; %}
**Mise à l&#x27;échelle automatique** : lorsque vous appliquez une table de correspondance pour la première fois, Chloros définit automatiquement les valeurs minimale et maximale en fonction de la plage de données réelle de l&#x27;image. Vous pouvez ensuite réduire cette plage pour vous concentrer sur des plages de valeurs spécifiques qui vous intéressent.
{% endhint %}

**Exemple d&#x27;ajustements de plage NDVI :**

* **Plage complète** : `-1.0` à `1.0` (afficher toutes les valeurs possibles)
* **Axée sur la végétation** : `0.2` à `0.9` (exclure le sol nu et l&#x27;eau)
* **Végétation saine uniquement** : `0.5` à `0.9` (mettre en évidence uniquement les plantes vigoureuses)
* **Détection du stress** : `0.2` à `0.5` (mettre l&#x27;accent sur les zones problématiques)
* **Plage personnalisée** : ajuster en fonction des valeurs de pixels observées

**Pourquoi ajuster les plages ?**

* **Augmenter le contraste** dans votre zone d&#x27;intérêt
* **Excluez les valeurs non pertinentes** (par exemple, les plans d&#x27;eau, le sol nu)
* **Standardisez la visualisation** sur plusieurs images ou dates
* **Mettez en évidence les différences subtiles** dans une plage de valeurs étroite

### Découpage des valeurs hors plage

Lorsque les valeurs de pixels se situent en dehors de la plage min/max que vous avez définie, vous pouvez contrôler leur affichage à l&#x27;aide des **modes de découpage**.

#### **Options de mode de découpage disponibles :**

#### 1. Minimum et maximum

* Pixels **inférieurs au minimum** → affichage à l&#x27;aide de la **première couleur** du dégradé (par exemple, le rouge)
* Pixels **supérieurs au maximum** → affichage à l&#x27;aide de la **dernière couleur** du dégradé (par exemple, le vert)
* **Cas d&#x27;utilisation** : mettre en évidence les extrêmes, afficher la plage complète des données avec des couleurs saturées aux limites
* **Exemple** : les valeurs NDVI inférieures à 0,2 apparaissent toutes en rouge, les valeurs supérieures à 0,9 apparaissent toutes en vert

#### 2. Arrière-plan transparent

* Les pixels **en dehors de la plage** deviennent **entièrement transparents**
* Seuls les pixels **dans la plage** affichent un dégradé de couleurs
* **Cas d&#x27;utilisation** : superposition SIG, isolation de plages de valeurs spécifiques, mise en évidence des seules zones d&#x27;intérêt
* **Exemple** : afficher uniquement NDVI 0,4-0,7 en couleur, tout le reste étant transparent

{% hint style=&quot;warning&quot; %}
**Limitation de la transparence** : les pixels transparents apparaîtront comme la couleur d&#x27;arrière-plan dans la visionneuse. Lors de l&#x27;exportation pendant le traitement, la transparence est conservée au format PNG, mais pas au format JPG.
{% endhint %}

#### 3. Arrière-plan de l&#x27;index

* Les pixels **hors plage** s&#x27;affichent en **niveaux de gris** (affichant les valeurs brutes de l&#x27;index)
* Les pixels **dans la plage** affichent un **dégradé de couleurs**
* **Cas d&#x27;utilisation** : mise en évidence subtile, maintien du contexte tout en mettant en valeur les zones d&#x27;intérêt
* **Exemple** : mise en évidence en couleur de la végétation stressée (NDVI 0,3-0,5) tout en affichant les zones saines en gris

#### 4. Arrière-plan d&#x27;origine

* Les pixels **hors plage** affichent l&#x27;**image multispectrale d&#x27;origine**
* Les pixels **dans la plage** affichent un **dégradé de couleurs**
* **Cas d&#x27;utilisation** : le plus intuitif - combine le contexte naturel de l&#x27;image avec une superposition de couleurs analytiques
* **Exemple** : voir l&#x27;apparence réelle du champ/de la culture avec une superposition des zones de stress codées par couleur

### Choisir le bon mode de découpage

| Mode de découpage              | Idéal pour                                   | Style de visualisation          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum et maximum**    | Affichage complet des données, analyse scientifique     | Tous les pixels colorés           |
| **Arrière-plan transparent** | Superpositions SIG, isolation de plages spécifiques    | Couleur sur la plage, vide au-delà |
| **Arrière-plan indexé**       | Accentuation subtile, maintien du contexte des données  | Couleur sur la plage, gris au-delà  |
| **Arrière-plan d&#x27;origine**    | Rapports, présentations, analyse intuitive | Couleur sur la plage, photo au-delà |

### Création de couleurs LUT personnalisées

Pour un contrôle total de votre visualisation, vous pouvez créer des **dégradés de couleurs personnalisés** en modifiant individuellement les arrêts de couleur.

**Pour créer un dégradé personnalisé :**

1. Dans le panneau LUT, localisez la **barre d&#x27;aperçu du dégradé**
2. Recherchez les **carrés d&#x27;échantillons de couleurs** sous le dégradé
3. **Cliquez sur un arrêt de couleur** pour le sélectionner
4. Un **sélecteur de couleurs** s&#x27;ouvre
5. Choisissez une nouvelle couleur à l&#x27;aide :
   * **de la roue chromatique** : sélection visuelle des couleurs
   * **des curseurs RGB/HSV** : contrôle précis des couleurs
   * **de la saisie du code hexadécimal** : spécification exacte de la couleur (par exemple, `#FF0000` pour le rouge)
6. Cliquez en dehors du sélecteur de couleurs **pour appliquer la nouvelle couleur**
7. Le dégradé **est immédiatement mis à jour** sur l&#x27;image

**Ajouter ou supprimer des arrêts de couleur :**

* **Ajouter un arrêt** : cliquez sur l&#x27;icône + pour ajouter un nouvel échantillon à la fin
* **Supprimer un arrêt** : double-cliquez sur le carré de couleur pour supprimer l&#x27;échantillon

**Stratégies de personnalisation :**

* **Inverser le dégradé** : inversez l&#x27;ordre des couleurs pour inverser la signification (par exemple, vert = faible, rouge = élevé)
* **Couleurs de la marque** : adaptez la palette de couleurs de votre organisation pour les rapports
* **Adapté aux daltoniens** : utilisez des combinaisons orange-bleu ou violet-jaune
* **Optimisation de l&#x27;impression** : choisissez des couleurs qui fonctionnent à la fois en impression couleur et en niveaux de gris
* **Multi-seuils** : utilisez des couleurs distinctes à des seuils de valeur spécifiques pour la classification

{% hint style=&quot;info&quot; %}
**Enregistrement des dégradés personnalisés** : les dégradés personnalisés peuvent être enregistrés et réutilisés. Cliquez sur l&#x27;icône d&#x27;enregistrement dans le panneau LUT pour conserver vos schémas de couleurs personnalisés en vue d&#x27;une utilisation future.
{% endhint %}

***

## Flux de travail interactif

### Mises à jour en temps réel

Tous les réglages LUT dans le bac à sable mettent à jour l&#x27;image **instantanément et de manière interactive** :

* **Changer de calque** → L&#x27;image change immédiatement
* **Sélectionner un dégradé** → Les couleurs sont mises à jour instantanément
* **Ajuster la plage de valeurs** → Le contraste change en temps réel
* **Changer de classe** → La fluidité du dégradé est mise à jour immédiatement
* **Modifier le découpage** → L&#x27;affichage de l&#x27;arrière-plan change instantanément
* **Modifier les couleurs** → Le dégradé personnalisé s&#x27;applique immédiatement

**Aucun bouton « Appliquer » n&#x27;est nécessaire** : toutes les modifications sont en direct et interactives !

{% hint style=&quot;success&quot; %}
**Retour en direct** : le retour visuel instantané vous permet d&#x27;expérimenter rapidement différents paramètres jusqu&#x27;à ce que vous trouviez la visualisation optimale pour vos besoins d&#x27;analyse.
{% endhint %}

### Workflow de raffinement itératif

**Workflow typique d&#x27;optimisation LUT :**

1. **Sélectionnez la couche d&#x27;index** (par exemple, RAW (réflectance))
2. **Appliquez l&#x27;index** - Choisissez le filtre de l&#x27;appareil photo et la formule d&#x27;index, faites glisser les cercles colorés vers l&#x27;emplacement approprié dans la formule d&#x27;index
3. **Appliquer le gradient LUT** - Commencez avec le préréglage Red-Yellow-Green
4. **Inspecter les valeurs des pixels** - Déplacez le curseur et notez les plages de valeurs
5. **Ajuster min/max** - Réduisez la plage pour vous concentrer sur la végétation (par exemple, 0,2 à 0,9)
6. **Choisissez le recadrage** - Essayez « Original Background » (Arrière-plan d&#x27;origine) pour le contexte.
7. **Affinez les couleurs** - Personnalisez le dégradé si nécessaire pour mettre l&#x27;accent sur certains éléments.
8. **Finalisez les paramètres** - Documentez les paramètres et copiez-les dans les paramètres du projet pour le traitement de l&#x27;exportation.

### Inspection des valeurs de pixels

Il est essentiel de comprendre les valeurs réelles des pixels pour définir des plages LUT efficaces :

**Comment inspecter les valeurs :**

1. Les valeurs de pixels s&#x27;affichent lorsque la case Index ou les cases Index et LUT sont **cochées**.
2. **Déplacez votre curseur** sur différentes zones de l&#x27;image.
3. **Observez les valeurs de pixels** affichées dans la légende lorsque vous passez le curseur dessus.
4. Zoomez pour voir les pixels individuels mis en évidence avec une valeur flottante.
5. **Notez** les plages de valeurs pour différentes caractéristiques :
   * **Végétation saine** : par exemple, NDVI 0,55-0,85
   * **Végétation stressée** : par exemple, NDVI 0,30-0,50
   * **Sol nu** : par exemple, NDVI 0,05-0,25
   * **Eau** (si présente) : par exemple, NDVI -0,05 à 0,10

**Utilisation des valeurs de pixels pour définir les plages LUT :**

Après avoir inspecté les valeurs de pixels, ajustez vos valeurs minimales/maximales LUT en conséquence :

**Exemple de scénario :**

* **Observation** : valeurs du sol = 0,05-0,25, stressé = 0,25-0,50, sain = 0,50-0,85
* **Objectif** : visualiser uniquement la santé des plantes (exclure le sol)
* **Paramètres LUT** : Min = `0.25`, Max = `0.85`
* **Découpage** : « Arrière-plan d&#x27;origine » pour voir le sol dans sa couleur naturelle
* **Résultat** : le dégradé de couleurs s&#x27;applique uniquement à la végétation, le sol s&#x27;affiche comme dans l&#x27;image d&#x27;origine

{% hint style=&quot;info&quot; %}
**Plage dynamique** : les différentes cultures, saisons et stades de croissance auront des plages de valeurs différentes. Vérifiez toujours les valeurs de pixels dans votre ensemble de données spécifique avant de définir les plages LUT.
{% endhint %}

***

## Indices personnalisés (Chloros+)

### Création de formules d&#x27;indice personnalisées

{% hint style=&quot;info&quot; %}
**Où créer** : les indices personnalisés peuvent être configurés dans les **Paramètres du projet** avant le traitement, ainsi que dans la barre latérale du bac à sable de la visionneuse d&#x27;images.
{% endhint %}

**Pour créer un index personnalisé :**

1. **Ouvrez les paramètres du projet** (avant le traitement) ou la barre latérale du bac à sable de la visionneuse d&#x27;images
2. Accédez au **menu déroulant Formule d&#x27;index**
3. Recherchez l&#x27;option **« Personnalisé »** (vous devez être connecté avec une licence Chloros+)
4. **Définissez votre formule** à l&#x27;aide des variables de bande :
   * Noms des bandes : `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Opérateurs : `+`, `-`, `*`, `/`, `^` (exposant)
   * Fonctions : `sqrt()`, `abs()`, etc. (si pris en charge)
   * Parenthèses : `()` pour l&#x27;ordre des opérations
5. **Nommez votre index** (par exemple, « MyIndex » ou « CustomNDVI »)
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

{% hint style=&quot;warning&quot; %}
**Validation de la formule** : assurez-vous que votre formule utilise les bandes disponibles dans votre caméra. Par exemple, RedEdge n&#x27;est disponible que sur les caméras équipées d&#x27;un filtre RedEdge.
{% endhint %}

***

## Étapes suivantes

Maintenant que vous comprenez le bac à sable Index/LUT :

* **Appliquer au traitement** : utilisez les paramètres découverts dans [Paramètres du projet](../project-settings/project-settings.md)
* **Traitement par lots** : appliquez les indices optimisés à l&#x27;ensemble des données
* **En savoir plus** : lisez [Formules d&#x27;indice multispectral](../project-settings/multispectral-index-formulas.md)

Documentation connexe :

* [**Couches d&#x27;image**](image-layers.md) - Gestion et visualisation des couches
* [**Ouverture d&#x27;une image en plein écran**](opening-an-image-full-screen.md) - Principes de base de la visionneuse d&#x27;images
* [**Traitement des images (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Flux de travail complet
