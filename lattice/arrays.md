# Réseaux multicaméras

Un **réseau**LATTICE est constitué d&#x27;au moins deux caméras LATTICE connectées pour former une unité synchronisée. Une caméra fait office de**maître**: elle émet une impulsion de déclenchement GPIO matérielle sur une ligne de synchronisation partagée (par défaut**Line2**), de sorte que chaque caméra capture le même instant. Chloros ajoute la synchronisation temporelle PTP, un aperçu en direct (mosaïque par caméra ou composite multibande unique aligné) et une capture synchronisée : chaque passe de capture produit un**groupe d’images** dans lequel toutes les caméras partagent le même horodatage et le même identifiant d’image (indiqué sous la forme `fid:N` dans la sortie de capture).

Les réseaux de caméras permettent aux caméras mono (M3M) de produire des indices de végétation : chaque caméra apporte une bande, et le réseau les aligne pour former une pile multibande. Voir [Caméras mono et indices de végétation](mono-indices.md).

Il existe trois façons équivalentes de connecter un tableau, et toutes exécutent le même flux « smart-prep » :

| Surface | Point d’entrée |
| --- | --- |
| Interface graphique | Onglet Caméras → **Connecter un tableau** (bouton bleu) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (premier numéro de série = maître) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (premier numéro de série = maître) |

Smart-prep effectue, dans l’ordre : un test de capacité réseau (ping ICMP DF + test GVSP), la sélection du niveau de synchronisation, la réduction automatique de la taille de trame pour l’adapter à la ligne, l’activation du PTP, la sélection automatique du format de pixel par caméra, la configuration automatique de l’exposition à partir de l’état enregistré de chaque caméra, et la configuration du déclencheur GPIO sur la ligne 2.

