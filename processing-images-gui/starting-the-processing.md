# Lancement du traitement

Une fois que vous avez importé vos images, marqué vos cibles d'étalonnage et configuré les paramètres de votre projet, vous êtes prêt à commencer le traitement. Cette page vous guide dans le lancement du pipeline de traitement d'Chloros.

## Liste de contrôle avant le traitement

Avant de cliquer sur le bouton « Démarrer », vérifiez que tout est prêt :

* [ ] **Fichiers importés** - Toutes les images apparaissent dans l'explorateur de fichiers
* [ ] **Images cibles marquées** - Colonne « Cible » cochée pour les images d'étalonnage (ou un enregistrement `.daq` importé pour LATTICE)
* [ ] **Modèles de caméras détectés** – La colonne « Modèle de caméra » affiche les caméras correctes
* [ ] **Paramètres configurés** – Paramètres du projet vérifiés et ajustés
* [ ] **Indices sélectionnés** – Indices multispectraux souhaités ajoutés (si nécessaire)
* [ ] **Format d’exportation choisi** - Format de sortie adapté à votre flux de travail

{% hint style="info" %}
**Astuce** : cliquez sur quelques images dans le navigateur de fichiers pour vérifier qu’elles se sont chargées correctement avant le traitement.
{% endhint %}

***

## Lancer le traitement

### Localiser le bouton Démarrer

Le bouton Démarrer/Lecture se trouve dans la barre d'en-tête supérieure d'Chloros :

* Emplacement : en haut au centre de la fenêtre
* Icône : **bouton Lecture/Démarrer** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* État : le bouton est activé (allumé) lorsqu'il est prêt à traiter

### Cliquez pour démarrer

1. Cliquez sur le **bouton Lecture/Démarrer** dans la barre d'en-tête supérieure
2. Le traitement commence immédiatement
3. Le bouton devient un bouton **Arrêt** pendant le traitement
4. La barre de progression s'actualise, indiquant l'état du traitement

{% hint style="success" %}
**Traitement lancé** : une fois que vous avez cliqué, Chloros gère automatiquement toutes les étapes du traitement : détection des cibles, débayérisation, étalonnage, calcul de l’indice et exportation. Il détecte automatiquement si votre projet est de type Survey3, LATTICE ou un mélange des deux, et applique le pipeline adapté à chaque caméra.
{% endhint %}

***

## Comprendre les modes de traitement

Chloros fonctionne selon deux modes de traitement différents en fonction de votre licence :

### Mode gratuit (traitement séquentiel)

**Disponible pour tous les utilisateurs**

**Fonctionnement :**

* Traite les images une par une, de manière séquentielle
* Fonctionnement monothread
* Faible consommation de mémoire

**La barre de progression affiche 2 étapes :**

1.**Détection des cibles** - Recherche des cibles d’étalonnage
2. **Traitement** - Application de l’étalonnage et exportation des images**Durée de traitement :**

* Beaucoup plus lent que le mode parallèle d’Chloros+
* Convient aux ensembles de données de petite à moyenne taille (< 200 images)

### Mode Chloros+ (traitement parallèle)

**Nécessite une licence Chloros+**

**Fonctionnement :**

* Traite plusieurs images simultanément à l’aide d’un [pipeline de traitement à 4 threads](../processing-architecture/processing-pipeline.md)
* L’[adaptation dynamique des calculs](../processing-architecture/dynamic-compute-adaptation.md) sélectionne automatiquement la stratégie optimale pour votre matériel au démarrage
* Accélération GPU (CUDA) avec les cartes graphiques NVIDIA
* **Le nombre de workers s'adapte au matériel**: les stratégies GPU exécutent**2 à 4 workers simultanés** (en fonction de la VRAM — 3 sur un GPU de 2 à 12 Go, 2 à 4 sur un GPU de 12 Go ou plus) ; les systèmes équipés uniquement d’un CPU exécutent un worker par cœur physique, moins un**La barre de progression affiche 4 étapes** (correspondant aux 4 threads du pipeline) :

1. **Détection** (Thread 1) – Recherche des cibles d’étalonnage
2. **Analyse** (thread 2) – Examen des métadonnées de l’image et calcul de l’étalonnage
3. **Étalonnage** (thread 3) – Débayérisation, correction du vignettage, étalonnage, calcul de l’indice
4. **Exportation** (thread 4) – Enregistrement des images traitées et des indices**Interaction avec la barre de progression :*** **Passez la souris** sur la barre pour afficher le menu déroulant détaillé des 4 étapes
* **Cliquez** sur la barre de progression pour figer le menu déroulant à l'écran
* **Cliquez à nouveau** pour débloquer et masquer le menu**Temps de traitement :**

