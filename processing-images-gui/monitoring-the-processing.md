# Suivi du traitement

Une fois que le traitement a commencé, Chloros fournit plusieurs moyens de suivre la progression, de vérifier les problèmes et de comprendre ce qui se passe avec votre jeu de données. Cette page explique comment suivre votre traitement et interpréter les informations fournies par Chloros.

## Vue d'ensemble de la barre de progression

La barre de progression située dans l'en-tête supérieur indique en temps réel l'état du traitement et le pourcentage d'achèvement.

### Barre de progression du mode libre

Pour les utilisateurs sans licence Chloros+ :

**Affichage de la progression en 2 étapes:**

1. **Target Detect** - Recherche de cibles d'étalonnage dans les images
2. **Traitement** - Application des corrections et exportation

**La barre de progression indique:**

* Le pourcentage d'achèvement global (0-100%)
* Le nom de l'étape en cours
* Visualisation simple de la barre horizontale

### Chloros+ Barre de progression

Pour les utilisateurs ayant la licence Chloros+ :

**Affichage de la progression en 4 étapes:**

1. **Détection** - Recherche de cibles d'étalonnage
2. **Analyse** - Examen des images et préparation du pipeline
3. **Calibrage** - Application des corrections de vignette et de réflectance
4. **Exportation** - Enregistrement des fichiers traités

**Fonctionnalités interactives:**

* **Passez la souris sur la barre de progression pour voir le panneau élargi en 4 étapes
* **Cliquez sur la barre de progression pour figer/épingler le panneau élargi
* **Cliquez à nouveau** pour libérer le panneau et le masquer automatiquement lorsque vous quittez la souris
* Chaque étape montre la progression individuelle (0-100%)

***

## Comprendre chaque étape de traitement

### Étape 1 : Détection (détection de la cible)

**What's happening:**

* Chloros analyse les images marquées d'une case à cocher Cible
* Des algorithmes de vision par ordinateur identifient les 4 panneaux d'étalonnage
* Les valeurs de réflectance sont extraites de chaque panneau
* Les horodatages des cibles sont enregistrés pour une programmation correcte de l'étalonnage

**Durée:**

* Avec des cibles marquées : 10-60 secondes
* Sans cibles marquées : 5-30+ minutes (balayage de toutes les images)

**Indicateur de progression:**

* Détection : 0% → 100%
* Nombre d'images scannées
* Nombre de cibles trouvées

**Ce qu'il faut surveiller:**

* L'opération devrait s'achever rapidement si les cibles sont correctement marquées
* Si cela prend trop de temps, il se peut que les cibles ne soient pas marquées
* Vérifier le journal de débogage pour les messages "Cible trouvée"

### Étape 2 : Analyse

**What's happening:*

* Lecture des métadonnées EXIF de l'image (horodatage, paramètres d'exposition)
* Détermination de la stratégie d'étalonnage sur la base des horodatages cibles
* Organisation de la file d'attente pour le traitement des images
* Préparation des opérateurs de traitement parallèle (Chloros+ uniquement)

**Durée:** 5-30 secondes

**Indicateur de progression:**

* Analyse : 0% → 100%
* Étape rapide, généralement terminée rapidement

**Ce qu'il faut surveiller:**

* La progression doit être régulière et sans pause
* Des avertissements concernant les métadonnées manquantes apparaîtront dans le journal de débogage

### Étape 3 : Étalonnage

**What's happening:**

* **Débayage** : Conversion du motif Bayer RAW en 3 canaux
* **Correction de la vignette** : Suppression de l'assombrissement des bords de l'objectif
* **Calibrage de la réflectance** : Étalonnage de la réflectance** : normalisation avec des valeurs cibles
* **Calcul de l'indice** : Calcul des indices multispectraux
* Traitement de chaque image par le pipeline complet

**Durée:** Majorité du temps de traitement total (60-80%)

**Indicateur de progrès:**

* Étalonnage : 0% → 100%
* Image en cours de traitement
* Images terminées / Total des images

**Comportement du traitement:**

* **Mode libre** : Traite une image à la fois de manière séquentielle
* **Chloros+ mode** : Traite jusqu'à 16 images simultanément
* **Accélération GPU** : Accélération du processeur graphique** : accélère considérablement cette étape

**Ce qu'il faut surveiller:**

* Progression régulière du nombre d'images
* Vérifier le journal de débogage pour les messages d'achèvement par image
* Avertissements concernant la qualité de l'image ou les problèmes d'étalonnage

