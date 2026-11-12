# Paramètres du projet

La barre latérale « Paramètres du projet » <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

de Chloros vous permet de configurer tous les aspects du traitement d’images, de la détection des cibles d’étalonnage, du calcul des indices multispectraux et des options d’exportation pour votre projet. Ces paramètres sont enregistrés avec votre projet et peuvent être sauvegardés sous forme de modèles afin d’être réutilisés dans plusieurs projets.

## Accéder aux paramètres du projet

Pour accéder aux paramètres du projet :

1. Ouvrez un projet dans Chloros


2. Cliquez sur l'onglet **Paramètres du projet** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

dans la barre latérale gauche
3. Le panneau des paramètres affichera toutes les options de configuration disponibles, classées par catégorie

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Les paramètres qui dépendent d’autres paramètres sont grisés.** Lorsqu’un paramètre parent rend un autre paramètre impossible (par exemple, décocher *Étalonnage de la réflectance / balance des blancs* rend l’option *Exporter la réflectance* impossible), le contrôle dépendant est désactivé et son info-bulle indique le paramètre qui doit être modifié.
{% endhint %}

***

## Affichage

### Résolution des vignettes d’image

* **Type** : liste déroulante
* **Options** : `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Par défaut** : Par défaut (512 px)
* **Description** : Résolution (côté le plus long, en pixels) à laquelle les vignettes de la grille d'images sont affichées. Des valeurs plus élevées offrent un rendu plus net lors du zoom, mais ralentissent le chargement et consomment davantage de mémoire. La résolution maximale correspond à la taille d'origine de l'image.
* **Remarque** : Affichage uniquement — cela n’affecte jamais le traitement ni les fichiers exportés.

***

## Détection des cibles

Ces paramètres contrôlent la manière dont Chloros détecte et traite les cibles d’étalonnage dans vos images. Ils ne sont actifs que lorsque l’option **Étalonnage de la réflectance / balance des blancs** est activée (ils sont grisés dans le cas contraire, car la détection des cibles est alors entièrement ignorée).

### Surface minimale de l’échantillon d’étalonnage (px)

* **Type** : Nombre
* **Plage** : de 0 à 10 000 pixels
* **Valeur par défaut** : 25 pixels
* **Description** : Définit la surface minimale (en pixels) requise pour qu’une région détectée soit considérée comme un échantillon de cible d’étalonnage valide. Des valeurs plus faibles permettront de détecter des cibles plus petites, mais peuvent augmenter le nombre de faux positifs. Des valeurs plus élevées nécessitent des régions cibles plus grandes et plus nettes pour la détection.
* **Quand ajuster** :
  * Augmentez cette valeur si vous obtenez de fausses détections sur de petits artefacts d’image
  * Diminuez cette valeur si vos cibles d’étalonnage apparaissent petites sur vos images et ne sont pas détectées

### Regroupement minimal des cibles (0-100)

* **Type** : Nombre
* **Plage** : de 0 à 100
* **Valeur par défaut** : 60
* **Description** : Contrôle le seuil de regroupement des zones de couleur similaire lors de la détection des cibles d’étalonnage. Des valeurs plus élevées exigent que des couleurs plus similaires soient regroupées, ce qui se traduit par une détection des cibles plus prudente. Des valeurs plus faibles autorisent davantage de variations de couleur au sein d’un groupe de cibles.
* **Quand ajuster** :
  * Augmentez-la si les cibles d’étalonnage sont fractionnées en plusieurs détections
  * Diminuez-la si les cibles d’étalonnage présentant des variations de couleur ne sont pas entièrement détectées

***

## Traitement

Ces paramètres contrôlent la manière dont Chloros traite et étalonne vos images.

### Correction du vignetage

* **Type** : Case à cocher
* **Par défaut** : Activé (coché)
* **Description** : Applique une correction du vignetage pour compenser l’assombrissement de l’objectif sur les bords des images. Le vignetage est un phénomène optique courant où les coins et les bords d’une image apparaissent plus sombres que le centre en raison des caractéristiques de l’objectif.
* **Effet secondaire** : cette option permet également de sélectionner le *produit de secours non calibré* qu’un traitement génère (voir ci-dessous).

### Calibrage de la réflectance / balance des blancs

* **Type** : case à cocher
* **Par défaut** : activé (coché)
* **Description** : Active l’étalonnage de la réflectance — à partir des cibles d’étalonnage détectées dans le cadre et/ou des données de lumière descendante du capteur d’éclairage DAQ, en fonction de la caméra et des données disponibles. Cela normalise les valeurs de réflectance dans l’ensemble de vos données et garantit des mesures cohérentes quelles que soient les conditions d’éclairage.
* **Lorsqu’elle est désactivée**: la détection des cibles est entièrement ignorée, et**aucune caméra ne peut générer de produit de réflectance** — qu’il s’agisse de cibles Survey3 ou du DAQ LATTICE. Les paramètres associés (*Exporter la réflectance*, *Intervalle minimum de recalibrage* et les seuils de détection des cibles) sont grisés.

### Produits de secours non calibrés : Exporter la réponse du capteur / Exporter la vignette corrigée

* **Type** : deux cases à cocher
* **Valeurs par défaut** : les deux sont activées (cochées)
* **Description** : lorsqu’une image ne peut pas être calibrée en réflectance (aucune cible de calibrage n’a été trouvée ou le calibrage de réflectance est désactivé), elle est enregistrée en tant que *produit de secours non calibré*. **Il n’existe qu’un seul des deux produits de secours par série, pour chaque modèle de caméra**, choisi par le commutateur *Correction de vignettage* :
  * Correction de vignettage **activée**→ `Vignette_Corrected_Images/` (régie par**Exporter avec correction de vignettage**)
  * Correction de vignettage **désactivée**→ `Sensor_Response_Images/` (régie par**Exporter la réponse du capteur**)
* Le produit de secours qui n’est pas utilisé est grisé. Décocher celui qui est utilisé empêche complètement l’écriture de ce fichier.

### Produits d’exportation LATTICE

Pour les projets contenant des captures LATTICE, chaque image LATTICE importée est répartie vers tous les produits activés **et applicables**en un seul passage de traitement. Quatre cases à cocher contrôlent cette répartition (toutes**activées** par défaut) :

| Paramètre | Dossier de sortie | Ce qu’il exporte |
| --- | --- | --- |
| **Exporter après débayérisation** | `Debayered_Images/` | L’image linéaire après débayérisation : le décalage de noir du pack d’étalonnage supprimé, puis un dématriçage tenant compte des contours, sans correction de linéarité, champ plat ni démixage — le même fichier qu’une capture enregistre à son niveau dématriçé. Conservée dans l’orientation native du capteur (enregistrée dans XMP). S’applique aux caméras RGB et multispectrales. |
| **Exporter l'aperçu** | `Preview_Images/` | L'aperçu à l'écran. RGB = balance des blancs (illuminant DAQ si disponible, sinon « gray-world ») + gamma ; multispectral = étirement en fausses couleurs. |
| **Exportation de la radiance** | `Radiance_Images/` | Radiance spectrale en Float32 (W/m²/sr/nm). Multispectral (M3C/M3M) uniquement — ne s'applique pas aux masters RGB. Toujours enregistré au format TIFF 32 bits, quel que soit le paramètre *Format d'image calibré*. |
| **Exportation de la réflectance**| `Reflectance_Calibrated_Images/` | Réflectance Uint16, mise à l'échelle de sorte que**32768 = réflectance 1,0** (marquée comme XMP `Chloros:PixelScale`). Multispectral uniquement, enregistrée lorsqu’un enregistrement descendant `.daq` correspondant (ou une cible dans le cadre ayant passé le contrôle qualité) recouvre le cadre. |

* Les caméras principales RGB émettent des données débayérisées + un aperçu ; la radiance et la réflectance sont ignorées pour celles-ci, car non applicables.
* La profondeur de bits des données débayérisées/de l’aperçu suit le paramètre *Format d’image calibré* ; la radiance est toujours en float32.
* Le traitement *Survey3

* n’est pas affecté par ces quatre commutateurs.
* Les produits de radiance et de réflectance des caméras multispectrales M3C peuvent également écrire un **masque de limitation** par pixel dans un dossier `clamp_masks/` situé à côté des dossiers de produits, en enregistrant les endroits où le démixage des bandes a écrêté une valeur négative à zéro. Voir [Formats d’images de sortie](../output-image-formats.md#clamp-masks).

Ces quatre commutateurs existent également en mode sans interface graphique sous le nom `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Ils ont remplacé l’ancien indicateur CLI `--radiometric-output`, qui n’existe plus.

