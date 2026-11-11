# Flux de travail liés à la réflectance

Un capteur de lumière DAQ convertit l’imagerie radiométrique en réflectance. Il existe deux flux de travail distincts :

1. **Capteur unique** — un capteur DAQ mesure l’irradiance descendante pendant qu’une caméra effectue la prise de vue, et Chloros divise la radiance de la caméra par cette valeur de référence.
2. **Double capteur** — deux capteurs DAQ, l’un orienté vers le ciel et l’autre vers un objet, produisent une courbe de réflectance spectrale en temps réel sans intervention d’une caméra.

## Capteur unique + caméra (référence descendante)

Le DAQ fait office de capteur de lumière descendante (DLS) : la caméra mesure la radiance ascendante **L**(W/m²/sr/nm), le DAQ mesure l’irradiance descendante**E** (W/m²/nm), et Chloros calcule la réflectance par bande selon la formule suivante :

> ρ = π · L / E

La lecture du DAQ est toujours **synchronisée en temps réel avec l’exposition** — c’est pourquoi le DAQ et les caméras partagent une horloge synchronisée via PTP (voir [Mise en réseau et synchronisation horaire du DAQ-E](ethernet-ptp.md)). Ajustez le cap cosinus Sunshine pour les travaux en extérieur et déclarez-le correctement ; la déclaration du cap met directement E à l’échelle (voir [Profils de cap et plage calibrée](caps-and-range.md)). Pour les travaux quantitatifs, gardez à l’esprit la caractéristique de l’instrument : l’irradiance quantitative est calculée à partir d’une moyenne de mesures prises sur au moins 15 s.

### Capture en direct

Associez le DAQ à une caméra dans l’onglet « Cameras » : le panneau de paramètres de chaque caméra comporte un menu déroulant **Light Sensor** répertoriant tous les DAQ connectés (DAQ-U/M/E) depuis l’onglet « Light Sensors » ; pour un réseau synchronisé, la sélection d’un capteur de lumière à l’échelle du réseau se répercute sur chaque élément (les caméras individuelles peuvent toutefois passer outre). Une fois associé, les spectres du capteur alimentent l’emplacement DLS de la caméra et les exportations de réflectance sont divisées par la mesure correspondante. Dans un réseau, un élément disposant de son propre capteur utilise sa propre mesure — pour ses bandes dans le composite en direct, ses exportations et son exposition automatique — tandis que les autres éléments utilisent celles du réseau.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Deux comportements à connaître :

* **Aucun DAQ associé → la réflectance est refusée, et non simulée.** *Chloros
* rejette le produit de réflectance et enregistre la raison du rejet plutôt que de renvoyer silencieusement un produit de qualité inférieure.
* **La mesure utilisée est conservée.** Pour chaque image de réflectance, la mesure DAQ effectivement appliquée est enregistrée sous la forme d’un fichier `.daq` dans le dossier `daq/` des captures, ce qui permet de retraiter la capture ultérieurement ([Enregistrement et format .daq](recording.md)). Un fichier `.daq.cameras.json` situé à côté identifie les caméras qu’il décrit, et le retraitement ne le fait correspondre qu’aux images de ces caméras.

### Traitement des images enregistrées

Pour le traitement post-vol, enregistrez un fichier `.daq` pendant la session et conservez-le avec les images — le pipeline résout automatiquement les données descendantes dont l’horodatage correspond, en récupérant tout étalonnage d’usine manquant depuis le cloud d’MAPIR lors de la première utilisation. Les enregistrements de l’interface graphique sont automatiquement ajoutés au projet ouvert lorsqu’ils s’arrêtent.

La référence de réflectance est sélectionnable au moment du traitement — `--reflectance-source` sur `chloros-cli process`, ou le paramètre de source de réflectance dans les « Paramètres du projet » de l’interface graphique :

| Valeur | Comportement |
| --- | --- |
| `auto` (par défaut) | Une cible d’étalonnage dans le cadre ayant passé le contrôle qualité (QA) sert de référence absolue ; le rayonnement descendant DAQ (ρ = π·L/E) est utilisé à défaut. Une cible ayant passé le contrôle qualité avec un indicateur d’erreur (un panneau manquant ou une bande défectueuse) n’est pas automatiquement privilégiée : les chaînes « cible », « cible + DAQ » et « DAQ uniquement » sont testées dans l’ordre de leur score mesuré sur des panneaux non utilisés. |
| `daq` | DAQ faisant autorité |
| `target` | Cible stricte « in-frame » ; pas de substitution par le DAQ |

