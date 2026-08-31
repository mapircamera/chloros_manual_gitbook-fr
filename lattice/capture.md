# Paramètres et modes de capture

La capture dans l’onglet « Caméras » s’effectue à l’aide d’un bouton rouge **Capturer tout**et d’un volet**Paramètres de capture** qui définit le résultat de ce bouton : quelles caméras participent, quels types d’exportation chaque caméra enregistre, et si l’obturateur se déclenche une seule fois, en continu ou à intervalles réguliers. Cette page décrit l’ensemble du processus : la configuration, la capture proprement dite, l’emplacement des fichiers sur le disque et la manière de les retraiter ultérieurement pour obtenir des produits calibrés. Les commandes relatives aux caméras et aux réseaux de caméras se trouvent dans [Paramètres des caméras](camera-settings.md).

{% hint style="info" %}
**Les captures nécessitent l’ouverture d’un projet.** Les boutons « Capture All » et l’icône en forme de roue dentée des paramètres de capture sont désactivés tant qu’aucun projet n’est ouvert (« Créez ou ouvrez un projet pour enregistrer les captures »). Chaque capture est enregistrée dans le dossier du projet sous `captures/`.
{% endhint %}

## Le volet « Paramètres de capture »

Ouvrez-le à l’aide de la **roue dentée située à côté de « Capture tout »**dans la liste des caméras de la barre latérale, ou à l’aide du bouton**« Ouvrir les paramètres de capture… »** situé en bas de n’importe quel volet de paramètres propre à une caméra. L’en-tête indique « Paramètres de capture » et comporte un bouton ← de retour.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Vos sélections ici — caméras incluses, cases à cocher par type et mode de capture — sont enregistrées **par projet** et restaurées lorsque vous le rouvrez.

### Modes de capture

Trois boutons de mode en haut du volet :

| Mode | Fonction | Sous-paramètres (par défaut) |
| --- | --- | --- |
| **Unique** *(par défaut)* | Une seule capture sur toutes les caméras sélectionnées. | — |
| **Continu**| Captures successives jusqu’à une condition d’arrêt. | Arrêt par**nombre de captures** (par défaut 1) *ou* **durée de capture** (par défaut 10 s ; unités : secondes / minutes / heures / jours). |
| **Intervalle**(time-lapse) | Rafales déclenchées par minuterie. |**Prises de vue / intervalle**(par défaut 1) ·**Toutes les**N unités (par défaut 5 s) ·**Pendant** N unités (par défaut 1 m). |

En mode Continu ou Intervalle, le bouton « Tout capturer » devient un bouton « **Arrêter (N)** » pendant l’exécution, comptant les captures au fur et à mesure qu’elles s’affichent.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Choix des caméras et des types d’exportation

Le texte d’aide du volet résume bien la situation : choisissez les caméras et les types d’exportation que la fonction « Capture Tout » doit générer — tout est activé par défaut, et les choix sont enregistrés avec ce projet.

* Les boutons **Tout sélectionner / Ne rien sélectionner** activent ou désactivent simultanément la case à cocher d’inclusion de chaque caméra.
* **Commutateurs de type d’exportation groupée**(deux rangées de boutons) :**Tout en brut / Tout débayérisé / Tout en aperçu / Tout en radiance / Tout en réflectance / Tout en indice**. Chacune présente trois états de couleur : vert ✓ = activé pour toutes les caméras qui le prennent en charge, orange – = activé pour certaines, gris = aucune. Un bouton est désactivé lorsqu’aucune caméra connectée ne prend en charge ce type. Tous sont grisés lorsque « Capture la plus rapide » est activé.
* **Lignes par caméra** : une case à cocher « Inclure », ainsi qu’une liste déroulante (▸/▾) des types d’exportation applicables à cette caméra, accompagnés de cases à cocher individuelles. La ligne affiche un compteur de types activés, par exemple « 4/6 ».

### Types d’exportation et caméras compatibles

