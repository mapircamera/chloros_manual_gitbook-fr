# Paramètres des caméras

L&#x27;onglet **Caméras** est la surface de contrôle en direct d&#x27;Chloros

pour les caméras LATTICE : une zone d&#x27;affichage principale présentant chaque caméra connectée sous forme de vignette en direct, et une barre latérale permettant de basculer entre trois pages — la **liste des caméras**, un**volet de paramètres**(paramètres par caméra, par réseau ou de capture — un à la fois), et le**Calculateur d’index**. Cette page documente chaque commande de la liste des caméras, du volet des paramètres par caméra et du volet des paramètres du réseau. Les modes de capture, la sélection du type d’exportation et le flux « Capture All » se trouvent sur la page associée [Paramètres et modes de capture](capture.md).

L’onglet « Caméras » apparaît dans la barre latérale dès que le backend Chloros

est prêt. Toutes les commandes ci-dessous communiquent avec le backend local via `127.0.0.1:5000` ; les modifications s’appliquent immédiatement à la caméra en direct, sauf indication contraire.

## Types de caméras utilisés sur cette page

Les commandes s’affichent ou se masquent en fonction du type de caméra sélectionné. Le manuel utilise les termes suivants tout au long du document :

| Terme | Signification | Canaux de filtre |
| --- | --- | --- |
| **Caméra RGB** | LATTICE M3C avec filtre FRGB (modèle contenant `-FRGB`) | Red

/ Green

/ Blue

|
| **Multispectrale Bayer** | LATTICE M3C avec FRGN, FOCN ou FNGB | FRGN : Red

/ Green

/ NIR

· FOCN : Orange

/ Cyan

/ NIR

· FNGB : NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — un filtre à bande étroite, une bande calibrée | Bande unique |
| **Élément de réseau** | Une caméra connectée au sein d’un réseau synchronisé (affichage combiné ou séparé) | Selon son filtre |

Les caméras RGB

bénéficient d’un traitement photométrique (balance des blancs, profils de couleur, gamma) ; les caméras multispectrales et mono bénéficient de la chaîne radiométrique et ne passent pas par les contrôles photométriques. Les éléments du réseau transmettent les paramètres au niveau du flux (format de pixel, résolution, binning, déclenchement, fréquence d’images) au réseau — ces lignes deviennent alors en lecture seule dans le volet dédié à chaque caméra et sont déplacées vers le volet des paramètres du réseau.

## La zone principale du flux



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

En l’absence de caméras connectées, la zone du flux affiche un écran d’accueil **« Connectez une caméra pour commencer »**avec deux boutons :**Connecter une caméra**(vert, ouvre la boîte de dialogue de connexion d’une seule caméra) et**Connecter un réseau** (bleu, ouvre la boîte de dialogue de connexion d’un réseau). Les boîtes de dialogue de connexion sont décrites dans [Connexion des caméras](connecting.md) ; les concepts relatifs aux réseaux (synchronisation, niveaux, bande passante) dans [Réseaux multicaméras](arrays.md). Lorsque vous ouvrez un projet enregistré contenant des caméras, l’écran d’accueil affiche à la place une icône tournante accompagnée du message « Réouverture de N caméras enregistrées… » pendant qu’Chloros

restaure les flux de la dernière session.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barre supérieure

| Commande | Fonction |
| --- | --- |
| **Basculer entre les modes d’affichage**| Bascule entre la**vue en grille**(toutes les vignettes sous forme de cellules) et la**vue en liste**(lignes pleine largeur : chaque réseau combiné forme une ligne composite, chaque élément d’un réseau**distinct** dispose de sa propre ligne avec son flux et son tableau de cibles d’étalonnage, puis UNE caméra autonome — celle qui est active — en dessous). Infobulles : « Passer en affichage en grille » / « Passer en affichage en liste ». |
| **Verrouillage de la grille**(cadenas) |**Verrouillé** par défaut — les tuiles sont figées à leur emplacement. Déverrouillez pour faire glisser et réorganiser les tuiles dans n’importe quel emplacement (les espaces vides sont conservés). La grille se verrouille automatiquement à chaque fois qu’une nouvelle caméra se connecte. Infobulles : « Déverrouiller la grille (activer le glissement des tuiles) » / « Verrouiller la grille (figer les tuiles en place) ». |
| Curseur **Zoom du flux** | Taille des tuiles, de 60 px jusqu’à la largeur totale du conteneur. Les cellules conservent un format 4:3. En dessous de 200 px de largeur de cellule, les superpositions du nom et de l’images par seconde (ips) se masquent pour garder la tuile épurée. |

### Tuiles du flux

