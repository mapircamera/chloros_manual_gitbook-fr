# Ajouter des fichiers à un projet

Une fois que vous avez créé ou ouvert un projet dans Chloros, l&#x27;étape suivante consiste à ajouter vos images multispectrales pour commencer le traitement. L&#x27;onglet Navigateur de fichiers<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilite l&#x27;importation d&#x27;images et la gestion de votre ensemble de données.

## Accéder au navigateur de fichiers

1. Ouvrez ou créez un projet dans Chloros
2. Cliquez sur l&#x27;icône **Navigateur de fichiers** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau Navigateur de fichiers affiche la liste des fichiers de votre projet

{% hint style=&quot;info&quot; %}
**Types de fichiers pris en charge** : Chloros prend en charge les fichiers image RAW+JPG et JPG provenant des appareils photo MAPIR Survey3W et Survey3N. Seuls les fichiers RAW+JPG sont recommandés.
{% endhint %}

***

## Ajouter des images à votre projet

Il existe deux méthodes principales pour ajouter des images à votre projet :

### Méthode 1 : Ajouter des fichiers

Utilisez cette option pour importer des fichiers image individuels ou une petite sélection de fichiers.

1. Cliquez sur le bouton **« Ajouter des fichiers »** en haut du panneau Navigateur de fichiers.
2. Accédez au dossier contenant vos images.
3. Sélectionnez un ou plusieurs fichiers image (maintenez la touche **Ctrl** enfoncée pour sélectionner plusieurs fichiers).
4. Cliquez sur **« Ouvrir »** pour importer les fichiers sélectionnés.

### Méthode 2 : ajouter un dossier

Utilisez cette option pour importer toutes les images d&#x27;un dossier en une seule fois.

1. Cliquez sur le bouton **« Ajouter un dossier »** en haut du panneau Navigateur de fichiers.
2. Accédez au dossier contenant les images de votre session de capture et sélectionnez-le.
3. Cliquez sur **« Sélectionner le dossier »** pour importer toutes les images prises en charge à partir de ce dossier.

***

## Comprendre le tableau du navigateur de fichiers

Une fois les images importées, elles apparaissent dans un tableau comportant les colonnes suivantes :

### Vignette

* Petit aperçu de chaque image.
* Cliquez sur la vignette pour afficher l&#x27;image en entier dans la zone d&#x27;aperçu principale.

### Nom du fichier

* Nom de fichier d&#x27;origine provenant de l&#x27;appareil photo.
* Conserve la convention de nommage de l&#x27;appareil photo (par exemple, IMG\_0001.RAW).

### Horodatage

* Date et heure de la capture de l&#x27;image.
* Extrait des métadonnées EXIF de l&#x27;image.
* Utilisé pour la synchronisation PPK et la détection des cibles d&#x27;étalonnage

### Modèle d&#x27;appareil photo

* Configuration de l&#x27;appareil photo et du filtre détectée automatiquement
* Exemples : Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Utilisé pour appliquer les profils de traitement corrects

### Colonne Cible (case à cocher)

* Cochez cette case pour les images qui contiennent des cibles d&#x27;étalonnage
* Accélère considérablement la détection des cibles pendant le traitement
* Voir [Choisir les images cibles](choosing-target-images.md) pour plus de détails

***

## Gérer les fichiers dans votre projet

### Suppression de fichiers

Pour supprimer les images indésirables de votre projet :

1. Sélectionnez une ou plusieurs images dans le tableau du navigateur de fichiers
2. Cliquez sur le bouton **« Supprimer la sélection »**
3. Confirmez la suppression (les fichiers ne sont pas supprimés du disque, mais uniquement du projet)

### Tri et filtrage

* **Trier par colonne** : cliquez sur n&#x27;importe quel en-tête de colonne pour trier les images
* **Tri par horodatage** : utile pour organiser les séquences de capture par ordre chronologique
* **Filtre par modèle d&#x27;appareil photo** : regroupez les images par type d&#x27;appareil photo si vous utilisez plusieurs appareils

***

## Aperçu des images

### Affichage de l&#x27;image complète

Cliquez sur n&#x27;importe quelle vignette d&#x27;image dans le navigateur de fichiers pour l&#x27;afficher dans la zone d&#x27;aperçu principale :

1. L&#x27;image apparaît dans le panneau d&#x27;aperçu central
2. Utilisez les commandes de zoom pour inspecter les détails de l&#x27;image
3. Naviguez entre les images à l&#x27;aide des touches fléchées