Il existe six types d’exportation : **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Seuls ceux qui sont applicables apparaissent dans la ligne de chaque caméra :

| Type d’exportation | Contenu | RGB (FRGB) | Multispectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaïque de Bayer (mono : bande unique) provenant directement du capteur | ✓ | ✓ | ✓ |
| **Démosaïqué** | Démosaïquage linéaire (mono : échelle de gris à 1 canal) | ✓ | ✓ | ✓ |
| **Aperçu** | Chaîne de traitement complète (balance des blancs + gamma selon le profil de la caméra ; multispectral : étirement en fausses couleurs) | ✓ | ✓ | ✓ |
| **Radiance** | float32 W/m²/sr/nm via la chaîne radiométrique complète | — (non disponible) | ✓ | ✓ |
| **Réflectance** | uint16 ρ (32768 = 1,0) | — (non disponible) | ✓ — affichée uniquement lorsque la caméra dispose d’un capteur de lumière DAQ (propre ou hérité de son réseau) | identique au mode multispectral |
| **Indice** | Rendu de l’indice de végétation (LUT) | — | ✓ — nécessite une expression d’indice activée et non vide sur la caméra ; non disponible pour les membres d’un réseau combiné (le réseau dispose d’un indice partagé) | — (un indice nécessite ≥ 2 bandes ; voir [Caméras mono et indices de végétation](mono-indices.md)) |

La radiance et la réflectance ne sont jamais proposées pour les caméras RGB — la radiance par pixel Bayer n’a pas de sens pour un capteur photométrique à large bande.

### Capture la plus rapide

Le bouton **⚡ Capture la plus rapide — format brut uniquement**(orange lorsqu’il est activé) remplace toutes les sélections d’exportation par**format brut uniquement** — plus un composite à indice combiné gratuit pour les réseaux — afin que l’image soit enregistrée aussi rapidement que possible : les calculs de radiance, de réflectance et d’affichage sont entièrement ignorés au moment de la capture.

