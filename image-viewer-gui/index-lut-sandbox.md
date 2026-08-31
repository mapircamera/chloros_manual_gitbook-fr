# Espace de test Index/LUT

L&#x27;espace de test Index/LUT est l&#x27;espace de travail interactif situé dans la barre latérale de la visionneuse d&#x27;images Chloros. Vous sélectionnez une formule, vous y associez les canaux de votre caméra, vous lui appliquez un dégradé de couleurs et vous ajustez la plage de valeurs — et l’image se met à jour en temps réel au fur et à mesure que vous effectuez ces modifications. Depuis la version 1.2.0, vous pouvez également **enregistrer ce que vous avez créé**, pour une seule image ou pour l’ensemble du projet, sans avoir à relancer le traitement.

## À quoi sert la Sandbox ?

| Sandbox Index/LUT (interactive)        | Traitement du projet (par lots)       |
| -------------------------------------- | -------------------------------- |
| Une image à la fois, retour instantané  | L’ensemble des données en un seul passage     |
| Expérimental et itératif             | Paramètres préconfigurés          |
| Rendu en temps réel ; enregistrement uniquement sur demande  | Écriture systématique des fichiers de sortie      |
| Idéal pour trouver les bons paramètres | Optimal une fois les paramètres finalisés |

{% hint style="success" %}
**Le flux de travail habituel** : affinez les réglages dans le Sandbox jusqu’à ce que la visualisation corresponde à ce que vous souhaitez, puis exportez directement depuis le Sandbox, ou copiez les mêmes paramètres d’index et de LUT dans [Paramètres du projet](../project-settings/project-settings.md) afin que le prochain cycle de traitement les intègre à chaque image.
{% endhint %}

***

## Ouverture du Sandbox

1. Cliquez sur une image dans la grille — elle s’ouvre en plein écran dans l’onglet **Visionneuse d’images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Cliquez sur l’icône **Visionneuse d’images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pour faire apparaître la barre latérale gauche si elle n’est pas déjà ouverte
3. Choisissez un calque multibande dans le menu déroulant des calques en haut à droite — **RAW (Réflectance)** est le choix habituel, car les valeurs d’indice calculées à partir de la réflectance calibrée sont comparables d’une image à l’autre

La barre latérale affiche, de haut en bas :

* le nom de l’image et le modèle de la caméra
* le bouton **Exporter/Enregistrer les images**— qui apparaît dès que l’option**Index**ou**LUT** est cochée
* les cases à cocher **Index**et**LUT**
* le panneau de configuration de l’indice
* le panneau **Valeurs du curseur** avec la lecture, l’histogramme et le contrôle GSD

{% hint style="warning" %}
**Non disponible pour les caméras monochromes.** Sur une image LATTICE M3M monobande, les deux cases à cocher sont désactivées, avec l’info-bulle _« Non disponible pour les capteurs monochromes (M3M) »_ — un indice multibande n’est pas défini sur une seule bande. Pour calculer des indices à partir de caméras M3M, combinez-en au moins deux en une pile multibande alignée et utilisez le moteur d&#x27;indexation LATTICE.
{% endhint %}

***

## Application d’un indice