Voir [Cibles d’étalonnage](../calibration-targets.md) pour les workflows des cibles et le [chapitre LATTICE](../lattice/README.md) ainsi que la [référence CLI](../reference/cli-reference.md) pour le pipeline de traitement complet. Lors de la lecture des pixels de réflectance exportés, utilisez l’échelle indiquée (LATTICE : 32768 = ρ 1,0, XMP `Chloros:PixelScale` ; Survey3 : 65535) — voir [Formats d’images de sortie](../output-image-formats.md).

### Bandes situées en dehors de la plage étalonnée du DAQ

La plage étalonnée radiométriquement du DAQ est d’environ 374–974 nm. Chloros-refuse la réflectance issue du DAQ pour toute bande de caméra dont moins de la moitié de la pondération spectrale se situe dans cette plage, en indiquant le motif de rejet `dls-uncalibrated-band-<nm>`. Parmi les références commercialisées, cela ne concerne que la F988 : la réflectance de la F988 est étalonnée à l’aide d’un panneau de réflectance intégré à la scène ; la bande se situant au-delà de la plage étalonnée du capteur de lumière du DAQ, Chloros utilise votre dernière capture du panneau et la conserve entre deux observations du panneau. Si une caméra F988 fonctionne en mode DAQ uniquement, Chloros refuse la réflectance basée sur le DAQ pour cette bande avec le motif de saut `dls-uncalibrated-band-988` — le workflow avec panneau est la méthode prise en charge.

## Double capteur (lumière ambiante + objet)

Deux capteurs DAQ — n’importe quelle paire, sur n’importe quel support — fournissent un spectre de réflectance en temps réel sans caméra : un capteur est orienté vers le ciel (**Source de lumière ambiante**), l’autre vers le sujet (**Scanner d’objet**), et Chloros effectue le calcul par longueur d’onde :

> R(λ) = objet(λ) / ambiante(λ)

(valeur nulle lorsque l’ambiance est ≤ 0).

### Dans l’interface graphique

Une fois les deux capteurs connectés dans l’onglet « Capteurs de lumière », ouvrez la fenêtre contextuelle d’ajout de capteur (bouton « + » sur une tuile de graphique en vue en grille) et sélectionnez **Combiner ambiante + objet**. Sélectionnez les deux capteurs dans les menus déroulants**Source de lumière ambiante**et**Scanner d’objet**, puis cliquez sur**Créer**. Le groupe apparaît sous la forme d’un graphique distinct et d’une ligne dans la barre latérale, accompagnée d’un badge vert**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Sous le graphique de réflectance (vue en liste), un **tableau d’indices de végétation** en temps réel calcule les indices à partir de la courbe en utilisant les centres de bande à 450 nm (bleu), 550 nm (vert), 670 nm (rouge) et 800 nm (NIR). Les indices basés sur des rapports qui annulent l’échelle absolue (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) sont toujours affichés ; les indices nécessitant une réflectance absolue (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) n’apparaissent que lorsque les deux capteurs sont des modèles calibrés en puissance.

### Apparente vs relative — la règle d’étiquetage

Chloros étiquette la sortie du double capteur en fonction de ce que la paire de capteurs peut réellement revendiquer :

| Paire de capteurs | Étiquette |
| --- | --- |
| Les deux capteurs étalonnés — ensemble d’étalonnage d’usine chargé | **Réflectance apparente** |
| L’un des capteurs n’est pas étalonné | **Réflectance relative** |

Les trois modèles sont radiométriques : une fois que le jeu d’étalonnage d’usine d’un capteur est chargé, ses spectres sont exprimés en W/m²/nm absolus ; ainsi, une paire de capteurs étalonnés donne lieu à une réflectance apparente absolue — ce n’est pas la fonction de transport qui la détermine. Un capteur continuant à transmettre des comptages bruts (aucun ensemble accessible) réduit le résultat à une courbe relative (la forme spectrale reste valable). Les deux capteurs doivent comporter des limites correctement déclarées ([Profils de limites et plage étalonnée](caps-and-range.md)).

### Depuis Python


Il n’existe pas d’appel dédié aux deux capteurs dans l’interface SDK : ouvrez deux sessions avec `chloros_sdk.connect_daq_sensor()` et calculez vous-même le rapport entre leurs spectres `latest()`, en appliquant la même convention de nommage. (Un outil d’enregistrement à double capteur existe également dans l’interface matérielle directe interne d’MAPIR, répertorié dans la [Référence CLI](../reference/cli-reference.md) par souci d’exhaustivité — il ne fait pas partie de la CLI fournie ; le workflow via l’interface graphique ci-dessus est la méthode prise en charge.)
