# Sélection des images cibles

En indiquant quelles images contiennent des cibles d’étalonnage, vous indiquez précisément à Chloros où les rechercher. Lorsqu’au moins une image est cochée dans la colonne « Cible », Chloros analyse **uniquement les images cochées** — le fait de marquer les cibles permet donc à la fois d’accélérer le traitement et d’éviter que des images de levé ne soient confondues avec une cible.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Pourquoi marquer les images cibles ?

### Le marquage contrôle l’analyse

Lorsque vous cochez la colonne « Cible » pour des images spécifiques :

* Chloros analyse uniquement les images cochées à la recherche de cibles
* La détection des cibles s’effectue beaucoup plus rapidement
* Les images d’étude ne peuvent pas générer de fausses détections de cibles

Si **aucune** image n’est cochée, Chloros passe en mode de balayage de toutes les images du projet :

* Les algorithmes de détection des cibles s’appliquent à chaque image
* Des centaines, voire des milliers d’images sont analysées inutilement
* Le traitement prend beaucoup plus de temps, en particulier pour les grands ensembles de données

{% hint style="success" %}
**Amélioration des performances** : le marquage de 2 à 3 images cibles dans un ensemble de données de 500 images peut réduire le temps de détection des cibles de plus de 30 minutes à moins d’une minute.
{% endhint %}

***

## Comment marquer les images cibles

### Étape 1 : Identifiez vos images cibles

Parcourez vos images importées dans le navigateur de fichiers et identifiez celles qui contiennent des cibles d’étalonnage.

**Scénarios courants :*** **Cible pré-capture** : capturée avant le début de la session
* **Cible post-capture** : capturée après la fin de la session
* **Cibles sur le terrain** : cibles placées dans la zone de capture
* **Cibles multiples** : 2 à 3 images de cibles par session (recommandé)

### Étape 2 : Vérifier l’<img src="../.gitbook/assets/image (33).png" alt="" data-size="original"> de la colonne « Cible »

Pour chaque image contenant une cible d’étalonnage :

1. Localisez l’image dans le tableau du navigateur de fichiers
2. Repérez la colonne **Cible** (colonne la plus à droite)
3. Cochez la case de la colonne « Cible » correspondant à cette image
4. Répétez l’opération pour toutes les images contenant des cibles

### Étape 3 : Vérifiez votre sélection

Avant le traitement, vérifiez bien que :

* [ ] Toutes les images comportant des cibles d’étalonnage sont cochées
* [ ] Aucune image ne contenant pas de cible n’est cochée par erreur
* [ ] Les cibles sont clairement visibles dans les images cochées

***

## LATTICE : les cibles sont facultatives lorsqu&#x27;un système d&#x27;acquisition de données (DAQ) enregistre

Pour les caméras multispectrales LATTICE, une cible d&#x27;étalonnage intégrée à l&#x27;image est **l&#x27;une des deux** références de réflectance possibles :

* **Cible intégrée à l’image**: lorsqu’une image de cible marquée passe les filtres de qualité (QA) d’Chloros, la cible devient la**référence de réflectance absolue** pour les images qui l’entourent.
* **Rayonnement descendant du DAQ**: lorsqu’aucune cible n’est présente (ou que le contrôle qualité échoue), Chloros calcule alors la réflectance à partir de l’irradiance descendante du capteur de lumière du DAQ (ρ = π·L/E). Si un enregistrement `.daq` ou DAQ-M `.csv` couvre vos captures, vous obtenez une réflectance calibrée**sans aucune image cible**.

