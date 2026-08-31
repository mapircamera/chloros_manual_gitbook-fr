# Lancement du traitement

Une fois que vous avez importé vos images, marqué vos cibles d’étalonnage et configuré les paramètres de votre projet, vous êtes prêt à commencer le traitement. Cette page vous guide dans le lancement du pipeline de traitement Chloros.

## Liste de contrôle avant le traitement

Avant de cliquer sur le bouton « Démarrer », vérifiez que tout est prêt :

* [ ] **Fichiers importés** - Toutes les images apparaissent dans le navigateur de fichiers
* [ ] **Images cibles marquées** - Colonne « Cible » cochée pour les images d&#x27;étalonnage (ou un enregistrement `.daq` importé pour LATTICE)
* [ ] **Modèles de caméras détectés** – La colonne « Modèle de caméra » affiche les caméras correctes
* [ ] **Paramètres configurés** – Les paramètres du projet ont été vérifiés et ajustés
* [ ] **Indices sélectionnés** – Les indices multispectraux souhaités ont été ajoutés (si nécessaire)
* [ ] **Format d&#x27;exportation choisi** - Format de sortie adapté à votre flux de travail

{% hint style="info" %}
**Astuce** : Cliquez sur quelques images dans le navigateur de fichiers pour vérifier qu&#x27;elles se sont chargées correctement avant le traitement.
{% endhint %}

***

## Lancer le traitement

### Localiser le bouton Démarrer

Le bouton Démarrer/Lecture se trouve dans la barre d&#x27;en-tête supérieure de Chloros :

* Emplacement : en haut au centre de la fenêtre
* Icône : **bouton Lecture/Démarrer** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* État : le bouton est activé (allumé) lorsqu’il est prêt à traiter

### Cliquez pour démarrer

1. Cliquez sur le **bouton Lecture/Démarrer** dans la barre d’en-tête supérieure
2. Le traitement commence immédiatement
3. Le bouton devient un bouton **Arrêt** pendant le traitement
4. La barre de progression s’actualise, indiquant l’état du traitement

{% hint style="success" %}
**Traitement lancé** : une fois que vous avez cliqué dessus, Chloros gère automatiquement toutes les étapes du traitement : détection des cibles, débayérisation, étalonnage, calcul de l’indice et exportation. Il détecte automatiquement si votre projet est de type Survey3, LATTICE ou un mélange des deux, et applique le pipeline adapté à chaque caméra.
{% endhint %}

***

## Comprendre les modes de traitement

Chloros fonctionne selon deux modes de traitement différents en fonction de votre licence :

### Mode gratuit (traitement séquentiel)

**Accessible à tous les utilisateurs**

**Fonctionnement :**

* Traite les images une par une, de manière séquentielle
* Fonctionnement monothread
* Faible consommation de mémoire

**La barre de progression affiche 2 étapes :**

1.**Détection des cibles** - Recherche des cibles d’étalonnage
2. **Traitement** - Application de l’étalonnage et exportation des images**Durée de traitement :**

* Beaucoup plus lent que le mode parallèle Chloros+
* Adapté aux ensembles de données de petite à moyenne taille (&lt; 200 images)

### Mode Chloros+ (Traitement parallèle)

**Nécessite une licence Chloros+**

**Fonctionnement :**

* Traite plusieurs images simultanément à l’aide d’un [pipeline de traitement à 4 threads](../processing-architecture/processing-pipeline.md)
* L’[adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md) sélectionne automatiquement la stratégie optimale pour votre matériel au démarrage de l’exécution
* Accélération GPU (CUDA) avec les cartes graphiques NVIDIA (ordinateurs de bureau et Jetson)
* **Le nombre de workers s’adapte au matériel**: les stratégies GPU exécutent**1 à 4 workers simultanés** (en fonction de la VRAM — un Jetson à faible mémoire en exécute 1, un GPU de bureau de 12 Go ou plus en exécute jusqu’à 4) ; les systèmes exclusivement CPU exécutent un worker par cœur physique, moins un**La barre de progression affiche 4 étapes** (correspondant aux 4 threads du pipeline) :

