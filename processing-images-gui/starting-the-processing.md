# Démarrage du traitement

Une fois que vous avez importé vos images, marqué vos cibles d&#x27;étalonnage et configuré les paramètres de votre projet, vous êtes prêt à commencer le traitement. Cette page vous guide dans le lancement du pipeline de traitement Chloros.

## Liste de contrôle avant le traitement

Avant de cliquer sur le bouton Démarrer, vérifiez que tout est prêt :

* [ ] **Fichiers importés** - Toutes les images apparaissent dans le navigateur de fichiers
* [ ] **Images cibles marquées** - Colonne Cible cochée pour les images d&#x27;étalonnage
* [ ] **Modèles d&#x27;appareils photo détectés** - La colonne Modèle d&#x27;appareil photo affiche les appareils photo corrects
* [ ] **Paramètres configurés** - Paramètres du projet vérifiés et ajustés
* [ ] **Indices sélectionnés** - Indices multispectraux souhaités ajoutés (si nécessaire)
* [ ] **Format d&#x27;exportation choisi** - Format de sortie adapté à votre flux de travail

{% hint style=&quot;info&quot; %}
**Conseil** : cliquez sur quelques images dans le navigateur de fichiers pour vérifier qu&#x27;elles se sont chargées correctement avant le traitement.
{% endhint %}

***

## Démarrer le traitement

### Localiser le bouton Démarrer

Le bouton Démarrer/Lecture se trouve dans la barre d&#x27;en-tête supérieure de Chloros :

* Position : en haut au centre de la fenêtre
* Icône : **bouton Lecture/Démarrer** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Statut : le bouton est activé (lumineux) lorsqu&#x27;il est prêt à traiter

### Cliquez pour démarrer

1. Cliquez sur le **bouton Lecture/Démarrer** dans l&#x27;en-tête supérieur
2. Le traitement commence immédiatement
3. Le bouton est désactivé (grisé) pendant le traitement
4. La barre de progression s&#x27;actualise, indiquant l&#x27;état du traitement

{% hint style=&quot;success&quot; %}
**Traitement lancé** : une fois cliqué, Chloros gère automatiquement toutes les étapes du traitement : détection de la cible, débayérisation, calibrage, calcul de l&#x27;indice et exportation.
{% endhint %}

***

## Comprendre les modes de traitement

Chloros fonctionne selon deux modes de traitement différents en fonction de votre licence :

### Mode gratuit (traitement séquentiel)

**Disponible pour tous les utilisateurs**

**Fonctionnement :**

* Traite les images une par une, de manière séquentielle.
* Fonctionnement à thread unique.
* Utilisation réduite de la mémoire.

**La barre de progression affiche 2 étapes :**

1. **Détection de la cible** - Recherche des cibles d&#x27;étalonnage.
2. **Traitement** - Application de l&#x27;étalonnage et exportation des images.

**Durée du traitement :**

* Beaucoup plus lent que le mode parallèle Chloros+
* Convient aux ensembles de données de petite à moyenne taille (&lt; 200 images)

### Mode Chloros+ (traitement parallèle)

**Nécessite une licence Chloros+**

**Fonctionnement :**

