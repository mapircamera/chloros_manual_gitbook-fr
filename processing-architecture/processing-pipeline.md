# Pipeline de traitement

ChlorosLa version 1.2.0 utilise un pipeline de traitement à 4 threads qui fonctionne comme une chaîne de montage par étapes. Chaque thread gère une phase distincte du flux de travail, ce qui permet à plusieurs images d&#x27;être en cours de traitement à différentes étapes simultanément.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architecture du pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Chaque image passe successivement par les quatre threads. Grâce au traitement multithread de Chloros+, plusieurs images occupent simultanément différents threads : tandis que le thread 3 traite une image, le thread 1 peut détecter la suivante, le thread 2 en calibrer une autre et le thread 4 enregistrer une image finalisée sur le disque.

La progression est signalée par thread et diffusée via Server-Sent Events (le backend les publie sur `/api/events`). Dans l’affichage en temps réel de la progression de l’CLI, les quatre étapes sont intitulées **Détection, Analyse, Traitement, Exportation**.***

## Détails des threads

### Thread 1 : Détection

**Objectif** : Charger les images et détecter les cibles d&#x27;étalonnage.

* Lit les fichiers image depuis le disque — paires Survey3 `.raw`+`.jpg`, captures LATTICE `.tif`/`.tiff`, et `.dng`
* Extrait les métadonnées EXIF (GPS, modèle d’appareil photo, horodatage, exposition)
* Détecte les cibles d&#x27;étalonnage : géométries de cibles marquées ArUco pour les captures LATTICE, et le panneau de détection classique pour les photos de cibles d&#x27;étalonnage d&#x27;Survey3
* Résultats : données d&#x27;image + métadonnées + résultats de détection des cibles

Il s&#x27;agit principalement d&#x27;un thread lié aux E/S et à l&#x27;utilisation du processeur.

### Thread 2 : Étalonnage

**Objectif** : calculer les paramètres d’étalonnage à partir des cibles détectées.

* Calcule les coefficients d&#x27;étalonnage de réflectance à partir des images des cibles
* Calcule les paramètres de correction du vignetage
* Détermine les courbes d&#x27;étalonnage par bande
* Sorties : paramètres d&#x27;étalonnage pour chaque image

Un thread de calcul lié au processeur (CPU). Le thread 3 attend son achèvement lorsque l’étalonnage de réflectance est activé, afin que ses coefficients soient prêts avant le traitement de toute image.

### Thread 3 : Traitement (GPU)

