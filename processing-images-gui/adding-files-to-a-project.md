# Ajouter des fichiers à un projet

Une fois que vous avez créé ou ouvert un projet dans Chloros, l'étape suivante consiste à ajouter vos images multispectrales pour commencer le traitement. L'onglet Navigateur de fichiers <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilite l'importation d'images et la gestion de votre ensemble de données.

## Accès à l'explorateur de fichiers

1. Ouvrez ou créez un projet dans Chloros
2. Cliquez sur l'icône **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau du navigateur de fichiers affiche la liste des fichiers de votre projet

{% hint style="info" %}
**Supported File Types**: Chloros supports RAW+JPG and JPG image files from MAPIR Survey3W and Survey3N cameras. Only RAW+JPG are recommended.
{% endhint %}

***

## Ajouter des images à votre projet

Il existe deux façons principales d'ajouter des images à votre projet :

### Méthode 1 : Ajouter des fichiers

Cette option permet d'importer des fichiers d'images individuels ou une petite sélection de fichiers.

1. Cliquez sur le bouton **"Ajouter des fichiers "** en haut du panneau du navigateur de fichiers
2. Naviguez jusqu'au dossier contenant vos images
3. Sélectionnez un ou plusieurs fichiers image (maintenez la touche **Ctrl** enfoncée pour sélectionner plusieurs fichiers)
4. Cliquez sur **"Ouvrir "** pour importer les fichiers sélectionnés

### Méthode 2 : Ajouter un dossier

Utilisez cette option pour importer toutes les images d'un dossier en une seule fois.

1. Cliquez sur le bouton **"Ajouter un dossier "** en haut du panneau du navigateur de fichiers
2. Naviguez jusqu'au dossier contenant les images de votre session de capture et sélectionnez-le
3. Cliquez sur **"Sélectionner un dossier "** pour importer toutes les images prises en charge à partir de ce dossier

***

## Comprendre le tableau du navigateur de fichiers

Une fois les images importées, elles apparaissent dans un tableau comportant les colonnes suivantes :

### Vignette

* Petit aperçu de chaque image
* Cliquez sur la vignette pour afficher l'image complète dans la zone de prévisualisation principale

### Nom du fichier

* Nom de fichier original de l'appareil photo
* Maintient la convention de dénomination de l'appareil photo (par exemple, IMG\_0001.RAW)

### Horodatage

* Date et heure de la prise de vue
* Extrait des métadonnées EXIF de l'image
* Utilisées pour la synchronisation PPK et la détection des cibles d'étalonnage

### Modèle de l'appareil photo

* Détection automatique de la caméra et de la configuration du filtre
* Exemples : Survey3W\RGN, Survey3N\_OCN, Survey3W\RGB
* Utilisé pour appliquer les profils de traitement corrects

### Colonne cible (case à cocher)

* Cochez cette case pour les images contenant des cibles d'étalonnage
* Accélère considérablement la détection des cibles lors du traitement
* Voir [Choosing Target Images](choosing-target-images.md) pour plus de détails

***

## Gestion des fichiers dans votre projet

### Supprimer des fichiers

Pour supprimer des images indésirables de votre projet :

1. Sélectionnez une ou plusieurs images dans le tableau du navigateur de fichiers
2. Cliquez sur le bouton **"Supprimer la sélection "**
3. Confirmez la suppression (les fichiers ne sont pas supprimés du disque, mais seulement du projet)

### Trier et filtrer

* **Tri par colonne** : Cliquez sur n'importe quel en-tête de colonne pour trier les images
* **Tri par horodatage** : Utile pour organiser des séquences de capture chronologiques
* **Filtre de modèle d'appareil photo** : Filtre de modèle d'appareil photo** : regrouper les images par type d'appareil photo si vous utilisez plusieurs appareils photo

***

## Aperçu de l'image

### Visualisation de l'image complète

Cliquez sur une vignette d'image dans le navigateur de fichiers pour l'afficher dans la zone de prévisualisation principale :

