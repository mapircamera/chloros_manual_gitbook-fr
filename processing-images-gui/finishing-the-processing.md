# Fin du traitement

Une fois que Chloros a terminé le traitement, il est temps d&#x27;examiner vos résultats, de vérifier la qualité de la sortie et de préparer vos images traitées pour les utiliser dans votre flux de travail. Cette page vous guide à travers les dernières étapes et les actions suivantes.

## Indication de fin de traitement

Lorsque le traitement est terminé, plusieurs indicateurs s&#x27;affichent :

* ✅ **Barre de progression** : atteint 100 %
* ✅ **Journal de débogage** : affiche le message « Traitement terminé »
* ✅ **Bouton Démarrer** : redevient actif (prêt pour le prochain traitement)
* ✅ **Fichiers de sortie** : toutes les images traitées sont enregistrées dans le sous-dossier du modèle d&#x27;appareil photo

***

## Localisation de vos images traitées

### Ouverture du dossier de sortie

1. Cliquez sur l&#x27;icône **Menu principal** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (en haut à gauche)
2. Sélectionnez **« Ouvrir le dossier du projet »**
3. Votre explorateur de fichiers s&#x27;ouvre sur le répertoire du projet
4. Localisez votre projet par son nom

***

## Examiner les images traitées

### Aperçu rapide dans l&#x27;explorateur de fichiers

**Aperçu intégré à Windows :**

1. Accédez au sous-dossier du modèle d&#x27;appareil photo
2. Sélectionnez un fichier image
3. L&#x27;aperçu s&#x27;affiche dans le volet d&#x27;aperçu de l&#x27;explorateur Windows
4. Utilisez les touches fléchées pour parcourir les images

### Aperçu dans des visionneuses d&#x27;images externes

**Visionneuses recommandées :**