Chaque caméra affiche une tuile composite en direct ; une caméra peut en outre afficher trois tuiles en niveaux de gris **séparées par canal** (voir [Séparation des canaux](#affichage-des-superpositions-dessinées-sur-le-flux-en-direct)), et les réseaux de caméras affichent une vignette combinée. La vignette active est signalée par un anneau de sélection de la couleur de la caméra (ou du réseau).

En survolant une vignette, un bouton de fermeture **X** apparaît :

* Fermer une vignette **composite** alors que ses séparations de canaux restent affichées masque simplement la vignette composite.
* Fermer la **dernière vignette visible d’une caméra autonome** déconnecte cette caméra.
* **Les vignettes de séparation des membres d’un réseau combiné ne déconnectent jamais** la caméra — elles la masquent simplement.

Lorsque la grille est déverrouillée, faites glisser n’importe quelle tuile vers n’importe quel emplacement ; la disposition est enregistrée avec le projet.

## Barre latérale — liste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

des caméras La première page de la barre latérale répertorie toutes les caméras et tous les réseaux connectés :

* **Connecter une caméra**(vert) /**Connecter un réseau** (bleu, affiche « Détection… » pendant l’analyse). Les deux options sont désactivées lorsqu’une boîte de dialogue de connexion est ouverte.
* **Tout capturer**(rouge) — capture toutes les caméras répertoriées avec les types d’exportation choisis dans les**Paramètres de capture**. Nécessite un projet ouvert. Entièrement documenté dans [Paramètres et modes de capture](capture.md).
* **Roue dentée des paramètres de capture** (à côté de « Capture tout ») — ouvre le [volet des paramètres de capture](capture.md#the-capture-settings-pane). Désactivé en l’absence de projet ou pendant une capture.

### Lignes de caméras

Chaque ligne de caméra affiche une bordure avec un code couleur (la couleur personnalisée de la caméra), une étiquette « CAM » — avec une lettre bleue **M**(maître) ou une lettre verte**S** (esclave) indiquant le rôle des membres du réseau — ainsi que le nom d’affichage. Le nom par défaut est `LATTICE-MODEL (serial)` ; vous pouvez le renommer depuis le volet des paramètres de chaque caméra. Boutons de la ligne :

| Bouton | Effet |
| --- | --- |
| **Œil**| Active/désactive la visibilité. Les caméras masquées disparaissent de la grille et sont**exclues de la fonction « Capture All »**. |
| **Roue dentée** | Ouvre le volet des paramètres propres à chaque caméra (section suivante). |
| **Pause / Lecture**| Fige l’aperçu en direct**côté affichage uniquement** — la capture côté serveur continue de fonctionner. Les caméras en pause ne peuvent pas capturer. |
| **X** | Déconnecter. L’interface utilisateur se met à jour immédiatement (dans le meilleur des cas) ; la déconnexion effective au niveau du backend peut prendre entre 10 et 30 s. |

### Lignes du tableau

Une ligne du tableau affiche un badge « ARRAY » dans la couleur du tableau, le nom du tableau (renommable dans les paramètres du tableau) et un bouton **DAQ · activé/désactivé**—**activé** lorsque le capteur de lumière au niveau du tableau est activé *ou* qu’un membre dispose d’un capteur par caméra ; son info-bulle indique précisément quel capteur alimente quoi. Les caméras membres sont répertoriées en retrait en dessous, chacune dans sa propre ligne. Boutons de la ligne de réseau : **œil**(masque/affiche TOUS les éléments à la fois),**roue dentée**(volet des paramètres du réseau),**X**(déconnecte l’ensemble du réseau).

L’état du capteur de lumière (DLS) utilisé dans les lignes de réseau et dans le volet des paramètres du réseau comporte quatre états :**éteint**,**en attente**(pas encore de spectre),**actif**(un spectre est arrivé au cours des 3 dernières secondes) et**périmé** — pas de spectre récent depuis 3 secondes, mais la dernière lecture est *toujours utilisée* (les relevés DAQ n’expirent jamais sur le chemin de capture).

Tant que vous ne modifiez pas l’ordre, la liste affiche d’abord les réseaux, puis les caméras autonomes, les caméras (et les éléments de chaque réseau) étant classées selon l’**ordre des caméras** : M3C (Bayer) avant M3M (mono) ; parmi les M3C, FRGB, FRGN, FOCN, puis FNGB ; les M3M par longueur d’onde du filtre, en commençant par la plus basse. La disposition par défaut de la grille suit le même ordre. Vous pouvez faire glisser les caméras autonomes et les groupes de matrices entiers les uns devant les autres dans la barre latérale pour réorganiser la liste ; les éléments d’une matrice ne peuvent pas être déplacés indépendamment.

## Volet des paramètres par caméra

Ouvrez-le à l’aide de l’**icône en forme d’engrenage** située sur une ligne de caméra. Le volet s’affiche par-dessus la liste des caméras.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**En-tête**: l’**échantillon de couleur**de la caméra (cliquez pour ouvrir un sélecteur de couleur natif — permet de définir la couleur de la bordure de la barre latérale et de l’anneau de sélection des vignettes), le**nom**avec un bouton**Renommer**en forme de crayon (si vous enregistrez un nom vide, le nom par défaut `MODEL (serial)` est rétabli), et**×** pour fermer.

### Histogramme en temps réel

En haut du volet se trouve un histogramme de luminance en temps réel calculé à partir de l’aperçu JPEG

à environ 8 Hz. La moyenne est pondérée selon le principe de Bayer — (R+2G+B)/4 — afin de correspondre à la mesure AE propre à la caméra.

* **Ligne pointillée Orange**= la cible de l&#x27;exposition automatique. **Faites-la glisser horizontalement pour redéfinir la cible** — une commande est envoyée lorsque vous relâchez le curseur, et le glissement bascule le mode de cible d&#x27;exposition automatique en mode manuel.
* **Ligne continue Green** = la luminance moyenne réelle (ce que l&#x27;exposition automatique fournit actuellement).
* **Bouton RGB** (en haut à droite) : active/désactive les histogrammes superposés par bande, colorés selon le filtre de l’appareil photo (par exemple, sur FRGN : gris pour le NIR

, vert, rouge). Sur les appareils photo mono (M3M), le bouton affiche « MONO » et est désactivé — le mode mono affiche toujours l’histogramme de luminance à bande unique.
* Les étiquettes de l’axe X suivent la profondeur de bits du capteur du format de pixel actuel : 0..255, 0..1023, 0..4095 ou 0..65535.

### Lignes

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

d’informations sur la caméra

| Ligne | Comportement |
| --- | --- |
| **Modèle** | En lecture seule (par ex. `LATT-M3C-L87-FRGN`). |
| **Étalonnage radiométrique** | Badge Green**« Actif »**avec une légende indiquant le niveau d’étalonnage, le hachage, la date d’étalonnage et la liste des bandes, chargés à partir du pack d’étalonnage de la caméra (voir [Étalonnage radiométrique d’usine](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Masqué pour les caméras RGB

** — celles-ci disposent d’un étalonnage photométrique de la balance des blancs, et non d’une radiance par bande. |
| **Rapport d’étalonnage**| Bouton**Télécharger** — ouvre le certificat d’étalonnage NIST propre à chaque numéro de série de la caméra au format PDF dans la visionneuse de votre système d’exploitation. Si le certificat n’est pas encore mis en cache, Chloros

affiche un message d’aide à la place. |
| **Numéro de série** | En lecture seule. |
| **Micrologiciel**| Affiche la version actuelle, puis recherche la version disponible pour ce modèle (mise en cache par modèle — un réseau de N caméras effectue une seule vérification sur le serveur). États : « Vérification en cours… » → bouton**« Mettre à jour vers X »**→ « Mise à jour en cours… » → « Mis à jour de A vers B » / « Échec : … » / « Ignoré : … » /**« À jour »** en vert. Info-bulle du bouton de mise à jour : « Réinitialisation d&#x27;usine + flashage + reprogrammation de UserSet1. ~2 à 3 minutes ; ne pas déconnecter. » |
| **IP** | En lecture seule. |
| **Température** | En lecture seule, actualisée toutes les 3 s. Passe à l’orange à partir de 65 °C et au rouge avec un ⚠ à partir de 75 °C. |
| **Cible d’étalonnage** | Active la détection de la cible de réflectance ArUco avec un tableau de validation NDVI

par panneau situé sous le flux en direct (vue en liste) — voir [le tableau des cibles d’étalonnage](#the-calibration-target-table). Valable uniquement pour la session — s’ouvre toujours désactivée. |
| Menu déroulant **Capteur de lumière** | Associe un capteur de lumière DAQ (DAQ-E/M/U, issu de la liste de l’onglet « Capteurs de lumière ») à cette caméra pour la correction de l’éclairage par lumière descendante (DLS) et l’exposition automatique prédictive. «« Aucun » annule l’association. Si aucun capteur n’est connecté, le menu déroulant affiche « (aucun capteur connecté — ouvrir l’onglet DAQ) ». L’association est enregistrée avec le projet. |

#### Le tableau « Cible d’étalonnage »

Ce tableau est un **affichage de validation uniquement** (« validation uniquement — ne modifie pas l’étalonnage »). Son en-tête indique le numéro de série de la caméra, le filtre, le numéro de la cible et la taille du marqueur en pixels, et il comporte une ligne par panneau :

* **Panneau**, puis une colonne par bande avec la valeur du panneau — une caméra Bayer affiche ses trois bandes par nom et longueur d’onde (par ex. FRGN : Red

660 nm / Green

550 nm / NIR

850 nm) ; une **caméra mono (M3M) affiche sa bande unique** (par exemple `F750`) et ne comporte pas de colonnes NDVI

.
* **Sat** — la proportion de pixels du panneau qui sont saturés (orange à partir de 1 %, rouge à partir de 5 %).
* **NDVI

Meas**,**NDVI

Theo** (NDVI

de réflectance de référence du panneau) et **Δ**— caméras multispectrales Bayer uniquement. Une petite marque après « NDVI

Meas » indique à partir de quoi l’NDVI

mesuré a été calculé :**ρ**(réflectance — une lecture du capteur de lumière est disponible),**L**(radiance — aucune lecture du capteur de lumière) ou**DN** (valeurs brutes de la caméra). Seul un NDVI**ρ** est comparable à l’NDVI

théorique : pour L et DN, la colonne Δ est grisée et une note l’indique (« Aucune mesure du capteur de lumière : le NDVI

est calculé à partir de la radiance, donc Δ ne constitue pas un contrôle d’étalonnage » / « Le NDVI

est calculé à partir des valeurs brutes de la caméra, donc Δ ne constitue pas un contrôle d’étalonnage »). Avec ρ, Δ est vert en dessous de 0,02, orange en dessous de 0,05, rouge dans les autres cas.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Exposition et gain

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Toutes les entrées numériques ici utilisent des curseurs à maintien pour l’accélération : tapoter = ±1, maintenir &gt; 1,5 s = ±10, maintenir &gt; 3 s = ±100. La valeur est envoyée à la caméra lorsque vous relâchez le doigt.

| Commande | Plage / choix | Par défaut | S&#x27;applique à | Fonction |
| --- | --- | --- | --- | --- |
| **Exposition (us)**| Min./max. en temps réel de la caméra | Auto | Tout | Durée d’exposition en microsecondes, avec un bouton**Auto/Manuel**. Auto = exposition automatique continue côté caméra. Le réglage d’une exposition manuelle laisse le**Gain** sur Auto s’il l’était, permettant d’utiliser toute la plage de gain de la caméra, jusqu’à ce que vous modifiiez le gain ou que vous le régliez sur Manuel. |
| **Gain (dB)**| Plages minimale et maximale en temps réel de la caméra (par ex. jusqu’à 48 dB) | Manuel (désactivé) | Tout | Gain analogique/numérique avec son propre commutateur**Auto/Manuel**. |
| **Luminosité cible AE**| 0–255 | 80, mode**Auto**| Tout (modifiable lorsque l’AE ou le gain automatique est activé) | Luminosité visée par l’AE. En**Auto**(réglage par défaut), un contrôleur interne basé sur l’histogramme sélectionne lui-même la valeur cible, en maintenant l’exposition entre 60 et 75 % de la valeur maximale du capteur. La saisie d’une valeur ou le glissement de la ligne orange de l’histogramme bascule le mode en**Manuel**. |
| **Lissage AE** | 0,5–40, par pas de 0,1 | 8,0 | Tout | Amortissement de l&#x27;AE. Info-bulle : «Plus la valeur est faible, plus l’AE réagit rapidement (peut provoquer des saccades à des fréquences d’images élevées). Plus la valeur est élevée, plus la réaction est douce et lente. » Des valeurs bien inférieures à la valeur par défaut peuvent provoquer des saccades de l’AE et déstabiliser le streaming à des fréquences d’images élevées ; 8,0 est la valeur par défaut stable. |
| **Zone d’intérêt de l’AE**| Case à cocher « Activer » + bouton**Viser**| Désactivé | Tout | Lorsque cette option est activée, AE mesure uniquement la zone verte en pointillés au lieu de l’image entière.**Cibler** active le placement par clic sur le flux en direct : un clic centre une zone à 30 % de l’image ; cliquer-glisser trace un rectangle personnalisé (minimum 5 % × 5 %). La fonction « Aim » se désactive automatiquement après un placement. La zone est remappée aux coordonnées natives de la caméra en fonction de toute rotation ou réflexion que vous avez définie, et est enregistrée avec le projet. |
| **Vitesse de réglage AE** | 0,1–5, par pas de 0,1 | 1,0 | Réservé aux membres du tableau | Vitesse à laquelle la cible AE automatique suit les changements de luminosité de la scène ; 1,0× effectue une nouvelle vérification toutes les 2,5 s. |
| **Protection des hautes lumières** | Stricte (1 %) / Normale (5 %) / Souplesse (15 %) | Stricte | Caméras permettant de régler ce paramètre | Pourcentage de l’image pouvant être écrêté en blanc avant que l’exposition automatique n’assombrisse l’image. |

{% hint style="info" %}
**Exigences en matière de luminosité pour les caméras multispectrales Bayer (RGN

/ OCN

/ NGB

) :** la scène doit être suffisamment éclairée sur les trois canaux, sinon l&#x27;étalonnage ne fonctionne pas correctement — une seule exposition du capteur couvre les trois spectres. Utilisez un capteur de lumière DAQ pour mesurer votre luminosité, ou passez en mode entièrement mono (M3M) afin que chaque bande bénéficie de sa propre exposition. Si une capture ne respecte pas cette condition, Chloros

le détecte et vous avertit (notification « unmix-clamp »). En temps réel, une mosaïque de séparation des canaux affiche **« Données de bande à faible confiance · N % »** lorsque le démixage a limité cette bande sur 25 % ou plus de ses pixels.
{% endhint %}

{% hint style="info" %}
**Comment l’exposition automatique gère l’exposition et le gain (1.3.0).** Lorsque le gain est en mode « Auto », Chloros

maintient le gain à 0 dB jusqu’à ce que l’exposition ait atteint la valeur maximale autorisée (la limite supérieure de l’exposition automatique, la période de déclenchement d’un réseau ou le plafond d’exposition du DAQ) ; ce n’est qu’alors que le gain augmente. Dans un réseau, la fréquence de déclenchement est prioritaire : l’exposition automatique remplit la période définie par cette fréquence, puis augmente le gain ; ainsi, une scène qui s&#x27;assombrit ne réduit plus la fréquence de déclenchement en mode Auto (ce qui était le cas dans la version 1.2.0). Une exposition **manuelle**sur un élément du réseau dont la durée est supérieure à la période de déclenchement ralentit la fréquence de déclenchement du réseau pour s&#x27;y adapter ; le taux d’exposition automatique est rétabli dès que chaque élément qui l’avait ralenti est revenu en exposition automatique. Lorsque les paramètres**Exposition**et**Luminosité cible AE** sont tous deux réglés sur Auto et qu’un capteur de lumière DAQ fournit une mesure, l’exposition de chaque caméra est maintenue une fois que l’exposition automatique s’est stabilisée, tant que la lumière mesurée reste dans une fourchette d’environ ±25 % ; un changement de luminosité important et prolongé, une réinitialisation des paramètres (par exemple un changement de zone d’intérêt) ou une modification de l’exposition de votre part le réactive.
{% endhint %}

{% hint style="info" %}
**Fonctionnement de l’exposition automatique (1.3.0).**

* Une exposition manuelle laisse le gain en mode « Auto » jusqu’à ce que vous le modifiiez.
* Les petites zones surexposées n’assombrissent pas la scène : une réflexion spéculaire représentant jusqu’à 1 % du cadre est ignorée, et dans les panneaux d’une cible d’étalonnage connue, jusqu’à 0,5 % d’un panneau.
* Le plafond d’exposition DAQ s’applique uniquement lorsqu’une cible d’étalonnage est dans le champ de vision ; en l’absence de panneau visible, la caméra expose en fonction de la scène.
* L’exposition reste stable en cas de luminosité constante et se stabilise en quelques secondes après un changement important. Un écrêtage ne limite plus définitivement l’exposition, et un changement de zone d’intérêt (ROI) permet de retrouver l’exposition adéquate.
* La cible de luminosité ne dépasse pas laportée ; ainsi, un retour de lumière ne provoque pas de reflets dans l’image.
* Une caméra dont l’exposition automatique cesse de réagir en plein soleil est maintenue au niveau correspondant à la luminosité ambiante, mesurée par le capteur de lumière ou, en l’absence de capteur, par les images de la caméra elle-même, et relâchée lorsque la luminosité change. Un capteur de lumière obscurci ou couvert ne perturbe pas ce fonctionnement.
{% endhint %}

### Format des pixels et résolution

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Les éléments du réseau** affichent des lignes en lecture seule « Actuel » (format + LxH) et « Binning » avec la remarque « Définir dans les paramètres du réseau » — un redémarrage du flux sur un élément romprait la synchronisation, c’est pourquoi ces paramètres sont gérés dans le [volet des paramètres du réseau](#array-settings-pane).

Les **caméras autonomes** disposent des options suivantes :

| Commande | Choix | Fonction |
| --- | --- | --- |
| **Format des pixels** | 8 bits / 10 bits / 12 bits / 16 bits | Profondeur de bits du capteur ; la famille de formats est celle du capteur. La valeur par défaut est 12 bits, et un projet enregistre le paramètre que vous définissez, y compris 8 bits, et se reconnecte en respectant ce paramètre. |
| **Résolution** | Pleine / Moitié / Quart | Par rapport au binning actuel : Pleine = 2048/N × 1536/N pour un regroupement N×N. |
| **Regroupement** | 1x1 (aucun) / 2x2 / 4x4 | Regroupement matériel N×N — des valeurs plus élevées réduisent la résolution mais améliorent le rapport signal/bruit et la fréquence d’images. Sa modification redémarre le flux et réinitialise toute zone d’intérêt (ROI) au nouveau champ de vision complet. |
| **Actuel** | en lecture seule | Les valeurs réelles de largeur × hauteur et de décalage (x, y) actuellement en vigueur. |

### Aperçu en direct

Tout ce qui se trouve dans cette section concerne **uniquement l’affichage**— cela modifie ce que vous voyez dans le flux en direct, tandis que les captures enregistrées restent linéaires et inchangées — à une exception près : la**vignette** est radiométrique et affecte également les exportations (comme indiqué ci-dessous).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Commande | Plage / choix | Par défaut | S&#x27;applique à | Fonction |
| --- | --- | --- | --- | --- |
| **Résolution de rendu** | 360p (la plus rapide) / 480p / 720p / 1080p / Résolution native du capteur (la plus lente) | 720p | Tout | La hauteur à laquelle le backend exécute la chaîne de prévisualisation radiométrique. Une valeur plus faible permet d’augmenter la fréquence d’images sans modifier le champ de vision. |
| **Index**| Case à cocher « Activer » + roue dentée | Désactivé | Multispectral Bayer uniquement,**pas** les membres à matrice combinée | Prévisualisation en direct de l’indice de végétation. L&#x27;icône en forme d&#x27;engrenage ouvre le [Calculateur d&#x27;indice](#index-calculator-pane) partagé, préchargé avec les bandes naturelles du filtre de la caméra (par ex. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). L’expression personnalisée ainsi que la table de conversion (LUT) (activée/désactivée, niveau par défaut 7 stops, min par défaut 0,2, max par défaut 1) est calculée à chaque image de prévisualisation. Les membres de l’array combiné masquent cette ligne — l’array possède un seul indice partagé. |
| **Balance des blancs** | Désactivé / Une fois / Continu + un bouton de recapture | Continu | RGB

uniquement | Balance des blancs en temps réel. Le bouton de rafraîchissement relance la mesure de la balance des blancs à partir du spectre DLS actuel (désactivé lorsque le mode est désactivé). |
| **Gamma** | Activé / Désactivé | Activé | RGB

uniquement | Affichage du gamma (γ = 2,2 LUT) sur l’aperçu en direct. Les captures enregistrées restent linéaires. |
| **Réduction du bruit** | Case à cocher + intensité 0–100 | Désactivé / 50 | Tout (par caméra, même au sein des matrices) | Filtre bilatéral sur l’aperçu en direct. Plus la valeur est élevée, plus l’image est lisse, mais les détails sont moins nets. |
| **Netteté** | Case à cocher + intensité 0–100 | Désactivé / 30 | Tout | Masque de netteté sur l’aperçu en direct, appliqué en dernier. Peut amplifier le bruit. Aperçu uniquement. |
| **Vignettage**| Case à cocher + intensité 0–100 | Désactivé / 0 | Tout | Suppression manuelle du vignettage résiduel (éclaircit les coins), superposée à l&#x27;estimation du vignettage intelligent du réseau.**Radiométrique — affecte la vue en direct ET les exportations**, contrairement au débruitage et à la netteté. |
| **Profil de couleur** | Raw / Linéaire / Naturel / Amélioré / Température personnalisée | Naturel | RGB

uniquement | Voir ci-dessous. |
| **Température de couleur** | 2 000–10 000 K, par pas de 100 | 5 500 K | RGB

uniquement, profil de température personnalisé uniquement | Fixe la balance des blancs à une température de couleur corrélée fixe (entrée DLS ignorée). La dernière valeur en Kelvin sélectionnée est mémorisée lors des changements de profil. |
| **Hautes lumières** | De −100 à 0, par pas de 1 | 0 (désactivé) | RGB

uniquement | Nouveauté de la version 1.3.0. Assombrit les zones les plus lumineuses et les plus blanches de l’image (0 = inchangé, −100 = intensité maximale). S’applique à l’aperçu en direct **et aux captures au niveau de l’affichage (Aperçu)**, qui enregistrent le paramètre (`Chloros:DisplayHighlights`) ; les exportations de radiance et de réflectance ne sont jamais affectées. Cette fonction ne permet pas de récupérer les données du capteur qui ont été écrêtées — réduisez l’exposition pour y remédier. La superposition zébrée suit l’image assombrie. Désactivée avec le profil de couleur Raw (« Non appliqué avec le profil de couleur Raw. »). |
| **Saturation** | 0–200 (100 = neutre) | 100 | RGB

uniquement | Saturation HSV sur l’aperçu en direct. |
| **Contraste** | 0–200 (100 = neutre) | 100 | RGB

uniquement | Contraste linéaire autour du gris moyen dans l’aperçu en direct. |
| **Miroir H / Miroir V** | Cases à cocher | Désactivé | Tout | Retourner l’aperçu horizontalement / verticalement. |
| **Rotation**| 0° / 90° / 180° / 270° | 0° | Tout | Faire pivoter l’aperçu. L’orientation est appliquée à la fin de la chaîne de prévisualisation du backend.**Captures enregistrées (1.3.0) :**les produits « Preview », « Radiance », « Reflectance » et « Index » d’une caméra autonome sont retournés par les fonctions « Miroir » et « Rotation », tandis que**les fichiers « Raw » et « Debayered » conservent l’orientation native de la caméra**; chaque fichier de capture enregistre ce paramètre dans son XMP, de sorte que le traitement ultérieur d’un fichier « Raw » l’applique de la même manière.**Les fichiers membres d’un tableau conservent leur orientation native** (le paramètre est enregistré, mais non appliqué), et les vues composites du tableau l’ignorent. |**Sémantique des profils de couleur** (caméras RGB

) :

* **Raw** — contourne entièrement la chaîne de traitement.
* **Linéaire** — signal sombre + champ plat + balance des blancs ; pas de matrice de couleur, pas de gamma.
* **Naturel** *(par défaut)* — linéaire plus la matrice de correction des couleurs mesurée et une courbe de tonalité adaptative à la scène.
* **Amélioré**— Naturel plus la vibrance et le contraste local CLAHE. Le surcoût s’applique**uniquement à l’aperçu en direct** — les captures enregistrées bénéficient toujours du traitement complet, quel que soit le profil.
* ****Température personnalisée** — « Naturel » avec la balance des blancs fixée à la valeur en Kelvin de votre choix.

{% hint style="warning" %}
Pour les modes « Naturel », « Amélioré » et « Température personnalisée », le volet affiche une remarque sur la tonalité : les images sont éclaircies en fonction de leur propre scène, de sorte que les images *d&#x27;affichage* enregistrées ne sont pas comparables d&#x27;une image à l&#x27;autre. **Exportez la radiance ou la réflectance pour les mesures.**
{% endhint %}

### Superpositions d’affichage (dessinées par-dessus le flux en direct)

Ces superpositions sont propres à l’interface utilisateur : elles sont dessinées par-dessus la vidéo, sans jamais toucher au flux ni aux captures.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Superposition | Commandes | Par défaut | Fonction |
| --- | --- | --- | --- |
| **Zébré** | Case à cocher + seuil 200–255 | Désactivé / 250 | Bandes diagonales magenta sur les pixels écrêtés. |
| **Réticule** | Case à cocher | Désactivé | Repère au centre de l&#x27;image. |
| **Grille** | Désactivée / 3 × 3 / 9 × 9 | Désactivée | Grille de composition. |
| **Histogramme** | Case à cocher + largeur 0,10–0,90 de l’image | Désactivée / 0,25 | Bande d’histogramme sur le flux. |
| **Focus Peak** | Case à cocher + seuil 20–200 + échantillon de couleur | Désactivé / 80 / `#ff5722` | Mise en évidence des contours par l’algorithme de Sobel pour la mise au point. |
| **Séparation des canaux** | Bouton « Afficher les séparations (Red

/ Green

/ NIR

) » / Bouton « Masquer les séparations » | Masqué | Ajoute trois vignettes indépendantes en niveaux de gris, une par canal, à côté de l’image composite (le libellé du bouton correspond aux canaux de filtre de la caméra). Chaque vignette séparée est déplaçable et reprend la couleur de bordure de la caméra. Non disponible sur les caméras monochromes. Enregistré avec le projet. |

### Posemètre ponctuel

* Case à cocher **Cliquer pour échantillonner**: cliquez sur le flux en direct pour échantillonner un seul pixel (un réticule en croix le marque), ou cliquez et faites glisser pour sélectionner une zone et obtenir une moyenne des pixels.**Effacer**supprime l’échantillon et le réticule. Fonction mutuellement exclusive avec le mode**Viser** de l’AE-ROI.
**Menu déroulant **Afficher**:**Brut (profondeur de bits)**— valeurs numériques natives à la profondeur de bits du capteur (par ex. 12 bits → 0..4095) — ou**Affichage (8 bits)** (par défaut). Lorsqu’un indice en temps réel est actif, l’option Affichage affiche à la place la valeur de l’indice calculée (par ex. NDVI

).
* Le panneau d’affichage répertorie les coordonnées des pixels, la taille de l’image, le format des pixels, la profondeur de bits et un tableau des canaux (Chan / Valeur / %) avec les libellés des bandes et les longueurs d’onde ; les paires vertes Bayer sont moyennées ; les échantillons de région affichent « N px avg ».

L’état du posemètre ponctuel est valable uniquement pour la session en cours.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Exposition automatique prédictive (pilotée par DLS)

Cette section n’apparaît que lorsqu’**au moins un capteur de lumière DAQ est connecté** — le solveur a besoin d’un spectre descendant en temps réel pour fonctionner.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Commande | Plage | Par défaut | Fonction |
| --- | --- | --- | --- |
| **Activer** | Case à cocher | Activé (caméras autonomes) | Un solveur de forme fermée utilise le spectre DLS ainsi que les scalaires du pack d’étalonnage de la caméra pour amener la bande la plus lumineuse près de la saturation tout en maintenant la bande la plus sombre au-dessus du seuil de rapport signal/bruit (SNR) — une seule écriture d’exposition par résolution, sans boucle de stabilisation. Conçu pour les time-lapses alimentés par énergie solaire où chaque prise de vue doit être correctement exposée. Le backend bascule silencieusement vers l’exposition automatique réactive dès que la lecture DLS est obsolète/manquante ou que le pack d’étalonnage n’est pas chargé. |
| **Lissage (α)** | 0,05–1,0, pas de 0,05 | 0,3 | Lissage des solutions prédictives successives (plus la valeur est faible, plus le lissage est important). |
| **Réflectance de la scène**| Bouton**Recalibrer ρ** | — | Réévalue le facteur de réflectance de la scène utilisé par le solveur. |

{% hint style="info" %}
**La connexion en réseau désactive l’exposition automatique prédictive par défaut** — pour les réseaux, l’exposition automatique intelligente d’Chloros

ainsi que l’exposition automatique côté appareil photo gèrent l’exposition (avec protection contre la saturation) et l’estimation de la réflectance d’une seule scènen’est pas fiable dans des scènes mixtes. Vous pouvez la réactiver ici pour chaque caméra si vous souhaitez spécifiquement une exposition radiométrique pilotée par DLS.
{% endhint %}

**Plafond d’exposition piloté par DAQ et AE indexée sur la lumière incidente.** Indépendamment de la case à cocher ci-dessus, lorsqu’un capteur de lumière DAQ est attribué à une caméra RGB

, Chloros

calcule — à partir de l’irradiance descendante absolue mesurée — l’exposition × gain maximal auquel une surface à 100 % de réflectance reste en dessous du seuil de clipping, et l’applique comme **plafond**de l’exposition automatique. Depuis la version 1.3.0, ce plafond ne s’applique que lorsqu’une cible d’étalonnage se trouve dans le champ de vision de cette caméra (apparue dans son cadre au cours des 10 dernières secondes) ; en l’absence de cible dans le champ de vision, c’est l’exposition automatique habituelle de la caméra, dotée de sa propre protection contre les surexpositions, qui prend le relais, afin que le reste de la scène ne soit pas assombri plus que nécessaire. Tant que la limite est active, la caméra est**verrouillée sur la lumière incidente** : elle fonctionne en boucle ouverte à l’exposition mesurée en lumière incidente avec un gain à 0 dB — l’exposition suit la lumière mesurée, et non le contenu de la scène. Comme la limite maximale ne peut que raccourcir l’exposition, elle ne peut pas en soi provoquer d’écrêtage. Depuis la version 1.3.0, le plafond est calculé à partir de la constante d’exposition propre à la caméra, issue de son pack d’étalonnage (une valeur moyenne du parc n’est utilisée que lorsque le pack n’en contient pas). Le plafond se désactive automatiquement — et l’exposition automatique normale de la scène reprend — dès qu’aucune cible d’étalonnage n’est dans le champ de vision, que la lecture DAQ est manquante, périmée (&gt; 30 s), ou sombre, ou si ≥ 15 % de l’image est écrêtée à l’exposition fixée (ce qui signifie que le capteur et la caméra perçoivent un éclairage différent). Il n’y a pas de commutateur dans l’interface graphique ; il s’agit d’un comportement standard dès lors qu’une caméra RGB

est liée à un DAQ.

### Acquisition et déclenchement

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Les membres du réseau affichent en outre des lignes en lecture seule : **Rôle**(Maître en bleu / Esclave en vert),**Ligne de synchronisation**et**Pairs**.

| Contrôle | Choix | Par défaut | Remarques |
| --- | --- | --- | --- |
| **Mode de déclenchement** | Désactivé / Activé | Activé | Désactivé pour les éléments du tableau (le tableau gère le déclenchement). |
| **Source de déclenchement** | Logiciel / Ligne 0 (M8) / Ligne 1 / Ligne 2 | Ligne 0 | Masquée lorsque le mode de déclenchement est désactivé ; désactivée pour les membres du réseau. La ligne 0 correspond à l’entrée de déclenchement externe opto-isolée M8. |
| **Fréquence d’images**| Auto / Manuel + valeur | Auto |**Auto**: la limite de fréquence d&#x27;images de la caméra est désactivée — l&#x27;exposition détermine le nombre d&#x27;images par seconde, et la fenêtre affiche la fréquence réelle en direct.**Manuel** : vous limitez le nombre d’images par seconde à l’aide d’un curseur (de 1 jusqu’au maximum autorisé par la bande passante), en vous basant sur la fréquence réelle actuelle. Les membres du réseau voient s’afficher une valeur en lecture seule « N images par seconde (en direct) » avec la mention « Définir dans les paramètres du réseau ». |

### Réseau / Transport

| Ligne | Comportement |
| --- | --- |
| **Taille de paquet**| 1 500 (Standard) / 9 000 (Jumbo) —**Jumbo** par défaut. |
| **Débit** | Limite de débit de la liaison en Mo/s, en lecture seule. Le backend rééquilibre cette valeur entre toutes les caméras connectées à chaque connexion/déconnexion. |
| **Gestion du tampon** | Mode de gestion du tampon, en lecture seule. |

### Capture

Le volet se termine par un bouton **« Ouvrir les paramètres de capture… »** qui permet d’accéder au [volet Paramètres de capture](capture.md#the-capture-settings-pane) (désactivé tant qu’aucun projet n’est ouvert — « Créez ou ouvrez un projet pour enregistrer les captures »). Si la caméra est masquée ou en pause, une remarque vous invite à la démasquer ou à la remettre en marche avant de lancer la capture.

## Volet des paramètres de matrice

Ouvrez-le à l’aide de l’**icône en forme d’engrenage**située sur une ligne**ARRAY**. En-tête : nom du réseau avec un crayon pour le renommer et un**×** pour fermer. Les sections ci-dessous marquées *« combiné uniquement »* n’apparaissent que pour les réseaux connectés en mode d’affichage combiné.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synchronisation

Lignes **Maître**,**Esclaves**et**Ligne de synchronisation** en lecture seule.

### Capteur de lumière ambiante

Affiché pour les réseaux combinés et séparés :

* Menu déroulant **Capteur de lumière**— associe un DAQ à l’ensemble du réseau. La sélection prend effet immédiatement, se répercute sur le menu déroulant**Capteur de lumière** de chaque caméra membre (vous pouvez toujours le remplacer individuellement pour chaque caméra) et commence à transmettre les spectres au réseau.
* Ligne **État** en temps réel : Désactivé · « En attente du premier spectre… » · « Actif — toutes les caméras du réseau bénéficient d’une correction d’éclairage » · « Aucun nouveau spectre au cours des 3 dernières secondes — utilisation de la dernière lecture (pas de délai d’expiration)… ».
* Remarque dans le volet : « Correction radiométrique à l’échelle du réseau. Les paramètres propres à chaque caméra prévalent sur cette correction. »

### Cible d’étalonnage

Son propre groupe se trouve juste après le capteur de lumière ambiante, tant pour les réseaux combinés que séparés :

* Case à cocher **Activer** — « Détecter la cible ArUco MAPIR

et valider la LUT NDVI

par rapport à la réflectance du panneau » ; détermine la superposition de la cible et les tables de validation de la mosaïque combinée. Sur un réseau **séparé**, elle active la [table de cible d’étalonnage](#the-calibration-target-table) de chaque élément (une par ligne de caméra dans la vue en liste).

### Capture — paramètres de capteur uniformes

Affiché pour les matrices combinées et séparées (l’en-tête du groupe indique « Capture (uniforme sur toutes les caméras) »). Ces paramètres s’appliquent de manière uniforme à chaque élément (des modifications par élément rompraient la synchronisation). Les modifications sont mises en attente et appliquées ensemble via **Appliquer**— à l’exception du bouton**ROI activé / ROI désactivé**, qui s’applique immédiatement.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Commande | Choix / plage | Fonction |
| --- | --- | --- |
| **Format de pixel** | 8 bits / 10 bits / 12 bits / 16 bits | Profondeur de bits uniforme pour tous les membres ; chaque caméra adopte le format correspondant à cette profondeur pour son propre capteur. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning matériel — conserve le champ de vision complet tout en améliorant le rapport signal/bruit et la fréquence d&#x27;images. Sa modification réinitialise les champs de la zone d’intérêt (ROI) en fonction du nouveau champ de vision complet. |
| Préréglage **Résolution** | Pleine / Moitié / Quart | Par rapport au regroupement de pixels ; remplit les champs de la zone d’intérêt (ROI) avec un recadrage centré. |
| **Recadrage ROI (px)**| Bouton bascule**ROI activé / désactivé**, bouton**Sélectionner**, champs L / H / X / Y, carte du capteur complet | Recadrage du capteur — voir [L&#x27;éditeur ROI](#the-roi-editor). La largeur et la hauteur s&#x27;alignent sur des multiples de 16 (minimum 64) ; les décalages s&#x27;alignent sur des multiples de 4. Une indication « max LxH » indique la limite supérieure et**Réinitialiser** ramène les champs au champ de vision complet. Pendant l’édition, le cadre de recadrage orange suit chaque modification de champ sur la carte et sur la tuile du capteur. |
| **Fréquence de déclenchement**| Bouton bascule Auto / Manuel + fps 0,5–10, pas de 0,5 |**Auto**(par défaut) : le backend détermine la fréquence de déclenchement en fonction de la résolution et de la bande passante — le champ de saisie est désactivé et affiche la valeur calculée.**Manuel** : fixe votre valeur lorsque vous cliquez sur « Appliquer ». Une fréquence fixée qu’une nouvelle taille d’image ne peut pas prendre en charge est maintenue à la fréquence la plus rapide autorisée par cette taille, puis revient à la valeur initiale dès que la taille d’image le permet à nouveau. |

Remarque dans le volet : « Les changements de format/résolution redémarrent brièvement toutes les caméras. La fréquence de déclenchement s’applique en temps réel. » Les boutons **Appliquer / Annuler** se trouvent au bas du volet.

#### L’éditeur de ROI

Nouveauté de la version 1.3.0. La ligne de recadrage de la zone d’intérêt (ROI) et la carte située en dessous fonctionnent conjointement :

* **ROI activée / ROI désactivée**— active ou désactive**immédiatement**le recadrage du capteur (les caméras redémarrent brièvement ; la mosaïque affiche « Application des paramètres… »).**Désactivé**lit l’intégralité du capteur et**mémorise votre recadrage**— les champs continuent de l’afficher ;**Activé** l’applique à nouveau. Activer la zone d’intérêt sans recadrage défini applique un recadrage centré de moitié (« Aucun recadrage n’a été défini, un recadrage centré de moitié a donc été appliqué. Ajustez-le à l’aide des champs. »).
* **Sélectionner**— dessine le recadrage sur la carte : faites glisser le curseur dessus ou cliquez sur deux coins opposés (Esc annule). Le rectangle s’affiche dans les champs L/H/X/Y, en s&#x27;alignant sur les étapes ci-dessus ; rien n’est appliqué tant que vous n’avez pas cliqué sur**Appliquer**, ce qui vous permet d’affiner le réglage à l’aide des flèches des champs au préalable. Un rectangle par pression sur Sélectionner.
* **La carte**affiche le recadrage sur l’ensemble du capteur, ce qui vous permet de l’élargir à nouveau : « Capteur complet, en direct » lorsque les caméras lisent le capteur complet, ou « Instantané plein format à {heure}. Les caméras ne lisent que le recadrage. » lorsqu’elles sont recadrées. Si le réseau n’a jamais diffusé que des flux recadrés, la carte indique « Pas encore de vue plein format : ce réseau n’a diffusé que des flux recadrés.» avec un bouton**Afficher le plein format**, qui désactive la zone d’intérêt (ROI). L’étiquette `W×H @ X,Y` du cadrage se trouve à l’extérieur du cadre (au-dessus de celui-ci, ou en dessous lorsque le cadre touche le haut), sur la carte et sur la vignette en direct.
* **Appliquer**valide les champs : un recadrage active la zone d’intérêt ; le plein format (après**Réinitialiser**) la désactive tout en conservant en mémoire le recadrage précédent ; une zone d’intérêt désactivée dont les champs affichent toujours le recadrage mémorisé reste désactivée lorsque vous appliquez une autre modification (format, binning, fréquence).
* **Sur un réseau aligné, la fenêtre de recadrage de chaque caméra est positionnée par l’alignement**, de sorte que chaque caméra cadre la même partie de la scène. Un réseau recadré avant même d’avoir été aligné est d’abord aligné sur l’intégralité du capteur (le recadrage est supprimé, l’alignement effectué, puis le recadrage rétabli avec des fenêtres spécifiques à chaque caméra).
* Après un changement de recadrage, l’exposition automatique se réajuste en fonction du nouveau cadrage. Lors d’un recadrage, l’exposition d’un élément peut encore se prolonger jusqu’à la fin de la période de déclenchement.
* L’état d’activation/désactivation de la zone d’intérêt (ROI) et le cadrage mémorisé sont enregistrés avec le projet.

### Alignement (co-enregistrement)

Affiché pour **les tableaux combinés et séparés** (la version 1.2.0 ne l’affichait que pour les tableaux combinés) ; l’en-tête du groupe indique « Alignement ».

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Bulle **Résiduel** : « RMS x,xx px » — verte en dessous de 1 px, orange en dessous de 3 px, rouge dans les autres cas ou si une caméra a échoué ; « aucun profil » avant la première résolution.
* Ligne de résumé : « ✓ Toutes les caméras alignées (N) » / « ⚠ p/N caméras alignées — <serial (filter)="">échec » / « Recadrage actif — Recalibrer pour aligner (utilise le capteur complet) » / « En attente de la stabilisation de l’exposition… » / « Application des paramètres… » (les caméras redémarrent) / « Le réseau a cessé de fournir des images — reconnecter le réseau ».
* Tableau par caméra : caméra (4 derniers chiffres du numéro de série + filtre), erreur de reprojection en px avec nombre de correspondances (« ref » pour la caméra maître), et score de corrélation croisée normalisé du chevauchement par rapport au seuil de réussite de 0,35.
* **Recalibrer l’alignement** (intitulé « Calibrer l’alignement » avant le premier profil) — relance la co-enregistrement sur de nouvelles images.
* **Case à cocher « Exposition automatique des caméras pour l’alignement »** (cochée par défaut) — éclaircit temporairement les caméras sombres ou sans relief (exposition d’abord, puis gain) afin qu’elles présentent une texture permettant la correspondance, puis rétablit l’exposition automatique.

Chloros aligne automatiquement chaque matrice après la connexion, dans les deux modes d’affichage, une fois que l’exposition automatique des caméras membres s’est stabilisée (le programme continue d’essayer pendant 5 minutes maximum) ; l’aperçu combiné s’aligne également à l’ouverture. Un réseau distinct ne disposant pas de mosaïque composite, cette section interroge l’état d’alignement et lance elle-même la recalibration. Recalibrez si la mise au point ou la profondeur de la scène a changé. L’alignement est **par conception limité à la session** — il n’est jamais enregistré dans un profil, car il dépend de la distance de la scène à ce moment-là. Les captures peuvent tout de même être exportées avec un alignement pixel par pixel (voir [Exportations alignées](capture.md#per-array-controls)).

### Vignettage intelligent

* Case à cocher **Activer la correction**— applique l’estimation du vignettage par caméra à la chaîne radiométrique (en temps réel**et** lors des exportations).
* **Calibrer à partir de la vue actuelle**— pointez d’abord le réseau vers une cible uniforme (écran plat, mur ou ciel) ; chaque caméra est aplanie individuellement et l’état indique un gain d’aplanissement de « n/N caméras · −x,x % ».**Effacer** supprime l’estimation.
* Statut : « Calibré », « Non calibré » ou « Partiellement calibré — relancer » lorsque seules les estimations de certains membres s’appliquent encore à leur cadrage actuel (par exemple après qu’une opération de recalibrage a déplacé la fenêtre de recadrage d’un membre) — relancez la calibration.
* Affinez le réglage pour chaque caméra à l’aide du curseur **Vignette** propre à chaque caméra dans [Aperçu en direct](#live-preview).

### Aperçu en direct

* Case à cocher **Histogramme**+ curseur de largeur (0,10–0,90 de l’image) — la bande d’histogramme en direct sur toutes les tuiles de chaque élément à la fois (tuiles à canaux séparés incluses), la [Superposition d’histogramme](#display-overlays-drawn-over-the-live-feed) définie pour l’ensemble du réseau. Elle s’affiche comme activée lorsque tous les membres en disposent déjà. Sur un réseau**distinct**, il s’agit de la seule ligne du groupe.
* *Combiné uniquement :* **Index**: case à cocher « Activer » + roue dentée — ouvre le [Calculateur d’index](#index-calculator-pane) partagé avec des bandes tracées à partir de**toutes** les caméras membres. Une ligne d’aperçu de l’expression située en dessous affiche l’expression actuelle (« Aucune expression définie — ouvrez le calculateur pour en créer une »), actualisée toutes les secondes.
* *Combiné uniquement :* menu déroulant **Résolution de rendu**(mêmes préréglages que par caméra, 720p par défaut) : la hauteur du flux de prévisualisation en direct**et** la taille d’exportation du composite enregistré. Remarque dans le volet : « Aperçu + taille du composite enregistré. Les images par caméra sont toujours exportées en pleine résolution. »

### Calques d’affichage *(mode combiné uniquement)** Case à cocher **Activer** (désactivée par défaut — la caméra principale s’affiche directement ; activée = composite en calques).
* Menus déroulants **Menus déroulants**Premier plan**/**Arrière-plan**: chaque caméra membre (par nom) ou**Index**. Lorsque le premier plan est réglé sur « Index », les pixels situés en dehors des limites min./max. de la LUT affichent la couche d’arrière-plan.

### Vue fractionnée *(uniquement en mode combiné)*

**« Afficher les caméras membres »**— un bouton**Diviser / Masquer les caméras membres** qui ajoute le flux en direct de chaque caméra membre sous forme de vignettes distinctes dans la grille, à côté de la composition. Les vignettes lisent le tampon d’images existant du réseau (aucune connexion supplémentaire à la caméra n’est nécessaire). Affichage en grille uniquement ; enregistré par réseau avec le projet.

### Fonctionnalités

Un panneau en lecture seule actualisé toutes les 5 s :

* **Libellé du niveau** : « Capture simultanée » (vert) · « Capture simultanée (émission décalée FTD) » (vert) · « Capture décalée (décalage de 100 ms) » (orange) · « Configuration trop volumineuse » (rouge).
* **État de la trame** : « x,xx % incomplète » — vert en dessous de 1 %, orange en dessous de 5 %, rouge à partir de 5 %.
* **Ligne de liaison** : « NIC {mbps} Mbps - débit soutenu {MB/s} MB/s ».

Il s&#x27;agit du budget de bande passante en temps réel du réseau de caméras. Pour en savoir plus sur le modèle de réseau et le nombre d’images par seconde (fps) sous-jacent — ainsi que sur les modifications à apporter lorsque le niveau passe à l’orange ou au rouge —, consultez [Réseaux multicaméras](arrays.md) et la [Référence CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Volet « Calculateur d’indice »

Troisième page de la barre latérale, commune à l’engrenage « Index » par caméra et à l’engrenage « Index » de la matrice combinée (un à la fois — l’en-tête indique « Calculateur d’indice — <camera name="">» ou « Calculateur d’indice — <array name="">»). Elle reçoit la liste des bandes (les bandes de filtresou toutes les bandes de l’ensemble), l’expression actuelle et la configuration de la LUT (activée/désactivée, niveau — par défaut 7 stops, min — par défaut 0,2, max — par défaut 1), ainsi qu’un histogramme d’indice en temps réel. Le bouton **Appliquer** valide l’expression ; les modifications de la table de conversion s’appliquent en temps réel à l’aperçu.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Paramètres par caméra vs paramètres gérés par le réseau

Référence rapide indiquant où se trouvent les paramètres lorsqu’une caméra fait partie d’un réseau :

| Géré par le réseau (en lecture seule dans le volet de la caméra) | Toujours par caméra au sein d’un réseau |
| --- | --- |
| Format des pixels, résolution, regroupement de pixels | Exposition automatique (exposition, gain, cible, lissage, ROI) |
| Mode/source de déclenchement, fréquence d’images | Débruitage, netteté, vignettage |
| | Orientation (miroir/rotation), superpositions d’affichage (paramètres du réseau → « Aperçu en direct » définit l’histogramme sur tous les membres à la fois), posemètre ponctuel |
| | Index (réseaux à affichage séparé), liaison au capteur de lumière |

Autres comportements transversaux :

* **Affichage combiné ou séparé** : le choix s’effectue lors de la connexion du réseau de caméras : combiné = une seule mosaïque composite alignée (les flux des caméras ne sont visibles que via la vue fractionnée) ; séparé = chaque caméra affiche sa propre mosaïque synchronisée. Une caméra n’affiche jamais à la fois un flux autonome et une mosaïque du réseau.
* ****Reconnexion automatique** : l’ouverture d’un projet enregistré restaure ses caméras et ses matrices et réapplique tous les paramètres enregistrés au backend avant la reprise des flux.
* **Filtrage de la capture** : les caméras masquées ou mises en pause sont exclues de la fonction « Capture All » ; une matrice n’est entièrement bloquée que lorsque TOUS ses membres sont masqués ou mis en pause. Voir [Paramètres et modes de capture](capture.md).

## Persistance des paramètres

L’état de l’onglet « Caméra » est enregistré **avec le projet**, et non dans le navigateur :

* Chaque modification réactive crée un instantané des caméras et des réseaux dans le fichier `cameras.json` du projet (débouncé à 500 ms). Cela inclut les noms et couleurs des caméras, les réglages d’exposition/gain/AE, le format de pixel/résolution/binning, la fréquence de déclenchement, les paramètres de prévisualisation (résolution de rendu, débruitage, netteté, vignettage, profil de couleur, hautes lumières, saturation/contraste), l’orientation, les superpositions, les divisions de canaux, la configuration de l’index, les réglages d’AE prédictif, la zone d’intérêt (ROI) pour l’AE, les noms de matrices, le mode d’affichage, les paramètres de capture des matrices (y compris le recadrage de la zone d’intérêt, l’activation ou non de cette zone, et le recadrage mémorisé lorsqu’elle est désactivée), le « Wire Budget » de la matrice et le bloc de grille (zoom sur le flux, mode d’affichage, verrouillage de la grille, ordre manuel des tuiles, caméras masquées, tuiles fermées, caméra active).
* Les associations de capteurs de lumière sont enregistrées dans le fichier `sensors.json` du projet.
* La réouverture du projet reconnecte le matériel et réapplique l’ensemble de ces paramètres.
* **Aucun projet ouvert = session uniquement** : en l’absence de projet, aucun paramètre n’est conservé après la fermeture d’Chloros.
* Exclusivement lié à la session, quel que soit le projet : l’état de pause, les échantillons du posemètre ponctuel, la case à cocher « Cible d’étalonnage » par caméra (toujours désactivée par défaut) et le profil d’alignement du réseau (recalculé à chaque session par défaut).
* Une exception : les sélections d’exportation des **paramètres de capture** et le mode de capture sont conservés par projet dans le stockage local de l’application plutôt que dans `cameras.json` — voir [Paramètres et modes de capture](capture.md).</array></camera></serial>
