# L&#x27;onglet DAQ dans Chloros

L&#x27;onglet DAQ — intitulé **Capteurs de lumière** dans la barre latérale de Chloros — est la surface de contrôle en temps réel des [capteurs de lumière DAQ-U, DAQ-M et DAQ-E](README.md) : connectez les capteurs via n&#x27;importe quel protocole de transport, observez les spectres étalonnés en temps réel, calculez la réflectance en direct à partir d&#x27;une paire de capteurs et enregistrez des fichiers `.daq` directement dans votre projet.

L&#x27;onglet devient disponible une fois que le backend Chloros a fini de démarrer. Les graphiques de l’onglet sont alimentés par le service DAQ de Chloros via une connexion en direct qui se reconnecte automatiquement (délai de 2 à 10 s) en cas d’interruption ; lorsque le service est inaccessible, la ligne « Statut » d’un capteur affiche **« Aucun serveur »**.

La mise en page se compose d’une **barre latérale des capteurs**(une ligne par capteur connecté) et d’une**zone de graphiques** (une tuile de graphique par capteur ou par groupe).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Connexion d’un capteur

Cliquez sur **Connecter un capteur** en haut de la barre latérale. La boîte de dialogue de connexion s’ouvre dans la zone principale (ou sous forme de fenêtre superposée lors de l’ajout d’un autre capteur — un bouton Annuler apparaît dans ce cas).

| Commande | Comportement |
| --- | --- |
| **Type d’appareil** | `DAQ-U (USB)` (par défaut), `DAQ-M (Bluetooth)` ou `DAQ-E (Ethernet)`. Le changement de type relance la recherche du nouveau protocole de transport sélectionné. |
| **Port / Appareil BLE / Nom d’hôte / IP** | Répertorie les appareils détectés sous la forme `device - description` ; la première entrée reconnue comme capteur est automatiquement sélectionnée. Pendant la recherche, l’écran affiche `Scanning...` (USB), `Scanning (N)...` avec un compte à rebours de 8 secondes (BLE) ou `Discovering ethernet sensors (N)...` avec un compte à rebours de 5 secondes (Ethernet). Les résultats vides s’affichent sous la forme `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Actualiser** | Relance immédiatement la recherche du transport sélectionné (désactivé en cours de recherche BLE/Ethernet). |
| **Se connecter** | Activé une fois qu&#x27;un appareil est sélectionné ; le libellé passe à `Connecting...` pendant l&#x27;établissement de la connexion. |

La détection ne s&#x27;exécute que **tant que la boîte de dialogue de connexion est affichée à l&#x27;écran**, et se répète toutes les 15 secondes pour le transport sélectionné uniquement — le simple fait d&#x27;ouvrir l&#x27;onglet ne déclenche pas la recherche. En cas d&#x27;échec, la boîte de dialogue affiche : *« Échec de la connexion. Essayez de débrancher puis de rebrancher le capteur, puis cliquez à nouveau sur Se connecter. »*

La barre latérale s’ouvre automatiquement lorsque votre premier capteur se connecte.

{% hint style="info" %}
**Le DAQ-E n&#x27;apparaît pas ?** Le DAQ-E ne dispose pas de LED d&#x27;état — vérifiez le voyant PoE/liaison sur le commutateur ou le port de l&#x27;injecteur auquel il est branché, et patientez quelques secondes après la mise sous tension pour qu&#x27;il démarre. L&#x27;appareil Chloros doit se trouver sur le même domaine de diffusion (le mDNS ne traverse pas les routeurs). Sur Windows, acceptez l’invite du pare-feu Defender la première fois que Chloros lie ses sockets de multidiffusion (mDNS UDP 5353, données DAQ-E UDP 5002, PTP UDP 319/320). Deux unités DAQ-E sur un même réseau local sont détectées séparément, chacune sous son propre nom d’hôte `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Le type de périphérique propose DAQ-U (USB), DAQ-M (Bluetooth) et DAQ-E (Ethernet)</figcaption></figure>***

## La barre latérale des capteurs

