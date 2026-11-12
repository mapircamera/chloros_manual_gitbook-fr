# Réseaux multicaméras

Un **réseau**LATTICE est constitué d’au moins deux caméras LATTICE connectées pour former une unité synchronisée. Une caméra fait office de**maître**: elle émet une impulsion de déclenchement GPIO matérielle sur une ligne de synchronisation partagée (par défaut**Line2**), afin que chaque caméra capture le même instant. Chloros ajoute la synchronisation temporelle PTP, un aperçu en direct (mosaïque par caméra ou image composite multibande alignée) et une capture synchronisée : chaque passe de capture produit un **groupe d’images** dans lequel toutes les caméras partagent le même horodatage et le même identifiant d’image (indiqué sous la forme `fid:N` dans la sortie de capture).

Les matrices permettent aux caméras monochromes (M3M) de produire des indices de végétation : chaque caméra apporte une bande, et la matrice les aligne pour former une pile multibande. Voir [Caméras mono et indices de végétation](mono-indices.md).

Il existe trois méthodes équivalentes pour connecter un réseau, et toutes exécutent le même flux « smart-prep » :

| Interface | Point d’entrée |
| --- | --- |
| Interface graphique | Onglet Caméras → **Connecter le réseau** (bouton bleu) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (premier numéro de série = maître) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (premier numéro de série = maître) |