* Nettement plus rapide que le mode gratuit
* L'accélération GPU améliore encore la vitesse

{% hint style="info" %}
**Vitesse de Chloros+** : le traitement parallèle peut être 5 à 10 fois plus rapide que le mode séquentiel pour les grands ensembles de données. Un projet de 500 images qui prend 2 heures en mode gratuit peut être terminé en 15 à 20 minutes avec Chloros+.
{% endhint %}

***

## Que se passe-t-il pendant le traitement ?

### Étape 1 : Détection des cibles

**Ce que fait Chloros :**

* Analyse les images que vous avez cochées dans la colonne « Cible » (toutes les images si aucune n’est cochée)
* Identifie les panneaux d’étalonnage dans chaque cible
* Extrait les valeurs de réflectance des panneaux cibles
* Enregistre les horodatages des cibles pour la planification de l’étalonnage

**Durée :** 1 à 30 secondes (avec des cibles marquées), 5 à 30 minutes ou plus (sans cibles marquées)

### Étape 2 : Débayérisation (conversion RAW)

**Ce que fait Chloros :**

* Convertit les données RAW au format Bayer en images complètes à 3 canaux (les modules mono LATTICE restent monobandes — le débayérage est ignoré pour ceux-ci, avec une note dans le journal)
* Applique l’algorithme de dématriçage sélectionné
* Préserve au maximum la qualité et les détails de l’image

**Durée :** Varie en fonction du nombre d’images et de la vitesse du CPU/GPU

### Étape 3 : Étalonnage

**Ce que fait Chloros :*** **Correction du vignetage** : Supprime l’assombrissement des bords dû à l’objectif
* **Étalonnage de la réflectance** : normalise à l’aide des valeurs de réflectance cibles et/ou des données de rayonnement descendant issues de l’acquisition de données (DAQ)
* Applique les corrections à toutes les bandes/canaux
* Utilise une référence d’étalonnage appropriée pour chaque image en fonction de l’horodatage

**Durée :** la majeure partie du temps de traitement

### Étape 4 : Calcul des indices

**Ce que fait Chloros :**

* Calcule les indices multispectraux configurés (NDVI, NDRE, etc.)
* Applique des opérations mathématiques sur les bandes aux images étalonnées
* Génère des images d’indice pour chaque indice sélectionné

**Durée :** Quelques secondes par image

### Étape 5 : Exportation

**Ce que fait Chloros :**

* Enregistre les images traitées dans le format sélectionné
* **Fan-out LATTICE** : chaque image brute LATTICE est exportée sous la forme de tous les produits activés en une seule passe — débayérisation, aperçu, radiance (toujours en float32), réflectance
* Enregistre les fichiers dans l’arborescence de sortie du projet : `<project>/<camera>/<format>/<Product>_Images/`
* **Conserve le nom de fichier source** — le dossier identifie le produit, aucun suffixe n’est ajouté**Durée :** varie en fonction du format d’exportation et de la taille des fichiers***

## Comportement du traitement

### Pipeline de traitement automatique

Une fois lancé, l’ensemble du pipeline s’exécute automatiquement :

* Aucune interaction de l’utilisateur n’est nécessaire
* Toutes les étapes configurées s’exécutent dans l’ordre
* Les mises à jour de progression s’affichent en temps réel
* Les fichiers exportés sont enregistrés sur le disque au fur et à mesure de leur achèvement — vous pouvez ouvrir les fichiers finis pendant que le traitement se poursuit

### Utilisation de l’ordinateur pendant le traitement

**Mode Free :**

* Utilisation du processeur relativement faible (monothread)
* L’ordinateur reste réactif pour d’autres tâches
* Vous pouvez minimiser Chloros et travailler dans d’autres applications en toute sécurité

**Mode parallèle Chloros+ :**

* Utilisation élevée du processeur au sein du pool de travailleurs de la stratégie
* Avec accélération GPU : utilisation élevée du GPU
* L'ordinateur peut être moins réactif pendant le traitement
* Évitez de lancer d'autres tâches gourmandes en ressources processeur

{% hint style="warning" %}
**Conseil de performance** : pour optimiser les performances de Chloros+, fermez les autres applications et laissez Chloros utiliser toutes les ressources du système.
{% endhint %}

### Le traitement ne peut pas être mis en pause (mais l'arrêt est propre)

* Une fois lancé, le traitement ne peut pas être mis en pause puis repris ultérieurement
* Cliquer sur **Arrêter** interrompt proprement l’exécution dès le premier clic
* Les produits déjà exportés avant l’arrêt restent sur le disque
* Une exécution arrêtée rend compte fidèlement de ce qu’elle a accompli (voir les lignes `[RUN-SUMMARY]` dans le journal)
* Un nouveau traitement relance le pipeline depuis le début

