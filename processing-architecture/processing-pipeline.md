# Pipeline de traitement

Chloros 1.3.0 utilise un pipeline de traitement à 4 threads qui fonctionne comme une chaîne de montage par étapes. Chaque thread gère une phase distincte du flux de travail, ce qui permet à plusieurs images d'être en cours de traitement à différentes étapes simultanément.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architecture du pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Chaque image passe successivement par les quatre threads. Grâce au traitement multithread d’Chloros+, plusieurs images occupent simultanément différents threads : tandis que le thread 3 traite une image, le thread 1 peut détecter la suivante, le thread 2 en calibrer une autre et le thread 4 enregistrer une image finalisée sur le disque.

La progression est signalée par thread et transmise via des événements Server-Sent Events (le backend les publie sur `/api/events`). Dans l’affichage en temps réel de la progression de l’CLI, les quatre étapes sont intitulées **Détection, Analyse, Traitement, Exportation**.***

## Détails des threads

### Thread 1 : Détection

**Objectif** : Charger les images et détecter les cibles d’étalonnage.

* Lit les fichiers image depuis le disque — paires Survey3 `.raw`+`.jpg`, captures LATTICE `.tif`/`.tiff`, et `.dng`
* Extrait les métadonnées EXIF (GPS, modèle d’appareil photo, horodatages, exposition)
* Détecte les cibles d’étalonnage : géométries de cibles marquées ArUco pour les captures LATTICE, et le détecteur à panneau classique pour les photos de cibles d’étalonnage d’Survey3
* Sorties : données d’image + métadonnées + résultats de détection des cibles

Fil principalement lié aux E/S et à l’utilisation du processeur.

### Thread 2 : Étalonnage

**Objectif** : calculer les paramètres d’étalonnage à partir des cibles détectées.

* Calcule les coefficients d’étalonnage de réflectance à partir des images des cibles
* Calcule les paramètres de correction de vignettage
* Détermine les courbes d’étalonnage par bande
* Sorties : paramètres d’étalonnage pour chaque image

Un thread de calcul dépendant du processeur. Lors d’une exécution Survey3 ou mixte, le thread 3 attend que ce thread ait terminé lorsque l’étalonnage de la réflectance est activé, afin que ses coefficients soient prêts avant le traitement de toute image. Lors d’une exécution LATTICE uniquement, le thread 3 s’exécute pendant la détection : chaque image démarre une fois que son point d’ancrage cible est définitif. Une image prend en compte la cible la plus proche dans le temps, dans un sens comme dans l’autre ; elle attend donc que toutes les images de sa caméra, plus proches dans le temps que sa cible la plus proche actuelle, aient été analysées. Les résultats sont identiques, mais obtenus plus rapidement.

### Fil 3 : Traitement (GPU)

**Objectif**: appliquer les corrections et calculer les indices de végétation.**Il s’agit du thread le plus gourmand en ressources de calcul.*** **Débayérisation** : convertit les données RAW Bayer en images multicanaux
  * Standard (rapide, qualité moyenne) — par défaut, `--debayer standard`
  * Texture Aware (lent, qualité maximale) — Chloros+ uniquement, `--debayer texture-aware`, utilise un modèle de débruitage basé sur l’IA/ML
  * Les captures LATTICE mono (M3M) sont monobandes : les étapes de dématriçage et de balance des blancs sont ignorées pour celles-ci (avec un message de journal d’une ligne), tandis que toutes les images M3C/Bayer du même cycle en bénéficient toujours
* **Correction du vignettage** : applique une correction du vignettage de l'objectif sur l'ensemble de l'image
* **Étalonnage de la réflectance** : applique des coefficients d'étalonnage pour convertir les valeurs en réflectance
* **Calcul des indices** : calcule les indices de végétation (NDVI, NDRE, GNDVI, …)
* Résultats : données d’image traitées prêtes à être exportées

C’est ce thread qui bénéficie le plus de l’accélération GPU, et c’est celui que [l’adaptation dynamique du calcul](dynamic-compute-adaptation.md) optimise.

### Thread 4 : Exportation

**Objectif** : enregistrer les images traitées sur le disque.

