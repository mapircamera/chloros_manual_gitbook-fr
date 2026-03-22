# Lancement du traitement

Une fois que vous avez importé vos images, marqué vos cibles d&#x27;étalonnage et configuré les paramètres de votre projet, vous êtes prêt à commencer le traitement. Cette page vous guide dans le lancement du pipeline de traitement Chloros.

## Liste de contrôle avant le traitement

Avant de cliquer sur le bouton Démarrer, vérifiez que tout est prêt :

* [ ] **Fichiers importés** - Toutes les images apparaissent dans le navigateur de fichiers
* [ ] **Images cibles marquées** - La colonne Cible est cochée pour les images d&#x27;étalonnage
* [ ] **Modèles de caméra détectés** - La colonne Modèle de caméra affiche les caméras correctes
* [ ] **Paramètres configurés** - Les paramètres du projet ont été vérifiés et ajustés
* [ ] **Indices sélectionnés** - Indices multispectraux souhaités ajoutés (si nécessaire)
* [ ] **Format d&#x27;exportation choisi** - Format de sortie adapté à votre flux de travail

{% hint style="info" %}
**Astuce** : Parcourez quelques images dans le navigateur de fichiers pour vérifier qu&#x27;elles se sont chargées correctement avant le traitement.
{% endhint %}

***

## Lancer le traitement

### Localiser le bouton Démarrer

Le bouton Démarrer/Lecture se trouve dans la barre d&#x27;en-tête supérieure de Chloros :

* Emplacement : en haut au centre de la fenêtre
* Icône : **bouton Lecture/Démarrer** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* État : le bouton est activé (allumé) lorsqu&#x27;il est prêt à traiter

### Cliquez pour démarrer

1. Cliquez sur le **bouton Lecture/Démarrer** dans l&#x27;en-tête supérieur
2. Le traitement commence immédiatement
3. Le bouton devient inactif (grisé) pendant le traitement
4. La barre de progression s&#x27;actualise, indiquant l&#x27;état du traitement

{% hint style="success" %}
**Traitement lancé** : une fois cliqué, Chloros gère automatiquement toutes les étapes du traitement : détection de la cible, débayérisation, étalonnage, calcul de l&#x27;indice et exportation.
{% endhint %}

***

## Comprendre les modes de traitement

Chloros fonctionne selon deux modes de traitement différents en fonction de votre licence :

### Mode gratuit (traitement séquentiel)

**Disponible pour tous les utilisateurs**

**Fonctionnement :**

* Traite les images une par une, de manière séquentielle
* Fonctionnement monothread
* Utilisation réduite de la mémoire

**La barre de progression affiche 2 étapes :**

1.**Détection des cibles** - Recherche des cibles d&#x27;étalonnage
2. **Traitement** - Application de l&#x27;étalonnage et exportation des images**Durée du traitement :**

* Beaucoup plus lent que le mode parallèle Chloros+
* Convient aux ensembles de données de petite à moyenne taille (&lt; 200 images)

### Mode Chloros+ (traitement parallèle)

**Nécessite une licence Chloros+**

**Fonctionnement :**

* Traite plusieurs images simultanément à l&#x27;aide d&#x27;un [pipeline de traitement à 4 threads](../processing-architecture/processing-pipeline.md)
* L&#x27;[adaptation dynamique des calculs](../processing-architecture/dynamic-compute-adaptation.md) sélectionne automatiquement la stratégie optimale pour votre matériel
* Accélération GPU (CUDA) avec les cartes graphiques NVIDIA (ordinateurs de bureau et Jetson)
* Évolutif d&#x27;un Jetson Nano (1 worker) à un ordinateur de bureau avec un GPU de 12 Go ou plus (3-4 workers)

**La barre de progression affiche 4 étapes** (correspondant aux 4 threads du pipeline) :