Ce comportement automatique est le comportement par défaut. Dans le fichier CLI / SDK, cela correspond à `--reflectance-source auto` ; vous pouvez également forcer `target` (strict — aucune substitution par le DAQ) ou `daq` (priorité au DAQ). Consultez la [Référence CLI](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Géométries des cibles LATTICE**: outre la détection classique par panneau utilisée pour l’Survey3, le traitement LATTICE prend en charge les**cibles marquées ArUco**, les**cibles à zone d’intérêt fixe**et les**cibles en bande**, configurées par projet. Des balayages de réflectance**mesurés** par unité de cible peuvent être fournis par numéro de série (CLI : `--target-reflectance-dir`, un `<serial>.csv` par unité de cible), avec les spectres T3/T4P nominaux comme solution de secours.

{% hint style="info" %}
**Module F988** : la réflectance du F988 est étalonnée à l’aide d’un panneau de réflectance intégré à la scène : la bande se situant au-delà de la plage d’étalonnage du capteur de lumière DAQ, Chloros utilise votre dernière capture de panneau et la conserve entre deux relevés du panneau. Si un module F988 est traité uniquement avec le DAQ, Chloros refuse la réflectance basée sur le DAQ pour cette bande (motif d’exclusion `dls-uncalibrated-band-988`) — le flux de travail avec la plaque est la méthode prise en charge.
{% endhint %}

***

## Bonnes pratiques pour les images de cible

### Consignes de capture de la cible

**Calendrier :**

* Capturez les images de cible immédiatement avant et tout au long de votre session de capture
* Dans les mêmes conditions d’éclairage que celles de votre capteur de lumière DAQ
* Idéalement, capturez des images de la cible aussi souvent que possible pour obtenir les meilleurs résultats. Sinon, les données du capteur de lumière seront utilisées pour ajuster l’étalonnage au fil du temps.

**Position de la caméra :**

* Tenez la caméra au-dessus de la cible de manière à ce qu’elle soit centrée et occupe environ 40 à 60 % du centre de l’image.
* Maintenez l’appareil photo parallèle ou dans la position nadirale par rapport à la surface de la cible

**Éclairage :**

* Utilisez le même éclairage ambiant que celui de votre capteur de lumière DAQ
* Évitez les ombres sur les surfaces de la cible
* Ne bloquez pas votre source lumineuse avec votre corps, votre véhicule ou la végétation
* Un ciel couvert offre les résultats les plus constants

**État de la cible :**

* Veillez à ce que les panneaux de la cible soient propres et secs.
* Tous les panneaux de votre cible (par exemple, les 4 d’un T4) doivent être clairement visibles et dégagés.
* Si possible, placez les cibles perpendiculairement ou à la verticale de la source lumineuse.

### Combien d’images de la cible ?

**Minimum :**1 image de cible par session.**Recommandé :** 3 à 5 images de cible par session.**Calendrier recommandé :**

* 3 à 5 images capturées peu après le début de l’enregistrement du capteur de lumière
* Faites pivoter la caméra entre chaque prise de vue pour obtenir les meilleurs résultats
* Facultatif : périodiquement en cours de session si les conditions d’éclairage changent constamment

***

## Utilisation de plusieurs caméras

### Configurations à deux caméras

Si vous utilisez simultanément deux caméras MAPIR (par exemple, Survey3W RGN + Survey3N OCN) :

1. Capturez des images cibles avec **les deux caméras** en même temps
2. Utilisez la **même cible physique** pour les deux caméras
3. Marquez les images de la cible pour les **deux types de caméras** dans le navigateur de fichiers
4. Chloros utilisera les cibles appropriées pour l&#x27;étalonnage de chaque caméra

### Colonne « Modèle de caméra »

La colonne **« Modèle de caméra »** permet d’identifier quelles images proviennent de quelle caméra :

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* etc.

Utilisez cette colonne pour vérifier que vous avez marqué des cibles pour chaque type de caméra dans votre projet.

***

## Paramètres de détection des cibles

### Réglage de la sensibilité de détection

Si Chloros ne détecte pas correctement vos cibles, modifiez ces paramètres dans [Paramètres du projet](adjusting-project-settings.md) :**Surface minimale de l&#x27;échantillon d&#x27;étalonnage (px) :*** **Par défaut** : 25 pixels
* **Augmentez** cette valeur si vous obtenez de fausses détections sur de petits artefacts
* **Réduisez** cette valeur si les cibles ne sont pas détectées**Regroupement minimal des cibles (0-100) :*** **Par défaut** : 60
* **Augmentez** cette valeur si les cibles sont fractionnées en plusieurs détections
* **Réduisez** cette valeur si les cibles présentant des variations de couleur ne sont pas entièrement détectées

{% hint style="info" %}
**Astuce CLI** : `chloros-cli process` accepte les mêmes paramètres (`--min-target-size`, `--target-clustering`), et son indicateur `--target`/`--targets` permet de marquer un dossier d’entrée entier comme étant réservé au panneau des cibles. Consultez la [Référence CLI](../reference/cli-reference.md).
{% endhint %}

***

## Problèmes courants liés aux images cibles

### Problème : aucune cible détectée

**Causes possibles :**

* Images cibles non cochées dans l’explorateur de fichiers
* Cible trop petite dans le cadre (&lt; 30 % de l’image)
* Mauvais éclairage (ombres, reflets)
* Paramètres de détection des cibles trop stricts

**Solutions :**

1. Vérifiez que la colonne « Cible » est cochée pour les images correctes
2. Vérifiez la qualité de l’image cible dans l’aperçu
3. Recaptez les cibles si la qualité est insuffisante
4. Ajustez les paramètres de détection des cibles si nécessaire

### Problème : fausses détections de cibles

**Causes possibles :**

* Bâtiments, véhicules ou couverture végétale blancs confondus avec des cibles
* Taches claires dans la végétation
* Sensibilité de détection trop faible

**Solutions :**

1. Ne marquez que les images de cibles réelles — seules les images cochées sont analysées
2. Augmentez la surface minimale de l’échantillon d’étalonnage
3. Augmentez la valeur minimale de regroupement des cibles
4. Assurez-vous que les images de cibles ne montrent que la cible (encombrement minimal de l’arrière-plan)

***

## Liste de contrôle de vérification

Avant de lancer le traitement, vérifiez votre sélection d’images de cibles :

* [ ] Au moins 1 image de cible marquée par session (ou, pour LATTICE, un enregistrement `.daq`/`.csv` couvrant la session)
* [ ] Les cases à cocher de la colonne « Cible » sont cochées pour toutes les images de cibles
* [ ] Les images de cibles ont été capturées pendant la même période que l’étude
* [ ] Les cibles sont clairement visibles dans l’aperçu lorsque l’on clique dessus
* [ ] Tous les panneaux d’étalonnage sont visibles sur chaque image de cible
* [ ] Aucune ombre ni obstruction sur les cibles
* [ ] Pour les systèmes à double caméra : les cibles sont marquées pour les deux types de caméra

***

## Traitement sans cible

### LATTICE : avec un enregistrement DAQ

Si un capteur de lumière DAQ a enregistré l’irradiance descendante pendant vos captures LATTICE, aucune cible n’est nécessaire :

1. Importez le fichier `.daq` (ou DAQ-M `.csv`) contenant les images
2. Laissez la colonne « Cible » décochée
3. La réflectance est calculée automatiquement à partir de la référence d&#x27;irradiation descendante du DAQ
4. La radiance ne nécessite jamais de cible ni de DAQ — elle provient uniquement de l’étalonnage radiométrique d’usine de la caméra

### Traitement sans aucune référence

Vous pouvez également effectuer le traitement sans cibles et sans DAQ :

1. Laissez toutes les cases à cocher de la colonne « Cible » décochées
2. **Désactivez** « Étalonnage de la réflectance / balance des blancs » dans les paramètres du projet — la détection des cibles est alors entièrement ignorée
3. La correction du vignetage sera tout de même appliquée
4. La sortie ne sera pas étalonnée pour la réflectance absolue (LATTICE multispectral exporte toujours les produits débayérés, d&#x27;aperçu et de radiance)

{% hint style="warning" %}
**Non recommandé pour les travaux scientifiques Survey3** : sans étalonnage de la réflectance, les valeurs de pixels d’Survey3 ne représentent qu’une luminosité relative, et non des mesures scientifiques de réflectance. Utilisez des cibles d’étalonnage (ou, pour LATTICE, un capteur de lumière DAQ) pour obtenir des résultats précis et reproductibles.
{% endhint %}

***

## Étapes suivantes

Une fois que vous avez marqué vos images cibles :

1. **Vérifiez vos paramètres** - Voir [Réglage des paramètres du projet](adjusting-project-settings.md)
2. **Lancez le traitement** – Voir [Lancer le traitement](starting-the-processing.md)
3. **Suivez la progression** – Voir [Suivi du traitement](monitoring-the-processing.md)

Pour plus d’informations sur les cibles d’étalonnage elles-mêmes, voir [Cibles d’étalonnage](../calibration-targets.md).
