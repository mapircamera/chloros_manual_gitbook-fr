# Démarrage du traitement

Une fois que vous avez importé vos images, marqué vos cibles d'étalonnage et configuré les paramètres de votre projet, vous êtes prêt à commencer le traitement. Cette page vous guide dans le lancement du pipeline de traitement Chloros.

## Liste de contrôle pour le prétraitement

Avant de cliquer sur le bouton Démarrer, vérifiez que tout est prêt :

* [ ] **Fichiers importés** - Toutes les images apparaissent dans le navigateur de fichiers
* [ ] **Images cibles marquées** - La colonne Cible a été vérifiée pour les images d'étalonnage
* [ ] **Modèles de caméra détectés** - La colonne Modèle de caméra affiche les caméras correctes
* [ ] **Paramètres configurés** - Les paramètres du projet sont examinés et ajustés
* [ ] **Indices sélectionnés** - Les indices multispectraux souhaités sont ajoutés (si nécessaire)
* X[Format d'exportation choisi** - Format de sortie approprié à votre flux de travail

___PROTÉGÉ_0003___

***

## Démarrage du traitement

### Localiser le bouton de démarrage

Le bouton Start/Play se trouve dans la barre d'en-tête supérieure de Chloros :

* Position : Centre supérieur de la fenêtre
* Icône : **Bouton Jouer/Démarrer** ZXZXZX000001ZXZXZX
* Statut : Le bouton est activé (lumineux) lorsqu'il est prêt à être traité

### Cliquez pour démarrer

1. Cliquez sur le **bouton Jouer/Démarrer** dans l'en-tête supérieur
2. Le traitement commence immédiatement
3. Le bouton est désactivé (grisé) pendant le traitement
4. La barre de progression se met à jour et indique l'état d'avancement du traitement

___PROTÉGÉ_0004___

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
* Convient aux ensembles de données de petite à moyenne taille (ZXZXZX000002ZXZXZX pour les messages d'erreur)
2. Vérifier l'espace disque disponible
3. Essayez de traiter un sous-ensemble d'images plus petit
4. Vérifier que les images ne sont pas corrompues

### Avertissement "Aucune cible détectée

**Causes possibles:**

* Oubli de marquer les images cibles
* Les images des cibles ne contiennent pas de cibles visibles
* Paramètres de détection des cibles trop stricts

**Solutions:**

1. Réviser [Choosing Target Images](choosing-target-images.md)
2. Cochez les images appropriées dans la colonne Cible
3. Vérifier que les cibles sont visibles sur les images marquées
4. Ajustez les paramètres de détection des cibles si nécessaire

***

## Conseils pour un traitement réussi

### Avant de commencer

1. **Testez d'abord avec un petit sous-ensemble** - Traitez 10 à 20 images pour vérifier les paramètres
2. **Vérifiez l'espace disque disponible** - Assurez-vous que 2 à 3 fois la taille de l'ensemble de données sont libres
3. **Fermez les applications inutiles** - Libérez les ressources du système
4. **Vérifiez les images cibles** - Prévisualisez les cibles marquées pour vous assurer de leur qualité
5. **Sauvegarder le projet** - Le projet est sauvegardé automatiquement, mais il est préférable de le sauvegarder manuellement

### Pendant le traitement

1. **Éviter la mise en veille du système** - Désactiver les modes d'économie d'énergie
2. **Garder ChlorosX au premier plan** - Ou au moins visible dans la barre des tâches
3. **Surveiller la progression de temps en temps** - Vérifier les avertissements ou les erreurs
4. **Ne chargez pas d'autres applications lourdes** - Surtout en mode parallèle +
Chloros
### ChlorosX+ Accélération GPU

Si vous utilisez l'accélération GPU NVIDIA :

1. Mettez à jour les pilotes NVIDIA avec la dernière version
2. Assurez-vous que le GPU dispose de plus de 4 Go de VRAM
3. Fermez les applications gourmandes en GPU (jeux, édition vidéo)
4. Surveillez la température du GPU (assurez-vous qu'il est correctement refroidi)

***

## Prochaines étapes

Une fois que le traitement a commencé :

1. **Surveillez la progression** - Voir ZXZXZ000004ZXZXZZXX
2. **Attendre la fin** - Le traitement s'exécute automatiquement
3. **Vérifier les résultats** - Voir ZXZXZ000005ZXZXXX

Pour plus d'informations sur ce qu'il faut faire pendant le traitement, voir [Monitoring the Processing](monitoring-the-processing.md)X.
