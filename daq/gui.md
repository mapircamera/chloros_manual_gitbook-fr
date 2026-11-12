# L'onglet « DAQ » dans Chloros

L'onglet « DAQ » — intitulé **Capteurs de lumière** dans la barre latérale de Chloros — constitue l'interface de contrôle en temps réel des [capteurs de lumière DAQ-U, DAQ-M et DAQ-E](README.md) : connectez des capteurs via n'importe quel protocole de transport, observez les spectres étalonnés en temps réel, calculez la réflectance en direct à partir d'une paire de capteurs et enregistrez des fichiers `.daq` directement dans votre projet.

Cet onglet devient disponible dès que le backend d’Chloros a fini de se lancer. Les graphiques de l’onglet sont alimentés par le service DAQ d’Chloros via une connexion en direct qui se reconnecte automatiquement (délai de 2 à 10 s) en cas d’interruption ; tant que le service est inaccessible, la ligne « État » d’un capteur affiche **Aucun serveur**.

La présentation se compose d’une **barre latérale des capteurs**(une ligne par capteur connecté) et d’une**zone de graphiques** (une tuile de graphique par capteur ou groupe).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Connexion d’un capteur

Cliquez sur **Connecter un capteur** en haut de la barre latérale. La boîte de dialogue de connexion s’ouvre dans la zone principale (ou sous forme de fenêtre superposée lors de l’ajout d’un autre capteur — un bouton Annuler apparaît dans ce cas).

| Commande | Comportement |
| --- | --- |
| **Type d’appareil** | `DAQ-U (USB)` (par défaut), `DAQ-M (Bluetooth)` ou `DAQ-E (Ethernet)`. Le changement de type relance la recherche du nouveau transport sélectionné. |
| **Port / Appareil BLE / Nom d’hôte / IP** | Répertorie les appareils détectés sous la forme `device - description` ; la première entrée reconnue comme capteur est sélectionnée automatiquement. Pendant la recherche, l’écran affiche `Scanning...` (USB), `Scanning (N)...` avec un compte à rebours de 8 secondes (BLE) ou `Discovering ethernet sensors (N)...` avec un compte à rebours de 5 secondes (Ethernet). Les résultats vides s’affichent sous la forme `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Actualiser** | Relance immédiatement la recherche du mode de transport sélectionné (désactivé en cours de recherche BLE/Ethernet). |
| **Connecter le capteur** | Activé une fois qu’un appareil est sélectionné ; le nom s’affiche alors sous la forme `Connecting...` pendant l’établissement de la connexion. |

La détection ne s'exécute que **lorsque la boîte de dialogue de connexion est affichée à l'écran** et se répète toutes les 15 secondes pour le transport sélectionné uniquement — le simple fait d'ouvrir l'onglet ne lance pas la recherche. En cas d'échec, la boîte de dialogue affiche : *« Échec de la connexion. Essayez de débrancher puis de rebrancher le capteur, puis cliquez à nouveau sur Connecter. »*

La barre latérale s'ouvre automatiquement lorsque votre premier capteur se connecte.

{% hint style="info" %}
**Le DAQ-E n'apparaît pas ?**Le DAQ-E ne dispose pas de LED d'état — vérifiez le voyant PoE/liaison sur le commutateur ou le port de l'injecteur auquel il est branché, et patientez quelques secondes après la mise sous tension pour qu'il démarre. La machine Chloros doit se trouver sur le même domaine de diffusion (le mDNS ne traverse pas les routeurs). La règle de pare-feu**« Chloros Backend (UDP-In) »** du programme d’installation autorise le mDNS (UDP 5353), les données du DAQ-E (UDP 5002) et le PTP (UDP 319/320) ; un pare-feu tiers doit autoriser les mêmes ports. Deux unités DAQ-E sur un même réseau local sont détectées séparément, chacune sous son propre nom d’hôte `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Le type de périphérique propose les options DAQ-U (USB), DAQ-M (Bluetooth) et DAQ-E (Ethernet)</figcaption></figure>

***

## La barre latérale des capteurs

Chaque capteur connecté dispose d’une ligne (plus une ligne par groupe « Ambient+Object »). Les lignes peuvent être réorganisées par glisser-déposer, et leur ordre redéfinit également l’ordre des vignettes du graphique. Cliquez sur une ligne pour faire de ce capteur ou de ce groupe le graphique actif dans la vue en liste.

