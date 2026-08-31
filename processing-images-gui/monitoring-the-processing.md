# Suivi du traitement

Une fois le traitement lancé, Chloros propose plusieurs moyens de suivre la progression, de détecter les problèmes et de comprendre ce qui se passe avec votre ensemble de données. Cette page explique comment suivre votre traitement et interpréter les informations fournies par Chloros.

## Présentation de la barre de progression

La barre de progression située dans l’en-tête supérieur affiche en temps réel l’état du traitement et le pourcentage d’achèvement. La progression est transmise en direct depuis le backend via des événements envoyés par le serveur (SSE), de sorte que la barre reflète ce que le pipeline est réellement en train de faire.

### Barre de progression en mode gratuit

Pour les utilisateurs ne disposant pas d’une licence Chloros+ :

**Affichage de la progression en deux étapes :**

1.**Détection des cibles** – Recherche des cibles d’étalonnage dans les images
2. **Traitement** – Application des corrections et exportation**La barre de progression affiche :**

* Pourcentage global d’achèvement (0-100 %)
* Nom de l’étape en cours
* Visualisation simple sous forme de barre horizontale

### Barre de progression Chloros+

Pour les utilisateurs disposant d’une licence Chloros+ :

**Affichage de la progression en 4 étapes :**

1.**Détection** - Recherche des cibles d’étalonnage
2. **Analyse** - Examen des images et préparation du pipeline
3. **Calibrage** - Application des corrections de vignettage et de réflectance
4. **Exportation** - Enregistrement des fichiers traités**Fonctionnalités interactives :*** **Passez la souris sur** la barre de progression pour afficher le panneau détaillé en 4 étapes
* **Cliquez sur** la barre de progression pour figer/épingler le panneau détaillé
* **Cliquez à nouveau** pour le débloquer et le masquer automatiquement lorsque vous éloignez la souris
* Chaque étape affiche sa progression individuelle (0-100 %)

{% hint style="info" %}
**Parité CLI** : lors d’une exécution de `chloros-cli process`, les quatre mêmes threads indiquent les états « Détection », « Analyse », « Traitement » et « Exportation », tandis que `chloros-cli export-status` affiche en temps réel la progression de l’exportation du thread 4 depuis un autre terminal. Consultez la [Référence CLI](../reference/cli-reference.md).
{% endhint %}

***

## Comprendre chaque étape du traitement

{% hint style="info" %}
**Architecture en pipeline** : ces 4 étapes de l&#x27;interface graphique correspondent au [pipeline de traitement à 4 threads](../processing-architecture/processing-pipeline.md). Sur les systèmes dotés d’une accélération GPU, le thread 3 (Calibrage) bénéficie de l’[adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md), qui optimise le traitement en fonction de votre matériel spécifique.
{% endhint %}

### Étape 1 : Détection (détection des cibles)

**Ce qui se passe :**

* Chloros analyse les images que vous avez sélectionnées à l’aide de la case à cocher « Cible » (toutes les images si aucune n’est cochée)
* Des algorithmes de vision par ordinateur identifient les panneaux d’étalonnage
* Les valeurs de réflectance sont extraites de chaque panneau
* Les horodatages des cibles sont enregistrés pour une planification correcte de l’étalonnage

**Durée :**

* Avec des cibles marquées : 10 à 60 secondes
* Sans cibles marquées : 5 à 30 minutes ou plus (analyse de toutes les images)

**Indicateur de progression :**

* Détection : 0 % → 100 %
* Nombre d’images analysées (ne compte que les images effectivement analysées)
* Nombre de cibles trouvées

**À surveiller :**

* Le processus devrait s’achever rapidement si les cibles sont correctement marquées
* Si cela prend trop de temps, les cibles ne sont peut-être pas marquées
* Vérifiez le journal de débogage pour voir s’il contient des messages « Cible trouvée »

### Étape 2 : Analyse

**Ce qui se passe :**

* Lecture des métadonnées EXIF des images (horodatages, paramètres d’exposition)
* Détermination de la stratégie d’étalonnage en fonction des horodatages des cibles et des données de rayonnement descendant DAQ disponibles
* Organisation de la file d’attente de traitement des images
* Préparation des processus de traitement en parallèle (Chloros+ uniquement)

**Durée :** 5 à 30 secondes**Indicateur de progression :**