1. **Détection** (Thread 1) – Recherche des cibles d’étalonnage
2. **Analyse** (Thread 2) – Examen des métadonnées de l’image et calcul de l’étalonnage
3. **Calibrage** (thread 3) – Débayérisation, correction de la vignettage, calibrage, calcul de l’indice
4. **Exportation** (thread 4) – Enregistrement des images traitées et des indices**Interaction avec la barre de progression :*** **Passez la souris** sur la barre pour afficher un menu déroulant détaillé en 4 étapes
* **Cliquez** sur la barre de progression pour figer le menu déroulant à l&#x27;écran
* **Cliquez à nouveau** pour débloquer et masquer le menu**Temps de traitement :**

* Nettement plus rapide que le mode gratuit
* L’accélération GPU améliore encore la vitesse

{% hint style="info" %}
**Chloros+ Vitesse** : Le traitement parallèle peut être 5 à 10 fois plus rapide que le mode séquentiel pour les grands ensembles de données. Un projet de 500 images qui prend 2 heures en mode gratuit peut être terminé en 15 à 20 minutes avec Chloros+.
{% endhint %}

***

## Que se passe-t-il pendant le traitement ?

### Étape 1 : Détection des cibles

**Fonctionnalités de Chloros :**

* Analyse les images que vous avez cochées dans la colonne « Cible » (toutes les images si aucune n’est cochée)
* Identifie les panneaux d’étalonnage dans chaque cible
* Extrait les valeurs de réflectance des panneaux cibles
* Enregistre les horodatages des cibles pour la planification de l&#x27;étalonnage

**Durée :** 1 à 30 secondes (avec des cibles marquées), 5 à 30 minutes ou plus (sans cibles marquées)

### Étape 2 : Débayérisation (Conversion RAW)

**Fonctionnalités de Chloros :**

* Convertit les données RAW au motif de Bayer en images complètes à 3 canaux (les modules mono LATTICE restent monobandes — le débayérage est ignoré pour ceux-ci, avec une note dans le journal)
* Applique l’algorithme de dématriçage sélectionné
* Préserve au maximum la qualité et les détails de l’image

**Durée :** varie en fonction du nombre d’images et de la vitesse du CPU/GPU

### Étape 3 : Étalonnage

**Fonctionnalités de Chloros :*** **Correction du vignettage** : supprime l’assombrissement des bords dû à l’objectif
* **Étalonnage de la réflectance** : normalise les valeurs à l’aide de valeurs de réflectance cibles et/ou de données de rayonnement descendant issues de l’acquisition de données (DAQ)
* Applique les corrections à toutes les bandes/canaux
* Utilise la référence d’étalonnage appropriée pour chaque image en fonction de l’horodatage

**Durée :** La majeure partie du temps de traitement

### Étape 4 : Calcul des indices

**Fonctionnalités de Chloros :**

* Calcule les indices multispectraux configurés (NDVI, NDRE, etc.)
* Applique des opérations mathématiques sur les bandes aux images calibrées
* Génère des images d’indice pour chaque indice sélectionné

**Durée :** Quelques secondes par image

### Étape 5 : Exportation

**Fonctionnalités de Chloros :**

* Enregistre les images traitées dans le format sélectionné
* **Fan-out LATTICE** : chaque image brute LATTICE est exportée sous la forme de tous les produits activés en une seule passe — débayérisation, aperçu, radiance (toujours en float32), réflectance
* Écrit les fichiers dans l’arborescence de sortie du projet : `<project>/<camera>/<format>/<Product>_Images/`
* **Conserve le nom de fichier source** — le dossier identifie le produit, aucun suffixe n’est ajouté**Durée :** varie en fonction du format d’exportation et de la taille des fichiers***

## Comportement du traitement

### Pipeline de traitement automatique

Une fois lancé, l’ensemble du pipeline s’exécute automatiquement :

* Aucune interaction de l’utilisateur n’est nécessaire
* Toutes les étapes configurées s’exécutent dans l’ordre
* Les mises à jour de progression s’affichent en temps réel
* Les fichiers exportés sont enregistrés sur le disque au fur et à mesure de leur finalisation — vous pouvez ouvrir les fichiers finis pendant que le traitement se poursuit

### Utilisation de l’ordinateur pendant le traitement

**Mode libre :**

* Utilisation du processeur relativement faible (monothread)
* L’ordinateur reste réactif pour d’autres tâches
* Vous pouvez sans risque réduire Chloros en arrière-plan et travailler dans d’autres applications

**Chloros+ Mode parallèle :**

