# Surveillance du traitement

Une fois le traitement lancé, Chloros offre plusieurs moyens de surveiller la progression, de vérifier les problèmes et de comprendre ce qui se passe avec votre ensemble de données. Cette page explique comment suivre votre traitement et interpréter les informations fournies par Chloros.

## Présentation de la barre de progression

La barre de progression située dans l&#x27;en-tête supérieur affiche l&#x27;état du traitement en temps réel et le pourcentage d&#x27;achèvement.

### Barre de progression en mode gratuit

Pour les utilisateurs ne disposant pas d&#x27;une licence Chloros+ :

**Affichage de la progression en deux étapes :**

1. **Détection de la cible** - Recherche des cibles d&#x27;étalonnage dans les images
2. **Traitement** - Application des corrections et exportation

**La barre de progression affiche :**

* Le pourcentage d&#x27;achèvement global (0-100 %)
* Le nom de l&#x27;étape en cours
* Une visualisation simple sous forme de barre horizontale

### Barre de progression Chloros+

Pour les utilisateurs disposant d&#x27;une licence Chloros+ :

**Affichage de la progression en 4 étapes :**

1. **Détection** - Recherche des cibles d&#x27;étalonnage
2. **Analyse** - Examen des images et préparation du pipeline
3. **Étalonnage** - Application des corrections de vignettage et de réflectance
4. **Exportation** - Enregistrement des fichiers traités

**Fonctionnalités interactives :**

* **Passez la souris sur** la barre de progression pour afficher le panneau à 4 étapes
* **Cliquez sur** la barre de progression pour figer/épingler le panneau détaillé
* **Cliquez à nouveau** pour le débloquer et le masquer automatiquement lorsque vous quittez la souris
* Chaque étape affiche la progression individuelle (0-100 %)

***

## Comprendre chaque étape du traitement

### Étape 1 : Détection (détection des cibles)

**Ce qui se passe :**

* Chloros analyse les images marquées avec la case à cocher Cible
* Les algorithmes de vision par ordinateur identifient les 4 panneaux d&#x27;étalonnage
* Les valeurs de réflectance sont extraites de chaque panneau
* Les horodatages des cibles sont enregistrés pour une planification correcte de l&#x27;étalonnage

**Durée :**

* Avec cibles marquées : 10 à 60 secondes
* Sans cibles marquées : 5 à 30 minutes ou plus (analyse toutes les images)

**Indicateur de progression :**

* Détection : 0 % → 100 %
* Nombre d&#x27;images scannées
* Nombre de cibles trouvées

**À surveiller :**

* Le processus devrait être rapide si les cibles sont correctement marquées.
* Si le processus prend trop de temps, il se peut que les cibles ne soient pas marquées.
* Vérifiez le journal de débogage pour voir s&#x27;il contient des messages « Cible trouvée ».

### Étape 2 : Analyse

**Ce qui se passe :**

