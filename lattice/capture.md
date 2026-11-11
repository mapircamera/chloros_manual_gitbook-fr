# Paramètres et modes de capture

La capture dans l’onglet « Caméras » s’effectue à l’aide d’un bouton rouge **Capturer tout**et d’un volet**Paramètres de capture** qui définit le résultat de ce bouton : quelles caméras sont utilisées, quels types de fichiers chaque caméra enregistre, et si l’obturateur se déclenche une seule fois, en continu ou à intervalles réguliers. Cette page décrit l’ensemble du processus : la configuration, la capture elle-même, l’emplacement des fichiers sur le disque et la manière de les retraiter ultérieurement pour obtenir des produits calibrés. Les commandes relatives aux caméras et aux réseaux de caméras se trouvent dans [Paramètres de caméra](camera-settings.md).

{% hint style="info" %}
**Les captures nécessitent l’ouverture d’un projet.** Les boutons « Capture All » et l’icône en forme de roue dentée des paramètres de capture sont désactivés tant qu’aucun projet n’est ouvert (« Créez ou ouvrez un projet pour enregistrer les captures »). Chaque capture est enregistrée dans le dossier du projet sous `captures/`.
{% endhint %}

## Le volet « Paramètres de capture »

Ouvrez-le à l’aide de l’**icône en forme d’engrenage située à côté de « Tout capturer »**dans la liste des caméras de la barre latérale, ou à l’aide du bouton**« Ouvrir les paramètres de capture… »** situé en bas de n’importe quel volet de paramètres propre à une caméra. L’en-tête indique « Paramètres de capture » et comporte un bouton ← de retour.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Vos sélections ici — caméras incluses, cases à cocher par type et mode de capture — sont enregistrées **par projet** et restaurées lorsque vous le rouvrez.

### Modes de capture

Trois boutons de mode en haut du volet :

| Mode | Fonction | Sous-paramètres (par défaut) |
| --- | --- | --- |
| **Unique** *(par défaut)* | Une seule capture sur toutes les caméras sélectionnées. | — |
| **Continu**| Captures successives jusqu’à ce qu’une condition d’arrêt soit remplie. | Arrêt par**nombre de captures** (par défaut 1) *ou* **durée de capture** (par défaut 10 s ; unités : secondes / minutes / heures / jours). |
| **Intervalle**(timelapse) | Série de captures selon un minuteur. |**Captures / intervalle**(par défaut 1) ·**Toutes les**N unités (par défaut 5 s) ·**Pendant** N unités (par défaut 1 m). |