* Enregistre les fichiers de sortie au format sélectionné — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Intègre des métadonnées dans les fichiers de sortie (GPS, horodatages, paramètres de traitement) — y compris les balises de traitement `Chloros:*`, qui, depuis la version 1.3.0, sont enregistrées dans tous les modes de traitement, aussi bien pour Survey3 que pour LATTICE
* Organise les fichiers de sortie dans le dossier du projet sous le format `<camera>/<format>/<Product>_Images/` — par exemple `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Les fichiers exportés conservent le nom du fichier source ; le dossier identifie le produit.**
* Pour les acquisitions LATTICE, une image source peut donner lieu à plusieurs produits (débayérisé, aperçu, radiance, réflectance, indice), chacun dans son propre dossier de produit. Les produits d’indice sont calculés à partir du produit de réflectance généré par le traitement (en l’absence de celui-ci, à partir d’un autre produit généré par le traitement, tel que la radiance). Les produits de radiance et de réflectance des caméras multispectrales M3C peuvent également bénéficier d’un masque de limitation par pixel dans `<format>/clamp_masks/` (voir [Formats d’images de sortie](../output-image-formats.md#clamp-masks))
* Sorties : fichiers finaux sur le disque

Il s’agit principalement d’un thread limité par les E/S — le stockage sur SSD améliore sensiblement ses performances.

***

## En coulisses : les exécuteurs

Au sein du thread 3, le travail par image est parallélisé comme suit :

* **Les stratégies GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) utilisent un pool de**processus de travail**, chacun étant un processus système distinct disposant de son propre contexte CUDA. Chloros démarre chaque processus de travail en relançant son propre programme backend installé — et non via la fonction `multiprocessing` de Python, dont le démarrage des processus de travail ne fonctionne pas à l’intérieur de l’application packagée
* **`CPU_PARALLEL`** utilise un pool de threads (la fonction `ThreadPoolExecutor` de Python) — NumPy et OpenCV libèrent le GIL, les threads suffisent donc
* Les appareils Jetson dotés de 8 Go ou moins de mémoire vive partagée ignorent complètement l’exécuteur et traitent les tâches en interne, de manière séquentielle
* Texture Aware sur un GPU disposant de moins de 7 Go de VRAM s’exécute également de manière séquentielle — le modèle de débruitage ne peut pas s’adapter plus d’une fois

Chloros n’utilise aucun framework distribué tiers (tel que Ray). Voir [Adaptation dynamique du calcul](dynamic-compute-adaptation.md) pour savoir comment la stratégie et le nombre de workers sont choisis.

***

## Traitement séquentiel vs traitement en pipeline

### Mode libre (séquentiel)

Dans la version gratuite de Chloros, les images sont traitées **une à la fois**, de manière séquentielle, à travers les quatre étapes :

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

L’interface graphique affiche une barre de progression simplifiée en mode gratuit ; ses phases séquentielles sont indiquées comme **Détection de cibles**puis**Traitement**.

### Mode Chloros+ (en pipeline)

Avec une licence Chloros+, les quatre threads fonctionnent **en parallèle** sur différentes images :

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

La barre de progression de l’interface graphique affiche les quatre étapes ; survolez-la pour voir la progression par thread. Dans l’CLI, ces mêmes quatre étapes s’affichent en temps réel sous les noms **Détection, Analyse, Traitement, Exportation**.

{% hint style="info" %}
**Une étiquette, deux noms.** L’CLI appelle l’étape 3 « _Traitement_ ». Le flux de progression en mode premium du backend — celui que la barre de progression de l’interface graphique affiche — nomme cette même étape « _Calibrage_ ». Il s’agit du même thread effectuant le même travail (Thread 3 : débayérisation, corrections, indices).
{% endhint %}

{% hint style="success" %}
**Le traitement en pipeline avec Chloros+** peut être 3 à 5 fois plus rapide que le traitement séquentiel, en fonction de votre matériel et de la taille de votre ensemble de données. Le gain de vitesse est maximal sur les systèmes équipés de GPU et de SSD rapides.
{% endhint %}

***

## Progression de l’exportation du thread 4

Le thread d’exportation dispose de son propre suivi de progression, que vous pouvez interroger séparément :**Interface CLI :**

```bash
chloros-cli export-status
```

**Interface SDK :**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Le traitement est terminé lorsque le thread 4 atteint 100 %.

{% hint style="info" %}
**Une exécution qui n'écrit aucune image est considérée comme un échec.**En cas de réussite, `chloros-cli process` indique le nombre de produits d'image écrits (`Image products written: N`). Si des produits ont été demandés et qu’**aucun**n’a été écrit — uniquement `project.json` et `calibration_data.json` —, l’CLI affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie différent de zéro**, en indiquant le nom du dossier du projet et les causes habituelles (le dossier d’entrée n’a pas été reconnu comme une capture — vérifiez la configuration et `--input-level` — ou tous les produits demandés n’étaient pas applicables à ces caméras). Les scripts peuvent s’appuyer sur le code de sortie.
{% endhint %}

***

## Lien avec l’adaptation dynamique du calcul

[L’adaptation dynamique du calcul](dynamic-compute-adaptation.md) affecte principalement le **thread 3 (traitement)** :

* **`GPU_PARALLEL`** : le thread 3 traite simultanément plusieurs images via le GPU à l’aide du pipeline `fused_gpu`
* **`GPU_SINGLE`** : le thread 3 sérialise l’accès au GPU à l’aide d’un sémaphore tandis que les processus de travail chevauchent les E/S, en utilisant `fused_gpu` ou le pipeline `tiled_gpu`, plus économe en mémoire
* **`CPU_PARALLEL`** : le thread 3 utilise un traitement basé sur le CPU avec un parallélisme multithread

L'allocation de mémoire GPU du thread 3 augmente également à mesure que les threads 1 et 2 se terminent — voir [Allocation dynamique de mémoire GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Étapes suivantes

* [Adaptation dynamique du calcul](dynamic-compute-adaptation.md) — Comment Chloros sélectionne la stratégie optimale pour votre matériel
* [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportement du pipeline spécifique à la plateforme sur Jetson
* [Surveillance du traitement](../processing-images-gui/monitoring-the-processing.md) — Suivi de la progression via l’interface graphique
* [Référence de l'interface CLI](../reference/cli-reference.md) — `process`, `export-status`, codes de sortie et structure de la sortie