* Lecture des métadonnées EXIF des images (horodatages, paramètres d&#x27;exposition)
* Détermination de la stratégie d&#x27;étalonnage en fonction des horodatages des cibles
* Organisation de la file d&#x27;attente de traitement des images
* Préparation des travailleurs de traitement parallèle (Chloros+ uniquement)

**Durée :** 5 à 30 secondes

**Indicateur de progression :**

* Analyse : 0 % → 100 %
* Étape rapide, généralement terminée rapidement

**À surveiller :**

* La progression doit être régulière, sans pause.
* Des avertissements concernant les métadonnées manquantes apparaîtront dans le journal de débogage.

### Étape 3 : Calibrage

**Ce qui se passe :**

* **Débayérisation** : conversion du motif Bayer RAW en 3 canaux
* **Correction du vignettage** : suppression du noircissement des bords de l&#x27;objectif
* **Calibrage de la réflectance** : normalisation avec les valeurs cibles
* **Calcul de l&#x27;indice** : calcul des indices multispectraux
* Traitement de chaque image via le pipeline complet

**Durée :** la majeure partie du temps de traitement total (60 à 80 %)

**Indicateur de progression :**

* Calibrage : 0 % → 100 %
* Image en cours de traitement
* Images terminées / Nombre total d&#x27;images

**Comportement du traitement :**

* **Mode libre** : traite une image à la fois de manière séquentielle
* **Mode Chloros+** : traite jusqu&#x27;à 16 images simultanément
* **Accélération GPU** : accélère considérablement cette étape

**À surveiller :**

* Progression régulière du nombre d&#x27;images
* Vérifiez le journal de débogage pour les messages de fin de traitement par image
* Avertissements concernant la qualité de l&#x27;image ou les problèmes d&#x27;étalonnage

### Étape 4 : Exportation

**Ce qui se passe :**

* Écriture des images calibrées sur le disque dans le format sélectionné
* Exportation des images d&#x27;index multispectral avec les couleurs LUT
* Création de sous-dossiers pour les modèles d&#x27;appareils photo
* Conservation des noms de fichiers d&#x27;origine avec les suffixes appropriés

**Durée :** 10 à 20 % du temps de traitement total

**Indicateur de progression :**

* Exportation : 0 % → 100 %
* Fichiers en cours d&#x27;écriture
* Format d&#x27;exportation et destination

**À surveiller :**

* Avertissements d&#x27;espace disque
* Erreurs d&#x27;écriture de fichiers
* Achèvement de toutes les sorties configurées

***

## Onglet Journal de débogage

Le journal de débogage fournit des informations détaillées sur la progression du traitement et les problèmes rencontrés.

### Accès au journal de débogage

1. Cliquez sur l&#x27;icône **Journal de débogage** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> dans la barre latérale gauche.
2. Le panneau du journal s&#x27;ouvre et affiche les messages de traitement en temps réel.
3. Défilement automatique pour afficher les derniers messages.

### Comprendre les messages du journal

#### Messages d&#x27;information (blanc/gris)

Mises à jour normales du traitement :

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Messages d&#x27;avertissement (jaune)

Problèmes non critiques qui n&#x27;interrompent pas le traitement :

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Action :** Vérifiez les avertissements après le traitement, mais n&#x27;interrompez pas celui-ci.

#### Messages d&#x27;erreur (Red)

Problèmes critiques pouvant entraîner l&#x27;échec du traitement :

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Action :** Arrêter le traitement, résoudre l&#x27;erreur, redémarrer.

### Messages courants du journal

| Message                          | Signification                                | Action requise                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| « Cible détectée dans \[nom de fichier] » | Cible d&#x27;étalonnage trouvée avec succès  | Aucune - normal                                         |
| « Traitement de l&#x27;image X sur Y »        | Mise à jour de la progression actuelle                | Aucune - normal                                         |
| « Aucune cible trouvée »               | Aucune cible d&#x27;étalonnage détectée        | Marquer les images cibles ou désactiver l&#x27;étalonnage de la réflectance |
| « Espace disque insuffisant »        | Espace de stockage insuffisant pour la sortie          | Libérer de l&#x27;espace disque                                    |
| « Ignorer le fichier corrompu »        | Le fichier image est endommagé                  | Recopier le fichier depuis la carte SD                             |
| « Données PPK appliquées »               | Corrections GPS du fichier .daq appliquées | Aucune - normal                                         |

### Copie des données du journal

Pour copier le journal à des fins de dépannage ou d&#x27;assistance :

1. Ouvrez le panneau Journal de débogage.
2. Cliquez sur le bouton **« Copier le journal »** (ou cliquez avec le bouton droit de la souris → Sélectionner tout).
3. Collez dans un fichier texte ou un e-mail.
4. Envoyez à l&#x27;assistance MAPIR si nécessaire.

***

## Surveillance des ressources système

### Utilisation du processeur

**Mode libre :**

* 1 cœur de processeur à environ 100 %
* Autres cœurs inactifs ou disponibles
* Le système reste réactif

**Chloros+ Mode parallèle :**

* Plusieurs cœurs à 80-100 % (jusqu&#x27;à 16 cœurs)
* Utilisation globale élevée du processeur
* Le système peut sembler moins réactif

**Pour surveiller :**

* Windows Gestionnaire de tâches (Ctrl+Maj+Échap)
* Onglet Performances → section CPU
* Recherchez les processus « Chloros » ou « chloros-backend »

### Utilisation de la mémoire (RAM)

**Utilisation type :**

* Petits projets (&lt; 100 images) : 2 à 4 Go
* Projets de taille moyenne (100 à 500 images) : 4 à 8 Go
* Grands projets (plus de 500 images) : 8 à 16 Go
* Le mode parallèle Chloros+ utilise davantage de RAM

**Si la mémoire est insuffisante :**

* Traitez des lots plus petits
* Fermez les autres applications
* Augmentez la RAM si vous traitez régulièrement de grands ensembles de données

### Utilisation du GPU (Chloros+ avec CUDA)

Lorsque l&#x27;accélération GPU est activée :

* Le GPU NVIDIA affiche une utilisation élevée (60-90 %)
* L&#x27;utilisation de la VRAM augmente (nécessite 4 Go+ de VRAM)
* La phase de calibrage est nettement plus rapide

**Pour surveiller :**

* Icône NVIDIA dans la barre d&#x27;état système
* Gestionnaire de tâches → Performances → GPU
* GPU-Z ou outil de surveillance similaire

### E/S disque

**À quoi s&#x27;attendre :**

* Lecture disque élevée pendant la phase d&#x27;analyse
* Écriture disque élevée pendant la phase d&#x27;exportation
* SSD nettement plus rapide que HDD

**Conseil de performance :**

* Utilisez un SSD pour le dossier du projet lorsque cela est possible
* Évitez les lecteurs réseau pour les ensembles de données volumineux
* Assurez-vous que le disque n&#x27;est pas proche de sa capacité maximale (ce qui affecte la vitesse d&#x27;écriture)

***

## Détection des problèmes pendant le traitement

### Signes avant-coureurs

**Progression bloquée (aucun changement pendant plus de 5 minutes) :**

* Vérifiez le journal de débogage pour détecter d&#x27;éventuelles erreurs.
* Vérifiez l&#x27;espace disque disponible.
* Vérifiez dans le Gestionnaire des tâches que Chloros est en cours d&#x27;exécution.

**Messages d&#x27;erreur fréquents :**

* Arrêtez le traitement et examinez les erreurs.
* Causes courantes : espace disque, fichiers corrompus, problèmes de mémoire.
* Consultez la section Dépannage ci-dessous.

**Le système ne répond plus :**

* Le mode parallèle Chloros+ utilise trop de ressources.
* Envisagez de réduire le nombre de tâches simultanées ou de mettre à niveau le matériel.
* Le mode libre est moins gourmand en ressources.

### Quand arrêter le traitement

Arrêtez le traitement si vous constatez :

* ❌ Des erreurs « Disque plein » ou « Impossible d&#x27;écrire le fichier »
* ❌ Erreurs répétées de corruption de fichiers image
* ❌ Système complètement bloqué (ne répond pas)
* ❌ Configuration incorrecte des paramètres
* ❌ Importation d&#x27;images incorrectes

**Comment arrêter :**

1. Cliquez sur le **bouton Arrêter/Annuler** (remplace le bouton Démarrer)
2. Le traitement s&#x27;arrête, la progression est perdue
3. Corrigez les problèmes et recommencez depuis le début

***

## Dépannage pendant le traitement

### Le traitement est très lent

**Causes possibles :**

* Images cibles non marquées (numérisation de toutes les images)
* Stockage sur disque dur au lieu de SSD
* Ressources système insuffisantes
* Nombreux index configurés
* Accès au lecteur réseau

**Solutions :**

1. Si le traitement vient de démarrer et est en phase de détection : annuler, marquer les cibles, redémarrer
2. Pour l&#x27;avenir : utiliser un SSD, réduire les index, mettre à niveau le matériel
3. Envisager l&#x27;utilisation de CLI pour le traitement par lots de grands ensembles de données

### Avertissements « Espace disque »

**Solutions :**

1. Libérer immédiatement de l&#x27;espace disque
2. Déplacer le projet vers un lecteur disposant de plus d&#x27;espace
3. Réduire le nombre d&#x27;index à exporter.
4. Utiliser le format JPG au lieu de TIFF (fichiers plus petits).

### Messages fréquents « Fichier corrompu »

**Solutions :**

1. Recopier les images depuis la carte SD pour garantir leur intégrité.
2. Tester la carte SD pour détecter d&#x27;éventuelles erreurs.
3. Supprimer les fichiers corrompus du projet.
4. Continuer le traitement des images restantes.

### Surchauffe / ralentissement du système

**Solutions :**

1. Assurez-vous que la ventilation est suffisante.
2. Nettoyez la poussière des évents de l&#x27;ordinateur.
3. Réduisez la charge de traitement (utilisez le mode Free au lieu de Chloros+).
4. Effectuez le traitement pendant les périodes les plus fraîches de la journée.

***

## Notification de fin de traitement

Lorsque le traitement est terminé :

* La barre de progression atteint 100 %.
* Le message **« Traitement terminé »** s&#x27;affiche dans le journal de débogage.
* Le bouton Démarrer redevient actif.
* Tous les fichiers de sortie se trouvent dans le sous-dossier du modèle d&#x27;appareil photo.

***

## Étapes suivantes

Une fois le traitement terminé :

1. **Vérifiez les résultats** - Voir [Fin du traitement](finishing-the-processing.md)
2. **Vérifiez le dossier de sortie** - Vérifiez que tous les fichiers ont été exportés correctement
3. **Consultez le journal de débogage** - Vérifiez s&#x27;il y a des avertissements ou des erreurs
4. **Prévisualisez les images traitées** - Utilisez la visionneuse d&#x27;images ou un logiciel externe

Pour plus d&#x27;informations sur la consultation et l&#x27;utilisation des résultats traités, consultez [Fin du traitement](finishing-the-processing.md).
