# Interface graphique : Navigation

Lorsque vous lancez Chloros pour la première fois, le moteur de traitement se met en route. Une fois celui-ci prêt, l&#x27;icône du menu principal en haut à gauche apparaît <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> et les onglets « Caméras » et « Capteurs de lumière » s&#x27;activent dans la barre latérale gauche (ils sont grisés jusqu&#x27;à ce moment-là).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

De gauche à droite, l’en-tête supérieur contient :

### Menu principal d’<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Depuis le menu principal, vous pouvez :

* **Nouveau projet**— créer un nouveau projet. Si vous avez enregistré des modèles de projet, un menu déroulant**Sélectionner un modèle** s’affiche afin que le nouveau projet reprenne les paramètres d’un modèle.
* **Ouvrir un projet**— ouvrir un projet existant. La liste comprend un bouton**Ouvrir le dossier du projet** qui ouvre le dossier des projets dans votre explorateur de fichiers.
* **Dupliquer le projet** — copier le projet actuellement ouvert sous un nouveau nom (un nom libre tel que « MonProjet (2) » est suggéré) et ouvrir la copie. _(visible une fois le projet ouvert)_
* **Ajouter des fichiers** — ajoute des fichiers image individuels au projet en cours _(visible une fois le projet ouvert)_
* **Ajouter un dossier** — ajoute un ou plusieurs dossiers d&#x27;images au projet en cours _(visible une fois le projet ouvert)_
* **Lancer le traitement / Arrêter le traitement** — lance ou arrête la chaîne de traitement des images _(disponible une fois les fichiers ajoutés)_
* **Se connecter à une caméra** — accède à l’[onglet Caméras](lattice/) pour connecter une caméra ou un réseau de caméras LATTICE. Fonctionne sans projet ouvert.
* **Se connecter à un capteur de lumière** — accède à l’[onglet Capteurs de lumière](daq/) pour connecter un capteur de lumière DAQ. Fonctionne sans projet ouvert.

{% hint style="info" %}
**Windows

uniquement** : l’interface graphique de bureau d’Chloros

est disponible surWindows

. Les utilisateurs d’Linux

doivent consulter les documentations [CLI

](CLI.md) et [Python

SDK

](api-python-sdk.md) pour le traitement en mode headless.
{% endhint %}

### Bouton « Play/Start » (Lecture/Démarrer) de<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">



Lorsqu’il est activé, le bouton « Démarrer le traitement » lance le pipeline de traitement des images.

### Barre de progression de<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

<img src=".gitbook/assets/image (5).png" alt="" data-size="line">



En mode gratuitChloros

, qui traite tous les fichiers de manière séquentielle, la barre de progression affiche deux étapes : « Détection des cibles » et « Traitement ».

En mode payantChloros

+ sous licence, qui traite tous les fichiers simultanément, la barre de progression affiche 4 étapes : Détection, Analyse, Calibrage, Exportation. Si vous passez le curseur de votre souris sur la barre de progressionChloros

+, un panneau déroulant comportant les 4 étapes s&#x27;affiche pour vous permettre de suivre l&#x27;avancement. Un clic sur la barre de progression supérieure fige le panneau déroulant ; un nouveau clic le débloque.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu latéral

Le menu de la barre latérale gauche contient diverses icônes interactives, dans l’ordre suivant de haut en bas :

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Paramètres du projet](project-settings/project-settings.md)

L’onglet Paramètres du projet vous permet de régler les paramètres globaux du projet et ceux relatifs au traitement. Réglez-les avant de commencer le traitement de vos fichiers.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Explorateur de fichiers

Ajoutez des fichiers/dossiers et supprimez-en du projet. Les fichiers en double sont ignorés. Cochez la case de la colonne « Cible » pour chaque image cible ; le traitement ne prendra alors en compte que les images cochées comme cibles, ce qui accélère considérablement le temps de traitement. Utilisez le bouton bascule « Image/Métadonnées » pour passer de l’affichage de la grille de vignettes de l’image sélectionnée à un tableau détaillé des métadonnées.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visionneuse d’images](image-viewer-gui/opening-an-image-full-screen.md)

Lorsque vous cliquez sur une image dans la visionneuse principale, celle-ci s’ouvre en plein écran dans l’onglet « Visionneuse d’images ».

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Visionneuse de cartes](image-viewer-gui/map-markers.md)

Visualisez vos images sur une carte 2D interactive en fonction de leurs coordonnées GPS. Prend en charge les fournisseurs de tuiles Google Maps et ESRI, en sélectionnant automatiquement le meilleur service en fonction de votre emplacement. Survolez les marqueurs pour afficher des aperçus sous forme de vignettes.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Caméras](lattice/)

Connectez-vous aux caméras LATTICE et contrôlez-les en direct — une à la fois ou sous forme de réseaux multicaméras synchronisés. Cet onglet affiche des vignettes de prévisualisation en direct avec superpositions et histogrammes, les paramètres par caméra et par réseau, ainsi que les paramètres de capture permettant de choisir les caméras et les types d’exportation générés par la fonction « Capture All ». Disponible dès que le backend est prêt ; consultez la [section LATTICE](lattice/) pour un guide complet.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Capteurs de lumière](daq/)

Connectez des capteurs de lumière DAQ — DAQ-U (USB), DAQ-M (Bluetooth) et DAQ-E (Ethernet) — et affichez leurs graphiques de spectre calibrés en temps réel en W/m²/nm. À partir de cette page, vous pouvez enregistrer des fichiers `.daq` dans le projet ouvert, renommer les capteurs, sélectionner des profils de correction de capacité et mettre à jour le micrologiciel du DAQ-E. Disponible dès que le backend est prêt ; consultez la [section DAQ](daq/) pour un guide complet.

#### Journal de débogage d’<img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

Consultez le journal pour les messages de débogage en cas de problème. Copiez/téléchargez le journal et envoyez-le au [support MAPIR](https://www.mapir.camera/community/contact) pour obtenir de l’aide.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Connexion utilisateur](chloros+-login.md)

La barre latérale de connexion utilisateur vous permet de vous connecter à votre compte Chloros+ pour débloquer des fonctionnalités avancées. Vous pouvez également consulter la version actuelle de l&#x27;application, ainsi que modifier la langue du texte affiché dans l&#x27;interface graphique de Chloros et dans CLI.
