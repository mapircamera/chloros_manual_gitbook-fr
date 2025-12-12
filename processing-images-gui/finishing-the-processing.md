# Fin du traitement

Une fois que Chloros a terminé le traitement, il est temps d'examiner les résultats, de vérifier la qualité de la sortie et de préparer les images traitées pour les utiliser dans votre flux de travail. Cette page vous guide dans les étapes finales et les actions suivantes.

## Indication de traitement terminé

Lorsque le traitement est terminé avec succès, plusieurs indicateurs s'affichent :

* ✅ **Barre de progression** : Le traitement est terminé à 100 %
* ✅ **Journal de débogage** : Affiche le message "Traitement terminé
* **Bouton de démarrage** : Redevient actif (prêt pour le traitement suivant)
* fichiers de sortie** : Toutes les images traitées sont enregistrées dans le sous-dossier du modèle de l'appareil photo

***

## Localiser vos images traitées

### Ouverture du dossier de sortie

1. Cliquez sur l'icône **Menu principal** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (en haut à gauche)
2. Sélectionnez **"Ouvrir le dossier du projet "**
3. Votre explorateur de fichiers s'ouvre sur le répertoire du projet
4. Localisez votre projet par son nom

***

## Examen des images traitées

### Aperçu rapide dans l'explorateur de fichiers

**Windows aperçu intégré:**

1. Naviguer vers le sous-dossier du modèle de l'appareil photo
2. Sélectionner un fichier image
3. Un aperçu apparaît dans le volet d'aperçu de l'explorateur
4. Utiliser les touches fléchées pour parcourir les images

### Prévisualisation dans les visionneuses d'images externes

**Visionneurs recommandés:**