1. **Détection** (Thread 1) - Recherche des cibles de calibrage
2. **Analyse** (Thread 2) - Examen des métadonnées de l&#x27;image et calcul du calibrage
3. **Calibrage** (thread 3) - Débayérisation GPU, correction du vignettage, calcul de l&#x27;indice
4. **Exportation** (thread 4) - Enregistrement des images traitées et des indices**Interaction avec la barre de progression :*** **Passez la souris** sur la barre pour afficher le panneau déroulant détaillé en 4 étapes
* **Cliquez** sur la barre de progression pour figer le panneau déroulant
* **Cliquez à nouveau** pour débloquer et masquer le panneau**Temps de traitement :**

* Nettement plus rapide que le mode gratuit
* Évolutif en fonction du nombre de cœurs du processeur
* L&#x27;accélération GPU améliore encore la vitesse

{% hint style="info" %}
**Chloros+ Vitesse** : le traitement parallèle peut être 5 à 10 fois plus rapide que le mode séquentiel pour les grands ensembles de données. Un projet de 500 images qui prend 2 heures en mode gratuit peut être terminé en 15 à 20 minutes avec Chloros+.
{% endhint %}

***

## Que se passe-t-il pendant le traitement ?

### Étape 1 : Détection des cibles

**Ce que fait Chloros :**

