# Interface graphique : Navigation

Lorsque vous lancez Chloros pour la première fois, le moteur de traitement se met en route. Une fois celui-ci prêt, l’icône du menu principal en haut à gauche s’affiche <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> et les onglets « Caméras » et « Capteurs de lumière » deviennent accessibles dans la barre latérale gauche (ils sont grisés jusqu’à ce moment-là).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

De gauche à droite, l'en-tête supérieur contient :

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principal

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Depuis le menu principal, vous pouvez :

* **Nouveau projet**— créer un nouveau projet. Si vous avez enregistré des modèles de projet, un menu déroulant**Sélectionner un modèle** s'affiche afin que le nouveau projet reprenne les paramètres d'un modèle.
* **Ouvrir un projet**— ouvrir un projet existant. La liste comprend un bouton**Ouvrir le dossier du projet** qui ouvre le dossier des projets dans votre explorateur de fichiers.
* **Dupliquer le projet** — copier le projet actuellement ouvert sous un nouveau nom (un nom libre tel que « MonProjet (2) » est suggéré) et ouvrir la copie. _(visible une fois le projet ouvert)_
* **Ajouter des fichiers** — ajoute des fichiers image individuels au projet en cours _(visible une fois le projet ouvert)_
* **Ajouter un dossier** — ajoute un ou plusieurs dossiers d’images au projet en cours _(visible une fois le projet ouvert)_
* **Démarrer le traitement / Arrêter le traitement** — démarre ou arrête le pipeline de traitement d’images _(disponible une fois les fichiers ajoutés)_
* **Se connecter à une caméra** — accède à l’[onglet Caméras](lattice/) pour connecter une caméra ou un réseau de caméras LATTICE. Fonctionne sans projet ouvert.
* **Se connecter à un capteur de lumière** — accède à l'[onglet Capteurs de lumière](daq/) pour connecter un capteur de lumière DAQ. Fonctionne sans projet ouvert.

{% hint style="info" %}
**Windows uniquement** : l’interface graphique Chloros Desktop est disponible sous Windows. Les utilisateurs de Linux doivent consulter la documentation relative à la [CLI](CLI.md) et au [Python SDK](api-python-sdk.md) pour le traitement en mode headless.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Bouton Lecture/Démarrer

Lorsqu'il est activé, le bouton de démarrage du traitement lance le pipeline de traitement d'images.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Barre de progression <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



En mode gratuit Chloros, qui traite tous les fichiers de manière séquentielle, la barre de progression affiche deux étapes : « Détection de cibles » et « Traitement ».

En mode payant Chloros+, sous licence, qui traite tous les fichiers simultanément, la barre de progression affiche quatre étapes : « Détection », « Analyse », « Calibrage » et « Exportation ». Si vous passez le curseur de votre souris sur la barre de progression « Chloros+ », un panneau déroulant contenant les 4 étapes s’affiche pour vous permettre de suivre l’avancement. Un clic sur la barre de progression supérieure fige le panneau déroulant ; un nouveau clic le débloque.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu latéral

Le menu de la barre latérale gauche contient diverses icônes avec lesquelles interagir, dans l’ordre suivant, de haut en bas :

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Paramètres du projet](project-settings/project-settings.md)

L’onglet « Paramètres du projet » vous permet de régler les paramètres globaux du projet et ceux relatifs au traitement. Modifiez ces paramètres avant de lancer le traitement de vos fichiers.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navigateur de fichiers

Ajoutez des fichiers/dossiers et supprimez des fichiers du projet. Les fichiers en double sont ignorés. Cochez la case de la colonne « Cible » pour chaque image cible ; le traitement ne prendra alors en compte que les images cochées comme cibles, ce qui accélère considérablement le temps de traitement. Utilisez le bouton bascule « Image/Métadonnées » pour passer de l’affichage de la grille de vignettes de l’image sélectionnée à un tableau détaillé des métadonnées.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visionneuse d’images](image-viewer-gui/opening-an-image-full-screen.md)

Lorsque vous cliquez sur une image dans la visionneuse d’images principale, celle-ci s’ouvre en plein écran dans l’onglet « Visionneuse d’images ».

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Visionneuse de cartes](image-viewer-gui/map-markers.md)

Affichez vos images sur une carte 2D interactive en fonction de leurs coordonnées GPS. Prend en charge les fournisseurs de tuiles Google Maps et ESRI, en sélectionnant automatiquement le service le mieux adapté à votre emplacement. Survolez les marqueurs pour afficher des aperçus sous forme de vignettes.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Caméras](lattice/)

Connectez-vous aux caméras LATTICE et contrôlez-les en direct — une à la fois ou sous forme de réseaux multicaméras synchronisés. Cet onglet affiche des vignettes de prévisualisation en direct avec superpositions et histogrammes, les paramètres par caméra et par réseau, ainsi que les paramètres de capture qui permettent de choisir les caméras et les types d’exportation générés par l’option « Capture All ». Disponible dès que le backend est prêt ; consultez la [section LATTICE](lattice/) pour un guide complet.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Capteurs de lumière](daq/)

Connectez les capteurs de lumière DAQ — DAQ-U (USB), DAQ-M (Bluetooth) et DAQ-E (Ethernet) — et visualisez en temps réel leurs graphiques de spectre calibrés en W/m²/nm. À partir de cette page, vous pouvez enregistrer des fichiers `.daq` dans le projet ouvert, renommer les capteurs, sélectionner des profils de correction de capacité et mettre à jour le micrologiciel du DAQ-E. Disponible dès que le backend est prêt ; consultez la [section DAQ](daq/) pour le guide complet.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Journal de débogage

Consultez le journal pour vérifier les messages de débogage en cas de problème. Copiez/téléchargez le journal et envoyez-le au [support MAPIR](https://www.mapir.camera/community/contact) pour obtenir de l’aide.

Nouveauté de la version 1.3.0 : l’icône de bug en haut de l’onglet « Journal » (**Signaler un bug**) envoie à MAPIR un rapport contenant votre description, jusqu’à 10 images, les journaux Chloros et les détails du système. Vous devez être connecté pour en envoyer un. Voir [Signaler un bug depuis l’onglet « Journal »](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Connexion utilisateur](chloros+-login.md)

La barre latérale de connexion utilisateur vous permet de vous connecter à votre compte Chloros+ pour débloquer des fonctionnalités avancées. Vous pouvez également consulter la version actuelle de l'application, ainsi que modifier la langue du texte affiché dans l'interface graphique et la CLI de Chloros.