* **QGIS** - Logiciel SIG gratuit (le meilleur pour l'analyse multispectrale géoréférencée)
* **IrfanView** - Visionneuse d'images rapide et légère (supporte TIFF)
* **Adobe Photoshop** - Edition professionnelle (prise en charge TIFF)
* **GIMP** - Alternative gratuite à Photoshop
* **Windows Photos** - Visualisation de base (peut ne pas prendre en charge les TIFF 16 bits)

### Aperçu dans Chloros Image Viewer

Utilisez la visionneuse d'images intégrée de Chloros pour une visualisation avancée :

1. Cliquez sur une vignette d'image dans le navigateur de fichiers
2. L'image s'ouvre dans la zone de prévisualisation principale
3. Cliquez sur l'onglet **Visionneuse d'images** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> dans la barre latérale gauche
4. Utilisez [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) pour une analyse interactive

Voir [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) pour des instructions détaillées.

***

## Examen du journal de débogage

### Vérifier la présence d'avertissements ou d'erreurs

1. Ouvrir l'onglet **Journal de débogage** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Faire défiler les messages
3. Rechercher les avertissements jaunes ou les erreurs rouges
4. Examiner les problèmes constatés
5. Contacter le support MAPIR pour obtenir de l'aide

### Sauvegarde du journal

Pour conserver une trace du traitement ou pour l'envoyer à l'assistance MAPIR :

1. Cliquez sur le bouton **"Copier "** ou **"Télécharger "**
2. Sauvegarder comme fichier texte dans le dossier du projet
3. Inclure dans la documentation du projet
4. Envoyer au support MAPIR si des problèmes sont rencontrés

***

## Problèmes de sortie courants et solutions

### Problème : Fichiers de sortie manquants

**Causes possibles:**

* Les fichiers ne répondent pas aux critères de traitement
* Images cibles uniquement (exclues de l'exportation)
* L'espace disque a été épuisé pendant l'exportation
* Corruption de fichier pendant le traitement

**Solutions:**

1. Vérifier le journal de débogage pour les messages de saut ou d'erreur
2. Vérifier que l'espace disque est suffisant
3. Compter les fichiers : Devrait correspondre à (nombre original - nombre cible) × (indices + 1)
4. Réimporter et retraiter les fichiers manquants

### Problème : Bords sombres ou brillants (vignettage toujours visible)

**Causes possibles:**

* Correction du vignettage désactivée
* L'appareil photo/l'objectif ne figure pas dans la base de données du profil Chloros
* Vignettage extrême dépassant la capacité de correction

**Solutions:**

1. Vérifier que la correction du vignettage a été activée dans les paramètres du projet
2. Vérifier que le modèle de l'appareil photo a été correctement détecté
3. Contacter l'assistance MAPIR si le vignettage persiste

### Problème : Couleurs ou valeurs incorrectes

**Causes possibles:**

* Aucune cible d'étalonnage détectée
* Mauvais modèle de cible d'étalonnage sélectionné
* Calibration de la réflectance désactivée
* Images de cibles de mauvaise qualité

**Solutions:**

1. Vérifier que l'étalonnage de la réflectance a été activé
2. Vérifier les messages "Cible trouvée" dans le journal de débogage
3. Vérifier la qualité de l'image de la cible
4. Retraiter avec les bonnes cibles marquées

### Problème : NDVI Les valeurs semblent incorrectes

**Les plages attendues NDVI:**

* **Eau, roches, sol** : -0,1 à 0,2
* **Végétation clairsemée/malsaine** : 0.2 à 0,4
* **Végétation modérée** : 0.4 à 0,6
* **Végétation saine et dense** : 0.6 à 0,9

**Si les valeurs se situent en dehors de ces fourchettes:**

1. Vérifier que l'étalonnage de la réflectance a été appliqué
2. Vérifier que le journal du capteur de lumière a été inclus
3. Vérifier que les cibles d'étalonnage ont été détectées
4. S'assurer que le bon modèle de caméra a été détecté
5. Examiner le moment et les conditions de la capture de l'image cible

***

## Utilisation des images traitées

### Pour la photogrammétrie / la création d'orthomosaïques

**Flux de travail recommandé:**

1. **Importer les images de réflectance calibrées** dans un logiciel de photogrammétrie :
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conserver les métadonnées EXIF** : Assurer la préservation des données GPS pour la géolocalisation
3. **Flux de travail calibrés** : Utiliser les images de réflectance pour une précision scientifique
4. **Traiter les mosaïques d'index** : Créer des orthomosaïques NDVI à partir d'images d'index individuelles
5. **Exporter des images géoréférencées GeoTIFF** : Pour utilisation dans les applications SIG

### Pour l'analyse SIG

**Flux de travail recommandé:**

1. **Charger dans QGIS, ArcGIS, ou similaire**
2. **Utiliser des images de réflectance 16 bits TIFF** pour l'analyse multibande
3. **Utiliser les images d'index** (NDVI, NDRE) comme couches de végétation prêtes à l'emploi
4. **Calculateur de trame** : Combinez les bandes pour une analyse personnalisée
5. **Exportation** : Création de cartes de classification, de détection des changements, de cartes de santé de la végétation

### Pour l'analyse directe / les rapports

**Flux de travail recommandé:**

1. **Utiliser les images d'index avec les couleurs LUT** pour les rapports visuels
2. **Extraire des statistiques** : Moyenne NDVI par champ/parcellaire
3. **Séries temporelles** : Comparer les indices sur plusieurs sessions
4. **Générer des rapports** : Inclure des cartes, des statistiques et des visualisations

***

## Archivage et sauvegarde

### Stratégie de sauvegarde recommandée

**Qu'est-ce qu'il faut sauvegarder:**

* images RAW/JPG originales** - Archivez-les sur un disque dur/cloud séparé
* ✅ **Sorties traitées** - Conservez les images calibrées et les indices
* **Fichier de projet** - Contient tous les paramètres pour le retraitement si nécessaire
* **Journal de débogage** - documente les détails du traitement
* **Images cibles d'étalonnage** - Pour la vérification et le retraitement

**Recommandations de stockage:**

* **Sauvegarde immédiate** : Disque dur externe
* **Archivage à long terme** : Stockage en nuage (Google Drive, Dropbox, etc.)
* **Données critiques** : Conservez 2 ou 3 copies dans des endroits différents

***

## Prochains cycles de traitement

### Réutilisation des paramètres du projet

Si vous traitez des ensembles de données similaires à l'avenir :

1. **Enregistrer le modèle de projet** (si ce n'est pas déjà fait)
2. **Créer un nouveau projet** en utilisant le modèle sauvegardé
3. **Importer de nouvelles images
4. **Traiter** avec des paramètres identiques pour plus de cohérence

### Traitement par lots de plusieurs sessions

Pour plusieurs sessions/ensembles de données :

**Option 1 : GUI - Projets multiples**

* Créer un projet distinct pour chaque session
* Utiliser des paramètres de modèle cohérents
* Traiter un projet à la fois

**Option 2 : Chloros CLI (Chloros+ seulement)**

* Automatiser le traitement par lots
* Traiter plusieurs dossiers à l'aide de scripts
* See [CLI Documentation](../CLI.md)

**Option 3 : Python SDK (Chloros+ seulement)**

* Contrôle programmatique
* Intégration avec les pipelines d'analyse
* See [API Documentation](../api-python-sdk.md)

***

## Résolution des problèmes liés au post-traitement

### Retraitement avec des réglages différents

Si les résultats ne sont pas satisfaisants :

1. Conserver les images originales (ne jamais les supprimer)
2. Ouvrir le même projet dans Chloros
3. Ajuster les paramètres dans le panneau Paramètres du projet
4. Répéter l'opération - les résultats écraseront les résultats précédents

### Traitement d'un sous-ensemble d'images

Pour retraiter uniquement des images spécifiques :

1. Créer un nouveau projet
2. Importer uniquement les images à retraiter
3. Utiliser le même modèle de paramètres
4. Traiter un ensemble de données plus petit

### Obtenir de l'aide

Si vous rencontrez des problèmes :

* 📧 **Email** : info@mapir.camera (inclure le journal de débogage)
* 🌐 **Support** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ** : [Foire aux questions](../faq.md)
* 📖 **Documentation** : [Chloros Manual](../)

***

## Résumé : flux de travail complet

Vous avez maintenant terminé le processus de traitement complet Chloros :

1. ✅ **Créé un projet** - Voir [Projets](../projets.md)
2. ✅ **Fichiers ajoutés** - Voir [Ajout de fichiers](ajout de fichiers à un projet.md)
3. ✅ **Paramètres ajustés** - Voir [Ajuster les paramètres du projet](ajusterlesparamètresduprojet.md)
4. ✅ **Cibles marquées** - Voir [Choix des images cibles](choosing-target-images.md)
5. ✅ **Démarrer le traitement** - Voir [Démarrer le traitement](démarrer-le-traitement.md)
6. ✅ **Suivi de la progression** - Voir [Suivi du traitement](suivi du traitement.md)
7. ✅ **Résultats examinés** - Cette page

**Vos images multispectrales calibrées et corrigées en fonction de la réflectance sont prêtes pour l'analyse !

***

## Ressources supplémentaires

### Fonctionnalités avancées

* [**Image Viewer**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualisation et analyse interactives
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Tests d'index personnalisés
* [Formules de l'indice multispectral**](../project-settings/multispectral-index-formulas.md) - Référence complète de l'indice

### Automatisation et intégration

* [**CLI Documentation**](../CLI.md) - Traitement par lots en ligne de commande
* [**Python SDK**](../api-python-sdk.md) - Automatisation programmatique
* [**Chloros+ Features**](../#chloros) - Capacités de traitement avancées

### Support et apprentissage

* [**FAQ**](../faq.md) - Réponses aux questions courantes
* [**Cibles d'étalonnage**](../calibration-targets.md) - Comprendre l'étalonnage de la réflectance
* [**Caméras compatibles**](../caméras compatibles.md) - Matériel compatible