* **QGIS** - Logiciel SIG gratuit (idéal pour l&#x27;analyse multispectrale géoréférencée)
* **IrfanView** - Visionneuse d&#x27;images rapide et légère (prend en charge TIFF)
* **Adobe Photoshop** - Édition professionnelle (prise en charge de TIFF)
* **GIMP** - Alternative gratuite à Photoshop
* **Windows Photos** - Visualisation de base (peut ne pas prendre en charge le format 16 bits TIFF)

### Aperçu dans Chloros Image Viewer

Utilisez la visionneuse d&#x27;images intégrée à Chloros pour une visualisation avancée :

1. Cliquez sur une vignette d&#x27;image dans le navigateur de fichiers.
2. L&#x27;image s&#x27;ouvre dans la zone d&#x27;aperçu principale.
3. Cliquez sur l&#x27;onglet **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> dans la barre latérale gauche.
4. Utilisez [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) pour une analyse interactive.

Consultez [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) pour obtenir des instructions détaillées.

***

## Examen du journal de débogage

### Vérifier les avertissements ou les erreurs

1. Ouvrez l&#x27;onglet **Journal de débogage** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> .
2. Faites défiler les messages.
3. Recherchez les avertissements jaunes ou les erreurs rouges.
4. Vérifiez tous les problèmes signalés.
5. Contactez l&#x27;assistance MAPIR pour obtenir de l&#x27;aide.

### Enregistrement du journal

Pour conserver une trace du traitement ou pour l&#x27;envoyer à l&#x27;assistance MAPIR :

1. Cliquez sur le bouton **« Copier »** ou **« Télécharger »**.
2. Enregistrez le fichier au format texte dans le dossier du projet.
3. Joignez-le à la documentation du projet.
4. Envoyez-le au support MAPIR si vous rencontrez des problèmes.

***

## Problèmes courants liés à la sortie et solutions

### Problème : fichiers de sortie manquants

**Causes possibles :**

* Les fichiers ne répondaient pas aux critères de traitement.
* Images cibles uniquement (exclues de l&#x27;exportation).
* Espace disque insuffisant pendant l&#x27;exportation.
* Fichiers corrompus pendant le traitement.

**Solutions :**

1. Vérifiez le journal de débogage pour voir s&#x27;il contient des messages d&#x27;erreur ou de saut.
2. Vérifiez que l&#x27;espace disque était suffisant.
3. Comptez les fichiers : le nombre doit correspondre à (nombre d&#x27;origine - nombre cible) × (indices + 1)
4. Réimportez et retraitement tous les fichiers manquants.

### Problème : bords sombres ou clairs (vignettage toujours visible)

**Causes possibles :**

* Correction du vignettage désactivée.
* Appareil photo/objectif non répertorié dans la base de données de profils Chloros.
* Vignettage extrême dépassant les capacités de correction.

**Solutions :**

1. Vérifiez que la correction du vignettage a été activée dans les paramètres du projet.
2. Vérifiez que le modèle de caméra a été correctement détecté.
3. Contactez l&#x27;assistance MAPIR si le vignettage persiste.

### Problème : couleurs ou valeurs incorrectes

**Causes possibles :**

* Aucune cible d&#x27;étalonnage détectée.
* Modèle de cible d&#x27;étalonnage incorrect sélectionné.
* Étalonnage de la réflectance désactivé.
* Images cibles de mauvaise qualité.

**Solutions :**

1. Vérifiez que l&#x27;étalonnage de la réflectance est activé.
2. Vérifiez les messages « Cible trouvée » dans le journal de débogage.
3. Vérifiez la qualité des images cibles.
4. Relancez le traitement en marquant les cibles appropriées.

### Problème : les valeurs NDVI semblent incorrectes

**Plages NDVI attendues :**

* **Eau, roches, sol** : -0,1 à 0,2
* **Végétation clairsemée/en mauvaise santé** : 0,2 à 0,4
* **Végétation modérée** : 0,4 à 0,6
* **Végétation dense et en bonne santé** : 0,6 à 0,9

**Si les valeurs sont en dehors de ces plages :**

1. Vérifiez que l&#x27;étalonnage de la réflectance a été appliqué.
2. Vérifiez que le journal du capteur de lumière a été inclus.
3. Vérifiez que les cibles d&#x27;étalonnage ont été détectées.
4. Assurez-vous que le modèle d&#x27;appareil photo correct a été détecté.
5. Vérifiez le moment et les conditions de capture de l&#x27;image cible.

***

## Utilisation de vos images traitées

### Pour la photogrammétrie / la création d&#x27;orthomosaïques

**Workflow recommandé :**

1. **Importez les images de réflectance calibrées** dans un logiciel de photogrammétrie :
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conservez les métadonnées EXIF** : assurez-vous que les données GPS sont conservées pour le géomarquage.
3. **Flux de travail calibrés** : utilisez des images de réflectance pour une précision scientifique.
4. **Traitez les mosaïques d&#x27;index** : Créez des orthomosaïques NDVI à partir d&#x27;images d&#x27;index individuelles
5. **Exportez les GeoTIFF géoréférencés** : pour une utilisation dans des applications SIG

### Pour l&#x27;analyse SIG

**Workflow recommandé :**

1. **Chargez dans QGIS, ArcGIS ou similaire**
2. **Utiliser des images de réflectance 16 bits TIFF** pour l&#x27;analyse multibande
3. **Utiliser des images d&#x27;index** (NDVI, NDRE) comme couches de végétation prêtes à l&#x27;emploi
4. **Calculateur raster** : combiner les bandes pour une analyse personnalisée
5. **Exportation** : créez des cartes de classification, détectez les changements, créez des cartes de santé de la végétation

### Pour l&#x27;analyse directe / la création de rapports

**Workflow recommandé :**

1. **Utilisez des images d&#x27;index avec des couleurs LUT** pour les rapports visuels
2. **Extrayez des statistiques** : moyenne NDVI par champ/parcelle
3. **Séries chronologiques** : comparez les indices sur plusieurs sessions
4. **Générez des rapports** : incluez des cartes, des statistiques et des visualisations

***

## Archivage et sauvegarde

### Stratégie de sauvegarde recommandée

**À sauvegarder :**

* ✅ **Images RAW/JPG originales** - Archivez-les sur un disque/cloud séparé
* ✅ **Résultats traités** - Conserver les images calibrées et les indices
* ✅ **Fichier de projet** - Contient tous les paramètres pour un nouveau traitement si nécessaire
* ✅ **Journal de débogage** - Documente les détails du traitement
* ✅ **Images cibles de calibrage** - Pour vérification et nouveau traitement

**Recommandations de stockage :**

* **Sauvegarde immédiate** : disque dur externe
* **Archivage à long terme** : stockage dans le cloud (Google Drive, Dropbox, etc.)
* **Données critiques** : conservez 2 à 3 copies à différents emplacements

***

## Prochains traitements

### Réutilisation des paramètres du projet

Si vous traitez des ensembles de données similaires à l&#x27;avenir :

1. **Enregistrez le modèle de projet** (si ce n&#x27;est déjà fait)
2. **Créez un nouveau projet** à l&#x27;aide du modèle enregistré
3. **Importez de nouvelles images**
4. **Traitez** avec des paramètres identiques pour plus de cohérence

### Traitement par lots de plusieurs sessions

Pour plusieurs sessions/ensembles de données :

**Option 1 : GUI - Projets multiples**

* Créez un projet distinct pour chaque session.
* Utilisez des paramètres de modèle cohérents.
* Traitez les projets un par un.

**Option 2 : Chloros CLI (Chloros+ uniquement)**

* Automatisez le traitement par lots.
* Traitez plusieurs dossiers à l&#x27;aide de scripts.
* Voir [Documentation CLI](../CLI.md)

**Option 3 : Python SDK (Chloros+ uniquement)**

* Contrôle programmatique
* Intégration avec des pipelines d&#x27;analyse
* Voir [Documentation API](../api-python-sdk.md)

***

## Dépannage du post-traitement

### Retraitement avec des paramètres différents

Si les résultats ne sont pas satisfaisants :

1. Conservez les images originales (ne les supprimez jamais)
2. Ouvrez le même projet dans Chloros
3. Ajustez les paramètres dans le panneau Paramètres du projet
4. Traitez à nouveau - les résultats écraseront les résultats précédents

### Traitement d&#x27;un sous-ensemble d&#x27;images

Pour retraiter uniquement des images spécifiques :

1. Créez un nouveau projet
2. Importez uniquement les images nécessitant un nouveau traitement
3. Utilisez le même modèle de paramètres
4. Traitez un ensemble de données plus petit

### Obtenir de l&#x27;aide

Si vous rencontrez des problèmes :

* 📧 **E-mail** : info@mapir.camera (inclure le journal de débogage)
* 🌐 **Assistance** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ** : [Foire aux questions](../faq.md)
* 📖 **Documentation** : [Manuel Chloros](../)

***

## Résumé : workflow complet

Vous avez maintenant terminé le flux de travail complet de traitement Chloros :

1. ✅ **Projet créé** - Voir [Projets](../projects.md)
2. ✅ **Fichiers ajoutés** - Voir [Ajouter des fichiers](adding-files-to-a-project.md)
3. ✅ **Paramètres ajustés** - Voir [Ajuster les paramètres du projet](adjusting-project-settings.md)
4. ✅ **Cibles marquées** - Voir [Choisir les images cibles](choosing-target-images.md)
5. ✅ **Lancement du traitement** - Voir [Lancement du traitement](starting-the-processing.md)
6. ✅ **Suivi de la progression** - Voir [Suivi du traitement](monitoring-the-processing.md)
7. ✅ **Résultats examinés** - Cette page

**Vos images multispectrales calibrées et corrigées en termes de réflectance sont prêtes à être analysées !**

***

## Ressources supplémentaires

### Fonctionnalités avancées

* [**Visionneuse d&#x27;images**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualisation et analyse interactives
* [**Sandbox d&#x27;index/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Test d&#x27;index personnalisé
* [**Formules d&#x27;index multispectral**](../project-settings/multispectral-index-formulas.md) - Référence complète des index

### Automatisation et intégration

* [**Documentation CLI**](../CLI.md) - Traitement par lots en ligne de commande
* [**Python SDK**](../api-python-sdk.md) - Automatisation programmatique
* [**Fonctionnalités Chloros+**](../#chloros) - Capacités de traitement avancées

### Assistance et apprentissage

* [**FAQ**](../faq.md) - Réponses aux questions courantes
* [**Cibles d&#x27;étalonnage**](../calibration-targets.md) - Comprendre l&#x27;étalonnage de la réflectance
* [**Appareils photo pris en charge**](../supported-cameras.md) - Matériel compatible