### Étape 4 : Exportation

**What's happening:**

* Écriture des images calibrées sur le disque dans le format sélectionné
* Exportation d'images d'index multispectrales avec des couleurs LUT
* Création de sous-dossiers de modèles de caméras
* Préservation des noms de fichiers originaux avec les suffixes appropriés

**Durée:** 10-20% du temps total de traitement

**Indicateur de progression:**

* Exportation : 0% → 100%
* Fichiers en cours d'écriture
* Format d'exportation et destination

**Ce qu'il faut surveiller:**

* Avertissements concernant l'espace disque
* Erreurs d'écriture de fichier
* Achèvement de toutes les sorties configurées

***

## Onglet Debug Log

Le journal de débogage fournit des informations détaillées sur l'état d'avancement du traitement et sur les problèmes rencontrés.

### Accès au journal de débogage

1. Cliquez sur l'icône **Journal de débogage** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> dans la barre latérale gauche
2. Le panneau du journal s'ouvre et affiche les messages de traitement en temps réel
3. Défilement automatique pour afficher les messages les plus récents

### Comprendre les messages du journal

#### Messages d'information (blanc/gris)

Mises à jour normales du traitement :

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Messages d'avertissement (jaune)

Problèmes non critiques qui n'interrompent pas le traitement :

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Action:** Examiner les avertissements après le traitement, mais ne pas interrompre le traitement

#### Messages d'erreur (Red)

Problèmes critiques susceptibles de faire échouer le traitement :

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Action:** Arrêter le traitement, résoudre l'erreur, redémarrer

### Messages courants du journal

| Message - Signification - Action à entreprendre
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| "Target detected in \[filename]" (Cible détectée dans [nom de fichier]) | "Cible détectée dans \N[nom de fichier]" | Cible d'étalonnage trouvée avec succès | Aucun - normal |
| "Traitement de l'image X de Y"        | "Traitement de l'image X de Y" | Mise à jour de la progression en cours | Aucun - normal
| "Aucune cible trouvée" | Aucune cible d'étalonnage détectée | Marquer les images cibles ou désactiver l'étalonnage de la réflectance | "Aucune cible trouvée" | Aucune cible d'étalonnage détectée
| "Espace disque insuffisant" | Espace disque insuffisant pour la sortie | Libérer de l'espace disque
| "Skipping corrupted file" | Fichier d'image endommagé | Recopier le fichier à partir de la carte SD
| "Données PPK appliquées" | Les corrections GPS du fichier .daq sont appliquées | Aucune - normal |

### Copie des données du journal

Pour copier le journal à des fins de dépannage ou d'assistance :

1. Ouvrez le panneau Debug Log
2. Cliquez sur le bouton **"Copier le journal "** (ou cliquez avec le bouton droit de la souris → Tout sélectionner)
3. Coller dans un fichier texte ou dans un courriel
4. Envoyer au support MAPIR si nécessaire

***

## Surveillance des ressources du système

### Utilisation de l'unité centrale

**Free Mode:**

* 1 cœur de CPU à \~100%
* Les autres cœurs sont inactifs ou disponibles
* Le système reste réactif

**Chloros+ Mode parallèle:**