* Analyse en cours : 0 % → 100 %
* Étape rapide, généralement terminée en peu de temps

**À surveiller :**

* La progression doit être régulière, sans interruption
* Des avertissements concernant des métadonnées manquantes apparaîtront dans le journal de débogage

### Étape 3 : Étalonnage

**Ce qui se passe :*** **Débayérisation** : conversion du motif RAW de Bayer en 3 canaux (ignorée pour les modules mono LATTICE, avec une remarque)
* **Correction du vignetage** : suppression de l&#x27;assombrissement sur les bords de l&#x27;objectif
* **Étalonnage de la réflectance** : normalisation par rapport aux valeurs cibles et/ou au rayonnement descendant du DAQ
* **Calcul des indices** : calcul des indices multispectraux
* Traitement de chaque image via l’ensemble du pipeline

**Durée :** la majeure partie du temps de traitement total (60 à 80 %)**Indicateur de progression :**

* Étalonnage : 0 % → 100 %
* Image en cours de traitement
* Images traitées / Nombre total d’images

**Comportement du traitement :*** **Mode libre** : traite une image à la fois, de manière séquentielle
* **Mode Chloros+** : Exécute un pool de workers adaptatif au matériel — 1 à 4 workers simultanés sur les systèmes GPU (en fonction de la VRAM), un worker par cœur physique (moins un) sur les systèmes exclusivement CPU. Voir [Adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md)
* **Accélération GPU** : accélère considérablement cette étape**À surveiller :**

* Progression régulière du nombre d’images
* Vérifier le journal de débogage pour les messages d’achèvement par image
* Avertissements concernant la qualité des images ou des problèmes d’étalonnage

### Étape 4 : Exportation

**Ce qui se passe :**

* Enregistrement des images traitées sur le disque au format sélectionné, au fur et à mesure de leur achèvement
* **LATTICE** : chaque image est répartie vers tous les produits activés (débayérisation / aperçu / radiance / réflectance)
* Exportation d’images d’index multispectrales avec les couleurs de la table de conversion (LUT)
* Création de l’arborescence de sortie `<project>/<camera>/<format>/<Product>_Images/` — les fichiers exportés conservent le nom du fichier source ; le dossier identifie le produit

**Durée :** 10 à 20 % du temps total de traitement**Indicateur de progression :**

* Exportation : 0 % → 100 %
* Écriture des fichiers en cours
* Format d’exportation et destination

**Éléments à surveiller :**

* Avertissements relatifs à l’espace disque
* Erreurs d’écriture de fichiers
* Achèvement de toutes les sorties configurées

***

## Onglet « Journal de débogage »

Le journal de débogage fournit des informations détaillées sur la progression du traitement et les éventuels problèmes rencontrés. Les messages de démarrage du backend sont également consignés dans la console de journalisation ; ainsi, le journal retrace l’intégralité du processus, même si vous l’ouvrez a posteriori.

### Accéder au journal de débogage

1. Cliquez sur l’icône **Journal de débogage**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

dans la barre latérale gauche
2. Le panneau du journal s’ouvre et affiche les messages de traitement en temps réel
3. Le défilement automatique permet d’afficher les messages les plus récents

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Comprendre les messages du journal

