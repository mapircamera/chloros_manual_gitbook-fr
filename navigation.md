# Interface graphique : Navigation

Lorsque vous lancez Chloros et Chloros (navigateur) pour la première fois, le backend démarre. Une fois qu&#x27;il est prêt, l&#x27;icône du menu principal en haut à gauche s&#x27;affiche <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

De gauche à droite, l&#x27;en-tête supérieur contient :

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principal

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Depuis le menu principal, vous pouvez :

* **Nouveau projet** — créer un nouveau projet
* **Ouvrir un projet** — ouvrir un projet existant
* **Ouvrir le dossier du projet** — ouvrir le dossier du projet dans votre explorateur de fichiers
* **Ajouter des fichiers** — ajouter des fichiers image individuels au projet en cours _(visible une fois le projet ouvert)_
* **Ajouter un dossier** — ajouter un dossier d&#x27;images au projet en cours _(visible une fois le projet ouvert)_
* **Démarrer le traitement / Arrêter le traitement** — démarrer ou arrêter le pipeline de traitement des images _(activé une fois les fichiers ajoutés)_

{% hint style="info" %}
**Windows uniquement** : l&#x27;interface graphique de bureau Chloros est disponible sur Windows. Les utilisateurs de Linux doivent consulter les pages [CLI](CLI.md) et [Python SDK](api-python-sdk.md) pour le traitement sans interface graphique.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Bouton Lecture/Démarrer

Lorsqu&#x27;il est activé, le bouton de démarrage du traitement lance le pipeline de traitement d&#x27;images.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barre de progression <img src=".gitbook/assets/image (5).png" alt="" data-size="line">En mode gratuit Chloros, qui traite tous les fichiers de manière séquentielle, la barre de progression affiche 2 étapes : Détection de la cible et Traitement.

En mode payant Chloros+, qui traite tous les fichiers simultanément, la barre de progression affiche 4 étapes : Détection, Analyse, Calibrage, Exportation. Si vous passez le curseur de votre souris sur la barre de progression Chloros+, un panneau déroulant contenant les 4 étapes s&#x27;affichera pour vous permettre de suivre le processus. Cliquer sur la barre de progression supérieure fige le panneau déroulant ; cliquer à nouveau le débloque.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu latéral

Le menu latéral de gauche contient diverses icônes avec lesquelles interagir :

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Paramètres du projet](project-settings/project-settings.md)

L&#x27;onglet Paramètres du projet vous permet de régler les paramètres généraux et de traitement du projet. Modifiez-les avant de commencer à traiter vos fichiers.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navigateur de fichiers

Ajoutez des fichiers/dossiers et supprimez des fichiers du projet. Les fichiers en double sont ignorés. Cochez la case de la colonne « Cible » pour toute image cible, et le traitement ne prendra en compte que les images cochées comme cibles, ce qui accélère considérablement le temps de traitement. Utilisez le bouton bascule « Image/Métadonnées » pour passer de l&#x27;affichage de la grille de vignettes de l&#x27;image sélectionnée à un tableau détaillé des métadonnées.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visionneuse d&#x27;images](image-viewer-gui/opening-an-image-full-screen.md)

Lorsque vous cliquez sur une image dans la visionneuse d&#x27;images principale, celle-ci s&#x27;ouvre en plein écran dans l&#x27;onglet « Visionneuse d&#x27;images ».

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Carte](image-viewer-gui/map-markers.md)

Affichez vos images sur une carte 2D interactive en fonction de leurs coordonnées GPS. Prend en charge les fournisseurs de tuiles Google Maps et ESRI, en sélectionnant automatiquement le meilleur service pour votre emplacement. Survolez les marqueurs pour voir des aperçus des vignettes d&#x27;images.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Journal de débogage

Consultez le journal pour obtenir des informations de débogage en cas de problème. Copiez/téléchargez le journal et envoyez-le au [Support MAPIR](https://www.mapir.camera/community/contact) pour obtenir de l&#x27;aide.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Connexion utilisateur](chloros+-login.md)

La barre latérale de connexion utilisateur vous permet de vous connecter à votre compte Chloros+ pour débloquer des fonctionnalités avancées. Vous pouvez également consulter la version actuelle de l&#x27;application, ainsi que modifier la langue du texte affiché dans l&#x27;interface graphique Chloros et CLI.