{% hint style="info" %}
**Un fichier `.daq` est tout de même enregistré.** Lorsqu’un capteur de lumière est attribué, la « Capture la plus rapide » enregistre toujours la mesure DAQ de la lumière descendante à côté des images brutes — ce qui permet de générer ultérieurement les produits de radiance, de réflectance et d’indice par un nouveau traitement (voir [Nouveau traitement des captures](#re-processing-captures-into-calibrated-products)). Fastest Capture n’altère pas non plus vos sélections par case à cocher : désactivez-le et elles réapparaissent.
{% endhint %}

### Commandes par réseau

Chaque réseau connecté dispose de sa propre fiche de groupe dans le volet :

* **Case à cocher « Include »** (à trois états pour tous les membres) et le nom du réseau avec son mode d’affichage : « (combiné | séparé) ».
* Case à cocher **Aligné** (activée par défaut) : déforme les exportations des éléments selon le profil d’alignement du réseau afin que les exportations soient alignées au pixel près entre les caméras. Les données brutes restent non déformées mais contiennent la transformation dans leurs métadonnées. (Le profil lui-même est calculé dans le [volet des paramètres de la matrice](camera-settings.md#alignment-co-registration-combined-only).)
* Les lignes des caméras membres sont imbriquées à l’intérieur de la carte.

La carte de matrice héberge également deux enregistreurs. Considérez-les comme **« surveillance » vs « analyse »** :

| Enregistreur | Niveau | Ce qu’il enregistre |
| --- | --- | --- |
| **● Enregistrer la vidéo d’index / ■ Arrêter l’enregistrement** *(matrices combinées uniquement)* | **Surveillance** | Le composite d’index combiné en direct sous forme de vidéo à 10 images par seconde — 8 bits, résolution de prévisualisation, LUT intégrée. Nécessite un projet ouvert et une vue en direct en streaming. Affiche le nombre d’images et le temps écoulé pendant l’enregistrement. |
| **⦿ Enregistrer une rafale brute / ■ Arrêter la rafale brute** *(tout réseau)* | **Analyse**| Images Bayer brutes à la fréquence de capture en direct (sans traitement), accompagnées d’un manifeste par image et des mesures `.daq`, enregistrées au format `captures/bursts/`. Après une rafale, un bouton**Créer une vidéo** apparaît : il retraite la rafale hors ligne pour obtenir une vidéo calibrée — indice combiné et/ou radiance / réflectance / indice par caméra — ainsi que des fichiers TIFF en option. La création de l’indice combiné démarre automatiquement lorsque vous arrêtez la rafale. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

Le flux

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

« Capture All » Appuyez sur **Capture All** dans la liste des caméras de la barre latérale :

1. Chaque caméra incluse, visible et non mise en pause effectue une capture selon les types d’exportation sélectionnés. **Les réseaux de caméras se déclenchent de manière synchronisée** (un seul groupe synchronisé pour tous les membres — voir [Réseaux multicaméras](arrays.md)) ; les caméras autonomes capturent individuellement.
2. Les caméras masquées (œil) ou en pause sont ignorées. Un réseau n’est entièrement bloqué que lorsque *tous* ses membres sont masqués ou en pause.
3. Lorsqu’un capteur de lumière est affecté, la mesure DAQ correspondante de l’irradiance descendante est enregistrée sous forme de fichier `.daq` avec les images — même pour les enregistrements en format brut uniquement — afin que des produits radiométriques puissent toujours être dérivés ultérieurement.
4. Le bouton affiche la progression en temps réel — « Capture en cours… terminée/total » — et, en mode Continu/Intervalle, devient **Arrêt (N)**. Chaque élément de capture dispose d’un délai d’expiration de 300 s.
5. À la fin du passage, un message s’affiche brièvement indiquant **« N fichiers enregistrés »**ou**« N fichiers enregistrés, F échecs »**, ainsi que « (S masqués/mis en pause/ignorés) » lorsque des caméras ont été ignorées.

## Emplacement des captures

Les captures sont enregistrées dans le projet ouvert sous `<project>/captures/`. Chaque type d’exportation est placé dans son **propre sous-dossier**, de sorte qu’une capture à plusieurs niveaux ne mélange jamais les types :

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` correspond à l&#x27;horodatage de la capture et `<serial>` au numéro de série de la caméra. Les captures autonomes sont nommées `capture_<ts>_SN<serial>_<level>` ; les captures en série issues d’un seul déclencheur synchronisé sont nommées `sync_<ts>_SN<serial>_<level>` et **partagent un même horodatage pour toutes les caméras du groupe** (le suffixe « niveau » est supprimé lorsqu&#x27;une caméra n&#x27;enregistre qu&#x27;un seul niveau).
* **Une particularité à noter :** le niveau d’affichage est stocké dans un dossier nommé `preview/` tandis que les fichiers conservent `_display` dans leur nom — le dossier et le suffixe diffèrent uniquement pour ce niveau.
* Les niveaux inconnus sont placés par défaut dans un dossier portant leur propre nom ; si un sous-dossier ne peut pas être créé, le fichier est enregistré à la racine du répertoire « captures » plutôt que d’être perdu.
* Les fichiers TIFF de capture sont compressés sans perte (DEFLATE) par défaut et contiennent l&#x27;intégralité de leurs métadonnées d&#x27;étalonnage et de traitement **dans le fichier XMP** — les captures sont autodescriptives, sans aucun fichier sidecar autre que le fichier `.daq`.

Il s’agit de la même structure que celle utilisée par `chloros-cli lattice capture` / `array-capture` pour écrire dans leur répertoire `-o` — documentée dans la [Référence CLI § À quoi ressemble un dossier de captures](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Retraitement des captures en produits calibrés

Les images brutes capturées ainsi que le fichier `.daq` enregistré constituent tout ce dont la chaîne de traitement a besoin — c’est pourquoi Fastest Capture peut être utilisé en toute sécurité pour un travail réel.

* **Interface graphique** : ajoutez le dossier des captures à un projet ([Ajouter des fichiers à un projet](../processing-images-gui/adding-files-to-a-project.md)) et procédez au traitement comme d&#x27;habitude.
* **CLI**: pointez `process` vers la**racine des captures** :

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` n&#x27;importe normalement que le dossier que vous nommez, mais lorsque ce dossier ne contient aucune image et comporte des sous-dossiers, il explore automatiquement la structure — ainsi, les sous-dossiers de niveau et les fichiers de la racine `.daq` sont récupérés en une seule fois. Chaque capture est importée sous forme d’une **seule image**, les autres niveaux y étant associés en tant que modes d’affichage, et non sous la forme d’une image par niveau.

Nommer directement un sous-dossier de niveau (par exemple `…/captures/raw/`) fonctionne également, mais cela laisse de côté les fichiers racine `.daq` — copiez-les en même temps lorsque vous dérivez à nouveau un produit radiométrique à partir de `raw/`, sinon la correspondance d’horodatage n’aura aucun élément de référence.

{% hint style="warning" %}
**Le traitement commence toujours à partir de `raw`.**Au sein de chaque capture, l&#x27;image brute constitue la source du pipeline ; `debayered`, `radiance`, `reflectance` et `preview` sont générés en tant que modes d’affichage, mais ne sont jamais réinjectés dans le pipeline — le retraitement d&#x27;un produit dérivé réappliquerait les calculs de vignettage, de couleur et de radiance déjà intégrés à ses pixels ; c&#x27;est pourquoi Chloros est rejeté plutôt que de subir un double traitement. Les rendus `index/` et `composite/` ne sont jamais traités (il s’agit de sorties, et non de captures). Un dossier « captures » enregistré**sans** importations brutes s’affiche normalement, mais `process` l’ignore et le signale ; `--input-level {raw,debayered,processed}` est la porte de secours délibérée qui force un point d’entrée. Consultez la [Référence CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) pour connaître les messages d’ignorance exacts.
{% endhint %}

Deux autres comportements à connaître lors de la création de scripts de retraitement :

* Une exécution `chloros-cli process` qui a demandé des produits mais n’a écrit **aucun produit d’image échoue de manière visible et se termine avec un code de sortie différent de zéro** — vous n’obtiendrez jamais d’exécution vide silencieuse. Les exécutions réussies indiquent le nombre de produits générés. (Une exécution délibérée portant uniquement sur les métadonnées est tout de même considérée comme réussie.)
* Les exportations traitées puis réimportées n’occupent jamais l’emplacement brut d’une capture — le fichier brut d’origine reste toujours la source du pipeline.

## Équivalents de CLI

Tout ce qui figure sur cette page peut être exécuté en mode sans interface graphique. Les modes de capture de l’interface graphique correspondent directement à `chloros-cli lattice array-capture` :

| Interface graphique | CLI |
| --- | --- |
| Unique | `chloros-cli lattice array-capture` |
| Continu | `array-capture --continuous [--count N] [--duration S]` |
| Par intervalle | `array-capture --interval S [--duration S]` |
| Capture la plus rapide | `array-capture --fastest` |
| Case à cocher alignée | `--aligned / --no-aligned` |
| Cases à cocher de type exportation | `--processing LEVEL` ou `--levels L1,L2,…` (par défaut `all`) |
| Enregistrer une vidéo d’index | `chloros-cli lattice array-record` |
| Enregistrer une rafale brute / Créer une vidéo | `chloros-cli lattice array-burst` / `array-build-video` |

Les tables d’indicateurs complètes, l’option de capture stabilisée par Smart-AE (`--smart`) et le modèle à débit constant sont décrits dans la section [CLI Référence § Modes de capture, enregistreurs et retraitement hors ligne](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