Les lignes du journal Chloros sont préfixées par des balises entre crochets indiquant le nom du sous-système — par exemple `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. La ligne la plus importante à connaître est le **résumé de l&#x27;exécution**, affiché à la fin de chaque exécution (y compris les exécutions interrompues) :

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Des lignes d’indications supplémentaires `[RUN-SUMMARY]` sont ajoutées chaque fois qu’une explication est nécessaire — par exemple, un cycle qui n’a rien produit, ou une caméra dont le produit demandé a été ignoré car inapplicable. Les lignes `[EXPORT-CHECK]` expliquent les omissions par caméra (par exemple, pourquoi une caméra RGB n’a pas obtenu de produit de radiance).

Les niveaux de gravité généraux des messages (les exemples ci-dessous sont donnés à titre indicatif et ne sont pas littéraux) :

#### Messages d’information (blanc/gris)

Mises à jour de traitement normales : début du traitement, cibles détectées (avec nombre de panneaux), progression de l’étalonnage par image, fichiers exportés, traitement terminé.

#### Messages d’avertissement (jaune)

Problèmes non critiques qui n’interrompent pas le traitement — par exemple, des données GPS manquantes dans une image, un écart important entre les horodatages de deux images cibles, ou un faible contraste dans un panneau d’étalonnage.

**Action :** Examiner les avertissements après le traitement, mais ne pas interrompre celui-ci

#### Messages d’erreur (Red)

Problèmes critiques pouvant entraîner l’échec du traitement — par exemple, disque plein, fichier image corrompu ou aucune cible détectée alors qu’un étalonnage de réflectance était demandé.

**Action :** Arrêtez le traitement, résolvez l’erreur, puis redémarrez

### Situations courantes dans le journal

| Situation                             | Signification                                       | Action requise                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Cible détectée dans \[nom_fichier]        | Cible d’étalonnage trouvée avec succès         | Aucune — normal                                         |
| Barres de progression par image              | Mise à jour de la progression actuelle                       | Aucune — normal                                         |
| Aucune cible trouvée                      | Aucune cible d’étalonnage détectée               | Marquer les images cibles ou désactiver l’étalonnage par réflectance |
| Espace disque insuffisant               | Espace de stockage insuffisant pour la sortie                 | Libérer de l’espace disque                                    |
| Fichier corrompu ignoré               | Le fichier image est endommagé                         | Recopier le fichier depuis la carte SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Une capture sans image brute ne peut pas être traitée | Recapturez avec une image brute, ou utilisez CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | L&#x27;exécution n&#x27;a généré aucun produit image — signalée comme un échec avec des conseils | Lisez les lignes de conseil ; vérifiez ce qui a été ignoré et pourquoi |

### Copie des données du journal

Pour copier le journal à des fins de dépannage ou d&#x27;assistance :

1. Ouvrez le panneau « Journal de débogage »
2. Cliquez sur le bouton **« Copier le journal »** (ou cliquez avec le bouton droit → Sélectionner tout)
3. Collez le contenu dans un fichier texte ou un e-mail
4. Envoyez-le au support MAPIR si nécessaire

***

## Surveillance des ressources système

### Utilisation du processeur

**Mode Free :**

* 1 cœur de processeur à environ 100 %
* Les autres cœurs sont inactifs ou disponibles
* Le système reste réactif

**Mode parallèle Chloros+ :**

* Plusieurs cœurs à forte utilisation — leur nombre dépend de la stratégie choisie par [Adaptation dynamique des ressources de calcul](../processing-architecture/dynamic-compute-adaptation.md)
* Le système peut sembler moins réactif

**Pour surveiller :**

* Gestionnaire de tâches Windows (Ctrl+Maj+Échap)
* Onglet « Performances » → section « Processeur »
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
* Augmentez la capacité de la RAM si vous traitez régulièrement de grands ensembles de données

### Utilisation du GPU (Chloros+ avec CUDA)

Lorsque l&#x27;accélération GPU est activée :

* Le GPU NVIDIA affiche un taux d&#x27;utilisation élevé (60 à 90 %)
* L&#x27;utilisation de la VRAM augmente (nécessite au moins 4 Go de VRAM ; au moins 7 Go pour le débayering « Texture Aware » en parallèle)
* L&#x27;étape d&#x27;étalonnage est nettement plus rapide

**À surveiller :**

* L&#x27;icône NVIDIA dans la barre d&#x27;état système
* Gestionnaire des tâches → Performances → GPU
* GPU-Z ou un outil de surveillance similaire

### E/S disque

**À quoi s’attendre :**

* Taux de lecture disque élevé pendant la phase d’analyse
* Taux d’écriture disque élevé pendant la phase d’exportation
* Les SSD sont nettement plus rapides que les disques durs (HDD)

**Conseil de performance :**

* Utilisez un SSD pour le dossier du projet dans la mesure du possible
* Évitez les lecteurs réseau pour les grands ensembles de données
* Assurez-vous que le disque n&#x27;est pas presque plein (cela affecte la vitesse d&#x27;écriture)

***

## Détection des problèmes pendant le traitement

### Signes avant-coureurs

**La progression stagne (aucun changement pendant plus de 5 minutes) :**

* Vérifiez le journal de débogage pour détecter d&#x27;éventuelles erreurs
* Vérifiez l&#x27;espace disque disponible
* Vérifiez dans le Gestionnaire des tâches que le processus Chloros est en cours d’exécution

**Des messages d’erreur apparaissent fréquemment :**

* Interrompez le traitement et examinez les erreurs
* Causes courantes : espace disque, fichiers corrompus, problèmes de mémoire
* Consultez la section « Dépannage » ci-dessous

**Le système ne répond plus :**

* Le mode parallèle de Chloros+ utilise trop de ressources
* Envisagez de réduire le nombre de tâches simultanées ou de mettre à niveau le matériel
* Le mode « Free » est moins gourmand en ressources

### Quand interrompre le traitement

Interrompez le traitement si vous constatez :

* ❌ Erreurs « Disque plein » ou « Impossible d&#x27;écrire le fichier »
* ❌ Erreurs répétées de corruption des fichiers image
* ❌ Système complètement bloqué (ne répond plus)
* ❌ Vous vous rendez compte que des paramètres incorrects ont été configurés
* ❌ Importation d&#x27;images incorrectes

**Comment arrêter :**

1. Cliquez sur le**bouton Arrêter** (qui remplace le bouton Démarrer) — une seule fois suffit
2. La barre affiche « Arrêt en cours... » pendant que l’image en cours de traitement se termine, puis l’exécution s’arrête
3. Les produits déjà exportés restent sur le disque ; le journal affiche un message `[RUN-SUMMARY]` détaillant ce qui a été effectué
4. Corrigez les problèmes et redémarrez — l’exécution recommence depuis le début

***

## Dépannage pendant le traitement

### Le traitement est très lent

**Causes possibles :**

* Images cibles non marquées (analyse de toutes les images)
* Stockage sur disque dur (HDD) au lieu d’un SSD
* Ressources système insuffisantes
* Nombreux index configurés
* Accès à un lecteur réseau

**Solutions :**

1. Si le traitement vient de démarrer et se trouve en phase de détection : arrêtez-le, marquez les cibles, puis redémarrez
2. À l’avenir : utilisez un SSD, réduisez le nombre d’index, mettez à niveau le matériel
3. Envisagez d’utiliser CLI pour le traitement par lots de grands ensembles de données

### Avertissements « Espace disque »

**Solutions :**

1. Libérez immédiatement de l’espace disque
2. Déplacez le projet vers un disque disposant de plus d’espace
3. Réduire le nombre d’index à exporter
4. Désactiver les produits d’exportation LATTICE dont vous n’avez pas besoin (Paramètres du projet → Traitement)
5. Utiliser le format JPG à la place de TIFF (fichiers plus légers)

### Messages fréquents indiquant « Fichier corrompu »

**Solutions :**

1. Recopiez les images depuis la carte SD pour garantir leur intégrité
2. Vérifiez que la carte SD ne présente pas d&#x27;erreurs
3. Supprimez les fichiers corrompus du projet
4. Poursuivez le traitement des images restantes

### Surchauffe / ralentissement du système

**Solutions :**

1. Assurez-vous que la ventilation est suffisante
2. Nettoyez la poussière des évents de l&#x27;ordinateur
3. Réduisez la charge de traitement (utilisez le mode « Free » au lieu de Chloros+)
4. Effectuez le traitement pendant les moments de la journée où il fait plus frais

***

## Notification de fin de traitement

Lorsque le traitement est terminé :

* La barre de progression atteint 100 %
* Les lignes `[RUN-SUMMARY]` apparaissent dans le journal de débogage avec les chiffres définitifs
* Le bouton Démarrer redevient actif
* Tous les fichiers de sortie se trouvent dans l’arborescence de sortie par caméra du projet : `<project>/<camera>/<format>/<Product>_Images/`

***

## Étapes suivantes

Une fois le traitement terminé :

1. **Vérifiez les résultats** - Consultez la section [Finalisation du traitement](finishing-the-processing.md)
2. **Vérifiez le dossier de sortie** - Assurez-vous que tous les fichiers ont été correctement exportés
3. **Consultez le journal de débogage** - Vérifiez s’il contient des avertissements ou des erreurs
4. **Prévisualisez les images traitées** - Utilisez la visionneuse d’images ou un logiciel externe

Pour plus d’informations sur la consultation et l’utilisation de vos résultats traités, consultez la section [Finalisation du traitement](finishing-the-processing.md).