* Utilisation élevée du processeur au sein du pool de travailleurs de la stratégie
* Avec accélération GPU : utilisation élevée du GPU
* L&#x27;ordinateur peut être moins réactif pendant le traitement
* Évitez de lancer d&#x27;autres tâches gourmandes en ressources processeur

{% hint style="warning" %}
**Conseil de performance** : pour optimiser les performances de Chloros+, fermez les autres applications et laissez Chloros utiliser pleinement les ressources du système.
{% endhint %}

### Le traitement ne peut pas être mis en pause (mais l’arrêt est propre)

* Une fois lancé, le traitement ne peut pas être mis en pause puis repris ultérieurement
* Cliquer sur **Arrêter** interrompt proprement l&#x27;exécution dès le premier clic
* Les produits déjà exportés avant l&#x27;arrêt restent sur le disque
* Une exécution arrêtée rend compte fidèlement de ce qu&#x27;elle a accompli (voir les lignes `[RUN-SUMMARY]` dans le journal)
* Un nouveau traitement relance le pipeline depuis le début

**Conseil de planification :** pour les très gros projets, envisagez de traiter par lots ou d&#x27;utiliser le fichier CLI pour un meilleur contrôle.***

## Suivi de votre traitement

Pendant l&#x27;exécution du traitement, vous pouvez :

* **Suivre la barre de progression** – Voir le pourcentage global d’avancement
* **Afficher l’étape en cours** – Détection, analyse, étalonnage ou exportation
* **Consulter l’onglet « Journal »** – Voir les messages détaillés de traitement et les avertissements
* **Prévisualiser les images finalisées** – Les fichiers d’exportation apparaissent sur le disque pendant le traitement

Pour plus d’informations sur la surveillance, consultez [Surveillance du traitement](monitoring-the-processing.md).

***

## Arrêt du traitement

Si vous devez arrêter le traitement :

### Comment arrêter

1. Repérez le **bouton Arrêter** (qui remplace le bouton « Démarrer » pendant le traitement)
2. Cliquez une fois dessus — la barre affiche **« Arrêt en cours... »** pendant que l’image en cours de traitement se termine
3. Le traitement se termine définitivement à l’arrêt et le journal affiche un rapport `[RUN-SUMMARY]` détaillé des tâches effectuées

### Quand interrompre le traitement

**Raisons valables pour interrompre le traitement :**

* Vous vous êtes rendu compte que des paramètres incorrects ont été utilisés
* Vous avez oublié de marquer les images cibles
* Vous avez importé les mauvaises images
* Le système fonctionne trop lentement ou ne répond plus

**Après l&#x27;interruption :**

* Les produits exportés avant l&#x27;interruption restent sur le disque
* Vérifiez et corrigez les éventuels problèmes, ajustez les paramètres si nécessaire
* Redémarrez le traitement — l’exécution recommence depuis le début

***

## Estimations du temps de traitement

Le temps de traitement réel varie considérablement en fonction des éléments suivants :

* Nombre d’images
* Résolution des images
* Format d’entrée RAW ou JPG
* Mode de traitement (version gratuite ou Chloros+)
* Vitesse du processeur et nombre de cœurs
* Disponibilité d’un GPU (Chloros+ uniquement)
* Nombre d’index à calculer
* Nombre de produits d’exportation activés (LATTICE)

### Estimations approximatives (Chloros+, images de 12 MP, processeur moderne)

| Nombre d’images | Mode gratuit | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 images   | 15 à 20 min | 5 à 8 min        | 3 à 5 min        |
| 100 images  | 30 à 40 min | 10 à 15 min      | 5 à 8 min        |
| 200 images  | 1 à 1,5 h | 20 à 30 min      | 10 à 15 min      |
| 500 images  | 2 à 3 h   | 45 à 60 min      | 20 à 30 min      |
| 1 000 images | 4 à 6 heures   | 1,5 à 2 heures      | 40 à 60 min      |

{% hint style="info" %}
**Premier lancement** : le traitement initial peut prendre plus de temps, car Chloros crée des caches et des profils. Les traitements suivants de jeux de données similaires seront plus rapides.
{% endhint %}

***

## Problèmes courants au démarrage

### Bouton « Démarrer » désactivé (grisé)

**Causes possibles :**

* Aucune image importée
* Le backend n’est pas entièrement démarré
* Le traitement précédent est toujours en cours
* Le projet n’est pas entièrement chargé

**Solutions :**