{% hint style="warning" %}
**La désactivation de tous les produits applicables entraîne l’échec de l’exécution.** À partir de la version 1.2.0, une exécution de traitement pour laquelle des produits ont été demandés mais qui n’a généré aucun produit d’image signale un échec et l’CLI se termine avec un code de sortie différent de zéro, au lieu de signaler une réussite silencieuse. Le journal indique le nom du produit qu’il n’a pas pu générer et la raison de cet échec. Une exécution délibérément limitée aux métadonnées (rien n’a été demandé) est toujours considérée comme réussie.
{% endhint %}

### Source de réflectance (paramètre du projet, défini via l’CLI)

Le projet stocke également la **référence de réflectance** utilisée par le produit de réflectance LATTICE. Il n’existe pas de commande dédiée dans le panneau de paramètres ; la valeur est stockée dans la configuration du projet sous la référence `Processing → "Target reflectance source"` et est définie à l’aide de `chloros-cli process --reflectance-source {auto,target,daq}` (la variable `configure()` du SDK ne dispose pas de mot-clé pour cela) :

* **`auto`** (par défaut) : une cible d’étalonnage « in-frame » ayant passé le contrôle qualité (QA) devient la référence absolue ; en l’absence de cible ou en cas d’échec du QA, le système se rabat sur la division de la lumière descendante du DAQ (ρ = πL/E). Une cible ayant réussi le contrôle qualité avec un indicateur d’erreur (un panneau tombé ou une bande hors service) n’est pas automatiquement privilégiée : les chaînes « cible », « cible + DAQ » et « DAQ uniquement » sont testées dans l’ordre de leur score mesuré sur des panneaux non utilisés.
* **`target`** : réflectance strictement déterminée par la cible — aucune substitution par le DAQ.
* **`daq`** : réflectance déterminée par le DAQ ; les cibles présentes dans le cadre ne sont pas utilisées comme référence.