**Objectif**: Appliquer les corrections et calculer les indices de végétation.**Il s’agit du thread le plus gourmand en ressources de calcul.*** **Débayérisation** : convertit les données RAW Bayer en images multicanaux
  * Standard (rapide, qualité moyenne) — par défaut, `--debayer standard`
  * Texture Aware (lent, qualité optimale) — Chloros+ uniquement, `--debayer texture-aware`, utilise un modèle de débruitage basé sur l’IA/ML
  * Les captures LATTICE mono (M3M) sont monobandes : les étapes de dématriçage et de balance des blancs sont ignorées pour celles-ci (avec un message de journal d&#x27;une ligne), tandis que toutes les images M3C/Bayer de la même série continuent de bénéficier de ces étapes
* **Correction du vignettage** : applique une correction du vignettage de l&#x27;objectif sur l&#x27;ensemble de l&#x27;image
* **Étalonnage de la réflectance** : applique des coefficients d&#x27;étalonnage pour convertir les valeurs en réflectance
* **Calcul des indices** : calcule les indices de végétation (NDVI, NDRE, GNDVI, …)
* Résultats : données d&#x27;image traitées prêtes à être exportées

C’est ce thread qui tire le meilleur parti de l’accélération GPU, et c’est celui que [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimise.

### Thread 4 : Exportation

**Objectif** : enregistrer les images traitées sur le disque.

* Écrit les fichiers de sortie au format sélectionné — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Intègre des métadonnées dans les fichiers de sortie (GPS, horodatages, paramètres de traitement)
* Organise les fichiers de sortie dans le dossier du projet sous la forme `<camera>/<format>/<Product>_Images/` — par exemple `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Les fichiers exportés conservent le nom du fichier source ; le dossier identifie le produit.**
* Pour les captures LATTICE, une image source peut donner lieu à plusieurs produits (débayérisé, aperçu, radiance, réflectance, indice), chacun dans son propre dossier de produit
* Résultats : fichiers finaux sur le disque

Il s’agit principalement d’un thread limité par les E/S — le stockage sur SSD améliore sensiblement ses performances.

***

## En coulisses : les exécuteurs

Au sein du thread 3, le traitement par image est parallélisé à l’aide de l’exécuteur standard `concurrent.futures` d’Python :

* **Les stratégies GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) utilisent une méthode**spawn** `ProcessPoolExecutor` — chaque thread de travail est un processus distinct disposant de son propre contexte CUDA (`fork` hériterait de l’état CUDA initialisé du parent et corromprait les threads enfants)
* **`CPU_PARALLEL`** utilise un `ThreadPoolExecutor` — NumPy et OpenCV libèrent le GIL, les threads suffisent donc
* Les appareils Jetson disposant de 8 Go ou moins de RAM partagée ignorent complètement l’exécuteur et traitent les tâches en interne, de manière séquentielle
* Texture Aware sur un GPU doté de moins de 7 Go de VRAM s’exécute également de manière séquentielle — le modèle de débruitage ne peut pas s’adapter plus d’une fois

Chlorosn’utilise aucun framework distribué tiers (tel que Ray). Consultez [Adaptation dynamique du calcul](dynamic-compute-adaptation.md) pour savoir comment la stratégie et le nombre de workers sont choisis.

***

## Traitement séquentiel vs traitement en pipeline

### Mode libre (séquentiel)

Dans la version gratuite d’Chloros, les images sont traitées **une par une**, de manière séquentielle, à travers les quatre étapes suivantes :

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

L’interface graphique affiche une barre de progression simplifiée en mode gratuit ; ses phases séquentielles sont indiquées comme **Détection de cibles**puis**Traitement**.

### Mode « Chloros» (en pipeline)

Avec une licence « Chloros», les quatre threads fonctionnent **en parallèle** sur différentes images :

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

La barre de progression de l’interface graphique affiche les quatre étapes ; survolez-la pour voir la progression par thread. Dans l’CLI, ces mêmes quatre étapes s’affichent en direct sous les noms **Détection, Analyse, Traitement, Exportation**.

{% hint style="info" %}
**Une étiquette, deux noms.** L’CLI appelle l’étape 3 _Traitement_. Le flux de progression en mode premium du backend — celui que la barre de progression de l’interface graphique affiche — nomme cette même étape _Calibrage_. Il s’agit du même thread effectuant le même travail (Thread 3 : débayérisation, corrections, indices).
{% endhint %}

{% hint style="success" %}
**Le traitement en pipeline avec Chloros+** peut être 3 à 5 fois plus rapide que le traitement séquentiel, en fonction de votre matériel et de la taille de votre ensemble de données. Le gain de vitesse est maximal sur les systèmes équipés de GPU et de SSD rapides.
{% endhint %}

***

## Fil 4 : progression de l’exportation

Le fil d’exportation dispose de son propre suivi de progression, que vous pouvez interroger séparément :

**CLI :**

```bash
chloros-cli export-status
```

**SDK :**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Le traitement est terminé lorsque le thread 4 atteint 100 %.

{% hint style="info" %}
**Une exécution qui n’écrit aucune image est considérée comme un échec.**En cas de réussite, `chloros-cli process` indique le nombre de produits d’image qu’il a écrits (`Image products written: N`). Si des produits ont été demandés et qu’**aucun**n’a été écrit — uniquement `project.json` et `calibration_data.json` —, l’CLIe affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie différent de zéro**, en indiquant le nom du dossier du projet et les causes habituelles (le dossier d’entrée n’a pas été reconnu comme une capture — vérifier la configuration et `--input-level` — ou tous les produits demandés étaient inapplicables à ces caméras). Les scripts peuvent s’appuyer sur le code de sortie.
{% endhint %}

***

## Lien avec l’adaptation dynamique du calcul

[L’adaptation dynamique du calcul](dynamic-compute-adaptation.md) affecte principalement le **thread 3 (traitement)** :

* **`GPU_PARALLEL`** : le thread 3 traite simultanément plusieurs images via le GPU à l’aide du pipeline `fused_gpu`
* **`GPU_SINGLE`** : le thread 3 sérialise l’accès au GPU à l’aide d’un sémaphore tandis que les processus de travail superposent les E/S, en utilisant le pipeline `fused_gpu` ou le pipeline `tiled_gpu`, plus économe en mémoire
* **`CPU_PARALLEL`** : le thread 3 utilise un traitement basé sur le CPU avec un parallélisme multithread

L&#x27;allocation de mémoire GPU du thread 3 augmente également à mesure que les threads 1 et 2 s&#x27;achèvent — voir [Allocation dynamique de mémoire GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Étapes suivantes

* [Adaptation dynamique des calculs](dynamic-compute-adaptation.md) — Comment Chloros sélectionne la stratégie optimale pour votre matériel
* [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportement du pipeline spécifique à la plateforme sur Jetson
* [Surveillance du traitement](../processing-images-gui/monitoring-the-processing.md) — Suivi de la progression via l’interface graphique
* [Référence CLI](../reference/cli-reference.md) — `process`, `export-status`, codes de sortie et structure de sortie
