# Ajustement des paramètres du projet

Avant de traiter vos images, il est important de configurer les paramètres de votre projet en fonction des exigences de votre flux de travail. Le panneau Paramètres du projet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> offre un contrôle complet sur l'étalonnage, les options de traitement, les indices multispectraux et les formats d'exportation.

## Accès aux paramètres du projet

1. Ouvrez votre projet dans Chloros
2. Cliquez sur l'icône **Paramètres du projet** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau Paramètres du projet affiche toutes les options de configuration

{% hint style="info" %}
**Settings are saved automatically** with your project. When you reopen a project, all settings are restored.
{% endhint %}

***

## Configuration rapide pour les flux de travail courants

### Paramètres par défaut (recommandés pour la plupart des utilisateurs)

Pour les flux de travail typiques des caméras MAPIR Survey3, les paramètres par défaut fonctionnent bien :

* ✅ **Correction de la vignette** : Activé
* ✅ **Calibrage de la réflectance** : Activé (nécessite des images des cibles MAPIR)
* ✅ **Méthode de débayerisation** : Haute qualité (plus rapide)
* ✅ **Format d'exportation** : TIFF (16 bits)

Importez simplement vos images et commencez le traitement avec ces valeurs par défaut.

***

## Aperçu des paramètres du projet

Le panneau des paramètres du projet est organisé en plusieurs catégories. Vous trouverez ci-dessous un résumé de chaque section. Pour une documentation complète, voir [Project Settings](../project-settings/project-settings.md).

### Détection de la cible

Contrôle la manière dont Chloros identifie les cibles d'étalonnage dans vos images.

**Paramètres clés:**

* **Zone minimale de l'échantillon d'étalonnage** : Seuil de taille pour la détection des cibles (par défaut : 25 pixels)
* **Minimum target clustering** : Seuil de similarité pour le regroupement des régions cibles (par défaut : 60) Seuil de similarité pour le regroupement des régions cibles (par défaut : 60)

**Quand ajuster:**

* Augmenter la zone d'échantillonnage en cas de fausses détections
* Diminuer si les cibles ne sont pas détectées
* Ajuster le regroupement si les cibles sont divisées en plusieurs détections

### Traitement

Principales options de traitement et d'étalonnage de l'image.

**Paramètres clés:**

* **Correction de la vignette** : Compense l'assombrissement de l'objectif sur les bords ✅ Recommandé
* **Calibrage de la réflectance** : Normalise les valeurs à l'aide de cibles d'étalonnage ✅ Recommandé
* **Méthode de débayerisation** : Algorithme de conversion des données RAW en données multispectrales à trois canaux
* **Intervalle minimum de réétalonnage** : Temps entre l'utilisation des cibles d'étalonnage (0 = utiliser toutes)

**Paramètres avancés:**

* **Décalage du fuseau horaire du capteur de lumière** : Pour la synchronisation de l'heure PPK (par défaut : 0)
* **Appliquer les corrections PPK** : Utilise les données GPS/exposure pin des fichiers .daq
* **Épingle d'exposition 1/2** : Affecte les caméras aux axes d'exposition pour les configurations à deux caméras

index ### (indices multispectraux)

Configure les indices de végétation à calculer et à exporter.

**Comment ajouter des indices:**

1. Cliquez sur le bouton **"Ajouter un indice "**
2. Sélectionnez un index dans le menu déroulant (NDVI, NDRE, GNDVI, etc.)
3. Configurer les paramètres de visualisation (couleurs de la LUT, plages de valeurs)
4. Ajouter plusieurs indices si nécessaire

**Indices populaires:**

* **NDVI** : Santé générale de la végétation (le plus courant)
* **NDRE** : Détection précoce du stress avec RedEdge
* **GNDVI** : Concentration de chlorophylle sensible
* **OSAVI** : Fonctionne bien avec le sol visible
* **EVI** : Régions à indice de surface foliaire élevé (LAI)

**Formules personnalisées (Chloros+ uniquement):**

* Créer des formules d'index multispectrales personnalisées
* Utiliser les mathématiques de bande avec tous les canaux de l'image
* Sauvegarde des formules personnalisées pour réutilisation

