# Démarrage du traitement

Une fois que vous avez importé vos images, marqué vos cibles d'étalonnage et configuré les paramètres de votre projet, vous êtes prêt à commencer le traitement. Cette page vous guide dans le lancement du pipeline de traitement Chloros.

## Liste de contrôle pour le prétraitement

Avant de cliquer sur le bouton Démarrer, vérifiez que tout est prêt :

* [ ] **Fichiers importés** - Toutes les images apparaissent dans le navigateur de fichiers
* [ ] **Images cibles marquées** - La colonne Cible a été vérifiée pour les images d'étalonnage
* [ ] **Modèles de caméra détectés** - La colonne Modèle de caméra affiche les caméras correctes
* [ ] **Paramètres configurés** - Les paramètres du projet sont examinés et ajustés
* [ ] **Indices sélectionnés** - Les indices multispectraux souhaités sont ajoutés (si nécessaire)
* [Format d'exportation choisi** - Format de sortie approprié pour votre flux de travail

[ PLH:000003]

***

## Démarrage du traitement

### Localiser le bouton de démarrage

Le bouton Démarrer/Lecture se trouve dans la barre d'en-tête supérieure de Chloros :

* Position : Centre supérieur de la fenêtre
* Icône : **Bouton Jouer/Démarrer** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* État : Le bouton est activé (lumineux) lorsqu'il est prêt à être traité

### Cliquer pour démarrer

1. Cliquez sur le **bouton Jouer/Démarrer** dans l'en-tête supérieur
2. Le traitement commence immédiatement
3. Le bouton est désactivé (grisé) pendant le traitement
4. La barre de progression se met à jour et indique l'état d'avancement du traitement

{% hint style="success" %}
**Processing Started**: Once clicked, Chloros automatically handles all processing steps - target detection, debayering, calibration, index calculation, and export.
{% endhint %}

***

## Comprendre les modes de traitement

Chloros fonctionne selon deux modes de traitement différents en fonction de votre licence :

### Mode libre (traitement séquentiel)

**Disponible pour tous les utilisateurs

**Comment ça marche:**

* Traite les images une par une, de manière séquentielle
* Opération à un seul fil
* Utilisation réduite de la mémoire

**La barre de progression indique 2 étapes:**

1. **Détection de cibles** - Recherche de cibles d'étalonnage
2. **Traitement** - Application de l'étalonnage et exportation des images

**Temps de traitement:**

* Beaucoup plus lent que le mode parallèle Chloros+
* Convient aux ensembles de données de petite à moyenne taille (< 200 images)

### Chloros+ Mode (Parallel Processing)

**Requires Chloros+ license**

**How it works:**

* Processes multiple images simultaneously
* Multi-threaded operation (up to 16 parallel workers)
* Utilizes multiple CPU cores
* Optional GPU (CUDA) acceleration with NVIDIA graphics cards

**Progress bar shows 4 stages:**

1. **Detecting** - Finding calibration targets
2. **Analyzing** - Examining image metadata and preparing pipeline
3. **Calibrating** - Applying corrections and calibrations
4. **Exporting** - Saving processed images and indices

**Progress bar interaction:**

* **Hover mouse** over bar to see detailed 4-stage dropdown panel
* **Click** progress bar to freeze the dropdown panel in place
* **Click again** to unfreeze and hide panel

**Processing time:**

* Significantly faster than free mode
* Scales with CPU core count
* GPU acceleration further improves speed

{% hint style="info" %}
**Chloros+ Speed**: Parallel processing can be 5-10x faster than sequential mode for large datasets. A 500-image project that takes 2 hours in free mode may complete in 15-20 minutes with Chloros+.
{% endhint %}

***

## What Happens During Processing

### Stage 1: Target Detection

**What Chloros does:**

* Scans marked target images (or all images if none marked)
* Identifies the 4 calibration panels in each target
* Extracts reflectance values from target panels
* Records target timestamps for calibration scheduling

**Duration:** 1-30 seconds (with marked targets), 5-30+ minutes (unmarked)

### Stage 2: Debayering (RAW Conversion)

**What Chloros does:**

* Converts RAW Bayer pattern data to full RGB images
* Applies high-quality demosaicing algorithm
* Preserves maximum image quality and detail

**Duration:** Varies by image count and CPU speed

### Stage 3: Calibration

**What Chloros does:**

* **Vignette correction**: Removes lens darkening at edges
* **Reflectance calibration**: Normalizes using target reflectance values
* Applies corrections across all bands/channels
* Uses appropriate calibration target for each image based on timestamp

**Duration:** Majority of processing time

### Stage 4: Index Calculation

**What Chloros does:**

* Calculates configured multispectral indices (NDVI, NDRE, etc.)
* Applies band math to calibrated images
* Generates index images for each selected index

**Duration:** A few seconds per image

### Stage 5: Export

**What Chloros does:**

* Saves calibrated images in selected format
* Exports index images with configured LUT colors
* Writes files to camera model subfolders
* Preserves original filenames with suffixes

**Duration:** Varies by export format and file size

***

## Processing Behavior

### Automatic Processing Pipeline

Once started, the entire pipeline runs automatically:

* No user interaction needed
* All configured steps execute in sequence
* Progress updates shown in real-time

### Computer Usage During Processing

**Free Mode:**

* Relatively low CPU usage (single-threaded)
* Computer remains responsive for other tasks
* Safe to minimize Chloros and work in other applications

**Chloros+ Parallel Mode:**

* High CPU usage (multi-threaded, up to 16 cores)
* With GPU acceleration: High GPU usage
* Computer may be less responsive during processing
* Avoid starting other CPU-intensive tasks

{% hint style="warning" %}
**Performance Tip**: For best Chloros+ performance, close other applications and let Chloros use full system resources.
{% endhint %}

### Processing Cannot Be Paused

**Important limitations:**

* Once started, processing cannot be paused
* You can cancel processing, but progress is lost
* Partial results are not saved
* Must restart from beginning if canceled

**Planning tip:** For very large projects, consider processing in batches or using CLI for better control.

***

## Monitoring Your Processing

While processing runs, you can:

* **Watch progress bar** - See overall completion percentage
* **View current stage** - Detect, Analyze, Calibrate, or Export
* **Check log tab** - See detailed processing messages and warnings
* **Preview completed images** - Some export files may appear during processing

For detailed information on monitoring, see [Monitoring the Processing](monitoring-the-processing.md).

***

## Canceling Processing

If you need to stop processing:

### How to Cancel

1. Locate the **Stop/Cancel button** (replaces Start button during processing)
2. Click the Stop button
3. Processing halts immediately
4. Partial results are discarded

### When to Cancel

**Valid reasons to cancel:**

* Realized incorrect settings were used
* Forgot to mark target images
* Wrong images imported
* System running too slow or unresponsive

**After canceling:**

* Review and fix any issues
* Adjust settings as needed
* Restart processing from the beginning
* For the cleanest experience, completely close Chloros and restart

{% hint style="warning" %}
**No Partial Results**: Canceling discards all progress. Chloros does not save partially processed images.
{% endhint %}

***

## Processing Time Estimates

Actual processing time varies greatly based on:

* Number of images
* Image resolution
* RAW vs JPG input format
* Processing mode (Free vs Chloros+)
* CPU speed and core count
* GPU availability (Chloros+ only)
* Number of indices to calculate
* Export format complexity

### Rough Estimates (Chloros+, 12MP images, modern CPU)

| Image Count | Free Mode | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 images   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 images  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 images  | 1-1.5 hrs | 20-30 min      | 10-15 min      |
| 500 images  | 2-3 hrs   | 45-60 min      | 20-30 min      |
| 1000 images | 4-6 hrs   | 1.5-2 hrs      | 40-60 min      |

{% hint style="info" %}
**First Run**: Initial processing may take longer as Chloros builds caches and profiles. Subsequent processing of similar datasets will be faster.
{% endhint %}

***

## Common Issues at Start

### Start Button Disabled (Grayed Out)

**Possible causes:**

* No images imported
* Backend not fully started
* Previous processing still running
* Project not fully loaded

**Solutions:**

1. Wait for backend to fully initialize (check main menu icon)
2. Verify images are imported in File Browser
3. Restart Chloros if button remains disabled
4. Check Debug Log for error messages

### Processing Starts Then Immediately Fails

**Possible causes:**

* No valid images in project
* Corrupted image files
* Insufficient disk space
* Insufficient memory (RAM)

**Solutions:**

1. Check Debug Log <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pour les messages d'erreur)
2. Vérifier l'espace disque disponible
3. Essayer de traiter un plus petit sous-ensemble d'images
4. Vérifier que les images ne sont pas corrompues

### Avertissement "Aucune cible détectée

**Causes possibles:**

* Oubli de marquer les images cibles
* Les images des cibles ne contiennent pas de cibles visibles
* Paramètres de détection des cibles trop stricts

**Solutions:**

1. Révision [Choosing Target Images](choosing-target-images.md)
2. Marquer les images appropriées dans la colonne Cible
3. Vérifier que les cibles sont visibles dans les images marquées
4. Ajuster les paramètres de détection des cibles si nécessaire

***

## Conseils pour un traitement réussi

### Avant de commencer

1. **Testez d'abord avec un petit sous-ensemble** - Traitez 10 à 20 images pour vérifier les paramètres
2. **Vérifier l'espace disque disponible** - S'assurer que 2 à 3 fois la taille du jeu de données sont libres
3. **Fermer les applications inutiles** - Libérer les ressources du système
4. **Vérifier les images cibles** - Prévisualiser les cibles marquées pour s'assurer de leur qualité
5. **Sauvegarder le projet** - Le projet est sauvegardé automatiquement, mais il est préférable de le sauvegarder manuellement

### Pendant le traitement

1. **Éviter la mise en veille du système** - Désactiver les modes d'économie d'énergie
2. **Garder Chloros au premier plan** - Ou au moins visible dans la barre des tâches
3. **Surveiller la progression de temps en temps** - Vérifier les avertissements ou les erreurs
4. **Ne chargez pas d'autres applications lourdes** - Surtout avec le mode parallèle Chloros+

### Chloros+ Accélération GPU

Si vous utilisez l'accélération GPU NVIDIA :

1. Mettre à jour les pilotes NVIDIA avec la dernière version
2. S'assurer que le GPU dispose de plus de 4 Go de VRAM
3. Fermez les applications gourmandes en GPU (jeux, édition vidéo)
4. Surveiller la température du GPU (assurer un refroidissement adéquat)

***

## Prochaines étapes

Une fois que le traitement a commencé :

1. **Surveiller la progression** - Voir [Monitoring the Processing](monitoring-the-processing.md)
2. **Attendre l'achèvement** - Le traitement s'exécute automatiquement
3. **Examiner les résultats** - Voir [Finishing the Processing](finishing-the-processing.md)

Pour plus d'informations sur ce qu'il faut faire pendant le traitement, voir [Monitoring the Processing](monitoring-the-processing.md).