Chaque capteur connecté dispose d’une ligne (plus une ligne par groupe « Ambient+Object »). Les lignes peuvent être réorganisées par glisser-déposer, et leur ordre détermine également l’ordre des vignettes du graphique. Cliquez sur une ligne pour faire de ce capteur ou de ce groupe le graphique actif dans la vue en liste.

| Élément | Signification |
| --- | --- |
| Bordure gauche colorée | La couleur du graphique du capteur. |
| Badge de transport | `DAQ-U` / `DAQ-M` / `DAQ-E`, ou un badge vert `REF` pour un groupe de réflectance « Ambient+Object ». |
| Nom de l’appareil | Par défaut : le numéro de série du capteur (son identifiant fixe pour l’étalonnage, les noms de fichiers `.daq` et la correspondance lors de l’importation) ; les noms personnalisés sont conservés pour chaque projet. |
| Bouton **Étalonné** (vert) | S&#x27;affiche lorsque le pack d&#x27;étalonnage d&#x27;usine du capteur est chargé, c&#x27;est-à-dire lorsque les spectres sont exprimés en W/m²/nm. |
| Icône **Mise à jour disponible** (orange, DAQ-E uniquement) | Le micrologiciel en cours d’exécution est plus ancien que l’image fournie avec cette version Chloros. Pendant une mise à jour, elle affiche la progression en temps réel (`Flashing… N%`, `Restarting sensor…`, puis `Updated X → Y` ou `Failed`). |
| Œil | Active ou désactive l&#x27;affichage de ce capteur sur son graphique. |
| Roue dentée | Ouvre la fenêtre modale des paramètres propres au capteur (ci-dessous). |
| ✕ (rouge) | Déconnecte le capteur ou supprime un groupe « Ambient+Object ». |

Au-dessus des lignes se trouvent deux boutons :

