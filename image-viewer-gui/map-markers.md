# Repères cartographiques

L&#x27;onglet « Carte » affiche vos images sur une carte 2D interactive à partir de leurs coordonnées GPS. Il vous offre une vue d&#x27;ensemble géographique d&#x27;une session de prise de vue et constitue le moyen le plus rapide, juste après l&#x27;importation, d&#x27;écarter les images que vous ne souhaitez pas traiter.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Accéder à l&#x27;onglet « Carte »

1. Ouvrez ou créez un projet dans Chloros
2. Importez des images contenant des métadonnées GPS
3. Cliquez sur l&#x27;onglet **Carte** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> dans la barre latérale gauche
4. La carte affiche un marqueur à l&#x27;emplacement GPS de chaque image

{% hint style="info" %}
**GPS requis** : seules les images dont les métadonnées EXIF contiennent des coordonnées GPS apparaissent sur la carte. Une image sans coordonnées reste dans le projet et est traitée normalement — elle n’affiche simplement pas de marqueur.
{% endhint %}

***

## Modification des images depuis l’onglet « Carte »

L’<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> **Carte**comporte les mêmes boutons d’ajout <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> et de suppression <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> de fichiers que l’onglet [**Explorateur de fichiers**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Elle affiche la même liste de fichiers de projet, avec des colonnes géographiques :

| Colonne        | Contenu                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nom**      | Nom du fichier tel qu’il a été enregistré par l’appareil photo                             |
| **Latitude**  | Degrés décimaux, six décimales                                |
| **Longitude** | Degrés décimaux, six décimales                                |
| **Altitude**  | Mètres, une décimale — `-` lorsque l&#x27;image ne comporte pas d&#x27;altitude |

{% hint style="info" %}
Cliquez sur l&#x27;en-tête d&#x27;une colonne pour trier les données selon ce critère ; cliquez à nouveau pour inverser l&#x27;ordre.
{% endhint %}

{% hint style="warning" %}
**L&#x27;altitude correspond à la hauteur au-dessus du niveau de la mer, et non à la hauteur au-dessus du sol.** Cette valeur provient de la balise EXIF `GPSAltitude` de l&#x27;image, qui fait référence au niveau moyen de la mer. Il ne s’agit pas de l’altitude de vol au-dessus du sol, et Chloros ne calculera pas la distance d’échantillonnage au sol à partir de cette valeur — au-dessus d’un champ situé à 300 m au-dessus du niveau de la mer, un drone volant à 100 m AGL enregistrera ici environ 400 m. Utilisez cette colonne pour repérer les valeurs aberrantes et vérifier la cohérence de l’altitude de vol, et non comme une mesure AGL.
{% endhint %}

***

## Marqueurs d’images

Chaque image comportant des données GPS est associée à un marqueur situé à ses coordonnées.

### Affichage des marqueurs

* Les marqueurs se trouvent exactement aux coordonnées enregistrées pour chaque prise de vue
* Les marqueurs proches les uns des autres peuvent se chevaucher visuellement lorsque l&#x27;on effectue un zoom arrière — effectuez un zoom avant pour les distinguer
* Les marqueurs sélectionnés et mis en surbrillance s&#x27;affichent au-dessus des autres

### Aperçu au survol

* **Survolez** n’importe quel marqueur pour afficher une vignette de cette image avec son nom de fichier
* **Cliquez**sur un marqueur pour sélectionner l’image et**fixer** la fenêtre contextuelle — celle-ci reste affichée jusqu’à ce que vous cliquiez ailleurs. Tant que la fenêtre contextuelle est fixée, le fait de survoler d’autres marqueurs ne la fait pas disparaître
* C’est le moyen le plus rapide de trouver une image particulière dans une session volumineuse sans quitter la carte

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>L&#x27;onglet « Carte » affiche toutes les images géolocalisées du projet</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM** : lorsque vous atteignez le zoom maximal pour lequel le fournisseur de tuiles dispose d’images, un zoom supplémentaire agrandit les tuiles au lieu de s’arrêter, ce qui vous permet de distinguer les marqueurs qui se superposent presque les uns aux autres.
{% endhint %}

* Le super-zoom ne s&#x27;active que lorsque vous êtes **au** niveau du zoom maximal du fournisseur pour cet emplacement et que le chargement des tuiles est terminé. En dessous de ce niveau, le zoom fonctionne normalement
* La plage va de **1× à 32×** au-delà du zoom maximal du fournisseur
* Un indicateur dans le coin affiche le niveau actuel de super-zoom sous forme de pourcentage, et un bouton **×** situé à côté vous permet de revenir au zoom normal en un seul clic
* Le dézoom passe toujours par la carte elle-même, vous ne pouvez donc jamais rester bloqué en mode super-zoom
* Le zoom et le panoramique en mode super-zoom transmettent le décalage résultant à la carte ; ainsi, la zone décentrée vers laquelle vous vous êtes déplacé continue de demander des tuiles au lieu de s&#x27;afficher en blanc
* Les marqueurs sont dessinés sous forme d’éléments vectoriels plutôt que pixellisés, ce qui leur permet de rester nets à tous les niveaux de super-zoom

***

## Fournisseurs de tuiles cartographiques

{% hint style="success" %}
**Sélection automatique** : Chloros choisit le service de tuiles offrant le meilleur niveau de zoom quel que soit l’emplacement de vos images. Vous pouvez changer manuellement de service à tout moment.
{% endhint %}

| Fournisseur        | Remarques                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Large couverture mondiale ; prend en charge les quatre types de tuiles                                                                                                            |
| **Esri ArcGIS**| Images aériennes souvent de plus haute résolution dans certaines régions. Le type de tuile**Terrain** n&#x27;est pas proposé pour Esri et son bouton est désactivé lorsque Esri est sélectionné |***

## Types de tuiles cartographiques

Choisissez le type de couche cartographique à l’aide des boutons (de gauche à droite) :

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Type                 | Affiche                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terrain**          | Ombrage d&#x27;altitude avec détails cartographiques (routes, libellés). Google uniquement       |
| **Carte**              | Tuiles de carte routière standard — l’option nécessitant le moins de bande passante              |
| **Satellite**        | Images satellite détaillées, sans libellés — l’option nécessitant le plus de bande passante |
| **Hybride** (par défaut) | Images satellite sur lesquelles sont superposées les routes et les libellés                |

L&#x27;onglet « Carte » s&#x27;ouvre sur le mode **Hybride**. Votre choix s&#x27;applique également au changement de fournisseur lorsque celui-ci le prend en charge.***

## Navigation sur la carte

* **Zoom** : molette de la souris ou boutons de zoom sur la carte
* **Panoramique** : cliquer et faire glisser
* **Plein écran** : la commande « Plein écran » agrandit la carte pour occuper toute la fenêtre***

## Cas d&#x27;utilisation

### Vérification de la trajectoire de vol

* Visualiser d’un seul coup d’œil la zone couverte par une session de drone
* Repérer les lacunes là où un passage a été manqué
* Vérifier que le vol a bien suivi le tracé prévu

### Vérification d’un levé au sol

* Visualiser la répartition des prises de vue au sol
* Localiser les cadres de cibles d’étalonnage par rapport à la zone de levé
* Déterminer où des prises de vue supplémentaires sont nécessaires

### Contrôle qualité

* Repérer les images capturées à un endroit inattendu et les supprimer avant le traitement
* Trier par altitude pour repérer une image capturée à une mauvaise hauteur, ou une image pour laquelle la position GPS était imprécise
* Recouper les emplacements des images avec les notes de terrain

***

## Dépannage

### Aucun repère n’apparaît

**Causes possibles**

* Les images ne contiennent pas de métadonnées GPS
* Le GPS était désactivé sur l’appareil photo lors de la prise de vue
* Les données EXIF ont été supprimées par un autre logiciel avant l’importation

**Que faire** : vérifiez que le GPS est activé sur l’appareil photo et réimportez les fichiers d’origine. Vous pouvez vérifier si un fichier spécifique possède des coordonnées en le recherchant dans le tableau des fichiers de l’onglet « Carte » — une image sans coordonnées n’apparaîtra pas dans ce tableau.

### Les marqueurs sont mal placés

**Causes possibles** : une mauvaise acquisition du signal satellite au moment de la prise de vue, ou une dérive du GPS pendant la session.**Que faire**: il s’agit d’un problème lié au moment de la prise de vue, que Chloros ne peut pas corriger a posteriori. Pour un travail de précision, utilisez un flux de travail GPS PPK/RTK — consultez le paramètre**Appliquer les corrections PPK** dans [Paramètres du projet](../project-settings/project-settings.md).

### La carte est vierge ou le chargement des tuiles s&#x27;arrête

Les fournisseurs de tuiles sont des services en ligne. Si les tuiles cessent d’arriver, vérifiez la connexion réseau de l’appareil, puis essayez de changer de fournisseur. Si vous aviez un zoom très poussé, appuyez sur le bouton de réinitialisation **×** pour revenir à un niveau de zoom normal et laissez la carte redemander les tuiles.***

## Pages associées

* [**Grille d’images**](image-grid.md) — le même ensemble d’images que les vignettes
* [**Ouverture d’une image en plein écran**](opening-an-image-full-screen.md) — inspection détaillée d’une image
* [**Ajout de fichiers à un projet**](../processing-images-gui/adding-files-to-a-project.md) — les boutons d’ajout et de suppression de fichiers communs à cet onglet
