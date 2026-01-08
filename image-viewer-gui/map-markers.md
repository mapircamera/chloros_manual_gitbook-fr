# Marqueurs cartographiques

L&#x27;onglet Carte affiche vos images sur une carte 2D interactive en fonction de leurs coordonnées GPS. Cela vous donne une vue d&#x27;ensemble géographique de votre session de capture et vous aide à visualiser la couverture spatiale. Cela est également utile lors de la première importation de vos images pour supprimer rapidement celles que vous n&#x27;avez pas besoin de traiter.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Accéder à l&#x27;onglet Carte

1. Ouvrez ou créez un projet dans Chloros
2. Importez des images contenant des métadonnées GPS
3. Cliquez sur l&#x27;onglet **Carte** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> dans la barre latérale gauche.
4. La carte affichera des marqueurs à l&#x27;emplacement GPS de chaque image.

{% hint style="info" %}
**GPS requis** : seules les images dont les métadonnées EXIF contiennent des coordonnées GPS apparaîtront sur la carte. Assurez-vous que le GPS de votre appareil photo est activé lors de la capture.
{% endhint %}

***

## Réglage des images à partir de l&#x27;onglet Carte

L&#x27;onglet **Carte** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> dispose des mêmes fonctions d&#x27;ajout  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  et supprimer  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  que l&#x27;onglet [**Navigateur de fichiers**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Il affiche également la même liste de fichiers de projet, mais avec des en-têtes de colonnes différents :

### Nom du fichier

* Nom de fichier d&#x27;origine de l&#x27;appareil photo
* Conserve la convention de nommage de l&#x27;appareil photo (par exemple, IMG\_0001.RAW)

### Latitude

* Latitude de l&#x27;image

### Longitude

* Longitude de l&#x27;image

### Altitude

* Altitude de l&#x27;image

{% hint style="info" %}
Cliquer sur les en-têtes des colonnes du tableau permet également de trier les données des lignes.
{% endhint %}

***

## Marqueurs d&#x27;image

Chaque image contenant des données GPS est représentée par un marqueur sur la carte :

### Affichage des marqueurs

* Les marqueurs indiquent les coordonnées GPS exactes où chaque image a été capturée.
* Les marqueurs regroupés peuvent s&#x27;afficher ensemble lorsque vous effectuez un zoom arrière.
* Effectuez un zoom avant pour voir l&#x27;emplacement de chaque image.

{% hint style="success" %}
SUPER-ZOOM : lorsque vous atteignez le niveau de zoom maximal du fournisseur de tuiles cartographiques, la tuile est alors agrandie lors d&#x27;un zoom supplémentaire, ce qui vous permet de voir les marqueurs qui sont proches les uns des autres.
{% endhint %}

### Aperçu au survol

* **Passez votre souris** sur n&#x27;importe quel marqueur pour voir un aperçu miniature de cette image.
* Cela permet une identification visuelle rapide sans quitter la vue cartographique.
* Utile pour localiser des images spécifiques dans une grande session de capture.

***

## Fournisseurs de tuiles cartographiques

{% hint style="success" %}
**Sélection automatique** : Chloros choisit automatiquement le service de tuiles qui offre le meilleur niveau de zoom pour votre emplacement actuel sur la carte. Vous pouvez passer manuellement d&#x27;un fournisseur à l&#x27;autre si vous le souhaitez.
{% endhint %}

L&#x27;onglet Carte prend en charge deux fournisseurs de tuiles pour les images de fond de carte :

### Google Maps

* Images satellite et cartographiques standard de Google.
* Idéal pour une couverture mondiale générale.

### ESRI

* Images satellite et aériennes d&#x27;ESRI ArcGIS.
* Fournit souvent des images de meilleure résolution dans certaines régions.

***

## Types de tuiles cartographiques

Vous pouvez choisir le type de couche cartographique (de gauche à droite) :

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Terrain

Affiche les profils d&#x27;altitude et les tuiles cartographiques avec des détails (routes, etc.)

### Carte

Affiche des tuiles cartographiques standard (bande passante inférieure) avec des détails (routes, etc.)

### Satellite

Affiche des tuiles cartographiques satellite détaillées (bande passante supérieure)

### Hybride

Affiche des tuiles cartographiques satellite avec des détails supplémentaires (routes, etc.)

***

## Navigation sur la carte

### Commandes de zoom

* **Zoom avant/arrière** : utilisez la molette de la souris ou les boutons de zoom.
* **Plein écran** : affichez la carte en plein écran.

### Commandes de panoramique

* **Panoramique** : cliquez et faites glisser pour vous déplacer sur la carte.

***

## Cas d&#x27;utilisation

### Visualisation de la trajectoire de vol

* Affichez la zone de couverture des sessions de capture par drone.
* Identifiez les lacunes dans la couverture des images.
* Vérifiez l&#x27;exécution de la trajectoire de vol.

### Examen des relevés au sol

* Affichez la distribution spatiale des captures au sol.
* Localisez les images cibles d&#x27;étalonnage par rapport à la zone d&#x27;étude.
* Planifiez des emplacements de capture supplémentaires.

### Contrôle qualité

* Identifiez rapidement les images capturées à des emplacements inattendus.
* Vérifiez la précision du GPS dans l&#x27;ensemble de données.
* Recoupez les emplacements des images avec les notes de terrain.

***

## Dépannage

### Aucun marqueur n&#x27;apparaît

**Causes possibles :**

* Les images ne contiennent pas de métadonnées GPS.
* Le GPS était désactivé sur l&#x27;appareil photo pendant la capture.
* Les données EXIF ont été supprimées par un logiciel externe.

**Solution** : vérifiez que le GPS est activé sur votre appareil photo et réimportez les fichiers d&#x27;origine.

### Marqueurs au mauvais emplacement

**Causes possibles :**

* Le GPS de l&#x27;appareil photo avait une mauvaise position satellite.
* Le GPS a dérivé pendant la capture.

**Solution** : il s&#x27;agit généralement d&#x27;un problème lié au moment de la capture ; envisagez d&#x27;utiliser un GPS PPK/RTK pour les applications de précision.
