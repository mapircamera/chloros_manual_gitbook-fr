# Réglage des paramètres du projet

Avant de traiter vos images, il est important de configurer les paramètres de votre projet en fonction des exigences de votre flux de travail. Le panneau « Paramètres du projet » (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) offre un contrôle complet sur l’étalonnage, les options de traitement, les indices multispectraux et les formats d’exportation.

## Accéder aux paramètres du projet

1. Ouvrez votre projet dans Chloros
2. Cliquez sur l’icône **Paramètres du projet** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> dans la barre latérale gauche
3. Le panneau « Paramètres du projet » affiche toutes les options de configuration

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Le panneau « Paramètres du projet » — Affichage, détection des cibles et traitement</p></figcaption></figure>{% hint style="info" %}
**Les paramètres sont enregistrés automatiquement** avec votre projet. Lorsque vous rouvrez un projet, tous les paramètres sont restaurés.
{% endhint %}

***

## Configuration rapide pour les flux de travail courants

### Paramètres par défaut (recommandés pour la plupart des utilisateurs)

Les paramètres par défaut conviennent parfaitement aux flux de travail Survey3 et LATTICE classiques :

* ✅ **Correction de la vignettage** : Activée
* ✅ **Étalonnage de la réflectance / balance des blancs** : Activé (utilise les cibles MAPIR et/ou les données du capteur de lumière DAQ)
* ✅ **Méthode de débayérisation** : Standard (rapide, qualité moyenne)
* ✅ **Format d’exportation** : TIFF (16 bits)
* ✅ **Tous les produits d’exportation** : Activé (LATTICE génère automatiquement les fichiers débayérisés, d’aperçu, de radiance et de réflectance)

Il vous suffit d’importer vos images et de lancer le traitement avec ces paramètres par défaut.

***

## Présentation des paramètres du projet

Le panneau **Paramètres du projet**est organisé en sections comme indiqué ci-dessous. Deux sections supplémentaires —**Capteur de lumière DAQ**et**Alignement de la matrice** — s’affichent automatiquement lorsque votre projet contient les fichiers correspondants. Pour une documentation complète, consultez [Paramètres du projet](../project-settings/project-settings.md).

### Affichage

* **Résolution des vignettes d’images**: résolution des vignettes de la grille d’images. Options :**Par défaut (512 px)**,**1 024 px**,**2 048 px**,**Résolution maximale**. À titre d’affichage uniquement — n’affecte jamais le traitement. Des valeurs plus élevées offrent un rendu plus net en zoom, mais le chargement est plus lent.

### Détection des cibles

Contrôle la manière dont Chloros identifie les cibles d’étalonnage dans vos images.

**Paramètres clés :*** **Surface minimale d’échantillonnage pour l’étalonnage (px)**: seuil de taille pour la détection des cibles (par défaut :**25**, plage 0–10 000)
* **Regroupement minimal des cibles (0-100)**: Seuil de similarité pour le regroupement des zones cibles (par défaut :**60**)**Quand ajuster :**

* Augmentez la surface d’échantillonnage en cas de fausses détections
* Diminuez-la si les cibles ne sont pas détectées
* Ajustez le regroupement si les cibles sont fractionnées en plusieurs détections

{% hint style="info" %}
Ces paramètres sont grisés lorsque l’option **« Calibrage de la réflectance / balance des blancs »** est désactivée — dans ce cas, la détection des cibles ne s’exécute jamais.
{% endhint %}

### Traitement

Options principales de traitement d&#x27;image et d&#x27;étalonnage.

**Paramètres clés :*** **Correction de la vignettage** : Compense l&#x27;assombrissement des bords dû à l&#x27;objectif ✅ Recommandé
* **Étalonnage de la réflectance / balance des blancs** : étalonne les images à l’aide des cibles détectées (Survey3) et/ou des données du capteur de lumière DAQ (LATTICE) ✅ Recommandé
* **Méthode de débayérisation** : algorithme permettant de convertir les données RAW en données multispectrales à 3 canaux
* **Intervalle minimum de recalibrage**: durée minimale en secondes entre deux utilisations d’une cible de calibrage (par défaut :**0** = toutes les cibles, plage 0–3 600)**Produits de secours non calibrés :**Lorsqu&#x27;une image ne peut pas être calibrée en réflectance (aucune cible disponible ou calibrage désactivé), elle est exportée sous la forme de l&#x27;un des deux produits de secours —**un seul des deux existe par série**, choisi par le commutateur de correction de vignettage :