{% hint style="info" %}
Les caméras doivent être accessibles sur la liaison avant que tout cela ne fonctionne — voir [Connexion des caméras](connecting.md) pour la détection, l’adressage et le téléchargement du calibrage lors de la première connexion. Pour les configurations à plusieurs caméras, les paramètres du « receive-ring » de la carte réseau hôte sont aussi importants que la vitesse de la liaison ; le tableau complet « symptôme → solution » se trouve dans la [CLI Référence § Configuration et réglage de la carte réseau hôte](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## La boîte de dialogue « Array Connect »

L&#x27;onglet « Caméras » → **Connect Array**ouvre un assistant en trois étapes :**Sélectionner → Mode d&#x27;affichage → Paramètres**.

### Étape 1 — Sélectionner la caméra maître et les caméras esclaves

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

La boîte de dialogue analyse le réseau dès son ouverture (« Analyse du réseau... »), puis vérifie le câblage du déclencheur GPIO (« Vérification du câblage GPIO... »). Vous devez disposer d’au moins **2 caméras** pour constituer un réseau.

La vérification du câblage préremplit la sélection des rôles lorsque cela est possible et affiche l’un des trois messages suivants :

| Message | Signification |
| --- | --- |
| « Maître GPIO détecté — sélections préremplies » (vert) | La vérification a détecté la topologie de déclenchement ; les cases à cocher « Maître radio » et « Esclave » sont déjà cochées. |
| « Aucun maître détecté — vérifiez le câble GPIO » (orange) | Aucune caméra n’a détecté d’impulsion de déclenchement ; vérifiez le câblage de synchronisation. Vous pouvez toujours choisir les rôles manuellement. |
| « Pas de câble de synchronisation : {numéros de série} » (orange) | Les caméras répertoriées ne sont pas raccordées à un câble de synchronisation. |

Le tableau des caméras comporte les colonnes **Caméra / Numéro de série / IP / Maître (radio) / Esclave (case à cocher)** :

* Sélectionnez exactement **un maître**et**un ou plusieurs esclaves**. Cliquer à nouveau sur la radio du maître actuel l&#x27;efface.
* Une caméra marquée **« Pas de câble de synchronisation »** ne peut jamais être sélectionnée comme esclave — un esclave sans câblage de déclenchement attendrait indéfiniment sur la ligne de synchronisation et fournirait un flux inerte. Connectez plutôt cette caméra en tant que caméra autonome.
* Les caméras déjà connectées en mode autonome ne sont *pas* désactivées : la connexion au réseau libère la session autonome et réintègre la caméra au sein du réseau.

**Suivant : Mode d’affichage →**s’active dès qu’un maître et au moins un esclave ont été choisis.**Nouvelle analyse** relance la détection et la vérification du câblage.

{% hint style="warning" %}
**Annuler** est désactivé tant qu’une analyse ou un test de câblage est en cours — l’annulation en cours de test peut provoquer un plantage de la caméra SDK sur le micrologiciel de la caméra LATTICE. Attendez que l’icône tournante s’arrête.
{% endhint %}

### Étape 2 — Mode

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

d’affichage | Mode | Résultat |
| --- | --- |
| **Caméras séparées** | Une vignette en direct par caméra, toutes déclenchées simultanément pour que les images restent synchronisées. Chaque caméra conserve sa propre couleur et ses propres paramètres. |
| **Caméras combinées** *(par défaut)* | Une seule vignette affichant le composite multibande aligné NDVI/index. Les caméras partagent la couleur du réseau. |

Le mode d’affichage ne modifie que la présentation de l’aperçu en direct — le comportement de capture est identique dans les deux cas.

### Étape 3 — Paramètres du réseau et résultat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

prévu À l’entrée de cette scène, Chloros demande au backend une **recommandation**et applique automatiquement une combinaison de ROI et de binning adaptée à l’anneau de réception de votre carte réseau (il privilégie le binning au recadrage ROI, car le binning préserve l’intégralité du champ de vision). Chaque modification que vous effectuez relance l’analyse en temps réel et met à jour le panneau**Résultat prévu** situé à droite.

Colonne de gauche — paramètres :

| Commande | Options | Par défaut | Remarques |
| --- | --- | --- | --- |
| **ROI (champ de vision)** | Plein (2048×1536) / Moitié (1024×768) / Quart (512×384) | Plein | Recadrage du capteur : recadrage à la moitié ou au quart pour obtenir une région plus petite avec le pas de pixel natif. |
| **Regroupement de pixels** | 1× / 2× (somme 2×2) / 4× (somme 4×4) | 1× | Regroupement matériel : 2×2 = champ de vision complet pour un quart du coût de transmission ; 4×4 = champ de vision complet pour 1/16. Masqué si les caméras ne prennent pas en charge le binning. |
| **Image côté câble** (lecture) | — | — | La largeur × hauteur post-binning effectivement transmise par le câble, arrondie à des multiples de 16 (minimum 64). |
| **Résolution des broches**| case à cocher | désactivée | Chloros active normalement le regroupement de pixels automatiquement à la connexion lorsque le débit prévu tombe en dessous de**1,5 image par seconde**. Le binning conserve la taille d’image choisie et accepte le débit inférieur — et transforme une configuration surchargée en un refus catégorique de connexion au lieu d’une réduction automatique du débit. |
| **Fréquence de déclenchement** | 0,5–60 images par seconde, par pas de 0,1 | vide = auto | Fréquence de déclenchement du maître. Laissez ce champ vide pour que Chloros la calcule automatiquement. |
| **Bande passante disponible**| 20–2000 Mo/s, par pas de 10 | vide = auto | La quantité de données que l’hôte peut réellement absorber, en Mo/s —**le seul chiffre dont dépend toute l’allocation du réseau.** Détecté automatiquement à partir de la carte réseau. Réduisez-le si la matrice signale des trames corrompues : la valeur détectée surestime les capacités des adaptateurs USB et des commutateurs partagés. Sa modification relance la projection en temps réel. |

Colonne de droite — **Résultat prévu** :

* **Niveau de synchronisation** — « Capture simultanée » (vert), « Capture simultanée (émission décalée FTD) » (vert), « Capture échelonnée (décalage de 100 ms) » (orange) ou « Configuration trop volumineuse » (rouge).
* **Projection en images par seconde** — affichée sous forme de plage (« faible → forte »), car la cadence d’un réseau synchronisé est limitée par l’exposition de la caméra la plus lente.
* **Ligne carte réseau** — vitesse de liaison et débit soutenu (« carte réseau {mbps} Mbps · soutenu {N} Mo/s »).
* **Vérification des rafales à émission simulée** — le circuit NIC de l&#x27;hôte peut-il absorber une rafale simultanée provenant de toutes les caméras (« Rafale à émission simulée : X Mo · anneau NIC utilisable : Y Mo ✓/✗ ») ?
* **Vérification du budget de la liaison** — demande agrégée en régime permanent par rapport au plafond de la liaison sans risque de collision (« Budget de la liaison : {demande} Mo/s demandés par {n} caméras · plafond {plafond} Mo/s ✓/✗ sursouscrit »).
* **« Nombre maximal de caméras sur ce lien : {n} — défini par le seuil de bande passante par caméra ; le regroupement ne l&#x27;augmente donc pas. »** — s&#x27;affiche lorsque vous approchez (ou dépassez) le plafond du nombre de caméras.
* **« DES IMAGES SERONT PERDUES avec ces paramètres. »**— avertissement rouge indiquant la raison fournie par le backend, accompagné d’une liste des obstacles et de**suggestions de correction** en bleu (« Pour adapter ce réseau de caméras au réseau » / « Pour débloquer la capture simultanée »).**« Appliquer et se connecter »** est désactivé tant qu’aucune projection n’existe, et son libellé vous indique pourquoi la fonction est refusée :

| Libellé du bouton | Signification | Ce qui aide réellement |
| --- | --- | --- |
| « Analyse en cours... » | L’analyse est toujours en cours. | Patientez. |
| **« Trop de caméras pour ce réseau »**| Le réseau est surchargé (échec de la vérification d’agrégation). | Moins de caméras, des trames jumbo de bout en bout ou une carte réseau plus rapide.**Une zone d’intérêt (ROI) plus petite n’aidera PAS** — voir ci-dessous. |
| **« Réduire le ROI pour activer »** | Des trames seraient perdues avec ces paramètres (échec du contrôle de rafale/anneau). | Réduire le ROI, augmenter le binning ou réparer l’anneau de réception de la carte réseau. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Lors de la connexion, un **panneau de téléchargement de l’étalonnage** vert peut s’afficher avec une barre de progression par port série : la première fois qu’une caméra est connectée à une machine, Chloros récupère son pack d’étalonnage d’usine d’environ 3,8 Mo depuis la caméra via GigE (environ 70 secondes par caméra). Les caméras mises en cache n’affichent jamais ce panneau. Voir [Connexion des caméras](connecting.md).

## Bande passante : combien de caméras peut-on connecter ?

La capacité d’un réseau dépend de la liaison physique, et non de Chloros ; les chiffres de planification figurent donc dans le manuel du matériel : **[Planification de la bande passante du réseau](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Ce que fait Chloros avec ces données : la boîte de dialogue de connexion lance un test de réseau, estime la fréquence d’images réalisable et choisit un niveau adapté. Si le réseau surcharge la ligne, il refuse la connexion plutôt que de laisser passer des paquets sans le signaler — voir le panneau « Résultats estimés » décrit ci-dessus.

## Lorsque des trames sont manquantes

Une caméra peut être absente d’un groupe publié pour deux raisons totalement différentes,
qui nécessitent des solutions opposées. Chloros les recense séparément plutôt que de signaler un
chiffre « incomplet » qui n’en précise aucune :

| Ce qui s’est passé | Ce que cela signifie | Où chercher |
| --- | --- | --- |
| **Corrompue**— la trame est arrivée mais était structurellement défectueuse | Perte de paquets GVSP sur le chemin réseau | Le**budget de ligne**, l’anneau de réception de la carte réseau, les trames jumbo, le commutateur |
| **N’est jamais arrivé**— aucune trame n’est arrivée | La caméra ne s’est pas déclenchée, ou rien n’en est sorti | Le**câble de synchronisation M8**, la ligne de synchronisation, le fait que tous les membres soient activés |

La répartition est réévaluée toutes les 10 secondes pendant que le réseau de caméras transmet. Au-delà de 5 %, elle est
consignée avec les deux chiffres indiqués, et chaque tampon corrompu est signalé la première fois qu’il
se produit par caméra, puis regroupé une fois par minute afin qu’une longue session reste lisible.

**Des images corrompues avec zéro « jamais reçue » signifient que le déclenchement et la synchronisation par câble sont parfaits**et que chaque image perdue se situe sur le chemin réseau. La solution consiste à réduire le**Wire Budget** et à
se reconnecter.

{% hint style="warning" %}
**La réduction du taux de déclenchement n’a aucun effet sur les images corrompues.** Le rythme d’envoi des paquets
de la caméra est défini une seule fois, lors de la connexion. La réduction du taux de déclenchement modifie la fréquence à laquelle une rafale
se produit, mais pas la vitesse à laquelle la rafale elle-même est transmise sur le réseau. Sur un système de 4 caméras testé, une
réduction par 5 du taux de déclenchement n’a rien changé, tandis que la diminution du budget de bande passante de 240 à
200 Mo/s a fait passer le taux de trames corrompues de ce même système de 10,4 % à zéro.
{% endhint %}

Un réseau en cours d’exécution ne peut pas se replanifier — déconnectez-le puis reconnectez-le afin que le sélecteur de temps de connexion
puisse fonctionner en fonction du nouveau budget.

### Les adaptateurs réseau USB sont limités à 200 Mo/s

Une carte Ethernet USB annonce son débit de liaison *Ethernet*, mais ce qu’elle peut réellement
maintenir est limité par le bus USB et son pilote. Une clé USB 10 GbE était autrefois créditée
d’un débit d’environ 1 000 Mo/s — un chiffre que personne n’avait jamais mesuré — et le régulage
quatre caméras en fonction de cette marge fantôme corrompait 6 à 18 % des images alors que le système
continuait d’afficher une fréquence d’images cible satisfaisante. Les adaptateurs connectés via USB sont désormais limités à
**200 Mo/s**. Cette limite est une valeur absolue et non un pourcentage, car c’est le
bus qui fait office de limite : un adaptateur USB 1 GbE atteint environ 80 Mo/s et n’est pas concerné.

Si votre hôte est réellement plus rapide que cette limite, augmentez le **Wire Budget** pour l’indiquer.

## Synchronisation temporelle PTP

La *synchronisation* des images provient du déclencheur matériel ; le **PTP** (IEEE 1588 PTPv2) fournit des *horodatages* comparables sur tous les périphériques. Il est activé par défaut lors de la connexion du réseau de caméras :

* Le **backend hôte Chloros exécute le grand maître PTP**. Les caméras LATTICE et les capteurs de lumière DAQ-E lui sont asservis dans le domaine 0, de sorte que les horodatages des images et les spectres DAQ s’alignent sur une même horloge (~1 ms).
* `--no-ptp` (CLI) le désactive pour les travaux en laboratoire — les horodatages entre caméras ne sont alors **pas** comparables.
* Vérifiez l’état de la synchronisation avec CLI :

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

L’onglet « Cameras » (Caméras) ne comporte pas d’indicateur PTP ; les informations de synchronisation par caméra qui y apparaissent sont le **Rôle**(Maître/Esclave) en lecture seule, la**Ligne de synchronisation**et le niveau de**capacités** du réseau. L’état PTP de DAQ-E est affiché dans les détails des capteurs de l’onglet « Light Sensors » (Capteurs de lumière).

## La vue

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

en direct du réseau La zone d’affichage principale propose deux dispositions (basculables dans la barre supérieure) : la **vue en grille**(chaque vignette correspond à une cellule ; faites glisser pour réorganiser lorsque le cadenas de la grille est déverrouillé) et**vue en liste**(matrices en pleine largeur en haut, une caméra active en dessous). Le curseur**Zoom du flux** permet de redimensionner les vignettes ; lorsque la largeur des cellules est inférieure à 200 px, les superpositions nom/fps se masquent automatiquement.

Le **mode séparé** affiche une tuile par caméra. Chaque tuile affiche :

* le nom de la caméra (en haut à gauche),
* une **indication du nombre d’images par seconde** (en haut à droite) — il s’agit du *taux de capture réel* de la caméra signalé par le backend, et non du taux de rafraîchissement de l’aperçu (l’aperçu en direct est plafonné à 30 images par seconde, quel que soit le taux de capture),
* un point d’état — vert (diffusion) / orange (chargement) / rouge (erreur),
* un **indicateur de trame obsolète** lorsqu’aucune nouvelle trame n’est arrivée depuis 2 s — ce qui est normal pendant environ 5 s après toute connexion/déconnexion, le temps que le backend rééquilibre la répartition de la bande passante entre les caméras.

Le **mode combiné**affiche une seule tuile composite : le backend effectue le débayérage, la mise à l’échelle, l’alignement, le débruitage, la conversion en radiance par bande (plus la réflectance DLS lorsqu’un capteur de lumière est associé), évalue l’expression d’index du tableau, applique la table de conversion (LUT) et diffuse le résultat au format MJPEG. Jusqu’à ce que la première image alignée soit rendue, la mosaïque indique son état : « Préparation du réseau… », « Calibrage de l’alignement… », « En attente de la première image… », ou — si le budget de tentatives d’alignement automatique (~30 s) est épuisé — « Alignement requis » avec un bouton**Calibrer l’alignement**.

Informations utiles sur le mode combiné :

* Le composite est aligné sur l’image de la caméra **maître**. Le cadrage AE-ROI et la mesure ponctuelle sur le composite sont exacts pour la caméra maître et approximatifs pour les caméras esclaves ; utilisez**Split View** (paramètres du réseau → « Afficher les caméras membres ») pour obtenir des tuiles au pixel près par caméra sans ouvrir de connexions supplémentaires.
* **Couche d’affichage**(paramètres de la matrice ; désactivée par défaut) vous permet de choisir une couche de premier plan et une couche d’arrière-plan — n’importe quelle caméra membre ou**Index**. Lorsque le premier plan est défini sur « Index », les pixels situés en dehors des limites min/max de la LUT affichent la couche d’arrière-plan.
* ****Résolution de rendu** (720p par défaut) définit la hauteur du flux en direct *et* la taille d’exportation du composite enregistré. Les images par caméra sont toujours exportées en pleine résolution.
* L’alignement est calculé à chaque session et n’est jamais conservé — consultez la section « Alignement » du volet des paramètres de la matrice pour connaître les résidus RMS et accéder au bouton « Recalibrer ».

## Capture : surveillance vs analyse

Les surfaces de capture du réseau se divisent clairement en **qualité surveillance**(enregistrer ce que vous voyez) et**qualité analyse** (enregistrer les données brutes, calibrer ultérieurement) :

| Flux de travail | Niveau | Ce qui est enregistré | Interface graphique | CLI |
| --- | --- | --- | --- | --- |
| **Capture** (images fixes) | Analyse | Un groupe d’images synchronisées par passage ; fichiers par caméra à chaque niveau d’exportation sélectionné (brut/débayérisé/radiance/réflectance/aperçu/index) + fichier sidecar `.daq` | *Bouton ***Tout capturer*** + Paramètres de capture | `lattice array-capture` |
| **Enregistrer la vidéo d&#x27;index** | Surveillance | Le composite d&#x27;index combiné en direct tel qu&#x27;affiché — 8 bits, résolution d&#x27;aperçu, LUT intégrée ; nécessite que le flux en direct soit ouvert | ● Enregistrer la vidéo d&#x27;index (matrices combinées) | `lattice array-record` |
| **Série brute → création d’une vidéo**| Analyse | Images brutes du capteur à la fréquence d’acquisition maximale + manifeste + `.daq`, puis reconstruction hors ligne en vidéo calibrée de radiance / réflectance / indice, synchronisée avec les relevés DAQ | ⦿ Enregistrer la série brute →**Créer une vidéo** | `lattice array-burst` → `lattice array-build-video` |

Règle générale : si les pixels doivent alimenter des *mesures*, utilisez la capture ou la rafale (qualité analyse) ; si vous avez simplement besoin de *regarder ou de montrer* ce que le capteur a vu, enregistrez la vidéo indexée (qualité surveillance).

### Paramètres de capture (interface graphique)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

L’icône en forme d’engrenage à côté de **Capturer tout** ouvre le volet des paramètres de capture (nécessite un projet ouvert — les captures y sont enregistrées) :

* **Mode de capture**:**Unique**(un seul passage) /**Continu**(captures successives ; limité par un nombre de captures, par défaut 1, ou par une durée, par défaut 10 s) /**Intervalle** (timelapse : N captures tous les X intervalles pour un total de Y, par défaut 1 toutes les 5 s pendant 1 minute).
* **Types d’exportation par caméra**: Raw, Débayérisé, Radiance, Réflectance, Aperçu, Index — toutes les options applicables sont activées par défaut. Les options Radiance et Réflectance sont masquées pour les caméras RGB-filter ;****La réflectance n’apparaît que lorsque la caméra dispose d’un capteur de lumière DAQ** (propre ou hérité du réseau) ; l’option « Index » nécessite une expression d’index configurée.
* **Aligné** (par matrice, activé par défaut) : déforme les exportations des éléments selon le profil d’alignement de la matrice afin que les exportations soient alignées au pixel près. Le format « Raw » reste toujours non déformé mais intègre la transformation dans les métadonnées.
* **Capture la plus rapide** (bouton bascule) : « raw » uniquement + la lecture DAQ attribuée + le composite d’indice combiné gratuit, en ignorant les calculs d’étalonnage au moment de la capture pour un débit maximal — la radiance, la réflectance et l’indice sont reconstruits ultérieurement à partir du fichier `.daq` enregistré.
* Les sélections sont conservées avec le projet. Les caméras masquées ou mises en pause sont ignorées.

L’équivalent CLI (même point de terminaison du backend, même sémantique) :

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

La compression TIFF pour les captures est `deflate` (sans perte, par défaut) ou `none` — les tables de drapeaux complètes, la structure du dossier de capture et les règles de retraitement se trouvent dans la [Référence CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Appairage d’un capteur de lumière DAQ

Les aperçus corrigés en termes de réflectance et d’éclairement nécessitent des données de lumière descendante provenant d’un capteur DAQ (connecté dans l’onglet **Capteurs de lumière**) :

* La **ligne de la matrice**dans la barre latérale affiche une**poupée « DAQ · activé/désactivé »** — *activée* lorsqu’un capteur de lumière au niveau de la matrice est configuré **ou** qu’une caméra membre dispose de son propre capteur ; son info-bulle indique précisément quel capteur alimente quelle caméra.
* Attribuez ce paramètre à l’ensemble du réseau dans les paramètres du réseau → **Capteur de lumière ambiante**→ menu déroulant**Capteur de lumière**. La sélection est conservée avec le projet, se propage à chaque caméra du réseau, et les caméras individuelles peuvent toujours la remplacer par leur propre capteur.
* La ligne d’état située en dessous indique l’état en temps réel : **Désactivé**→ « En attente du premier spectre… » →**« Actif — toutes les caméras du réseau bénéficient d’une correction d’éclairage »** → ou, si aucun nouveau spectre n’est arrivé au cours des 3 dernières secondes, un message indiquant que la dernière mesure est obsolète — la dernière mesure continue d’être utilisée (les mesures n’expirent jamais sur le chemin de capture).

Lorsqu’un capteur est attribué : le type d’exportation « Réflectance » devient disponible, les aperçus en temps réel sont corrigés en fonction de l’éclairage, l’exposition automatique prédictive peut utiliser le spectre, et chaque capture de réflectance enregistre la mesure DAQ réellement utilisée sous forme de **fichier sidecar `.daq`** à côté de l’image afin que la capture puisse être retraité ultérieurement.

## Options `array-connect` CLI

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Détection automatique de toutes les caméras LATTICE (nécessite ≥2) | **La première caméra en série est le MAÎTRE.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Ligne de synchronisation GPIO. |
| `--target-fps F` | auto | Fréquence de déclenchement du maître. |
| `--binning {1,2,4}` | auto | Binning matériel. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Remplacement par l’expert du sélecteur de niveau de synchronisation. |
| `--wire-ceiling-mbps MB_PER_S` | détecté automatiquement | Budget de bande passante de l&#x27;hôte en Mo/s — la forme CLI du champ **Budget de bande passante**. Réduisez-le si la matrice signale des trames corrompues. Enregistré avec le projet, de sorte qu’une reconnexion ultérieure le rétablisse. |
| `--no-recommend` | désactivé | Ignorer l’étape d’analyse du réseau. |
| `--no-ptp` | désactivé | Désactiver le PTP (les horodatages entre caméras ne sont alors plus comparables). |

`lattice array-list`, `array-status` et `array-disconnect` gèrent la session persistante. La référence complète des sous-commandes, y compris l’alignement (`align-calibrate` / `align-apply`) et les outils réseau, se trouve dans la [Référence CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice) ; les équivalents de SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) se trouvent dans la [référence SDK](../reference/sdk-reference.md). À partir de Python, le budget de câblage est `connect_array(..., wire_ceiling_mbps=120)`, et la répartition entre les données corrompues et celles qui ne sont jamais arrivées se trouve sur [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