La valeur enregistrée est comparée sans distinction de majuscules et de minuscules, et quelques variantes orthographiques sont acceptées comme alias : `target`, `target_image`, `empirical` et `empirical_line` désignent tous la **cible**; `daq`, `dls`, `light_sensor` et `sensor` signifient tous**daq**. Toute autre valeur — y compris une clé manquante — est interprétée comme**auto**.

Les analyses de cibles **mesurées** par unité sont recherchées à l’aide du numéro de série/QR de l’unité cible, sous la forme `<serial>.csv`, à trois endroits : le répertoire indiqué par `--target-reflectance-dir` (stocké sous le nom `Processing → "Target reflectance dir"`), le dossier `target_reflectance/` propre au projet, et le chemin indiqué dans la variable d’environnement `CHLOROS_TARGET_REFLECTANCE_DIR`. Lorsqu’aucun scan mesuré n’existe pour cette unité, la courbe nominale publiée pour le modèle cible est utilisée à la place.

### Méthode de débayérisation

* **Type** : sélection dans un menu déroulant
* **Options** :
  * Standard (rapide, qualité moyenne)
  * Texture Aware (lent, qualité optimale) \[Chloros+]
* **Par défaut** : Standard (rapide, qualité moyenne)
* **Description** : Permet de sélectionner l’algorithme de dématriçage utilisé pour convertir les données brutes du capteur à matrice de Bayer en images en couleur. La méthode « Standard (rapide, qualité moyenne) » offre un équilibre optimal entre vitesse de traitement et qualité d’image. La méthode « Texture Aware (lente, qualité optimale) » \[Chloros+] utilise un dématriçage de haute qualité tenant compte des contours, associé à un modèle de débruitage basé sur l’IA/ML qui élimine la quasi-totalité du bruit de dématriçage. Le modèle « Texture Aware » nécessite de la mémoire GPU (VRAM) pour fonctionner. Nous vous recommandons de l’utiliser lorsque vous disposez de plus de 4 Go de VRAM pour un traitement plus rapide.
* **Lorsque la ligne correspond à un menu déroulant**: le menu déroulant à deux options n’apparaît que lorsque**les deux** conditions sont remplies : vous êtes connecté avec un abonnement Chloros+ éligible, **et** le projet ne contient aucune capture LATTICE. Sinon, la ligne s’affiche sous forme de texte brut indiquant « `Standard (Fast, Medium Quality)` » sans aucune option à sélectionner.
* **Remarque concernant LATTICE** : Il n’existe pas de modèle « Texture Aware » entraîné pour LATTICE, et le pipeline impose le dématriçage standard pour les images LATTICE, quelle que soit la valeur enregistrée. Si vous ajoutez un dossier LATTICE à un projet pour lequel l’option « Texture Aware » était déjà sélectionnée, Chloros réinitialise le paramètre sur « Standard » plutôt que de conserver une valeur obsolète dans `project.json`.

### Intervalle minimal de recalibrage

* **Type** : Nombre
* **Plage** : 0 à 3 600 secondes
* **Valeur par défaut** : 0 seconde
* **Description** : Définit l’intervalle de temps minimum (en secondes) entre l’utilisation de cibles d’étalonnage. Lorsqu’il est défini sur 0, Chloros utilisera toutes les cibles d’étalonnage détectées. Lorsqu’il est défini sur une valeur supérieure, Chloros n’utilisera que les cibles d’étalonnage séparées d’au moins ce nombre de secondes, ce qui réduit le temps de traitement pour les ensembles de données comportant des captures fréquentes de cibles d’étalonnage.
* **Quand ajuster** :
  * Réglez sur 0 pour une précision d’étalonnage maximale lorsque les conditions d’éclairage varient
  * Augmentez cette valeur (par exemple, de 60 à 300 secondes) pour un traitement plus rapide lorsque l’éclairage est constant et que vous disposez d’images fréquentes de cibles d’étalonnage