**Conseil de planification :** pour les très grands projets, envisagez de traiter par lots ou d’utiliser l’interface en ligne de commande (CLI) pour un meilleur contrôle.

***

## Suivi de votre traitement

Pendant l’exécution du traitement, vous pouvez :

* **Suivre la barre de progression** – Voir le pourcentage global d’avancement
* **Afficher l’étape en cours** – Détection, analyse, étalonnage ou exportation
* **Consulter l’onglet « Journal »** – Voir les messages détaillés de traitement et les avertissements
* **Prévisualiser les images finalisées** – Les fichiers exportés apparaissent sur le disque pendant le traitement

Pour plus d’informations sur la surveillance, consultez [Surveillance du traitement](monitoring-the-processing.md).

***

## Arrêt du traitement

Si vous devez arrêter le traitement :

### Comment arrêter

1. Repérez le **bouton Arrêter** (qui remplace le bouton « Démarrer » pendant le traitement)
2. Cliquez une fois dessus — la barre affiche **« Arrêt en cours... »** pendant que l’image en cours de traitement est finalisée
3. Le traitement se termine dans un état d’arrêt définitif et le journal affiche un rapport `[RUN-SUMMARY]` détaillé de ce qui a été effectué

### Quand interrompre le traitement

**Motifs valables pour interrompre le traitement :**

* Vous vous êtes rendu compte que des paramètres incorrects avaient été utilisés
* Vous avez oublié de marquer les images cibles
* Vous avez importé les mauvaises images
* Le système fonctionne trop lentement ou ne répond plus

**Après l’interruption :**

* Les produits exportés avant l’interruption restent sur le disque
* Vérifiez et corrigez les éventuels problèmes, ajustez les paramètres si nécessaire
* Redémarrez le traitement — la session recommence depuis le début

***

## Estimations du temps de traitement

Le temps de traitement réel varie considérablement en fonction :

* Du nombre d’images
* De la résolution des images
* Du format d’entrée (RAW ou JPG)
* Du mode de traitement (Mode gratuit ou Chloros+)
* Vitesse du processeur et nombre de cœurs
* Disponibilité d’un GPU (Chloros+ uniquement)
* Nombre d’indices à calculer
* Nombre de produits d’exportation activés (LATTICE)

### Estimations approximatives (Chloros+, images de 12 MP, processeur moderne)

| Nombre d'images | Mode gratuit | Chloros+ (processeur) | Chloros+ (carte graphique) |
| ----------- | --------- | -------------- | -------------- |
| 50 images   | 15 à 20 min | 5 à 8 min        | 3 à 5 min        |
| 100 images  | 30 à 40 min | 10 à 15 min      | 5 à 8 min        |
| 200 images  | 1 à 1,5 h | 20 à 30 min      | 10 à 15 min      |
| 500 images  | 2 à 3 h   | 45 à 60 min      | 20 à 30 min      |
| 1 000 images | 4 à 6 heures   | 1,5 à 2 heures      | 40 à 60 min      |

{% hint style="info" %}
**Premier lancement** : le traitement initial peut prendre plus de temps, le temps que Chloros crée ses caches et ses profils. Les traitements suivants de jeux de données similaires seront plus rapides.
{% endhint %}

***

## Problèmes courants au démarrage

### Bouton « Démarrer » désactivé (grisé)

**Causes possibles :**

* Aucune image importée
* Backend pas entièrement démarré
* Traitement précédent toujours en cours
* Projet pas entièrement chargé

**Solutions :**

1. Attendez que le backend soit entièrement initialisé (vérifiez l’icône du menu principal)
2. Vérifiez que les images sont bien importées dans l’explorateur de fichiers
3. Redémarrez Chloros si le bouton reste désactivé
4. Consultez le journal de débogage pour rechercher des messages d’erreur

### Le traitement démarre puis échoue immédiatement

**Causes possibles :**

* Aucune image valide dans le projet
* Fichiers image corrompus
* Espace disque insuffisant
* Mémoire insuffisante (RAM)

**Solutions :**

1. Vérifiez le journal de débogage <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pour voir s’il contient des messages d’erreur
2. Vérifiez l’espace disque disponible
3. Essayez de traiter un sous-ensemble plus restreint d’images
4. Vérifiez que les images ne sont pas corrompues

### L’exécution se termine mais n’écrit aucune image

Une exécution qui a demandé des produits d’image mais n’en a écrit aucun est considérée comme un **échec, et non comme une réussite** — Chloros le signale clairement :

