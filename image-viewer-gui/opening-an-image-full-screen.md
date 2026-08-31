# Ouvrir une image en plein écran

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Une image ouverte en plein écran, avec le sélecteur de calques en haut à droite</p></figcaption></figure>

La visionneuse d’images Chloros est l’interface plein écran permettant de visualiser, d’inspecter et de mesurer vos images. C’est là que vous pouvez lire les **valeurs réelles des pixels** — DN par canal, pourcentage de réflectance ou radiance en W/m²/sr/nm — plutôt que l’aperçu étiré affiché à l’écran.

## Accéder à la visionneuse d’images

### À partir du navigateur de fichiers

1. Ouvrez l’onglet **Navigateur de fichiers** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Cliquez sur n’importe quelle **vignette** dans la [grille d’images](image-grid.md)
3. L’image s’ouvre en plein écran dans l’onglet **Visionneuse d’images**

L’image s’ouvre sur le produit affiché par la grille. Si la grille est définie sur `RAW (Reflectance)`, c’est sur ce calque que vous atterrissez.

### Ouvrir la barre latérale de la visionneuse d’images

Cliquez sur l’icône **Visionneuse d’images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> dans la barre latérale gauche pour faire apparaître le panneau d’analyse. Il contient, de haut en bas :

* le nom de l’image et le modèle de la caméra
* le bouton **Exporter/Enregistrer les images** (uniquement lorsqu’un index ou une LUT est actif)
* les cases à cocher **Index**et**LUT** ainsi que le panneau de configuration de l’index — voir [Index/LUT Sandbox](index-lut-sandbox.md)
* le panneau **Valeurs du curseur** : lecture par canal, histogramme de couche et contrôle GSD***

## Navigation et zoom

### Parcourir les images

* **Image suivante**: le bouton → ou la touche**→** (flèche droite)
* **Image précédente**: le bouton ← ou la touche**←** (flèche gauche)
* **Aller à une image spécifique** : revenir à la grille et cliquer sur sa vignette

Le zoom et le panoramique sont conservés lorsque vous passez d’une image à l’autre, ce qui vous permet de parcourir une série tout en restant sur la même partie de l’image.

### Zoom

Le zoom s’effectue à l’aide de la **molette de la souris**, par paliers de 15 %, et est ancré sur le curseur : le point situé sous le pointeur reste sous le pointeur. La plage est limitée par la taille de l’image et celle de la fenêtre : vous ne pouvez pas dézoomer au-delà de l’ajustement à la fenêtre, et la limite supérieure est fixée par la résolution native de l’image.

Il n’y a pas de touches dédiées au zoom dans la visionneuse en plein écran. (Dans la grille, **Ctrl + `+` / `−`** redimensionne les vignettes — il s’agit d’une commande différente.)

### Panoramique en mode zoom

Cliquez sur l’image avec le bouton gauche de la souris, maintenez-le enfoncé et faites glisser. Le panoramique est limité : l’image ne peut pas être déplacée hors de l’écran.

### Inspection pixel par pixel à fort grossissement

Dès que le grossissement effectif dépasse **60×**, Chloros affiche un cadre de mise en évidence autour du pixel affiché sous le curseur, ainsi qu&#x27;une valeur flottante à côté de celui-ci.

Le grossissement « effectif » tient compte de la taille du bloc GSD : avec une taille de bloc de 8, la surbrillance apparaît à un grossissement de 7,5× plutôt qu’à 60×, car un pixel affiché correspond déjà à 8 × 8 pixels source. Si vous réduisez le zoom en dessous du seuil, la surbrillance disparaît.

### Raccourcis clavier

| Touche                             | Emplacement       | Action                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Plein écran | Image suivante                          |
| **←**                           | Plein écran | Image précédente                      |
| **Ctrl + R**                    | Plein écran | Réinitialiser l&#x27;index/le bac à sable LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Grille        | Vignettes plus grandes (4 px par pression)  |
| **Ctrl + `−`**                  | Grille        | Vignettes plus petites (4 px par pression) |***

## Valeurs du curseur

Passez le curseur sur l’image et le panneau **Valeurs du curseur** affiche la valeur de chaque canal situé en dessous.

{% hint style="success" %}
**Ce sont les nombres réels du fichier.** La zone d&#x27;affichage à l&#x27;écran est un aperçu étiré de 8 bits qui ne peut pas les fournir ; c&#x27;est pourquoi Chloros échantillonne le fichier produit réel pour l&#x27;affichage. C&#x27;est pourquoi une image brute de 12 bits affiche des valeurs supérieures à 255, et qu’un calque de radiance de type float32 affiche des unités physiques.
{% endhint %}

