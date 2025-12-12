# Démarrage du traitement

Une fois que vous avez importé vos images, marqué vos cibles d'étalonnage et configuré les paramètres de votre projet, vous êtes prêt à commencer le traitement. Cette page vous guide dans le lancement du pipeline de traitement Chloros.

## Liste de contrôle du prétraitement

Avant de cliquer sur le bouton Start, vérifiez que tout est prêt :

* [ ] **Fichiers importés** - Toutes les images apparaissent dans le navigateur de fichiers
* [ ] **Images cibles marquées** - La colonne Cible a été vérifiée pour les images d'étalonnage
* [ ] **Modèles de caméra détectés** - La colonne Modèle de caméra affiche les caméras correctes
* [ ] **Paramètres configurés** - Les paramètres du projet sont examinés et ajustés
* [ ] **Indices sélectionnés** - Les indices multispectraux souhaités sont ajoutés (si nécessaire)
* [Format d'exportation choisi** - Format de sortie approprié pour votre flux de travail

{% hint style="info" %}
**Tip**: Click through a few images in the File Browser to verify they loaded correctly before processing.
{% endhint %}

***

## Démarrage du traitement

### Localiser le bouton de démarrage

Le bouton Démarrer/Lecture est situé dans la barre d'en-tête supérieure de Chloros :

* Position : Centre supérieur de la fenêtre
* Icône : **Bouton Jouer/Démarrer** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Statut : Le bouton est activé (lumineux) lorsqu'il est prêt à être utilisé

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
{% endhint %})

***

## Ce qui se passe pendant le traitement

### Étape 1 : Détection de la cible

**Ce que Chloros fait:**