* Le journal de l’interface graphique affiche des messages `[RUN-SUMMARY]` indiquant la cause probable — aucune image importée, aucune cible détectée, ou tous les produits demandés ignorés car inapplicables (par exemple, demande de radiance/réflectance à partir de caméras RGB uniquement)
* L’équivalent en CLI (`chloros-cli process`) affiche `Processing finished but wrote no image products.` et **se termine avec un code de sortie différent de zéro**, ce qui permet aux scripts de le détecter
* Une exécution délibérée portant uniquement sur les métadonnées (tous les produits d’exportation désactivés, pas d’index) est tout de même considérée comme réussie

Consultez [la référence de la CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) pour connaître la sémantique complète.

### Le dossier du projet est trop profond pour cet ordinateur

**Windows uniquement, nouveauté de la version 1.3.0.** Windows est livré avec les chemins d’accès longs désactivés ; aucun chemin d’accès ne peut alors dépasser 260 caractères. Les produits Chloros se trouvent jusqu’à environ 130 caractères en dessous du dossier de projet ; ainsi, un dossier de projet situé plus en profondeur ne pourrait pas recevoir ses produits. Au lieu de s’exécuter sans rien écrire, Chloros effectue désormais une vérification avant le début du traitement :

* Cliquer sur « Démarrer » ne lance pas l’exécution : la barre de progression affiche la raison, et le journal de débogage indique `[PROCESS] refused: The project folder is too deep for this computer…`. Le message précise la longueur du dossier de projet et le dossier de projet le plus long qui peut y être placé.
* La création ou l’ouverture d’un projet dans un dossier trop profond, même pour les propres fichiers du projet, est également refusée. Une boîte de message explique pourquoi, en indiquant le nom du dossier de projet le plus long pouvant être pris en charge et comment activer les chemins d’accès longs ; rien n’est créé ni ouvert ; le journal de débogage enregistre cet événement sous le code `[PROJECT] refused: …`. La duplication d’un projet sous un nom plus long peut également dépasser la limite : la copie est effectuée mais le projet ne s’ouvre pas, et le même message en explique la raison.

**Solutions :**

1. Placez les projets dans un dossier plus court — modifiez le**répertoire de travail** dans [Paramètres du projet](../project-settings/project-settings.md) et recréez le projet, ou
2. Activez les chemins d’accès longs sous Windows et redémarrez Chloros : relancez le programme d’installation de Chloros en cochant la case **Activer les chemins d’accès longs sous Windows** (celle-ci est cochée par défaut), ou utilisez la stratégie de groupe : *Configuration de l’ordinateur > Modèles d’administration > Système > Système de fichiers > Activer les chemins d’accès longs Win32*.

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

1. **Effectuez d’abord un test avec un petit sous-ensemble** - Traitez 10 à 20 images pour vérifier les paramètres
2. **Vérifiez l’espace disque disponible** - Assurez-vous de disposer d’un espace libre équivalent à 2 à 3 fois la taille de l’ensemble de données (davantage si tous les produits LATTICE sont activés)
3. **Fermez les applications inutiles** – Libérez des ressources système
4. **Vérifiez les images cibles** – Prévisualisez les cibles marquées pour vous assurer de leur qualité
5. **Enregistrez le projet** – Le projet s’enregistre automatiquement, mais il est recommandé de l’enregistrer manuellement

### Pendant le traitement

1. **Évitez la mise en veille du système** - Désactivez les modes d’économie d’énergie
2. **Gardez Chloros au premier plan** - Ou au moins visible dans la barre des tâches
3. **Surveillez la progression de temps en temps** - Vérifiez s’il y a des avertissements ou des erreurs
4. **Ne lancez pas d’autres applications gourmandes en ressources** - Surtout en mode parallèle Chloros+

### Accélération GPU d’Chloros+

Si vous utilisez l’accélération GPU NVIDIA :

1. Mettez à jour les pilotes NVIDIA vers la dernière version
2. Assurez-vous que le GPU dispose d’au moins 4 Go de VRAM (7 Go ou plus pour le débayering Texture Aware simultané)
3. Fermez les applications gourmandes en ressources GPU (jeux, montage vidéo)
4. Surveillez la température du GPU (assurez-vous que le refroidissement est suffisant)

***

## Étapes suivantes

Une fois le traitement lancé :

1. **Surveillez la progression** - Voir [Surveillance du traitement](monitoring-the-processing.md)
2. **Attendez la fin du traitement** - Le traitement s'exécute automatiquement
3. **Vérifiez les résultats** – Voir [Fin du traitement](finishing-the-processing.md)

Pour savoir comment procéder pendant le traitement, consultez [Surveillance du traitement](monitoring-the-processing.md).
