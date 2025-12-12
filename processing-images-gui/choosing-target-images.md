# Sélection des images cibles

Le marquage des images contenant des cibles d&#x27;étalonnage est une étape cruciale qui accélère considérablement le pipeline de traitement Chloros. En présélectionnant les images cibles, vous évitez à Chloros d&#x27;avoir à analyser chaque image de votre ensemble de données à la recherche de cibles d&#x27;étalonnage.

## Pourquoi marquer les images cibles ?

### Vitesse de traitement

Sans marquage des images cibles, Chloros doit :

* Analyser chaque image de votre projet
* Exécuter des algorithmes de détection de cibles sur chaque image
* Vérifier inutilement des centaines ou des milliers d&#x27;images

**Résultat** : le traitement peut prendre beaucoup plus de temps, en particulier pour les grands ensembles de données.

### Avec des images cibles marquées

Lorsque vous cochez la colonne Cible pour des images spécifiques :

* Chloros analyse uniquement les images cochées à la recherche de cibles
* La détection des cibles est beaucoup plus rapide
* Le temps de traitement global est considérablement réduit

{% hint style=&quot;success&quot; %}
**Amélioration de la vitesse** : le marquage de 2 à 3 images cibles dans un ensemble de données de 500 images peut réduire le temps de détection des cibles de plus de 30 minutes à moins d&#x27;une minute.
{% endhint %}

***

## Comment marquer les images cibles

### Étape 1 : identifiez vos images cibles

Parcourez les images importées dans le navigateur de fichiers et identifiez celles qui contiennent des cibles d&#x27;étalonnage.

**Scénarios courants :**

* **Cible pré-capture** : capturée avant le début de la session
* **Cible post-capture** : capturée après la fin de la session
* **Cibles sur le terrain** : cibles placées dans la zone de capture
* **Cibles multiples** : 2 à 3 images cibles par session (recommandé)

### Étape 2 : vérifiez la colonne Cible

Pour chaque image contenant une cible d&#x27;étalonnage :

1. Localisez l&#x27;image dans le tableau du navigateur de fichiers.
2. Trouvez la colonne **Cible** (colonne la plus à droite).
3. Cochez la case dans la colonne Cible pour cette image.
4. Répétez l&#x27;opération pour toutes les images contenant des cibles.

### Étape 3 : vérifiez votre sélection

Avant le traitement, vérifiez :

* [ ] Toutes les images avec des cibles d&#x27;étalonnage sont cochées.
* [ ] Aucune image sans cible n&#x27;est cochée par erreur.
* [ ] Les cibles sont clairement visibles dans les images cochées.

***

## Bonnes pratiques pour les images cibles

### Consignes de capture des cibles

**Timing :**

* Capturez les images cibles immédiatement avant et pendant votre session de capture.
* Dans les mêmes conditions d&#x27;éclairage que votre capteur de lumière DAQ.
* Idéalement, capturez les images cibles aussi souvent que possible pour obtenir les meilleurs résultats. Sinon, les données du capteur de lumière seront utilisées pour ajuster l&#x27;étalonnage au fil du temps.

**Position de la caméra :**

* Tenez la caméra au-dessus de la cible de manière à ce qu&#x27;elle soit centrée et occupe environ 40 à 60 % du centre de l&#x27;image.
* Maintenez la caméra parallèle/nadir par rapport à la surface cible.

**Éclairage :**

* Même éclairage ambiant que votre capteur de lumière DAQ.
* Évitez les ombres sur les surfaces cibles.
* Ne bloquez pas votre source de lumière avec votre corps, un véhicule ou de la végétation.
* Les conditions nuageuses fournissent les résultats les plus cohérents.

**Condition de la cible :**

* Gardez les panneaux cibles propres et secs.
* Les 4 panneaux doivent être clairement visibles et sans obstruction.
* Les cibles doivent être perpendiculaires/nadir par rapport à la source de lumière si possible.

### Combien d&#x27;images cibles ?

**Minimum :** 1 image cible par session. **Recommandé :** 3 à 5 images cibles par session.

**Calendrier des meilleures pratiques :**

* 3 à 5 images capturées peu après le début de l&#x27;enregistrement par le capteur de lumière
* Faites pivoter la caméra entre les captures pour obtenir les meilleurs résultats
* Facultatif : périodiquement au milieu de la session si les conditions d&#x27;éclairage changent constamment

***

## Utilisation de plusieurs caméras

### Configurations à deux caméras

Si vous utilisez deux caméras MAPIR simultanément (par exemple, Survey3W RGN + Survey3N OCN) :