1. Attendez que le backend s’initialise complètement (vérifiez l’icône du menu principal)
2. Vérifiez que les images sont bien importées dans l’explorateur de fichiers
3. Redémarrez Chloros si le bouton reste désactivé
4. Consultez le journal de débogage pour rechercher d’éventuels messages d’erreur

### Le traitement démarre puis échoue immédiatement

**Causes possibles :**

* Aucune image valide dans le projet
* Fichiers image corrompus
* Espace disque insuffisant
* Mémoire insuffisante (RAM)

**Solutions :**

1. Vérifiez le journal de débogage <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pour voir s’il contient des messages d’erreur
2. Vérifiez l’espace disque disponible
3. Essayez de traiter un sous-ensemble plus petit d’images
4. Vérifiez que les images ne sont pas corrompues

### L&#x27;exécution se termine mais n&#x27;écrit aucune image

Une exécution qui a demandé des produits d&#x27;imagerie mais n&#x27;en a écrit aucun est considérée comme un **échec, et non comme une réussite** — Chloros le signale clairement :

* Le journal de l’interface graphique affiche des messages `[RUN-SUMMARY]` indiquant la cause probable : aucune image importée, aucune cible détectée, ou tous les produits demandés ignorés car inapplicables (par exemple, demande de radiance/réflectance à partir de caméras RGB uniquement)
* L’équivalent de CLI (`chloros-cli process`) affiche `Processing finished but wrote no image products.` et **se termine avec un code de sortie différent de zéro**, ce qui permet aux scripts de le détecter
* Une exécution délibérée limitée aux métadonnées (tous les produits d&#x27;exportation désactivés, pas d&#x27;index) est tout de même considérée comme réussie

Consultez [la référence CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) pour connaître la sémantique complète.

### Avertissement « Aucune cible détectée »

**Causes possibles :**

* Oubli de marquer les images cibles
* Les images cibles ne contiennent pas de cibles visibles
* Paramètres de détection des cibles trop stricts

**Solutions :**

1. Consultez la section [Choix des images cibles](choosing-target-images.md)
2. Marquez les images appropriées dans la colonne « Cible »
3. Vérifiez que les cibles sont visibles dans les images marquées
4. Ajustez les paramètres de détection des cibles si nécessaire

***

## Conseils pour un traitement réussi

### Avant de commencer

1. **Effectuez d’abord un test sur un petit sous-ensemble** - Traitez 10 à 20 images pour vérifier les paramètres
2. **Vérifiez l’espace disque disponible** - Assurez-vous de disposer d’un espace libre équivalent à 2 à 3 fois la taille de l’ensemble de données (davantage si tous les produits LATTICE sont activés)
3. **Fermez les applications inutiles** - Libérez des ressources système
4. **Vérifiez les images des cibles** — Prévisualisez les cibles marquées pour vous assurer de leur qualité
5. **Enregistrez le projet** — Le projet s’enregistre automatiquement, mais il est recommandé de l’enregistrer manuellement

### Pendant le traitement

1. **Évitez la mise en veille du système** - Désactivez les modes d’économie d’énergie
2. **Gardez Chloros au premier plan** - Ou au moins visible dans la barre des tâches
3. **Surveillez la progression de temps en temps** - Vérifiez s’il y a des avertissements ou des erreurs
4. **Ne lancez pas d’autres applications gourmandes en ressources** – En particulier avec le mode parallèle Chloros+

### Chloros+ Accélération GPU

Si vous utilisez l’accélération GPU NVIDIA :

1. Mettez à jour les pilotes NVIDIA vers la dernière version
2. Assurez-vous que le GPU dispose d’au moins 4 Go de VRAM (7 Go ou plus pour le débayering simultané avec prise en charge des textures)
3. Fermez les applications gourmandes en ressources GPU (jeux, montage vidéo)
4. Surveillez la température du GPU (assurez-vous que le refroidissement est suffisant)

***

## Étapes suivantes

Une fois le traitement lancé :

1. **Suivez la progression** - Voir [Suivi du traitement](monitoring-the-processing.md)
2. **Attendez la fin du traitement** - Le traitement s&#x27;exécute automatiquement
3. **Vérifiez les résultats** - Voir [Finalisation du traitement](finishing-the-processing.md)

Pour savoir comment procéder pendant le traitement, consultez [Surveillance du traitement](monitoring-the-processing.md).