| Élément | Signification |
| --- | --- |
| Bordure gauche colorée | La couleur du graphique du capteur. |
| Badge de transport | `DAQ-U` / `DAQ-M` / `DAQ-E`, ou un badge vert `REF` pour un groupe de réflectance « Ambient+Object ». |
| Nom de l’appareil | Par défaut, il s’agit du numéro de série du capteur (son identifiant stable pour l’étalonnage, les noms de fichiers `.daq` et la correspondance lors de l’importation) ; les noms personnalisés sont conservés pour chaque projet. |
| Icône **Étalonné** (verte) | S'affiche lorsque le pack d'étalonnage d'usine du capteur est chargé, c'est-à-dire que les spectres sont exprimés en W/m²/nm. |
| Icône **Mise à jour disponible** (orange, DAQ-E uniquement) | Le micrologiciel en cours d’exécution est plus ancien que l’image fournie avec cette version d’Chloros. Pendant une mise à jour, elle affiche la progression en temps réel (`Flashing… N%`, `Restarting sensor…`, puis `Updated X → Y` ou `Failed`). |
| Œil | Active ou désactive l’affichage de ce capteur sur son graphique. |
| Roue dentée | Ouvre la fenêtre modale des paramètres propres à chaque capteur (ci-dessous). |
| ✕ (rouge) | Déconnecte le capteur ou supprime un groupe « Ambient+Object ». |

Au-dessus des lignes se trouvent deux boutons :

* **Connecter le capteur** — ouvre la boîte de dialogue de connexion (son libellé devient `Connecting...` lorsqu’il est occupé).
* **Tout enregistrer / Tout arrêter**— lance ou arrête un enregistrement `.daq` sur**tous les**capteurs connectés. Nécessite au moins un capteur**et un projet ouvert** (info-bulle : « Ouvrez un projet pour enregistrer ») ; il devient rouge tant qu’un enregistrement est en cours.

À l’état vide, il affiche « Aucun capteur connecté ».

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Paramètres par capteur (fenêtre modale en forme d’engrenage)

Ouvrez-la à l’aide de l’icône en forme de roue dentée sur une ligne de capteur. Contenu par ordre :

