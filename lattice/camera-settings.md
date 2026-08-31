# Paramètres des caméras

L&#x27;onglet **Caméras** constitue la surface de contrôle en direct d&#x27;Chloros

pour les caméras LATTICE : une zone d&#x27;affichage principale présentant chaque caméra connectée sous forme de vignette en direct, et une barre latérale permettant de basculer entre trois pages — la **liste des caméras**, un**volet de paramètres**(paramètres par caméra, par réseau ou de capture — un à la fois) et le**calculateur d’index**. Cette page documente chaque commande de la liste des caméras, du volet des paramètres par caméra et du volet des paramètres de réseau. Les modes de capture, la sélection du type d’exportation et le flux « Capture All » sont décrits sur la page associée [Paramètres et modes de capture](capture.md).

L’onglet « Caméras » apparaît dans la barre latérale dès que le backend «Chloros

» est prêt. Tous les éléments de contrôle ci-dessous communiquent avec le backend local via `127.0.0.1:5000` ; les modifications s’appliquent immédiatement à la caméra en direct, sauf indication contraire.

## Types de caméras utilisés sur cette page

Les commandes s’affichent ou se masquent en fonction du type de caméra sélectionné. Le manuel utilise les termes suivants tout au long du document :

| Terme | Signification | Canaux du filtre |
| --- | --- | --- |
| **CaméraRGB** | LATTICE M3C avec filtre FRGB (modèle contenant `-FRGB`) |Red

/Green

/Blue

|
| **Multispectrale Bayer** | LATTICE M3C avec FRGN, FOCN ou FNGB | FRGN :Red

/Green

/NIR

· FOCN :Orange

/Cyan

/NIR

· FNGB :NIR

/Green

/Blue

|
| **Mono (M3M)** | LATTICE M3M — un filtre à bande étroite, une bande calibrée | Bande unique |
| **Élément de réseau** | Une caméra connectée au sein d’un réseau synchronisé (affichage combiné ou séparé) | Selon son filtre |

Les camérasRGB

s bénéficient d’un traitement photométrique (balance des blancs, profils de couleur, gamma) ; les caméras multispectrales et mono bénéficient de la chaîne radiométrique et ne passent pas par les contrôles photométriques. Les éléments du réseau transmettent les paramètres au niveau du flux (format de pixel, résolution, binning, déclenchement, fréquence d’images) au réseau — ces lignes deviennent en lecture seule dans le volet dédié à chaque caméra et sont déplacées vers le volet des paramètres du réseau.

## La zone principale de flux



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

En l’absence de caméras connectées, la zone de flux affiche un écran de bienvenue **« Connectez une caméra pour commencer »**avec deux boutons :**Connecter une caméra**(vert, ouvre la boîte de dialogue de connexion d’une seule caméra) et**Connecter un réseau** (bleu, ouvre la boîte de dialogue de connexion d’un réseau). Les boîtes de dialogue de connexion sont décrites dans [Connexion des caméras](connecting.md) ; les concepts relatifs aux réseaux de caméras (synchronisation, niveaux, bande passante) sont expliqués dans [Réseaux de caméras multiples](arrays.md). Lorsque vous ouvrez un projet enregistré contenant des caméras, l’écran de démarrage affiche à la place une icône tournante avec le message « Réouverture de N caméras enregistrées… » pendant qu’Chloros

e restaure les flux de la dernière session.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barre supérieure

| Commande | Fonction |
| --- | --- |
| **Bouton de basculement du mode d’affichage**| Permet de basculer entre le**mode grille**(toutes les vignettes sous forme de cellules) et le**mode liste** (matrices en pleine largeur en haut, UNE caméra active en dessous). Infobulles : « Passer en mode grille » / « Passer en mode liste ». |
| **Verrouillage de la grille**(cadenas) |**Verrouillé** par défaut — les tuiles sont figées à leur emplacement. Déverrouillez pour réorganiser les tuiles par glisser-déposer dans n&#x27;importe quel emplacement (les espaces sont conservés). La grille se verrouille automatiquement à chaque fois qu’une nouvelle caméra se connecte. Infobulles : « Déverrouiller la grille (activer le glissement des vignettes) » / « Verrouiller la grille (figer les vignettes en place) ». |
| Curseur **Zoom du flux** | Taille des vignettes, de 60 px jusqu’à la largeur totale du conteneur. Les tuiles conservent un format 4:3. En dessous de 200 px de largeur, les superpositions du nom et de l’images par seconde (ips) se masquent pour garder la tuile épurée. |

### Tuiles du flux

