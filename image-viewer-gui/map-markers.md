# Repères cartographiques

L&#x27;onglet « Carte » affiche vos images sur une carte 2D interactive en fonction de leurs coordonnées GPS. Cela vous offre une vue d&#x27;ensemble géographique de votre session de capture et vous aide à visualiser la couverture spatiale. Cet onglet est également utile lors de la première importation de vos images pour supprimer rapidement celles que vous n&#x27;avez pas besoin de traiter.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Accéder à l&#x27;onglet Carte

1. Ouvrez ou créez un projet dans Chloros
2. Importez les images contenant des métadonnées GPS
3. Cliquez sur l&#x27;onglet **Carte** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> dans la barre latérale gauche
4. La carte affichera des marqueurs à l&#x27;emplacement GPS de chaque image

{% hint style="info" %}
**GPS requis** : seules les images dont les métadonnées EXIF contiennent des coordonnées GPS s&#x27;afficheront sur la carte. Assurez-vous que le GPS de votre appareil photo est activé lors de la prise de vue.
{% endhint %}

***

## Réglage des images depuis l&#x27;onglet Carte

L&#x27;onglet **Carte**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> dispose des mêmes boutons  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  et de suppression  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  que l&#x27;onglet [**Parcourir les fichiers**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Elle affiche également la même liste de fichiers de projet, mais avec des en-têtes de colonnes différents :

### Nom du fichier

* Nom de fichier d&#x27;origine de l&#x27;appareil photo
* Respecte la convention de nommage de l&#x27;appareil photo (par exemple, IMG\_0001.RAW)

### Latitude

* Latitude de l&#x27;image

### Longitude

* Longitude de l&#x27;image

### Altitude

* L&#x27;altitude de l&#x27;image

{% hint style="info" %}
Cliquer sur les en-têtes de colonne du tableau permet également de trier les données des lignes
{% endhint %}

***

## Marqueurs d&#x27;images

Chaque image comportant des données GPS est représentée par un marqueur sur la carte :

### Affichage des marqueurs

* Les marqueurs indiquent les coordonnées GPS exactes où chaque image a été capturée
* Les marqueurs regroupés peuvent apparaître ensemble lorsque vous effectuez un zoom arrière
* Effectuez un zoom avant pour voir l&#x27;emplacement de chaque image

{% hint style="success" %}
SUPER-ZOOM : lorsque vous atteignez le niveau de zoom maximal fourni par le fournisseur de tuiles cartographiques, la tuile est alors agrandie lors d&#x27;un zoom supplémentaire, ce qui vous permet de voir les marqueurs proches les uns des autres.
{% endhint %}

### Aperçu au survol

* **Passez votre souris** sur n&#x27;importe quel marqueur pour voir un aperçu miniature de cette image
* Cela permet une identification visuelle rapide sans quitter la vue de la carte
* Utile pour localiser des images spécifiques au sein d&#x27;une grande session de capture

***

## Fournisseurs de tuiles cartographiques

{% hint style="success" %}
**Sélection automatique** : Chloros choisit automatiquement le service de tuiles offrant le meilleur niveau de zoom pour votre emplacement actuel sur la carte. Vous pouvez passer manuellement d&#x27;un fournisseur à l&#x27;autre si vous le souhaitez.
{% endhint %}

L&#x27;onglet Carte prend en charge deux fournisseurs de tuiles pour les images de fond de la carte :

### Google Maps

* Images satellite et cartographiques standard de Google
* Idéal pour une couverture mondiale générale

### ESRI

* Images satellite et aériennes provenant d&#x27;ESRI ArcGIS
* Fournit souvent des images à plus haute résolution dans certaines régions

***

## Types de tuiles cartographiques

Vous pouvez choisir le type de couche cartographique (de gauche à droite) :

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terrain

Affiche les profils d&#x27;élévation et les tuiles cartographiques avec des détails (routes, etc.)

### Carte

Affiche des tuiles cartographiques standard (bande passante réduite) avec des détails (routes, etc.)

### Satellite

Affiche des tuiles cartographiques satellite détaillées (bande passante plus élevée)

### Hybride

Affiche des tuiles cartographiques satellite avec des détails supplémentaires (routes, etc.)

***

## Navigation sur la carte

### Commandes de zoom

* **Zoom avant/arrière** : utilisez la molette de la souris ou les boutons de zoom
* **Plein écran** : affichez la carte en plein écran

### Commandes de panoramique

* **Panoramique** : cliquez et faites glisser pour vous déplacer sur la carte***

## Cas d&#x27;utilisation

### Visualisation de la trajectoire de vol

* Visualiser la zone de couverture des sessions de capture par drone
* Identifier les lacunes dans la couverture des images
* Vérifier l&#x27;exécution de la trajectoire de vol

### Examen des relevés au sol

* Voir la répartition spatiale des captures au sol
* Localiser les images de cibles d&#x27;étalonnage par rapport à la zone de relevé
* Planifier des emplacements de capture supplémentaires

### Contrôle qualité

* Identifier rapidement les images capturées à des emplacements inattendus
* Vérifier la précision du GPS sur l&#x27;ensemble des données
* Recouper les emplacements des images avec les notes de terrain

***

## Dépannage

### Aucun marqueur n&#x27;apparaît

**Causes possibles :**

* Les images ne contiennent pas de métadonnées GPS
* Le GPS était désactivé sur la caméra pendant la capture
* Les données EXIF ont été supprimées par un logiciel externe

**Solution** : Vérifiez que le GPS est activé sur votre appareil photo et réimportez les fichiers d&#x27;origine

### Repères à un emplacement incorrect

**Causes possibles :**

* Le GPS de l&#x27;appareil photo avait une mauvaise position satellite
* Dérive du GPS pendant la capture

**Solution** : Il s&#x27;agit généralement d&#x27;un problème lié au moment de la capture ; envisagez d&#x27;utiliser un GPS PPK/RTK pour les applications de précision
