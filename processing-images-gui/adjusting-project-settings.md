# Réglage des paramètres du projet

Avant de traiter vos images, il est important de configurer les paramètres de votre projet en fonction des exigences de votre flux de travail. Le panneau Paramètres du projet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> vous permettent de contrôler de manière exhaustive le calibrage, les options de traitement, les indices multispectraux et les formats d&#x27;exportation.

## Accéder aux paramètres du projet

1. Ouvrez votre projet dans Chloros
2. Cliquez sur l&#x27;icône **Paramètres du projet** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau Paramètres du projet affiche toutes les options de configuration

{% hint style=&quot;info&quot; %}
**Les paramètres sont enregistrés automatiquement** avec votre projet. Lorsque vous rouvrez un projet, tous les paramètres sont restaurés.
{% endhint %}

***

## Configuration rapide pour les flux de travail courants

### Paramètres par défaut (recommandés pour la plupart des utilisateurs)

Pour les flux de travail typiques avec une caméra MAPIR Survey3, les paramètres par défaut fonctionnent bien :

* ✅ **Correction du vignettage** : activée
* ✅ **Calibrage de la réflectance** : activé (nécessite des images de cibles MAPIR)
* ✅ **Méthode de débayérisation** : haute qualité (plus rapide)
* ✅ **Format d&#x27;exportation** : TIFF (16 bits)

Il vous suffit d&#x27;importer vos images et de lancer le traitement avec ces paramètres par défaut.

***

## Présentation des paramètres du projet

Le panneau Paramètres du projet est organisé en plusieurs catégories. Vous trouverez ci-dessous un résumé de chaque section. Pour une documentation complète, consultez [Paramètres du projet](../project-settings/project-settings.md).

### Détection des cibles

Contrôle la manière dont Chloros identifie les cibles d&#x27;étalonnage dans vos images.

**Paramètres clés :**

* **Zone d&#x27;échantillonnage minimale pour l&#x27;étalonnage** : seuil de taille pour la détection des cibles (valeur par défaut : 25 pixels)
* **Regroupement minimal des cibles** : seuil de similarité pour le regroupement des régions cibles (valeur par défaut : 60)

**Quand ajuster :**

* Augmentez la zone d&#x27;échantillonnage si vous obtenez de fausses détections.
* Diminuez-la si les cibles ne sont pas détectées.
* Ajustez le regroupement si les cibles sont divisées en plusieurs détections.

### Traitement

Options principales de traitement et d&#x27;étalonnage des images.

**Paramètres clés :**

* **Correction du vignettage** : compense l&#x27;assombrissement de l&#x27;objectif sur les bords ✅ Recommandé
* **Étalonnage de la réflectance** : normalise les valeurs à l&#x27;aide de cibles d&#x27;étalonnage ✅ Recommandé
* **Méthode de débayérisation** : algorithme de conversion du format RAW en multispectral à 3 canaux
* **Intervalle minimum de recalibrage** : temps entre l&#x27;utilisation des cibles de calibrage (0 = utiliser toutes)

**Paramètres avancés :**

* **Décalage horaire du capteur de lumière** : pour la synchronisation horaire PPK (par défaut : 0)
* **Appliquer les corrections PPK** : utilise les données GPS/exposition des fichiers .daq
* **Broche d&#x27;exposition 1/2** : affecte les caméras aux broches d&#x27;exposition pour les configurations à deux caméras

### Indice (indices multispectraux)

Configurez les indices de végétation à calculer et à exporter.

**Comment ajouter des indices :**

1. Cliquez sur le bouton **« Ajouter un indice »**
2. Sélectionnez un indice dans le menu déroulant (NDVI, NDRE, GNDVI, etc.)
3. Configurez les paramètres de visualisation (couleurs LUT, plages de valeurs)
4. Ajoutez plusieurs indices si nécessaire

**Indices populaires :**

* **NDVI** : santé générale de la végétation (le plus courant)
* **NDRE** : détection précoce du stress avec RedEdge
* **GNDVI** : sensible à la concentration en chlorophylle
* **OSAVI** : fonctionne bien avec les sols visibles
* **EVI** : régions à indice de surface foliaire élevé (LAI)

**Formules personnalisées (Chloros+ uniquement) :**

* Créez des formules d&#x27;indice multispectral personnalisées
* Utilisez des calculs de bande avec tous les canaux d&#x27;image
* Enregistrez les formules personnalisées pour les réutiliser

