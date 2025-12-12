# Choisir les images cibles

Le marquage des images contenant des cibles de calibration est une étape cruciale qui accélère considérablement le pipeline de traitement Chloros. En présélectionnant les images cibles, vous évitez à ChlorosX de scanner chaque image de votre jeu de données à la recherche de cibles de calibration.

## Pourquoi marquer les images cibles ?

### Vitesse de traitement

Sans marquer les images cibles, ZXZXZZ000008ZXZZXXXX doit :

* Scanner toutes les images de votre projet
* Exécuter des algorithmes de détection de cibles sur chaque image
* Vérifier inutilement des centaines ou des milliers d'images

**Résultat** : Le traitement peut prendre beaucoup plus de temps, en particulier pour les grands ensembles de données.

### Avec des images cibles marquées

Lorsque vous cochez la colonne Cible pour des images spécifiques :

* ChlorosX ne recherche des cibles que dans les images cochées
* La détection des cibles s'effectue beaucoup plus rapidement
* Le temps de traitement global est considérablement réduit

___PROTÉGÉ_0001___

***

## Comment marquer les images cibles

### Étape 1 : Identifier les images cibles

Examinez les images importées dans l'Explorateur de fichiers et identifiez celles qui contiennent des cibles de calibrage.

**Scénarios courants:**

* **Cible pré-capture** : Capturée avant le début de la session
* **Cible post-capture** : Cible post-capture** : capturée après avoir terminé la session
* **Cibles dans le champ** : Cibles placées dans la zone de capture
* **Cibles multiples** : 2-3 images de cibles par session (recommandé)

### Étape 2 : Vérifier la colonne des cibles

Pour chaque image contenant une cible d'étalonnage :

1. Localisez l'image dans le tableau de l'explorateur de fichiers
2. Trouver la colonne **Cible** (colonne la plus à droite)
3. Cliquez sur la case à cocher de la colonne Cible pour cette image
4. Répéter l'opération pour toutes les images contenant des cibles

### Étape 3 : Vérifier votre sélection

Avant le traitement, vérifiez deux fois :

* [ ] Toutes les images comportant des cibles d'étalonnage sont vérifiées
* ZXZXZX000001ZXXXZX :

**Surface minimale de l'échantillon d'étalonnage:**

* **Par défaut** : 25 pixels
* **Augmentez cette valeur si vous obtenez de fausses détections sur de petits artefacts
* **Diminuer** si les cibles ne sont pas détectées

**Groupement minimal de cibles:**

* **Par défaut** : 60
* **Augmentation** si les cibles sont divisées en plusieurs détections
* **Diminuer** si les cibles avec des variations de couleur ne sont pas entièrement détectées

***

## Problèmes courants liés à l'image de la cible

### Problème : Aucune cible n'est détectée

**Causes possibles:**

* Les images cibles ne sont pas marquées dans le navigateur de fichiers
* Cible trop petite dans le cadre (< 30 % de l'image)
* Mauvais éclairage (ombres, reflets)
* Paramètres de détection de la cible trop stricts

**Solutions:**

1. Vérifier que la colonne Cible est vérifiée pour les images correctes
2. Vérifier la qualité de l'image cible dans l'aperçu
3. Recapturer les cibles si la qualité est mauvaise
4. Ajuster les paramètres de détection des cibles si nécessaire

### Problème : fausses détections de cibles

**Causes possibles:**

* Bâtiments, véhicules ou couverture végétale blancs pris pour des cibles
* Taches lumineuses dans la végétation
* Sensibilité de détection trop faible

**Solutions:**

1. Ne marquer que les images de la cible réelle pour limiter la portée de la détection
2. Augmenter la surface minimale de l'échantillon d'étalonnage
3. Augmenter la valeur minimale de regroupement des cibles
4. S'assurer que les images de la cible ne montrent que la cible (minimum de bruit de fond)

***

## Liste de contrôle pour la vérification

Avant de commencer le traitement, vérifiez la sélection de l'image cible :

* ZXZXZX000002ZXXXZXX
2. **Lancer le traitement** - Voir [Starting the Processing](starting-the-processing.md)XX
3. **Suivre la progression** - Voir [Monitoring the Processing](monitoring-the-processing.md)XX

Pour plus d'informations sur les cibles d'étalonnage elles-mêmes, voir ZXZXZ000005ZXZXZXXX.
