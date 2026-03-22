# Pipeline de traitement

Chloros 1.1.0 utilise un pipeline de traitement à 4 threads qui fonctionne comme une chaîne de montage par étapes. Chaque thread gère une phase distincte du flux de travail de traitement, ce qui permet de traiter simultanément plusieurs images à différentes étapes.

***

## Architecture du pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Chaque image passe par les quatre threads dans l&#x27;ordre. Grâce au traitement multithread de Chloros+, plusieurs images peuvent se trouver simultanément dans différents threads : tandis que le thread 3 traite une image, le thread 1 peut détecter la suivante, le thread 2 peut en calibrer une autre et le thread 4 peut écrire une image précédemment traitée sur le disque.

***

## Détails des threads

### Thread 1 : Détection

**Objectif** : Charger les images et détecter les cibles de calibrage.

* Lit les fichiers image depuis le disque (RAW, JPG)
* Extrait les métadonnées EXIF (GPS, modèle d&#x27;appareil photo, horodatage, exposition)
* Détecte les cibles d&#x27;étalonnage ArUco dans les images de cibles marquées
* Sorties : données d&#x27;image + métadonnées + résultats de détection des cibles

Il s&#x27;agit principalement d&#x27;un thread lié aux E/S et au CPU.

### Thread 2 : Étalonnage

**Objectif** : Calculer les paramètres d&#x27;étalonnage à partir des cibles détectées.

* Calcule les coefficients de calibrage de réflectance à partir des images de cibles
* Calcule les paramètres de correction de vignettage
* Détermine les courbes de calibrage par bande
* Sorties : paramètres de calibrage pour chaque image

Il s&#x27;agit d&#x27;un thread de calcul lié au CPU.

### Thread 3 : Traitement (GPU)

**Objectif**: Appliquer les corrections et calculer les indices de végétation.**C&#x27;est le thread le plus gourmand en calcul.*** **Débayérisation** : Convertit les données brutes au format Bayer en images multicanaux
  * Standard (rapide, qualité moyenne) — par défaut
  * Texture Aware (lent, qualité optimale) — Chloros+ uniquement, utilise le débruitage par IA/ML
* **Correction du vignettage** : Applique la correction du vignettage de l&#x27;objectif à l&#x27;ensemble de l&#x27;image
* **Calibrage de la réflectance** : applique des coefficients de calibrage pour convertir les valeurs en réflectance
* **Calcul des indices** : calcule les indices de végétation (NDVI, NDRE, GNDVI, etc.)
* Sorties : données d&#x27;image traitées prêtes à être exportées

Ce thread tire le meilleur parti de l&#x27;accélération GPU. Le système [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimise principalement le comportement de ce thread.

### Thread 4 : Exportation

**Objectif** : Enregistrer les images traitées sur le disque.

* Enregistre les fichiers de sortie au format sélectionné (TIFF 16 bits, TIFF 32 bits %, PNG, JPG)
* Intègre les métadonnées EXIF dans les fichiers de sortie (GPS, horodatages, paramètres de traitement)
* Organise les fichiers de sortie dans des sous-dossiers par modèle d&#x27;appareil photo
* Sorties : fichiers finaux sur le disque

Il s&#x27;agit principalement d&#x27;un thread lié aux E/S. Le stockage SSD améliore considérablement les performances du Thread 4.

***

## Traitement séquentiel vs traitement en pipeline

### Mode gratuit (séquentiel)

Dans la version gratuite de Chloros, les images sont traitées **une à la fois**, de manière séquentielle, à travers les quatre étapes :

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

La barre de progression de l&#x27;interface graphique affiche deux étapes : Détection de la cible et Traitement.

### Mode Chloros+ (en pipeline)

Avec une licence Chloros+, les quatre threads fonctionnent **simultanément** sur différentes images :

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

La barre de progression de l&#x27;interface graphique affiche 4 étapes : Détection, Analyse, Calibrage, Exportation. Survolez la barre de progression pour voir la progression par thread.

{% hint style="success" %}
**Le traitement en pipeline avec Chloros+** peut être 3 à 5 fois plus rapide que le traitement séquentiel, en fonction de votre matériel et de la taille de votre ensemble de données. Le gain de vitesse est maximal sur les systèmes équipés de GPU et de SSD rapides.
{% endhint %}

***

## Progression de l&#x27;exportation du thread 4

Dans Chloros 1.1.0, le thread d&#x27;exportation (thread 4) dispose de son propre suivi de progression dédié. Vous pouvez surveiller la progression de l&#x27;exportation séparément :**CLI :**
```bash
chloros-cli export-status
```

**SDK :**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Le traitement est terminé lorsque le Thread 4 atteint 100 %.

***

## Relation avec l&#x27;adaptation dynamique du calcul

Le système [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) affecte principalement le **thread 3 (traitement)** :

* Stratégie **`GPU_PARALLEL`** : le thread 3 traite simultanément plusieurs images via le GPU à l&#x27;aide du pipeline `fused_gpu`
* Stratégie **`GPU_SINGLE`** : le thread 3 traite une image à la fois à l&#x27;aide du pipeline `tiled_gpu`, qui optimise l&#x27;utilisation de la mémoire
* Stratégie **`CPU_PARALLEL`** : le thread 3 utilise un traitement basé sur le CPU avec un parallélisme multithread

L&#x27;allocation de mémoire GPU du thread 3 évolue également de manière dynamique à mesure que les threads 1 et 2 terminent leur traitement — voir [Allocation dynamique de mémoire GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Étapes suivantes

* [Adaptation dynamique du calcul](dynamic-compute-adaptation.md) — Comment Chloros sélectionne la stratégie optimale pour votre matériel
* [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportement du pipeline spécifique à la plateforme sur Jetson
* [Surveillance du traitement](../processing-images-gui/monitoring-the-processing.md) — Surveillance de la progression via l&#x27;interface graphique