1. Capturez les images cibles avec **les deux caméras** en même temps.
2. Utilisez la **même cible physique** pour les deux caméras.
3. Marquez les images cibles pour **les deux types de caméras** dans le navigateur de fichiers.
4. Chloros utilisera les cibles appropriées pour l&#x27;étalonnage de chaque caméra.

### Colonne Modèle de caméra

La colonne **Modèle de caméra** permet d&#x27;identifier quelles images proviennent de quelle caméra :

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Utilisez cette colonne pour vérifier que vous avez marqué les cibles pour chaque type de caméra dans votre projet.

***

## Paramètres de détection des cibles

### Réglage de la sensibilité de détection

Si Chloros ne détecte pas correctement vos cibles, réglez ces paramètres dans [Paramètres du projet](adjusting-project-settings.md) :

**Zone d&#x27;échantillonnage minimale pour l&#x27;étalonnage :**

* **Par défaut** : 25 pixels
* **Augmentez** si vous obtenez de fausses détections sur de petits artefacts
* **Diminuez** si les cibles ne sont pas détectées

**Regroupement minimal des cibles :**

* **Par défaut** : 60
* **Augmentez** si les cibles sont divisées en plusieurs détections.
* **Diminuez** si les cibles présentant des variations de couleur ne sont pas entièrement détectées.

***

## Problèmes courants liés aux images cibles

### Problème : aucune cible détectée

**Causes possibles :**

* Images cibles non marquées dans le navigateur de fichiers.
* Cible trop petite dans le cadre (&lt; 30 % de l&#x27;image).
* Mauvais éclairage (ombres, reflets)
* Paramètres de détection des cibles trop stricts

**Solutions :**

1. Vérifiez que la colonne Cible est cochée pour les images correctes
2. Vérifiez la qualité de l&#x27;image cible dans l&#x27;aperçu
3. Recapturez les cibles si la qualité est mauvaise
4. Ajustez les paramètres de détection des cibles si nécessaire

### Problème : fausses détections de cibles

**Causes possibles :**

* Bâtiments blancs, véhicules ou couverture végétale confondus avec des cibles
* Taches lumineuses dans la végétation
* Sensibilité de détection trop faible

**Solutions :**

1. Ne marquer que les images cibles réelles afin de limiter la portée de la détection
2. Augmenter la zone d&#x27;échantillonnage minimale pour l&#x27;étalonnage
3. Augmenter la valeur minimale de regroupement des cibles
4. S&#x27;assurer que les images cibles ne montrent que la cible (encombrement minimal de l&#x27;arrière-plan)

***

## Liste de contrôle de vérification

Avant de commencer le traitement, vérifiez votre sélection d&#x27;images cibles :

* [ ] Au moins 1 image cible marquée par session
* [ ] Les cases à cocher de la colonne Cible sont cochées pour toutes les images cibles
* [ ] Images cibles capturées dans le même laps de temps que l&#x27;enquête
* [ ] Cibles clairement visibles dans l&#x27;aperçu lorsque vous cliquez dessus
* [ ] Les 4 panneaux d&#x27;étalonnage sont visibles dans chaque image cible
* [ ] Aucune ombre ni obstruction sur les cibles
* [ ] Pour les appareils à double caméra : cibles marquées pour les deux types de caméra

***

## Traitement sans cible

### Traitement sans cibles d&#x27;étalonnage

Bien que cela ne soit pas recommandé pour les travaux scientifiques, vous pouvez effectuer le traitement sans cibles :

1. Laissez toutes les cases à cocher de la colonne Cible décochées
2. **Désactivez** « Étalonnage de la réflectance » dans les paramètres du projet
3. La correction du vignettage sera toujours appliquée
4. La sortie ne sera pas calibrée pour la réflectance absolue

{% hint style=&quot;warning&quot; %}
**Non recommandé** : sans calibrage de la réflectance, les valeurs des pixels représentent uniquement la luminosité relative, et non des mesures scientifiques de la réflectance. Utilisez des cibles de calibrage pour obtenir des résultats précis et reproductibles.
{% endhint %}

***

## Étapes suivantes

Une fois que vous avez marqué vos images cibles :

1. **Vérifiez vos paramètres** - Voir [Ajuster les paramètres du projet](adjusting-project-settings.md)
2. **Lancez le traitement** - Voir [Démarrage du traitement](starting-the-processing.md)
3. **Suivez la progression** - Voir [Suivi du traitement](monitoring-the-processing.md)

Pour plus d&#x27;informations sur les cibles d&#x27;étalonnage elles-mêmes, voir [Cibles d&#x27;étalonnage](../calibration-targets.md).