* Analyse les images de cibles marquées (ou toutes les images si aucune n&#x27;est marquée)
* Identifie les 4 panneaux d&#x27;étalonnage dans chaque cible
* Extrait les valeurs de réflectance des panneaux de cibles
* Enregistre les horodatages des cibles pour la planification de l&#x27;étalonnage

**Durée :** 1 à 30 secondes (avec des cibles marquées), 5 à 30 minutes ou plus (sans cibles marquées)

### Étape 2 : Démosaïquage (conversion RAW)

**Ce que fait Chloros :**

* Convertit les données RAW au format Bayer en images RGB complètes
* Applique un algorithme de démosaïquage de haute qualité
* Préserve au maximum la qualité et les détails de l&#x27;image

**Durée :** Varie en fonction du nombre d&#x27;images et de la vitesse du processeur

### Étape 3 : Étalonnage

**Fonctionnalités de Chloros :*** **Correction de la vignettage** : supprime l&#x27;assombrissement des bords dû à l&#x27;objectif
* **Calibrage de la réflectance** : normalise à l&#x27;aide de valeurs de réflectance cibles
* Applique des corrections sur toutes les bandes/canaux
* Utilise une cible de calibrage appropriée pour chaque image en fonction de l&#x27;horodatage

**Durée :** la majeure partie du temps de traitement

### Étape 4 : Calcul des indices

**Fonctionnalités de Chloros :**

* Calcule les indices multispectraux configurés (NDVI, NDRE, etc.)
* Applique des opérations mathématiques sur les bandes aux images calibrées
* Génère des images d&#x27;indice pour chaque indice sélectionné

**Durée :** Quelques secondes par image

### Étape 5 : Exportation

**Fonctionnalités de Chloros :**

* Enregistre les images calibrées dans le format sélectionné
* Exporte les images d&#x27;indice avec les couleurs LUT configurées
* Enregistre les fichiers dans les sous-dossiers correspondant aux modèles de caméra
* Conserve les noms de fichiers d&#x27;origine avec des suffixes

**Durée :** Varie en fonction du format d&#x27;exportation et de la taille des fichiers***

## Comportement du traitement

### Pipeline de traitement automatique

Une fois lancé, l&#x27;ensemble du pipeline s&#x27;exécute automatiquement :

* Aucune interaction de l&#x27;utilisateur n&#x27;est nécessaire
* Toutes les étapes configurées s&#x27;exécutent dans l&#x27;ordre
* Les mises à jour de progression s&#x27;affichent en temps réel

### Utilisation de l&#x27;ordinateur pendant le traitement

**Mode libre :**

* Utilisation relativement faible du processeur (monothread)
* L&#x27;ordinateur reste réactif pour d&#x27;autres tâches
* Vous pouvez réduire Chloros en arrière-plan et travailler dans d&#x27;autres applications en toute sécurité

**Chloros+ Mode parallèle :**

* Utilisation élevée du processeur (multithread, jusqu&#x27;à 16 cœurs)
* Avec accélération GPU : utilisation élevée du GPU
* L&#x27;ordinateur peut être moins réactif pendant le traitement
* Évitez de lancer d&#x27;autres tâches gourmandes en ressources CPU

{% hint style="warning" %}
**Conseil de performance** : pour des performances optimales, fermez les autres applications et laissez Chloros utiliser toutes les ressources du système.
{% endhint %}

### Le traitement ne peut pas être mis en pause

**Restrictions importantes :**

* Une fois lancé, le traitement ne peut pas être mis en pause
* Vous pouvez annuler le traitement, mais la progression sera perdue
* Les résultats partiels ne sont pas enregistrés
* Il faut recommencer depuis le début en cas d&#x27;annulation

**Conseil de planification :** Pour les très grands projets, envisagez de traiter par lots ou d&#x27;utiliser CLI pour un meilleur contrôle.***

## Suivi de votre traitement

Pendant l&#x27;exécution du traitement, vous pouvez :

* **Observer la barre de progression** - Voir le pourcentage global d&#x27;achèvement
* **Afficher l&#x27;étape en cours** - Détection, analyse, étalonnage ou exportation
* **Consulter l&#x27;onglet Journal** - Voir les messages et avertissements détaillés relatifs au traitement
* **Prévisualiser les images terminées** - Certains fichiers d&#x27;exportation peuvent apparaître pendant le traitement

Pour plus d&#x27;informations sur la surveillance, consultez [Surveillance du traitement](monitoring-the-processing.md).

***

## Annulation du traitement

Si vous devez interrompre le traitement :

### Comment annuler

1. Repérez le **bouton Arrêter/Annuler** (qui remplace le bouton Démarrer pendant le traitement)
2. Cliquez sur le bouton Arrêter
3. Le traitement s&#x27;arrête immédiatement
4. Les résultats partiels sont supprimés

### Quand annuler

**Raisons valables pour annuler :**

* Vous vous êtes rendu compte que des paramètres incorrects ont été utilisés
* Vous avez oublié de marquer les images cibles
* Des images incorrectes ont été importées
* Le système est trop lent ou ne répond plus

**Après l&#x27;annulation :**

* Vérifiez et corrigez les éventuels problèmes
* Ajustez les paramètres si nécessaire
* Redémarrez le traitement depuis le début
* Pour une expérience optimale, fermez complètement Chloros et redémarrez

{% hint style="warning" %}
**Pas de résultats partiels** : l&#x27;annulation supprime toute la progression. Chloros n&#x27;enregistre pas les images partiellement traitées.
{% endhint %}

***

## Estimations du temps de traitement

Le temps de traitement réel varie considérablement en fonction des facteurs suivants :

* Nombre d&#x27;images
* Résolution des images
* Format d&#x27;entrée RAW ou JPG
* Mode de traitement (Free ou Chloros+)
* Vitesse du processeur et nombre de cœurs
* Disponibilité d&#x27;un GPU (Chloros+ uniquement)
* Nombre d&#x27;index à calculer
* Complexité du format d&#x27;exportation

### Estimations approximatives (Chloros+, images 12 MP, processeur moderne)

| Nombre d&#x27;images | Mode gratuit | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 images   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 images  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 images  | 1-1,5 h | 20-30 min      | 10-15 min      |
| 500 images  | 2 à 3 heures   | 45 à 60 min      | 20 à 30 min      |
| 1 000 images | 4 à 6 heures   | 1 h 30 à 2 heures      | 40 à 60 min      |

{% hint style="info" %}
**Première exécution** : le traitement initial peut prendre plus de temps car Chloros crée des caches et des profils. Les traitements suivants de jeux de données similaires seront plus rapides.
{% endhint %}

***

## Problèmes courants au démarrage

### Bouton Démarrer désactivé (grisé)

**Causes possibles :**

* Aucune image importée
* Le backend n&#x27;est pas entièrement démarré
* Le traitement précédent est toujours en cours
* Le projet n&#x27;est pas entièrement chargé

**Solutions :**

1. Attendez que le backend s&#x27;initialise complètement (vérifiez l&#x27;icône du menu principal)
2. Vérifiez que les images sont bien importées dans le navigateur de fichiers
3. Redémarrez Chloros si le bouton reste désactivé
4. Consultez le journal de débogage pour les messages d&#x27;erreur

### Le traitement démarre puis échoue immédiatement

**Causes possibles :**

* Aucune image valide dans le projet
* Fichiers image corrompus
* Espace disque insuffisant
* Mémoire insuffisante (RAM)

**Solutions :**

1. Vérifiez le journal de débogage <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pour vérifier s&#x27;il contient des messages d&#x27;erreur
2. Vérifiez l&#x27;espace disque disponible
3. Essayez de traiter un sous-ensemble plus petit d&#x27;images
4. Vérifiez que les images ne sont pas corrompues

### Avertissement « Aucune cible détectée »

**Causes possibles :**

* Oubli de marquer les images cibles
* Les images cibles ne contiennent pas de cibles visibles
* Paramètres de détection des cibles trop stricts

**Solutions :**

1. Consultez la section [Choix des images cibles](choosing-target-images.md)
2. Marquez les images appropriées dans la colonne Cible
3. Vérifiez que les cibles sont visibles dans les images marquées
4. Ajustez les paramètres de détection des cibles si nécessaire

***

## Conseils pour un traitement réussi

### Avant de commencer

1. **Testez d&#x27;abord avec un petit sous-ensemble** - Traitez 10 à 20 images pour vérifier les paramètres
2. **Vérifiez l&#x27;espace disque disponible** - Assurez-vous de disposer d&#x27;un espace libre équivalent à 2 à 3 fois la taille de l&#x27;ensemble de données
3. **Fermez les applications inutiles** - Libérez des ressources système
4. **Vérifiez les images cibles** - Prévisualisez les cibles marquées pour vous assurer de leur qualité
5. **Enregistrez le projet** - Le projet est enregistré automatiquement, mais il est recommandé de l&#x27;enregistrer manuellement

### Pendant le traitement

1. **Évitez la mise en veille du système** - Désactivez les modes d&#x27;économie d&#x27;énergie
2. **Gardez Chloros au premier plan** - Ou au moins visible dans la barre des tâches
3. **Surveillez la progression de temps à autre** - Vérifiez s&#x27;il y a des avertissements ou des erreurs
4. **Ne chargez pas d&#x27;autres applications lourdes** - En particulier avec le mode parallèle Chloros+

### Chloros+ Accélération GPU

Si vous utilisez l&#x27;accélération GPU NVIDIA :

1. Mettez à jour les pilotes NVIDIA vers la dernière version
2. Assurez-vous que le GPU dispose d&#x27;au moins 4 Go de VRAM
3. Fermez les applications gourmandes en ressources GPU (jeux, montage vidéo)
4. Surveillez la température du GPU (assurez-vous que le refroidissement est adéquat)

***

## Étapes suivantes

Une fois le traitement lancé :

1. **Surveillez la progression** - Voir [Surveillance du traitement](monitoring-the-processing.md)
2. **Attendez la fin du traitement** - Le traitement s&#x27;exécute automatiquement
3. **Vérifiez les résultats** - Voir [Fin du traitement](finishing-the-processing.md)

Pour savoir comment procéder pendant le traitement, consultez [Surveillance du traitement](monitoring-the-processing.md).