En mode Continu ou Intervalle, le bouton « Capture All » devient un bouton **Arrêt (N)** pendant l’exécution, comptant les captures au fur et à mesure qu’elles sont enregistrées. Un disque plein arrête l’exécution (voir [Le flux « Capture All »](#the-capture-all-flow)).**Le mode Continu sur les matrices s’exécute au sein du backend (nouveauté de la version 1.3.0).** Lorsque toutes les caméras de la liste appartiennent à un réseau (les caméras autonomes étant masquées ou exclues), le mode « Continu » n’envoie plus une requête par capture : le backend Chloros

exécute lui-même la boucle de capture, écrivant un groupe synchronisé tout en sélectionnant le suivant, et le bouton interroge son compteur. Si la liste contient des caméras autonomes, Chloros

effectue les captures passe par passe, comme auparavant.

{% hint style="warning" %}
**Capture rapide sous Windows

: veillez à ce que le backend Chloros

soit exclu de Microsoft Defender.** L&#x27;analyse en temps réel de chaque fichier écrit lors d&#x27;une capture à haut débit coûte environ 30 % du débit de capture selon les tests de MAPIR

(des centaines de fichiers par seconde pour une petite zone d’intérêt). Le programme d’installation Windows

propose l’option **« Exclure le backend Chloros

de l’analyse de Microsoft Defender (capture bien plus rapide ; recommandé) »**, cochée par défaut, et supprime cette exclusion lors de la désinstallation. Si vous l’avez décochée, relancez le programme d’installation en la cochant, ou ajoutez une exclusion de processus Defender pour `chloros-backend.exe` (dans le dossier `backend` du dossier d’installation d’Chloros

). Lorsque Defender utilise un cœur de processeur ou plus pendant la capture continue d’un ensemble de caméras, le backend Chloros

l’indique une fois dans son journal, en proposant ces deux mêmes solutions.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Sélection des caméras et des types d’exportation

Le texte d’aide du volet résume la situation : choisissez les caméras et les types d’exportation générés par « Capture All » (Tout capturer) — tout est activé par défaut, et les choix sont enregistrés avec ce projet.

* Les boutons **Tout sélectionner / Ne rien sélectionner** activent ou désactivent simultanément la case à cocher d’inclusion de chaque caméra.
* **Commutateurs de type d’exportation groupés**(deux rangées de boutons) :**Tout en Raw / Tout en débayérisé / Tout en aperçu / Tout en radiance / Tout en réflectance / Tout en indice**. Chacune est indiquée par une couleur à trois états : vert ✓ = activé pour toutes les caméras qui le prennent en charge, orange – = activé pour certaines, gris = aucune. Un bouton est désactivé lorsqu’aucune caméra connectée ne prend en charge ce type. Tous sont grisés lorsque l’option « Capture la plus rapide » est activée.
* **Lignes par caméra** : une case à cocher « Inclure », ainsi qu’une liste déroulante (▸/▾) des types d’exportation applicables à cette caméra, avec des cases à cocher individuelles. La ligne affiche un compteur de présence tel que « 4/6 ».

### Types d’exportation et caméras compatibles

Il existe six types d’exportation : **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Seuls ceux qui sont applicables apparaissent sur la ligne de chaque caméra :

| Type d&#x27;exportation | Contenu | RGB

(FRGB) | Multispectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaïque de Bayer (mono : la bande unique) provenant directement du capteur | ✓ | ✓ | ✓ |
| **Démosaïqué** | Démosaïquage linéaire : le décalage de noir (DSNU) du pack d’étalonnage est retiré du fichier brut, puis un démosaïquage tenant compte des contours est effectué — sans correction de linéarité, champ plat ni démixage. Enregistré au format RGB

TIFF

à 3 canaux (mono : 1 canal en niveaux de gris) | ✓ | ✓ | ✓ |
| **Aperçu** | Chaîne d&#x27;affichage complète (balance des blancs + gamma selon le profil de la caméra, plus le réglage « Highlights » de la caméra RGB

; multispectral : étirement en fausses couleurs) | ✓ | ✓ | ✓ |
| **Radiance** | float32 W/m²/sr/nm via la chaîne radiométrique complète | — (non disponible) | ✓ | ✓ |
| **Réflectance** | uint16 ρ (32768 = 1,0) | — (non disponible) | ✓ — affiché uniquement lorsque la caméra dispose d’un capteur de lumière DAQ (propre ou hérité de son réseau) | identique au multispectral |
| **Indice** | Rendu de l’indice de végétation (LUT) | — | ✓ — nécessite une expression d’indice activée et non vide sur la caméra, et n’est pas proposée aux membres d’un réseau combiné (le réseau possède un indice partagé) | — (un indice nécessite ≥2 bandes ; voir [Caméras mono et indices de végétation](mono-indices.md)) |

La radiance et la réflectance ne sont jamais proposées pour les caméras RGB

— la radiance par pixel Bayer n’a pas de sens pour un capteur photométrique à large bande.

### Capture la plus rapide

Le bouton **⚡ Capture la plus rapide — format brut uniquement**(orange lorsqu’il est activé) remplace toutes les sélections d’exportation par**format brut uniquement** — avec en plus un composite d’indices combinés gratuit pour les matrices — afin que l’image soit enregistrée aussi rapidement que possible : les calculs de radiance/réflectance/affichage sont entièrement ignorés au moment de la capture.

{% hint style="info" %}
**Un fichier `.daq` est tout de même enregistré.** Lorsqu’un capteur de lumière est attribué, la « Capture la plus rapide » enregistre tout de même la mesure DAQ descendante (dans le dossier `daq/` des captures) — ce qui permet de générer ultérieurement les produits de radiance, de réflectance et d’indice par un nouveau traitement (voir [Retraitement des captures](#re-processing-captures-into-calibrated-products)). Fastest Capture n’altère pas non plus vos sélections de cases à cocher : désactivez-le et elles réapparaissent.
{% endhint %}

### Commandes par réseau

Chaque réseau connecté dispose de sa propre fiche de groupe dans le volet :

* **Case à cocher « Inclure »** (à trois états pour tous les membres) et le nom du réseau avec son mode d’affichage : « (combiné | séparé) ».
* Case à cocher **Aligné** (activée par défaut) : déforme les exportations des éléments selon le profil d’alignement du réseau afin que les exportations soient alignées au pixel près entre les caméras. Les données brutes restent non déformées mais comportent la transformation dans leurs métadonnées. (Le profil lui-même est calculé dans le [volet des paramètres du réseau](camera-settings.md#alignment-co-registration).) Les fichiers des membres du réseau sont enregistrés dans l’orientation native de chaque caméra, les paramètres de miroir/rotation étant enregistrés dans le fichier plutôt qu’appliqués.
* Les lignes des caméras membres sont imbriquées à l’intérieur de la carte.

La carte de matrice héberge également deux enregistreurs. Considérez-les comme **la surveillance par opposition à l’analyse** :

| Enregistreur | Niveau | Ce qu’il enregistre |
| --- | --- | --- |
| **● Enregistrer la vidéo d’index / ■ Arrêter l’enregistrement** *(matrices combinées uniquement)* | **Surveillance** | Le composite d’index combiné en direct sous forme de vidéo à 10 images par seconde — 8 bits, résolution de prévisualisation, LUT intégrée. Nécessite un projet ouvert et une vue en direct en streaming. Affiche les images et le temps écoulé pendant l’enregistrement. |
| **⦿ Enregistrer une rafale brute / ■ Arrêter la rafale brute** *(tout réseau)* | **Analyse**| Images Bayer brutes à la fréquence de capture en direct (sans traitement), accompagnées d’un manifeste par image et des mesures `.daq`, enregistrées au format `captures/bursts/`. Après une rafale, un bouton**Créer une vidéo** apparaît : il retraite la rafale hors ligne pour obtenir une vidéo calibrée — indice combiné et/ou radiance / réflectance / indice par caméra — ainsi que des fichiers TIFF en option. La création de l’indice combiné démarre automatiquement lorsque vous arrêtez la rafale. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Le flux

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

« Capture All »

Appuyez sur **Capture All** dans la liste des caméras de la barre latérale :

1. Chaque caméra incluse, visible et non mise en pause effectue une capture selon les types d’exportation sélectionnés. **Les réseaux de caméras se déclenchent de manière synchronisée** (un seul groupe synchronisé pour tous les membres — voir [Réseaux multicaméras](arrays.md)) ; les caméras autonomes capturent individuellement.
2. Les caméras masquées (œil) ou mises en pause sont ignorées. Un ensemble n’est entièrement bloqué que lorsque *tous* ses membres sont masqués ou mis en pause.
3. Lorsqu’un capteur de lumière est attribué, la mesure DAQ correspondante de l’irradiance descendante est enregistrée sous forme de fichier `.daq` dans le dossier `daq/` des captures — même pour les captures en format brut uniquement — afin que des produits radiométriques puissent toujours être dérivés ultérieurement.
4. Le bouton affiche la progression en temps réel — « Capture en cours… terminée/total » — et, en mode Continu/Intervalle, devient **Arrêt (N)**. Chaque élément de capture dispose d’un délai d’expiration de 300 s.
5. À la fin du passage, un message de résultat s’affiche indiquant **« N fichiers enregistrés »**ou**« N enregistrés, F échecs »**, suivi de « (S masqués/mis en pause/ignorés) » lorsque des caméras ont été ignorées. Si le disque est plein, l’exécution s’arrête et affiche**« Arrêté : disque plein. N fichiers enregistrés, F échecs »** — chaque fichier qui n’a pas pu être écrit est compté comme un échec.

## Emplacement de stockage des captures

Les captures sont enregistrées dans le projet ouvert sous le chemin `<project>/captures/`. Chaque type d’exportation est stocké dans son **propre sous-dossier**, de sorte qu’une capture à plusieurs niveaux ne mélange jamais les types :

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` correspond à l’horodatage de la capture et `<serial>` au numéro de série de la caméra. Les captures autonomes sont nommées `capture_<ts>_SN<serial>_<level>` ; les captures en série issues d’un seul déclencheur synchronisé sont nommées `sync_<ts>_SN<serial>_<level>` et **partagent un même horodatage pour toutes les caméras du groupe** (le suffixe de niveau est supprimé lorsqu&#x27;une caméra n&#x27;enregistre qu&#x27;un seul niveau).
* **Une particularité à noter :** le niveau d’affichage est stocké dans un dossier nommé `preview/` tandis que les fichiers conservent `_display` dans leur nom — le dossier et le suffixe diffèrent pour ce niveau uniquement.
* Les niveaux inconnus sont placés par défaut dans un dossier portant leur propre nom ; si un sous-dossier ne peut pas être créé, le fichier est enregistré à la racine du répertoire « captures » plutôt que d’être perdu.
* Les fichiers TIFF de capture sont compressés sans perte (DEFLATE) par défaut et contiennent l’intégralité de leurs métadonnées d’étalonnage et de traitement **dans le fichier XMP** — les captures sont auto-descriptives, sans fichier d’accompagnement autre que la lecture `.daq` (et son `.daq.cameras.json`).
* **Masques de limitation (nouveauté de la version 1.3.0).**Lorsque le démixage spectral 3×3 devait limiter une bande à 0 sur au moins un pixel, les produits de radiance, de réflectance et d’indice reçoivent un masque par pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = la bande dans le canal R du produit, bit 1 = G, bit 2 = B) ; la balise XMP `Chloros:UnmixClampMask` du produit pointe vers ce masque. Les masques se trouvent**à côté** des dossiers de produits, jamais à l’intérieur de ceux-ci ; ainsi, un dossier de produit tel que `reflectance/` peut être transmis tel quel à un logiciel de photogrammétrie. L’importation propre à Chloros

ignore les masques.
* **Orientation (1.3.0).** Les produits « Preview », « Radiance », « Reflectance » et « Index » d’une caméra autonome sont pivotés selon son réglage « Mirror/Rotation » ; les fichiers « Raw » et « Debayered » restent tels que le capteur les a lus. Les fichiers membres du tableau restent tous au format natif. Chaque fichier enregistre ce paramètre dans son XMP ; ainsi, tout nouveau traitement l’applique de la même manière.
* **À quelle caméra appartient un fichier `.daq` (1.3.0) ?** Lors du traitement, le code `.daq` d’une capture n’est associé qu’aux images provenant des caméras dont le nom figure dans son code `.daq.cameras.json`. Un appareil photo autonome ne disposant pas de son propre capteur de lumière n’emprunte donc plus la lecture d’un autre appareil photo : associez-lui un capteur ou enregistrez-en un dans l’onglet « Capteurs de lumière ». Un enregistrement de capteur de luminosité (ainsi que tout `.daq` ne comportant pas ce fichier, y compris les captures plus anciennes) continue de servir toutes les caméras.

Il s’agit de la même structure que celle que les fichiers `chloros-cli lattice capture` / `array-capture` écrivent dans leur répertoire `-o` — documentée dans la [Référence CLI

§ À quoi ressemble un dossier de captures](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Retraitement des captures en produits calibrés

Les images brutes capturées ainsi que le fichier `.daq` enregistré constituent tout ce dont le pipeline de traitement a besoin — c’est pourquoi Fastest Capture peut être utilisé en toute sécurité pour un travail réel.

* ****Interface graphique** : ajoutez le dossier de captures à un projet ([Ajout de fichiers à un projet](../processing-images-gui/adding-files-to-a-project.md)) et procédez au traitement comme d’habitude.
* ****CLI**: indiquez à `process` l’**emplacement racine des captures** :

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` n’importe normalement que le dossier que vous spécifiez, mais lorsque ce dossier ne contient aucune image et comporte des sous-dossiers, il explore automatiquement la hiérarchie — ainsi, les sous-dossiers de niveau et le dossier `daq/` sont récupérés en une seule fois. Chaque capture est importée sous la forme d’une **seule image**, les autres niveaux y étant associés en tant que modes d’affichage, et non pas sous la forme d’une image par niveau.

Nommer directement un sous-dossier de niveau (par exemple `…/captures/raw/`) fonctionne également, mais laisse de côté le dossier `daq/` — copiez-le à côté lorsque vousdériver un produit radiométrique à partir de `raw/`, sinon la correspondance d’horodatage n’aura aucun élément de référence.

{% hint style="warning" %}
**Le traitement commence toujours à partir de `raw`.** Au sein de chaque capture, l’image brute constitue la source du pipeline ; `debayered`, `radiance`, `reflectance`, et `preview` apparaissent en tant que modes d’affichage mais ne sont jamais renvoyés dans le pipeline — le retraitement d’un produit dérivé réappliquerait les calculs de vignettage, de couleur et de radiance déjà intégrés à ses pixels ; c’est pourquoi Chloros

préfère y renoncer plutôt que de procéder à un double traitement. Les rendus `index/` et `composite/` ne sont jamais traités (il s’agit de sorties, et non de captures). Un dossier « captures » enregistré **sans** importations brutes s’affiche normalement, mais `process` l’ignore et l’indique ; `--input-level {raw,debayered,processed}` est la porte de secours délibérée qui force un point d’entrée. Consultez la [Référence CLI

](../reference/cli-reference.md#what-a-captures-folder-looks-like) pour connaître les messages d’ignorance exacts.
{% endhint %}

Deux autres comportements à connaître lors de la création de scripts de retraitement :

* Une exécution `chloros-cli process` qui a demandé des produits mais n’a écrit **aucun produit d’image échoue de manière visible et se termine avec un code de sortie non nul** — vous n’obtiendrez jamais une exécution vide silencieuse. Les exécutions réussies indiquent le nombre de produits générés. (Une exécution délibérée portant uniquement sur les métadonnées est tout de même considérée comme réussie.)
* Les exportations traitées réimportées n’occupent jamais l’emplacement brut d’une capture — le fichier brut d’origine reste toujours la source du pipeline.

## Équivalents en CLI



Tout ce qui figure sur cette page peut être piloté en mode sans interface graphique. Les modes de capture de l’interface graphique correspondent directement à `chloros-cli lattice array-capture` :

| Interface graphique | CLI

|
| --- | --- |
| Unique | `chloros-cli lattice array-capture` |
| Continu | `array-capture --continuous [--count N] [--duration S]` |
| Par intervalle | `array-capture --interval S [--duration S]` |
| Capture la plus rapide | `array-capture --fastest` |
| Case à cocher « Aligné » | `--aligned / --no-aligned` |
| Cases à cocher de type « Exportation » | `--processing LEVEL` ou `--levels L1,L2,…` (par défaut `all`) |
| Enregistrer une vidéo d’index | `chloros-cli lattice array-record` |
| Enregistrer une rafale brute / Créer une vidéo | `chloros-cli lattice array-burst` / `array-build-video` |

Les tables d’indicateurs complètes, l’option de capture stabilisée par Smart-AE (`--smart`) et le modèle à débit soutenu sont décrits dans la section [Référence CLI

§ Modes de capture, enregistreurs et retraitement hors ligne](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