* **Connecter le capteur** — ouvre la boîte de dialogue de connexion (son libellé passe à « `Connecting...` » pendant l&#x27;opération).
* **Tout enregistrer / Tout arrêter**— lance ou arrête un enregistrement `.daq` sur**tous les**capteurs connectés. Nécessite au moins un capteur**et un projet ouvert** (info-bulle : « Ouvrez un projet pour enregistrer ») ; il devient rouge tant qu’un enregistrement est en cours.

Lorsqu’il est vide, le message « Aucun capteur connecté » s’affiche.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Paramètres par capteur (fenêtre modale en forme d’engrenage)

Ouvrez-la à l’aide de l’icône en forme d’engrenage située sur la ligne d’un capteur. Contenu par ordre :

* **Lignes d’informations** — Type d’appareil (DAQ-U/M/E), Connexion (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (port COM, adresse BLE ou hôte) et numéro de série.
* **Rapport d’étalonnage : Télécharger** — récupère le certificat d’étalonnage traçable NIST (PDF) de cet appareil et l’ouvre dans votre lecteur PDF. Disponible une fois le numéro de série connu ; le certificat est mis en cache lors de la première connexion.
* **Nom de l’appareil** — cliquez sur le crayon pour renommer ; cette valeur est conservée par projet.
* **Couleur de la ligne du graphique** — nuancier ; reste inchangée pour chaque projet.
* **Temps d’intégration (ms)**— curseur + valeur numérique,**1 à 500 ms**, valeur par défaut**32 ms**. Désactivé lorsque l’exposition automatique (AE) est activée.
* **Moyenne d’images**— curseur + nombre,**1 à 50 images**, valeur par défaut**20**.
* **AE : ON/OFF**— bouton de basculement de l’exposition automatique ;**activée par défaut** à la connexion. Désactivez-la pour régler manuellement le temps d’intégration.
* **Arrêter la diffusion / Démarrer la diffusion** — met en pause ou reprend la diffusion en direct.
* **Enregistrer / Arrêter l’enregistrement** — enregistrement `.daq` par capteur (nécessite un projet ouvert).
* **Cap** — le profil de correction de la couronne (section suivante).
* **Lignes d’informations en direct** — Temps d’intégration (ms), FPS, Échantillons, Enregistrement (rouge `REC` ou `Off`) et État (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### DAQ-E uniquement : lignes « Réseau », « Micrologiciel » et « PTP »

* **Nom d’hôte / IP** — l’adresse actuelle de l’appareil.
* **Micrologiciel** — version actuelle du micrologiciel, accompagnée d’une cellule d’action : un<version\>

bouton</version\>

**Mettre à jour vers \<version\>** apparaît lorsque cette version Chloros intègre une image de micrologiciel DAQ-E plus récente. La mise à jour s’effectue via le réseau en environ 30 secondes ; le capteur redémarre et se reconnecte automatiquement, et un transfert interrompu laisse le micrologiciel actuel intact. La progression s’affiche en temps réel (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), et la cellule indique `Up to date` lorsqu’elle est à jour.
* **Synchronisation PTP** — l’état PTP en temps réel (revient à `unknown`). Le micrologiciel DAQ-E v1.2.0+ participe au protocole IEEE 1588 PTPv2 en tant qu’horloge esclave uniquement ; le backend de l’hôte Chloros est le grand maître PTP, et toutes les caméras DAQ-E et LATTICE du réseau local lui sont asservies dans le domaine 0, en maintenant les horodatages à environ 1 ms près.

Pour un groupe « Ambient+Object », la fenêtre modale « Gear » n’affiche que les capteurs sources du groupe, le nom de l’appareil et la couleur de la ligne du graphique.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Sélection du capuchon

Le menu déroulant **Cap** indique à Chloros quel capuchon physique est monté sur le diffuseur du capteur, et applique à chaque spectre le profil de correction mesuré en usine pour ce capuchon. Les choix dépendent du modèle :

| Modèle | Choix de capuchons |
| --- | --- |
| DAQ-U | Aucun (capteur nu), champ de vision (FOV) 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (correcteur cosinus) |
| DAQ-M | Aucun (capteur nu), Sunshine (correcteur cosinus) |
| DAQ-E | Aucun (capteur nu), champ de vision (FOV) 15°, champ de vision (FOV) 45°, champ de vision (FOV) 90°, Sunshine (correcteur cosinus) |

**La configuration par défaut pour tous les modèles est « Sunshine » (correcteur de cosinus)** — MAPIR livre chaque DAQ avec le capuchon « Sunshine » installé ; il s&#x27;agit de la configuration standard pour une utilisation en extérieur : une vue hémisphérique à 180° avec une erreur de cosinus ≤ ±4 % jusqu’à 60° et ≤ ±4,5 % jusqu’à 70° (non recommandé en dessous d’une élévation solaire d’environ 15°), avec une atténuation intrinsèque (~12×). Votre sélection est conservée dans le projet.

{% hint style="warning" %}
**Le choix du capuchon doit correspondre au capuchon physique.**Ni le capteur ni le logiciel ne peuvent détecter quel capuchon est installé. Cette sélection détermine à la fois la correction en temps réel et la mention inscrite dans chaque fichier `.daq` — compte tenu de l&#x27;atténuation d&#x27;environ 12× du capuchon Sunshine, un changement de capuchon non signalé entraîne une correction erronée des spectres d&#x27;un facteur proche de cette valeur. (Le retrait puis la remise en place du même capuchon entraînent une répétition d’environ 1,5 %.) Ne sélectionnez**Aucun (capteur nu)** que lorsque le capuchon est physiquement retiré ; sur un DAQ-E, « Aucun » applique tout de même un profil géométrique d&#x27;usine pour son diffuseur en verre encastré — ce n&#x27;est pas une opération nulle — et un DAQ-E nu correspond à une configuration de laboratoire, et non à une configuration de terrain prise en charge.
{% endhint %}

{% hint style="info" %}
Mise à jour par rapport à un manuel antérieur : le bouton bascule « Diffuseur Sunshine installé » côté navigateur, présent dans la version 1.1.0, a disparu. La gestion du capuchon s’effectue désormais via ce profil de capuchon par capteur, appliqué côté serveur.
{% endhint %}

***

## La zone des graphiques

Une barre supérieure fixe comporte un **bouton de basculement entre la vue liste et la vue en grille**ainsi qu’un curseur de**zoom sur le graphique** (taille des tuiles : 200 à 2 000 px). L&#x27;affichage bascule automatiquement en mode grille lorsqu&#x27;il existe plusieurs groupes de graphiques, et revient en mode liste lorsqu&#x27;il n&#x27;y en a qu&#x27;un seul ou moins. Le mode d&#x27;affichage et la taille des graphiques sont conservés pour chaque projet.

Le **graphique de spectre** de chaque capteur affiche :

* **Axe X** — Longueur d&#x27;onde (nm). La grille du capteur s’étend de 340 à 1 010 nm par pas de 5 nm (135 points), interpolée à 1 nm pour l’affichage.
* **Axe Y** — Puissance (W/m²), avec un préfixe SI automatique (m/µ/n) choisi en fonction du pic. Les spectres sont exprimés en irradiance spectrale (W/m²/nm) calibrée radiométriquement sur les trois modes de transport.
* Un remplissage spectral arc-en-ciel sous une trace unique ; plusieurs capteurs sur un même graphique se superposent sous forme de lignes colorées avec des remplissages estompés.
* **Survol**— un curseur vertical indiquant la longueur d’onde et la valeur par capteur ;**glisser** pour zoomer (un bouton de dézoom apparaît lorsque le zoom est activé).
* Un bouton **+** (en mode grille uniquement) pour ajouter un capteur à ce graphique ou créer un groupe (ci-dessous).
* Le nom de l’appareil centré en haut, et un indicateur de chargement jusqu’à l’arrivée de la première image.

La **saturation** n&#x27;est pas indiquée sur le graphique lui-même : un capteur saturé affiche un texte d&#x27;état rouge `SATURATED` et une ligne rouge `Saturated: Yes` dans le tableau des données en temps réel. Réduisez le temps d&#x27;intégration ou réactivez l&#x27;AE pour le désactiver.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tableau des données en temps réel (vue en liste)

Sous le graphique, en vue en liste, actualisé toutes les 500 ms :

* **Tous les modèles** : Échantillon de couleur de la lumière (sRGB à partir de CIE XYZ), Saturé (Oui/Non), CIE 1931 X/Y/Z, Chromaticité x/y, CIE u′/v′, CCT (K), IRC (Ra), Longueur d’onde dominante (nm), longueur d’onde de crête (nm), pureté d’excitation, Duv, CIE L\*/a\*/b\* et Munsell H/V/C.
* **Capteurs étalonnés uniquement**(n&#x27;importe quel modèle DAQ-U / DAQ-M / DAQ-E une fois son kit d&#x27;étalonnage d&#x27;usine chargé — le badge vert**Étalonné** sur la ligne du capteur en est la preuve) : Puissance totale (W/m²), lux photopique (lx), lux scotopique (lx), rapport S/P, PPFD et PPFD Red/Green/Blue (µmol/m²/s), ainsi que les irradiances opiques — cône S, mélanopique, rhodopique, cône M, cône L (toutes en W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Groupes de réflectance (Ambiante + Objet)

Deux capteurs connectés peuvent être combinés pour afficher la réflectance en temps réel — sans caméra :

1. En mode grille, cliquez sur **+**sur une tuile de graphique et sélectionnez**Combiner ambiant + objet**.
2. Sélectionnez un capteur **Source de lumière ambiante**et un capteur**Scanner d’objet**(deux capteurs distincts), puis cliquez sur**Créer**.

Chloros calcule R(λ) = objet(λ) / ambiant(λ) par longueur d&#x27;onde à partir des deux flux en temps réel (0 lorsque l’ambiant est ≤ 0). Le libellé du groupe dépend de la classe d’étalonnage des capteurs :

* Les deux capteurs sont étalonnés (ensemble chargé) → **« Réflectance apparente »**.
* L’un des capteurs n’est pas étalonné → **« Réflectance relative »**.

Le groupe apparaît sous la forme d’une ligne verte `REF` dans la barre latérale et dispose de son propre graphique (fond arc-en-ciel, valeurs affichées au survol avec 4 décimales, zoom par glissement).

Le menu **+**propose également**Ajouter un nouveau capteur** avec trois options de placement : *Combiner le nouveau capteur* (l&#x27;ajouter à ce graphique), *Déplacer un capteur existant ici* ou *Afficher le nouveau capteur* (dans son propre graphique).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tableau des indices de végétation

En vue liste, un tableau des indices de végétation se trouve sous le graphique d’un groupe de réflectance ; ces indices sont calculés à partir de la réflectance en temps réel aux centres de bande **bleu 450 / vert 550 / rouge 670 / NIR 800 nm** (valeurs à 4 décimales, `---` lorsque le calcul est impossible ; survolez le nom d’un indice pour afficher son nom complet) :

* **Toujours affichés** (invariants d&#x27;échelle, quelle que soit la combinaison de capteurs) : NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Uniquement lorsque les deux capteurs sont étalonnés en puissance** (les deux ensembles chargés) : EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## Enregistrement des fichiers `.daq`

* L&#x27;enregistrement nécessite un **projet ouvert** — sinon, les boutons « Enregistrer tout » (barre latérale) et « Enregistrer » propres à chaque capteur sont désactivés.
* Les fichiers sont enregistrés au format **`<project folder>/light_sensor/`** ; les noms de fichiers comportent l’identifiant du capteur et un horodatage, et le nom de l’appareil est stocké avec l’enregistrement.
* Lorsqu’un enregistrement s’arrête (via « Arrêter », « Tout arrêter » ou une déconnexion en cours d’enregistrement), le fichier `.daq` finalisé est **ajouté automatiquement au projet ouvert** — il apparaît dans la liste des fichiers du projet sans ajout manuel, prêt à servir de données d’irradiation descendante pour le [traitement de la réflectance](README.md).
* Un indicateur rouge `REC` s’affiche dans les lignes en temps réel de la fenêtre contextuelle des paramètres pendant l’enregistrement.

Pour obtenir des valeurs quantitatives d’irradiance, effectuez une moyenne sur au moins 15 secondes de données — il s’agit d’une caractéristique de l’instrument, et non d’un défaut.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Disposition multi-capteurs et persistance des projets

* Combinez plusieurs capteurs sur un même graphique (axes partagés), conservez des graphiques distincts (mise en page en grille automatique), déplacez les capteurs d&#x27;un graphique à l&#x27;autre, réorganisez les lignes/tuiles par glisser-déposer et masquez des capteurs individuels à l&#x27;aide du bouton « œil ».
* Pour chaque projet, Chloros conserve : les noms des appareils, les couleurs des graphiques, la taille des graphiques, le mode d’affichage et les paramètres de chaque capteur (temps d’intégration, moyennage des trames, état AE, sélection du plafond).
* **La réouverture d’un projet reconnecte automatiquement ses capteurs** par adresse — port COM pour DAQ-U, périphérique BLE pour DAQ-M, nom d’hôte mDNS pour DAQ-E (résolu même si l’adresse IP de l’appareil a changé) — et réapplique le profil de capteur enregistré pour chaque capteur, ainsi que le moyennage des images, l’état AE et le temps d’intégration manuel.***

## Appairage de la caméra (DLS)

Il n’y a rien à appairer. Contrairement aux flux de travail DLS pour drones qui associent d’emblée un capteur de lumière à une caméra, Chloros associe les données DAQ aux images en aval : au moment de l’importation/du traitement, les relevés `.daq` sont interpolés à l’horodatage d’exposition de chaque capture. Enregistrez avec n’importe quel capteur connecté (le `.daq` s’ajoute automatiquement au projet), et le traitement de la réflectance détermine les mesures correctes en fonction du temps — consultez [Capteurs de lumière DAQ](README.md) pour savoir comment les données de rayonnement descendant sont utilisées.</version\>