### Signification des colonnes

Le panneau s’adapte au calque que vous visualisez :

| Calque visualisé              | Colonnes affichées    | Remarques                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Réflectance                        | **DN**et**%** | Le pourcentage est calculé selon l&#x27;échelle propre à ce fichier — voir ci-dessous                                      |
| Radiance                           | **W/m²/sr/nm**   | Valeurs physiques en nombre à virgule flottante ; pas de colonne DN, car un DN n’a pas de sens ici                           |
| Brute / Débayérisée / Aperçu / JPG    | **DN**           | Nombres numériques entiers                                                                         |
| Exportations de réflectance en pourcentage 32 bits | **%** uniquement       | La valeur à virgule flottante stockée n’est pas un DN ; l’arrondir à un entier afficherait donc une valeur dénuée de sens telle que `0` ou `1` |

Chaque ligne est identifiée par le nom du canal correspondant au filtre de votre appareil photo — `Red / Green / NIR` pour RGN, `Orange / Cyan / NIR` pour OCN, `NIR / Green / Blue` pour NGB, `Red / Green / Blue` pour RGB, et le nom de la bande unique pour les caméras RE, NIR et mono M3M. Chaque étiquette comporte un point de couleur correspondant aux cercles de canal utilisés dans l’éditeur de formule d’index.