Pour tous les indices et formules disponibles, consultez [Formules d&#x27;indice multispectral](../project-settings/multispectral-index-formulas.md).

### Exporter

Contrôle le format et la qualité du fichier de sortie.

**Formats disponibles :**

* **TIFF (16 bits)** : recommandé pour les SIG et les analyses scientifiques (plage de 0 à 65 535)
* **TIFF (32 bits, pourcentage)** : valeurs de réflectance à virgule flottante (plage de 0,0 à 1,0)
* **PNG (8 bits)** : compression sans perte pour la visualisation (plage 0-255)
* **JPG (8 bits)** : fichiers les plus petits, compression avec perte (plage 0-255)

***

## Enregistrement et chargement des paramètres

### Enregistrer le modèle de projet

Créez des modèles réutilisables pour des flux de travail cohérents :

1. Configurez tous les paramètres souhaités dans le panneau Paramètres du projet.
2. Faites défiler jusqu&#x27;à la section **« Enregistrer le modèle de projet »** en bas.
3. Entrez un nom descriptif pour le modèle (par exemple, « Survey3N\_RGN\_Agriculture »).
4. Cliquez sur l&#x27;icône d&#x27;enregistrement.

**Avantages :**

* Appliquez des paramètres identiques à plusieurs projets.
* Partagez les configurations avec les membres de votre équipe.
* Maintenez la cohérence pour les enquêtes répétées.

### Charger un modèle dans un nouveau projet

Lorsque vous créez un nouveau projet :

1. Sélectionnez **« Nouveau projet »** dans le menu principal.
2. Choisissez l&#x27;option **« Charger à partir d&#x27;un modèle »**.
3. Sélectionnez votre modèle enregistré.
4. Tous les paramètres sont automatiquement appliqués.

### Répertoire de travail

Le paramètre **« Enregistrer le dossier du projet »** spécifie l&#x27;emplacement par défaut où les nouveaux projets sont créés :

* **Emplacement par défaut** : `C:\Users\[Username]\Chloros Projects`
* **Modifier l&#x27;emplacement** : cliquez sur l&#x27;icône Modifier et sélectionnez un nouveau dossier
* **Quand modifier** :
  * Lecteur réseau pour la collaboration en équipe
  * Lecteur différent avec plus d&#x27;espace de stockage
  * Structure de dossiers organisée par année/client

***

## Configuration PPK (cinématique post-traitée)

Si vous utilisez des enregistreurs DAQ MAPIR avec GPS pour une géolocalisation précise :

### Conditions préalables

* DAQ MAPIR avec module GPS (GNSS)
* Fichier journal .daq avec entrées de broches d&#x27;exposition
* Caméra connectée aux broches d&#x27;exposition DAQ pendant la session de capture

### Étapes de configuration

1. Placez le fichier journal .daq dans votre dossier de projet.
2. Dans les paramètres du projet, cochez la case **« Appliquer les corrections PPK »**.
3. Définissez le **« décalage horaire du capteur de lumière »** si nécessaire (par défaut : 0 pour UTC).
4. Attribuez les caméras aux broches d&#x27;exposition :
   * **Caméra unique** : Attribuée automatiquement à la broche 1
   * **Deux caméras** : attribuez manuellement chaque caméra à la broche appropriée

**Attribution des broches d&#x27;exposition :**

* **Broche d&#x27;exposition 1** : sélectionnez le modèle de caméra dans le menu déroulant
* **Broche d&#x27;exposition 2** : sélectionnez la deuxième caméra ou « Ne pas utiliser »
* Une même caméra ne peut pas être attribuée aux deux broches

{% hint style=&quot;warning&quot; %}
**Important** : les broches d&#x27;exposition doivent être correctement attribuées à leurs caméras respectives. Une attribution incorrecte entraînera des données de géolocalisation erronées.
{% endhint %}

***

## Scénarios avancés

### Projets multi-caméras

Lors du traitement d&#x27;images provenant de plusieurs caméras MAPIR dans un même projet :

1. Chloros détecte automatiquement chaque modèle de caméra
2. Chaque caméra obtient le profil de traitement approprié
3. PPK : attribuez manuellement à chaque caméra la broche d&#x27;exposition correcte
4. Toutes les caméras utilisent le même format d&#x27;exportation et les mêmes indices

**Exemple** : Survey3W RGN + Survey3N OCN double caméra

### Levés en accéléré ou sur plusieurs dates

Pour les levés répétés d&#x27;une même zone au fil du temps :

1. Créez un modèle avec vos paramètres standard.
2. Utilisez une configuration de cible d&#x27;étalonnage cohérente à chaque session.
3. Traitez chaque date comme un projet distinct.
4. Utilisez des paramètres identiques pour obtenir des résultats comparables.
5. Exportez dans le même format pour l&#x27;analyse temporelle.

### Ensembles de données volumineux

Pour les projets comportant de nombreuses images (plus de 500) :

* Envisagez de diviser le projet en plusieurs projets plus petits par date ou par zone.
* Utilisez le traitement parallèle Chloros+ pour obtenir des résultats plus rapides.
* Envisagez d&#x27;utiliser CLI ou API pour l&#x27;automatisation par lots.
* Ajustez l&#x27;intervalle de recalibrage minimum pour réduire le temps de détection de la cible.

***

## Vérification de vos paramètres

Avant de commencer le traitement, vérifiez les paramètres clés suivants :

* [ ] Modèle de caméra correctement détecté dans le navigateur de fichiers
* [ ] Correction du vignettage activée
* [ ] Calibrage de la réflectance activé
* [ ] Au moins une image cible de calibrage importée
* [ ] Indices multispectraux souhaités ajoutés
* [ ] Format d&#x27;exportation adapté à votre flux de travail
* [ ] Paramètres PPK configurés (si vous utilisez .daq avec des événements d&#x27;exposition)

***

## Étapes suivantes

Une fois vos paramètres configurés :

1. **Marquez les images cibles d&#x27;étalonnage** - Voir [Choisir les images cibles](choosing-target-images.md)
2. **Lancez le traitement** - Voir [Lancer le traitement](starting-the-processing.md)
3. **Surveillez la progression** - Voir [Surveiller le traitement](monitoring-the-processing.md)

Pour plus de détails sur tous les paramètres disponibles, consultez la documentation de référence [Paramètres du projet](../project-settings/project-settings.md).