* **Exporter la réponse du capteur**: écrit `Sensor_Response_Images` — utilisé lorsque la correction de vignettage est**désactivée*** **Exporter avec correction de la vignette**: écrit `Vignette_Corrected_Images` — utilisé lorsque la correction de la vignette est**activée**La case à cocher qui n&#x27;est pas sélectionnée est grisée. Décocher celle qui est active empêche complètement l&#x27;écriture de ce fichier.**Produits d’exportation LATTICE** (affichés pour chaque projet ; ils s’appliquent aux captures LATTICE) :

* **Exporter sans débayérisation** : l’image débayérisée linéaire (`Debayered_Images`). S’applique à RGB et aux modules multispectraux.
* **Exporter l&#x27;aperçu** : l&#x27;aperçu à l&#x27;écran (`Preview_Images`). RGB = balance des blancs (illuminant DAQ lorsqu&#x27;il est disponible, sinon « gray-world ») + gamma ; multispectral = étirement en fausses couleurs.
* **Exportation de la radiance** : radiance spectrale de type float32 (`Radiance_Images`, W/m²/sr/nm). Modules multispectraux uniquement — ne s&#x27;applique pas aux masters RGB.
* ****Exporter la réflectance** : réflectance de type uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) lorsqu’une mesure descendante `.daq` ou une cible intégrée à l’image couvre l’image. Modules multispectraux uniquement.

Les quatre options sont **activées par défaut**: une image brute LATTICE importée est répartie vers tous les produits activés et applicables en un seul passage de traitement. La case à cocher**Exporter la réflectance** est grisée lorsque l’étalonnage de la réflectance est désactivé. Les paramètres rendus inactifs par un commutateur parent sont toujours grisés et accompagnés d’une info-bulle indiquant le commutateur à modifier.**Paramètres avancés :*** **Décalage horaire du capteur de lumière** : nombre d’heures par rapport à l’UTC pour la synchronisation horaire du capteur de lumière (par défaut : 0, plage de −12 à +12)
* **Appliquer les corrections PPK** : utilise les données GPS/broches d’exposition issues des fichiers `.daq` (par défaut : désactivé)
* **Broches d’exposition 1/2** : attribue les caméras aux broches d’exposition pour les configurations à deux caméras