Smart-prep effectue : un test des capacités réseau (ping ICMP DF + test GVSP), la sélection du niveau de synchronisation, une projection de la fréquence d’images à la taille d’image demandée, la décision relative au transport 12 bits compressé (voir [Transport 12 bits compressé](#packed-12-bit-transport)), l’activation du PTP, la sélection automatique du format de pixel par caméra, l’initialisation de l’exposition automatique à partir de l’état enregistré de chaque caméra, et la configuration du déclencheur GPIO sur la ligne 2.

**Capteur complet par défaut (nouveauté de la version 1.3.0).** Connect conserve la taille d’image que vous avez demandée — le capteur complet de 2048×1536, sans regroupement de pixels ni recadrage, sauf si vous en décidez autrement — et laisse la fréquence d’images s’adapter à ce que l’hôte et le réseau peuvent fournir. Il n’effectue jamais de regroupement de pixels ni de recadrage de son propre chef (la version 1.2.0 activait automatiquement le regroupement de pixels en dessous de 1,5 image par seconde). C’est à vous de choisir entre le binning ou une zone d’intérêt (ROI) plus petite : dans la boîte de dialogue « Connect », sélectionnez l’une ou l’autre option et la section « Projected Outcome » affiche la fréquence d’images correspondante ; `array-connect` affiche les options les plus rapides avec leurs fréquences prévues.

{% hint style="info" %}
Les caméras doivent être accessibles sur la liaison pour que tout cela fonctionne — voir [Connexion des caméras](connecting.md) pour la détection, l’adressage et le téléchargement du calibrage lors de la première connexion. Pour les configurations à plusieurs caméras, les paramètres du circuit en anneau de réception de la carte réseau hôte sont tout aussi importants que la vitesse de la liaison ; le tableau complet des symptômes et solutions se trouve dans la [Référence CLI § Configuration et réglage de la carte réseau hôte](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## La boîte de dialogue « Connect Array »

L'onglet Caméras → **Connect Array**ouvre un assistant en trois étapes :**Sélectionner → Mode d’affichage → Paramètres**.

### Étape 1 — Sélectionner la caméra maître et les caméras esclaves

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



La boîte de dialogue analyse le réseau dès son ouverture (« Analyse du réseau... »), puis détecte les câbles de synchronisation (« Détection des câbles de synchronisation... ») afin d’identifier les caméras connectées à la ligne de synchronisation M8. Vous avez besoin d’au moins **2 caméras** pour constituer un réseau.

Les câbles de synchronisation M8 se rejoignent en un seul point de jonction ; ainsi, **n’importe quelle caméra câblée peut servir de maître**. La détection attribue automatiquement les rôles — la caméra RGB (`-FRGB`) comme maître lorsqu’elle est connectée, sinon la première caméra connectée, et les autres caméras connectées comme esclaves — et affiche l’une des trois bannières suivantes :

| Bannière | Signification |
| --- | --- |
| « Câble de synchronisation détecté sur {n} caméras. N’importe laquelle d’entre elles peut être maître ; la caméra RGB (sinon la première) est présélectionnée. » (vert) | La sonde a détecté les caméras sur la ligne de synchronisation ; les cases à cocher « maître » et « esclave » sont déjà cochées. |
| « Aucun maître détecté — vérifiez le câble GPIO » (orange) | Aucune caméra n’a détecté d’impulsion de déclenchement ; vérifiez le câblage de synchronisation. Les rôles sont toujours pré-définis (la caméra RGB, ou à défaut la première caméra de la liste, en tant que maître) et vous pouvez les modifier. |
| « Pas de câble de synchronisation : {série} » (orange) | Les caméras répertoriées ne sont pas connectées à un câble de synchronisation. |

Le tableau des caméras comporte les colonnes **Caméra / Série / IP / Maître (radio) / Esclave (case à cocher)**. Les lignes sont classées par**ordre des caméras** — les caméras M3C (Bayer) avant les M3M (mono) ; parmi les M3C, FRGB, FRGN, FOCN, puis FNGB ; les M3M par longueur d’onde du filtre, en commençant par la plus basse (F450 avant F808). La liste des caméras dans la barre latérale et la disposition par défaut de la grille utilisent le même ordre.

* Choisissez exactement **un maître**et**un ou plusieurs esclaves**. Cliquer à nouveau sur le bouton radio du maître actuel le désélectionne. Choisir un autre maître se fait en un clic : le nouveau maître quitte les esclaves et l’ancien maître devient un esclave (à moins qu’il ne dispose pas de câble de synchronisation).
* Une caméra marquée **« Pas de câble de synchronisation »** ne peut jamais être sélectionnée comme esclave — un esclave sans câblage de déclenchement attendrait indéfiniment sur la ligne de synchronisation et fournirait un flux inactif. Connectez plutôt cette caméra en mode autonome.
* Les caméras déjà connectées en mode autonome ne sont *pas* désactivées : la connexion au réseau libère la session autonome et réintègre la caméra au sein du réseau.

**Suivant : Mode d’affichage →**s’active dès qu’un maître et au moins un esclave ont été choisis.**Nouvelle analyse** relance la détection et la vérification du câblage.

{% hint style="warning" %}
**Annuler** est désactivé tant qu’une analyse ou un test de câblage est en cours — l’annulation en cours de test peut provoquer le plantage du SDK de la caméra sur le micrologiciel de la caméra LATTICE. Attendez que l’icône tournante s’arrête.
{% endhint %}

### Étape 2 — Mode d’affichage



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Mode | Résultat |
| --- | --- |
| **Caméras séparées** | Une vignette en direct par caméra, toutes déclenchées simultanément pour que les images restent synchronisées. Chaque caméra conserve sa propre couleur et ses propres paramètres. |
| **Caméras combinées** *(par défaut)* | Une seule vignette affichant le composite multibande NDVI/indice aligné. Les caméras partagent la couleur de la matrice. |

Le mode d’affichage ne modifie que la présentation de l’aperçu en direct — le comportement de capture reste identique dans les deux cas.

### Étape 3 — Paramètres de la matrice et résultat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

prévu

En entrant dans cette scène, Chloros demande une **recommandation**au backend et l’applique : le**capteur complet — sans regroupement de pixels ni recadrage de la zone d’intérêt (ROI) — dès lors que la fréquence d’images le permet**, quelle que soit la fréquence fournie par l’hôte et la connexion. Ce n’est que lorsqu’aucune fréquence ne permet d’utiliser le capteur complet que la recommandation réduit l’image, et privilégie alors le regroupement de pixels plutôt que le recadrage de la zone d’intérêt, car le regroupement de pixels préserve l’intégralité du champ de vision. À partir de là, vous faites un compromis entre la résolution et la fréquence d’images. Chaque modification que vous effectuez relance l’analyse en temps réel et met à jour le panneau**Résultat prévu** situé à droite.

Colonne de gauche — paramètres :

| Commande | Options | Par défaut | Remarques |
| --- | --- | --- | --- |
| **ROI (champ de vision)** | Plein (2048×1536) / Moitié (1024×768) / Quart (512×384) | Plein | Recadrage du capteur : recadrage à la moitié ou au quart vers une région plus petite avec le pas de pixel natif. |
| **Regroupement de pixels** | 1× / 2× (somme 2×2) / 4× (somme 4×4) | 1× | Regroupement matériel : 2×2 = champ de vision complet pour un quart du coût de transmission ; 4×4 = champ de vision complet pour 1/16e. Masqué si les caméras ne prennent pas en charge le binning. |
| **Image côté câble** (lecture) | — | — | La largeur × hauteur post-binning effectivement transmise par le câble, arrondie aux multiples de 16 (minimum 64). |
| **Fréquence de déclenchement** | 0,5–60 images/s, par pas de 0,1 | vide = auto | Fréquence de déclenchement du maître. Laissez ce champ vide pour laisser *Chloros* le calculer. |
| **Bande passante disponible**| 20–2000 Mo/s, par pas de 10 | vide = auto | Quantité de données que l’hôte peut réellement absorber, en Mo/s —**le chiffre unique dont dépend toute l’allocation de la matrice.** Détecté automatiquement à partir de la carte réseau. Réduisez-le si la matrice signale des trames corrompues : la valeur détectée surestime les capacités des adaptateurs USB et des commutateurs partagés. Sa modification relance la projection en temps réel. Le champ s’ouvre avec le budget utilisé lors de la dernière connexion de la matrice (enregistré avec le projet) ; ainsi, un budget que vous avez corrigé une fois est conservé. |

La case à cocher **Résolution des broches** de la version 1.2.0 a disparu : la connexion conserve désormais toujours la taille de trame que vous choisissez et réduit la fréquence d’images à la place.

Colonne de droite — **Résultat projeté** :

* **Niveau de synchronisation** — « Capture simultanée » (vert), « Capture simultanée (émission décalée FTD) » (vert), « Capture décalée (décalage de 100 ms) » (orange) ou « Configuration trop volumineuse » (rouge). Chloros choisit le niveau : capture simultanée dès lors que la liaison peut être partagée à un débit adapté au rythme des caméras (la fréquence d’images diminue à mesure que la part de chaque caméra diminue), et capture décalée uniquement lorsque ce n’est pas possible — ainsi, un réseau de caméras se connecte sur n’importe quel réseau, à la fréquence d’images autorisée par ce réseau.
* **Projection en images par seconde** — la fréquence d’images à laquelle le réseau de caméras est censé fonctionner. Avec la capture simultanée, il s’agit d’un seul chiffre : la fréquence est définie en premier, puis l’exposition automatique remplit la période qu’elle détermine, avant d’augmenter le gain, de sorte qu’une scène peu éclairée ne la réduise pas (voir [Comment l’exposition automatique gère l’exposition et le gain](camera-settings.md)). Avec la capture décalée, il s’agit d’une plage (« sombre → lumineux »), car l’exposition de chaque caméra allonge le décalage dans une scène sombre.
* **Ligne NIC** — vitesse de liaison et débit soutenu (« NIC {mbps} Mbps · soutenu {N} Mo/s »).
* **Vérification des rafales émises simultanément** — la carte réseau de l’hôte est-elle capable de recevoir une rafale simultanée provenant de toutes les caméras (« Rafales émises simultanément : X Mo · capacité utile du réseau en anneau : Y Mo ✓/✗ »).
* **Vérification du budget de la liaison** — demande agrégée en régime permanent par rapport au plafond de la liaison sans risque de collision (« Budget de la liaison : {demande} Mo/s demandés par {n} caméras · plafond {plafond} Mo/s ✓/✗ sursouscrit »).
* **« Nombre maximal de caméras sur cette liaison : {n} — défini par le seuil de bande passante par caméra ; le regroupement ne l'augmente donc pas. »** — s'affiche lorsque vous approchez (ou dépassez) le plafond du nombre de caméras.
* **« DES TRAMES SERONT PERDUES avec ces paramètres.»**— avertissement rouge indiquant la raison fournie par le backend, accompagné d’une liste des obstacles et de**suggestions de correction** en bleu (« Pour adapter ce réseau de caméras au réseau » / « Pour activer la capture simultanée »).**Appliquer & Se connecter** est désactivé tant qu’aucune projection n’existe, et son libellé vous explique pourquoi l’opération est refusée :

| Libellé du bouton | Signification | Ce qui aide réellement |
| --- | --- | --- |
| « Analyse en cours... » | L’analyse est toujours en cours. | Patientez. |
| **«Trop de caméras pour ce réseau »**| Le réseau ne peut pas prendre en charge le réseau de caméras, même une caméra à la fois (échec de la vérification d’agrégation) : il y a bien plus de caméras qu’il ne peut en gérer, ou d’autres caméras diffusent déjà sur ce réseau. | Moins de caméras, des trames jumbo de bout en bout ou une carte réseau plus rapide.**Une zone d’intérêt (ROI) plus petite n’aidera PAS** — voir ci-dessous. |
| **« Réduire la ROI pour activer »** | Des trames seraient perdues avec ces paramètres (échec du contrôle de rafale/anneau). | Réduire la ROI, augmenter le binning ou réparer l’anneau de réception de la carte réseau. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Lors de la connexion, un **panneau de téléchargement de l’étalonnage** vert peut s’afficher avec une barre de progression par port série : la première fois qu’une caméra est connectée à une machine, Chloros récupère son pack d’étalonnage d’usine d’environ 3,8 Mo depuis la caméra via GigE (environ 70 secondes par caméra). Les caméras déjà mises en cache n’affichent jamais ce panneau ; lorsqu’il y a plus de six caméras, les lignes défilent. Voir [Connexion des caméras](connecting.md).

### Transport compressé 12 bits

Les formats 12 bits (`BayerRG12`, `Mono12`) sont normalement transmis sous forme de mots de 16 bits. **Le transport**compressé**envoie les mêmes 12 bits en 1,5 octet par pixel et les décompresse sur l’hôte, ce qui réduit de 25 % le temps d’envoi de chaque image — à pleine capacité du capteur, cela représente environ un tiers d’images par seconde en plus.**Les données enregistrées sont identiques.** La connexion « Array » le détermine automatiquement (`auto`) : elle utilise le mode « packed » uniquement lorsque cela est avantageux et que l’hôte dispose de la puissance de calcul nécessaire pour le décompressage. L’interface graphique ne propose pas d’option pour cela ; depuis l’CLI, utilisez `array-connect --packed {auto,on,off}` (valeur par défaut : `auto`). Les formats 8 bits et 16 bits ne sont jamais compressés.

## Bande passante : combien de caméras peut-on connecter ?

La capacité d’un réseau de caméras dépend de la ligne, et non d’Chloros ; les chiffres de planification se trouvent donc dans le manuel du matériel : **[Planification de la bande passante du réseau de caméras](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Ce qu’Chloros en fait : la boîte de dialogue de connexion lance un test de réseau, estime la fréquence d’images réalisable et choisit un niveau adapté. Un réseau de caméras se connecte sur n’importe quel réseau : sur une liaison lente, chaque caméra est adaptée à sa part de la bande passante et la fréquence d’images diminue, plutôt que le réseau de caméras soit refusé ou que ses images soient réduites. Seul un réseau dont la bande passante ne permet pas de prendre en charge ne serait-ce qu’une seule caméra à la fois est refusé (**Trop de caméras pour ce réseau**) plutôt que de laisser tomber des paquets en silence — voir le panneau des résultats projetés décrit ci-dessus.

## Lorsque des images sont manquantes

Une caméra peut être absente d’un groupe publié pour deux raisons totalement différentes,
qui nécessitent des solutions opposées. Chloros les recense séparément plutôt que de signaler un
chiffre « incomplet » qui n’en identifie aucune :

| Ce qui s’est passé | Ce que cela signifie | Où chercher |
| --- | --- | --- |
| **Corrompue**— la trame est arrivée mais était structurellement défectueuse | Perte de paquets GVSP sur le chemin réseau | Le**budget de bande passante**, l’anneau de réception de la carte réseau, les trames jumbo, le commutateur |
| **N’est jamais arrivée**— aucune trame n’est arrivée | La caméra ne s’est pas déclenchée, ou rien n’en est sorti | Le**câble de synchronisation M8**, la ligne de synchronisation, le fait que tous les membres soient activés |

La répartition est réévaluée toutes les 10 secondes pendant la diffusion du flux par le réseau de caméras. Au-delà de 5 %, elle est
consignée avec mention des deux chiffres, et chaque tampon corrompu est signalé la première fois qu’il
se produit par caméra, puis regroupé une fois par minute afin qu’une longue session reste lisible.

**Des images corrompues avec zéro « jamais reçues » signifient que le déclenchement et la synchronisation par câble sont parfaits**et que chaque image perdue se trouve sur le chemin réseau. La solution consiste à réduire le**Wire Budget** et à
se reconnecter.

{% hint style="warning" %}
**La réduction du taux de déclenchement n’a aucun effet sur les images corrompues.** Le rythme d’envoi des paquets
de la caméra est défini une seule fois, lors de la connexion. La réduction du taux de déclenchement modifie la fréquence à laquelle une rafale
se produit, mais pas la vitesse à laquelle la rafale elle-même est transmise sur le réseau. Sur un système à 4 caméras testé, une
réduction par 5 du taux de déclenchement n’a rien changé, tandis que la réduction du « wire budget » de 240 à
200 Mo/s a permis de faire passer le taux de trames corrompues de 10,4 % à zéro sur ce même système.
{% endhint %}

Un réseau en cours de fonctionnement ne peut pas se replanifier — déconnectez-le puis reconnectez-le afin que le sélecteur de temps de connexion
puisse fonctionner en fonction du nouveau budget.

### Les adaptateurs réseau USB sont limités

Un adaptateur Ethernet USB annonce son débit de liaison *Ethernet*, mais ce qu’il peut réellement
maintenir est limité par le bus USB et son pilote. Un dongle USB 10 GbE était autrefois crédité
d’un débit d’environ 1 000 Mo/s — un chiffre que personne n’avait jamais mesuré — et le fait de synchroniser
quatre caméras sur cette marge fantôme a corrompu 6 à 18 % des images alors que la matrice
continuait d’afficher une fréquence d’images cible satisfaisante. Les adaptateurs connectés via USB sont désormais plafonnés à
**200 Mo/s** sous Windows et sur NVIDIA Jetson, et à **400 Mo/s** sous Linux x86_64. Cette limite est une valeur absolue et non un pourcentage, car c’est le
bus qui fait office de limite : un adaptateur USB 1 GbE atteint environ 80 Mo/s et n’est pas concerné.

Si votre hôte est réellement plus rapide que la limite, augmentez le **Wire Budget** pour l’indiquer.

## Synchronisation temporelle PTP

La *synchronisation* des images provient du déclencheur matériel ; le **PTP** (IEEE 1588 PTPv2) fournit des *horodatages* comparables sur tous les périphériques. Elle est activée par défaut lors de la connexion à la matrice :

* Le **backend hôte Chloros exécute le grand maître PTP**. Les caméras LATTICE et les capteurs de lumière DAQ-E s’y synchronisent en tant qu’esclaves dans le domaine 0, de sorte que les horodatages des images et les spectres DAQ s’alignent sur une même horloge (~1 ms).
* `--no-ptp` (CLI) le désactive pour les travaux en laboratoire — les horodatages entre caméras ne sont alors **pas** comparables.
* Vérifiez l’état de la synchronisation via la CLI :

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

L’onglet « Cameras » ne comporte pas d’indicateur PTP ; les informations de synchronisation par caméra qui y sont affichées sont le **Rôle**(Maître/Esclave) en lecture seule, la**Ligne de synchronisation** et le niveau de capacités du réseau. L’état PTP du DAQ-E est affiché dans les détails du capteur de l’onglet « Light Sensors ».

## La vue

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

en direct du réseau La zone d’affichage principale propose deux dispositions (à basculer dans la barre supérieure) : la **vue en grille**(chaque vignette est une cellule ; glisser-déposer pour réorganiser lorsque le cadenas de la grille est déverrouillé) et**la vue en liste**(lignes pleine largeur : un réseau**combiné**occupe une ligne — sa tuile composite — tandis qu’un réseau**séparé**attribue à**chaque caméra membre sa propre ligne**, avec son propre flux en direct et son propre tableau de cibles d’étalonnage ; sous les réseaux, une caméra autonome — celle qui est active). Le curseur**Zoom du flux** redimensionne les tuiles ; en dessous de 200 px de largeur de cellule, les superpositions nom/fps se masquent automatiquement. Les caméras membres apparaissent dans la barre latérale et dans la disposition par défaut de la grille dans l’ordre des caméras (M3C avant M3M, FRGB en premier, caméras mono par longueur d’onde) ; le déverrouillage de la grille et le glisser-déposer prévalent toujours sur l’ordre de la grille.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



Le **mode séparé** affiche une mosaïque par caméra. Chaque mosaïque affiche en superposition :

* le nom de la caméra (en haut à gauche),
* un **affichage des images par seconde** (en haut à droite) — il s’agit du *véritable taux de capture* de la caméra signalé par le backend, et non du taux de sondage de l’aperçu (l’aperçu en direct est plafonné à 30 images par seconde quel que soit le taux de capture),
* un point d’état — vert (diffusion en continu) / orange (chargement) / rouge (erreur),
* un **indicateur de trame périmée** lorsqu’aucune nouvelle trame n’est arrivée depuis 2 s — ce qui est normal pendant environ 5 s après toute connexion/déconnexion, le temps que le backend rééquilibre la répartition de la bande passante entre les caméras.**Le mode combiné**affiche une seule tuile composite : le backend effectue le débayérage, la mise à l’échelle, l’alignement, le débruitage, la conversion en radiance par bande (plus la réflectance DLS lorsqu’un capteur de lumière est associé), évalue l’expression d’index du tableau, applique la table de conversion (LUT), puis diffuse le résultat au format MJPEG. Jusqu’à ce que la première image alignée soit affichée, la mosaïque indique son état : « Préparation du réseau… », « Calibrage de l’alignement… », « En attente de la première image… », ou — si les tentatives d’alignement automatique de la tuile (environ une minute) ont échoué — « Alignement requis » avec un bouton**Calibrer l’alignement**.

L’affichage du nombre d’images par seconde (fps) de la tuile du réseau indique également l’état du flux :**« Application des paramètres… »**lorsqu’une modification des paramètres (format, regroupement de pixels, zone d’intérêt) redémarre les caméras,**« Bloqué »**lorsque le réseau est actif mais a cessé de fournir des groupes d’images, et**«Arrêté »**lorsqu’il a cessé de diffuser — dans ce cas, reconnectez le réseau. Lorsque le démixage spectral a dû limiter une bande affichée dans le composite sur une grande partie de ses pixels (25 % ou plus), la tuile affiche**« Données de bande à faible confiance · N %"** : les valeurs d’indice calculées à partir de cette bande sont suspectes (généralement, une bande a reçu trop peu de lumière).

Informations utiles sur le mode combiné :

* L’image composite est alignée sur l’image de la caméra **maîtresse**. Le cadrage AE-ROI et la mesure ponctuelle sur l’image composite sont exacts pour la caméra maîtresse et approximatifs pour les esclaves ; utilisez**Split View** (paramètres du réseau → « Afficher les caméras membres ») pour obtenir des tuiles par caméra au pixel près sans ouvrir de connexions supplémentaires vers les caméras.
* **Couche d’affichage** (paramètres de la matrice ; désactivée par défaut) vous permet de choisir une couche de premier plan et une couche d’arrière-plan — n’importe quelle caméra membre ou**Index**. Lorsque le premier plan est défini sur**Index**, les pixels situés en dehors des valeurs min/max de la LUT affichent la couche d’arrière-plan.
* **Résolution de rendu** (720p par défaut) définit la hauteur du flux en direct *et* la taille d’exportation de l’image composite enregistrée. Les images par caméra sont toujours exportées en pleine résolution.
* L’alignement est calculé par session et n’est jamais conservé. Chloros aligne **chaque réseau automatiquement après la connexion, dans les deux modes d’affichage**, une fois que l’exposition automatique des caméras membres s’est stabilisée (le logiciel continue d’essayer pendant 5 minutes maximum). La section**Alignement**du volet des paramètres du réseau affiche les résidus RMS et le bouton**Recalibrer l’alignement** — également pour les réseaux distincts, qui ne disposent pas de mosaïque composite.

## Capture : surveillance vs analyse

Les surfaces de capture des réseaux se divisent clairement en **niveau surveillance**(enregistrer ce que l’on voit) et**niveau analyse** (enregistrer les données brutes, calibrer ultérieurement) :

| Flux de travail | Niveau | Ce qu’il enregistre | Interface graphique | Interface CLI |
| --- | --- | --- | --- | --- |
| **Capture**(images fixes) | Analyse | Un groupe d’images synchronisées par passage ; fichiers par caméra à chaque niveau d’exportation sélectionné (brut/débayérisé/radiance/réflectance/aperçu/index) + fichier sidecar `.daq` | Bouton**Capturer tout** + Paramètres de capture | `lattice array-capture` |
| **Enregistrer la vidéo d’index** | Surveillance | Le composite d’index combiné en direct tel qu’affiché — 8 bits, résolution d’aperçu, LUT intégrée. Depuis l’CLI, sans interface graphique ouverte, le backend effectue le rendu de l’NDVI (ou `--expr`) du réseau de capteurs une fois celui-ci aligné ; un enregistrement qui n’écrit aucune image échoue | ● Enregistrement d’une vidéo d’index (matrices combinées) | `lattice array-record` |
| **Série brute → création d’une vidéo**| Analyse | Images brutes du capteur à la fréquence d’acquisition maximale + manifeste + `.daq`, puis reconstruction hors ligne en vidéo calibrée de radiance / réflectance / indice, synchronisée avec les relevés DAQ | ⦿ Enregistrement d’une série brute →**Création d’une vidéo** | `lattice array-burst` → `lattice array-build-video` |

Règle générale : si les pixels doivent servir à des *mesures*, utilisez la capture ou la rafale (qualité analyse) ; si vous avez simplement besoin de *regarder ou de montrer* ce que le capteur a vu, enregistrez la vidéo indexée (qualité surveillance).

### Paramètres de capture (interface graphique)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



L’icône en forme d’engrenage à côté de **Capturer tout** ouvre le volet Paramètres de capture (nécessite un projet ouvert — les captures y sont enregistrées) :

* **Mode de capture**:**Unique**(un seul passage) /**Continu**(enchaîné ; limité par un nombre de captures, par défaut 1, ou une durée, par défaut 10 s) /**Intervalle** (time-lapse : N captures tous les X intervalles pour un total de Y, par défaut 1 toutes les 5 s pendant 1 minute).
* **Types d’exportation par caméra** : Raw, Débayérisé, Radiance, Réflectance, Aperçu, Index — toutes les options applicables sont activées par défaut. Les options Radiance/Réflectance sont masquées pour les caméras à filtre RGB ; **la réflectance n’apparaît que lorsque la caméra dispose d’un capteur de lumière DAQ** (propre à la caméra ou hérité du réseau) ; l’index nécessite une expression d’index configurée.
* **Aligné**(par réseau, par défaut**activé**) : déforme les exportations des éléments selon le profil d’alignement du réseau afin que les exportations soient alignées au pixel près. Les données brutes restent toujours non déformées mais contiennent la transformation dans les métadonnées. Les fichiers des éléments du tableau sont enregistrés dans l’orientation native de chaque caméra (tout réglage de miroir/rotation est enregistré dans le fichier, mais n’est pas appliqué).
* **Capture la plus rapide** (bouton bascule) : mode « raw » uniquement + la lecture DAQ attribuée + le composite à indice combiné libre, en ignorant les calculs d’étalonnage au moment de la capture pour un débit maximal — la radiance, la réflectance et l’indice seront reconstruits ultérieurement à partir du fichier `.daq` enregistré.
* Les sélections sont conservées avec le projet. Les caméras masquées ou mises en pause sont ignorées.
* Le mode **Continu** sur les réseaux s’exécute au sein du backend Chloros (nouveauté de la version 1.3.0) : lorsque toutes les caméras de la liste appartiennent à un réseau, le backend exécute lui-même la boucle de capture, enregistrant un groupe tout en sélectionnant le suivant, sans requête par capture. Un disque plein interrompt l’exécution (« Arrêté : disque plein. N enregistrés, F a échoué »). Voir [Paramètres et modes de capture](capture.md#capture-modes) pour la remarque de Microsoft Defender concernant les taux de capture élevés.

L’CLI équivalente (même point de terminaison du backend, même sémantique) :

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

## Association d’un capteur de lumière DAQ

Les aperçus avec correction de la réflectance et de l’éclairage nécessitent des données de lumière descendante provenant d’un capteur DAQ (connecté dans l’onglet **Capteurs de lumière**) :

* La **ligne de matrice**de la barre latérale affiche un**bouton « DAQ · activé/désactivé »** — *activé* lorsqu’un capteur de lumière au niveau de la matrice est configuré **ou** qu’une caméra membre dispose de son propre capteur ; son info-bulle indique précisément quel capteur alimente quelle caméra.
* Effectuez l’attribution à l’échelle de la matrice dans les paramètres de la matrice → **Capteur de lumière ambiante**→ menu déroulant**Capteur de lumière**. La sélection est conservée avec le projet, se répercute sur toutes les caméras de la matrice, et chaque caméra peut toujours la remplacer par son propre capteur.
* La ligne d’état située en dessous indique l’état en temps réel : **Désactivé**→ « En attente du premier spectre… » →**« Actif — toutes les caméras du réseau bénéficient d’une correction d’éclairage »** → ou, si aucun nouveau spectre n’est arrivé au cours des 3 dernières secondes, un message indiquant que la dernière mesure est obsolète — la dernière mesure continue d’être utilisée (les mesures n’expirent jamais sur le chemin de capture).

Lorsqu’un capteur est attribué : le type d’exportation « Réflectance » devient disponible, les aperçus en temps réel sont corrigés en fonction de l’éclairage, l’exposition automatique prédictive peut utiliser le spectre, et chaque capture de réflectance enregistre la lecture DAQ effectivement utilisée sous la forme d’un **fichier `.daq`** dans le dossier `daq/`, ce qui permet de retraiter la capture ultérieurement. À côté, un fichier `.daq.cameras.json` répertorie les caméras dont les images sont décrites par cette mesure ; lors du retraitement, Chloros associe la mesure uniquement aux images provenant de ces caméras.

## Options de la CLI

** pour `array-connect`

| Option | Par défaut | Description |
| --- | --- | --- |
| `--serials SN1,SN2,…` | détection automatique de toutes les caméras LATTICE (nécessite ≥2) | **Le premier numéro de série correspond au MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Ligne de synchronisation GPIO. |
| `--target-fps F` | auto | Fréquence de déclenchement du maître. |
| `--binning {1,2,4}` | auto | Binning matériel. La valeur « auto » correspond au capteur complet (sans binning) dès qu’une fréquence d’images le prend en charge ; « connect » désactive le binning pour maintenir la fréquence. |
| `--packed {auto,on,off}` | `auto` | [Transport 12 bits compressé](#packed-12-bit-transport): `auto` effectue le compactage uniquement lorsque cela est utile et que l’hôte dispose de la capacité nécessaire pour le décompresser. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Remplacement par un expert du sélecteur de niveau de synchronisation. |
| `--wire-ceiling-mbps MB_PER_S` | détecté automatiquement | Budget de bande passante de l’hôte en Mo/s — la forme CLI du champ **Budget de bande passante**. Réduisez-le si la baie signale des trames corrompues. Enregistré avec le projet, il est donc restauré lors d’une reconnexion ultérieure. |
| `--no-recommend` | désactivé | Ignorer l’étape d’analyse du réseau. |
| `--no-ptp` | désactivé | Désactiver le PTP (les horodatages entre caméras ne sont alors plus comparables). |

`lattice array-list`, `array-status` et `array-disconnect` gèrent la session persistante. La référence complète des sous-commandes, y compris l'alignement (`align-calibrate` / `align-apply`) et les outils réseau, se trouve dans la [Référence CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); les équivalents SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) se trouvent dans la [Référence SDK](../reference/sdk-reference.md). En Python, le budget de transmission est `connect_array(..., wire_ceiling_mbps=120)`, et la distinction entre les données corrompues en temps réel et celles qui ne sont jamais arrivées se trouve dans [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