### Navigation rapide

* **Image précédente** : cliquez sur la flèche gauche ou appuyez sur la touche ←
* **Image suivante** : cliquez sur la flèche droite ou appuyez sur la touche →
* **Zoom avant/arrière** : utilisez la molette de la souris ou les boutons de zoom
* **Panoramique** : cliquez et faites glisser sur l&#x27;image lorsque vous zoomez

***

## Gestion des fichiers en double

Chloros détecte et ignore automatiquement les fichiers en double :

* Les fichiers portant des noms identiques sont ignorés.
* Évite le double traitement accidentel.
* Un message d&#x27;avertissement s&#x27;affiche lorsque des doublons sont détectés.

{% hint style=&quot;warning&quot; %}
**Important** : ne renommez pas et ne modifiez pas vos fichiers image originaux avant de les importer. Chloros s&#x27;appuie sur les noms de fichiers et les métadonnées d&#x27;origine pour un traitement correct.
{% endhint %}

***

## Ensembles de données mixtes provenant de plusieurs caméras

Si votre projet contient des images provenant de plusieurs caméras MAPIR :

1. Chloros détecte automatiquement chaque modèle de caméra.
2. Chaque type de caméra est traité avec son profil d&#x27;étalonnage approprié.
3. Le navigateur de fichiers affiche le modèle de caméra dans la colonne Modèle de caméra.
4. Le traitement applique les paramètres corrects pour chaque type de caméra.

**Exemple de scénario** : configuration à deux caméras Survey3W RGN + Survey3N OCN.

***

## Bonnes pratiques

### Organisez-vous avant l&#x27;importation

* Conservez les images cibles d&#x27;étalonnage dans le même dossier que les images de levé.
* Conservez la structure de dossiers d&#x27;origine de votre caméra/carte SD.
* Ne mélangez pas les ensembles de données provenant de différentes sessions dans un même projet.

### Nommage des fichiers

* Conservez les noms de fichiers d&#x27;origine de la caméra (IMG\_0001.RAW, etc.).
* Ne renommez pas les fichiers avant l&#x27;importation.
* Les noms d&#x27;origine contiennent des métadonnées importantes.

### Images cibles d&#x27;étalonnage

* Incluez toujours 1 à 2 images cibles d&#x27;étalonnage par session.
* Capturez les cibles avant et après la session de capture.
* Placez les cibles dans les mêmes conditions d&#x27;éclairage que la zone de capture.
* Marquez les images cibles à l&#x27;aide de la case à cocher Cible pour accélérer le traitement.

***

## Problèmes courants et solutions

### Les images n&#x27;apparaissent pas après l&#x27;importation

**Causes possibles :**

* Format de fichier non pris en charge (uniquement RAW+JPG et JPG des appareils photo MAPIR)
* Les images proviennent d&#x27;appareils photo autres que MAPIR (voir [Appareils photo pris en charge](../supported-cameras.md))
* Fichier corrompu ou transfert incomplet depuis la carte SD

**Solution** : vérifiez la compatibilité du format de fichier et du modèle d&#x27;appareil photo.

### Modèle d&#x27;appareil photo non détecté

**Causes possibles :**

* Métadonnées EXIF modifiées
* Images modifiées dans un logiciel externe
* Transfert de fichiers incomplet

**Solution** : réimportez les fichiers originaux non modifiés depuis l&#x27;appareil photo/la carte SD.

### Horodatages manquants

**Causes possibles :**

* Horloge de l&#x27;appareil photo mal réglée
* Données EXIF supprimées par un logiciel externe

**Solution** : vérifiez que les paramètres de l&#x27;heure de l&#x27;appareil photo étaient corrects lors de la capture.

***

## Étapes suivantes

Une fois vos fichiers importés :

1. **Vérifiez la liste des fichiers** - Assurez-vous que toutes les images ont été chargées correctement.
2. **Vérifiez les modèles d&#x27;appareils photo** - Vérifiez que les appareils photo ont été détectés correctement.
3. **Marquez les images cibles** - Voir [Choisir les images cibles](choosing-target-images.md)
4. **Ajustez les paramètres** - Configurez les options de traitement dans [Paramètres du projet](adjusting-project-settings.md)
5. **Lancez le traitement** - Consultez [Lancer le traitement](starting-the-processing.md)

Pour plus d&#x27;informations sur la configuration du projet, consultez [Ajuster les paramètres du projet](adjusting-project-settings.md).