1. L'image apparaît dans le panneau d'aperçu central
2. Utilisez les commandes de zoom pour inspecter les détails de l'image
3. Naviguer entre les images à l'aide des touches fléchées

### Navigation rapide

* **Image précédente** : Cliquez sur la flèche gauche ou appuyez sur la touche ←
* **Image suivante** : Cliquez sur la flèche droite ou appuyez sur la touche →
* **Zoom avant/arrière** : Utilisez la molette de la souris ou les boutons de zoom
* **Panoramique** : Cliquez sur l'image et faites-la glisser lorsque vous effectuez un zoom avant

***

## Gestion des fichiers en double

Chloros détecte et ignore automatiquement les fichiers en double :

* Les fichiers dont le nom est identique sont ignorés
* Empêche le double traitement accidentel
* Un message d'avertissement s'affiche lorsque des doublons sont détectés

{% hint style="warning" %}
**Important**: Do not rename or modify your original image files before importing. Chloros relies on original filenames and metadata for proper processing.
{% endhint %}

***

## Ensembles de données de caméras mixtes

Si votre projet contient des images provenant de plusieurs caméras MAPIR :

1. Chloros détecte automatiquement chaque modèle de caméra
2. Chaque type de caméra est traité avec le profil d'étalonnage approprié
3. L'explorateur de fichiers affiche le modèle de caméra dans la colonne Modèle de caméra
4. Le traitement applique les paramètres corrects pour chaque type de caméra

**Exemple de scénario** : Survey3W RGN + Survey3N OCN configuration à deux caméras

***

## Meilleures pratiques

### Organiser avant l'importation

* Conserver les images de la cible d'étalonnage dans le même dossier que les images de l'enquête
* Conservez la structure originale des dossiers de votre appareil photo/carte SD
* Ne mélangez pas des ensembles de données provenant de différentes sessions dans un même projet

### Nom des fichiers

* Conservez les noms de fichiers originaux de l'appareil photo (IMG\_0001.RAW, etc.)
* Ne pas renommer les fichiers avant l'importation
* Les noms originaux contiennent des métadonnées importantes

### Images cibles d'étalonnage

* Inclure toujours 1 à 2 images de cibles d'étalonnage par session
* Capturer les cibles avant et après la session de capture
* Placer les cibles dans les mêmes conditions d'éclairage que la zone de capture
* Marquez les images cibles à l'aide de la case à cocher Cible pour accélérer le traitement

***

## Problèmes courants et solutions

### Les images n'apparaissent pas après l'importation

**Causes possibles:**

* Format de fichier non pris en charge (uniquement RAW+JPG et JPG des appareils photo MAPIR)
* Les images proviennent d'appareils photo non MAPIR (voir [Supported Cameras](../supported-cameras.md))
* Corruption de fichier ou transfert incomplet depuis la carte SD

**Solution** : Vérifiez la compatibilité du format de fichier et du modèle d'appareil photo

### Modèle d'appareil photo non détecté

**Causes possibles:**

* Métadonnées EXIF modifiées
* Images modifiées dans un logiciel externe
* Transfert de fichier incomplet

**Solution** : Réimporter les fichiers originaux non modifiés de l'appareil photo/carte SD

### Horodatage manquant

**Causes possibles:**

* L'horloge de l'appareil photo n'est pas réglée correctement
* Données EXIF supprimées par un logiciel externe

**Solution** : Vérifiez que les réglages de l'heure de l'appareil photo étaient corrects lors de la prise de vue

***

## Prochaines étapes

Une fois vos fichiers importés :

1. **Revoir la liste des fichiers** - S'assurer que toutes les images ont été chargées correctement
2. **Vérifier les modèles d'appareil photo** - Vérifier la détection correcte de l'appareil photo
3. **Marquer les images cibles** - Voir [Choosing Target Images](choosing-target-images.md)
4. **Régler les paramètres** - Configurez les options de traitement dans [Project Settings](adjusting-project-settings.md)
5. **Lancer le traitement** - Voir [Starting the Processing](starting-the-processing.md)

Pour des informations détaillées sur la configuration du projet, voir [Adjusting Project Settings](adjusting-project-settings.md).