Chaque caméra génère une tuile composite en direct ; une caméra peut en outre afficher trois **tuiles séparées par canal** en niveaux de gris (voir [Séparations par canal](#display-overlays-drawn-over-the-live-feed)), et les réseaux de caméras affichent une tuile combinée. La tuile active est signalée par un anneau de sélection de la couleur de la caméra (ou du réseau).

En survolant une tuile, un bouton de fermeture **X** apparaît :

* Fermer une vignette **composite** alors que ses divisions de canaux restent affichées masque simplement la vignette composite.
* Fermer la **dernière vignette visible d’une caméra autonome** déconnecte cette caméra.
* **Les vignettes de division des éléments d’un réseau combiné ne déconnectent jamais** la caméra — elles la masquent simplement.

Lorsque la grille est déverrouillée, faites glisser n’importe quelle tuile vers n’importe quel emplacement ; la disposition est enregistrée avec le projet.

## Barre latérale — liste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

des caméras La première page de la barre latérale répertorie toutes les caméras et tous les réseaux connectés :

* **Connecter une caméra**(vert) /**Connecter un réseau** (bleu, affiche « Détection… » pendant l’analyse). Les deux boutons sont désactivés lorsqu’une boîte de dialogue de connexion est ouverte.
* **Tout capturer**(rouge) — capture toutes les caméras répertoriées avec les types d’exportation choisis dans les**Paramètres de capture**. Nécessite un projet ouvert. Entièrement documenté dans [Paramètres et modes de capture](capture.md).
* **Icône en forme de roue dentée des paramètres de capture** (à côté de « Capture tout ») — ouvre le [volet Paramètres de capture](capture.md#the-capture-settings-pane). Désactivé en l’absence de projet ou pendant une capture.

### Lignes de caméras

Chaque ligne de caméra affiche une bordure avec un code couleur (la couleur personnalisée de la caméra), une étiquette « CAM » — accompagnée d’une lettre de rôle bleue **M**(maître) ou verte**S** (esclave) pour les membres du réseau — et le nom d’affichage. Le nom par défaut est `LATTICE-MODEL (serial)` ; vous pouvez le renommer depuis le volet des paramètres propres à chaque caméra. Boutons de la ligne :

| Bouton | Effet |
| --- | --- |
| **Œil**| Active/désactive la visibilité. Les caméras masquées disparaissent de la grille et sont**exclues de la fonction « Capture All »**. |
| **Roue dentée** | Ouvre le volet des paramètres propres à chaque caméra (section suivante). |
| **Pause / Lecture**| Fige l’aperçu en direct**côté affichage uniquement** — la capture en arrière-plan continue de s’exécuter. Les caméras en pause ne peuvent pas capturer. |
| **X** | Déconnecter. L’interface utilisateur se met à jour immédiatement (dans le meilleur des cas) ; la déconnexion en arrière-plan peut prendre entre 10 et 30 s. |

### Lignes du tableau

Une ligne du tableau affiche un badge « ARRAY » dans la couleur du tableau, le nom du tableau (renommable dans les paramètres du tableau) et une **pille DAQ · activé/désactivé**—**activé** lorsque le capteur de lumière au niveau du tableau est activé *ou* qu’un membre dispose d’un capteur par caméra ; son info-bulle indique précisément quel capteur alimente quoi. Les caméras membres sont répertoriées en retrait en dessous, avec leurs propres lignes. Boutons de la ligne de réseau : **œil**(masque/affiche TOUS les membres ensemble),**roue dentée**(volet des paramètres du réseau),**X**(déconnecte l&#x27;ensemble du réseau).

L’état du capteur de lumière (DLS) utilisé dans les lignes du réseau et dans le volet des paramètres du réseau comporte quatre états :**désactivé**,**en attente**(pas encore de spectre),**actif**(un spectre est arrivé au cours des 3 dernières secondes) et**périmé** — aucun spectre récent depuis 3 secondes, mais la dernière lecture est *toujours utilisée* (les relevés DAQ n’expirent jamais sur le chemin de capture).

Vous pouvez faire glisser des caméras autonomes et des groupes de réseaux entiers les uns sur les autres dans la barre latérale pour réorganiser la liste ; les éléments d’un réseau ne peuvent pas être déplacés indépendamment.

## Volet des paramètres par caméra

Ouvrez-le à l’aide de l’**icône en forme d’engrenage** sur une ligne de caméra. Le volet s’affiche par-dessus la liste des caméras.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**En-tête**: l’**échantillon de couleur**de la caméra (cliquez pour ouvrir un sélecteur de couleur natif — permet de définir la couleur de la bordure de la barre latérale et de l’anneau de sélection des vignettes), le**nom**avec un crayon et le bouton**Renommer**(si vous enregistrez un nom vide, la caméra reprend le nom par défaut `MODEL (serial)`), et**×** pour fermer.

### Histogramme en temps réel

En haut du volet se trouve un histogramme de luminance en temps réel calculé à partir de l’aperçu de l’JPEG

à environ 8 Hz. La moyenne est pondérée selon le principe de Bayer — (R+2G+B)/4 — afin de correspondre à la mesure AE propre à la caméra.

* **Orange

ligne pointillée**= la cible de l’exposition automatique.**Faites-la glisser horizontalement pour redéfinir la cible** — une commande est envoyée lorsque vous relâchez le glissement, et le glissement fait basculer le mode de cible d’exposition automatique en mode manuel.
* **Ligne continueGreen** = la luminance moyenne réelle (ce que l’exposition automatique fournit actuellement).
* **BoutonRGB** (en haut à droite) : active ou désactive les histogrammes superposés par bande, colorés selon le filtre de l’appareil photo (par exemple, sur FRGN : grisNIR

e, vert, rouge). Sur les caméras mono (M3M), le bouton porte la mention « MONO » et est désactivé — le mode mono affiche toujours l’histogramme de luminance à bande unique.
* Les étiquettes de l’axe X suivent la profondeur de bits du capteur du format de pixel actuel : 0..255, 0..1023, 0..4095 ou 0..65535.

### Lignes

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

d’informations sur la caméra

| Ligne | Comportement |
| --- | --- |
| **Modèle** | En lecture seule (par ex. `LATT-M3C-L87-FRGN`). |
| **Étalonnage radiométrique** | Badge «Green

» **« Active »**accompagné d’une légende indiquant le niveau d’étalonnage, le hachage, la date d’étalonnage et la liste des bandes, chargés à partir du pack d’étalonnage de la caméra (voir [Étalonnage radiométrique d’usine](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Masqué pour les caméras d&#x27;RGB

** — celles-ci disposent d&#x27;un étalonnage photométrique de la balance des blancs, et non d&#x27;un étalonnage de la radiance par bande. |
| **Rapport d’étalonnage**| Bouton**Télécharger** — ouvre le certificat d’étalonnage NIST propre à chaque numéro de série de la caméra au format PDF dans la visionneuse de votre système d’exploitation. Si le certificat n’est pas encore mis en cache, l’Chloros

affiche un message d’aide à la place. |
| **Numéro de série** | En lecture seule. |
| **Micrologiciel**| Affiche la version actuelle, puis recherche la version disponible pour ce modèle (mise en cache par modèle — un ensemble de N caméras effectue une seule vérification sur le serveur). États : « Vérification en cours… » → Bouton**« Mettre à jour vers X »**→ « Flashage en cours… » → « Mis à jour de A vers B » / « Échec : … » / « Ignoré : … » /**« À jour »** en vert. Info-bulle du bouton de mise à jour : « Réinitialisation d’usine + flashage + reprogrammation de UserSet1. ~2 à 3 minutes ; ne pas déconnecter. » |
| **IP** | En lecture seule. |
| **Température** | En lecture seule, actualisée toutes les 3 s. Passe à l’orange à partir de 65 °C et au rouge avec un ⚠ à partir de 75 °C. |
| Case à cocher **Cible d’étalonnage** | Active la détection de la cible de réflectance ArUco à l’aide d’une table de validation «NDVI

» par panneau située sous le flux en direct (vue en liste). Valable uniquement pour la session — toujours désactivée par défaut. |
| Menu déroulant **Capteur de lumière** | Associe un capteur de lumière DAQ (DAQ-E/M/U, à partir de la liste de l’onglet « Capteurs de lumière ») à cette caméra pour la correction de l’éclairage par lumière descendante (DLS) et l’exposition automatique prédictive. « Aucun » annule l’association. Si aucun capteur n’est connecté, le menu déroulant affiche « (aucun capteur connecté — ouvrez l’onglet DAQ) ». L’association est enregistrée avec le projet. |

### Exposition et gain

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Toutes les entrées numériques ici utilisent des curseurs à maintien pour accélérer : appuyer = ±1, maintenir &gt;1,5 s = ±10, maintenir &gt;3 s = ±100. La valeur est envoyée à la caméra lorsque vous relâchez le curseur.

| Commande | Plage / choix | Par défaut | S&#x27;applique à | Fonction |
| --- | --- | --- | --- | --- |
| **Exposition (us)**| Min./max. en temps réel de la caméra | Auto | Tout | Temps d’exposition en microsecondes, avec un bouton**Auto/Manuel**. Auto = exposition automatique continue côté caméra. |
| **Gain (dB)**| Plage minimale/maximale en temps réel de la caméra (par ex. jusqu’à 48 dB) | Manuel (désactivé) | Tous | Gain analogique/numérique avec son propre commutateur**Auto/Manuel**. |
| **Luminosité cible AE**| 0–255 | 80, mode**Auto**| Tout (modifiable lorsque l&#x27;AE ou le gain automatique est activé) | La luminosité visée par l&#x27;AE. En mode**Auto**(par défaut), un contrôleur interne basé sur l&#x27;histogramme sélectionne lui-même la valeur cible, en maintenant l&#x27;exposition entre 60 et 75 % du maximum du capteur. La saisie d’une valeur ou le glissement de la ligne orange de l’histogramme bascule le paramètre en mode**Manuel**. |
| **Lissage AE** | 0,5–40, par pas de 0,1 | 8,0 | Tout | Amortissement de l’AE. Info-bulle : « Une valeur plus faible = l’exposition automatique réagit plus rapidement (peut produire des saccades à des fréquences d’images élevées). Une valeur plus élevée = plus fluide / plus lent. » Des valeurs bien inférieures à la valeur par défaut peuvent provoquer des saccades de l’exposition automatique et déstabiliser la diffusion en continu à des fréquences d’images élevées ; 8,0 est la valeur par défaut stable. |
| **Zone d’intérêt AE**| Case à cocher « Activer » + bouton**Viser**| Désactivé | Tout | Lorsqu’elle est activée, l’AE mesure uniquement la zone verte en pointillés au lieu de l’image entière.****Aim**active le placement par clic sur le flux en direct : un clic centre une zone à 30 % de l’image ; cliquer-glisser trace un rectangle personnalisé (minimum 5 % × 5 %). La fonction**Aim** se désactive automatiquement après un placement. La zone est remappée aux, en tenant compte de toute rotation ou symétrie que vous avez définie, et est enregistrée avec le projet. |
| **Vitesse de réglage de l’AE** | 0,1–5, par pas de 0,1 | 1,0 | Réservé aux membres du tableau | Vitesse à laquelle la cible AE automatique suit les changements de luminosité de la scène ; 1,0× effectue une nouvelle vérification toutes les 2,5 s. |
| **Protection des hautes lumières** | Stricte (1 %) / Normale (5 %) / Souplesse (15 %) | Stricte | Caméras permettant de régler ce paramètre | Pourcentage de l’image pouvant être écrêté en blanc avant que l’exposition automatique n’assombrisse l’image. |

{% hint style="info" %}
**Exigences en matière d’éclairage pour les caméras multispectrales Bayer (RGN

/OCN

/NGB

) :** la scène doit être suffisamment éclairée sur les trois canaux, sinon l’étalonnage ne fonctionne pas correctement — une seule exposition du capteur couvre les trois spectres. Utilisez un capteur de lumière DAQ pour mesurer votre éclairage, ou optez pour le mode entièrement mono (M3M) afin que chaque bande bénéficie de sa propre exposition. Si une capture ne respecte pas cette consigne,Chloros

le détecte et vous en avertit (notification « unmix-clamp »).
{% endhint %}

### Format et résolution des pixels

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Les éléments de la matrice** affichent des lignes en lecture seule « Current » (format + LxH) et « Binning » accompagnées de la remarque « Définir dans les paramètres de la matrice » — un redémarrage du flux sur un élément romprait la synchronisation ; ces paramètres sont donc gérés dans le [volet des paramètres de la matrice](#array-settings-pane).**Les caméras autonomes** disposent des options suivantes :

| Commande | Choix | Fonction |
| --- | --- | --- |
| **Format des pixels** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Format des pixels du capteur (profondeur de bits). |
| **Résolution** | Pleine / Moitié / Quart | Par rapport au regroupement actuel : Pleine = 2048/N × 1536/N pour un regroupement N×N. |
| **Regroupement** | 1x1 (aucun) / 2x2 / 4x4 | Regroupement matériel N×N — des valeurs plus élevées réduisent la résolution mais améliorent le rapport signal/bruit et la fréquence d’images. Sa modification redémarre le flux et réinitialise toute zone d’intérêt (ROI) au nouveau champ de vision complet. |
| **Actuel** | en lecture seule | Les dimensions réelles L × H et le décalage (x, y) actuellement appliqués. |

### Aperçu en direct

Tout ce qui se trouve dans cette section concerne **uniquement l’affichage**— cela modifie ce que vous voyez dans le flux en direct, tandis que les captures enregistrées restent linéaires et inchangées — à une exception près : la**vignette** est radiométrique et affecte également les exportations (comme indiqué ci-dessous).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Commande | Plage / choix | Par défaut | S’applique à | Fonction |
| --- | --- | --- | --- | --- |
| **Résolution de rendu** | 360p (la plus rapide) / 480p / 720p / 1080p / Résolution native du capteur (la plus lente) | 720p | Tout | La résolution à laquelle le backend exécute la chaîne de prévisualisation radiométrique. Une valeur plus faible permet d’augmenter la fréquence d’images sans modifier le champ de vision. |
| **Indice**| Case à cocher « Activer » + roue dentée | Désactivé | Multispectral Bayer uniquement,**pas** les membres à matrice combinée | Aperçu en direct de l’indice de végétation. La roue dentée ouvre le [Calculateur d’indice](#index-calculator-pane) préchargé avec les bandes naturelles du filtre de la caméra (par ex. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). L’expression personnalisée associée à la table de conversion (LUT) (activée/désactivée, niveau par défaut 3, min par défaut 0,2, max par défaut 1) est calculée à chaque image de prévisualisation. Lesles éléments du tableau masquent cette ligne — le tableau possède un index partagé. |
| **Balance des blancs** | Désactivée / Une fois / Continue + un bouton de nouvelle capture | Continue |RGB

uniquement | Balance des blancs en temps réel. Le bouton de rafraîchissement effectue une nouvelle capture de la balance des blancs à partir du spectre DLS actuel (désactivé lorsque le mode est désactivé). |
| **Gamma** | Activé / Désactivé | Activé | Uniquement en mode «RGB

» | Affiche le gamma (LUT γ = 2,2) sur la prévisualisation en direct. Les captures enregistrées restent linéaires. |
| **Réduction du bruit** | Case à cocher + intensité 0–100 | Désactivé / 50 | Toutes les (par caméra, même au sein des matrices) | Filtre bilatéral sur l’aperçu en direct. Plus la valeur est élevée, plus le résultat est lisse, mais les détails sont moins nets. |
| **Netteté** | Case à cocher + intensité 0–100 | Désactivé / 30 | Tout | Masque de netteté sur l’aperçu en direct, appliqué en dernier. Peut amplifier le bruit. Aperçu uniquement. |
| **Vignettage**| Case à cocher + intensité 0–100 | Désactivé / 0 | Tout | Suppression manuelle du vignettage résiduel (éclaircit les coins), superposée à l&#x27;estimation du vignettage intelligent du réseau.**Radiométrique — affecte l’aperçu en temps réel ET les exportations**, contrairement aux options « Dénoyage » et « Netteté ». |
| **Profil de couleur** | Raw / Linéaire / Naturel / Amélioré / Température personnalisée | Naturel |RGB

uniquement | Voir ci-dessous. |
| **Température de couleur** | 2 000–10 000 K, par pas de 100 | 5 500 K |RGB

, profil de température personnalisée uniquement | Fixe la balance des blancs à une température de couleur corrélée fixe (entrée DLS ignorée). La dernière valeur en Kelvin sélectionnée est mémorisée lors des changements de profil. |
| **Saturation** | 0–200 (100 = neutre) | 100 |RGB

uniquement | Saturation HSV sur l’aperçu en direct. |
| **Contraste** | 0–200 (100 = neutre) | 100 |RGB

uniquement | Contraste linéaire autour du gris moyen dans l’aperçu en direct. |
| **Miroir H / Miroir V** | Cases à cocher | Désactivé | Tous | Retourne l’aperçu horizontalement / verticalement. |
| **Rotation**| 0° / 90° / 180° / 270° | 0° | Toutes | Faire pivoter l’aperçu. L’orientation est appliquée à la fin de la chaîne d’aperçu du backend —**les captures enregistrées conservent l’orientation native de l’appareil photo**, et les vues composites de type tableau l’ignorent. |**Sémantique des profils de couleur** (appareils photoRGB

) :

* **Raw** — contourne entièrement la chaîne de traitement.
* **Linéaire** — signal sombre + champ plat + balance des blancs ; pas de matrice de couleur, pas de gamma.
* **Naturel** *(par défaut)* — linéaire plus la matrice de correction des couleurs mesurée et une courbe de tonalité adaptative à la scène.
* **Amélioré**— Naturel plus la vibrance et le contraste local CLAHE. Le surcoût s’applique**uniquement à l’aperçu en direct** — les captures enregistrées bénéficient toujours du traitement complet, quel que soit le profil.
* **Température personnalisée** — « Naturel » avec une balance des blancs fixée à la valeur en Kelvin de votre choix.

{% hint style="warning" %}
Pour les modes Naturel, Amélioré et Température personnalisée, le volet affiche une remarque sur la tonalité : les images sont éclaircies en fonction de leur propre scène ; par conséquent, les images *d&#x27;affichage* enregistrées ne sont pas comparables d&#x27;une image à l&#x27;autre. **Exportez la radiance ou la réflectance pour les mesures.**
{% endhint %}

### Superpositions d’affichage (dessinées par-dessus le flux en direct)

Ces superpositions sont propres à l’interface utilisateur : elles sont superposées à la vidéo, sans jamais modifier le flux ni les captures.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Superposition | Commandes | Par défaut | Fonction |
| --- | --- | --- | --- |
| **Zébré** | Case à cocher + seuil 200–255 | Désactivé / 250 | Rayures diagonales magenta sur les pixels écrêtés. |
| **Réticule** | Case à cocher | Désactivé | Repère au centre de l&#x27;image. |
| **Grille** | Désactivée / 3 × 3 / 9 × 9 | Désactivée | Grille de composition. |
| **Histogramme** | Case à cocher + largeur comprise entre 0,10 et 0,90 de l’image | Désactivé / 0,25 | Une bande d’histogramme sur le flux. |
| **Focus Peak** | Case à cocher + seuil 20–200 + échantillon de couleur | Désactivé / 80 / `#ff5722` | Mise en évidence des contours par l’algorithme de Sobel pour la mise au point. |
| **Séparation des canaux** | Boutons « Afficher les séparations (Red

/Green

/NIR

) » / « Masquer les séparations » | Masqué | Ajoute trois vignettes indépendantes en niveaux de gris par canal à côté de l’image composite (le libellé du bouton suit les canaux de filtre de la caméra). Chaque vignette de séparation est déplaçable et reprend la couleur de bordure de la caméra. Non disponible sur les caméras monochromes. Enregistré avec le projet. |

### Posemètre ponctuel

* Case à cocher **Cliquer pour échantillonner**: cliquez sur l’image en direct pour échantillonner un seul pixel (un réticule en croix le marque), ou cliquez et faites glisser pour sélectionner une zone et obtenir une moyenne des pixels.**Effacer**supprime l’échantillon et le réticule. Fonction incompatible avec le mode**Viser** de la zone d’intérêt AE (AE-ROI).
* Menu déroulant **Afficher**:**Brut (profondeur de bits)**— valeurs numériques natives à la profondeur de bits du capteur (par ex. 12 bits → 0..4095) — ou**Affichage (8 bits)** (par défaut). Lorsqu’un indice en temps réel est actif, l’option Affichage présente à la place la valeur d’indice calculée (par ex.NDVI

).
* Le panneau d’affichage répertorie les coordonnées des pixels, la taille de l’image, le format des pixels, la profondeur de bits et un tableau des canaux (Chan / Valeur / %) avec les libellés des bandes et les longueurs d’onde ; les paires vertes de Bayer sont moyennées ; les échantillons de région affichent « N px avg ».

L’état du posemètre ponctuel est valable uniquement pour la session en cours.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Exposition automatique prédictive (pilotée par DLS)

Cette section n’apparaît que lorsqu’**au moins un capteur de lumière DAQ est connecté** — le solveur a besoin d’un spectre descendant en temps réel pour fonctionner.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Commande | Plage | Par défaut | Fonction |
| --- | --- | --- | --- |
| **Activer** | Case à cocher | Activé (caméras autonomes) | Un solveur de forme fermée utilise le spectre DLS ainsi que les scalaires du pack d’étalonnage de la caméra pour amener la bande la plus lumineuse près de la saturation tout en maintenant la bande la plus sombre au-dessus du seuil de rapport signal/bruit (SNR) — une seule écriture d’exposition par résolution, sans boucle de stabilisation. Conçu pour les time-lapses alimentés par énergie solaire où chaque prise de vue doit être correctement exposée. Le backend bascule silencieusement vers l’exposition automatique réactive dès que la lecture DLS est obsolète/manquante ou que le pack d’étalonnage n’est pas chargé. |
| **Lissage (α)** | 0,05–1,0, pas de 0,05 | 0,3 | Lissage des solutions prédictives successives (valeur plus faible = lissage plus important). |
| **Réflectance de la scène**| Bouton**Recalibrer ρ** | — | Réévalue le facteur de réflectance de la scène utilisé par le solveur. |

{% hint style="info" %}
**La connexion en réseau désactive l’exposition automatique prédictive par défaut** — pour les réseaux, l’exposition automatique intelligente d’Chloros

ainsi que l’exposition automatique côté caméra gèrent l’exposition (avec protection contre la saturation) et l’estimation de la réflectance d’une seule scènen’est pas fiable dans des scènes mixtes. Vous pouvez la réactiver ici pour chaque caméra si vous souhaitez spécifiquement une exposition radiométrique pilotée par le DLS.
{% endhint %}

**Plafond d’exposition piloté par le DAQ et exposition automatique (AE) verrouillée sur l’incidence.** Indépendamment de la case à cocher ci-dessus, lorsqu’un capteur de lumière DAQ est attribué à une caméraRGB

,Chloros

calcule — à partir de l’irradiance descendante absolue mesurée — l’exposition × gain maximale à laquelle une surface de réflectance de 100 % reste en dessous du seuil de clipping, et l’applique comme **plafond**à l’exposition automatique. Tant que la limite est active, la caméra est en mode**« incident-pinned »** : elle fonctionne en boucle ouverte à l’exposition mesurée sur la lumière incidente avec un gain à 0 dB — l’exposition suit la lumière mesurée, et non le contenu de la scène. Comme le plafond ne peut que raccourcir l’exposition, il ne peut pas en soi provoquer de saturation. Le plafond se désactive automatiquement — et l’exposition automatique normale reprend — dès que la lecture DAQ est manquante, périmée (&gt; 30 s), ou sombre, ou si ≥ 15 % de l’image est saturée à l’exposition verrouillée (ce qui signifie que le capteur et la caméra perçoivent un éclairage différent). Il n’y a pas de commutateur dans l’interface graphique ; il s’agit d’un comportement standard dès lors qu’une caméraRGB

est associée à un DAQ.

### Acquisition et déclenchement

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Les éléments du réseau affichent en outre des lignes en lecture seule : **Rôle**(Maître en bleu / Esclave en vert),**Ligne de synchronisation**et**Pairs**.

| Contrôle | Choix | Par défaut | Remarques |
| --- | --- | --- | --- |
| **Mode de déclenchement** | Désactivé / Activé | Activé | Désactivé pour les membres du réseau (le réseau gère le déclenchement). |
| **Source de déclenchement** | Logiciel / Ligne 0 (M8) / Ligne 1 / Ligne 2 | Ligne 0 | Masquée lorsque le mode de déclenchement est désactivé ; désactivée pour les membres du réseau. La ligne 0 correspond à l&#x27;entrée de déclenchement externe opto-isolée M8. |
| **Fréquence d’images**| Auto / Manuel + valeur | Auto |**Auto**: la limite de fréquence d’images de la caméra est désactivée — l’exposition détermine le nombre d’images par seconde (fps), et la fenêtre affiche la fréquence réelle en direct.**Manuel** : vous limitez le nombre d’images par seconde à l’aide d’un curseur (de 1 jusqu’au maximum autorisé par la bande passante), en vous basant sur la fréquence réelle actuelle. Les membres du réseau voient s’afficher une valeur en lecture seule « N images par seconde (en direct) » avec la mention « Définir dans les paramètres du réseau ». |

### Réseau / Transport

| Ligne | Comportement |
| --- | --- |
| **Taille de paquet**| 1 500 (Standard) / 9 000 (Jumbo) —**Jumbo** par défaut. |
| **Débit** | Limite de débit de la liaison en Mo/s, en lecture seule. Le backend rééquilibre cette valeur entre toutes les caméras connectées à chaque connexion/déconnexion. |
| **Gestion du tampon** | Mode de gestion du tampon, en lecture seule. |

### Capture

Le volet se termine par un bouton **« Ouvrir les paramètres de capture… »** qui permet d’accéder au [volet Paramètres de capture](capture.md#the-capture-settings-pane) (désactivé tant qu’aucun projet n’est ouvert — « Créez ou ouvrez un projet pour enregistrer les captures »). Si la caméra est masquée ou en pause, une remarque vous invite à la démasquer ou à la réactiver avant de procéder à la capture.

## Volet des paramètres de matrice

Ouvrez-le à l’aide de l’**icône en forme d’engrenage**située sur une ligne de la MATRICE. En-tête : nom du réseau avec un crayon pour le renommer et un**×** pour fermer. Les sections ci-dessous marquées *« combiné uniquement »* n’apparaissent que pour les réseaux connectés en mode d’affichage combiné.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synchronisation

Lignes **Maître**,**Esclaves**et**Ligne de synchronisation** en lecture seule.

### Capteur de lumière ambiante

Affiché pour les réseaux combinés et séparés :

* Case à cocher **Cible d’étalonnage** — « Détecter la cible ArUco d’MAPIR

et valider laNDVI

par rapport à la table de conversion (LUT) de réflectance du panneau » ; contrôle la superposition de la cible et la table de validation de la tuile combinée.
* Menu déroulant **Capteur de lumière** — associe un DAQ à l’ensemble du réseau. La sélection prend effet immédiatement, se répercute sur le menu déroulant « Capteur de lumière » de chaque caméra du réseau (vous pouvez toujours la remplacer au cas par cas) et commence à transmettre les spectres au réseau.
* Ligne **Statut** en temps réel : Désactivé · « En attente du premier spectre… » · « Actif — toutes les caméras du réseau bénéficient d’une correction d’éclairage » · « Aucun nouveau spectre au cours des 3 dernières secondes — utilisation de la dernière lecture (pas de délai d’expiration)… ».
* Remarque dans le volet : « Correction radiométrique à l’échelle du réseau. Les paramètres propres à chaque caméra prévalent sur celle-ci. »

### Capture — paramètres uniformes du capteur *(combinés uniquement)*

Ces paramètres s’appliquent de manière uniforme à tous les membres (des modifications au cas par cas rompraient la synchronisation). Les modifications sont mises en attente puis appliquées simultanément.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Commande | Choix / plage | Fonction |
| --- | --- | --- |
| **Format des pixels** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Format de capteur uniforme pour tous les éléments. |
| **Regroupement de pixels** | 1x1 / 2x2 / 4x4 | Regroupement matériel — conserve le champ de vision complet tout en améliorant le rapport signal/bruit et la fréquence d&#x27;images. Sa modification réinitialise les champs de la région d’intérêt (ROI) au nouveau champ de vision complet. |
| **Préréglage de résolution** | Pleine / Moitié / Quart | Relatif au regroupement de pixels ; remplit les champs ROI avec un recadrage centré. |
| **Recadrage de la zone d&#x27;intérêt (px)**| Champs numériques L / H / X / Y | Recadrage du capteur. La largeur et la hauteur s&#x27;alignent sur des multiples de 16 (minimum 64) ; les décalages s&#x27;alignent sur des multiples de 4. Une indication « LxH max. » indique la limite supérieure et la commande**Réinitialiser** rétablit le champ de vision complet. Pendant l’édition, un cadre de prévisualisation orange en temps réel s’affiche sur la tuile de la matrice (y compris un schéma du capteur complet lorsque le recadrage s’étend vers l’extérieur). |
| **Fréquence de déclenchement**| Bouton bascule Auto / Manuel + fps 0,5–10, pas de 0,5 |**Auto**(par défaut) : le backend calcule la fréquence de déclenchement à partir de la résolution et de la bande passante — le champ de saisie est désactivé et affiche la valeur calculée.**Manuel** : verrouille votre valeur lorsque vous cliquez sur « Appliquer ». |

Remarque dans le volet : « Les changements de format/résolution redémarrent brièvement toutes les caméras. La fréquence de déclenchement s’applique en temps réel. » Les boutons **Appliquer / Annuler** se trouvent en bas du volet.

### Alignement (co-enregistrement) *(combiné uniquement)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Champ **Résiduel** : « RMS x,xx px » — vert en dessous de 1 px, orange en dessous de 3 px, rouge dans le cas contraire ou si une caméra a échoué ; « aucun profil » avant la première résolution.
* Ligne de résumé : « ✓ Toutes les caméras alignées (N) » / « ⚠ p/N caméras alignées —  <serial (filter)="">échec » / « Recadrage actif — Recalibrer pour aligner (utilise le capteur complet) » / « En attente de la stabilisation de l’exposition… ».
* Tableau par caméra : caméra (4 derniers chiffres du numéro de série + filtre), erreur de reprojection en px avec nombre de correspondances (« ref » pour la caméra maître), et score de corrélation croisée normalisée du chevauchement par rapport au seuil de réussite de 0,35.
* **Bouton**« Recalibrer l’alignement »** (intitulé « Calibrer l’alignement » avant le premier profil) — relance le co-enregistrement sur de nouvelles images.
* **Case à cocher «Cochez la case « Exposition automatique des caméras pour l’alignement »** (cochée par défaut) — éclaircit temporairement les caméras sombres ou plates (exposition d’abord, puis gain) afin qu’elles présentent une texture à faire correspondre, puis rétablit l’exposition automatique.

L’aperçu combiné s’aligne automatiquement à l’ouverture ; recalibrez si la mise au point ou la profondeur de la scène a changé. L’alignement est **par conception limité à la session** — il n’est jamais enregistré dans un profil, car il dépend de la distance de la scène à ce moment-là. Les captures peuvent toujours être exportées avec un alignement pixel par pixel (voir [Exportations alignées](capture.md#per-array-controls)).

### Vignette intelligente

* Case à cocher **Activer la correction**— applique l’estimation de la vignette par caméra à la chaîne radiométrique (en temps réel**et** lors des exportations).
* **Calibrer à partir de la vue actuelle**— pointez d’abord le réseau de caméras vers une cible uniforme (écran plat, mur ou ciel) ; chaque caméra est aplanie individuellement et l’état indique un gain d’aplanissement de « n/N caméras · −x,x % ».**Effacer** supprime l’estimation.
* Affinez le réglage par caméra à l’aide du curseur **Vignette** propre à chaque caméra dans [Aperçu en direct](#live-preview).

### Aperçu en direct *(combiné uniquement)** **Index**: cochez la case + icône en forme d’engrenage — cela ouvre le [Calculateur d’index](#index-calculator-pane) partagé avec des bandes tracées à partir de**toutes** les caméras du réseau. Une ligne d’aperçu de l’expression située en dessous affiche l’expression actuelle (« Aucune expression définie — ouvrez le calculateur pour en créer une »), actualisée toutes les secondes.
* Menu déroulant **Résolution de rendu**(mêmes préréglages que par caméra, 720p par défaut) : la hauteur du flux de prévisualisation en direct**et** la taille d’exportation du composite enregistré. Remarque dans le volet : « Aperçu + taille du composite enregistré. Les images par caméra sont toujours exportées en pleine résolution. »

### Calques d’affichage *(combinés uniquement)** Case à cocher **Activer** (désactivée par défaut — la caméra principale s’affiche directement ; activée = composite en calques).
* Menus déroulants **Premier plan**/**Arrière-plan**: chaque caméra membre (par nom) ou**Index**. Lorsque le premier plan est réglé sur « Index », les pixels situés en dehors des limites min/max de la LUT affichent le calque d’arrière-plan.

### Vue fractionnée *(combinaison uniquement)*

**« Afficher les caméras membres »**— un bouton**« Diviser / Masquer les caméras membres »** qui ajoute le flux en direct de chaque caméra membre sous forme de tuiles distinctes dans la grille, à côté de la composition. Les tuiles lisent le tampon d’images existant du réseau (aucune connexion supplémentaire par caméra). Affichage en grille uniquement ; enregistré par réseau avec le projet.

### Fonctionnalités

Un panneau en lecture seule actualisé toutes les 5 s :

* **Libellé du niveau** : « Capture simultanée » (vert) · « Capture simultanée (émission décalée FTD) » (vert) · « Capture décalée (dérive de 100 ms) » (orange) · « Configuration trop volumineuse » (rouge).
* **État des images** : « x,xx % incomplètes » — vert en dessous de 1 %, orange en dessous de 5 %, rouge à partir de 5 %.
* **Ligne de liaison** : « Carte réseau {mbps} Mbps - débit soutenu {MB/s} MB/s ».

Il s’agit du budget de bande passante en temps réel du réseau de caméras. Pour connaître le nombre d’images par seconde (fps) et le modèle réseau sous-jacents — ainsi que les modifications à apporter lorsque le niveau passe à l’orange ou au rouge —, consultez [Réseaux de caméras multiples](arrays.md) et la [Référence CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Volet « Calculateur d’indice »

La troisième page de la barre latérale, commune à l’engrenage « Index » par caméra et à l’engrenage « Index » de la matrice combinée (un à la fois — l’en-tête indique « Calculateur d’indice — <camera name="">» ou « Calculateur d’indice — <array name="">»). Elle reçoit la liste des bandes (les bandes naturelles du filtre de la caméra, ou toutes les bandes des membres du réseau), l’expression actuelle et la configuration de la LUT (activée/désactivée, niveau — par défaut 3, min — par défaut 0,2, max — par défaut 1), ainsi qu’un histogramme d’indice en temps réel. **Appliquer** valide l’expression ; les modifications de la table de conversion s’appliquent en temps réel à l’aperçu.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Paramètres par caméra vs paramètres gérés par le réseau

Référence rapide indiquant ce qui se trouve où lorsqu’une caméra fait partie d’un réseau :

| Géré par le réseau (en lecture seule dans le volet de la caméra) | Toujours par caméra au sein d’un réseau |
| --- | --- |
| Format des pixels, résolution, regroupement de pixels | Exposition automatique (exposition, gain, cible, lissage, zone d’intérêt) |
| Mode/source de déclenchement, fréquence d’images | Débruitage, netteté, vignettage |
| | Orientation (miroir/rotation), superpositions d’affichage, posemètre ponctuel |
| | Index (réseaux à affichage séparé), liaison au capteur de lumière |

Autres comportements transversaux :

* **Affichage combiné ou séparé** : ce choix s’effectue lors de la connexion au réseau de caméras : combiné = une seule tuile composite alignée (les flux des caméras membres ne sont disponibles que via Split View) ; séparé = chaque caméra membre affiche sa propre tuile synchronisée. Une caméra n’affiche jamais à la fois un flux autonome et une tuile du réseau.
* **Reconnexion automatique** : l’ouverture d’un projet enregistré restaure ses caméras et ses matrices et réapplique tous les paramètres enregistrés au backend avant la reprise des flux.
* **Contrôle de la capture** : les caméras masquées ou mises en pause sont exclues de la fonction « Capture All » ; un réseau n’est entièrement bloqué que lorsque TOUS les membres sont masqués/en pause. Voir [Paramètres et modes de capture](capture.md).

## Comment les paramètres sont conservés

L’état de l’onglet « Caméra » est enregistré **avec le projet**, et non dans le navigateur :

* Chaque modification réactive crée un instantané des caméras et des réseaux dans le fichier `cameras.json` du projet (avec un délai de 500 ms). Cela inclut les noms et couleurs des caméras, les réglages d’exposition/gain/AE, le format de pixel/résolution/binning, la fréquence de déclenchement, les paramètres de prévisualisation (résolution de rendu, débruitage, netteté, vignettage, profil de couleur, saturation/contraste), l’orientation, les superpositions, les divisions de canaux, la configuration de l’index, les réglages d’AE prédictif, la zone d’intérêt (ROI) pour l’exposition automatique, les noms de matrices, le mode d’affichage, les paramètres de capture des matrices (y compris la position de recadrage de la zone d’intérêt) et le bloc de grille (zoom sur le flux, mode d’affichage, verrouillage de la grille, ordre manuel des mosaïques, caméras masquées, mosaïques fermées, caméra active).
* Les associations de capteurs de lumière sont enregistrées dans le fichier `sensors.json` du projet.
* La réouverture du projet reconnecte le matériel et réapplique l’ensemble de ces paramètres.
* **Aucun projet ouvert = session uniquement** : en l’absence de projet, rien n’est conservé après la fermeture d’Chloros
* Exclusivement lié à la session, quel que soit le projet : l’état de pause, les échantillons du posemètre ponctuel, la case à cocher « Cible d’étalonnage » par caméra (toujours désactivée par défaut) et le profil d’alignement du réseau (recalculé à chaque session par défaut).
* Une exception : les sélections d’exportation des **paramètres de capture** et le mode de capture sont conservés par projet dans le stockage local de l’application plutôt que dans `cameras.json` — voir [Paramètres et modes de capture](capture.md).</array></camera></serial>