### Décalage horaire du capteur de lumière

* **Type** : Nombre
* **Plage** : de -12 à +12 heures
* **Valeur par défaut** : 0 heure
* **Description**: Spécifie le décalage horaire (en heures par rapport à l’UTC) pour les horodatages des données du capteur de lumière, utilisé lors de la mise en correspondance des journaux du capteur de lumière avec les heures de capture des images. Les enregistrements `.daq` plus récents comportent leur propre provenance de fuseau horaire ; ce paramètre est donc principalement nécessaire pour les anciens journaux enregistrés en heure locale.

### Appliquer les corrections PPK

* **Type** : Case à cocher
* **Par défaut** : Désactivé (décochée)
* **Description** : Active l’utilisation des corrections cinématiques post-traitées (PPK) provenant des enregistreurs DAQ MAPIR équipés d’un GPS (GNSS). Lorsque cette option est activée, Chloros utilisera tous les fichiers journaux .daq contenant des données d’exposition dans le répertoire de votre projet et appliquera des corrections de géolocalisation précises à vos images.
* **Condition préalable** : un fichier journal .daq contenant des entrées de broches d’exposition doit être présent dans le répertoire de votre projet
* **Quand l’activer** : il est recommandé d’activer systématiquement la correction PPK si votre fichier journal .daq contient des entrées de retour d’exposition.

### Broche d’exposition 1

* **Type** : menu déroulant
* **Visibilité** : visible uniquement lorsque l’option « Appliquer les corrections PPK » est activée ET que des données d’exposition sont disponibles pour la broche 1
* **Options** :
  * Noms des modèles d’appareils photo détectés dans le projet
  * « Ne pas utiliser » : ignore cette broche d’exposition
* **Par défaut** : sélection automatique en fonction de la configuration du projet
* **Description** : attribue une caméra spécifique à la broche d’exposition 1 pour la synchronisation temporelle PPK. La broche d’exposition enregistre le moment exact où l’obturateur de la caméra est déclenché, ce qui est essentiel pour une géolocalisation PPK précise.
* **Comportement de sélection automatique** :
  * Une seule caméra + une seule broche : sélectionne automatiquement la caméra
  * Une seule caméra + deux broches : la broche 1 est automatiquement attribuée à la caméra
  * Plusieurs caméras : sélection manuelle requise

### Broche d'exposition 2

* **Type** : sélection dans une liste déroulante
* **Visibilité** : visible uniquement lorsque l'option « Appliquer les corrections PPK » est activée ET que des données d'exposition sont disponibles pour la broche 2
* **Options** :
  * Noms de modèles de caméras détectés dans le projet
  * « Ne pas utiliser » : ignore cette broche d’exposition
* **Par défaut** : sélection automatique en fonction de la configuration du projet
* **Description** : attribue une caméra spécifique à la broche d’exposition 2 pour la synchronisation temporelle PPK lors de l’utilisation d’une configuration à deux caméras.
* **Comportement de la sélection automatique** :
  * Une caméra + une broche : la broche 2 est automatiquement réglée sur « Ne pas utiliser »
  * Une caméra + deux broches : la broche 2 est automatiquement réglée sur « Ne pas utiliser »
  * Plusieurs caméras : sélection manuelle requise
* **Remarque** : une même caméra ne peut pas être attribuée simultanément à la broche 1 et à la broche 2.

***

## Capteur de lumière DAQ

Cette section apparaît dans les Paramètres du projet et répertorie tous les fichiers DAQ de rayonnement descendant du projet — enregistrements `.daq` et journaux DAQ-M `.csv`. Les enregistrements effectués dans l’onglet « Capteurs de lumière » sont automatiquement ajoutés au projet ouvert.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Chaque ligne affiche le fichier, le modèle de capteur et la correction du capuchon diffuseur effectivement appliquée à ce fichier. Au-dessus des lignes se trouve un contrôle unique valable pour l’ensemble du projet :

### Remplacement du capuchon (tous les fichiers)

* **Type** : menu déroulant
* **Options** : `Auto` ainsi que les profils de correction du capuchon valables pour les types de capteurs présents dans le projet
* **Par défaut** : Auto
* **Enregistré sous** : `Processing → "DAQ cap id"` (par défaut `auto`)
* **Description** : `Auto` utilise le cap enregistré dans chaque fichier (le cap Sunshine est supposé lorsque rien n’a été enregistré, mais les DAQ-U, DAQ-M et DAQ-E sont livrés sans cap et Chloros ne peut pas voir ce qui est installé, le traitement affiche donc un avertissement concernant un cap supposé : choisissez le cap qui était installé). Le choix d’un cap spécifique remplace **tous** les fichiers de rayonnement descendant du projet, à l’exception de ceux d’un DAQ-E-S, dont le diffuseur fait partie de son étalonnage ; un projet composé uniquement de DAQ-E-S n’affiche pas de liste déroulante : les enregistrements bruts sont corrigés avec celui-ci, et les enregistrements comportant déjà un capuchon sont ré-étalonnés (la correction enregistrée est annulée et celle sélectionnée est appliquée).
* **Important** : le capuchon sélectionné doit correspondre à celui qui était physiquement installé lors de l’enregistrement. Ni le capteur ni le logiciel ne peuvent détecter le capuchon physique — un identifiant de capuchon incorrect entraîne une correction erronée des spectres.