* Traite plusieurs images simultanément
* Fonctionnement multithread (jusqu&#x27;à 16 processus parallèles)
* Utilise plusieurs cœurs de processeur
* Accélération GPU (CUDA) en option avec les cartes graphiques NVIDIA

**La barre de progression affiche 4 étapes :**

1. **Détection** - Recherche des cibles d&#x27;étalonnage
2. **Analyse** - Examen des métadonnées de l&#x27;image et préparation du pipeline
3. **Calibrage** - Application des corrections et des calibrages
4. **Exportation** - Enregistrement des images et des index traités

**Interaction avec la barre de progression :**

* **Passez la souris** sur la barre pour afficher le panneau déroulant détaillé en 4 étapes
* **Cliquez** sur la barre de progression pour figer le panneau déroulant
* **Cliquez à nouveau** pour débloquer et masquer le panneau

**Temps de traitement :**

* Nettement plus rapide que le mode libre
* Évolutif en fonction du nombre de cœurs du processeur
* L&#x27;accélération GPU améliore encore la vitesse

{% hint style=&quot;info&quot; %}
**Chloros+ Vitesse** : le traitement parallèle peut être 5 à 10 fois plus rapide que le mode séquentiel pour les grands ensembles de données. Un projet de 500 images qui prend 2 heures en mode gratuit peut être réalisé en 15 à 20 minutes avec Chloros+.
{% endhint %}

***

## Que se passe-t-il pendant le traitement ?

### Étape 1 : Détection des cibles

**Ce que fait Chloros :**

* Scanne les images cibles marquées (ou toutes les images si aucune n&#x27;est marquée)
* Identifie les 4 panneaux d&#x27;étalonnage dans chaque cible
* Extrait les valeurs de réflectance des panneaux cibles
* Enregistre les horodatages des cibles pour la planification de l&#x27;étalonnage

**Durée :** 1 à 30 secondes (avec cibles marquées), 5 à 30 minutes ou plus (sans cibles marquées)

### Étape 2 : Débayérisation (conversion RAW)

**Ce que fait Chloros :**

* Convertit les données RAW du motif Bayer en images RGB complètes
* Applique un algorithme de démosaïquage de haute qualité
* Préserve au maximum la qualité et les détails de l&#x27;image

**Durée :** varie en fonction du nombre d&#x27;images et de la vitesse du processeur

### Étape 3 : Calibrage

**Ce que fait Chloros :**

* **Correction du vignettage** : supprime l&#x27;assombrissement des bords de l&#x27;objectif
* **Calibrage de la réflectance** : normalise à l&#x27;aide des valeurs de réflectance cibles
* Applique des corrections sur toutes les bandes/canaux
* Utilise une cible de calibrage appropriée pour chaque image en fonction de l&#x27;horodatage

**Durée :** la majeure partie du temps de traitement

### Étape 4 : Calcul de l&#x27;indice

**Ce que fait Chloros :**

* Calcule les indices multispectraux configurés (NDVI, NDRE, etc.)
* Applique des calculs mathématiques aux images calibrées
* Génère des images d&#x27;indice pour chaque indice sélectionné

**Durée :** Quelques secondes par image

### Étape 5 : Exportation

**Ce que fait Chloros :**

* Enregistre les images calibrées dans le format sélectionné
* Exporte les images d&#x27;indice avec les couleurs LUT configurées
* Écrit les fichiers dans les sous-dossiers du modèle d&#x27;appareil photo
* Conserve les noms de fichiers d&#x27;origine avec les suffixes

**Durée :** varie en fonction du format d&#x27;exportation et de la taille du fichier

***

## Comportement du traitement

### Pipeline de traitement automatique

Une fois lancé, l&#x27;ensemble du pipeline s&#x27;exécute automatiquement :

* Aucune interaction de l&#x27;utilisateur n&#x27;est nécessaire
* Toutes les étapes configurées s&#x27;exécutent dans l&#x27;ordre
* Les mises à jour de la progression s&#x27;affichent en temps réel

### Utilisation de l&#x27;ordinateur pendant le traitement

**Mode libre :**

* Utilisation relativement faible du processeur (monothread)
* L&#x27;ordinateur reste réactif pour d&#x27;autres tâches
* Vous pouvez minimiser Chloros et travailler dans d&#x27;autres applications en toute sécurité

**Chloros+ Mode parallèle :**

* Utilisation élevée du processeur (multithread, jusqu&#x27;à 16 cœurs)
* Avec accélération GPU : utilisation élevée du GPU
* L&#x27;ordinateur peut être moins réactif pendant le traitement
* Évitez de lancer d&#x27;autres tâches gourmandes en ressources processeur

{% hint style=&quot;warning&quot; %}
**Conseil de performance** : pour obtenir les meilleures performances de Chloros+, fermez les autres applications et laissez Chloros utiliser toutes les ressources du système.
{% endhint %}

### Le traitement ne peut pas être mis en pause

**Limitations importantes :**

* Une fois lancé, le traitement ne peut pas être mis en pause.
* Vous pouvez annuler le traitement, mais la progression sera perdue.
* Les résultats partiels ne sont pas enregistrés.
* Vous devrez recommencer depuis le début si vous annulez.

**Conseil de planification :** pour les projets très volumineux, envisagez de traiter par lots ou d&#x27;utiliser CLI pour un meilleur contrôle.

***

## Surveillance de votre traitement

Pendant le traitement, vous pouvez :

* **Observer la barre de progression** - Voir le pourcentage global d&#x27;achèvement
* **Afficher l&#x27;étape en cours** - Détecter, analyser, calibrer ou exporter
* **Consulter l&#x27;onglet Journal** - Voir les messages et avertissements détaillés relatifs au traitement
* **Prévisualiser les images terminées** - Certains fichiers d&#x27;exportation peuvent apparaître pendant le traitement

Pour plus d&#x27;informations sur la surveillance, consultez [Surveillance du traitement](monitoring-the-processing.md).

***

## Annulation du traitement

Si vous devez arrêter le traitement :

### Comment annuler

1. Localisez le **bouton Arrêter/Annuler** (remplace le bouton Démarrer pendant le traitement)
2. Cliquez sur le bouton Arrêter
3. Le traitement s&#x27;arrête immédiatement
4. Les résultats partiels sont supprimés

### Quand annuler

**Raisons valables pour annuler :**

* Vous vous êtes rendu compte que des paramètres incorrects ont été utilisés
* Vous avez oublié de marquer les images cibles
* Des images incorrectes ont été importées
* Le système est trop lent ou ne répond pas

**Après l&#x27;annulation :**

* Vérifiez et corrigez les éventuels problèmes
* Ajustez les paramètres si nécessaire
* Redémarrez le traitement depuis le début
* Pour une expérience optimale, fermez complètement Chloros et redémarrez

{% hint style=&quot;warning&quot; %}
**Pas de résultats partiels** : l&#x27;annulation supprime toute la progression. Chloros n&#x27;enregistre pas les images partiellement traitées.
{% endhint %}

***

## Estimations du temps de traitement

Le temps de traitement réel varie considérablement en fonction des éléments suivants :

* Nombre d&#x27;images
* Résolution des images
* Format d&#x27;entrée RAW ou JPG
* Mode de traitement (Free ou Chloros+)
* La vitesse du processeur et le nombre de cœurs
* La disponibilité du processeur graphique (Chloros+ uniquement)
* Le nombre d&#x27;indices à calculer
* La complexité du format d&#x27;exportation

### Estimations approximatives (Chloros+, images 12 MP, processeur moderne)

| Nombre d&#x27;images | Mode gratuit | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 images   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 images  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 images  | 1-1,5 h | 20-30 min      | 10-15 min      |
| 500 images  | 2-3 heures   | 45-60 min      | 20-30 min      |
| 1000 images | 4-6 heures   | 1,5-2 heures      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Première exécution** : le traitement initial peut prendre plus de temps, car Chloros crée des caches et des profils. Le traitement ultérieur d&#x27;ensembles de données similaires sera plus rapide.
{% endhint %}

***

## Problèmes courants au démarrage

### Bouton Démarrer désactivé (grisé)

**Causes possibles :**

* Aucune image importée
* Backend pas complètement démarré
* Traitement précédent toujours en cours
* Projet pas complètement chargé

**Solutions :**

1. Attendez que le backend soit complètement initialisé (vérifiez l&#x27;icône du menu principal)
2. Vérifiez que les images sont importées dans le navigateur de fichiers
3. Redémarrez Chloros si le bouton reste désactivé
4. Vérifiez le journal de débogage pour voir s&#x27;il contient des messages d&#x27;erreur

### Le traitement démarre puis échoue immédiatement

**Causes possibles :**

* Aucune image valide dans le projet
* Fichiers image corrompus
* Espace disque insuffisant
* Mémoire insuffisante (RAM)

**Solutions :**

1. Vérifiez le journal de débogage <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pour voir s&#x27;il y a des messages d&#x27;erreur
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

1. **Testez d&#x27;abord avec un petit sous-ensemble** - Traitez 10 à 20 images pour vérifier les paramètres.
2. **Vérifiez l&#x27;espace disque disponible** - Assurez-vous de disposer d&#x27;un espace libre équivalent à 2 ou 3 fois la taille de l&#x27;ensemble de données.
3. **Fermez les applications inutiles** - Libérez des ressources système.
4. **Vérifiez les images cibles** - Prévisualisez les cibles marquées pour vous assurer de leur qualité.
5. **Enregistrez le projet** - Le projet est enregistré automatiquement, mais il est recommandé de l&#x27;enregistrer manuellement.

### Pendant le traitement

1. **Évitez la mise en veille du système** - Désactivez les modes d&#x27;économie d&#x27;énergie.
2. **Gardez Chloros au premier plan** - Ou au moins visible dans la barre des tâches.
3. **Vérifiez la progression de temps en temps** - Vérifiez s&#x27;il y a des avertissements ou des erreurs.
4. **Ne chargez pas d&#x27;autres applications lourdes** - En particulier avec le mode parallèle Chloros+.

### Accélération GPU Chloros+

Si vous utilisez l&#x27;accélération GPU NVIDIA :

1. Mettez à jour les pilotes NVIDIA vers la dernière version.
2. Assurez-vous que le GPU dispose d&#x27;au moins 4 Go de VRAM.
3. Fermez les applications gourmandes en ressources GPU (jeux, montage vidéo).
4. Surveillez la température du GPU (assurez-vous qu&#x27;il est suffisamment refroidi).

***

## Étapes suivantes

Une fois le traitement lancé :

1. **Surveillez la progression** - Voir [Surveillance du traitement](monitoring-the-processing.md)
2. **Attendez la fin du traitement** - Le traitement s&#x27;exécute automatiquement.
3. **Vérifiez les résultats** - Voir [Fin du traitement](finishing-the-processing.md).

Pour plus d&#x27;informations sur ce qu&#x27;il faut faire pendant le traitement, consultez [Surveillance du traitement](monitoring-the-processing.md).
