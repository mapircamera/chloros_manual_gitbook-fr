# Choisir les images cibles

Le marquage des images contenant des cibles de calibration est une étape cruciale qui accélère considérablement le pipeline de traitement Chloros. En présélectionnant les images cibles, vous évitez à Chloros de scanner chaque image de votre jeu de données à la recherche de cibles d'étalonnage.

## Pourquoi marquer les images cibles ?

### Vitesse de traitement

Sans marquer les images cibles, Chloros doit :

* Scanner chaque image de votre projet
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

1. Capture des images cibles avec **les deux caméras** en même temps
2. Utiliser la **même cible physique** pour les deux caméras
3. Marquer les images cibles pour **les deux types de caméras** dans le navigateur de fichiers
4. Chloros utilisera les cibles appropriées pour l'étalonnage de chaque caméra

### Colonne du modèle de caméra

La colonne **Modèle de caméra** permet d'identifier quelles images proviennent de quelle caméra :

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Utilisez cette colonne pour vérifier que vous avez marqué des cibles pour chaque type de caméra dans votre projet.

***

## Paramètres de détection des cibles

### Réglage de la sensibilité de détection

Si Chloros ne détecte pas correctement vos cibles, réglez ces paramètres dans [Project Settings] (réglage de -project-settings.md) :

**Zone d'échantillonnage d'étalonnage minimum:**

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

* Les images de la cible ne sont pas marquées dans l'explorateur de fichiers
* La cible est trop petite dans le cadre (< 30% of image)
* Poor lighting (shadows, glare)
* Target detection settings too strict

**Solutions:**

1. Verify Target column is checked for correct images
2. Review target image quality in preview
3. Recapture targets if quality is poor
4. Adjust target detection settings if needed

### Problem: False Target Detections

**Possible causes:**

* White buildings, vehicles, or ground cover mistaken for targets
* Bright patches in vegetation
* Detection sensitivity too low

**Solutions:**

1. Mark only actual target images to limit detection scope
2. Increase minimum calibration sample area
3. Increase minimum target clustering value
4. Ensure target images show only the target (minimal background clutter)

***

## Verification Checklist

Before starting processing, verify your target image selection:

* [ ] At least 1 target image marked per session
* [ ] Target column checkboxes are checked for all target images
* [ ] Target images captured within same timeframe as survey
* [ ] Targets clearly visible in preview when clicked
* [ ] All 4 calibration panels visible in each target image
* [ ] No shadows or obstructions on targets
* [ ] For dual-camera: Targets marked for both camera types

***

## Target-Free Processing

### Processing Without Calibration Targets

While not recommended for scientific work, you can process without targets:

1. Leave all Target column checkboxes unchecked
2. **Disable** "Reflectance calibration" in Project Settings
3. Vignette correction will still be applied
4. Output will not be calibrated for absolute reflectance

{% hint style="warning" %}
**Not Recommended**: Without reflectance calibration, pixel values represent relative brightness only, not scientific reflectance measurements. Use calibration targets for accurate, repeatable results.
{% endhint %})

***

## Prochaines étapes

Une fois que vous avez marqué vos images cibles :

1. **Revoyez vos paramètres** - Voir [Ajuster les paramètres du projet](ajuster-project-settings.md)
2. **Lancer le traitement** - Voir [Démarrer le traitement](démarrer-the-processing.md)
3. **Surveiller la progression** - Voir [Surveiller le traitement](surveiller-the-processing.md)

Pour plus d'informations sur les cibles d'étalonnage elles-mêmes, voir [Cibles d'étalonnage](../calibration-targets.md).