{% hint style="info" %}
**Le niveau d’entrée LATTICE est automatique.** Les captures LATTICE comportent leur niveau de traitement dans les métadonnées XMP, et le traitement entre toujours dans le pipeline au niveau de l’image brute — il n’y a rien à configurer dans l’interface graphique. (L&#x27;indicateur CLI `--input-level` existe en tant que paramètre avancé permettant de contourner le comportement par défaut pour les captures dont les métadonnées ont été perdues ; voir la [Référence CLI](../reference/cli-reference.md).)
{% endhint %}

### Méthode de débayérisation

Nous proposons actuellement deux méthodes de débayérisation dans Chloros :

#### Standard (rapide, qualité moyenne)

La méthode de débayérisation standard est rapide, mais génère un bruit de couleur lié au débayérisation, ce qui se traduit par des images moins précises et plus bruitées.

#### « Texture Aware » (lent, qualité optimale) \[Uniquement dans Chloros+]

La méthode « Texture Aware » utilise un débayériseur de haute qualité sensible aux contours, combiné à un modèle de débruitage basé sur l’IA/ML qui élimine la quasi-totalité du bruit de débayérisation. Le modèle nécessite de la mémoire GPU (VRAM) pour fonctionner : avec **7 Go ou plus de VRAM**, il peut traiter plusieurs images simultanément ; en dessous de 7 Go, il traite une image à la fois (ce qui est nettement plus lent). Voir [Adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Les captures LATTICE utilisent toujours le démosaïquage standard.**Il n&#x27;existe pas de modèle « Texture Aware » entraîné pour LATTICE ; cette option n&#x27;est donc pas proposée pour les images LATTICE — les images**Survey3** du même projet peuvent toutefois l&#x27;utiliser.
{% endhint %}

### Indices (indices multispectraux)

Configurez les indices de végétation à calculer et à exporter. Le menu déroulant de l’interface graphique propose **27 formules d’indices prédéfinies**.**Comment ajouter des indices :**

1. Cliquez sur le bouton**« Ajouter un indice »**

2. Sélectionnez un indice dans le menu déroulant (NDVI, NDRE, GNDVI, etc.)
3. Configurez les paramètres de visualisation (couleurs de la table de conversion, plages de valeurs)
4. Ajoutez plusieurs indices selon vos besoins

**Indices courants :*** **NDVI** : état général de la végétation (le plus courant)
* **NDRE** : détection précoce du stress avec RedEdge
* **GNDVI** : sensible à la concentration en chlorophylle
* **OSAVI** : fonctionne bien avec un sol visible
* **EVI** : régions à indice de surface foliaire élevé (LAI)**Formules personnalisées :**

* Créez des formules d’indices multispectraux personnalisées à l’aide d’opérations mathématiques sur toutes les bandes de l’image
* Enregistrez vos formules personnalisées pour les réutiliser
* Les formules personnalisées sont une fonctionnalité de Chloros+ ; leur disponibilité dépend de votre niveau d’abonnement

Pour connaître tous les indices et formules disponibles — y compris ceux qui sont réservés à l&#x27;interface graphique et ceux qui fonctionnent également dans CLI/SDK —, consultez [Formules d&#x27;indices multispectraux](../project-settings/multispectral-index-formulas.md).

### Exportation

Permet de contrôler le format du fichier de sortie.

**Formats disponibles**(paramètre :**Format d’image calibrée**, valeur par défaut**TIFF (16 bits)**) :

* **TIFF (16 bits)** : recommandé pour les SIG et l’analyse scientifique
* **TIFF (32 bits, pourcentage)** : valeurs à virgule flottante
* **PNG (8 bits)** : compression sans perte pour la visualisation
* **JPG (8 bits)** : fichiers les plus légers, compression avec perte

Les fichiers de sortie sont enregistrés dans le dossier du projet, regroupés par caméra et par format : `<project>/<camera>/<format>/<Product>_Images/`. Radiance est **toujours** enregistré au format float32 dans le dossier `tiff32`, quel que soit ce paramètre. Les fichiers exportés conservent le nom du fichier source — le dossier identifie le produit. Voir [Fin du traitement](finishing-the-processing.md) pour l’arborescence complète des fichiers de sortie.

{% hint style="warning" %}
**Lecture des valeurs de réflectance** : la valeur DN correspondant à ρ = 1,0 dépend de la caméra source — LATTICE utilise 32 768 (indiqué dans le fichier XMP `Chloros:PixelScale`), tandis que Survey3 utilise 65 535. Lisez la balise plutôt que de supposer une valeur constante. Voir [Formats d’images de sortie](../output-image-formats.md).
{% endhint %}

### Capteur de lumière DAQ

Cette section répertorie tous les fichiers de rayonnement descendant DAQ (`.daq` / `.csv`) de votre projet, à raison d’une ligne par fichier, en indiquant le modèle de capteur, le nom de fichier et la correction **du cap** du diffuseur en vigueur pour ce fichier.

* **Remplacement de la correction de cap (tous les fichiers)**: un menu déroulant unique valable pour l’ensemble du projet. L’option**Auto** (par défaut) utilise la correction de cap enregistrée dans chaque fichier — en l’absence d’enregistrement, on suppose un ensoleillement maximal, car tous les DAQ MAPIR sont livrés avec le correcteur d’ensoleillement maximal. La sélection d’une valeur de cap remplace celle de tous les fichiers : les enregistrements bruts sont corrigés en fonction de celle-ci, et les enregistrements comportant déjà une valeur de cap sont recalibrés (la correction enregistrée est annulée, la valeur de cap sélectionnée est appliquée).
* Des lignes d’avertissement s’affichent lorsqu’une limite enregistrée correspond à la valeur par défaut supposée du concentrateur plutôt qu’à une valeur confirmée par l’opérateur, et lorsque la limite sélectionnée ne dispose d’aucun profil pour ce modèle d’appareil (la substitution est refusée pour ce fichier).

Les enregistrements DAQ effectués dans l’onglet « Capteurs de lumière » sont automatiquement ajoutés au projet ouvert, et les fichiers `.daq` / `.csv` importés apparaissent ici dès leur ajout.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Paramètres inférieurs du projet — Index, format d’exportation, section DAQ « Capteurs de lumière » et commandes relatives au modèle/dossier du projet</p></figcaption></figure>### Alignement des matrices

Cette section n’apparaît **que** lorsqu’au moins une image du projet comporte la transformation d’alignement de module à module que les matrices LATTICE apposent lors de la capture (`Chloros:Alignment*` XMP). Elle indique le nombre d’images comportant ces balises et quelle caméra sert de référence, à l’aide des commandes suivantes :

* **Appliquer l’alignement du réseau** (par défaut : activé) : déforme chaque produit traité (débayérisation / aperçu / radiance / réflectance / indice) pour l’adapter à la géométrie de référence commune du réseau. Désactivé = exportation dans la géométrie native du capteur.
* **Recadrer sur le chevauchement commun** (par défaut : activé) : recadre les exportations alignées sur la zone commune à tous les modules, afin que chaque bande ait la même empreinte. Désactivé conserve la zone d’image complète du capteur (remplissage noir en dehors de la source).
* **Rééchantillonnage**:**Bilinéaire (lisse, par défaut)**,**Plus proche (conserve les valeurs exactes)**— sans mélange entre pixels, pour une analyse radiométrique rigoureuse — ou**Cubique (plus net)**.***

## Enregistrement et chargement des paramètres

### Enregistrer un modèle de projet

Créez des modèles réutilisables pour des flux de travail cohérents :

1. Configurez tous les paramètres souhaités dans le panneau Paramètres du projet
2. Faites défiler jusqu’à la section **« Enregistrer le modèle de projet »** en bas
3. Saisissez un nom descriptif pour le modèle (par ex. « Survey3N\_RGN\_Agriculture »)
4. Cliquez sur l’icône d’enregistrement

**Avantages :**

* Appliquer des paramètres identiques à plusieurs projets
* Partager les configurations avec les membres de l’équipe
* Assurer la cohérence des enquêtes répétées

### Charger un modèle sur un nouveau projet

Lors de la création d’un nouveau projet :

1. Sélectionnez **« Nouveau projet »** dans le menu principal
2. Choisissez un modèle de projet dans la liste de sélection facultative
3. Tous les paramètres du modèle sont automatiquement appliqués

### Répertoire de travail

Le paramètre **« Répertoire de travail »** spécifie l’emplacement par défaut où les nouveaux projets sont créés :

* **Emplacement par défaut** : `C:\Users\[Username]\Chloros Projects`
* **Modifier l’emplacement** : cliquez sur l’icône d’édition et sélectionnez un nouveau dossier
* **Partagé avec CLI** : `chloros-cli` utilise le même paramètre de dossier de projet par défaut
* **Quand modifier** :
  * Lecteur réseau pour la collaboration en équipe
  * Un autre lecteur offrant davantage d’espace de stockage
  * Une structure de dossiers organisée par année/client

***

## Configuration PPK (cinématique post-traitée)

Si vous utilisez des enregistreurs DAQ MAPIR équipés d’un GPS pour une géolocalisation précise :

### Prérequis

* Enregistreur DAQ MAPIR avec module GPS (GNSS)
* Fichier journal .daq contenant les entrées relatives aux broches d&#x27;exposition
* Caméra connectée aux broches d&#x27;exposition du DAQ pendant la session de capture

### Étapes de configuration

1. Placez le fichier journal .daq dans le dossier de votre projet
2. Dans les paramètres du projet, cochez la case **« Appliquer les corrections PPK »**

3. Définissez le**« Décalage horaire du capteur de lumière »** si nécessaire (par défaut : 0 pour l’UTC)
4. Attribuez les caméras aux broches d&#x27;exposition :
   * **Caméra unique** : attribuée automatiquement à la broche 1
   * **Deux caméras** : attribuez manuellement chaque caméra à la broche correspondante**Attribution des broches d&#x27;exposition :*** **Broche d’exposition 1** : sélectionnez le modèle de caméra dans le menu déroulant
* **Broche d’exposition 2** : sélectionnez une deuxième caméra ou « Ne pas utiliser »
* Une même caméra ne peut pas être attribuée aux deux broches

{% hint style="warning" %}
**Important** : les broches d’exposition doivent être correctement attribuées à leurs caméras respectives. Une attribution incorrecte entraînera des données de géolocalisation erronées.
{% endhint %}

***

## Scénarios avancés

### Projets multi-caméras

Lors du traitement d’images provenant de plusieurs caméras MAPIR dans un même projet :

1. Chloros détecte automatiquement chaque modèle de caméra (Survey3 et LATTICE notamment)
2. Chaque caméra se voit attribuer les profils de traitement appropriés, ainsi que sa propre arborescence de dossiers de sortie
3. PPK : attribuez manuellement à chaque caméra Survey3 la broche d’exposition correcte
4. Toutes les caméras utilisent le même format d’exportation et les mêmes index

**Exemples** : Survey3W RGN + Survey3N OCN (monture à deux caméras), ou un réseau LATTICE combinant un maître RGB avec des modules à bande étroite

### Observations en time-lapse ou sur plusieurs dates

Pour des observations répétées de la même zone au fil du temps :

1. Créez un modèle avec vos paramètres standard
2. Utilisez une configuration de cibles d’étalonnage cohérente à chaque session
3. Traitez chaque date comme un projet distinct
4. Utilisez des paramètres identiques pour obtenir des résultats comparables
5. Exportez dans le même format pour l’analyse temporelle

### Ensembles de données volumineux

Pour les projets comportant un grand nombre d’images (plus de 500) :

* Envisagez de diviser le projet en plusieurs projets plus petits, par date ou par zone
* Utilisez le traitement parallèle Chloros+ pour obtenir des résultats plus rapides
* Envisagez d’utiliser CLI ou API pour l’automatisation par lots
* Ajustez l&#x27;intervalle minimal de recalibrage pour réduire le temps de détection des cibles

***

## Vérification de vos paramètres

Avant de lancer le traitement, vérifiez ces paramètres clés :

* [ ] Modèle de caméra correctement détecté dans l&#x27;explorateur de fichiers
* [ ] Correction de la vignettage activée
* [ ] Calibrage de la réflectance activé
* [ ] Pour Survey3 : au moins une image cible d’étalonnage importée et vérifiée ; pour LATTICE : présence d’une cible et/ou d’un enregistrement descendante `.daq`
* [ ] Indices multispectraux souhaités ajoutés
* [ ] Format d’exportation adapté à votre flux de travail
* [ ] Paramètres PPK configurés (si vous utilisez des fichiers .daq avec des événements d’exposition)

***

## Étapes suivantes

Une fois vos paramètres configurés :

1. **Marquez les images cibles d&#x27;étalonnage** - Voir [Choix des images cibles](choosing-target-images.md)
2. **Lancez le traitement** - Voir [Lancement du traitement](starting-the-processing.md)
3. **Suivez la progression** - Voir [Suivi du traitement](monitoring-the-processing.md)

Pour plus de détails sur tous les paramètres disponibles, consultez la documentation de référence [Paramètres du projet](../project-settings/project-settings.md).
