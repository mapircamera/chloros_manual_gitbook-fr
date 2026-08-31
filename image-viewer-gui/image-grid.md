# Grille d’images

Une fois les images importées dans un projet, elles s’affichent sous forme de grille dans la zone principale. C’est dans cette grille que vous choisissez **la version de chaque image que vous souhaitez visualiser** : les boutons situés au-dessus permettent de basculer simultanément toutes les vignettes entre les fichiers sources et chaque produit traité.

## Taille des vignettes

Utilisez le curseur de zoom en haut à droite pour ajuster la taille des vignettes. Le curseur va de **64 px à 1 200 px**.

* **Ctrl + molette de la souris** permet également de redimensionner les vignettes.
* **Ctrl + `+`**/**Ctrl + `=`**et**Ctrl + `−`** modifient la taille par incréments de 4 px à chaque pression. La plage de réglage s&#x27;arrête à 64 px à la plus petite valeur et, à la plus grande valeur, à la taille permettant d&#x27;afficher exactement deux vignettes par ligne dans la fenêtre actuelle.
* La taille que vous choisissez est enregistrée avec le projet (`UI → Grid thumbnail size` dans `project.json`, par défaut `160`) ; ainsi, la rouvrir permet de la rétablir.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>La *résolution* des vignettes est un paramètre distinct de la *taille* des vignettes : voir **Affichage → Résolution des vignettes d’image** dans [Paramètres du projet](../project-settings/project-settings.md) (par défaut 512 px sur le côté le plus long). La taille correspond à la dimension de la tuile affichée ; la résolution correspond au niveau de détail utilisé pour la remplir.***

## La barre d’outils de la grille

La rangée de boutons située au-dessus de la grille comporte jusqu’à trois groupes, de gauche à droite :

1. **Par déclencheur / Par caméra** — mode de regroupement. Apparaît uniquement pour les projets contenant des captures LATTICE.
2. **Boutons de filtre de caméra** — un par caméra LATTICE. Apparaissent uniquement en mode Par caméra.
3. **Boutons de mode d’exportation/d’affichage** — qui déterminent quel produit chaque vignette affiche.

Lorsque la fenêtre est trop étroite pour les afficher tous, les groupes se replient de droite à gauche pour former des menus déroulants au survol : les boutons d’exportation/d’affichage se replient en premier, puis les boutons de caméra. Le groupe replié ne laisse apparaître qu’un seul bouton indiquant le choix actuellement actif ; en passant la souris dessus, l’ensemble complet s’affiche. **Les modes « Par déclencheur » / « Par caméra » ne se replient jamais.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Boutons d’exportation et d’affichage

Ces boutons permettent de basculer entre les différents types d’images dans la grille de vignettes. **Un bouton apparaît dès que le produit qu’il désigne existe** — ce qui, pour les fichiers source, signifie immédiatement lors de l’importation, et non après le traitement. Chloros réanalyse les produits du projet pendant qu’une session est en cours ; les boutons apparaissent donc au cours du traitement, à mesure que chaque produit est enregistré sur le disque.

### Le bouton de base

Le bouton d’exportation situé à l’extrême gauche indique **ce que vous avez réellement importé** :

| Ce que vous avez importé | Libellé du bouton |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Captures LATTICE avec un aperçu à l’écran à côté de l’image brute | `PNG` ou `TIFF`, selon les aperçus |
| Captures LATTICE où le fichier de base **est** l&#x27;image RAW | *pas de bouton* — `RAW (Original)` affiche déjà ce fichier |

Dans un projet mixte, le libellé correspond à l&#x27;extension utilisée par la plupart des images.

### Boutons de produit

| Bouton | Affiche | Quand il apparaît |
| --- | --- | --- |
| **Cibles** | Images comportant une cible d’étalonnage détectée | Après une session ayant détecté des cibles |
| **Réflectance** | Images de réflectance calibrées | Projets Survey3 uniquement — les projets LATTICE utilisent `RAW (Reflectance)` à la place, de sorte que la grille n’affiche jamais deux boutons de réflectance |
| **Balance des blancs** | Le produit avec balance des blancs (caméras RGB) | Après traitement |
| **Correction de la vignette** | Le produit de secours non calibré avec correction de la vignette | Après une session où le calibrage de la réflectance n’a pas pu être appliqué et où la *correction de la vignette* était activée |
| **Réponse du capteur** | La solution de secours non calibrée basée sur la réponse du capteur | Idem, mais avec la *correction de vignettage* désactivée |
| **`RAW (<INDEX> Index)`** | Un bouton par indice calculé | Après une session avec des indices configurés |
| **`<INDEX> LUT`** | Un bouton par indice avec mappage de couleurs | Après une exécution avec une table de conversion (LUT) configurée |
| **`<Index> <Index\|LUT> <NNN>`** | Un bouton par exécution d&#x27;exportation [Index/LUT Sandbox](index-lut-sandbox.md) | Dès la fin d&#x27;une exportation Sandbox |

### Boutons de niveau LATTICE

Les projets contenant des captures LATTICE ajoutent ces boutons, libellés par le nom du niveau plutôt que par le nom du produit :

| Bouton | Niveau |
| --- | --- |
| **RAW (Original)** | L&#x27;image brute source, telle qu&#x27;elle a été importée |
| **RAW (Radiance)** | Radiance spectrale en Float32, W/m²/sr/nm |
| **RAW (Réflectance)** | Réflectance en uint16, 32768 = ρ 1,0 |

`RAW (Original)` est disponible dès l’importation — il ne nécessite aucun traitement. Lorsqu’une importation LATTICE ne comporte aucun bouton de base (le fichier de base de chaque capture étant son image brute), la grille se déplace automatiquement vers le premier bouton de niveau disponible afin que la mise en surbrillance de la barre d’outils corresponde à ce que vous voyez.

Les exportations à deux niveaux Chloros ne disposent **d’aucun bouton de grille qui leur soit propre** :

* **Débayérisé** — la vue `RAW (Original)` s’affiche déjà sous forme débayérisée ; l’ajout d’un deuxième bouton sur une image visuellement identique serait donc superflu. Le produit `RAW (Debayered)` est toujours enregistré sur le disque et reste sélectionnable dans le menu déroulant des calques en plein écran.
* **Aperçu** — sur les caméras RGB, l’aperçu est enregistré sous le nom de couche `White Balanced`, qui dispose bien d’un bouton. Sur les caméras multispectrales, elle est enregistrée sous le nom `RAW (Preview)` et est accessible depuis le menu déroulant des couches en plein écran.

{% hint style="info" %}
Ces boutons de niveau ne s’affichent que pour les projets contenant effectivement des images LATTICE. Les projets Survey3 enregistrent certains de ces mêmes noms de couches internes, et les boutons sont masqués pour ceux-ci ; ainsi, une grille Survey3 conserve son ensemble familier `JPG / Targets / Reflectance`.
{% endhint %}

Un clic sur la vignette d’une grille ouvre la [Visionneuse d’images](opening-an-image-full-screen.md) en plein écran sur **le même produit que celui affiché par la grille** — si la grille est configurée sur `Targets`, la vignette ouvre l’image cible exportée.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Regroupement d’un projet LATTICE : par déclencheur ou par caméra

Les captures en matrice produisent plusieurs images d’un même instant provenant de différents modules de caméra. Le regroupement détermine la manière dont la grille les empile. Les deux modes affichent des barres d’en-tête repliables sur toute la largeur ; **chaque groupe s’ouvre par défaut**, et Chloros mémorise celles que vous fermez. L’état de repliement est suivi séparément pour chaque mode ; ainsi, fermer un groupe en mode « Par caméra » ne ferme rien en mode « Par déclenchement ».

### Par caméra (par défaut)

Un groupe par module de caméra. L&#x27;en-tête affiche le modèle et le numéro de série de la caméra (`LATT-M3M — <serial>`) ainsi que le nombre de photos. Les vignettes à l&#x27;intérieur d&#x27;un groupe sont classées par ordre chronologique selon l&#x27;événement de capture.

Dans ce mode, la barre d’outils comporte également un **bouton de filtrage par caméra**, libellé `MODEL (SERIAL)`. Toutes les caméras sont sélectionnées par défaut ; cliquer sur un bouton désélectionne cette caméra et supprime son groupe de la grille. C’est le moyen le plus rapide de passer en revue une bande sur l’ensemble d’un vol.

### Par déclenchement

Un groupe par événement de capture — l’ensemble des images prises par tous les modules lors du même déclenchement. L’en-tête indique l’heure de capture, le nombre de caméras ayant participé et un badge par modèle de caméra dans le groupe. Les vignettes au sein d’un groupe sont classées par numéro de série de caméra, de sorte que la même bande se trouve dans la même colonne pour chaque déclenchement.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Les images non-LATTICE d’un projet mixte ne sont pas regroupées — elles s’affichent sous forme de vignettes simples après les groupes.

***

## Les vignettes de la grille respectent la taille de bloc GSD

Si vous avez défini une taille de bloc **GSD (px)** dans la barre latérale de l’onglet « Image », les vignettes de la grille sont affichées avec cette même résolution au sol — et pas seulement en mode plein écran. Une taille de bloc de 8 signifie que chaque pixel affiché correspond à la moyenne d’un bloc de 8 × 8 pixels sources, partout dans l’application où l’image est affichée.

Comme une tuile ne mesure au départ que quelques centaines de pixels de large, les tailles de bloc grossières cessent d’avoir un impact visible sur la grille bien avant qu’elles ne le soient en mode plein écran : un cadre de 4 000 px dessiné dans une tuile de 160 px correspond déjà à environ 25 pixels source par pixel affiché. Voir [Ouvrir une image en plein écran](opening-an-image-full-screen.md#gsd-block-size) pour le contrôle lui-même.

***

## Pages associées

* [**Ouverture d’une image en plein écran**](opening-an-image-full-screen.md) — la visionneuse plein écran, les valeurs du curseur et l’histogramme
* [**Calques d’image**](image-layers.md) — le menu déroulant des calques dans la visionneuse plein écran
* [**Bac à sable Index/LUT**](index-lut-sandbox.md) — création et exportation de visualisations d’index
* [**Paramètres du projet**](../project-settings/project-settings.md) — les boutons d’exportation qui déterminent quels produits sont disponibles