Pour tous les indices et formules disponibles, voir [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

### Exportation

Contrôle le format et la qualité du fichier de sortie.

**Formats disponibles:**

* **TIFF (16 bits)** : Recommandé pour les SIG et les analyses scientifiques (plage de 0 à 65 535)
* **TIFF (32 bits, pourcentage)** : Valeurs de réflectance en virgule flottante (plage de 0,0 à 1,0)
* **PNG (8-bit)** : Compression sans perte pour la visualisation (plage 0-255)
* **JPG (8 bits)** : Fichiers les plus petits, compression avec perte (plage de 0 à 255)

***

## Sauvegarde et chargement des paramètres

### Enregistrer le modèle de projet

Créez des modèles réutilisables pour des flux de travail cohérents :

1. Configurez tous les paramètres souhaités dans le panneau Paramètres du projet
2. Faites défiler l'écran jusqu'à la section **"Enregistrer le modèle de projet "** en bas
3. Saisissez un nom de modèle descriptif (par exemple, "Survey3N\RGN\_Agriculture")
4. Cliquez sur l'icône de sauvegarde

**Benefits:**

* Appliquer des paramètres identiques à plusieurs projets
* Partager les configurations avec les membres de l'équipe
* Maintenir la cohérence pour des enquêtes répétées

### Charger un modèle sur un nouveau projet

Lors de la création d'un nouveau projet :

1. Sélectionnez **"Nouveau projet "** dans le menu principal
2. Choisissez l'option **"Charger à partir d'un modèle "**
3. Sélectionnez votre modèle sauvegardé
4. Tous les paramètres sont automatiquement appliqués

### Répertoire de travail

Le paramètre **"Save Project Folder "** indique l'endroit où les nouveaux projets sont créés par défaut :

* **Emplacement par défaut** : `C:\Users\[Username]\Chloros Projects`
* **Changer l'emplacement** : Cliquez sur l'icône d'édition et sélectionnez un nouveau dossier
* **Quand changer** :
  * Lecteur réseau pour la collaboration d'équipe
  * Lecteur différent avec plus d'espace de stockage
  * Structure de dossier organisée par année/client

***

## Configuration PPK (cinématique post-traitement)

Si vous utilisez des enregistreurs DAQ MAPIR avec GPS pour une géolocalisation précise :

### Conditions préalables

* MAPIR DAQ avec module GPS (GNSS)
* fichier journal .daq avec entrées de broches d'exposition
* Caméra connectée aux broches d'exposition du DAQ pendant la session de capture

### Étapes de configuration

1. Placez le fichier journal .daq dans le dossier de votre projet
2. Dans les paramètres du projet, activez la case à cocher **"Appliquer les corrections PPK "**
3. Définissez **"Décalage du fuseau horaire du capteur de lumière "** si nécessaire (par défaut : 0 pour UTC)
4. Attribuer des caméras aux broches d'exposition :
   * **Caméra unique** : Affectation automatique à la broche 1
   * **Caméras doubles** : Affectation manuelle de chaque caméra à la broche correcte

**Attribution de la broche d'exposition

* **Broche d'exposition 1** : Sélectionnez le modèle de l'appareil photo dans la liste déroulante
* **Broche d'exposition 2** : Sélectionnez le deuxième appareil photo ou "Ne pas utiliser"
* La même caméra ne peut pas être assignée aux deux broches

{% hint style="warning" %}
**Important**: Exposure pins must be correctly assigned to their respective cameras. Incorrect assignment will result in wrong geolocation data.
{% endhint %}

***

## Scénarios avancés

### Projets multi-caméras

Lors du traitement d'images provenant de plusieurs caméras MAPIR dans un projet :

1. Chloros détecte automatiquement chaque modèle d'appareil photo
2. Chaque caméra reçoit le profil de traitement approprié
3. PPK : assigner manuellement chaque appareil photo à l'axe d'exposition correct
4. Toutes les caméras utilisent le même format d'exportation et les mêmes indices

**Exemple** : Survey3W RGN + Survey3N OCN équipement à deux caméras

### Enquêtes à intervalles réguliers ou à dates multiples

Pour des études répétées de la même zone au fil du temps :

1. Créez un modèle avec vos paramètres standard
2. Utilisez la même configuration de cible d'étalonnage à chaque session
3. Traiter chaque date comme un projet distinct
4. Utiliser des paramètres identiques pour obtenir des résultats comparables
5. Exporter dans le même format pour l'analyse temporelle

### Grands ensembles de données

Pour les projets comportant de nombreuses images (500+) :

* Envisagez de diviser les projets en plus petits projets par date ou par zone
* Utilisez Chloros+ le traitement parallèle pour des résultats plus rapides
* Envisager CLI ou API pour l'automatisation des lots
* Ajuster l'intervalle minimum de recalibration pour réduire le temps de détection de la cible

***

## Vérification des paramètres

Avant de commencer le traitement, vérifiez ces paramètres clés :

* [Le modèle de l'appareil photo est correctement détecté dans le navigateur de fichiers
* [Correction de la vignette activée
* [L'étalonnage de la réflectance est activé
* [Au moins une image cible d'étalonnage a été importée
* [ ] Indices multispectraux souhaités ajoutés
* [Format d'exportation approprié pour votre flux de travail
* [Paramètres PPK configurés (si vous utilisez .daq avec des événements d'exposition)

***

## Prochaines étapes

Une fois vos paramètres configurés :

1. **Marquer les images cibles d'étalonnage** - Voir [Choosing Target Images](choosing-target-images.md)
2. **Lancer le traitement** - Voir [Starting the Processing](starting-the-processing.md)
3. **Surveiller la progression** - Voir [Monitoring the Processing](monitoring-the-processing.md)

Pour plus de détails sur tous les paramètres disponibles, voir la documentation de référence [Project Settings](../project-settings/project-settings.md).