* **Lignes d’informations** — Type d’appareil (DAQ-U/M/E), Connexion (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (port COM, adresse BLE ou hôte) et numéro de série.
* **Rapport d’étalonnage : Télécharger** — récupère le certificat d’étalonnage traçable NIST de cet appareil (PDF) et l’ouvre dans votre lecteur PDF. Disponible une fois le numéro de série connu ; le certificat est mis en cache lors de la première connexion.
* **Nom de l’appareil** — cliquez sur le crayon pour le renommer ; ce nom est conservé pour chaque projet.
* **Couleur de la ligne du graphique** — nuancier ; cette couleur est conservée pour chaque projet.
* **Temps d’intégration (ms)**— curseur + nombre,**1 à 500 ms**, valeur par défaut**32 ms**. Désactivé lorsque l’exposition automatique (AE) est activée.
* **Moyenne des images**— curseur + nombre,**1 à 50 images**, valeur par défaut**20**.
* **AE : activé/désactivé**— bouton de basculement de l’exposition automatique ;**activé par défaut** à la connexion. Désactivez-le pour régler manuellement le temps d’intégration.
* **Arrêter la diffusion / Démarrer la diffusion** — met en pause ou reprend la diffusion en direct.
* **Enregistrer / Arrêter l’enregistrement** — enregistrement `.daq` par capteur (nécessite un projet ouvert).
* **Cap** — le profil de correction « cap » (section suivante).
* **Lignes d’informations en direct** — Temps d’intégration (ms), FPS, Échantillons, Enregistrement (rouge `REC` ou `Off`) et État (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### DAQ-E uniquement : lignes « Réseau », « Micrologiciel » et « PTP »

* **Nom d’hôte / IP** — l’adresse actuelle de l’appareil.
* **Micrologiciel** — version actuelle du micrologiciel, accompagnée d’une cellule d’action : un bouton **Mettre à jour vers \<version\>** apparaît lorsque cette version d’Chloros intègre une image de micrologiciel DAQ-E plus récente. La mise à jour s’effectue via le réseau en environ 30 secondes ; le capteur redémarre et se reconnecte automatiquement, et une interruption du transfert laisse le micrologiciel actuel intact. La progression s’affiche en temps réel (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), et la cellule indique `Up to date` lorsqu’elle est à jour.
* **Synchronisation PTP** — l’état PTP en temps réel (revient à `unknown`). Le micrologiciel DAQ-E v1.2.0+ participe au protocole IEEE 1588 PTPv2 en tant qu’horloge esclave uniquement ; le backend de l’hôte Chloros est le grand maître PTP, et chaque caméra DAQ-E et LATTICE sur le réseau local lui est asservie dans le domaine 0, ce qui permet de maintenir les horodatages à environ 1 ms près.
* **Réseau**— l’adresse sous laquelle l’appareil fonctionne et la manière dont il l’a obtenue (DHCP, locale de liaison ou statique), ainsi qu’un paramètre enregistré qui s’applique après un redémarrage.**Modifier…**ouvre un formulaire :**Automatique (DHCP)**ou**IP statique**avec une adresse IP, un masque de sous-réseau et une passerelle facultative.**Appliquer et redémarrer** enregistre les modifications, redémarre le capteur et le reconnecte. Une adresse statique en dehors du réseau de cet ordinateur est refusée, car Chloros ne pourrait pas atteindre le capteur après le redémarrage ; une boîte de dialogue permettant de l’autoriser quand même apparaît alors, pour un ordinateur qui sera réadressé ou acheminé vers ce réseau. **Diffusion à la mise sous tension**permet de définir si le capteur commence à diffuser dès qu’il est mis sous tension, avant que tout logiciel ne se connecte.**Redémarrer le capteur** le redémarre et le reconnecte avec ses paramètres actuels. Un redémarrage est refusé tant que le capteur est en cours d’enregistrement. Nouveauté de la version 1.3.0.

Pour un groupe « Ambient+Object », la fenêtre modale en forme d’engrenage affiche uniquement les capteurs sources du groupe, le nom de l’appareil et la couleur de la ligne du graphique.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Sélection du capuchon

Le menu déroulant **Cap** indique à Chloros quel capuchon physique est monté sur le diffuseur du capteur, et applique à chaque spectre le profil de correction mesuré en usine pour ce capuchon. Les choix dépendent du modèle :

| Modèle | Choix de capuchons |
| --- | --- |
| DAQ-U | Aucun (capteur nu), champ de vision (FOV) 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (correcteur cosinus) |
| DAQ-M | Aucun (capteur nu), Sunshine (correcteur cosinus) |
| DAQ-E | Aucun (capteur nu), champ de vision (FOV) 15°, champ de vision (FOV) 45°, champ de vision (FOV) 90°, Sunshine (correcteur cosinus) |

**Le réglage par défaut pour tous les modèles est « Sunshine » (correcteur cosinus)** — MAPIR livre chaque DAQ avec le capuchon « Sunshine » installé ; il s’agit de la configuration standard pour une utilisation en extérieur : une vue hémisphérique à 180° avec une erreur de cosinus ≤ ±4 % jusqu'à 60° et ≤ ±4,5 % jusqu'à 70° (non recommandé en dessous d'une élévation solaire d'environ 15°), avec une atténuation par conception (~12×). Votre sélection est conservée dans le projet.

{% hint style="warning" %}
**Le choix du capuchon doit correspondre au capuchon physiquement installé.**Ni le capteur ni le logiciel ne peuvent détecter quel capuchon est en place. Ce choix détermine à la fois la correction en temps réel et la mention inscrite dans chaque fichier `.daq` — l’atténuation d’environ 12× du capuchon « Sunshine » un changement de capuchon non déclaré entraîne une correction erronée des spectres d’environ ce facteur. (Le retrait puis le remontage du même capuchon se répète à environ 1,5 %.) Ne choisissez**Aucun (capteur nu)** que lorsque le capuchon est physiquement retiré ; sur un DAQ-E, « Aucun » applique toujours un profil géométrique d’usine pour son diffuseur en verre encastré — ce n’est pas une opération nulle — et un DAQ-E nu correspond à une configuration de laboratoire, et non à une configuration de terrain prise en charge.
{% endhint %}

{% hint style="info" %}
Mise à jour par rapport à un manuel précédent : le bouton « Diffuseur Sunshine installé » côté navigateur, présent depuis la version 1.1.0, a disparu. La gestion du capuchon s’effectue désormais via ce profil de capuchon propre à chaque capteur, appliqué côté serveur.
{% endhint %}

***

## La zone des graphiques

Une barre supérieure fixe comporte un **bouton permettant de basculer entre la vue en liste et la vue en grille**ainsi qu’un curseur de**zoom sur le graphique** (taille des tuiles : 200 à 2 000 px). L’affichage bascule automatiquement en vue en grille lorsqu’il existe plusieurs groupes de graphiques, et revient à la vue en liste lorsqu’il n’y en a qu’un seul ou moins. Le mode d’affichage et la taille du graphique sont conservés pour chaque projet.

Le **graphique de spectre** de chaque capteur affiche :

* **Axe X** — Longueur d’onde (nm). La grille du capteur s’étend de 340 à 1 010 nm par pas de 5 nm (135 points), interpolée à 1 nm pour l’affichage.
* **Axe Y**— pour un capteur étalonné (pastille verte**Étalonné**),**Puissance (W/m²/nm)**— irradiance spectrale étalonnée radiométriquement sur les trois modes de transport — avec un préfixe SI automatique (m/µ/n) choisi à partir du pic. Un capteur dont le fichier d’étalonnage n’est pas chargé trace ses mesures brutes sous forme d’**Intensité (comptes)**, sans mise à l’échelle. Lorsqu’un graphique superpose les deux types de données, la puissance est représentée sur l’axe de gauche et les comptes sur l’axe de droite.
* Un remplissage spectral arc-en-ciel sous une trace unique ; plusieurs capteurs sur un même graphique se superposent sous forme de lignes colorées avec des remplissages estompés.
* **Survol**— un curseur vertical indiquant la longueur d’onde et la valeur par capteur (W/m²/nm pour un capteur étalonné, nombre total de comptes dans le cas contraire) ;**glisser** pour zoomer (un bouton de dézoom apparaît lorsque le zoom est activé).
* Un bouton **+** (mode grille uniquement) pour ajouter un capteur à ce graphique ou créer un groupe (voir ci-dessous).
* Le nom de l’appareil centré en haut, et un indicateur de chargement jusqu’à l’arrivée de la première image.

La **saturation** n’est pas indiquée sur le graphique lui-même : un capteur saturé affiche le texte d’état rouge `SATURATED` et une ligne rouge `Saturated: Yes` dans le tableau des données en temps réel. Réduisez le temps d’intégration ou réactivez l’AE pour le désactiver.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tableau des données en temps réel (vue en liste)

Sous le graphique, en vue en liste, actualisé toutes les 500 ms :

* **Tous les modèles** : Échantillon de couleur de la lumière (sRGB à partir de CIE XYZ), Saturé (Oui/Non), CIE 1931 X/Y/Z, Chromaticité x/y, CIE u′/v′, CCT (K), IRC (Ra), longueur d’onde dominante (nm), longueur d’onde de crête (nm), pureté d’excitation, Duv, CIE L\*/a\*/b\* et Munsell H/V/C.
* **Capteurs étalonnés uniquement**(n'importe lequel parmi DAQ-U / DAQ-M / DAQ-E une fois que son ensemble d'étalonnage d'usine est chargé — le badge vert**Étalonné** sur la ligne du capteur en est l'indicateur) : Puissance totale (W/m²), Lux photopique (lx), Lux scotopique (lx), rapport S/P, PPFD et PPFD Red /Green /Blue (µmol/m²/s), ainsi que les irradiances opiques — cône S, mélanopique, rhodopique, cône M, cône L (toutes en W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Groupes de réflectance (Ambiante + Objet)

Deux capteurs connectés peuvent être combinés pour afficher la réflectance en temps réel — sans caméra :

1. En vue en grille, cliquez sur **+**sur une tuile de graphique et choisissez**Combiner ambiant + objet**.
2. Sélectionnez un capteur **Source de lumière ambiante**et un capteur**Scanner d’objet**(deux capteurs distincts), puis cliquez sur**Créer**.

Chloros calcule R(λ) = objet(λ) / ambiant(λ) par longueur d’onde à partir des deux flux en temps réel (0 lorsque l’ambiant est ≤ 0). Le libellé du groupe dépend de la classe d’étalonnage des capteurs :

* Les deux capteurs sont étalonnés (ensemble chargé) → **« Réflectance apparente »**.
* L’un des capteurs n’est pas étalonné → **« Réflectance relative »**.

Le groupe apparaît sous la forme d’une ligne verte `REF` dans la barre latérale et dispose de son propre graphique (fond arc-en-ciel, valeurs affichées au survol avec 4 décimales, zoom par glissement).

Le menu **+**propose également l’option**Ajouter un nouveau capteur** avec trois emplacements possibles : *Combiner le nouveau capteur* (l’ajouter à ce graphique), *Déplacer un capteur existant ici* ou *Afficher le nouveau capteur* (dans son propre graphique).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tableau des indices de végétation

En mode liste, un tableau des indices de végétation apparaît sous le graphique d’un groupe de réflectance ; ces indices sont calculés à partir de la réflectance en temps réel aux centres de bande **bleu 450 / vert 550 / rouge 670 / NIR 800 nm** (valeurs à 4 décimales, `---` lorsque le calcul est impossible ; survolez le nom d’un indice pour afficher son nom complet) :

* **Toujours affichés** (invariants d’échelle, quelle que soit la combinaison de capteurs) : NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Uniquement lorsque les deux capteurs sont étalonnés en puissance** (les deux ensembles chargés) : EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Enregistrement des fichiers `.daq`

* L’enregistrement nécessite un **projet ouvert** — sinon, le bouton « Enregistrer tout » (barre latérale) et le bouton « Enregistrer » propre à chaque capteur sont désactivés.
* Les fichiers sont enregistrés dans le répertoire **`<project folder>/light_sensor/`** ; les noms de fichiers comportent l’identifiant du capteur et un horodatage, et le nom de l’appareil est stocké avec l’enregistrement.
* Lorsqu’un enregistrement s’arrête (via « Arrêter », « Tout arrêter » ou une déconnexion en cours d’enregistrement), le fichier `.daq` finalisé est **automatiquement ajouté au projet ouvert** — il apparaît dans la liste des fichiers du projet sans ajout manuel, prêt à servir de données d’irradiation descendante pour le [traitement de la réflectance](README.md).
* Un indicateur rouge `REC` s’affiche dans les lignes en temps réel de la fenêtre modale des paramètres pendant l’enregistrement.

Pour obtenir des valeurs quantitatives d’irradiance, il faut calculer la moyenne sur au moins 15 secondes de données — il s’agit d’une caractéristique de l’instrument, et non d’un défaut.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Disposition multisenseurs et persistance des projets

* Combinez plusieurs capteurs sur un même graphique (axes partagés), conservez des graphiques distincts (disposition en grille automatique), déplacez les capteurs d’un graphique à l’autre, réorganisez les lignes/tuiles par glisser-déposer et masquez des capteurs individuels à l’aide de l’icône en forme d’œil.
* Pour chaque projet, Chloros conserve : les noms des périphériques, les couleurs des graphiques, la taille des graphiques, le mode d’affichage et les paramètres de chaque capteur (temps d’intégration, moyennage des images, état AE, sélection du capuchon).
* **La réouverture d’un projet reconnecte automatiquement ses capteurs** par adresse — port COM pour DAQ-U, périphérique BLE pour DAQ-M, nom d’hôte mDNS pour DAQ-E (résolu même si l’adresse IP de l’appareil a changé) — et réapplique le profil de capteur enregistré, le moyennage des images, l’état AE et le temps d’intégration manuel de chaque capteur.

***

## Appairage de la caméra (DLS)

Il n’y a rien à coupler. Contrairement aux flux de travail DLS des drones qui associent un capteur de lumière à une caméra en amont, Chloros fait correspondre les données DAQ aux images en aval : au moment de l’importation/du traitement, les mesures `.daq` sont interpolées à l’horodatage d’exposition de chaque capture. Enregistrez avec n’importe quel capteur connecté (le `.daq` s’ajoute automatiquement au projet), et le traitement de la réflectance détermine les valeurs appropriées en fonction de l’heure — consultez la section [Capteurs de lumière DAQ](README.md) pour savoir comment les données de rayonnement descendant sont utilisées. Pour les captures LATTICE **en direct**, sélectionnez plutôt le capteur dans le menu déroulant**Capteur de lumière** de l’onglet Caméras ; chaque capture enregistre alors la mesure utilisée, associée uniquement aux caméras de cette capture (voir [Workflows de réflectance](reflectance.md)).
