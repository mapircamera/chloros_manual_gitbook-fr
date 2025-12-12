# Choisir les images cibles

Le marquage des images contenant des cibles de calibration est une étape cruciale qui accélère considérablement le pipeline de traitement Chloros. En présélectionnant les images cibles, vous évitez à Chloros de scanner chaque image de votre jeu de données à la recherche de cibles de calibration.

## Pourquoi marquer les images cibles ?

### Vitesse de traitement

Sans marquer les images cibles, Chloros doit :

* Scanner toutes les images de votre projet
* Exécuter des algorithmes de détection de cibles sur chaque image
* Vérifier inutilement des centaines ou des milliers d'images

**Résultat** : Le traitement peut prendre beaucoup plus de temps, en particulier pour les grands ensembles de données.

### Avec des images cibles marquées

Lorsque vous cochez la colonne Cible pour des images spécifiques :

* Chloros ne recherche des cibles que dans les images cochées
* La détection des cibles s'effectue beaucoup plus rapidement
* Le temps de traitement global est considérablement réduit

{% hint style="success" %}
**Speed Improvement**: Marking 2-3 target images in a 500-image dataset can reduce target detection time from 30+ minutes to under 1 minute.
{% endhint %}

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
* [Aucune image ne comportant pas de cible n'a été vérifiée accidentellement
* [Les cibles sont clairement visibles sur les images vérifiées

***

## Bonnes pratiques pour les images cibles

### Lignes directrices pour la capture de cibles

**Timing:**

* Capturez les images de la cible immédiatement avant et pendant la session de capture
* Dans les mêmes conditions d'éclairage que le capteur de lumière de votre DAQ
* Pour obtenir les meilleurs résultats, il est préférable de capturer des images cibles aussi souvent que possible. Sinon, les données du capteur de lumière seront utilisées pour ajuster la calibration au fil du temps.

**Position de la caméra:**

* Tenez la caméra au-dessus de la cible de manière à ce qu'elle soit centrée et remplisse environ 40 à 60 % du centre de l'image.
* Maintenez la caméra parallèle/nadir à la surface de la cible

**Eclairage:**

* Même éclairage ambiant que le capteur de lumière de votre DAQ
* Évitez les ombres sur les surfaces cibles
* Ne bloquez pas votre source de lumière avec votre corps, votre véhicule ou la végétation
* Un temps couvert permet d'obtenir les résultats les plus cohérents

**Target Condition:**

* Gardez les panneaux de la cible propres et secs
* Les 4 panneaux doivent être clairement visibles et non obstrués
* Cibles perpendiculaires/nadir à la source lumineuse si possible

### Combien d'images cibles ?

**Minimum:** 1 image cible par session. **Recommandé:** 3 à 5 images cibles par session.

**Horaire de pratique optimale:**

* 3 à 5 images capturées peu après l'enregistrement du capteur de lumière
* Pour obtenir les meilleurs résultats, faites pivoter l'appareil photo entre les captures
* Facultatif : périodiquement au milieu de la session si les conditions d'éclairage changent constamment

***

## Travailler avec plusieurs caméras

### Configurations à deux caméras

Si vous utilisez deux caméras MAPIR simultanément (par exemple, Survey3W RGN + Survey3N OCN) :

1. Capturez les images cibles avec **les deux caméras** en même temps
2. Utiliser la **même cible physique** pour les deux caméras
3. Marquer les images cibles pour **les deux types de caméras** dans le navigateur de fichiers
4. Chloros utilisera les cibles appropriées pour l'étalonnage de chaque caméra

### Colonne du modèle de caméra

La colonne **Modèle de caméra** permet d'identifier quelles images proviennent de quelle caméra :

* Survey3W\RGN
* Survey3N\N-OCN
* Survey3W\RGB
* etc.

Utilisez cette colonne pour vérifier que vous avez marqué des cibles pour chaque type de caméra dans votre projet.

***

## Paramètres de détection des cibles

### Réglage de la sensibilité de détection

Si Chloros ne détecte pas correctement vos cibles, réglez ces paramètres dans [Project Settings](adjusting-project-settings.md) :

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

* [Au moins une image cible est marquée par session
* [Les cases de la colonne cible sont cochées pour toutes les images cibles
* [Les images cibles ont été capturées dans le même laps de temps que l'enquête
* [Les cibles sont clairement visibles dans l'aperçu lorsqu'on clique dessus
* [Les 4 panneaux d'étalonnage sont visibles sur chaque image cible
* [Pas d'ombres ou d'obstructions sur les cibles
* [ ] Pour les caméras doubles : Cibles marquées pour les deux types de caméra

***

## Traitement sans cible

### Traitement sans cibles d'étalonnage

Bien que cela ne soit pas recommandé pour les travaux scientifiques, il est possible d'effectuer des traitements sans cibles :

1. Ne pas cocher toutes les cases de la colonne Cible
2. **Désactiver** la "calibration de la réflectance" dans les paramètres du projet
3. La correction de la vignette sera toujours appliquée
4. La sortie ne sera pas calibrée pour la réflectance absolue

{% hint style="warning" %}
**Not Recommended**: Without reflectance calibration, pixel values represent relative brightness only, not scientific reflectance measurements. Use calibration targets for accurate, repeatable results.
{% endhint %}

***

## Prochaines étapes

Une fois que vous avez marqué vos images cibles :

1. **Revoyez vos paramètres** - Voir [Adjusting Project Settings](adjusting-project-settings.md)
2. **Lancer le traitement** - Voir [Starting the Processing](starting-the-processing.md)
3. **Suivre la progression** - Voir [Monitoring the Processing](monitoring-the-processing.md)

Pour plus d'informations sur les cibles de calibration elles-mêmes, voir [Calibration Targets](../calibration-targets.md).