1. Cochez la case **Indice** en haut de la barre latérale
2. Choisissez le filtre de votre caméra dans le menu déroulant de gauche (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Choisissez une formule d’index dans le menu déroulant de droite — 27 formules intégrées, auxquelles s’ajoutent les formules personnalisées que vous avez enregistrées
4. La formule s’affiche sous forme d’expression mathématique ci-dessous, avec un cercle vide à chaque emplacement de bande. **Faites glisser un cercle de canal coloré sur un emplacement** pour l’associer
5. Une fois que tous les emplacements utilisés par la formule sont associés, l’image se met à jour et affiche les valeurs d’index
6. Passez le curseur sur l’image pour lire les valeurs ; le panneau **Valeurs du curseur** ajoute une ligne d’index avec la valeur située sous le curseur

Double-cliquez sur un emplacement lié pour le vider. Une formule incomplète correspond à un état normal pendant le glissement, et non à une erreur — l’image ne se met simplement pas à jour tant que la formule n’est pas complète.

Les cercles de canal sont codés par couleur : rouge = Red, vert = Green, bleu = Blue, orange = Orange, cyan = Cyan, violet = NIR, magenta = RE. Les mêmes couleurs sont utilisées pour les points de canal et les courbes d’histogramme dans le panneau « Valeurs du curseur ».

### Exemple NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Pour consulter la référence complète des formules — les trois listes de préréglages et les noms correspondants à chaque contexte —, voir [Formules d’indices multispectraux](../project-settings/multispectral-index-formulas.md).

### Avec l’option « Index » cochée mais sans LUT

L’image est affichée en **niveaux de gris**, étirée entre les deux valeurs de seuil. Ce rendu est intentionnel : l’image d’indice est constituée de données scalaires, et les niveaux de gris en constituent le rendu le plus fidèle. Ajoutez une table de correspondance (LUT) lorsque vous souhaitez obtenir des couleurs.***

## Utilisation des tables de correspondance (LUT)

Une **table de correspondance** associe des valeurs d’index à des couleurs : en entrée NDVI 0,65, en sortie un vert particulier. Elle ne modifie pas les données, mais change la façon dont vous les interprétez.

### Ajouter une LUT

1. Cliquez sur le bouton **« + Ajouter une LUT »**<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">, situé sous la formule
2. Choisissez un dégradé de couleurs
3. Définissez les valeurs minimale et maximale de découpage
4. Choisissez un mode de découpage
5. Cochez la case **LUT** dans la barre latérale pour l’appliquer

La case à cocher LUT reste désactivée tant qu’aucune table de conversion n’a été configurée sur l’index.

### Choisir un dégradé de couleurs

Passez la souris sur la **barre de dégradé**pour ouvrir la liste des préréglages — Chloros propose**sept** préréglages de dégradés :

| # | Dégradé                            | Forme                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Jaune → Green (**par défaut**)  | Divergent — correspond à l’intuition habituelle concernant la végétation : vert = en bonne santé |
| 2 | Violet → Jaune → Green             | Divergent, avec une extrémité basse distincte                                  |
| 3 | Marron → Blanc → Blue                | Divergent autour d’un point médian clair                                   |
| 4 | Noir → Violet → Rose → Jaune pâle | Séquentiel, du foncé au clair                                           |
| 5 | Red → Jaune → Blue                 | Divergent autour d’un point médian clair                                   |
| 6 | Violet → Blue → Green → Jaune      | Séquentiel, du foncé au clair                                           |
| 7 | Orange → Blanc → Violet             | Divergence autour d&#x27;un point médian clair                                   |

Un dégradé **divergent**place une couleur neutre au milieu de votre fenêtre, ce qui est particulièrement adapté lorsque le point médian a une signification particulière (un seuil, une date de référence). Un dégradé**séquentiel** va de foncé à clair de manière monotone, ce qui convient bien à une quantité qui ne comporte que les notions de « plus » et « moins ».

Chaque préréglage comporte sept arrêts de couleur. Cliquez sur un préréglage pour que l’image s’actualise immédiatement (lorsque la case LUT est cochée).

### Modification des arrêts de couleur

Sous la barre de dégradé se trouve une rangée d’échantillons de couleur, à raison d’un par arrêt :

* **Modifier une couleur** : cliquez sur un échantillon pour ouvrir le sélecteur de couleur (roue chromatique, curseurs RGB/HSV, ou un code hexadécimal tel que `#FF0000`)
* **Ajouter un arrêt**: cliquez sur le bouton**+** à la fin de la rangée — un arrêt blanc est ajouté
* **Supprimer un arrêt**:**double-cliquez** sur l’échantillon
* **Conserver un dégradé modifié** : cliquez sur l’icône d’enregistrement à côté de la barre de dégradé pour ajouter votre dégradé modifié à la liste des préréglages afin de pouvoir le sélectionner à nouveau

Le dégradé que vous avez configuré sur un index est enregistré avec cet index dans les paramètres du projet ; il est donc conservé lorsque vous fermez puis rouvrez le projet.

**Un nombre réduit d’étapes**produit des zones distinctes qui s’interprètent comme une classification ;**un nombre plus élevé d’étapes** produit des transitions fluides, quasi photographiques. Trois à cinq étapes conviennent aux diapositives de présentation et aux cartes de classification ; six à dix conviennent à l’analyse générale ; quinze ou plus conviennent à l’inspection détaillée et aux figures de publication.

### Définition de la plage de valeurs

La commande de seuil est un **curseur à deux poignées**allant de −1 à +1, doté d’une zone de texte modifiable à chaque extrémité pour saisir des valeurs exactes, ainsi que d’un bouton**AUTO**.

* Faites glisser l’une des poignées, ou saisissez un nombre dans la zone correspondante et appuyez sur Entrée
* **AUTO**définit la plage entre le**2e et le 98e centile** des valeurs d’indice valides de l’image — un bon point de départ qui ignore les valeurs aberrantes. Chloros arrondit le résultat de manière adaptative : à 4 décimales pour une plage très étroite, à 3 pour une plage étroite, et à 2 dans les autres cas
* Tout réglage manuel prévaut sur AUTO jusqu’à ce que vous appuyiez à nouveau sur AUTO

Exemple de fenêtres NDVI :

| Objectif                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Tout afficher                         | −1,0 | 1,0 |
| Végétation uniquement, exclure le sol et l&#x27;eau | 0,2  | 0,9 |
| Végétation saine uniquement                 | 0,5  | 0,9 |
| Mettre l’accent sur le stress                | 0,2  | 0,5 |

Le fait de réduire la fenêtre augmente le contraste à l’intérieur de votre zone d’intérêt et fait sortir tout le reste de la plage — c’est alors que le **mode de découpage** détermine ce qu’il advient de ces éléments.***

## Modes de rognage

Lorsque la valeur d’index d’un pixel se situe en dehors de la plage min/max, le mode de rognage détermine comment il est affiché.

| Libellé du menu déroulant                  | Valeur enregistrée      | Les pixels hors plage sont affichés comme                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum et maximum** (par défaut) | `clip`            | La couleur d’extrémité la plus proche du dégradé — les valeurs inférieures au minimum prennent la première couleur, celles supérieures au maximum prennent la dernière |
| **Arrière-plan transparent**      | `transparent`     | Entièrement transparent (alpha réel)                                                                                                  |
| **Arrière-plan indexé**| `indexColor`      | Niveaux de gris, étendu sur**toute** la plage d’index de l’image, de sorte que les structures hors plage restent visibles en gris                |
| **Arrière-plan d&#x27;origine**         | `backgroundColor` | L&#x27;image sous-jacente elle-même, de sorte que la superposition de couleurs se superpose à la scène réelle                                                |

| Mode                       | Idéal pour                               | Aspect                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum et maximum**      | Affichage complet des données, analyse scientifique | Chaque pixel est coloré                      |
| **Arrière-plan transparent** | Superpositions SIG, isolation d’une plage de valeurs   | Couleur à l’intérieur de la fenêtre, rien à l’extérieur |
| **Arrière-plan indexé**       | Mise en évidence tout en conservant le contexte des données    | Couleur à l&#x27;intérieur, gris à l&#x27;extérieur               |
| **Arrière-plan d&#x27;origine**    | Rapports et présentations              | Couleur à l&#x27;intérieur, photographie à l&#x27;extérieur         |

{% hint style="info" %}
**Les pixels sans données sont toujours transparents, quel que soit le mode.** Un pixel dont l’indice n’est pas fini (division par 0) ou est exactement égal à −1,0 ou +1,0 (sentinelles de saturation, résultant du fait qu’une bande affiche zéro tandis que l’autre ne le fait pas) est traité comme une absence de données plutôt que comme une valeur extrême. Cela permet d’exclure les hautes lumières brûlées et les ombres noires de votre échelle de couleurs, au lieu de les représenter comme les valeurs les plus extrêmes de l’image. La même règle définit quels pixels alimentent les seuils AUTO et l’histogramme d’index, de sorte que les trois s’accordent.
{% endhint %}

La transparence est préservée lorsque l’exportation est enregistrée au format PNG. Elle ne peut pas être représentée au format JPG.

***

## Lecture des valeurs pendant le réglage

Le panneau **Valeurs du curseur** situé sous le panneau de configuration sert d’instrument de mesure pour le Sandbox :

* Déplacez le curseur sur l’image et lisez les valeurs source par canal, ainsi que la valeur d’indice sur sa propre ligne
* Activez le bouton **INDEX** au-dessus de l’histogramme pour visualiser la distribution des valeurs d’indice dans l’image, vos deux seuils de découpage étant représentés par des lignes pointillées orange et la valeur du curseur par une ligne blanche — c’est le moyen le plus rapide de sélectionner une fenêtre contenant réellement vos données
* Activez **CURSOR** pour afficher des lignes de repère aux valeurs situées sous le pointeur
* Zoomez au-delà de 60× (moins si une taille de bloc GSD est définie) pour mettre en évidence les pixels affichés individuellement avec une valeur flottante

Une procédure pratique :

1. Notez les valeurs au-dessus de la végétation saine, de la végétation stressée, du sol nu et de l’eau
2. Observez où se situent ces groupes sur l’histogramme d’indice
3. Définissez les valeurs min/max pour encadrer le groupe qui vous intéresse
4. Choisissez un mode de recadrage — _Original Background_ conserve la scène visible autour de celui-ci

***

## Exportation depuis le Sandbox

Tout ce qui précède n’est qu’un aperçu en temps réel tant que vous ne l’avez pas enregistré. Le bouton **Exporter/Enregistrer les images** en haut de la barre latérale ouvre un volet qui s’affiche par-dessus la barre latérale (plutôt que de recouvrir l’image, ce qui vous permet de continuer à voir ce sur quoi vous vous basez pour prendre vos décisions).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Options

| Option                          | Effet                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Appliquer à l’image actuelle**      | Enregistre exactement l’image affichée, avec ces paramètres                                                                                                |
| **Appliquer à toutes les images du projet** | Réexécute la configuration identique sur chaque image du projet. Les images ne comportant pas les bandes requises par cet index sont ignorées et ne sont pas considérées comme des échecs |
| **Barre de gradient d’index/LUT**      | Enregistre également une image de légende distincte par exportation, avec la plage de valeurs indiquée                                                                     |
| **Histogramme d’index**             | Enregistre également une image d’histogramme distincte par exportation, indiquant les valeurs minimales et maximales des données ainsi que les seuils de coupure                                               |

Si la **taille de bloc GSD** de l&#x27;onglet de l&#x27;image est supérieure à 1, le volet vous en informe avant que vous ne validiez : l&#x27;exportation enregistre ce que vous voyez, moyennage par bloc inclus. Remettez d&#x27;abord le paramètre GSD à 1 si vous souhaitez la pleine résolution.

### Emplacement des fichiers

Chaque clic sur **Exporter**crée un**nouveau dossier qui ne sera jamais réutilisé** :

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Exemples : `Sandbox_Exports/NDVI_LUT_001/`, puis `Sandbox_Exports/NDVI_LUT_002/` pour la session suivante. La numérotation est générée en analysant le contenu existant sur le disque ; elle est donc conservée même après un redémarrage ou la suppression manuelle de dossiers. Rien n’est jamais écrasé — l’intérêt même du bac à sable est de comparer une tentative à la précédente.

À l’intérieur du dossier, pour chaque image :

| Fichier                                                   | Contenu                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | L’image rendue, pixel par pixel telle qu’elle s’affichait dans la visionneuse |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Le fichier d’accompagnement de la barre de dégradé, si demandé                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Le fichier d’accompagnement de l’histogramme d’index, si demandé                  |

Les deux fichiers d&#x27;accompagnement sont toujours enregistrés en **pleine résolution**, même lorsque l&#x27;image principale est moyennée par blocs : la taille d&#x27;un bloc correspond à la résolution d&#x27;affichage, et les deux fichiers d&#x27;accompagnement contiennent les valeurs d&#x27;index réelles par pixel. Elles affichent également davantage d’informations que les versions à l’écran : elles indiquent à la fois la fenêtre d’étirement _et_ les valeurs minimales et maximales réelles des données, ce qui permet de consulter une légende enregistrée plusieurs mois plus tard sans avoir à ouvrir le projet.

### Progression et résultats

L’exportation de l’ensemble d’un projet ne prend que quelques minutes ; l’exécution affiche donc ses progrès via un canal en temps réel plutôt que de bloquer le système :

* Une barre de progression affiche `current / total` et le fichier en cours d’écriture
* Une fois l’opération terminée, le volet indique le nombre d’images exportées, le nombre d’images ignorées et le chemin d’accès au dossier de sortie
* Les images ignorées sont répertoriées avec la raison correspondante (jusqu’à cinq raisons affichées, puis une ligne « +N autres »). La raison la plus courante est un calque ne disposant pas des canaux requis par cet index
* Si **aucune** image du projet ne peut utiliser l’index, l’exécution signale un échec plutôt que de vous laisser un dossier vide

Une seule exportation en mode sandbox peut s’exécuter à la fois. Le lancement d’une deuxième exportation alors qu’une autre est en cours est refusé par un message clair, afin d’éviter que deux exécutions ne se disputent le même fichier de projet.

### La grille reprend l’exécution

Chaque exécution terminée apparaît sous la forme d’un bouton distinct dans la [grille d’images](image-grid.md) de la barre d’outils, libellé `<IndexName> <Index|LUT> <NNN>`. C’est ainsi que vous comparez les exécutions : effectuez deux exportations avec des gradients ou des seuils différents, puis basculez entre les deux boutons de la grille.

***

## Formules d’indice personnalisées (Chloros+)

{% hint style="info" %}
**Où les créer**: dans la barre latérale du Sandbox, ou dans les**Paramètres du projet** avant le traitement. Les deux options écrivent dans la même liste au niveau du projet.
{% endhint %}

1. Ouvrez la calculatrice de formules personnalisées à partir du menu déroulant des formules d’index (nécessite de se connecter avec un abonnement Chloros+ éligible)
2. Saisissez la formule en utilisant les **symboles de plage de bande** `x`, `y`, `z`, `a`, `b`, `c` — et non les noms des bandes
3. Opérateurs disponibles : `+`, `-`, `*`, `/`, `^` et `()` pour le regroupement
4. Fonctions disponibles : `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Nommez-la et enregistrez-la — elle apparaît au bas du menu déroulant des formules et vous pouvez lier ses emplacements en faisant glisser les cercles de canaux, exactement comme pour un préréglage intégré

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Les formules personnalisées sont disponibles uniquement via l&#x27;interface graphique.** L’option CLI/SDK `--indices` étend les 22 noms de préréglages intégrés et ignore silencieusement tout le reste, y compris vos formules personnalisées. Pour traiter une formule personnalisée par lots, configurez-la dans les paramètres du projet et lancez le traitement, ou utilisez la fonction d&#x27;exportation « Appliquer à toutes les images du projet » de la Sandbox.
{% endhint %}

***

## Dépannage

### « Ce calque ne dispose pas des canaux requis par cet index »

La formule lit une position de canal que le calque actuel ne possède pas — par exemple, un index à trois emplacements sur un fichier à un ou deux canaux. Passez à un calque multibande (réflectance ou débayérisé), ou choisissez un index adapté au filtre de votre appareil photo.

### « Impossible d’accéder au backend de traitement d’image »

Le backend ne répond pas. Vérifiez l’onglet « Log » ; si le backend est en cours de redémarrage, Sandbox se rétablit automatiquement dès qu’il est de nouveau opérationnel.

### L’image n’a pas changé lorsque j’ai fait glisser un cercle

La formule n’est pas encore complète. Une formule incomplète est traitée comme un état normal en cours de glissement — rien n’est rendu et aucune erreur n’est signalée. Remplissez tous les champs utilisés par la formule.

### L’image entière est d’une seule couleur

Votre fenêtre de clip se trouve probablement bien en dehors des données. Appuyez sur **AUTO**pour l’aligner sur les 2e et 98e centiles, ou activez l’histogramme**INDEX** pour voir où se situent réellement les données.

### Les couleurs exportées ne correspondent pas à ce que j’ai vu

Elles devraient — le chemin d’exportation est un reflet fidèle de l’aperçu en direct, y compris l’alpha en mode de détourage, et le calcul de la moyenne par bloc est appliqué _après_ la colorisation, exactement comme le fait la visionneuse. Si elles diffèrent, vérifiez que la taille de bloc GSD n’a pas changé entre la visualisation et l’exportation.

***

## Étapes suivantes

* [**Calques d’image**](image-layers.md) — sur quel calque appliquer un indice, et que signifient ses valeurs
* [**Ouverture d’une image en plein écran**](opening-an-image-full-screen.md) — lecture du curseur, histogramme et contrôle du GSD en détail
* [**Formules d’indices multispectraux**](../project-settings/multispectral-index-formulas.md) — tous les préréglages, pour toutes les surfaces
* [**Paramètres du projet**](../project-settings/project-settings.md) — intégration des paramètres que vous avez définis dans un cycle de traitement