Il existe délibérément **un seul** paramètre valable pour l’ensemble du projet plutôt que des menus déroulants par fichier : ce réglage s’applique à toutes les sources descendantes du projet.

***

## Alignement des réseaux

Cette section n’apparaît **que** lorsqu’au moins une image du projet comporte la transformation d’alignement de module à module que les réseaux LATTICE apposent au moment de la capture (balises XMP `Chloros:Alignment*`). Elle indique le nombre d’images comportant des balises d’alignement, quelle caméra sert de référence (badge `REF`), ainsi qu’un tableau du nombre d’images par caméra.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Appliquer l’alignement des matrices

* **Type** : case à cocher
* **Par défaut** : activé (coché)
* **Enregistré sous** : `Processing → "Array alignment"`
* **Description** : Déforme chaque produit traité (débayérisé / aperçu / radiance / réflectance / indice) pour l'adapter à la géométrie de référence partagée du réseau à l'aide de la transformation enregistrée lors de la capture. Désactivé = exportation dans la géométrie native propre à chaque capteur.

### Recadrer sur le chevauchement commun

* **Type** : Case à cocher (active uniquement lorsque l’option *Appliquer l’alignement du tableau* est activée)
* **Par défaut** : Activée (cochée)
* **Enregistrée sous** : `Processing → "Array alignment crop"`
* **Description** : Recadre les exportations alignées à la zone commune à tous les modules de caméra, de sorte que chaque bande ait la même empreinte. La désactivation conserve la zone complète du capteur (remplissage noir à l'extérieur de la source).

### Rééchantillonnage

* **Type** : Menu déroulant (actif uniquement lorsque l'option *Appliquer l'alignement du réseau* est activée)
* **Options** : `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Par défaut** : Bilinéaire
* **Enregistré sous** : `Processing → "Array alignment interpolation"`
* **Description** : Interpolation utilisée par la transformation d’alignement. L’option « Proche » préserve les valeurs exactes de la source (pas de mélange entre pixels) pour une analyse radiométrique rigoureuse ; l’option « Bilinéaire » est la mieux adaptée à la cartographie et à l’utilisation visuelle.

Ces trois options existent également sans préfixe : `chloros-cli process --array-alignment`, `--array-alignment-crop` et `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indices

Ces paramètres vous permettent de configurer des indices multispectraux à des fins d’analyse et de visualisation.

### Ajouter un indice

* **Type** : Panneau de configuration des indices spéciaux
* **Description** : Ouvre un panneau interactif dans lequel vous pouvez sélectionner et configurer des indices de végétation multispectraux (NDVI, NDRE, EVI, etc.) à calculer lors du traitement de l’image. Vous pouvez ajouter plusieurs indices, chacun avec ses propres paramètres de visualisation.
* **Indices disponibles**: le menu déroulant de l’interface graphique comprend**27** formules d’indices multispectraux prédéfinies (voir [Formules d’indices multispectraux](multispectral-index-formulas.md) pour la liste complète, y compris les noms également acceptés par l’option CLI/SDK `--indices`).
* **Fonctionnalités** :
  * Sélectionnez parmi les formules d’indices prédéfinies
  * Faites glisser les canaux de filtrage de votre caméra vers les emplacements de bande de la formule
  * Configurez les dégradés de couleurs de visualisation (LUT - tables de correspondance)
  * Définissez des valeurs de seuil et des modes de rognage
  * Créez des formules d’indice personnalisées
* **Remarque** : les indices ne sont pas calculés pour les caméras mono LATTICE M3M à bande unique — les indices multibandes ne sont pas définis sur une seule bande. Les modèles Survey3 et LATTICE M3C ne sont pas concernés.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Chaque indice que vous ajoutez affiche sa formule sous forme mathématique, avec un cercle coloré par emplacement de bande : rouge = Red, vert = Green, bleu = Blue, orange = Orange, cyan = Cyan, violet = NIR, magenta = RE. Faites glisser un cercle depuis la ligne située au-dessus de la formule vers un emplacement pour l’associer ; double-cliquez sur un emplacement associé pour le désassocier. L’indice n’est calculé que si chaque emplacement utilisé par la formule dispose d’un canal.

### Formules personnalisées (fonctionnalité Chloros+)

* **Type** : Tableau de définitions de formules personnalisées
* **Disponibilité** : Nécessite de se connecter avec un abonnement Chloros+ éligible.
* **Description** : Permet de créer et d’enregistrer des formules d’indices multispectraux personnalisées à l’aide de calculs mathématiques sur les bandes. Les formules personnalisées sont enregistrées avec les paramètres de votre projet et peuvent être utilisées exactement comme les indices intégrés.
* **Comment créer** :
  1. Dans le panneau de configuration de l’indice, ouvrez la calculatrice de formules personnalisées
  2. Écrivez la formule en utilisant les **symboles de bande et d’emplacement**, et non les noms de bandes
  3. Enregistrez la formule sous un nom descriptif — elle apparaîtra alors au bas du menu déroulant des formules, et vous pourrez y faire glisser les cercles de canaux de votre caméra sur ses emplacements, exactement comme pour un préréglage intégré
* **Syntaxe de la formule** :
  * Emplacements de bandes : `x`, `y`, `z`, `a`, `b`, `c` — six positions que vous associez à des canaux réels par glisser-déposer
  * Opérateurs : `+`, `-`, `*`, `/`, `^` et `()` pour le regroupement
  * Fonctions : `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Pourquoi des symboles et non des noms de groupes** : une formule écrite sous la forme `(y-x)/(y+x)` fonctionne sur n’importe quelle caméra, car le mappage par glisser-déposer détermine si `y` correspond au NIR à 850 nm d’un filtre RGN ou au NIR à 808 nm d’un filtre OCN. Les préréglages intégrés sont stockés de la même manière — voir [Formules d’indices multispectraux](multispectral-index-formulas.md) pour connaître la forme exacte des symboles des 27 formules.
* **Où elles fonctionnent**: les formules personnalisées sont enregistrées avec les paramètres du projet et peuvent être utilisées dans le [Sandbox Index/LUT](../image-viewer-gui/index-lut-sandbox.md) ainsi que lors du traitement. Elles ne sont**pas** prises en charge par la liste de noms de l’CLI/SDK `--indices`, qui ne propose que les 22 noms de préréglages intégrés.

***

## Exportation

Ces paramètres contrôlent le format et la qualité des images traitées exportées.

### Format d’image calibrée

* **Type** : menu déroulant
* **Options** :
  * **TIFF (16 bits)** - Format TIFF 16 bits non compressé
  * **TIFF (8 bits)** - Format TIFF 8 bits non compressé
  * **TIFF (32 bits, pourcentage)** - TIFF 32 bits à virgule flottante avec valeurs de réflectance exprimées en pourcentage
  * **PNG (8 bits)** - Format PNG 8 bits compressé
  * **JPG (8 bits)** - Format JPEG compressé de 8 bits
* **Par défaut** : TIFF (16 bits) ; un projet entièrement en 8 bits affiche TIFF (8 bits) et ne propose que les formats 8 bits
* **Description** : permet de sélectionner le format de fichier pour l'enregistrement des images traitées et calibrées. Les fichiers exportés sont placés dans un sous-dossier dédié à chaque format, à l'intérieur du dossier de chaque caméra (`tiff16`, `tiff32`, `png8`, `jpg8`), avec un dossier `<Product>_Images/` par produit. Les fichiers exportés conservent le nom de fichier source — c’est le dossier, et non l’extension du nom de fichier, qui identifie le produit.
* **Recommandations de format** :
  * **TIFF (16 bits)** : Recommandé pour l’analyse scientifique et les flux de travail professionnels. Préserve une qualité maximale des données sans artefacts de compression. Idéal pour l’analyse multispectrale et le traitement ultérieur dans des logiciels SIG.
  * **Format TIFF (32 bits, pourcentage)** : Idéal pour les flux de travail nécessitant des valeurs de réflectance exprimées en pourcentage (0-100 %). Offre une précision maximale pour les mesures radiométriques.
  * **PNG (8 bits)** : Convient à la consultation sur le Web et à la visualisation générale. Taille de fichier réduite grâce à une compression sans perte, mais plage dynamique réduite.
  * **JPG (8 bits)** : Taille de fichier minimale, idéal uniquement pour les aperçus et l’affichage sur le Web. Utilise une compression avec perte qui ne convient pas à l’analyse scientifique.
* **Remarque** : LATTICE Radiance est toujours exporté au format TIFF 32 bits à virgule flottante, quel que soit ce paramètre.

***

## Enregistrer un modèle de projet

Cette fonctionnalité vous permet d’enregistrer les paramètres actuels de votre projet sous forme de modèle réutilisable.

* **Type** : Champ de saisie de texte + bouton « Enregistrer »
* **Description** : Saisissez un nom descriptif pour votre modèle de paramètres, puis cliquez sur l’icône d’enregistrement. Le modèle conservera tous les paramètres actuels de votre projet (détection des cibles, options de traitement, indices, et format d’exportation) afin de pouvoir les réutiliser facilement dans de futurs projets. Les modèles sont stockés dans le dossier `Project Templates/` situé dans le dossier de sauvegarde de votre projet, et peuvent également être sélectionnés ou exportés depuis le menu principal (*Sélectionner un modèle* / *Enregistrer un modèle* / *Exporter un modèle*).
* **Cas d'utilisation** :
  * Créer des modèles pour différents systèmes de caméras (RGB, multispectral, NIR)
  * Enregistrer des configurations standard pour des types de cultures spécifiques ou des workflows d'analyse
  * Partager des paramètres cohérents au sein d'une équipe
* **Mode d'emploi** :
  1. Configurez tous les paramètres souhaités pour votre projet
  2. Saisissez un nom de modèle (par exemple, « RedEdge Survey3 NDVI Standard »)
  3. Cliquez sur l’icône d’enregistrement
  4. Le modèle peut désormais être chargé lors de la création de nouveaux projets

***

## Dossier d’enregistrement des projets

Ce paramètre spécifie l’emplacement par défaut où les nouveaux projets sont enregistrés.

* **Type** : affichage du chemin d’accès au répertoire + bouton « Modifier »
* **Par défaut (sous Windows)** : `Documents\MAPIR\Chloros Projects` dans votre dossier « Documents » — généralement `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, ou à l’emplacement où OneDrive ou un profil itinérant a déplacé le dossier « Documents »
* **Par défaut (sous Linux)** : `~/Documents/MAPIR/Chloros Projects`
* **Description** : affiche le répertoire par défaut actuel dans lequel les nouveaux projets Chloros sont créés. Cliquez sur l’icône de modification pour sélectionner un autre répertoire. La modification est enregistrée sous la forme d’une seule ligne de texte dans `~/.chloros/working_directory.txt` — sous Windows, il s’agit de `C:\Users\<Username>\.chloros\working_directory.txt`. Si ce fichier est manquant ou indique un chemin d’accès qui n’existe plus, Chloros revient à la valeur par défaut indiquée ci-dessus. L’interface CLI lit et écrit le même fichier ; ainsi, `chloros-cli` et l’interface graphique s’accordent toujours sur l’emplacement des projets.
* **Les modèles de projet** se trouvent dans un sous-dossier `Project Templates/` de ce répertoire.
* **Quand modifier ce paramètre** :
  * Configurez-le sur un lecteur réseau pour la collaboration en équipe
  * Choisissez un lecteur disposant de plus d’espace de stockage pour les grands ensembles de données
  * Organisez les projets par année, client ou type de projet dans des dossiers distincts
* **Remarque** : la modification de ce paramètre n’affecte que les NOUVEAUX projets. Les projets existants restent à leur emplacement d’origine.
* **Longueur des chemins d’accès Windows** : à moins que les chemins d’accès longs Windows ne soient activés (option **Activer les chemins d’accès longs Windows** du programme d’installation), un dossier de projet dont le chemin comporte plus de 130 caractères environ est rejeté avant le traitement, car ses produits dépasseraient la limite de 260 caractères imposée par Windows pour les chemins d’accès. Voir [Lancement du traitement](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Persistance des paramètres

Un projet Chloros est un **dossier**. Tous les paramètres du projet sont enregistrés dans le fichier `project.json` qu’il contient ; les périphériques connectés sont mémorisés parallèlement dans les fichiers `cameras.json` et `sensors.json` ; ainsi, la réouverture d’un projet reconnecte automatiquement ses caméras et ses capteurs de lumière. Lorsque vous rouvrez un projet, tous les paramètres sont restaurés exactement tels que vous les aviez laissés. Les projets enregistrés peuvent également être pilotés en mode « headless » à l’aide du fichier `chloros-cli project` ou du fichier `open_project` fourni avec le SDK.

### Hiérarchie des paramètres

Les paramètres sont appliqués dans l’ordre suivant :

1. **Paramètres par défaut du système** - Paramètres par défaut intégrés définis par Chloros

2. **Paramètres du modèle** - Si vous chargez un modèle lors de la création d’un projet
3. **Paramètres du projet enregistré** - Paramètres enregistrés avec le fichier de projet
4. **Ajustements manuels** – Toutes les modifications que vous effectuez au cours de la session en cours

### Paramètres et traitement d’images

Les paramètres de traitement sont lus au démarrage d’une session de traitement. La modification d’un paramètre n’affecte pas rétroactivement les produits déjà enregistrés sur le disque — relancez le traitement pour appliquer les nouveaux paramètres. Certains paramètres n’ont aucune incidence sur le traitement :

* Résolution des vignettes d’image (affichage uniquement)
* Enregistrer le modèle de projet
* Enregistrer le dossier du projet

***

## Référence des clés de configuration

Pour l’automatisation (CLI `--config`, SDK `configure` ou lecture directe de `project.json`), voici les clés exactes sous `Project Settings` :

| Chemin d'accès à la clé | Type | Valeur par défaut |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | nombre compris entre 0 et 10 000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | nombre compris entre 0 et 100 | `60` |
| `Processing → Vignette correction` | booléen | `true` |
| `Processing → Reflectance calibration / white balance` | booléen | `true` |
| `Processing → Export sensor response` | booléen | `true` |
| `Processing → Export vignette corrected` | booléen | `true` |
| `Processing → Export debayered` | booléen | `true` |
| `Processing → Export preview` | booléen | `true` |
| `Processing → Export radiance` | booléen | `true` |
| `Processing → Export reflectance` | booléen | `true` |
| `Processing → Array alignment` | booléen | `true` |
| `Processing → Array alignment crop` | booléen | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | nombre 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | nombre -12..12 | `0` |
| `Processing → Apply PPK corrections` | booléen | `false` |
| `Processing → DAQ cap id` | ID du profil de limite ou `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | liste des configurations d'index | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Les clés `Array alignment` sont écrites lors du premier affichage de la section « Alignement du tableau » ou lorsqu’un appel d’automatisation les définit. En leur absence, le pipeline utilise les mêmes valeurs que celles indiquées ci-dessus (`true`, `true`, bilinéaire) ; ainsi, un projet .json qui ne les contient pas se comporte exactement de la même manière qu’un projet qui les contient.

### Clés stockées dans `project.json` sans contrôle dans le panneau des paramètres

Celles-ci se trouvent dans la même arborescence `Project Settings` et sont lues par le traitement, mais vous ne trouverez pas de widget correspondant dans la barre latérale :

| Chemin de la clé | Type | Par défaut | Définie par |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Remplace la manière dont les fichiers TIFF d’entrée LATTICE sont interprétés ; `auto` déduit cette information à partir de la balise XMP de chaque fichier `Chloros:ProcessingLevel` ainsi que du nombre de canaux. Ignoré pour les captures `.raw` de type Survey3. Ce n’est délibérément pas un paramètre de l’interface graphique — la valeur « auto » est correcte dans tous les cas normaux. |
| `Processing → Target reflectance dir` | chaîne de caractères de chemin d’accès | `""` | `chloros-cli process --target-reflectance-dir`, ou l’API cible du projet |
| `Processing → Target reflectance config` | dictionnaire indexé par le numéro de série de la caméra | `{}` | Enregistrement d’une cible dans l’image (mode `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | chaîne de caractères de chemin d’accès | `""` | SDK `process_folder(daq_log_path=…)`. Pointe vers un enregistrement `.daq` ou un dossier contenant plusieurs enregistrements |
| `Target Detection → Minimum calibration target squares` | nombre | `4` | Valeur par défaut héritée ; aucun contrôle et aucun indicateur CLI |
| `UI → Grid thumbnail size` | nombre | `160` | Curseur de zoom des vignettes propre à la grille d’images |

Deux préférences de la visionneuse sont stockées **au niveau supérieur dans `project.json`**, entièrement en dehors de `Project Settings`, car il s’agit d’états d’affichage plutôt que de paramètres de traitement :

| Chemin d'accès | Type | Valeur par défaut | Définie par |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | entier 1–256 | `1` | Contrôle GSD (px) de l’onglet « Image » — voir [Ouvrir une image en plein écran](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Bonnes pratiques

1. **Commencez par les valeurs par défaut** : les paramètres par défaut conviennent à la plupart des systèmes de caméras MAPIR et aux flux de travail courants.
2. **Créez des modèles** : une fois que vous avez optimisé les paramètres pour un flux de travail ou une caméra spécifique, enregistrez-les sous forme de modèle afin de garantir la cohérence d’un projet à l’autre.
3. **Testez avant le traitement complet** : lorsque vous testez de nouveaux paramètres, effectuez des essais sur un petit sous-ensemble d’images avant de traiter l’ensemble de vos données.
4. **Documentez vos paramètres** : utilisez des noms de modèles descriptifs qui indiquent le système de caméra, le type de traitement et l’utilisation prévue (par exemple, « Survey3\_RGB\_NDVI\_Agriculture »).
5. **Choix du format d’exportation** : choisissez votre format d’exportation en fonction de l’utilisation finale :
   * Analyse scientifique → TIFF (16 bits ou 32 bits)
   * Traitement SIG → TIFF (16 bits)
   * Visualisation rapide → PNG (8 bits)
   * Partage sur le Web → JPG (8 bits)

***

Pour plus d’informations sur les indices multispectraux dans Chloros, consultez la page [Formules des indices multispectraux](multispectral-index-formulas.md).
