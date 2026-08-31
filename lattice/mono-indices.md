# Caméras monochromes et indices de végétation

## Une caméra = une bande

Une caméra **M3M**est la version monochrome de la**M3C**de Bayer : elle est équipée d’un capteur IMX265 monochrome derrière un seul filtre d’interférence à bande étroite. La chaîne de caractères du modèle désigne la bande — `M3M-<lens>-F<wavelength>`, par exemple `M3M-L87-F685` (affiché dans Chloros sous la forme `LATT-M3M-L87-F685`). Le capteur fournit une**seule bande en niveaux de gris** sans mosaïque de Bayer : il n’y a donc rien à démosaiquer, aucune diaphonie intercanal à démêler et aucun équilibre des blancs à régler.

Conséquences à connaître avant de planifier un système monochrome :

* **La radiance et la réflectance sont entièrement définies par bande.**Il s’agit de cartes radiométriques par bande ; ainsi, une caméra M3M produit une radiance calibrée en float32 (W/m²/sr/nm) et une réflectance en uint16 (`32768` = ρ 1,0), exactement comme le fait une bande M3C. Les images monochromes comportent une matrice de réponse du capteur**d’identité** — aucun démixage 3×3 n’est nécessaire ni appliqué.
* **Une seule caméra mono ne peut pas produire d’indice de végétation.** NDVI, NDRE et leurs équivalents nécessitent au moins deux bandes. Pour calculer des indices à partir d’un matériel mono, il faut combiner plusieurs caméras M3M — voir ci-dessous.
* Les caméras M3M transmettent en continu **Mono12** (12 bits, 2 octets/pixel sur le réseau), ce qui est important pour la [gestion de la bande passante du réseau de capteurs](arrays.md#bandwidth-the-rules-of-thumb).

## Ce que Chloros ignore pour le mode mono — et comment il vous l’indique

Les étapes du pipeline couleur ne s’appliquent tout simplement pas à un capteur monobande. Chloros **les ignore en affichant un message d’une ligne** plutôt que de générer une erreur, tout en continuant à les exécuter normalement pour toute caméra M3C (Bayer) dans la même session :

| Étape | Comportement Mono (M3M) | Comportement M3C |
| --- | --- | --- |
| Démosaïquage / débayérisation | Ignoré — le niveau d’exportation `debayered` est une image en niveaux de gris à 1 canal. | Démosaïquage à 3 canaux. |
| Balance des blancs (`lattice white-balance`) | Ignoré avec un message d’une ligne. | S’exécute normalement. |
| Profil de couleur (`lattice color-profile`) | Ignoré avec un message d’une ligne. | S’exécute normalement. |
| Saturation/contraste (`lattice color`) | Ignoré avec un message d’une ligne. | S’exécute normalement. |
| Démixage de la diaphonie spectrale | Identité (pas de matrice 3×3). | Matrice 3×3 appliquée par caméra. |
| Radiance / réflectance | **S&#x27;exécute** — par bande, entièrement calibré. | S&#x27;exécute par bande. |

L&#x27;interface graphique applique le même filtrage : pour une caméra mono, le volet des paramètres par caméra masque les lignes réservées à RGB (balance des blancs, gamma, profil de couleur, saturation, Contraste, division des canaux), et l’histogramme en temps réel est verrouillé sur une seule courbe **MONO**. Le discriminant tout au long de l’empilement est le jeton `M3M` dans la chaîne du modèle, affiché dans l’interface graphique/SDK sous la forme `is_mono`.

## Les index nécessitent ≥ 2 bandes : alignement → empilement → indexation

Le flux de travail d’indexation mono se déroule toujours en trois étapes :

1. **Alignement** — orienter plusieurs caméras M3M sur différentes longueurs d’onde (par exemple, une F650 « Red » et une F850 « NIR »), les relier sous forme de [réseau multicaméra](arrays.md), puis laisser Chloros calculer la transformation de co-enregistrement entre les caméras.
2. **Pile** — les images alignées forment une seule image multibande (chaque caméra apporte une bande nommée).
3. **Index** — évaluez une formule d’index sur les bandes de la pile, en la rendant éventuellement via une table de conversion (LUT).

Dans l’interface graphique, l’ensemble de cette chaîne correspond au mode d’affichage **Combined Cameras**: la composition en direct est déjà alignée, et le calculateur d’indice (ci-dessous) de la matrice définit la formule qu’elle rend. Les exportations capturées peuvent être déformées pour obtenir le même alignement grâce à l’option de capture**Aligned**.

## Le calculateur d’indice

Le calculateur d’indice définit l’expression d’indice utilisée par la vue en direct et les exportations d’indice par caméra. Il s’agit d’une interface commune, accessible depuis deux emplacements dans la barre latérale de l’onglet « Caméras » :

* **Par caméra**— Aperçu en direct → roue dentée**Index** (caméras Bayer RGN/OCN/NGB uniquement ; une caméra mono seule ne dispose pas de contrôle d’indice, car une seule bande ne permet pas de calculer un indice).
* **Par matrice**— Paramètres de la matrice → Aperçu en direct → roue dentée**Indice**. Il s’agit du chemin mono : la liste des bandes couvre**toutes les caméras de la matrice**, de sorte qu’une paire mono apporte ici ses deux bandes.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Ses commandes, de haut en bas :

* **Puces de bande** (« Bandes — cliquez pour ajouter à l’expression ») — un bouton par bande disponible, libellé par le nom de la couleur + la longueur d’onde en nm (les noms de couleurs en double sont désambiguïsés, par exemple « Couleur 850 »). Un clic insère le jeton de bande à l’emplacement du curseur. Les bandes provenant de caméras incapables de produire une radiance par bande (RGB/FRGB) sont filtrées.
* **Boutons d’opérateurs et de fonctions** — `+ - * / ( ) ^ ,` et `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Zone de texte d’expression** — formule à saisir librement ; l’espace réservé affiche la forme classique NDVI de `(NIR - Red) / (NIR + Red)`. Un aperçu tokenisé en lecture seule situé au-dessus rend les puces de bande, les nombres et les indicateurs sous forme de tokens inconnus.
* **Bannière de validité**— grise « Vide — aucun index ne sera appliqué » ; verte « Expression valide » ; rouge avec l’erreur d’analyse spécifique (bande inconnue, bande ambiguë captée par plusieurs caméras, parenthèse manquante, …) ; ou orange lorsque l’expression est valide mais**constante** (par ex. `X/X`, ou un dénominateur de type NDVI saisi avec `−` au lieu de `+`) — une constante mappe l’image entière sur une seule couleur.
* Un avertissement orange distinct s’affiche si l’expression appliquée est correcte mais que l’**image en direct est uniforme** (scène plate ou saturée) — l’affaissement de l’histogramme est détecté automatiquement.
* **Appliquer la LUT**(activée par défaut ; désactivée = étirement en niveaux de gris),**Niveau**2/3/5/7 stops (7 stops par défaut), et les entrées**Min / Max**encadrant la barre de dégradé. La valeur minimale par défaut est**0,2**— elle agrandit la rampe de couleurs dans la plage pertinente pour la végétation, tandis que les valeurs inférieures sont affichées en niveaux de gris ; réglez Min sur −1 pour obtenir la plage d’indice complète (le bouton**Réinitialiser** rétablit la plage −1…+1). La valeur maximale par défaut est 1.
* **Histogramme en temps réel** de la distribution de l’indice — barres à échelle carrée, lignes de percentiles p2/p98 en ambre, une ligne médiane blanche et des indications de valeurs hors plage («◀ N % &lt; lo » / « hi &lt; N % ▶») qui passent à l’ambre au-delà de 1 % pour signaler la nécessité d’élargir la fenêtre Min/Max.
* **Appliquer**valide l’expression sur le flux en direct ; les ajustements de la table de conversion (LUT) s’appliquent en temps réel sans avoir à appuyer sur « Appliquer ». Les expressions sont délibérément**limitées à la session** — elles ne sont pas conservées d’une session à l’autre.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Le chemin CLI

La même chaîne alignement → pile → index, scriptable de bout en bout :

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mappe les symboles d’un préréglage aux noms de bandes de la pile. Deux règles vous éviteront un échec d’exécution :

* **Les symboles sont sensibles à la casse** et doivent correspondre exactement aux noms de canaux du préréglage — les préréglages utilisent des minuscules (les NDVI sont `red`,`nir` ; vérifiez `--list-presets`). `--channel red=Red_660` fonctionne ; `--channel RED=660` échoue avec une erreur `channel_map missing entries`.
* Le côté « bande » doit désigner une bande de la pile alignée (`lattice align-info --profile align.json` en donne la liste). Le mode hors ligne accepte également les indices de bande à partir de 0, par exemple `--channel red=0 --channel nir=1`.

`lattice index` s&#x27;exécute également entièrement hors ligne sur un fichier TIFF multibande aligné et enregistré :

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Préréglages d’index

`lattice index --preset` (ainsi que l’option [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) de l’onglet Image, qui utilise le même moteur) propose ces **22 préréglages** :

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Exécutez `chloros-cli lattice index --list-presets` pour obtenir la formule et les symboles de canal de chaque préréglage, et `--list-gradients` pour les dégradés de couleurs disponibles. Les formules personnalisées utilisent `--formula EXPR` avec la même syntaxe que le calculateur d’index. Notez que cette liste de préréglages est spécifique au moteur d’indices LATTICE — le menu déroulant « Traitement » des paramètres de projet pour les images importées propose une liste différente (voir [Formules d’indices multispectraux](../project-settings/multispectral-index-formulas.md)).

L&#x27;ensemble complet des indicateurs (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, les curseurs d&#x27;alignement et de déformation pour `--live`, et bien d’autres) est documenté dans la [Référence CLI § Indices / Mathématiques de la végétation](../reference/cli-reference.md#index--vegetation-maths) ; les équivalents de SDK se trouvent dans la [Référence SDK](../reference/sdk-reference.md).

## Capture des produits d’index à partir d’un tableau mono

Lorsqu’un tableau est connecté et qu’une expression d’index est appliquée, `array-capture` (ou la commande **Capture All** de l’interface graphique) enregistre les niveaux d’exportation par caméra *ainsi que* le rendu de l’index — `--index`/`--no-index` permet de l’activer ou de le désactiver sur CLI, et la capture inclut par défaut tous les niveaux applicables. La contribution d’une caméra mono à chaque groupe de capture correspond à sa bande unique aux niveaux brut/débayérisé (niveaux de gris)/radiance/réflectance, à laquelle s’ajoute le composite d’index combiné partagé lorsque le réseau fonctionne en mode combiné. Voir [Réseaux multicaméras § Capture](arrays.md#capturing-monitoring-vs-analysis).