* Analyse les images de cibles marquées (ou toutes les images si aucune n'est marquée)
* Identifie les 4 panneaux d'étalonnage de chaque cible
* Extrait les valeurs de réflectance des panneaux cibles
* Enregistre les horodatages des cibles pour la programmation de l'étalonnage

**Durée:** 1-30 secondes (avec des cibles marquées), 5-30+ minutes (non marquées)

### Étape 2 : Débayerisation (conversion RAW)

**Ce que Chloros fait:**

* Convertit les données de motif de Bayer RAW en images complètes RGB
* Applique un algorithme de démosaïquage de haute qualité
* Préserve la qualité et les détails de l'image

**Durée:** Varie en fonction du nombre d'images et de la vitesse de l'unité centrale

### Étape 3 : Étalonnage

**Ce que Chloros fait:**

* **Correction de la vignette** : Supprime l'assombrissement de l'objectif sur les bords
* **Étalonnage de la réflectance** : Normalisation à l'aide des valeurs de réflectance cibles
* Applique les corrections à toutes les bandes/canaux
* Utilise la cible d'étalonnage appropriée pour chaque image en fonction de l'horodatage

**Durée:** Majorité du temps de traitement

### Étape 4 : Calcul de l'indice

**Ce que fait Chloros:**

* Calcule les indices multispectraux configurés (NDVI, NDRE, etc.)
* Applique les mathématiques de bande aux images calibrées
* Génère des images d'index pour chaque index sélectionné

**Durée:** Quelques secondes par image

### Étape 5 : Exportation

**Ce que fait Chloros:**

* Sauvegarde les images calibrées dans le format sélectionné
* Exporte les images d'index avec les couleurs LUT configurées
* Écrit les fichiers dans les sous-dossiers du modèle de caméra
* Préserve les noms de fichiers originaux avec les suffixes

**Durée:** Varie en fonction du format d'exportation et de la taille du fichier

***

## Comportement de traitement

### Pipeline de traitement automatique

Une fois lancé, l'ensemble du pipeline s'exécute automatiquement :

* Aucune interaction de l'utilisateur n'est nécessaire
* Toutes les étapes configurées s'exécutent en séquence
* Les mises à jour de la progression sont affichées en temps réel

### Utilisation de l'ordinateur pendant le traitement

**Mode libre:**

* Utilisation relativement faible de l'unité centrale (simple)
* L'ordinateur reste réactif pour d'autres tâches
* Il est possible de minimiser l'utilisation de Chloros et de travailler avec d'autres applications

**Chloros+ Mode parallèle:**

* Utilisation élevée du processeur (multithread, jusqu'à 16 cœurs)
* Avec accélération GPU : Utilisation élevée du GPU
* L'ordinateur peut être moins réactif pendant le traitement
* Évitez de lancer d'autres tâches gourmandes en ressources processeur

{% hint style="warning" %}
**Performance Tip**: For best Chloros+ performance, close other applications and let Chloros use full system resources.
{% endhint %}

### Le traitement ne peut pas être interrompu

**Important limitations:**

* Une fois lancé, le traitement ne peut pas être interrompu
* Il est possible d'annuler le traitement, mais la progression est perdue
* Les résultats partiels ne sont pas sauvegardés
* Le traitement doit être repris depuis le début s'il est annulé

**Conseil de planification:** Pour les très grands projets, envisagez de traiter par lots ou d'utiliser CLI pour un meilleur contrôle.

***

## Suivi de votre traitement

Pendant l'exécution du traitement, vous pouvez

* **regarder la barre de progression** - voir le pourcentage d'achèvement global
* **Afficher l'étape en cours** - Détecter, Analyser, Étalonner ou Exporter
* **Vérifier l'onglet du journal** - Voir les messages de traitement détaillés et les avertissements
* **Visualiser les images terminées** - Certains fichiers d'exportation peuvent apparaître pendant le traitement

Pour des informations détaillées sur la surveillance, voir [Surveillance du traitement] (surveillance-the-processing.md).

***

## Annulation du traitement

Si vous devez interrompre le traitement :

### Comment annuler

1. Localisez le bouton **Stop/Cancel** (qui remplace le bouton Start pendant le traitement)
2. Cliquez sur le bouton Stop
3. Le traitement s'arrête immédiatement
4. Les résultats partiels sont rejetés

### Quand annuler

**Raisons valables d'annulation:**

* Réalisation que des paramètres incorrects ont été utilisés
* Oubli de marquer les images cibles
* Mauvaises images importées
* Le système est trop lent ou ne répond pas

**Après l'annulation:**

* Examiner et résoudre les problèmes éventuels
* Ajuster les paramètres si nécessaire
* Recommencer le traitement depuis le début
* Pour une expérience plus propre, fermez complètement Chloros et redémarrez

{% hint style="warning" %}
**No Partial Results**: Canceling discards all progress. Chloros does not save partially processed images.
{% endhint %}

***

## Estimation du temps de traitement

Le délai de traitement réel varie considérablement en fonction des éléments suivants

* Du nombre d'images
* Résolution de l'image
* Format d'entrée RAW ou JPG
* Mode de traitement (libre ou Chloros+)
* Vitesse du processeur et nombre de cœurs
* Disponibilité du GPU (Chloros+ uniquement)
* Nombre d'indices à calculer
* Complexité du format d'exportation

### Estimations approximatives (Chloros+, images de 12MP, CPU moderne)

| Nombre d'images | Mode libre | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 images | 15-20 min | 5-8 min | 3-5 min | 100 images | 30-40 min | 5-8 min | 3-5 min
| 100 images | 30-40 min | 10-15 min | 5-8 min | | 200 images | 1-1,5 h
| 200 images | 1-1.5 h | 20-30 min | 10-15 min | 500 images | 2-3 h | 45 min | 5-8 min | 3-5 min
| 500 images | 2-3 heures | 45-60 min | 20-30 min | 1000 images | 4-6 heures | 5-8 min | 5-5 min | 500 images | 4-6 heures | 5-8 min | 5-5 min
| 1000 images | 4-6 heures | 1,5-2 heures | 40-60 min | 1,5-2 heures | 1,5-2 heures | 1,5-2 heures | 1,5-2 heures | 1,5-2 heures

{% hint style="info" %}
**First Run**: Initial processing may take longer as Chloros builds caches and profiles. Subsequent processing of similar datasets will be faster.
{% endhint %}

***

## Problèmes communs au début

### Bouton de démarrage désactivé (grisé)

**Causes possibles:**

* Aucune image n'a été importée
* Le backend n'est pas complètement démarré
* Traitement précédent toujours en cours
* Le projet n'est pas complètement chargé

**Solutions:**

1. Attendre l'initialisation complète du backend (vérifier l'icône du menu principal)
2. Vérifier que les images sont importées dans le navigateur de fichiers
3. Redémarrer Chloros si le bouton reste désactivé
4. Vérifier les messages d'erreur dans le journal de débogage

### Le traitement commence puis échoue immédiatement

**Causes possibles:**

* Aucune image valide dans le projet
* Fichiers d'images corrompus
* Espace disque insuffisant
* Mémoire insuffisante (RAM)

**Solutions:**

1. Vérifier le journal de débogage <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pour les messages d'erreur
2. Vérifier l'espace disque disponible
3. Essayer de traiter un plus petit sous-ensemble d'images
4. Vérifier que les images ne sont pas corrompues

### Avertissement "Aucune cible détectée

**Causes possibles:**

* Oubli de marquer les images cibles
* Les images des cibles ne contiennent pas de cibles visibles
* Paramètres de détection des cibles trop stricts

**Solutions:**

1. Revoir [Choix des images cibles](choosing-target-images.md)
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
4. **Ne pas charger d'autres applications lourdes** - Surtout avec le mode parallèle Chloros+

### Chloros+ Accélération GPU

Si vous utilisez l'accélération GPU NVIDIA :

1. Mettez à jour les pilotes NVIDIA avec la dernière version
2. S'assurer que le GPU dispose de plus de 4 Go de VRAM
3. Fermez les applications gourmandes en GPU (jeux, édition vidéo)
4. Surveiller la température du GPU (assurer un refroidissement adéquat)

***

## Prochaines étapes

Une fois que le traitement a commencé :

1. **Surveiller la progression** - Voir [Suivi du traitement] (suivi-the-processing.md)
2. **Attendre l'achèvement** - Le traitement s'exécute automatiquement
3. **Examen des résultats** - Voir [Fin du traitement](Fin-the-processing.md)

Pour plus d'informations sur ce qu'il convient de faire pendant le traitement, voir [Suivi du traitement](suivi-the-processing.md).