* Plusieurs cœurs à 80-100% (jusqu'à 16 cœurs)
* Utilisation globale élevée de l'unité centrale
* Le système peut sembler moins réactif

**To monitor:**

* Windows Gestionnaire des tâches (Ctrl+Shift+Esc)
* Onglet Performance → section CPU
* Rechercher les processus "Chloros" ou "chloros-backend"

### Utilisation de la mémoire (RAM)

**Utilisation typique:**

* Petits projets (< 100 images) : 2-4 GO
* Projets moyens (100-500 images) : 4-8 GO
* Grands projets (500+ images) : 8-16 GO
* Chloros+ Le mode parallèle utilise plus de RAM

**Si la mémoire est faible:**

* Traiter des lots plus petits
* Fermer les autres applications
* Augmenter la RAM si vous traitez régulièrement de grands ensembles de données

### Utilisation du GPU (Chloros+ avec CUDA)

Lorsque l'accélération GPU est activée :

* Le GPU NVIDIA affiche une utilisation élevée (60-90%)
* L'utilisation de la VRAM augmente (nécessite plus de 4 Go de VRAM)
* L'étape d'étalonnage est nettement plus rapide

**To monitor:**

* Icône de la barre d'état système de NVIDIA
* Gestionnaire des tâches → Performances → GPU
* GPU-Z ou outil de surveillance similaire

### E/S disque

**Ce à quoi il faut s'attendre:**

* Nombre élevé de lectures sur le disque pendant la phase d'analyse
* Nombre élevé d'écritures sur le disque pendant la phase d'exportation
* Les disques SSD sont nettement plus rapides que les disques durs

**Conseil de performance:**

* Utiliser un disque SSD pour le dossier du projet lorsque c'est possible
* Éviter les lecteurs réseau pour les grands ensembles de données
* S'assurer que le disque n'est pas proche de sa capacité (affecte la vitesse d'écriture)

***

## Détection des problèmes en cours de traitement

### Signes d'alerte

**La progression est bloquée (pas de changement pendant plus de 5 minutes):**

* Vérifier le journal de débogage pour les erreurs
* Vérifier l'espace disque disponible
* Vérifier dans le gestionnaire des tâches que Chloros est en cours d'exécution

**Error messages appear frequently:**

* Arrêter le traitement et examiner les erreurs
* Causes courantes : espace disque, fichiers corrompus, problèmes de mémoire
* Voir la section Dépannage ci-dessous

**Le système ne répond plus:**

* Chloros+ Le mode parallèle utilise trop de ressources
* Envisagez de réduire le nombre de tâches simultanées ou de mettre à niveau le matériel
* Le mode libre est moins gourmand en ressources

### Quand arrêter le traitement

Arrêtez le traitement si vous constatez :

* ❌ des erreurs "Disque plein" ou "Impossible d'écrire un fichier"
* ❌ Erreurs répétées de corruption du fichier image
* ❌ Le système est complètement bloqué (ne répond pas)
* ❌ Réalisation que des paramètres erronés ont été configurés
* mauvaises images importées

**How to stop:**

1. Cliquez sur le bouton **Arrêter/Annuler** (remplace le bouton Démarrer)
2. Le traitement s'arrête, la progression est perdue
3. Corriger les problèmes et recommencer depuis le début

***

## Dépannage en cours de traitement

### Le traitement est très lent

**Causes possibles:**

* Images cibles non marquées (balayage de toutes les images)
* Disque dur au lieu d'un disque SSD
* Ressources système insuffisantes
* Nombreux indices configurés
* Accès au lecteur réseau

**Solutions:**

1. Si vous venez de démarrer et que vous êtes en phase de détection : Annuler, marquer les cibles, redémarrer
2. Pour l'avenir : Utiliser un disque SSD, réduire les indices, mettre à niveau le matériel
3. Envisager CLI pour le traitement par lots de grands ensembles de données

### Avertissements concernant l'espace disque

**Solutions:**

1. Libérer immédiatement de l'espace disque
2. Déplacer le projet vers un disque disposant de plus d'espace
3. Réduire le nombre d'indices à exporter
4. Utiliser le format JPG au lieu de TIFF (fichiers plus petits)

### Messages fréquents "Fichier corrompu"

**Solutions:**

1. Recopier les images de la carte SD pour en assurer l'intégrité
2. Tester la carte SD pour vérifier qu'il n'y a pas d'erreurs
3. Supprimer les fichiers corrompus du projet
4. Poursuivre le traitement des images restantes

### Surchauffe du système / étranglement

**Solutions:**

1. Assurer une ventilation adéquate
2. Éliminer la poussière des évents de l'ordinateur
3. Réduire la charge de traitement (utiliser le mode libre au lieu de Chloros+)
4. Traiter aux heures les moins chaudes de la journée

***

## Avis de fin de traitement

Lorsque le traitement est terminé :

* La barre de progression atteint 100 %
* **Le message "Traitement terminé "** apparaît dans le journal de débogage
* Le bouton Démarrer est à nouveau activé
* Tous les fichiers de sortie se trouvent dans le sous-dossier du modèle de l'appareil photo

***

## Prochaines étapes

Une fois le traitement terminé :

1. **Examen des résultats** - Voir [Fin du traitement](Fin du traitement.md)
2. **Vérifier le dossier de sortie** - Vérifier que tous les fichiers ont été exportés correctement
3. **Consulter le journal de débogage** - Vérifier qu'il n'y a pas d'avertissement ou d'erreur
4. **Visualiser les images traitées** - Utiliser la visionneuse d'images ou un logiciel externe

Pour plus d'informations sur l'examen et l'utilisation des résultats traités, voir [Fin du traitement] (Fin du traitement.md).