Les images **d’index et de LUT** enregistrées constituent un cas particulier : elles contiennent des composants de carte de couleurs plutôt que des bandes spectrales ; leurs lignes sont donc libellées `Red / Green / Blue` (ou `Index` pour un fichier d&#x27;index monocanal) plutôt qu’avec les noms de filtres de la caméra.

Lorsqu’un index est actif dans le bac à sable, une ligne supplémentaire apparaît sous les canaux, indiquant la **valeur d’index** au niveau du curseur, avec le nom de l’index et un point blanc qui correspond à son repère sur l’histogramme.

### Le pourcentage de réflectance utilise l’échelle propre à chaque fichier

{% hint style="warning" %}
**Ne partez pas du principe que 65 535 = 100 %.** Chloros stocke la réflectance à différentes échelles selon l’appareil photo qui l’a produite, et la visionneuse détermine celle qui convient à chaque fichier.
{% endhint %}

| Source                  | DN correspondant à une réflectance de 1,0 | Comment il est identifié                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Balise XMP `Chloros:PixelScale=32768` intégrée à chaque exportation de réflectance LATTICE. La marge de 2× permet au fichier de contenir des valeurs de ρ supérieures à 1,0 sans écrêtage |
| **Survey3**|**65535**                      | Pas de balise d&#x27;échelle XMP Chloros — l&#x27;étalonnage Survey3 écrit ρ × dtype-max et écrête à 1,0                                                               |

La visionneuse, le bac à sable d’index/LUT et l’exportation d’index déterminent tous l’échelle via la même implémentation unique ; ainsi, une valeur que vous lisez au niveau du curseur est la même que celle utilisée par les calculs de l’index.

Deux conséquences à connaître :

* Un **pourcentage 32 bits**TIFF stocke DN/65535 sous forme de nombre à virgule flottante, tandis qu’un**8 bits** PNG/JPG stocke DN × 255/65535 — la visionneuse convertit les deux valeurs avant d’afficher un pourcentage.
* Un cas ne peut pas être récupéré : une **exportation TIFF sur 8 bits d’une capture provenant d’une source sur 8 bits** est écrêtée à la plage 0–255 au lieu d’être redimensionnée, et ne comporte délibérément aucune balise d’échelle. Pour ces fichiers, le panneau affiche uniquement les valeurs DN, sans colonne de pourcentage. Il s’agit là d’une réponse honnête, et non d’un bug.***

## L’histogramme du calque

Sous les lignes du curseur se trouve un histogramme en temps réel du calque que vous visualisez, en **256 classes**. Par défaut, il trace une courbe combinée, pondérée selon `(R + 2G + B) / 4` — le même espace de mesure que celui utilisé par les histogrammes de la caméra LATTICE. L&#x27;activation de**RGB**, il est remplacé par des courbes par canal dans les couleurs des canaux, mélangées de manière additive afin que les chevauchements restent lisibles. Les couches monochromes affichent toujours la courbe unique.

L’axe horizontal est exprimé dans l’unité propre à la couche :

| Calque       | Unité de l’axe  | Maximum de l’axe                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Réflectance | pourcentage    | 125 % — la marge de la caméra permet une ρ supérieure à 1,0           |
| Radiance    | W/m²/sr/nm | Le pic propre à l&#x27;image, arrondi à deux chiffres significatifs |
| Données 8 bits | DN         | 255                                                        |
| Données 12 bits | DN         | 4095                                                       |
| Données 16 bits | DN         | 65535                                                      |

Lorsque l&#x27;axe est en DN et atteint l&#x27;un de ces trois plafonds, Chloros connaît également la profondeur de bits de ce que vous visualisez.

Trois boutons se trouvent au-dessus de l&#x27;histogramme :

| Bouton     | Par défaut | Effet                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSEUR** | Activé      | Trace des lignes de repère sur l&#x27;histogramme aux valeurs exactes indiquées dans les lignes ci-dessus, afin que vous puissiez voir où se situe le pixel sous votre curseur dans la distribution de l&#x27;image. En mode RGB, il y a un repère par canal, chacun dans sa propre couleur ; sinon, un seul repère blanc apparaît à la valeur combinée |
| **INDEX**| Activé      | N&#x27;apparaît que lorsqu&#x27;un index est actif. Fait basculer l&#x27;histogramme des bandes sources vers la**distribution des valeurs d&#x27;index**, les deux seuils de coupure étant représentés par des lignes pointillées orange et la valeur d&#x27;index du curseur par une ligne blanche                                                          |
| **RGB**| Désactivé     | Passe de la courbe combinée aux courbes par canal. Sur un capteur monochrome, ce bouton affiche**MONO** et est désactivé — il n’y a qu’un seul canal à afficher                                                                                                                                  |

L’histogramme est calculé à partir des **blocs visibles**, et non des pixels sources situés derrière eux : modifiez la taille de bloc GSD et la distribution est recalculée, de sorte que l’histogramme, le marqueur du curseur et l’image affichée concordent toujours.***

## Taille de bloc GSD

Au bas du panneau se trouve la commande **GSD (px)**: une zone de saisie numérique, un curseur allant de**1 à 256**et un bouton**RESET**.

Il grossit l’image _affichée_ en calculant la moyenne d’un bloc N × N de pixels sources pour former un seul pixel affiché. `1` correspond à la résolution native.

* Cela affecte **l’affichage en plein écran, les vignettes de la grille, l’affichage du curseur et les deux histogrammes** — tout ce qui affiche l’image utilise la même résolution au sol.
* Ce paramètre concerne **uniquement l’affichage**. Le traitement et l’exportation ne sont pas affectés. La seule exception est intentionnelle : l’exportation via [Index/LUT Sandbox](index-lut-sandbox.md) enregistre ce que vous voyez à l’écran ; elle conserve donc la taille de bloc actuelle, et le panneau d’exportation vous avertit lorsque la taille de bloc est supérieure à 1.
* La valeur est stockée **par projet** sous le nom `viewer_display.gsd_bin` dans `project.json` ; elle est donc conservée lors de la fermeture et de la réouverture du logiciel.
* L&#x27;affichage du curseur indique la valeur du bloc, et non celle du pixel source, dès que la taille du bloc est supérieure à 1 — la valeur affichée correspond à la moyenne du bloc situé sous votre curseur.

{% hint style="info" %}
**Pourquoi « taille de bloc » et non pas centimètres par pixel ?** Une valeur en cm/px nécessite une hauteur au-dessus du sol. Les données EXIF d’une image unique indiquent l’altitude GPS au-dessus du niveau moyen de la mer, et non au-dessus du terrain vers lequel l&#x27;appareil était orienté ; par conséquent, Chloros n&#x27;affichera pas de distance au sol qu&#x27;il ne peut pas calculer avec précision. La taille de bloc en pixels source constitue la même solution de repli que celle utilisée par les outils de traitement des nuages MAPIR lorsque la distance d&#x27;échantillonnage au sol est inconnue.
{% endhint %}

***

## Types d’images que vous pouvez visualiser

Le menu déroulant des calques en haut à droite de la visionneuse répertorie toutes les versions de l’image actuelle. Les entrées affichées dépendent de la caméra et de ce qui a été traité — voir [Calques d’image](image-layers.md) pour la liste complète et le fonctionnement du menu déroulant.

### Survey3

* **JPG** — le fichier d&#x27;aperçu propre à la caméra
* **RAW (Original)** — le fichier source `.RAW`, débayérisé pour l’affichage, sans corrections
* **RAW (Cible)** — une image identifiée comme contenant une cible d’étalonnage
* **RAW (réflectance)** — le produit de réflectance calibré (65535 = ρ 1,0)
* **Correction de vignettage**/**Réponse du capteur** — le produit de secours non calibré
* **Balance des blancs** — le produit avec balance des blancs
* **RAW (indice `<INDEX>`)**et**LUT `<INDEX>`** — images d’indice calculées

### LATTICE

Les captures LATTICE utilisent le même menu déroulant, avec les noms des niveaux du pipeline :

| Calque                 | Contenu                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | L&#x27;image brute source telle qu&#x27;elle a été capturée                                     |
| **RAW (Débayérisée)**   | L&#x27;image linéaire débayérisée                                           |
| **RAW (Aperçu)**     | L&#x27;aperçu à l&#x27;écran — étirement en fausses couleurs pour les caméras multispectrales |
| **Balance des blancs**    | L&#x27;aperçu à l&#x27;écran pour les caméras principales RGB (balance des blancs + gamma)   |
| **RAW (radiance)**    | Radiance spectrale en Float32, en W/m²/sr/nm                              |
| **RAW (réflectance)** | Réflectance en uint16, 32768 = ρ 1,0                                    |

La radiance et la réflectance sont propres au mode multispectral : une caméra maître RGB ne dispose pas de radiométrie par bande, ces couches ne sont donc pas générées pour celle-ci.

***

## Application d’indices et de tables de correspondance (LUT)

Appliquez des indices multispectraux et des tables de correspondance (LUT) de couleurs à partir de la barre latérale :

1. Ouvrez la barre latérale **Visionneuse d’images**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Cochez **Index**

3. Choisissez le filtre de votre caméra et une formule d’indice, puis faites glisser les cercles de canal vers les emplacements de la formule
4. Ajoutez une LUT et sélectionnez un dégradé, des seuils et un mode de rognage
5. Consultez les valeurs au niveau du curseur, puis enregistrez le résultat avec **Exporter/Enregistrer l’image(s)**Consultez [Index/LUT Sandbox](index-lut-sandbox.md) pour le guide complet.***

## Dépannage

### L&#x27;image ne s&#x27;ouvre pas

**Causes possibles** : le fichier a été déplacé ou supprimé après l’importation ; le produit n’a jamais été enregistré ; mémoire insuffisante pour une image très volumineuse.**Que faire** :

1. Vérifiez que le fichier du calque existe toujours dans l’arborescence de sortie du projet
2. Ouvrez le fichier dans une visionneuse externe pour vous assurer qu’il est intact
3. Fermez les autres applications pour libérer de la mémoire

### L’image est noire, blanche ou présente des couleurs aberrantes

**Causes possibles** : l’étirement de l’affichage ne dispose d’aucune donnée à traiter (une image presque constante) ; un calque de type float32 contenant des valeurs inhabituelles ; un index n’ayant produit aucune donnée valide.**Que faire** :

1. Lisez les valeurs du curseur — si chaque canal est à zéro ou proche de zéro, le problème provient des données, et non de l’affichage
2. Vérifiez l’histogramme : un pic isolé à l’une des extrémités indique que l’image est écrêtée ou vide
3. Vérifiez le journal de traitement correspondant à la session qui a généré la couche

### Les valeurs semblent erronées

**Causes possibles** : vous êtes sur une couche différente de celle que vous pensez ; vous comparez un pourcentage à une valeur DN brute ; vous comparez un fichier LATTICE à un fichier Survey3 en utilisant le même diviseur.**Que faire** :

1. Vérifiez la couche sélectionnée dans le menu déroulant — les unités du panneau dépendent de la couche
2. Pour la réflectance, utilisez la colonne **%** plutôt que de diviser vous-même la valeur DN ; si vous devez diviser, utilisez la valeur `Chloros:PixelScale` de ce fichier (32768 pour LATTICE ; s’il est absent, cela signifie 65535 pour Survey3)
3. Rétablissez la taille de bloc GSD à 1 — au-delà de 1, vous lisez une moyenne de bloc, et non un pixel
4. Vérifiez que l’étalonnage de la réflectance a bien été effectué pour cette image ; un produit de secours non étalonné (réponse du capteur / correction de la vignette) ne correspond pas à la réflectance

***

## Étapes suivantes

* [**Calques d’image**](image-layers.md) — le nom de chaque calque, le cas échéant, et la signification de ses valeurs
* [**Bac à sable Index/LUT**](index-lut-sandbox.md) — créer, ajuster et exporter des visualisations d’index
* [**Marqueurs cartographiques**](map-markers.md) — le même ensemble d’images sur une carte
* [**Formules d’indices multispectraux**](../project-settings/multispectral-index-formulas.md) — la référence des indices

Pour le flux de traitement, voir [Traitement des images (interface graphique)](../processing-images-gui/adding-files-to-a-project.md).
