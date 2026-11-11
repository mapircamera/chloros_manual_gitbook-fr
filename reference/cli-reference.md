# Référence de l&#x27;CLI Chloros

**Version :**

1.3.0**Générée le :**29/07/2026 à 19 h 19 ·**Révisée le :** 06/11/2026**Public visé :** Optimisé pour une utilisation par les LLM ; lisible par l&#x27;homme.**Portée :** Toutes les sous-commandes de `chloros-cli` destinées aux utilisateurs, avec leurs options et des exemples copiables-collables.

Ce document constitue la référence complète de l’outil en ligne de commande `chloros-cli` fourni avec MAPIR Chloros. Il est volontairement exhaustif afin qu’un LLM (ou un humain) puisse composer n’importe quel workflow pris en charge à partir des listes ci-dessous sans avoir à inspecter le code source.

Si vous souhaitez uniquement consulter les points essentiels, rendez-vous directement à :
- [Guide de démarrage rapide en cinq minutes](#five-minute-quickstart)
- [Workflow de première connexion de la caméra LATTICE](#lattice-camera-first-connect-workflow)
- [Workflow de première connexion du capteur DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modes de capture, enregistreurs et retraitement hors ligne](#capture-modes-recorders--offline-reprocess)

> **Vous pilotez des caméras Chloros à partir d’un assistant IA ?** Claude Desktop, Claude Code, Cursor et d’autres clients MCP peuvent faire fonctionner le traitement, les caméras et matrices LATTICE, ainsi que les capteurs de lumière DAQ via `chloros-mcp`, le serveur MCP qui s’installe avec le Python SDK (ainsi qu’avec le fichier .deb Linux, dans `/usr/bin`) : `claude mcp add chloros -- chloros-mcp`. La configuration, les options et les outils se trouvent dans la section [Serveur MCP (assistants IA)](sdk-reference.md#mcp-server-ai-assistants) du SDK. Un assistant doté d’un shell peut également exécuter directement toutes les commandes de cette page.

> **Modifications apportées dans la version 1.3.0** (ce que remarque un script ou un workflow existant de la version 1.2.0).
>
> **Modifications qu&#x27;un script 1.2.0 devra peut-être prendre en compte :**
>
> - **`--preset` s&#x27;applique.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` et `align-stream` l&#x27;ont supprimé dans la version 1.2.0 et fonctionna avec les réglages propres à la caméra. Il définit désormais le format, l’exposition, le gain et la fréquence d’images du préréglage : `--preset high_speed` capture en 8 bits à 5 ms, 6 dB, 60 images par seconde. `lattice cam-connect --preset` conserve également le format du préréglage (la version 1.2.0 réglait la caméra sur 12 bits après l’application du préréglage). Voir [Options communes](#common-options-shared-by-most-subcommands).
> - **Le format de pixels Bayer sur une caméra M3M (mono) correspond à sa profondeur.** Les préréglages désignent des formats Bayer qu’un capteur mono ne peut pas prendre en charge : dans la version 1.2.0, la caméra refusait ce format et conservait celui qu’elle avait, de sorte que `--preset high_speed` n’était pas en 8 bits dans ce cas. Il s’agit désormais du format Mono de même profondeur : `high_speed` et `--pixel-format BayerRG8` donnent `Mono8`, `high_quality`, `triggered` et `BayerRG12` donnent `Mono12`. Dans un tableau mixte M3C + M3M, chaque caméra adopte le format partagé au sein de sa propre famille, et les projets `network-analysis` / `analyze-array --pixel-format` projettent chaque caméra à cette profondeur. Il en va de même dans l’autre sens : une M3C sollicitée pour un format Mono adopte le format Bayer de cette profondeur ; ainsi, un réseau dont le maître est une M3M ne réactive plus ses caméras M3C en mode Mono. Voir [Caméras mono (M3M) à bande unique](#mono-m3m-single-band-cameras).
> - **Les exportations `lattice array-capture` sont alignées par défaut, tout comme celles de l’application.** Les fichiers `debayered`, `display`, `radiance`, `reflectance` et fichiers d’index de chaque membre sont alignés sur le cadre de la caméra principale et recadrés à la zone de chevauchement des membres (plus petite que le capteur en raison des décalages des caméras, `XMP-mapir:AlignmentJSON` `"applied": true`), de sorte qu’une capture CLI correspond aux mêmes fichiers que la capture de la même scène par l’application Chloros ; `raw` reste au format natif. Auparavant, une capture sans `--aligned` enregistrait chaque fichier à sa taille native, sans transformation. `--no-aligned` conserve ce comportement. Le même comportement par défaut s’applique à `--smart`, `--continuous` et `--interval`, ainsi qu’au SDK et à `chloros-mcp`. Voir [modes `array-capture`](#array-capture-modes).
> - **`lattice array-capture --compression` utilise par défaut `none`.** Les fichiers sont enregistrés sans compression (environ 6,3 Mo par image en pleine résolution) pour obtenir le débit soutenu le plus élevé ; la version 1.2.0 utilisait le format `deflate` (environ 4,1 Mo). Utilisez `--compression deflate` pour obtenir des fichiers plus petits. Les deux formats sont sans perte.
> - **Les formats PNG et JPEG ne prennent en charge que les produits `display` / `preview` et les fichiers indexés.** Une capture `--format png` ou `jpg` dont le niveau dépasse la capacité d’un fichier 8 bits est refusée avant toute capture (code HTTP400, sortie `1`), tout comme la capture par défaut `--processing all` avec `--format png`/`jpg`. Voir [Niveaux d&#x27;exportation des captures](#capture-export-levels-the-all-default).
> - **`lattice dls` nécessite `--mac`.**Il n&#x27;y a plus de MAC par défaut (il s&#x27;agissait de l&#x27;adresse d&#x27;une unité) ; `daq pool-discover --only ble` détecte celle d&#x27;un capteur.**`lattice dls --camera SERIAL`** est une nouveauté.
> - **D&#x27;autres erreurs entraînent la sortie de `1`.** `process` se produit lorsque les exportations d&#x27;une session n&#x27;ont pas pu être vérifiées ou qu&#x27;une image a été perdue, bien que les autres produits aient été enregistrés ; une commande de capture avec un fichier quelconque qu&#x27;il n&#x27;a pas pu écrire (les autres fichiers sont toujours enregistrés) ; `lattice continuous` et `multi-stream` en cas de disque plein ; `array-burst` / `array-record` lorsque l’enregistreur signale une erreur ; `array-build-video` lorsqu’un produit demandé a été ignoré ou n’a pas été créé comme demandé ; `project burst` / `stream` lorsqu’une capture échoue en cours d’exécution ; `project sensor log` lorsque l&#x27;enregistrement ne contient aucune lecture ; `lattice exposure --set`, `gain --set` et `configure --set` avec une valeur refusée par la caméra. `lattice network-analysis --json` se termine de la même manière que le résumé (`1` pour `needs_force_slip`). À l’inverse : `daq pool-disconnect` d’un capteur qui ne fait plus partie du pool quitte `0`. Voir [Codes de sortie](#exit-codes).
> - **Un `lattice array-connect` lent ne se termine jamais par un échec lorsque le tableau est connecté.** Il attend jusqu’à 600 s (180 s dans la version 1.2.0 ; `project connect` 90 s), affichant l’étape du backend au fur et à mesure, et une connexion qui n’est pas établie à ce moment-là est annulée par le backend — chaque caméra étant remise dans l’état où elle a été trouvée — ainsi, un `Error:` (sortie `1`) signifie qu’aucun réseau de caméras n’est connecté. Dans la version 1.2.0, une connexion établie juste après la mise sous tension des caméras pouvait dépasser le délai d’attente et connecter le réseau après l’échec de la commande. La combinaison Ctrl+C annule désormais la connexion. Voir [Connexions persistantes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` conserve le capteur complet.** En l’absence des options `--width` / `--height` / `--binning`, chaque caméra est configurée pour utiliser l’intégralité du capteur et la fréquence d’images chute au niveau offert par cet ordinateur et ce réseau ; la sortie de connexion répertorie des options plus rapides (regroupement de pixels ou zone d’intérêt plus petite) avec leurs fréquences prévues. La version 1.2.0 réduisait ou regroupait les images pour maintenir la fréquence. Voir [Options `array-connect`](#array-connect-options).
> - **Un réseau lent n’empêche plus le fonctionnement d’un réseau de caméras.** Chaque caméra est synchronisée en fonction de sa bande passante disponible et la fréquence d’images diminue en conséquence. Lorsque les caméras ne peuvent pas être synchronisées à un rythme aussi lent, le réseau de caméras fonctionne en mode d’émission série (`slip-emit-and-capture`) : chaque image arrive, mais les caméras ne sont pas exposées simultanément. `--force-tier sim-capture-sim-emit` exige une exposition simultanée. Voir [Fréquence d’images du réseau et modèle de rafale](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` indiquent quelle connexion s&#x27;applique** : en l’absence de `--pixel-format`, les formats définis par « connect » (Mono12 / BayerRG12, auparavant BayerRG8) et le choix [transport 12 bits compressé](#packed-12-bit-transport---packed). La même commande peut donc indiquer une fréquence d’images différente de celle de la version 1.2.0.
> - **Les chemins relatifs sont calculés à partir de l’emplacement où vous exécutez la commande.** `-o` et les autres chemins ouverts par le backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), le chemin `output` d’une étape de recette, et le chemin par défaut `output/` de `lattice index --live` sont résolus par rapport au dossier à partir duquel vous exécutez `chloros-cli`. Dans la version 1.2.0, le backend les résolvait dans son propre dossier d’installation, ce qui, sur une version installée, entraînait une erreur « Accès refusé ». Voir [Conventions](#conventions).
> - **`lattice index` s&#x27;exécute sur le backend sous toutes ses formes**, à l&#x27;instar des autres commandes `lattice` ; veillez donc à démarrer le backend en premier. L&#x27;index hors ligne (`--input`) est calculé par le backend, qui lit le fichier TIFF et écrit les résultats à côté de celui-ci ou dans `-o`, tandis que `--list-presets` et `--list-gradients` affichent les tables du backend. Sous Windows, celles-ci fonctionnent désormais : auparavant, l’CLI de Windows répondait à toutes les formes de `lattice index`, y compris `--live`, par `Error: lattice_sdk not available`. Sous Linux, l’index hors ligne s’exécutait auparavant au sein de l’CLI sans backend ni authentification ; il nécessite désormais les deux, comme toute autre commande de l’CLI. Voir [Index / Mathématiques de la végétation](#index--vegetation-maths).
> - **Nouveaux fichiers à côté des produits.** Chaque produit de radiance, de réflectance et d’indice dans lequel le démixage de bande a limité un pixel reçoit un masque dans un dossier `clamp_masks/` situé à côté du dossier du produit, et la sortie affiche une ligne `clamp mask:`. Voir [Masque de limitation du démixage](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientation de la caméra.** Une capture à caméra unique enregistre les fichiers `raw` et `debayered` tels qu’ils ont été lus par le capteur et consigne le retournement / la rotation de la caméra dans le fichier XMP ; `process` applique ces informations aux produits qu’il génère. Les fichiers membres du tableau restent au format natif à moins que vous ne transmettiez le nouveau `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Voir [Orientation de la caméra](#à-quoi-ressemble-un-dossier-de-captures).
> - **Un nom de niveau d’exportation qui n’est pas « un » est refusé.** Les listes `lattice capture --processing` et toutes les listes `--levels` prennent en charge `raw`, `debayered`, `radiance`, `reflectance` ou `display` (ou un alias tel que `preview` ; `--processing` ainsi que `all`). Tout autre nom est refusé avant que la caméra ne soit sélectionnée (400 HTTP `UNKNOWN_LEVEL`, sortie `1`) ; 1.2.0 a enregistré `raw` pour celle-ci et a quitté `0`.
> - **`daq pool-set-cap` nécessite `--cap-id`.** Sans cela, la commande se termine par `2` ; la version 1.2.0 a supprimé la correction du capuchon, laissant le capteur nu. `--cap-id none` demande le capteur nu.
> - **`lattice resolution --binning-mode` a disparu.** Le binning est toujours « Average » ; l&#x27;indicateur est toujours accepté, affiche un avertissement et n&#x27;a aucun effet. Voir [Réglage des capteurs](#sensor-tuning).
> - **`process --debayer texture-aware` est refusé pour les captures LATTICE.** Pour un dossier contenant des captures LATTICE, `process` quitte `1` après l’importation, en indiquant leur nombre, avant de traiter quoi que ce soit. Les captures LATTICE sont toujours soumises au dématriçage standard ; la version 1.2.0 acceptait donc cette option mais l’ignorait ; l’application Chloros ne la propose pas non plus pour ces captures. Voir [`chloros-cli process`](#chloros-cli-process).
> - **Les caméras reviennent à leur état initial — sauf si `--keep-settings`.** `lattice array-disconnect`, chaque commande transitoire (`multi-capture` sans tableau, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) et `lattice continuous` remettent chaque caméra dans l’état où elle se trouvait : son câblage de déclenchement et de ligne de synchronisation, son PTP, ainsi que tous les réglages de la caméra modifiés au cours de la session (exposition et gain, cible AE, limites et amortissement, balance des blancs, niveau de noir, format de pixel, regroupement de pixels et zone d’intérêt, inversions, fréquence d’images). `--keep-settings`, quant à lui, enregistre lesdans la mémoire propre de la caméra, afin qu’elle démarre avec ces paramètres ; `--clear-settings` réinitialise cette mémoire aux paramètres d’usine. Dans les deux cas, les caméras sont laissées en fonctionnement libre. Un projet configure ses caméras à partir de ses propres paramètres enregistrés lorsqu’il les connecte. Voir [Mémoire de la caméra](#camera-memory----keep-settings----clear-settings).
> - **Tous les modes enregistrent un projet de la même manière.** Toute modification apportée à une caméra, un réseau ou un capteur de lumière connecté à un projet — depuis l’application, l’CLI, le SDK ou le serveur MCP — est enregistrée dans le fichier `cameras.json` / `sensors.json` de ce projet, et l’ouverture du projet dans n’importe quel mode configure les périphériques à partir de celui-ci. Auparavant, seule l’application enregistrait le projet, et sa sauvegarde suivante écrasait tout le reste. Voir [Un seul enregistrement de projet, tous modes confondus](#one-project-record-every-mode).
> - **Les fichiers créés à partir de plusieurs caméras mentionnent chacune d’entre elles.** Une pile multibande (`align-apply`, `align-stream`, `index --live --save-multiband`), un index en temps réel constitué des bandes de plusieurs caméras, ainsi que l’image `composite/` issue d’une capture en tableau, contiennent chacun le numéro de série, le modèle et le pack d’étalonnage de chaque caméra source sous forme de listes (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), tandis que les listes EXIF `SerialNumber` répertorient les numéros de série séparés par des virgules. Auparavant, elles ne contenaient aucune caméra, ou uniquement le numéro de série de la première caméra. Voir [Fichiers créés à partir de plusieurs caméras](#files-made-from-several-cameras).
> - **Seul un fichier `raw` contient les balises d’étalonnage de la caméra.** L’ensemble compatible MicaSense (`BlackLevel`, `RadiometricCalibration`, le fichier par canal `Band{R,G,B}RadiometricCalibration`, les ensembles `MicaSenseVignetting*`) ainsi que la charge utile `CalibrationJSON` décrivent les valeurs brutes du capteur ; ils sont donc uniquement intégrés au fichier de capture `raw`. Tous les autres fichiers — les fichiers de capture `debayered`, `display`, `radiance`, `reflectance` et les fichiers d’index, les trames `lattice continuous` et `multi-stream`, ainsi que chaque produit de type `process` — porte l’identifiant de la caméra (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) et, lors d’une capture, les données propres à l’image (`XMP-mapir:FrameJSON` : exposition, gain, température, format des pixels, heure de capture). Dans la version 1.2.0, ils contenaient également l’ensemble des données brutes, qu’un outil les appliquant (Pix4D, Metashape) appliquait aux pixels qui ne sont pas des valeurs brutes. `process` identifie l’étalonnage d’un tel fichier grâce à son numéro de série et à `CalBundleSha256`.
>
> **Autres modifications :**
>
> - **La fréquence d’images automatique d’un réseau s’adapte à ce que celui-ci fournit.** Une fréquence qui perd plus de 1 % de ses déclenchements est réduite, après évaluation sur plusieurs observations prolongées plutôt que sur une seule, et la fréquence à partir de laquelle elle a été réduite est réessayée de temps à autre. Une fréquence que vous définissez vous-même reste inchangée.
> - **L&#x27;exposition automatique est plus stable.** Elle se maintient en cas de luminosité constante, se stabilise en quelques secondes après un changement, ignore les petites zones surexposées et rétablit le fonctionnement d&#x27;une caméra qui cesse de répondre en plein soleil. Voir [Comment fonctionne l’exposition automatique](#how-auto-exposure-behaves).
> - **`process` ne compte que vos images et vos enregistrements.** Si vous relancez l’opération sur un dossier contenant le fichier `camera_cal/` d’un projet, ces fichiers d’étalonnage ne sont plus comptabilisés (« 96 images trouvées » sur un ensemble de 90 images), et un projet qui les avait déjà pris en compte les supprime.
> - **`lattice multi-test` prend en compte l’état de mise sous tension des caméras.** Il active le PTP des caméras pour la session, attend que leurs horloges se synchronisent et se stabilisent avant le premier déclenchement — chaque caméra reste verrouillée pendant 120 s, soit environ 2 minutes à compter de la mise sous tension (`--ptp-settle SECONDS` le configure), car une horloge qui vient de se synchroniser présente un décalage pouvant atteindre environ 200 µs pendant environ 2 minutes — et rétablit le PTP de chaque caméra tel qu’il était initialement. Une caméra encore synchronisée depuis un test précédent n’est pas attendue à nouveau. Auparavant, le PTP restait désactivé sur un réseau à commutateur unique où aucun DAQ-E ne répondait, et l’horloge libre de chaque caméra était interprétée comme une horloge PTP qui ne se verrouillerait pas (`PTP is not locked`, à chaque exécution). Voir [Multi-caméras (sessions transitoires)](#multi-camera-transient-sessions).
> - **Un enregistrement `.daq` indique si son capuchon a été sélectionné ou supposé.** Les DAQ-U, DAQ-M et DAQ-E sont livrés sans capuchon, et Chloros ne peut pas détecter si un capuchon est installé ; par conséquent, un capteur connecté sans capuchon spécifié reçoit par défaut la valeur `sunshine_cosine`, ce qui constitue une *supposition*. Chaque enregistrement Chloros (schéma 1.30) stocke désormais `cap_id_source` à côté de `cap_id` : `operator` si vous avez nommé un capuchon (`none` inclus), `auto_default` s’il s’agissait de la valeur par défaut. Lors du traitement d’un enregistrement dont le capuchon a été supposé, avec le capuchon DAQ du projet laissé sur « Auto », affiche un message `[daq-dls] WARNING` indiquant le nom du fichier : un DAQ-U nu, après correction de la luminosité, affiche une valeur environ 30 fois trop élevée. **Nommez la limite que vous avez définie** lors de la connexion, ou définissez la limite DAQ du projet, et l’avertissement disparaîtra.
> - **Plus de fenêtres de console pip sous Windows.** Sur un PC équipé de Python, le backend et chacun de ses image workers exécutaient une fois le message `pip` au démarrage, et chacun ouvrait une fenêtre de console. Ce n’est plus le cas.
> - **Les enregistrements `.daq` en direct survivent à une coupure de courant.**Un enregistrement est écrit avec un journal et une synchronisation complète du disque à chaque validation, en arrière-plan du capteur, de sorte qu’un disque lent retarde le fichier, mais jamais les mesures. Un enregistrement interrompu en cours de validation (coupure de courant, arrêt forcé) peut laisser un fichier `<name>.daq-journal` à côté :**copiez les deux ensemble.** Chloros effectue un retour en arrière du journal lors de la première lecture du fichier ; l’enregistrement s’ouvre donc avec toutes les données validées avant l’arrêt (seules les 3 dernières secondes environ sont perdues) ; dans la version 1.2.0, une coupure de courant au mauvais moment pouvait rendre l’intégralité du fichier illisible (« l’image disque de la base de données est mal formée »).
> - **Un enregistrement sur un disque très sollicité ne bloque plus le capteur.** `daq pool-record` met les lectures en file d’attente et les écrit sur un thread dédié ; une lecture qui n’a pas pu être écrite est comptabilisée (`daq pool-list` affiche `N write error(s)` avec la dernière erreur) au lieu de bloquer le capteur (une écriture a pris 8,4 s à côté de quatre caméras enregistrant des fichiers TIFF).
> - **`lattice multi-capture` sans tableau ne perd plus les images qu’il a enregistrées lorsqu’un déclencheur expire.** Le dépassement du délai d’un déclencheur de caméra (`SC_ERR_TIMEOUT -1011`, observé avec quatre caméras sur un commutateur 1 GbE) met fin à la session à ce groupe d’images : les groupes déjà enregistrés sont répertoriés avec `Saved:`, puis l’erreur, sortie `1`. Dans la version 1.2.0, il s&#x27;agissait d&#x27;une erreur HTTP500 sans liste, bien que les fichiers se trouvaient sur le disque.
> - **Les caméras à déclenchement logiciel ne laissent plus passer un déclenchement sur deux.** Une caméra en mode de déclenchement logiciel met désormais en file d’attente un déclenchement qui arrive pendant sa lecture (ou Chloros attend qu’elle soit prête), comme le faisaient déjà les caméras synchronisées par câble.
> - **`daq pool-discover` désigne un DAQ-E-S `DAQ-E-S`** dans sa colonne `MODEL` (et celle de `--json`, `model`). Il indiquait `DAQ-E` pour chaque capteur Ethernet ; ceux qui ne répondent pas au contrôle d’identité dans un délai d’environ 2 s figurent toujours dans la liste, sous la référence `DAQ-E`.
> - **Amélioration des produits d’indices LATTICE** (NDVI et autres indices) à partir de `process`, dans tous les modes de traitement.
> - **Caméras OCN : amélioration des produits de radiance, de réflectance et d’indices en plein soleil.** Une correction de flotte est appliquée après le démixage des bandes ; les caméras RGN restent inchangées, et les valeurs OCN diffèrent de celles de la version 1.2.0. Cela s&#x27;applique également à une capture OCN réalisée avec une version antérieure, lorsqu&#x27;elle est traitée sur un ordinateur contenant l&#x27;étalonnage utilisé lors de la capture (le logiciel Chloros conserve l’étalonnage d’une caméra une fois que celle-ci a été connectée).
> - **`lattice continuous` enregistre l’image brute.** Une exécution de `tiff` générait des images démosaïquées en 3plans, à partir desquelles `process` ne peut pas démarrer. Ses images correspondent désormais au fichier `raw` généré par une capture de tableau : un plan mosaïque aux valeurs propres au capteur, non compressé, avec le paquet d’étalonnage. Un script qui lit trois plans à partir de ces fichiers n&#x27;en lit qu&#x27;un seul.
>
> - **Chaque capture enregistre la lecture DAQ qu&#x27;elle a utilisée, tout comme le fait l&#x27;interface graphique.** `lattice capture`, `lattice continuous` (un `.daq` par trame), `array-capture` et `multi-capture` avec un tableau connecté (et le SDK) écrivent les fichiers `.daq` et son `.match.json` chaque fois qu’un DAQ est lié ou mis en pool, y compris lors d’une capture en format brut uniquement ; dans la version 1.2.0, une capture brute ou débayérisée n’en enregistrait qu’une seule avec `--force-daq`. `--no-daq` désactive cette fonctionnalité, `--daq-tolerance-ms MS` définit la tolérance de correspondance pour cette capture, et le nouveau `daq pool-bind` lie une caméra ou un réseau à un DAQ depuis l’CLI. Avec `.daq`, une capture enregistrée n’est mise en correspondance, dans `process`, qu’avec les caméras qu’elle désigne. Une capture attend désormais la lecture suivante du DAQ afin de pouvoir fusionner les lectures de part et d’autre de l’exposition. Voir [La lecture du DAQ enregistrée par une capture](#the-daq-reading-a-capture-saves) et [Une lecture du DAQ éloignée de l’exposition](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** ajoute l’index au projet pour les bandes de chaque caméra, y compris un nouveau projet. Dans la version 1.2.0, un nom que le projet ne répertoriait pas encore était ignoré avec `Unknown index name(s) ignored`.
> - **`process` à côté d’un tableau en temps réel :**il ne redémarre plus un backend contenant des caméras ou des DAQ, et attend jusqu’à 15 s qu’une exécution touche à sa fin.**Ctrl+C** arrête l’exécution dans le backend, et pas seulement dans l’CLI. Voir [Notes sur `process`](#notes).
> - **Un dossier de projet dont la structure est trop profonde pour la limite de 260 caractères des chemins d&#x27;Windows** est refusé d&#x27;emblée, en privilégiant le dossier le plus long possible, plutôt que de lancer une exécution qui n&#x27;écrit rien. Le programme d’installation Windows propose désormais l’option « Activer les chemins d’accès longs Windows » et une exclusion Microsoft Defender pour le backend, toutes deux cochées par défaut. Voir [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** exécute la capture continue propre au backend : une barre de progression toutes les 2 s, un résumé au lieu d’une liste fichier par fichier, et un code de sortie différent de zéro en cas de fichier non écrit ou de disque plein. Voir [les modes `array-capture`](#array-capture-modes).
> - **Le premier `array-capture` après `array-connect`** attend jusqu’à 20 s le premier groupe d’images du tableau au lieu de renvoyer une erreur HTTP503 « Array still warming up » (Le tableau est encore en phase de préchauffage).
> - **Exposition d’un réseau sous une zone d’intérêt (ROI) :** une petite zone d’intérêt ne limite plus l’exposition d’un élément du réseau à la période de fonctionnement libre du capteur (4,9 ms à 768 × 304). Voir [Exposition sur un élément du réseau](#exposure-on-an-array-member).
> - **L’exposition automatique ne réduit plus la fréquence d’images automatique d’un réseau.** Un réseau synchronisé par câble fonctionne à la fréquence autorisée par son réseau et la lecture des caméras ; l’exposition automatique remplit la période correspondant à cette fréquence, puis augmente le gain. Une exposition manuelle plus longue que la période (`lattice exposure --set`, `configure --set`) ralentit le déclenchement pour s’y adapter au lieu d’être refusée, et la fréquence revient à la normale lorsque l’élément repasse en mode automatique. Voir [Exposition sur un élément de réseau](#exposure-on-an-array-member).
> - **Les réseaux génèrent moins de groupes d’images incomplets** et moins d’erreurs de capture 503 (« Aucun nouveau groupe d’images synchronisé ») : l’entrée de déclenchement de chaque caméra esclave est désormais débarrassée des parasites.
> - **Transport compressé 12 bits** (`--packed`, par défaut `auto`) augmente la fréquence d’images des matrices à capteur complet d’environ un tiers. `array-connect --packet-size` est désormais appliqué (la version 1.2.0 l’acceptait mais l’ignorait). Voir [Transport 12 bits compressé](#packed-12-bit-transport---packed).
> - **Un réseau dont les caméras se rétablissent d’elles-mêmes après un redémarrage.** Après un redémarrage du commutateur, le réseau de `array-connect`-ouvre les caméras lorsqu’elles répondent et recommence la diffusion une fois le PTP réactivé ; dans la version 1.2.0, il s’arrêtait définitivement et nécessitait un nouveau `array-connect`. `array-status` signale `ptp_enabled: False` (et `ptp_off:`) lorsqu’une caméra détecte que le PTP est désactivé, alors qu’auparavant, il signalait le paramètre avec lequel le réseau était connecté. Voir [Connexions persistantes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` enregistre sans que l’interface graphique soit ouverte.** Lorsque la vue combinée de l’interface graphique est fermée, le backend génère lui-même la vue : leNDVI de la matrice, ou l’index que vous transmettez avec le nouveau `--expr`. Un enregistrement qui n’écrit aucune image échoue désormais (`no-frames: …`, sortie `1`). Auparavant, l’enregistrement ne s’effectuait que lorsque la vue combinée de l’interface graphique était ouverte, et un enregistrement qui n’écrivait rien quittait tout de même `0`. Voir [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Une caméra déjà détenue par Chloros n’est pas ouverte une deuxième fois.** Une commande qui tenterait de l’ouvrir renvoie à la place un code d’erreur HTTP 409 `CAMERA_HELD`, indiquant qui la détient et la commande qui la libère. Cela inclut une caméra qu’une autre commande utilise actuellement (une commande de type `multi-test` ou `multi-capture` ouvrant ses caméras, ou une commande de type `lattice capture`) : la deuxième commande est refusée jusqu’à ce que la première soit terminée. Le rouvrir pourrait interrompre le flux du détenteur ou provoquer un plantage du backend. Voir [Connexions persistantes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Les commandes multi-caméras transitoires transmettent toutes les caméras ou échouent.** `lattice multi-capture` sans matrice connectée, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` et `index --live` quittent `1` en nommant une caméra qui n’a transmis aucune image. Les flux sautent un groupe d’images correspondant à une caméra manquante et s’arrêtent après trois enchaînements consécutifs ; `multi-test` affiche une ligne par déclenchement, `(no frame)` indique où une caméra a manqué un déclenchement ; `align-stream` quitte également `1` en cas d’erreur de tâche. Chaque exécution rétablit ensuite chaque caméra dans l’état où elle l’a trouvée : son câblage de déclenchement et de ligne de synchronisation, son PTP (`PtpEnable`) ainsi que tous les paramètres de la caméra modifiés par l’exécution — exposition et gain, cible AE, limites et amortissement, balance des blancs, niveau de noir, format de pixel, binning et ROI, retournements, fréquence d’images — à moins que `--keep-settings` / `--clear-settings` (voir [Mémoire de la caméra](#camera-memory----keep-settings----clear-settings)). Auparavant, ces commandes conservaient toutes les images reçues et quittaient `0` (`align-stream` affichait une erreur de tâche comme `Note:`), et laissaient toutes les caméras dans `TriggerMode` `On`. Elles réglaient également l’exposition de chaque caméra à partir de la scène précédant la capture d’une image (quelques secondes tout au plus), sur la cible où un réseau connecté démarre chaque caméra à : 40 multispectrales, 60 RGB (`--brightness N` le définit), et de la conserver pendant toute la durée de la prise de vue ; il en va de même pour un seul `lattice capture` d’une caméra que Chloros ne prend pas en charge. Auparavant, la première image de chaque caméra était prise à la limite d’exposition propre à la caméra, soit environ 100 ms, quelle que soit la luminosité, et était souvent écrêtée. Voir [Multi-caméras (sessions transitoires)](#multi-camera-transient-sessions).
> - **Les commandes multi-caméras transitoires partagent le réseau de la même manière que le fait `array-connect`, et n’ouvrent que `--serials`.** Chaque caméra est synchronisée en fonction de sa part de la liaison hôte ; ainsi, quatre caméras fonctionnent sur une liaison de 1 Gbit/s ; auparavant, les quatre caméras réclamaient 125 Mo/s à elles toutes, soit la totalité de la bande passante 1 GbE, et chaque trame perdait des paquets. Une configuration que le réseau ne peut pas prendre en charge est refusée avant que la moindre caméra ne commence à diffuser. `--serials` choisit désormais les caméras, la première déclenchant l’action ; auparavant, chaque commande ouvrait toutes les caméras du réseau, quelle que soit la valeur de `--serials`. Une défaillance due à une perte de paquets GVSP est désormais signalée comme telle, au lieu d’être attribuée au câble de synchronisation M8. Voir [Multi-caméras (sessions transitoires)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` laisse les caméras fonctionner librement, telles qu’elles ont été détectées.** `TriggerMode`, `Off`, ainsi que tous les autres paramètres tels qu’ils ont été détectés lors de la connexion — les réglages du déclencheur et de la ligne de synchronisation (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), le PTP et les paramètres de caméra modifiés par la session — ainsi, une simplecommande « -camera » sur un ancien membre permet d’obtenir toutes les images en une seule fois, et le maître n’est plus laissé sur `TriggerSource` `Software`. `--keep-settings` enregistre les paramètres de la session dans la mémoire de chaque caméra, tandis que `--clear-settings` réinitialise cette mémoire (voir [Mémoire de la caméra](#camera-memory----keep-settings----clear-settings)). `lattice continuous` configure également une caméra qu’il détecte comme étant activée en mode « free-run » pour la course (et la remet en état normal après celle-ci), et une course sans sortie de cadre quitte `1`.
> - **`lattice configure --get` / `lattice exposure`** affichent un `note:` indiquant l’exposition ou gain réel de la caméra pendant que son mode automatique est activé. Un `lattice exposure --set` / `configure --set` refusé indique la plage acceptée par la caméra et rétablit le mode automatique au lieu de désactiver l’exposition automatique. Voir [Réglage du capteur](#sensor-tuning).
> - **Nouvelles commandes de caméra :**`lattice highlights` (atténue les surfaces blanches brillantes dans le rendu d’RGB) et `lattice exposure --brightness-auto`.**`lattice multi-detect`** détecte les caméras câblées en un seul cycle de sondage et définit par défaut la caméra RGB comme caméra maître.
> - **Nouvelles commandes DAQ :** `daq pool-bind` ; `pool-enable-ptp` ; et pour les paramètres propres à un DAQ-E : `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` et `pool-update-firmware`. Voir [Paramètres du périphérique DAQ-E](#daq-e-device-settings).
> - **Un DAQ-E qui cesse de transmettre est reconnecté par le backend.** Lorsqu’un flux provenant d’un DAQ-E mis en pool devient silencieux (à la suite d’un redémarrage du commutateur qui l’alimente, une perte de connexion de la caméra), le backend le reconnecte avec ses paramètres d’origine, en effectuant de nouvelles tentatives à des intervalles de plus en plus longs pendant son indisponibilité. Un enregistrement en cours se poursuit dans le même fichier `.daq`, l’interruption étant marquée dans le fichier, et le fichier `pool-record --stop` répertorie chaque interruption. Dans la version 1.2.0, le capteur restait inactif jusqu’aux fichiers `pool-disconnect` et `pool-connect`, ce qui mettait également fin à l’enregistrement. Voir [Un DAQ-E qui cesse de transmettre](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** indique si les paquets du grand maître PTP parviennent au réseau (`health`) ; en cas d’échec, un message `WARNING:` s’affiche sur stderr. Voir [`time-sync`](#chloros-cli-time-sync).
> - **Paquet .deb pour Linux :** il augmente la taille des tampons de réception des sockets du noyau dont les flux vidéo des caméras ont besoin (avec les paramètres par défaut du noyau, les tableaux perdent des images et la fréquence d’images diminue), ajoute un service backend s’exécutant sous votre compte utilisateur, et `lattice network` vérifie les tampons. L’CLI n’arrête jamais un backend exécuté par un service, et une mise à jour redémarre les services backend activés. Voir [Exécuter le backend en tant que service](#running-the-backend-as-a-service-headless) et [Hôtes Linux — tampons de réception des sockets](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` indique les cas où l’alignement n’a pas pu être effectué :**chaque raison pour laquelle une capture a été enregistrée sans alignement (absence de profil d’alignement, éléments à des résolutions différentes, tableau non synchronisé par câble, une trame sur laquelle le profil ne peut pas être mappé) s’affiche sous la forme d’une ligne `Alignment:`.**`chloros-cli daq help`** correspond à `daq --help` (il a quitté `2`), et sous Linux, l’aide répertorie les commandes matérielles directes `daq` que la version inclut.
> - **Jetson :** le paquet est `chloros_1.3.0_arm64_jp6.deb`, pour JetPack 6, et son traitement GPU s&#x27;exécute sur l&#x27;Orin avec CUDA PyTorch de JetPack. Voir [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Conventions

- Toutes les commandes sont préfixées par `chloros-cli`. Sous Windows, le binaire est `chloros-cli.exe` ; sous Linux/Jetson, il s’agit de `chloros-cli`.
- Les arguments facultatifs sont indiqués sous la forme `--flag`. Les arguments positionnels obligatoires sont indiqués sans crochets.
- Lorsqu’une valeur par défaut est fournie, l’omission du drapeau entraîne l’utilisation de cette valeur.
- L’CLI est un client HTTP léger s’appuyant sur le backend Chloros (le serveur sur `127.0.0.1:5000` que l’application de bureau utilise également). **Quelles commandes le lancent :**
  - Les commandes principales — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` et `selftest` — utilisent un backend en cours d’exécution, ou en lancent un lorsqu’aucun n’est en cours d’exécution, puis l’arrêtent à nouveau une fois leur tâche terminée.
  - **Les commandes `lattice`, `project` et `daq` n&#x27;en lancent jamais.** Elles nécessitent un backend déjà en cours d’exécution : celui de l’application de bureau Windows, le [service backend](#running-the-backend-as-a-service-headless) Linux, ou un backend que vous lancez vous-même (voir [Lancer le backend](#start-the-backend)). En l’absence de backend, elles affichent le message `Error: Chloros backend not reachable at http://127.0.0.1:5000`, accompagné de la commande permettant de le démarrer, puis se ferment (`1`).
- `CHLOROS_BACKEND_URL=<url>` redirige les familles de commandes **`lattice`**,**`project`**et**`daq pool-*`** vers un autre backend (`lattice network-analysis` pointe quant à lui vers `--backend-url`). Les commandes principales fixent délibérément `http://127.0.0.1:<port>` et l&#x27;ignorent (la valeur littérale IPv4 évite la pénalité d’environ 2 s par requête imposée par Windows lors de la conversion de `localhost` en `::1`). Voir [Variables d’environnement](#environment-variables).
- **Les chemins relatifs sont résolus par rapport au dossier à partir duquel vous exécutez `chloros-cli`.** Le backend est le processus qui ouvre les dossiers de sortie et d’entrée ; ainsi, avant d’envoyer un chemin, l’CLI le rend absolu : `-o` / `--output` (y compris sa valeur par défaut, `output`), `--input`, `--burst-dir`, `--profile`, `--save` et `--mask`, ainsi que étape [recette](#recette-dsl) `output` (par défaut `output`), ainsi que le dossier par défaut `output` de `lattice index --live`. `-o captures/` écrit donc dans `<current folder>/captures/`, et les chemins d&#x27;accès affichés sont absolus. `~` est développé. En cas de backend distant (`CHLOROS_BACKEND_URL` désignant une autre machine), les chemins sont envoyés tels qu’ils ont été saisis et résolus sur cette machine.
- Une connexion à un compte Chloros+ est requise pour tous les appels SDK/CLI (exécutez `chloros-cli login` une fois par machine ; mis en cache dans `~/.chloros/`).
- Les exemples utilisent des chemins d&#x27;accès Linux ; sous Windows, remplacez `/home/user/...` par `C:/Users/.../...`. Le port série d&#x27;un DAQ-Uest de type `COM3` sous Windows et `/dev/ttyUSB0` / `/dev/ttyACM0` sous Linux.
- Les synopsis indiquent les indicateurs mutuellement exclusifs sous la forme `{--a | --b}` et les indicateurs facultatifs sous la forme `[...]`.

---

## Synopsis de niveau supérieur

```
chloros-cli [global options] COMMAND [command options]
```

### Options globales

| Indicateur | Description |
| --- | --- |
| `--backend-exe PATH` | Remplacer l&#x27;exécutable du backend détecté automatiquement. |
| `--port N` | Port HTTP du backend (par défaut : `5000`). Utilisé uniquement par les commandes principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ; `lattice`, `project` et `daq` utilisent `CHLOROS_BACKEND_URL` (`http://127.0.0.1:5000` par défaut). |
| `-v, --verbose` | Active la sortie détaillée. |
| `--restart` | Redémarre de force le backend (tue tout processus `backend_server.py` en cours d’exécution). Un backend exécuté par un service systemd (paquet .deb Linux) n’est jamais arrêté : la commande affiche alors `sudo systemctl restart <unit>` et se termine `1`. |
| `--version` | Afficher la version (`Chloros CLI 1.3.0`). |
| `--help` | Afficher l’aide de premier niveau. |

### Index des commandes

| Commande | Objectif |
| --- | --- |
| [`process`](#chloros-cli-process) | Traiter de bout en bout un dossier contenant des captures Survey3 ou LATTICE. |
| [`login`](#chloros-cli-login) | Authentifier cette machine avec un compte Chloros+. |
| [`logout`](#chloros-cli-logout) | Effacer les identifiants mis en cache. |
| [`status`](#chloros-cli-status) | Afficher l&#x27;état actuel de la licence et de l&#x27;authentification. |
| [`export-status`](#chloros-cli-export-status) | Afficher la progression en temps réel de l’exportation Thread-4 pendant l’exécution de `process`. |
| [`language`](#chloros-cli-language) | Définir ou répertorier la langue d&#x27;affichage de l&#x27;CLI (38 langues prises en charge). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Dossier de projet par défaut (partagé avec l&#x27;interface graphique). |
| [`update`](#chloros-cli-update) | Rechercher et installer les mises à jour de Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostics système + tests de fonctionnement. |
| [`time-sync`](#chloros-cli-time-sync) | État / contrôle du « grandmaster » PTP. |
| [`lattice`](#chloros-cli-lattice) | Contrôle et capture de la caméra LATTICE (plus de 45 sous-commandes). |
| [`daq`](#chloros-cli-daq) | Contrôle des capteurs spectraux DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Ouvrir et exécuter un projet Chloros enregistré (caméras + DAQ). |

---

## Installation

`chloros-cli` est inclus dans le programme d&#x27;installation de Chloros pour ordinateur de bureau sur toutes les plateformes prises en charge — il n’y a pas de téléchargement séparé pour l’CLI. L’installation du package de la plateforme ajoute `chloros-cli` à votre `PATH`, aux côtés de l’application de bureau et du binaire backend qu’elle pilote.

Derniers téléchargements : [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Le programme d’installation fournit également des scripts de lancement pratiques (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) qui ouvrent un shell CLI prêt à l’emploi.

### Windows (.exe)

1. Téléchargez le programme d&#x27;installation Windows depuis la page de téléchargement.
2. Exécutez `Chloros Installer 1.3.0.exe` et suivez les instructions de l&#x27;assistant. Le chemin d’installation par défaut est `C:\Program Files\Chloros\` (l’interface CLI se trouve dans `C:\Program Files\Chloros\cli\`, que le programme d’installation ajoute à la variable PATH).
3. Ouvrez un nouveau terminal (`cmd.exe`, PowerShell ou Terminal Windows) afin que la version mise à jour `PATH` soit détectée.

```powershell
chloros-cli --version
```

Le programme d’installation ajoute automatiquement `chloros-cli.exe` à votre système `PATH` et intègre les pilotes de caméra LATTICE. Le backend est `C:\Program Files\Chloros\backend\chloros-backend.exe` (le dossier `backend` de votre dossier d’installation), qui ne se trouve pas sur `PATH`.

**Paramètres système.** La page *Sélectionner des tâches supplémentaires* du programme d’installation comporte deux options sous **Paramètres système**, toutes deux**cochées par défaut** :

| Option (tel que libellée dans le programme d’installation) | Fonction | Désactiver l’installation silencieuse |
| --- | --- | --- |
| **Activer les chemins d’accès longs sous Windows** (permet aux dossiers de projet d’être situés à un niveau plus profond que 260 caractères ; recommandé) | Définit la valeur `LongPathsEnabled` de Windows à l’échelle de la machine sur 1. Windows est livré avec cette option désactivée, et aucun chemin d’accès ne peut alors dépasser 260 caractères ; les produits Chloros se trouvent jusqu’à environ 130 caractères en dessous du dossier de projet, ce qui fait qu’un dossier de projet plus profond est refusé (voir [Dépannage](#troubleshooting-pointers)). Prend effet pour les programmes lancés après l’installation. Reste activé lors de la désinstallation, car d’autres programmes peuvent en dépendre. | `/MERGETASKS="!longpaths"` |
| **Exclure le backend Chloros de l’analyse de Microsoft Defender**(capture vidéo nettement plus rapide ; recommandé) | Ajoute une exclusion de**processus** Defender pour `chloros-backend.exe` dans le dossier d’installation, afin que la protection en temps réel n’analyse pas chaque fichier écrit par le backend, quel que soit l’emplacement où il est écrit. Aucun autre programme n’est exclu ; l’exclusion est supprimée lors de la désinstallation. Si Defender est absent ou verrouillé par une stratégie, l’étape échoue silencieusement et l’installation se poursuit. | `/MERGETASKS="!defenderexclusion"` |**Pourquoi l’option Defender est importante :** à des taux de capture élevés, l’analyse en temps réel des fichiers de capture représente environ 30 % du taux de capture (mesuré sur une capture continue avec 9 caméras : 77,7 captures/s non analysées, 52,7 analysées). `lattice array-capture --continuous` affiche un message lorsqu’il détecte que Defender utilise un cœur de processeur ou plus lors de la capture ; relancez le programme d’installation avec cette option cochée.

Une installation silencieuse (`/SILENT` ou `/VERYSILENT`) applique ces deux options sauf si elles sont désactivées, par exemple `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Pare-feu Windows.** Le programme d’installation ajoute toujours deux règles entrantes pour `chloros-backend.exe`, sur chaque profil réseau, et le programme de désinstallation les supprime : `Chloros Backend (UDP-In)`, pour les flux de caméra, PTP, mDNS et le flux de données DAQ-E, et `Chloros Backend (TCP-In)`, limité au sous-réseau local, pour une mise à jour du micrologiciel DAQ-E, au cours de laquelle le capteur se reconnecte à cet ordinateur pour récupérer l’image (voir [Paramètres du périphérique DAQ-E](#daq-e-device-settings)). L’API HTTP du backend reste liée à `127.0.0.1`.

### Linux amd64 (.deb)

Pour Ubuntu 22.04 LTS ou version plus récente / postes de travail x86_64 basés sur Debian.

> **Ubuntu 20.04 n&#x27;est pas pris en charge.** La liste des dépendances du paquet est dérivée de
> ce à quoi le backend est réellement lié, ce qui inclut `libc6 (>= 2.35)` ;
> Focal fournit glibc 2.31. `apt` refuse l’installation plutôt que de la laisser échouer lors de
> l’exécution. La liste mentionne également `libgl1`, que l’OpenCV fourni charge et qui peut manquer dans une
> image de serveur minimale ; `apt install` l’apporte.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Le fichier .deb installe :
- `chloros-cli` à `/usr/bin/chloros-cli`, et `chloros-backend` (lance le backend) jusqu’à `/usr/bin/chloros-backend`
- Le backend et l’CLI compilés, décompressés, dans `/usr/lib/chloros/backend/` et `/usr/lib/chloros/cli/`
- Les pilotes de caméra LATTICE
- Les modèles de débruitage, les ensembles d’étalonnage et la configuration du canal de mise à jour
- Le SDK Python pour le système `python3` (`import chloros_sdk`). `apt install` apporte également ses prérequis, `python3-numpy` et `python3-requests` (recommandés, mais non obligatoires : les programmes ci-dessus sont autonomes ; `dpkg -i` les ignore, et la sortie de l’installation indique ce qui manque). Dans un virtualenv : `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, qui augmente la taille des tampons de réception des sockets du noyau pour les flux de caméra à 32 Mo et les applique immédiatement (une valeur déjà définie à un niveau supérieur n’est pas modifiée). La suppression du paquet supprime le fichier. Voir [Hôtes Linux — tampons de réception des sockets](#linux-hosts--socket-receive-buffers).
- Deux services systemd optionnels pour le backend, aucun n’étant activé (section suivante)

#### Exécution du backend en tant que service (sans interface graphique)

Sans service, le backend (et avec lui le grand maître PTP) ne fonctionne que tant qu’il est lancé : `chloros-backend` dans un terminal, un script SDK ou une commande `chloros-cli` principale, pendant toute la durée de son exécution. Pour disposer d’un backend toujours actif sur une machine sans interface graphique, activez le service **en tant qu’utilisateur** :

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` exécute le backend en tant que cet utilisateur. Les captures qu’il écrit lui appartiennent, son journal détaillé se trouve dans son `~/.cache/chloros/logs`, et il utilise son fichier `chloros-cli login` (`~/.chloros`) ; vous devez donc d’abord vous connecter une fois en tant que cet utilisateur. PTP n’a pas besoin des droits root. Comme tout backend installé, il écoute uniquement sur le port `127.0.0.1:5000`.

`chloros-backend.service` exécute le backend en tant que **root**. Il reste installé afin qu’une machine sur laquelle il est activé continue de fonctionner, mais tout ce qu’un backend root écrit appartient à root ; un utilisateur ne peut donc pas supprimer ses propres captures. Pour basculer vers votre compte utilisateur :

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Le journal détaillé du service root se trouve dans `/var/log/chloros`, accessible aux membres du groupe `adm` (le service définit `CHLOROS_LOG_DIR` ; voir [Variables d’environnement](#environment-variables)). L’installation du paquet affiche la commande `enable` avec votre nom d’utilisateur prérempli, ainsi que le commutateur lorsque le service root est activé.

**Activer une unité de backend.** Une seule unité backend peut s’exécuter à la fois. Une deuxième unité activée (`chloros-backend.service` en plus de `chloros-backend@<user>.service`, ou deux unités `@` appartenant à deux utilisateurs différents) refuse de démarrer et reste bloqué sur `failed` avec un code de sortie 98 ; systemd ne le redémarre pas. Son `systemctl status` en explique la raison : `Another instance is already running. Exiting.`, puis ce qui retient le backend (le port 5000, ou le fichier de verrouillage et l’utilisateur auquel il appartient), puis `systemd will not restart <unit> (exit status 98)`. Après un redémarrage, l’une ou l’autre de ces unités peut être celle qui s’exécute. Pour changer, désactivez celle que vous n’utilisez pas, comme indiqué ci-dessus.**L’CLI n’arrête jamais le backend d’un service.** `chloros-cli` l’utilise tel quel, tout comme l’interface graphique et le SDK : `process` s’exécute dessus, en chargeant son propre projet à la place du dernier. Là où la CLI, dans le cas contraire, redémarrerait le backend (`--restart`, ou un `process` dont l’importation n’a rien trouvé), elle nomme l&#x27;unité, affiche `sudo systemctl restart <unit>` et quitte `1`, car un backend arrêté de cette manière resterait à l&#x27;arrêt. `GET /api/status` nomme l&#x27;unité `service` (`null` pour un backend sur lequel aucun service ne s&#x27;exécute).**Mise à niveau et suppression.** La mise à niveau du paquet arrête tous les services backend, puis redémarre chacun de ceux qui sont activés, en commençant par les unités `@<user>` (une unité démarrée manuellement mais jamais activée reste à l’arrêt, comme après un redémarrage). Si plusieurs sont activés, un seul s’exécute et les autres restent en état d’échec comme ci-dessus ; la mise à niveau affiche les codes `Chloros: N backend units are enabled: …` et `Keep one enabled: sudo systemctl disable --now <unit>`. La désinstallation les arrête et les désactive. Sa purge supprime également `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Même structure que le fichier .deb amd64, avec les tampons de réception et les [services backend](#running-the-backend-as-a-service-headless) inclus, pour Jetson Orin / Orin NX / Orin Nano sur **JetPack 6** (L4T R36). Son PyTorch est la version CUDA de JetPack 6 pour Orin ; ainsi, le traitement GPU, y compris le débayériseur tenant compte des textures, s’exécute sur le GPU du Jetson. Le paquet ne contient pas CUDA en lui-même : il utilise les bibliothèques CUDA et cuDNN installées par JetPack 6 (`sudo apt install nvidia-jetpack` sur une image qui ne les contient pas). Lorsqu’aucun GPU CUDA n’est disponible, le traitement s’effectue sur le CPU et le mode « texture-aware » utilise le débayériseur standard (voir [`process`](#chloros-cli-process)).

### Authentification unique par machine

Chaque plateforme nécessite une connexion unique à Chloros+ avant que les appels SDK ou CLI ne fonctionnent :

```bash
chloros-cli login user@example.com 'YourPassword'
```

Les identifiants sont mis en cache dans `~/.chloros/user_session.json`.

### Démarrer le backend

Les commandes `lattice`, `project` et `daq` communiquent avec un backend déjà en cours d’exécution ; elles n’en lancent jamais un (voir [Conventions](#conventions)). Veillez à ce qu’un backend reste en cours d’exécution pendant que vous les utilisez :

| Plateforme | Comment |
| --- | --- |
| Windows | Ouvrez l’application de bureau Chloros (son backend dessert également la CLI), ou exécutez `"C:\Program Files\Chloros\backend\chloros-backend.exe"` dans un terminal dédié (dans le dossier `backend` de votre dossier d’installation) et laissez-le ouvert. |
| Linux / Jetson | Exécutez `chloros-backend` dans un terminal dédié, ou gardez-en un toujours en cours d’exécution avec le [service backend](#running-the-backend-as-a-service-headless). |

Un seul backend s&#x27;exécute à la fois ; tous les clients (l&#x27;interface graphique, la CLI, les scripts SDK) le partagent. Les commandes principales (`process`, `login`, `status`, …) l’utilisent également et en lancent un lorsqu’aucun n’est en cours d’exécution.

**Un backend continue de fonctionner d’une commande à l’autre**, de sorte que la commande suivante n’a pas à le redémarrer. Il s’arrête lorsque vous vous déconnectez (`chloros-cli logout`, sauf si l’application Chloros est ouverte ou si une exécution ou une compilation hors ligne est en cours), lorsque vous exécutez `chloros-cli backend stop`, ou de lui-même après 30 minutes d’inactivité : aucune commande, aucun script ni aucune application, aucune exécution ni aucune compilation hors ligne, aucune caméra ni aucun capteur de lumière connecté. `backend stop` refuse tant qu’une exécution ou une compilation hors ligne est en cours, en indiquant son nom ; `--force` l’arrête immédiatement et cette exécution ou cette compilation est perdue (une rafale est conservée, et `array-build-video` la recompile). Configurez `CHLOROS_BACKEND_IDLE_MIN` avant le démarrage du backend pour modifier le délai de 30 minutes (`0` : jamais) ; le [service backend](#running-the-backend-as-a-service-headless) ne s’arrête jamais de lui-même. `chloros-cli backend status` indique si un backend est en cours d’exécution, ce qui l’utilise et si une compilation hors ligne est en cours. `process` ouvre son projet dans le backend en cours d’exécution, sauf si l’application Chloros est ouverte avec un projet qu’elle : dans ce cas, il refuse l’opération, et vous devez effectuer l’opération depuis l’application ou fermer ce projet au préalable. Un projet laissé ouvert par une version antérieure de `process` ou par un script SDK est simplement remplacé.

### Vérification de l’installation

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` ne quitte `0` que lorsque les sept vérifications ont été réussies. La dernière nécessite un GPU CUDA (un GPU NVIDIA, ou un Jetson Orin) ainsi que les modèles de débruitage ; par conséquent, sur un ordinateur qui n’en dispose pas, il affiche `Failed: CUDA + Denoiser` et quitte `1` ; toutes les fonctionnalités, à l’exception du débayériseur sensible aux textures (réservé au GPU), continuent de fonctionner dans ce cas.

> **Abonnement Chloros+ requis.**L’interface CLI nécessite un forfait Chloros+ actif.**Copper**est le niveau d’entrée de gamme de Chloros+ — tous les niveaux payants de Chloros+ donnent accès à l’CLI et au SDK ; seul le niveau gratuit**Iron** n’y donne pas accès. (Correspondance des identifiants de forfait : `0`=Iron/gratuit, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Mise à niveau à [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Cette limite minimale est appliquée par le backend, et pas seulement par la CLI : une requête marquée par le SDK ou la CLI sans forfait payant est rejetée avec le code `403 PLAN_UPGRADE_REQUIRED`, qu&#x27;elle provienne de `chloros-cli`, du Python SDK ou d&#x27;un client HTTP développé sur mesure. Un appelant déconnecté reçoit à la place le code `401 AUTH_REQUIRED`. L’accès fonctionne hors ligne pendant la période de grâce du forfait (30 jours pour les forfaits mensuels, jusqu’à expiration pour les forfaits annuels) et cesse à l’expiration de celle-ci ; le code `chloros-cli status` continue de fonctionner afin que la raison soit visible (il s’agit de la seule route SDK/CLI exemptée du contrôle d’accès par niveau — `GET /api/license-status`).

---

## Démarrage rapide en cinq minutes

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (étape 2) lance un backend pour son propre exécution lorsqu’aucun n’est en cours et l’arrête une fois celle-ci terminée ; lancez-en donc un à partir de l’étape 4. Voir [Démarrer le backend](#start-the-backend).

---

## `chloros-cli process`

Traitez un dossier d’images via l’intégralité du pipeline Chloros (détection de cible → étalonnage → vignette → réflectance → exportation de l’indice).

### Synopsis

```
chloros-cli process INPUT [OPTIONS]
```

### Arguments de position

| Argument | Description |
| --- | --- |
| `INPUT` | Chemin d&#x27;accès au dossier d&#x27;entrée contenant les fichiers `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ou `.dng`. |

### Options communes

| Indicateur | Valeur par défaut | Description |
| --- | --- | --- |
| `-o, --output PATH` | un nouveau dossier nommé `-n` dans le dossier de projet par défaut | Le dossier de projet. Le projet porte le même nom, et un dossier contenant déjà un fichier `project.json` se voit attribuer un nom de type `_1` / `_2` au lieu d’être écrasé. Le dossier de projet par défaut est celui choisi par `set-project-folder` (ou l’interface graphique), sinon `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (horodatage) | Nom du projet, utilisé uniquement **sans** `-o` (avec `-o`, le projet prend le nom du dossier). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utilise un débayeur neuronal « Chloros+ » pour les images Survey3 ; plus lent mais de meilleure qualité. Il est réservé à Survey3 : les captures LATTICE utilisent toujours le dématriçage standard ; ainsi, pour un dossier contenant des captures LATTICE, `process` refuse `texture-aware` après l’importation, en affichant une erreur indiquant leur nombre, puis quitte `1` avant de traiter quoi que ce soit (l’application Chloros ne propose pas non plus la fonction « Texture Aware » pour celles-ci). Il nécessite un GPU CUDA (un GPU NVIDIA sous Windows / Linux x64, le GPU de l’Orin sur un Jetson) : sans celui-ci, l&#x27;exécution utilise `standard`, et le backend consigne une ligne `[DEBAYER] … needs a CUDA GPU …` expliquant pourquoi (illustrée par `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Correction de la vignettage. |
| `--reflectance / --no-reflectance` | `--reflectance` | Étalonnage de la réflectance (utilise la cible de panneau si elle est détectée, étalonnage NIST par série pour LATTICE). Pour LATTICE multispectral, cela fait également office de **produit** de réflectance — voir [Commutateurs d’exportation par produit](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | désactivé | Appliquer les corrections GNSS PPK à partir des fichiers sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | aucun | Modèle de caméra connecté à la broche d’exposition n° 1, pour la synchronisation temporelle PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | aucun | Modèle de caméra connecté à la broche d’exposition n° 2, pour la synchronisation temporelle PPK. |
| `--recal-interval SECONDS` | `0` | Intervalle minimal de recalibrage, en secondes (l&#x27;*intervalle minimal de recalibrage* du projet). |
| `--timezone-offset HOURS` | `0` | Décalage horaire du capteur de lumière par rapport à l&#x27;UTC, en heures (le *Décalage horaire du capteur de lumière* du projet). |
| `--format FORMAT` | `TIFF (16-bit)` | L&#x27;une des valeurs suivantes : `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros n&#x27;augmente jamais la profondeur de bits d&#x27;une image : pour toutes les images 8 bits, un format plus profond est enregistré sous la forme `TIFF (8-bit)` et le script l&#x27;indique une seule fois. Dans un projet mélangeant des captures de 8 bits et de 12/16 bits, chaque capture est exportée avec sa propre profondeur de bits ou une profondeur inférieure. La luminosité est toujours en 32 bits à virgule flottante. |
| `--indices NAME [NAME ...]` | aucun | Indices de végétation par nom de préréglage (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), sans distinction de majuscules/minuscules. Chacun est ajouté au projet pour les bandes de sa caméra, un nouveau projet étant inclus. Un nom qui n’est pas un préréglage, ou que les bandes de la caméra ne permettent pas de créer (par exemple, « `NDRE` » nécessite les bandes RE et NIR, qu’aucune caméra ne fournit à elle seule), est signalé par un avertissement et ne reçoit pas d’image d’index. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forcer le point d’entrée du pipeline pour les fichiers TIFF LATTICE (les fichiers .raw de Survey3 ne sont pas concernés). C&#x27;est également la porte de secours qui permet de traiter une capture **sans fichier brut** — voir [À quoi ressemble un dossier de captures](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | activé | Génère le produit débayérisé linéaire (`Debayered_Images`). Voir [Commutateurs d’exportation par produit](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | activé | Émet l’aperçu à l’écran (`Preview_Images`) : RGB = balance des blancs (illuminant DAQ si disponible, sinon « gray-world ») + gamma ; multispec = étirement en fausses couleurs. |
| `--radiance / --no-radiance` | activé | Émet la radiance en float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Référence pour le produit de réflectance LATTICE : `auto` = la cible dans le cadre ayant passé le contrôle qualité (QA) constitue la référence absolue (une cible dotée d’un indicateur QA entre en concurrence avec le DAQ sur le score mesuré), repli sur le DAQ descendant (ρ = π·L/E) ; `target` = mode strict (pas de substitution par le DAQ) ; `daq` = autorité du DAQ. Voir [Commutateurs d&#x27;exportation par produit](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | aucun | Répertoire des balayages de réflectance cible **mesurés** par unité (`<serial>.csv`) ; reporte les spectres T3/T4P nominaux en cas d’échec. |
| `--array-alignment / --no-array-alignment` | activé | Matrices LATTICE : applique l’alignement module à module enregistré dans le fichier XMP `Chloros:Alignment*` de chaque capture à chaque produit traité (débayérisation / aperçu / radiance / réflectance / indice). Aucune opération pour les images ne comportant pas ces balises. |
| `--array-alignment-crop / --no-array-alignment-crop` | recadrage | Recadrer les exportations alignées sur la zone de chevauchement commune du réseau afin que tous les modules partagent une même empreinte ; `--no-…` conserve la zone complète du capteur (remplissage noir en dehors de la source). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Rééchantillonnage pour la déformation due à l&#x27;alignement. `nearest` préserve les DN exacts de la source (pas de mélange inter-pixels des valeurs radiométriques). |

### Options de détection des cibles

| Indicateur | Description |
| --- | --- |
| `--min-target-size PIXELS` | Taille minimale en pixels des cibles d&#x27;étalonnage pour le détecteur. |
| `--target-clustering 0-100` | Pourcentage de regroupement des cibles (valeur par défaut : `60`). |
| `--target / --targets` | Recherche des cibles d&#x27;étalonnage uniquement dans un sous-dossier `target` ou `targets` du fichier d&#x27;entrée (plus rapide). |

### Exemples

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Commutateurs d’exportation par produit (LATTICE multispectral)

Le traitement LATTICE se décompose en **tous les produits applicables en un seul passage**. Les quatre commutateurs par type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sont tous**activés par défaut** ; utilisez le formulaire `--no-<type>` pour en désactiver un. Les caméras principales RGBn&#x27;émettent jamais que des données débayérisées + aperçu (pas de radiance/réflectance par bande), donc `--radiance`/`--reflectance` n’ont aucun effet sur celles-ci. Les commutateurs sont ignorés pour `.raw` d’Survey3 (qui suit le chemin standard de réflectance/cible). *(L’ancien indicateur `--radiometric-output {reflectance,radiance,sensor-response}` a été **supprimé** et remplacé par ces commutateurs ; il n’y a plus de niveau `sensor-response`.)*

| Produit | Sortie | DAQ descendante nécessaire ? |
| --- | --- | --- |
| `--debayered` | Démosaïquage linéaire (`Debayered_Images`) : le même fichier qu’une capture enregistre à son niveau `debayered` — décalage de fond (DSNU) du pack d’étalonnage appliqué aux données brutes, puis dématriçage sensible aux contours d’OpenCV ; pas de table de conversion linéaire (LUT), de champ plat ni de démixage. | Non. |
| `--preview` | Aperçu à l&#x27;écran (`Preview_Images`) : RGB = balance des blancs + gamma ; multispec = étirement en fausses couleurs. | Non. |
| `--radiance` | float32 W/m²/sr/nm issu de la chaîne radiométrique complète (`Radiance_Images`). | N° |
| `--reflectance` | uint16 réflectance ρ (`32768` = 1,0), Compatible Pix4D. | **Oui**, sauf si une cible dans le cadre ayant passé le contrôle qualité (QA) l’ancre (voir ci-dessous). |

`--reflectance-source` sélectionne la référence de réflectance :**`auto`**(par défaut) fait d’une cible présente dans le cadre et ayant passé le contrôle qualité la**référence absolue**— les chaînes de lignes empiriques ancrées à la cible sont comparées sur des panneaux mis de côté et la valeur gagnante mesurée est appliquée, bande par bande : une bande que le DAQ ne peut pas calibrer, ou dont l’ajustement de la pente par le DAQ échoue, prend la ligne propre à la cible (`ReflectanceElBandMethods` précise laquelle) — une cible qui passe le contrôle qualité avec un drapeau (un panneau écarté, une bande hors service) est également en concurrence avec la division du DAQ sur ce score mesuré, et une trame à laquelle elle perd est marquée `ReflectanceChain=daq` — on revient à la limite descendante du DAQ (ρ = π·L/E) lorsqu’aucune cible n’est présente ou que le contrôle qualité échoue ;**`target`**est strict (pas de substitution par le DAQ);**`daq`**opte pour le comportement faisant autorité du DAQ. La géométrie de la cible (ArUco / ROI fixe / bande) provient de la configuration de la cible du projet ; `--target-reflectance-dir DIR` conserve les balayages**mesurés** par unité (`<serial>.csv`) recherchés à partir du numéro de série/QR de l’unité cible, avec les spectres T3/T4P nominaux comme solution de secours.

Le chemin de réflectance DAQ résout automatiquement le **rayonnement descendant apparié par horodatage**à partir d’un fichier**`.daq`**(DAQ-U/M/E)**ou d’un fichier `.csv` natif DAQ-M**trouvé à côté de l’imagerie. Si un ensemble d’étalonnages par caméra ou par DAQ n’est pas mis en cache localement, le pipeline**le récupère automatiquement depuis AWS** lors de la première utilisation (nécessite une connexion Internet une seule fois ; mis en cache sous `~/.chloros/`).**À qui appartient cette mesure ?**Un fichier `.daq` enregistré lors d’une capture correspond à la mesure de la ou des caméras qu’il décrit — dont le nom figure dans un fichier sidecar `<file>.daq.cameras.json` qui l’accompagne (`lattice capture`, `multi-capture`, `array-capture` et l’interface graphique ; une rafale brute nomme la lecture de chaque élément dans son manifeste) — et `process` ne la fait correspondre**qu’aux images provenant de ces caméras**. Une caméra autonome ne disposant d’aucune lecture propre n’emprunte donc plus le DAQ d’une autre caméra ; il faut lui associer un DAQ ou en enregistrer un. L’enregistrement propre à un DAQ (`daq pool-record`, ou un enregistrement lancé depuis l’interface graphique), ou tout fichier `.daq` ne comportant pas ce fichier « sidecar » (y compris les captures plus anciennes), continue de servir toutes les caméras.

#### Lecture des pixels de réflectance (Pix4D / Metashape / vos propres scripts)

La réflectance est stockée sous forme de DN entier, et **le DN correspondant à ρ = 1,0 dépend de la caméra source** :

| Source | ρ = 1,0 correspond à | Comment le déterminer |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (marge jusqu’à ρ 2,0) | Le fichier porte la mention XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (limité à ρ 1,0) | Pas de balises XMP `Chloros:*` — cette absence *est* le signe distinctif. |

**Lisez `Chloros:PixelScale` et divisez par cette valeur** plutôt que de supposer une constante. La balise est définie dans le domaine uint16, elle reste donc `32768` dans tous les formats de sortie qui effectuent un redimensionnement — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` et `TIFF (32-bit, Percent)` sont tous auto-descriptifs (normaliser d’abord le type de données stocké en uint16 : ×257 à partir de 8 bits, ×65535 à partir de float).

> **Les fichiers provenant d’anciennes versions peuvent ne pas comporter d’échelle.** Un fichier TIFF de réflectance 8 bits est redimensionné et nommé `Chloros:PixelScale` comme les autres. Les versions antérieures à la 1.3.0 *le limitaient* plutôt à 0..255 ; ainsi, toute valeur supérieure à ρ ≈ 0,008 était ramenée à 255, et omettaient à la fois le tuple `Chloros:PixelScale` et le tuple `MicaSense:RadiometricCalibration`. **Si la balise est absente d’un fichier de réflectance LATTICE, ne supposez pas d’échelle — retraitez-le** plutôt que de diviser des pixels qui n’ont jamais été divisibles.

#### Lecture des pixels d’indice (NDVI et autres indices de végétation)

**Une grille d’indice entière correspond à un étirement du contraste par image, et non à une carte de valeurs comprises entre −1 et 1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` et `JPG (8-bit)` — y compris le fichier TIFF 8 bits dans lequel une capture 8 bits est enregistrée — stockent

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

où `IndexMin` / `IndexMax` sont **les valeurs d’index finies la plus basse et la plus haute propres à cette image**. DN 0 n’est pas −1, et la pleine échelle n’est pas +1.

Chloros appose des horodatages aux deux extrémités afin que la correspondance reste inversible :

| Balise | Signification |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — la convention ci-dessous |
| `Chloros:IndexMin` | la valeur d’indice stockée en tant que DN 0 |
| `Chloros:IndexMax` | la valeur d’indice stockée en tant que pleine échelle |

Lisez-les et inversez-les, en normalisant d’abord le DN stocké entre 0 et 1 par rapport à la valeur maximale du type de données stocké, afin qu’une seule expression couvre tous les formats d’entiers :

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

La lecture ne nécessite pas `-config pix4d.config` — exiftool lit ces propriétés XMP de lui-même. La configuration n’est requise que pour les *écrire*.

> **`IndexMin == IndexMax`** signifie que chaque pixel de cette image contenait la même valeur d’index. La trame est remplie à mi-échelle et il n’y a rien par quoi diviser — prenez la valeur extrême.

> **L’absence de balises `Chloros:Index*` signifie qu’aucun mappage n’est spécifié.** Une exportation LUT est un rendu colorisé dont les pixels sont des couleurs en dégradé indexées sur la fenêtre de seuil de votre LUT, et une image ne comportant aucun pixel à indice fini n’a pas de plage à spécifier. Ne présumez pas d’un mappage dans aucun des deux cas — la règle est la même que pour un `PixelScale` manquant.**Comme l’étirement s’applique à chaque image, deux images d’un même vol ne sont pas comparables en termes de DN**— chacune possède ses propres valeurs extrêmes, de sorte que le même NDVI correspond à des DN différents selon les images. Pour les analyses quantitatives portant sur l’ensemble d’un vol (statistiques zonales, seuils fixes, séries chronologiques), exportez**`TIFF (32-bit, Percent)`**: les rasters d’indice y sont enregistrés sous forme de**valeurs d’indice elles-mêmes** de type float32, sans qu’aucun redimensionnement ni aucune balise ne soient nécessaires. Malgré le nom du format, les produits d’indice ne sont pas multipliés par 100 — cette mise à l’échelle s’applique uniquement à la réflectance de type entier.

#### Masque de limitation du démixage (pixels sur lesquels une bande a été limitée)

Le démixage spectral 3×3 peut rendre une bande négative — lorsque la lumière contient à peine cette bande, ou lorsque la scène se situe en dehors de ce que décrit la matrice de démixage de la caméra. Comme il ne s’agit pas d’une valeur réelle, l’exportation la fixe à 0 — et la vue en direct affiche également ce 0. Une bande peut être écrêtée sur quelques pixels ou sur la quasi-totalité d’entre eux ; le masque enregistre les deux cas. `Chloros:Band{R,G,B}UnmixClampFraction` indique la proportion de l’image concernée par ce phénomène ; le masque de limitation précise **quels pixels** :

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— dans un dossier `clamp_masks/`**à côté** du dossier du produit, jamais à l’intérieur de celui-ci (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif` ; une exportation de `process` vers `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — pour chaque produit de radiance, de réflectance et d’indice pour lequel au moins un pixel a été écrêté : captures et exportations `process` de même nature (ainsi que les fichiers TIFF par image de `array-build-video --save-tiffs`). `uint8` à bande unique, sur la grille de pixels propre au produit (orientation, ROI, regroupement de pixels et alignement du tableau inclus). Chaque masque indique la caméra de son produit (`Make` / `Model` / numéro de série), l’heure de capture et les balises d’objectif, et répertorie les bandes que ses bits représentent (`XMP-Camera:BandName`, bit 0 en premier) ; ses bits `ImageDescription` et `Software` l’identifient comme un masque de limitation, et il ne comporte aucun étalonnage.
- **Bits** : bit 0 = la bande du canal R du produit (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Un bit activé signifie que la valeur de ce pixel dans cette bande a été limitée à 0 — considérez-le, ainsi que tout indice dérivé de celui-ci, comme peu fiable.
- **`Chloros:UnmixClampMask`** est apposé sur chaque produit ayant fait l’objet d’un démixage : le chemin d’accès au masque par rapport au dossier du produit (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, toujours séparé par `/` — ajoutez-le au dossier de l’image pour ouvrir le masque), ou `none` lorsqu’aucun pixel n’a été limité. L’absence de balise signifie que le produit n’a jamais subi de démixage (caméras RGB ou monochromes, niveaux bruts/débayérisés).
- `lattice capture` / `array-capture` affichent le chemin d’accès du masque sous le fichier enregistré (`clamp mask: …`), et la liste des résultats de capture l&#x27;indique comme étant le `clamp_mask` de l&#x27;image.
- `lattice index --live --save-multiband` écrit un masque par caméra source pour l’empilement multibande — voir [`lattice index` — masques d’empilement multibande](#lattice-index--multiband-stack-masks). Une vidéo ne comporte pas de masque : `array-build-video` indique à la place les fractions de limitation de chaque image (voir [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Comme aucun dossier de produit ne contient de masque, à quelque niveau que ce soit, un dossier de produit (`radiance/`, `reflectance/`, le dossier `*_Images/` d’une exportation `process`) peut être transmis tel quel à Pix4D, Metashape ou tout autre logiciel de photogrammétrie — sous-dossiers compris — et qu’il n’accepte que les images. Transférez-lui le dossier de produit, et non le dossier situé au-dessus (celui-ci contient également `clamp_masks/` et tous les autres types d’exportation). L’importation propre à Chloros ignore les masques. Lecture d’un masque : `(mask & 1) != 0` signifie « la bande du canal R a été écrêtée ici », `& 2` correspond à G, `& 4` à B.

#### Données EXIF transférées lors de l’exportation

`process` copie le **bloc GPS et son ExifIFD** de la capture source vers chaque produit ; ainsi, une
exportation inclut `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` et
`CameraSerialNumber` en plus du géoréférencement.

**`FocalLength` n’est pas facultatif pour la photogrammétrie.** Pix4D calcule la distance d’échantillonnage au sol (GSD) à partir de la
distance focale et de l’altitude ; en l’absence de cette balise, le logiciel utilise une échelle totalement erronée. Lors d’un
vol au-dessus d’une orangeraie comprenant 49 prises de vue, l’absence de cette balise a transformé un site de 411 m × 160 m en une
de 47,8 km × 13 km — une orthophoto de 455 MP composée en grande partie de « nodata », ce qui a ensuite été interprété comme un problème de mosaïquage et
un problème de BigTIFF avant que quiconque ne vérifie la GSD. Si votre orthophoto s’affiche à une échelle invraisemblable, exécutez d’abord `exiftool -FocalLength` sur le produit exporté.

La copie n’est délibérément **pas** `-all:all` : les balises structurelles de l’IFD0car ses balises structurelles perturbent la sortie de LATTICE lorsqu’elles
sont copiées, et `ExifImageWidth` / `ExifImageHeight` sont exclus car ils décrivent la
capture *source* — une exportation ayant subi un redimensionnement comporterait sinon des dimensions
en contradiction avec sa propre trame. Le fichier XMP est écrit directement plutôt que copié, car ExifTool
supprime les balises XMP issues d’une même invocation lorsque le bloc XMP est copié (ce qui entraînerait la perte des balises d’étalonnage MAPIR
).

### Emplacement des fichiers de sortie

Les fichiers de sortie sont enregistrés **dans le dossier du projet, regroupés par appareil photo puis par format de fichier** :

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Le dossier de l’appareil photo est `LATT-<sensor>-<lens>-F<filter>` pour LATTICE (correspondant à l’EXIF de la capture
`Model`) et `<model>_<filter>` pour Survey3 — deux caméras partageant un capteur et un filtre mais présentant des différences
au niveau de l’objectif conservent des arborescences distinctes, car la vignettage, le champ de vision et la distorsion diffèrent. Le dossier de format
dossier suit le schéma `--format` : `tiff16`, `tiff8`, `png8`, `jpg8`, ou `tiff32` pour
`TIFF (32-bit, Percent)`.

> **Chaque produit exporté conserve le nom du fichier SOURCE.** Une exportation de radiance de
> `capture_…_raw.tif` s&#x27;appelle toujours `capture_…_raw.tif` — elle se trouve simplement dans
> `tiff32/Radiance_Images/`. **C&#x27;est le dossier qui identifie le produit, pas le nom de fichier**, donc une recherche globale
> sur `*radiance*.tif` ne donne aucun résultat ; utilisez plutôt le répertoire comme critère de correspondance.

### Enregistrements du capteur de lumière — calibrés `.daq` + `.csv`

`process` gère également les enregistrements `.daq` présents dans votre dossier d’entrée, et il n’a **pas**
a pas besoin d’images pour cela : un DAQ-U / DAQ-M / DAQ-E utilisé seul constitue une
capture complète, et un dossier ne contenant que des fichiers `.daq` est une entrée valide.

Un DAQ peut être enregistré **sans** son étalonnage — c’est ce que font par défaut les enregistreurs publics
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) le font par défaut : ils enregistrent les comptages bruts des capteurs et horodatent le fichier afin que
Chloros récupère l’étalonnage d’usine de ce capteur **par port série** (d&#x27;abord dans le cache local,
puis dans le cloud MAPIR) et l&#x27;applique. `process` réécrit le résultat :

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Le fichier `.csv` contient une ligne par mesure : horodatage UTC, temps d’intégration, puissance totale,
lux photopiques/scotopiques, PPFD (et sa répartition bleu/vert/rouge), longueur d’onde de pic, puis le
spectre complet sur la grille de longueurs d’onde propre au capteur. Le fichier `.daq` est réimporté sans être
étalonné une seconde fois.

`<name>` correspond au nom de fichier de l’enregistrement. Deux enregistrements situés dans des dossiers différents peuvent partager
une même série de mesures par capture (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) enregistrées à la
même milliseconde par deux caméras — et chacun conserve néanmoins ses propres résultats : le premier conserve
`<name>`, la suivante ajoute les dossiers qui la distinguent **après** le nom
(`<name>__<capture>_daq_calibrated.daq`), de sorte que la date et l’heure de l’enregistrement restent en tête.

En cas de réussite, l’exécution renvoie `Light-sensor products written: N (calibrated .daq + .csv)`.
La partie entre parenthèses décrit ce qui a été effectivement écrit ; on lit donc
`(RAW COUNTS — this sensor has no calibration bundle)` pour un capteur sans ensemble et
`(N calibrated, M raw counts)` pour un dossier contenant les deux. Les en-têtes propres au backend,
`[DAQ-EXPORT]` et `[RUN-SUMMARY]`, tirent leur libellé de la même manière — aucun des
trois ne peut qualifier une exportation brute de « calibrée ».

Un enregistrement DAQ-U / DAQ-M / DAQ-E dont le bundle d’étalonnage ne peut pas être récupéré — soit parce que vous êtes
hors ligne, soit parce que ce capteur n’a pas d’étalonnage enregistré — est **ignoré avec une raison indiquée** sur une
ligne `[DAQ-EXPORT]`, et n’est jamais enregistré en tant que fichier « étalonné » contenant des comptages bruts.
Connectez-vous à Internet et relancez l’opération. La raison est celle que le lecteur a effectivement
déterminée pour ce fichier (schéma illisible, absence de jeu de données, erreur d’écriture), et le résumé de la
session répertorie des raisons **distinctes** — vingt fichiers ignorés pour une même cause sont considérés comme une seule
cause, et non comme vingt répétitions de celle-ci. L’CLI nomme chaque enregistrement ignoré à la fin de
l’exécution, avec sa raison :

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Un dossier d’enregistrements qui n’en exporte **aucun** échoue : voir
[Une exécution qui n’écrit aucune image échoue](#a-run-that-writes-no-images-fails).

Un enregistrement dont le nom se termine par `_calibrated` ou `_raw` (`field_raw.daq`) n’est **pas
importé** : ce sont les noms que `process` attribue à ses propres produits, et un produit relu
comme un enregistrement serait compté deux fois. L’CLI l’indique et nomme le fichier —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— au lieu de signaler un dossier vide. Renommez un tel enregistrement avant de le traiter.

#### Les enregistrements DAQ-A s’exportent sous forme de comptes bruts

La **famille DAQ-A** est antérieure au système de pack par numéro de série et ne dispose d’aucun pack d’étalonnage
à récupérer — elle est plutôt étalonnée sur le terrain à l’aide d’une cible de réflectance, ce qui
explique pourquoi elle n’en a jamais eu besoin. Refuser ces enregistrements les privait de tout moyen d’extraire leurs
données ; ils s’exportent donc sous un **nom différent** :

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Un nom de fichier différent plutôt qu’un indicateur à l’intérieur du fichier, car l’information doit rester intacte
lorsqu’elle est transmise par e-mail sous forme de nom brut. L’en-tête `.csv` indique
`raw spectral sensor counts (NOT irradiance)` et précise que les valeurs sont comparables
**au sein** du fichier — ce qui correspond exactement à l’usage que leur fait l’étalonnage par cible — et
non entre différents capteurs. Les colonnes photométriques dépendantes de la puissance (puissance totale, lux photopiques et
scotopiques, PPFD) sont enregistrées avec la valeur **NULL** plutôt que d’être calculées à partir des comptages, et le
résumé de la session indique `RAW COUNTS` ; ainsi, « exporté» dans un journal ne peut pas être interprétée comme une irradiance.

Les enregistrements hérités **v1.01 / v1.02** (générés par un DAQ-A-SD) ne comportent pas d’époque par lecture,
mais uniquement l’heure d’écriture du fichier. Le module de correspondance image↔rayonnement descendant continue de les rejeter — faire correspondre une
image à une heure d’écriture entraînerait une erreur invisible — mais l’exportateur les lit, et
le fichier CSV affiche `clock=daq_created_on`, ce qui permet au produit d’indiquer sur quelle horloge il se base.

### Remarques

- `process` détecte automatiquement si votre dossier contient des données Survey3, « LATTICE » ou un mélange des deux.
- La progression est diffusée via des événements « Server-Sent Events » ; l’CLI affiche en temps réel la progression par thread (Détection, Analyse, Traitement, Exportation).
- Sous Linux/Jetson, la CLI vérifie l’espace d’échange et peut afficher un avertissement avant le traitement de dossiers volumineux. Le débayérage sensible aux textures applique également automatiquement une limite de fréquence du GPU sur les Jetson à faible consommation (Nano, Orin Nano).
- En cas de réussite, l’exécution indique le nombre de produits d’image générés (`Image products written: N`, ou `(plus M clamp masks)` lorsque le démixage a limité un pixel).
- **`process` partage un backend en cours d’exécution en toute sécurité.** Il redémarre automatiquement un backend déjà en cours d’exécution uniquement lorsque celui-ci est inactif, qu’un projet précédent y est chargé et qu’il ne gère aucune caméra, aucun tableau ni aucun DAQ. Un backend contenant l’un de ces éléments (l’interface graphique diffusant un tableau, un script SDK, un enregistrement DAQ) est utilisé tel quel, sans jamais être redémarré ; ainsi, `process` peut s’exécuter parallèlement à un tableau en direct. Si le backend signale une exécution en cours, `process` attend jusqu’à 15 s que celle-ci se termine (une exécution qui vient de s’achever met environ une seconde à se désactiver), puis renvoie un message d’erreur `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. L’indicateur global `--restart` force toujours le démarrage d’un nouveau backend et met fin à toute session de caméra, de tableau ou de DAQ en cours sur celui-ci. Un backend exécuté par un service systemd sous Linux n’est jamais redémarré : `process` l’utilise, et `--restart` affiche `sudo systemctl restart <unit>` à la place (voir [Exécuter le backend en tant que service](#running-the-backend-as-a-service-headless)). Il en va de même pour le backend propre à l’application Chloros : lorsque l’application est ouverte, `process` l’utilise, chargeant son projet à la place de celui de l’application (un backend démarré à sa place empêcherait l’application ouverte d’y accéder).
- **Une exécution en cours conserve son projet.** Tant qu’une exécution de traitement est en cours — lancée par l’application, `process`, un script SDK ou `chloros-mcp` —, le backend refuse de créer, d’ouvrir, d’importer, d’ajouter ou de supprimer des fichiers, de déplacer, supprimer ou modifier les paramètres de traitement d’un projet, avec un code d’erreur HTTP 409 `PROCESSING_IN_PROGRESS` et `A processing run is in progress on project '<name>', so ... now is refused` ; un `process` qui y parvient quitte `1`. Les paramètres d’affichage (taille des vignettes) sont toujours enregistrés. Le mode de traitement et les paramètres de réflectance cible du projet sont refusés de la même manière (une exécution conserve le mode dans lequel elle a démarré). Une capture enregistrée dans le projet en cours d’exécution (depuis l’application, la CLI, le SDK ou `chloros-mcp`) est enregistrée comme d’habitude, mais n’est ajoutée à la liste des fichiers du projet qu’à la fin de l’exécution ; ce que confirme la réponse (`project_add`), tout comme un enregistrement du capteur de lumière interrompu dans le projet. Auparavant, un deuxième client pouvait basculer vers le projet ouvert en cours d’exécution, et la session en cours était exportée vers l’autre projet.
- **La combinaison Ctrl+C arrête l&#x27;exécution dans le backend, et pas seulement dans l&#x27;CLI.** L’CLI demande au backend d’arrêter le traitement, affiche `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` et attend jusqu’à 120 s pour `Processing stopped.`, afin que `chloros-cli process … && next` ne lance pas `next` sur un backend encore occupé par l’exécution. Une deuxième pression sur Ctrl+C met fin à l’attente. `process` quitte alors `130`, comme le ferait n’importe quelle commande interrompue.

#### Un cycle qui n’écrit aucune image échoue

Si vous avez demandé des produits et que l&#x27;exécution n&#x27;en a écrit **aucun** — uniquement `project.json` et
`calibration_data.json` — `process` considère cela comme un échec : il affiche
`Processing finished but wrote no image products.` et **se termine avec un code de sortie différent de zéro**, ce qui permet à un script de
le détecter. Le message indique le nom du dossier du projet et les causes habituelles :

- le dossier d’entrée n’a pas été reconnu comme une capture (vérifiez la configuration et `--input-level`), ou
- tous les produits demandés ont été ignorés car inapplicables pour ces caméras (par exemple, demande de
  la radiance/réflectance à partir de caméras RGB uniquement).

Relancez l’opération avec `--verbose` et vérifiez le journal du backend à la recherche des lignes `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
qui expliquent les omissions par caméra qui n’apparaissent pas autrement dans la sortie de la CLI.

Une exécution délibérée « métadonnées uniquement » — tous les produits désactivés et pas de `--indices` — est tout de même un
**succès**, car une sortie d’image vide est le résultat attendu dans ce cas.

#### Une exécution dont les exportations n’ont pas pu être vérifiées échoue

Après une exécution, `process` vérifie les balises des fichiers exportés à l’aide d’ExifTool. Si ExifTool n’a pas pu s’exécuter, il
affiche « Échec de la vérification des exportations : ExifTool n’a pas pu s’exécuter, les balises de N fichiers exportés sur M
n’ont pas été vérifiées (voir les lignes [EXIFTOOL] dans le journal du backend)` and `Les produits sont enregistrés et restent
à leur emplacement ; relancez le traitement pour les vérifier.`, does not print `Traitement terminé !`, et **se termine avec un code de sortie non nul**. Les
produits sont conservés.

#### Quelles captures une exécution prend-elle en compte ?

Les captures terminées avant l’exécution de `process` sont incluses. Celles enregistrées pendant l’exécution ne le sont pas ; `process`
indique combien ont été exclues, et une nouvelle exécution les inclut.

#### Un cycle ayant perdu une image échoue

Si une image a échoué au cours d’une exécution, `process` affiche `N image(s) failed and were not exported` avec la
cause la plus courante, n’affiche pas `Processing complete!` et **se termine avec un code de sortie différent de zéro**. Les autres produits sont
enregistrés et restent en place, et le journal du backend répertorie chaque échec.

Il en va de même pour une **exécution avec le capteur de lumière uniquement** : un dossier d’enregistrements `.daq` ne contient, par définition, aucune image à exporter
et l’exécution est évaluée sur la base des fichiers calibrés `.daq` / `.csv` qu’elle a créés à la place.
S’il n’a **rien** écrit — tous les enregistrements ont été ignorés —, il affiche
`Processing finished but wrote no light-sensor products.`, nomme chaque enregistrement en indiquant la
raison pour laquelle il a été ignoré, et **se termine avec un code de sortie non nul**.

---

## `chloros-cli login`

Authentifier cette machine avec un compte cloud Chloros+. Les identifiants sont mis en cache de manière sécurisée dans `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemples

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (en supprimant ou en dupliquant certaines parties du mot de passe). En cas d’erreur 401, l’CLI effectue automatiquement une nouvelle tentative avec `$$`,-ajoute, puis avec une moitié du mot de passe sans doublons ; si une nouvelle tentative aboutit, elle vous connecte et affiche la syntaxe correcte avec des guillemets simples à utiliser la prochaine fois.

> **Utilisation sans interface graphique/par script : l’absence de session mise en cache entraîne l’affichage d’une invite de connexion.** Une commande principale (`process`, `status`, `export-status`, `time-sync`, …) exécutée sans session mise en cache demande `Email:` / `Password:` sur l’entrée standard (stdin) avant de poursuivre. Si l’entrée standard est fermée (cron, unité systemd), l’opération échoue immédiatement avec le code d’erreur `login cancelled` ; si l’entrée standard est un terminal ou un tube ouvert, le programme attend une entrée. Exécutez `chloros-cli login EMAIL PASSWORD` une fois par machine avant de planifier des tâches sans interface graphique.

---

## `chloros-cli logout`

Efface la session mise en cache et force une nouvelle connexion lors de la prochaine requête.

Se déconnecter alors qu’un traitement est en cours ne l’interrompt pas : le traitement se termine dans le mode dans lequel il a démarré, et le traitement suivant utilise le traitement standard. `logout` l’indique (`The processing run in progress finishes in <mode> mode; the next run uses standard`), tout comme l’application lorsque vous vous déconnectez de là.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Affiche le niveau de licence actuel (Iron/Copper/Bronze/Silver/Gold), l’utilisateur authentifié et le nombre de liaisons d’appareils.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Interroger la progression en temps réel de l’exportation Thread-4. Peut être appelé en toute sécurité **pendant** l’exécution de `process` depuis un autre shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Définit la langue d’affichage de l’CLI (38 langues prises en charge, y compris CJK, RTL et les langues indiennes). Reste en anglais de manière transparente sur les consoles héritées qui ne peuvent pas afficher le script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exemples

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Commandes relatives au dossier de projet

Ces commandes permettent de gérer l’emplacement par défaut du dossier de projet (partagé avec l’interface graphique).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` affiche le dossier dans lequel les nouveaux projets sont créés. Si le dossier configuré n&#x27;existe plus (s&#x27;il a été supprimé, renommé ou s&#x27;il se trouve sur un disque non connecté), l&#x27;CLI, tout comme l&#x27;interface graphique, utilise alors le dossier par défaut : le dossier par défaut est affiché sur stdout, un avertissement indiquant le nom du dossier manquant est envoyé vers stderr, et le code de sortie est `0`.

---

## `chloros-cli update`

Linux/Jetson uniquement. Vérifie `version_url` par rapport à `/etc/chloros/update.conf` et, lorsqu’une version plus récente existe, télécharge la version correspondante `.deb` et l’installe immédiatement avec `sudo dpkg -i` (sans demande de confirmation ; `sudo` peut vous demander votre mot de passe). Sur un Jetson, seule la mise à jour du système JetPack 6 est proposée. Sous Windows, la commande ne fait rien et affiche le message `up to date` : mise à jour avec un nouveau programme d&#x27;installation.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Sous Linux/Jetson, l’interface CLI effectue également une **vérification automatique des mises à jour à chaque démarrage** (sans blocage, cela ne retarde jamais l’exécution de la commande) : elle lit `/etc/chloros/update.conf`, met le résultat en cache pendant 1 heure dans `~/.chloros/update_cache.json`, et affiche `Update available: vX.Y.Z / Run: chloros-cli update` lorsqu’une version plus récente existe. Ignoré silencieusement en cas d’erreur et sous Windows ; `CHLOROS_DISABLE_UPDATE_CHECK=1` désactive cette fonctionnalité. En raison de la durée de mise en cache d’une heure, une version publiée au cours de cette heure peut encore être considérée comme à jour.

---

## `chloros-cli selftest`

Exécute un test de fonctionnement en 7 étapes : version, disponibilité des ports, démarrage du backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), présence du modèle de débruitage, état de préparation de CUDA + débruiteur. Il utilise un backend en cours d’exécution, ou en lance un pour le test avant de l’arrêter par la suite. Il affiche `N/7 checks passed` et les vérifications ayant échoué, et ne quitte `0` que lorsque les sept vérifications ont réussi ; le contrôle CUDA + débruiteur échoue sur un ordinateur dépourvu de GPU CUDA (voir [Vérifier l’installation](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Statut et contrôle du « grandmaster » PTP. L’hôte Chloros exécute le « grandmaster » PTP ; les caméras LATTICE et les unités DAQ-E lui sont asservies pour la synchronisation des horodatages entre les appareils (environ 1 ms entre les appareils). Le PTP aligne les **horodatages** : les caméras d’un réseau exposent simultanément parce que le déclencheur matériel M8 les active (en moins de 100 µs entre les modules), et non grâce au PTP. `chloros-cli time-sync` sans sous-commande exécute `status`.

| Sous-commande | Description |
| --- | --- |
| `status` | Affiche l&#x27;état du grand maître, les priorités BMCA, l&#x27;identité de l&#x27;horloge et indique si ses paquets atteignent le réseau : `health` correspond à `OK` (`STOPPED` lorsque le grand maître ne fonctionne pas), ou `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` avec un `health_detail`, puis une ligne `WARNING:` est affichée sur stderr (stdout reste au format JSON). `sync_sent` / `announce_sent` comptabilisent uniquement les messages qui sont partis ; `*_send_errors` et `last_send_error` comptabilisent ceux qui ne sont pas partis. Après une perte puis un rétablissement de la connexion d’une caméra, le grand maître reconstruit ses sockets de lui-même (`rebinds`, `last_rebind`). |
| `peers` | Liste des esclaves détectés via Delay_Req (caméras + capteurs DAQ-E). |
| `cameras` | État de santé PTP par caméra (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Redémarrage du processus grandmaster. |
| `set-priority --priority1 N --priority2 N` | Remplacement des priorités BMCA. |

Le grandmaster diffuse des annonces sur les cartes réseau des caméras (toute carte réseau dont l’adresse `169.254.x.x` est incluse). Si aucune n’est active, il se rabat sur la carte réseau de la route par défaut uniquement lorsque celle-ci est câblée, jamais sur une connexion Wi-Fi ou une carte réseau virtuelle (Tailscale, WSL), car aucune caméra ni aucun DAQ-E ne se trouve derrière une telle carte. En l’absence d’annonce à effectuer, il fonctionne sans sockets (`health` `NO_SOCKETS`), et la connexion de la caméra ou du DAQ suivante le reconstruit sur la carte réseau alors active ; `time-sync restart` le fait immédiatement.

Le « grandmaster » s’exécute dans son propre processus, à côté du backend (`own_process` / `process_pid` dans `time-sync status`), avec une priorité élevée sous Windows (`process_priority`), de sorte qu’un backend occupé (un traitement en cours, une exportation, plusieurs commandes simultanées) ne peut pas retarder les horodatages qu’il appose ; auparavant, un tel retard pouvait décaler l’horloge de toutes les caméras en même temps. Ce processus se termine avec le backend ; s’il ne peut pas être lancé, le grand maître s’exécute à l’intérieur du backend comme auparavant (`own_process` false, et le journal du backend l’indique).

### Exemples

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Contrôle des caméras LATTICE. Chaque sous-commande transite par le backend Chloros ; le backend gère le pool de caméras, de sorte que les appels CLIs suivants réutilisent le même descripteur ouvert.

### Options courantes (communes à la plupart des sous-commandes)

| Indicateur | Description |
| --- | --- |
| `-d, --device N` | Index de la caméra (par défaut : 0). |
| `-s, --serial SN` | Numéro de série spécifique ; remplace `--device`. |
| `--serials SN1,SN2,…` | Numéros de série séparés par des virgules pour un fonctionnement multi-caméras. Les [commandes transitoires multi-caméras](#multi-camera-transient-sessions) n’ouvrent que ces caméras, la première déclenchant la prise de vue. |
| `--all` | Agit sur toutes les caméras détectées. |
| `--exposure US` | Temps d’exposition en microsecondes. |
| `--gain DB` | Gain en dB. |
| `--pixel-format FMT` | Par exemple : `BayerRG8`, `BayerRG12` ; `Mono8`, `Mono12` sur un M3M. Le nom définit la profondeur de bits au sein de la famille de la caméra : sur un M3M, un nom de type « Bayer » définit le format « Mono » de sa profondeur (`BayerRG8` définit `Mono8`) ; sur un M3C, un nom de type « Mono » définit celui de type « Bayer » (`Mono12` définit `BayerRG12`). |
| `--width N` / `--height N` | Dimensions de l&#x27;image. |
| `--preset {default,high_quality,high_speed,triggered}` | Appliquer un préréglage : format de pixels, exposition, gain et fréquence d&#x27;images (`high_speed` : `BayerRG8`, 5 ms, 6 dB, 60 images par seconde ; `high_quality` : `BayerRG12`, 20 ms, 0 dB ; `lattice presets` les répertorie). Sur un M3M, le format est le format mono de même profondeur : `high_speed` capture `Mono8`, `high_quality` et `triggered` `Mono12`. Un indicateur fourni avec celui-ci (`--exposure`, `--gain`, `--pixel-format`) prévaut sur le préréglage, et un paramètre non spécifié par le préréglage conserve la valeur par défaut de la caméra. Toutes les exécutions en mode libre, sauf `triggered`, qui arme la caméra pour un front matériel sur la ligne 2 — si rien ne pilote cette ligne, elle attendra indéfiniment plutôt que de capturer. |
| `-o, --output DIR` | Répertoire de sortie (par défaut : `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Taille des paquets GVSP. `auto` exécute des sondes ICMP+GVSP ; `jumbo` = 9000 ; `standard` = 1 500. |

### Workflow de première connexion des caméras LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Référence des sous-commandes

#### Découverte et informations

| Sous-commande | Objectif |
| --- | --- |
| `lattice info` | Liste des caméras connectées (fabricant, modèle, numéro de série, adresse IP, adresse MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analyse le système hôte pour une configuration optimale de la caméra. `--no-discover` ignore la détection des caméras (plus rapide, analyse par carte réseau uniquement). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Vérification/correction des paramètres de la carte réseau (sous Linux, également les tampons de réception des sockets du noyau — voir [hôtes Linux](#linux-hosts--socket-receive-buffers)) ; estimation de la bande passante et du nombre d&#x27;images par seconde (FPS). |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Recommandation de capacité réseau du backend à schéma stable+ recommandation de tableau (renvoie `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` utilisent par défaut le capteur complet, `2048` × `1536`. Il quitte `0` pour `ok`, `auto_capped_fps` et `auto_shrunk`, puis de `1` vers `needs_force_slip` et `error`, avec ou sans `--json`. Il atteint le backend à `--backend-url` (par défaut `http://127.0.0.1:5000`), et non à `CHLOROS_BACKEND_URL`. `auto_capped_fps` conserve la résolution demandée mais limite le nombre d’images par seconde cible — lisez `recommended.recommended_target_fps` et transmettez-le comme cible de connexion ; considérez cela comme un succès, et non comme une erreur. `auto_shrunk` n’apparaît que lorsque l’image demandée ne peut être fournie à aucune fréquence d’images ; la connexion ne réduit jamais les images de son propre chef. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analyse de scénarios sans ouvrir les caméras (`--width` / `--height` par défaut `2048` × `1536`). Ces deux options, ainsi que `network-analysis`, projettent ce que connect va diffuser : sans `--pixel-format`, les formats par caméra appliqués par « connect » (Mono12 / BayerRG12), et avec `--packed auto` (par défaut), la décision que « connect » prendrait concernant le [transport 12 bits compressé](#transport-12-bits-compressé---compressé) que Connect prendrait sur cet ordinateur — le nombre d’images par seconde (fps) indiqué correspond donc à la fréquence à laquelle le réseau fonctionne. Les deux affichent le choix de compression. **`--n-active` correspond au nombre total de caméras sur le réseau, et pas seulement à celui de ce réseau**— augmentez-le lorsque des caméras autonomes transmettent simultanément, ou lorsque le budget du réseau est calculé par rapport à une demande qui les sous-estime (par défaut : `len(--models)`). Affiche toujours les lignes agrégées `Wire budget:` (Mo/s demandés par rapport au plafond sans risque de collision) et `Max cameras:`, et signale `** OVER-SUBSCRIBED**` lorsque le réseau est surchargé — voir [Modèle de fps et de rafales du réseau de cartes](#array-fps--burst-model). |
| `lattice gpu` | Afficher l’état du GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Vérifier ou mettre à jour le micrologiciel de la caméra. La sélection locale `.fwa` est verrouillée : le fichier situé dans `firmware/<MODEL_PREFIX>/` correspondant au `MIN_FIRMWARE_VERSION` de la version est flashé lorsqu’il est présent (la version la plus récente uniquement en tant que solution de secours) ; ainsi, une image du fournisseur plus récente stockée sur le disque reste inactive jusqu’à ce que ce pin soit modifié — les versions plus récentes sont délibérément déployées sur les appareils via le manifeste AWS signé, ce qui est préférable lorsqu’elles sont plus récentes. |
| `lattice presets [--apply NAME]` | Liste ou application des préréglages de caméra. `--apply` définit les caractéristiques du préréglage (format de pixels, exposition, gain, fréquence d’images, déclenchement) et affiche ces paramètres tels que la caméra les lit après : sur un M3M, le format Mono de la profondeur du préréglage ; la fréquence d’images à laquelle il fonctionne, qui peut être inférieure à celle demandée par le préréglage (`high_speed` demande 60 images par seconde ; une caméra plein format sur une liaison à 1 Gbit/s fonctionne à environ 35). Tous les autres paramètres écrits par « open » sont restaurés tels qu’ils ont été trouvés. `--apply default` réinitialise la caméra à ses paramètres d’usine par défaut : ses réglages d’usine avec exposition automatique en mode libre sur la ligne 2, la limite de débit de la liaison et la correction des défauts désactivées, la balance des blancs calibrée d’une caméra RGB conservée (valeur unitaire sur toute autre caméra). Rien n’est enregistré dans la mémoire de la caméra ; ainsi, lors de sa prochaine mise sous tension, elle redémarre à partir de sa propre mémoire comme auparavant. En tant que commande de capture ou de connexion, `--preset` et `default` ne constituent pas des préréglages. |
| `lattice status` | État de la caméra en direct. |

#### Capture

| Sous-commande | Objectif |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Image unique. **Enregistre tous les types d’exportation par défaut**(`--processing all`) ; voir [Niveaux d’exportation de capture](#capture-export-levels-the-all-default). `--processing` accepte `all`, `raw`, `debayered`, `radiance`, `reflectance` ou `display` (`preview` est accepté pour `display`) ; tout autre nom est refusé avant que la caméra ne soit activée (sortie `1`). `--levels` enregistre un sous-ensemble explicite (remplace `--processing`). Une caméra que Chloros ne détient pas est ouverte pour la capture, et son exposition est d’abord définie à partir de la scène (quelques secondes tout au plus), comme le font les [commandes transitoires](#multi-camera-transient-sessions) ; lorsque la capture prend fin, quelle qu’en soit la manière, cette caméra est rétablie dans l’état où elle a été trouvée (format, exposition, pipeline de couleurs, tous les paramètres modifiés par la capture — la liste figurant sous [Multi-caméra (sessions transitoires)](#multi-camera-transient-sessions)). La lecture DAQ de la capture utilisée est enregistrée sous la forme d’un fichier sidecar `.daq`, comprenant une capture au format brut uniquement ; `--no-daq` / `--daq-tolerance-ms` permettent de l’ajuster ([Le DAQ qui lit une capture enregistre](#the-daq-reading-a-capture-saves)). `--force-daq` est toujours accepté. `--jpeg-quality` = qualité JPEG 1–100 (par défaut 95).**Les fichiers enregistrés dans le projet ouvert sont ajoutés à celui-ci.** Comme dans l’application, une capture dont les fichiers sont enregistrés dans le dossier du projet ouvert dans Chloros les ajoute à la liste des fichiers de ce projet ; les fichiers enregistrés ailleurs ne sont pas modifiés. La commande affiche une ligne l’indiquant (`Project: ...`) ou, si un traitement est en cours, qu’ils sont mis en attente et intégrés à la fin du traitement. Il en va de même pour `multi-capture`, `array-capture` (ainsi que `--smart`) et `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Transfert vers le disque jusqu’à la combinaison Ctrl+C. Une exécution `tiff`constituent le fichier `raw`, un tableau dans lequel la capture écrit les données de cette caméra : la mosaïque du capteur, un plan avec ses propres valeurs, non compressé, avec le paquet d’étalonnage (une image de 2 048 x 1 536 en 12 bits fait 6,0 Mio plus le paquet, quelle que soit la scène), c’est donc à partir de celles-ci que commence `process`. `png` et `jpg` contiennent 8 bits ; il s’agit donc d’images démoisicées destinées à l’affichage, dont `process` ne tire pas son origine. Avec une liaison DAQ (`daq pool-bind`) ou en pool, chaque image dispose de ses propres fichiers `.daq` et `.match.json` dans le dossier `daq/` de la série, car `lattice capture` en enregistre une ; `--no-daq` désactive cette fonction, et l&#x27;arrêt affiche le nombre de trames. Chaque image est enregistrée une seule fois, avec l&#x27;intégralité de ses métadonnées ; lorsque l’on appuie sur Ctrl+C, les images déjà capturées sont enregistrées avant que la caméra ne soit relâchée. Le programme s’arrête de lui-même lorsque le disque est plein, affiche `Failed:` / `Disk full:` et quitte `1` ; toute image qu’il n’a pas pu enregistrer entraîne également la sortie de `1`. Un appareil photo laissé en mode déclenchement matériel (`TriggerMode` `On`) est réglée en mode libre pour la série. À la fin de la série, la caméra est remise dans l’état où elle a été trouvée — ses paramètres de déclenchement et tous les réglages modifiés par la série (exposition, gain, format, …) — sauf si `--keep-settings` est utilisé, ce qui enregistre les paramètres de la session dans la mémoire propre de la caméra (mode libre), ou si `--clear-settings` est utilisé, ce qui réinitialise cette mémoire aux paramètres d’usine. Une session qui n’enregistre aucune image est considérée comme un échec : en l’absence d’image pendant 10 s, elle s’arrête d’elle-même, affiche `No frames:` avec la raison et quitte `1`. Les images que les modules d’écriture n’ont pas pu traiter sont comptabilisées comme perdues, et l’arrêt affiche `Dropped:` avec la raison (fréquence d’images de la caméra, des modules d’écriture et de la file d’attente) ; une fréquence d’images inférieure ou un format de pixel 8 bits permet de toutes les conserver. La commande effectue une vérification auprès du backend toutes les 2 s ; si elle se termine sans avoir arrêté l’enregistrement (`kill -9`, mémoire insuffisante), le backend l’arrête 30 s après la dernière vérification et remet la caméra en (`stop_reason` `client_gone`, et une ligne dans le journal du backend). `multi-stream` et `align-stream` font de même. Un démarrage dont l’ouverture de la caméra prend plus de temps que le délai d’attente de la commande (60 s ; 180 s pour `multi-stream` / `align-stream`) — une caméra encore en cours de démarrage — est annulé par le backend : la caméra est réinitialisée et fermée, et aucun enregistrement n’est laissé en cours (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Aperçu MJPEG en direct via le navigateur : se connecte à la caméra, affiche l’URL du flux et l’ouvre dans votre navigateur. `--ae-damping` définit l’amortissement de l’exposition automatique (0,4–100). |

##### Flux en direct (MJPEG ; pas de RTSP)

Chloros diffuse la vidéo en direct au format **MJPEG sur HTTP**(`multipart/x-mixed-replace`, limite `frame`) depuis le backend.**Le protocole RTSP n’est pas pris en charge.** Ouvrez une URL dans un navigateur, ou lisez-la avec n&#x27;importe quel client MJPEG (`cv2.VideoCapture(url)` d&#x27;OpenCV, ffmpeg, VLC). La caméra doit d’abord être connectée au backend (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` ou via l’interface graphique).

| URL (`http://127.0.0.1:5000` + …) | Flux |
| --- | --- |
| `/api/camera/stream/<serial>` | L’aperçu à l’écran, jusqu’à environ 30 images par seconde. Pour un élément du tableau, `?max_h=` (par défaut `720`) et `?jpeg_q=` (par défaut `80`) définissent la hauteur et la qualité JPEG. Il s&#x27;agit de l&#x27;URL que `lattice viewer` ouvre. |
| `/api/camera/<serial>/stream-annotated` | Le même aperçu avec les superpositions de l&#x27;interface graphique affichées : `zebra=1`, `crosshair=1`, `grid=3x3` (ou `9x9`), `histogram=1`, `peaking=1`, `spot=1` ; sans demande, les paramètres de superposition enregistrés par le projet pour cette caméra. |
| `/api/camera/<serial>/stream-reflectance` | Réflectance de 8 bits par bande : ρ = pixel ÷ la partieen-tête `X-Chloros-Pixel-Scale` (255 sauf si `refl_max` est déclenché). `fps` (par défaut `10`), `jpeg_q` (par défaut `95`), `max_h` (par défaut `0` = natif), `band=<name>`, `align=1`. Format JPEG, donc avec perte. Refusé avec un code d&#x27;état HTTP409 lorsque la caméra n&#x27;a pas de réflectance (caméra RGB, pas de pack d&#x27;étalonnage, pas de lecture DAQ). |

- **Correspondance des images entre les caméras.**Chaque élément envoyé par un**membre du tableau**, sur les trois flux, comporte les en-têtes `X-Chloros-Group-Id` (le compteur de groupe synchronisé du tableau), ainsi que les en-têtes `X-Chloros-Frame-Id` et `X-Chloros-Timestamp-Ns`. Les parties provenant de caméras différentes mais portant le même `X-Chloros-Group-Id` proviennent du même déclencheur. Une caméra autonome n’appartient à aucun groupe et n’envoie aucun identifiant de groupe. Les navigateurs ignorent les en-têtes propres à chaque partie ; c’est le client qui analyse lui-même le flux multipart qui les lit.
- **En pause tant que la vue de la matrice est ouverte.** Le `/api/camera/stream/<serial>` d’un élément de la matrice envoie une image statique « en pause » tant que le flux en direct combiné de la matrice est ouvert (par exemple, la vue de la matrice dans l’interface graphique), et reprend lorsque celui-ci se ferme.

#### Réglage des capteurs

| Sous-commande | Objectif |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Lecture/écriture de n&#x27;importe quel nœud GenICam. `--set ExposureTime=…` (`Gain=…`) sans ses commutateurs `ExposureAuto` (`GainAuto`) qui désactivent d’abord le mode automatique, comme le fait `lattice exposure --set` ; une valeur refusée par la caméra affiche `Failed to set <node>: …` sur stderr et quitte `1`. `--get` et la vue groupée par défaut affichent un message `note:` sous `ExposureTime` / `Gain` pendant que ce mode automatique est actif — voir [Lecture de l’exposition et du gain en mode automatique](#reading-exposure-and-gain-en-mode-automatique). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Exposition et AE. Une exposition `--set` ou une cible `--brightness` est conservée comme la vôtre (le contrôle d’exposition automatique de Chloros ne la modifie pas) ; le gain reste inchangé, de sorte qu’en mode automatique, le système continue de s’ajuster vers la cible jusqu’à ce que vous définissiez `lattice gain`. `--auto` rend le contrôle de l’exposition, `--brightness-auto` rend la cible au système automatique d’Chloros. En l’absence de drapeaux, il lit les nœuds d’exposition, avec les mêmes paramètres `note:` et `configure --get`. Si la caméra refuse une valeur `--set`, elle affiche `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` sur stderr — en mode automatique, lorsque l&#x27;écriture a été désactivée, celle-ci est rétablie plutôt que de rester désactivée — n&#x27;affiche aucune ligne `Exposure: … us (auto OFF)` et quitte `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Gain et gain automatique. Si la caméra refuse une valeur `--set`, elle affiche `Failed to set Gain: …` sur stderr, n&#x27;affiche aucune ligne `Gain: … dB (auto OFF)` et quitte `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Zone d’intérêt (ROI) du capteur et regroupement de pixels. Le regroupement de pixels est toujours **moyen** — voir ci-dessous. |
| `lattice format [--set FMT] [--list]` | Format des pixels. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Déclenchement matériel/logiciel. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (aucun indicateur = balance des blancs en mode « one-shot ») | Opérations de balance des blancs. Caméras RGB/Bayer uniquement ; opération nulle (ignorée) sur les M3M monochromes. Le mode « one-shot » conserve la balance des blancs qu’il calcule (`BalanceWhiteAuto` et les gains) ; tous les autres paramètres dont il modifie la valeur sont rétablis tels qu’ils étaient, et l’ensemble des paramètres est réinitialisé lorsque le mode « one-shot » ne peut pas s’exécuter. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Pipeline de couleurs d’affichage RGB. `natural` (par défaut) correspond au traitement en temps réel le plus simple ; `enhanced` ajoute la suppression des franges + la vibrance + le contraste local CLAHE pour un rendu « hub-parity » complet, à un coût de traitement par image environ deux fois plus élevé, ce qui entraîne une fréquence d’images **en direct** plus faible — les captures enregistrées bénéficient toujours du traitement complet dans tous les cas. Caméras RGB/Bayer uniquement ; ignoré sur le M3M mono. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Saturation/contraste en prévisualisation en direct (caméras à filtre RGB) ; `100` est neutre, `--reset` règle les deux paramètres sur 100. Les captures enregistrées ne sont pas affectées. Ignoré sur les caméras M3M mono. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (aucun indicateur = lecture) | Affichage des **hautes lumières** (caméras à filtres RGB) : atténue les pixels les plus lumineux et les plus blancs — par exemple, une surface blanche brillante qui s&#x27;affiche comme une tache blanche uniforme. `0` = désactivé, `-100` = intensité maximale. L’aperçu en direct et les captures au niveau de l’affichage s’y conforment ; la radiance/réflectance ne le font jamais, et cette option ne permet pas de récupérer les données du capteur qui ont été écrêtées (réduisez l’exposition pour cela). Ignoré sur le M3M mono. |
| `lattice filter [--list]` | Liste des modèles de filtres connus d’Chloros (`RGN-IMX265`, `OCN`, `NGB`, …). Le filtre d’une caméra est lu à partir de sa chaîne de modèle et ne peut pas être défini : `--set NAME` se contente de rechercher un modèle, puis quitte `1` avec `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Teste les nœuds d’alimentation/thermiques ; active/désactive le mode veille à faible consommation. `--sleep` laisse la caméra en veille (la limite de débit de la liaison est activée (`DeviceLinkThroughputLimitMode` `On`) à son seuil minimal, le mode d’économie d’énergie est activé si la caméra en dispose) tous les autres paramètres de sa connexion sont rétablis tels qu’ils étaient initialement ; un passage en veille qui échoue est réinitialisé dans son intégralité. Le réveil rétablit la liaison telle que la veille l’a trouvée (la limite, son mode et le délai de paquet `GevSCPD`, que la caméra maintient à 0 tant que la limite est activée ; l’enregistrement est conservé sur cet ordinateur, de sorte qu’un réveil après le redémarrage du backend le retrouve également), ou, pour une caméra que cet ordinateur n’a pas mise en veille mais qui semble en veille (sa limite activée au seuil minimum), avec la limite désactivée et un `GevSCPD` laissé à 0 à 80 (réglages d’usine) ; une limite définie au-dessus du seuil minimal est laissée telle quelle. La sonde et le réveil rétablissent ce que leur ouverture a écrit. |

> **Le binning est toujours de type « Average ».** `lattice resolution --binning N` définit le binning « Average » et
> ne propose pas de changement de mode. Le regroupement en somme multiplie chaque DN par N², ce qui coûte environ deux diaphragmes de
> marge dynamique dans les hautes lumières par étape de regroupement ; Chloros divise ce facteur avant la
> chaîne radiométrique, mais les pixels que le regroupement en somme a poussés jusqu’au plafond du capteur sont écrêtés et ne peuvent pas être
> récupérés. Le binning « Sum » n’apporte rien de plus que le binning « Average » : un indice est invariant d’échelle et les
> scalaires de radiance sont calibrés à l’échelle « Average ». Si une caméra est livrée déjà réglée sur « Sum » (à la suite
> d’une intervention d’un autre outil ou d’une réinitialisation d’usine), Chloros le détecte lors de la relecture des nœuds et
> compense ; l’exécution de `lattice resolution --binning N` la remet en mode « Average ». La version 1.2.0
> `--binning-mode` est toujours accepté, ce qui permet aux anciens scripts de s’exécuter, mais il ne génère qu’un avertissement :
> le binning reste en mode « Average ».

> **La modification du regroupement, du format de pixel et du déclencheur sur une caméra n’est pas prise en compte.** Sur une caméra dont le
> backend reste ouvert et diffuse en continu (`cam-connect`, l’interface graphique), `lattice resolution --binning`
> ne prend pas effet (il affiche que le regroupement de pixels est toujours en vigueur, `Binning: 1x1`), et
> `lattice format --set` et `lattice trigger --mode` sont refusés par la caméra
> (`SC_ERR_ERROR -1001`). Libérez-le d’abord (`lattice cam-disconnect -s <SN>`), ou configurez-les
> lors de la connexion (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Lecture de l’exposition et du gain en mode automatique

Lorsqu’un nœud `ExposureAuto` (`GainAuto`) d’une caméra est en cours d’exécution, son nœud `ExposureTime` (`Gain`) conserve la **dernière valeur écrite**, et non l’état actuel de la caméra. Pour une caméra dont le backend reste ouvert (`cam-connect`, `array-connect`, l’interface graphique), `lattice configure --get`, la vue par défaut `lattice configure` et `lattice exposure` (sans indicateurs) affichent donc la valeur du nœud et, en dessous, une note indiquant la dernière image de la caméra :

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Lisez la valeur de l’image dans la note ; la valeur du nœud correspond uniquement à la dernière écriture. Via HTTP, la même réponse `POST /api/camera/<serial>/read-nodes` laisse `nodes` inchangé et ajoute `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, celle de la trame la plus récente) et `notes` (la phrase ci-dessus). Ces deux éléments ne sont présents que si le mode automatique n’est pas `Off` et qu’une trame est connue ; une caméra ouverte uniquement en lecture n’a pas d’image et n’obtient ni l’une ni l’autre. `lattice gain` (sans indicateurs) affiche uniquement le nœud — utilisez `lattice configure --get Gain` pour la note.

##### Exposition sur un élément du réseau

Pendant qu’un réseau transmet en continu, chaque élément fonctionne avec **`AcquisitionFrameRateEnable`**, à une fréquence d’images légèrement supérieure à la fréquence de déclenchement du tableau : sa période correspond à la période de déclenchement moins 15 % (au moins 3 ms), ou à la fréquence maximale propre à la caméra lorsque le déclenchement est déjà proche de cette valeur. `configure --get` sur un élément lit donc `AcquisitionFrameRateEnable` comme `True` et `AcquisitionFrameRate` avec ce réglage ; l’activation est désactivée lorsque le réseau s&#x27;arrête.**N’écrivez pas ces deux nœuds sur un élément de matrice** — la matrice les synchronise avec son déclenchement et les réécrit.

Ce réglage correspond également à la durée d’exposition maximale de l’élément : environ 85 ms pour un déclenchement à 10 images par seconde (période de 100 ms), quel que soit **le ROI ou le binning**. Avant la version 1.3.0, une petite ROI limitait l’exposition d’un élément à la période de fonctionnement libre du capteur (4,9 ms à 768 × 304), quelle que soit la lenteur du déclenchement du réseau.**Un `ExposureTime` manuel plus long que le plafond ralentit le déclenchement pour s’y adapter** lorsqu’il est écrit avec `lattice exposure --set` ou `configure --set` (ou depuis l’interface graphique). Le réseau ouvre d’abord la limite supérieure de ce membre pour cette valeur et laisse la fréquence plus lente fonctionner pendant quelques périodes de déclenchement, ce qui peut allonger l’écriture de quelques secondes ; le déclenchement ne se produit alors pas plus rapidement que ne le permet l’exposition. Sur un réseau à fréquence automatique, la fréquence à laquelle il fonctionnait auparavant est rétablie dès que chaque membre dont l’exposition manuelle l’avait ralenti est revenu en exposition automatique. Une valeur que la caméra elle-même ne peut pas accepter est toujours refusée, avec la plage qu’elle accepte indiquée dans le message d’erreur, et l’exposition automatique du membre est rétablie si elle avait été désactivée uniquement pour cette écriture. Une recette d’exposition `apply` ne ralentit pas le déclencheur : celui-ci est maintenu à la limite supérieure. Chloros ne raccourcit jamais une exposition manuelle pour s’adapter à un déclenchement plus rapide ; une fréquence verrouillée (`array-connect --target-fps F`) plus rapide que ne le permet une exposition laisse à ce membre une durée maximale suffisante pour celle-ci, et ce membre ne déclenche pas plus rapidement que ne le permet son exposition.

#### Étalonnage et capteurs

| Sous-commande | Objectif |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Étalonnage à partir d’une cible de réflectance. Jusqu’à `--attempts` trames (par défaut `30`) sont analysées à cette fin ; une trame incomplète (perte de paquet GVSP) coûte une tentative, et non l’exécution. L’opération échoue avec `no target detected in N attempts`, ou à `no complete frame in N attempts: <last error>` si aucune trame n’est arrivée intacte. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Le spectre et les amplitudes de bande d’un DAQ-M, lues via Bluetooth (BLE) par le backend. `--mac` correspond à l’adresse MAC BLE propre au capteur et est requise pour l’une des trois actions : il n’y a pas de valeur par défaut, car chaque unité possède la sienne (`daq pool-discover --only ble` les répertorie) ; sans cette valeur, la commande se termine par `1`. En l’absence d’action, elle affiche son aide. `--camera SERIAL` intègre les amplitudes de bande affichées à la réponse spectrale mesurée propre à cette caméra, plutôt qu’à la courbe de flotte du modèle de filtre — caméras RGB, lorsque le pack d’étalonnage de la caméra est chargé (connecté) ou mis en cache sur cet ordinateur. L’en-tête indique quelle courbe a été utilisée : `Band magnitudes (camera <SERIAL>):`, ou `Band magnitudes (model <filter>):` lorsque la caméra n’en possède pas (il ne s’agit pas d’une caméra RGB, ou si aucun pack n’est chargé ou mis en cache), et c’est la courbe par défaut qui a été utilisée à la place. Via HTTP, `POST /api/dls/cli-read` prend `camera_serial` et renvoie `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Applique une correction de vignettage aux images existantes, chacune avec le champ plat de son propre objectif (issu de son modèle EXIF). `--lens-model` s’applique aux images dont le modèle ne mentionne aucune caméra ; une image dont le modèle contredit cette condition est ignorée. |

#### Multi-caméras (sessions transitoires)

Une commande transitoire ouvre les caméras pour une seule exécution et les ferme à la fin de celle-ci : `multi-capture` sans tableau persistant associé, `multi-stream` et `multi-test` ici, et `align-calibrate`, `align-apply`, `align-stream` et `index --live` ci-dessous. Les règles suivantes s’appliquent à toutes ces commandes :

- **`--serials SN1,SN2,…` sélectionne les caméras.** Seules celles-ci sont ouvertes, et la première déclenche l’action (pour `align-calibrate` et `--reference` lorsqu’elles sont spécifiées). Une caméra série qui n’est pas sur le réseau entraîne l’échec de l’exécution avant même qu’une caméra ne soit ouverte. Sans `--serials`, toutes les caméras du réseau sont ouvertes. Une caméra déjà détenue par Chloros (via l’interface graphique, `cam-connect`, `array-connect`) refuse l’exécution, que `--serials` la nomme ou non : elle partage le réseau pour lequel l’exécution est prévue. (`index --live` sans `--serials` ouvre à la place les caméras activées de son profil : voir [Index / Mathématiques de la végétation](#index--vegetation-maths).)
- **Les caméras partagent le réseau de la même manière que `array-connect` le partage.** Chaque caméra est synchronisée en fonction de sa part du trafic transporté par la liaison hôte (un délai inter-paquets, et non la limite de débit de la caméra, dont le seuil minimal de 31,25 Mo/s est déjà dépassé par quatre caméras sur une liaison 1 GbE), de sorte que quatre caméras fonctionnent sur une liaison de 1 Gbps. Lorsque les caméras ne peuvent pas toutes envoyer simultanément au rythme le plus lent qu’elles atteignent, elles sont déclenchées l’une après l’autre, chacune disposant d’une liaison dédiée, et la commande affiche `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Dès qu’une seule caméra à la fois ne peut pas s’intégrer, l’exécution est refusée avant que la moindre caméra ne transmette : `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, sortie `1`.
- **Toutes les caméras doivent transmettre, sinon l&#x27;exécution échoue.** Une caméra qui ne transmet aucune image constitue une erreur : la commande affiche `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, où `<why>` correspond à l’erreur de récupération de cette caméra, puis à la cause identifiée, et se termine par `1`. Une image arrivée avec une perte de paquets GVSP (`IncompleteBufferError: GVSP packet loss …`) signifie que le réseau ne supporte pas la bande passante combinée des caméras (`Every missing frame arrived with GVSP packet loss: …`): vérifiez la vitesse de liaison de l’hôte et de la liaison montante du commutateur (`lattice network`), utilisez des trames jumbo, ou ouvrez moins de caméras avec `--serials`. Une caméra qui n’a rien transmis ne reçoit généralement pas le signal de déclenchement du maître : vérifiez son câble de synchronisation M8, puis exécutez `lattice multi-detect`. La fonction de chaque commande lorsqu’un groupe de trames manque une caméra est indiquée dans la ligne correspondante.
- **Chaque caméra est réinitialisée dans l’état où elle a été trouvée.** À la fin de l’exécution, chaque caméra retrouve l’état dans lequel elle se trouvait au moment de son ouverture : `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, et `LineMode` / `LineSource` sur `Line2` et `Line3` (et le filtre anti-parasites de l&#x27;entrée de déclenchement d&#x27;un tableau fournit une ligne d&#x27;esclave, `LineFilterSelector` / `LineFilterWidth`) ; son `PtpEnable` (et les `PtpSlaveOnly` / `PtpDomainNumber` définis avec lui) ; `ExposureAuto` / `GainAuto` (ainsi que `ExposureTime` / `Gain`, où la valeur était définie manuellement) ; `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping` ; `BalanceWhiteAuto` (et chaque `BalanceRatio` où la balance des blancs était manuelle) ; `BlackLevelAuto` / `BlackLevel` ; `PixelFormat`, regroupement de pixels, `Width` / `Height` / `OffsetX` / `OffsetY` ; `GammaEnable` (et `Gamma` lorsque le gamma était activé), `LUTEnable`, `ColorTransformationEnable` ; `ReverseX` / `ReverseY` ; `UserSetSelector` ; la taille du paquet (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`) ; le rythme, `DeviceLinkThroughputLimitMode` (et `DeviceLinkThroughputLimit` lorsque le mode était `On`), lepaquet `GevSCPD` et le décalage de démarrage `GevSCFTD` qu’un tableau attribue à chaque caméra ; les données de bloc que chaque flux active (`ChunkModeActive`, chaque `ChunkEnable` sous son `ChunkSelector`) ; `AcquisitionFrameRateEnable` (et `AcquisitionFrameRate` lorsque l’activation était activée — lorsqu’elle est désactivée, le débit est en lecture seule et indique la valeur maximale autorisée par la cadence, ce n’est donc pas un paramètre). Seules les valeurs différentes sont enregistrées : la caméra s’est d’abord arrêtée, puis la cadence et la fréquence d’images avant une exposition nécessitant leur période d’image (une caméra limite `ExposureTime` à une période d’image). Avec `--keep-settings`, le câblage du déclencheur et de la ligne de synchronisation est rétabli, la caméra fonctionne en mode libre, et le reste est enregistré dans la mémoire propre de la caméra selon les paramètres définis par la session ; `--clear-settings` y rétablit les paramètres d’usine.
- **L’exposition est définie à partir de la scène avant que la première image ne soit conservée.** La séquence capture des groupes d’images synchronisés et les écarte tout en réglant l’exposition et le gain de chaque caméra à partir de ces images. L’objectif est d’obtenir une luminosité moyenne sur la cible de la caméra avec un écrêtage ne dépassant pas 1 % de l’image ; une scène présentant des hautes lumières intenses ne permet pas d’atteindre cette cible. Cela nécessite généralement 2 à 6 groupes d’images, et prend au maximum 5 s.
  - Chaque caméra vise la cible à partir de laquelle un réseau connecté la lance, selon son filtre : TargetBrightness **40**sur une caméra multispectrale (RGN, OCN, NGB) et**60** sur une caméra RGB. Cela correspond à la moitié de la cible d’exposition donnée par 80 (les trois quarts sur une caméra RGB), ce qui réduit le risque de coupure sur un panneau à forte réflectance. `--brightness N` (0-255) définit à la place la cible de chaque caméra (`multi-test` n’a pas de `--brightness` : son exposition automatique est désactivée). Le journal du backend nomme la cible de chaque caméra : `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - L’exposition est déterminée en premier, jusqu’à la limite d’exposition automatique propre à la caméra. Le gain ne couvre que ce qui reste.
  - L’exposition et le gain déterminés sont conservés pendant toute la durée de l’exécution ; ainsi, chaque image d’une caméra `multi-stream` est prise avec la même exposition. Une capture devant s’adapter à des variations de luminosité doit être effectuée sur un tableau connecté (`array-connect`), dont l’exposition automatique s’ajuste en permanence.
  - Les séquences `--exposure` et `--gain` sont conservées exactement telles qu’elles ont été définies. Lorsque les deux sont spécifiés, aucune image n’est écartée.
  - À la fin de la série, l’exposition automatique de chaque caméra est réactivée.
  - Le journal du backend rapporte le résultat : `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` ou `exposure NOT solved after …` avec la raison, et `(not settled)` pour chaque caméra concernée. Dans ce cas, les images sont tout de même capturées.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` modifie la limite de 5 s ; `0` désactive cette fonctionnalité (voir [Variables d’environnement](#environment-variables)).

**Tant que Chloros détient une caméra** (l’interface graphique, `cam-connect` ou un réseau connecté), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` et `index --live` refusent de s’exécuter, car ils ouvrent toutes les caméras du réseau et une deuxième ouverture d’une caméra bloquée peut provoquer le plantage du backend. (`index --live` échoue lorsqu’une caméra qu’il tenterait d’ouvrir est bloquée : il s’agit de `--serials` ou des caméras de son profil, ainsi que de toute caméra d’un profil qui n’en spécifie aucune.) Le backend renvoie une réponse HTTP409 à `CAMERA_HELD`, en indiquant chaque caméra bloquée ainsi que les commandes de déblocage (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), puis la commande se termine (`1`). Si un tableau est connecté, `array-capture --continuous` l’enregistre à la place de `multi-stream`, et `array-capture` (aligné par défaut) enregistre un ensemble aligné à la place de `align-apply` ; `multi-capture` et `align-calibrate` utilisent un tableau connecté qui contient leurs caméras au lieu de les ouvrir.

| Sous-commande | Objectif |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Répertorie toutes les caméras ayant un rôle de synchronisation ; avec `--serials`, uniquement celles-ci, et la première avec `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Une image synchronisée par caméra. Enregistre **tous les types d’exportation par défaut**lorsqu’un tableau persistant est connecté, ainsi que la lecture DAQ utilisée ([Ce qu’enregistre la lecture DAQ lors d’une capture](#the-daq-reading-a-capture-saves)) ; la solution de secours transitoire sans matrice est**uniquement débayérisée** (exécutez d’abord `array-connect` pour le reste). `--brightness` s’applique uniquement au mode de secours : un réseau connecté conserve sa propre exposition automatique. Le mode de secours n’enregistre pas `.daq` (`--no-daq` / `--daq-tolerance-ms` n’ont aucun effet ici), et `multi-capture` ne prend pas en compte `--force-daq`. Dans le mode de secours, une caméra qui ne fournit aucune image échoue la capture (`Error: multi-capture failed: no frame from …`, sortie `1`) et aucun fichier de ce groupe d’images n’est enregistré. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Flux d’images synchronisées (transitoire), `--fps` par défaut `3`, jusqu’à ce que l’on appuie sur Ctrl+C ou que l’on atteigne les groupes `--count`. Un groupe d’images pour lequel une caméra n’a rien transmis n’est pas écrit, et trois groupes consécutifs mettent fin au flux. Si un groupe a été ignoré, l’exécution se termine par l’affichage de `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, ainsi que de `Error: the stream stopped: …` indiquant le nom de la caméra qui a mis fin au flux, puis quitte `1`X. Il s&#x27;arrête automatiquement lorsque le disque est plein, affiche `Failed:` / `Disk full:` et quitte `1` ; tout fichier qu’il n’a pas pu écrire provoque également la sortie de `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test de synchronisation GPIO (transitoire), `--count` déclencheurs par défaut `10`. Affiche la manière dont le réseau est partagé (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, et la ligne `Serial emit:` lorsque les caméras sont déclenchées les unes après les autres, dont le décalage est alors pris en compte dans les horodatages), une ligne `Sync group: master SN:<serial> -> SN:<serial>, …` par groupe de synchronisation (`(no slaves: not synced)` pour une caméra sans câble de synchronisation, qui se déclenche de manière autonome), puis une ligne par déclenchement : `[  0] SN:<serial>=<timestamp> \| …`, avec `SN:<serial>=(no frame)` pour une caméra n’ayant rien transmis lors de ce déclenchement. Puis une ligne `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` et une ligne `SN:<serial>: mean <±x> us, jitter <y> us` par caméra, chaque caméra par rapport à la caméra maître au cours d’un cycle d’horloge : un déclenchement dont les horodatages mélangent les horloges des appareils des caméras et le PTP (PTP désactivé ou encore en cours de verrouillage) est ignoré, et une ligne `Note:` les comptabilise. Toute image manquante entraîne l’échec du test (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, sortie `1`) ; trois déclenchements incomplets d’affilée y mettent fin prématurément. Le test échoue également (sortie `1`) lorsque moins de deux déclenchements ont synchronisé toutes les caméras sur une même horloge (`… the clocks cannot be compared`), ou lorsque les horloges ne concordent jamais : les horloges PTP présentent un écart supérieur à 1 ms (plus tout décalage au niveau de la couche série) par rapport au maître (`… PTP is not locked`), ou lorsque l’écart entre les horloges des périphériques varie de plus de cette valeur d’un déclenchement à l’autre. Il démarre le grand maître PTP s’il n’est pas en cours d’exécution, active le PTP de chaque caméra (comme le fait `array-connect`, quelle que soit la configuration du réseau), et avant le premier déclencheur, attend que l’horloge PTP de chaque caméra se verrouille sur le grand maître et se stabilise : `PtpStatus` : « Slave » sur un grand maître, avec un écart inférieur à 100 µs par rapport à celui-ci et stable pendant 3 s, et chaque caméra verrouillée de cette manière, sans interruption, pendant 120 s — soit environ 2 minutes à compter de la mise sous. Une horloge qui vient de se synchroniser présente un écart pouvant atteindre environ 200 µs pendant environ 2 minutes après avoir affiché « Slave », et la valeur `PtpOffsetFromMaster` de la caméra elle-même ne le reflète pas ; au bout de 120 s, les caméras se situent à moins de 10 µs d’une horloge stabilisée. Une caméra est toujours verrouillée depuis un test précédent qui avait laissé le PTP activé, tel que ce backend l’a détecté, compte à partir de ce verrouillage ; ainsi, un test effectué juste après un autre n’attend pas à nouveau ; une caméra que le test met sous tension, ou une caméra détectée avec le PTP activé mais dont ce backend n’a pas vu le verrouillage (allumée ailleurs, ou avant un redémarrage du backend), attend ses 120 s, et dans un ensemble comprenant les deux caméras qui viennent de se verrouiller, celles-ci attendent toujours leur tour. Une caméra qui quitte le mode Esclave, dépasse les 100 µs ou suit un autre grand maître pendant 3 lectures consécutives (1,5 s) au cours de l’attente recommence son décompte ; une ou deux de ces lectures sont enregistrées (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) et le décompte se poursuit ; un identifiant de grand maître illisible est considéré comme inconnu, et non comme un changement. Le fait que le décompte de toutes les caméras redémarre simultanément (en moins d’une seconde) est dû au décalage de l’horloge du « grandmaster », et non à celui des caméras : le journal du backend indique `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` le répertorie dans `grandmaster_steps`, et un délai d’attente qui expire alors commence son erreur par `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. La limite n’est pas prolongée pour cela. `--ptp-settle SECONDS` définit le temps de stabilisation pour l’exécution (0-600 ; `0` pour aucune attente de stabilisation). L&#x27;attente est limitée au temps de stabilisation plus 90 s (210 s par défaut, au moins 90 s), et la commande attend le backend pendant 240 s plus le temps de stabilisation (360 s par défaut). Elle affiche `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, et le journal du backend contient une ligne toutes les 5 s indiquant l&#x27;état et le décalage de chaque caméra, depuis combien de temps elle est verrouillée et le temps restant : `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Chaque caméra s’allume avec le PTP désactivé ; à partir de là, les horloges se synchronisent en environ 10 s. Ensuite, chaque caméra est réinitialisée dans l’état où le test l’a trouvée — son PTP, l’exposition automatique et le gain que le test maintient désactivés pour son exécution, et le reste (sauf si `--keep-settings`) — ainsi, une caméra dont le PTP est resté activé (`lattice configure -s <serial> --set PtpEnable=True`, ou un cycle avec `--keep-settings`) reste synchronisée, et une fois que ce backend a constaté qu’elle est synchronisée pendant le temps de stabilisation, un test ultérieur n’attend pas qu’elle le soit. Si les horloges ne se stabilisent pas pendant l’attente, le test échoue avant le déclenchement (`PTP: NOT locked …`, sortie `1`) et indique la marche à suivre : une caméra qui ne s’est jamais verrouillée (`PtpStatus Listening`) signifie que le grand maître ne parvient pas à la joindre (`time-sync status`) ; celle qui n’a cessé de se déconnecter est identifiée par la durée pendant laquelle elle est restée verrouillée (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`) ; si les horloges sont toujours en cours de convergence : vérifiez `time-sync status`, et pour qu’elles soient verrouillées avant le test, activez le PTP et laissez-le activé (`lattice configure -s <serial> --set PtpEnable=True` pour chaque caméra, ou lancez le test avec `--keep-settings` ; le test rétablit les paramètres qu’il a détectés, donc il reste activé), puis lancez-le. |
| `lattice multi-detect [--line LINE] [--json]` | Détection des caméras sur la ligne de synchronisation M8 et d’un maître par défaut : la caméra RGB (modèle se terminant par `-FRGB`) si elle est câblée, sinon la première caméra câblée par numéro de série. Les câbles M8 se rejoignent en un seul point de jonction, de sorte que **n’importe quelle caméra câblée peut être désignée comme maître** — la mention imprimée « `MASTER` » est une valeur par défaut, et non une réalité de câblage. Chaque caméra pilote la ligne à tour de rôle et la présence d’une impulsion est vérifiée sur les autres ; une caméra déjà détectée dans le groupe câblé n’est pas sondée à nouveau, ainsi, un réseau entièrement câblé nécessite un seul cycle. En l’absence de `--line`, il interroge `Line2`, puis `Line3`. Il affiche `MASTER: SN:…`, un `SLAVE: SN:…` par caméra câblée supplémentaire et `(probed on <line>)` ; `--json` ajoute le résultat complet (`master`, `slaves`, `no_cable` — caméras non atteintes par l’impulsion — `line`, et par caméra `results`). Une fois l&#x27;opération terminée, chaque caméra retrouve ses paramètres de déclenchement et de ligne de synchronisation d&#x27;origine (les mêmes nœuds que ci-dessus). |

#### Alignement

| Sous-commande | Objectif |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — ajout des boutons de détection/correspondance `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, paramètres RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinaison multi-images `[--averaging mean\|median\|inlier_weighted]`, contraintes géométriques `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restriction spatiale `[--roi X0,Y0,X1,Y1] [--mask PATH]` etremplacements par esclave `[--per-cam-override SN:KEY=VALUE]` (répétable) | Calculer le profil d’alignement à partir de caméras en temps réel. Valeurs par défaut : `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (dans le dossier à partir duquel vous l&#x27;exécutez), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` possède ses propres paramètres par défaut ; voir [son tableau](#project-align-calibrate-options).) `--prefilter` utilise par défaut `gradient` (carte des contours ; correspond à l’aligneur GUI/tableau — les contours sont conservés d’une bande spectrale à l’autre). `--matcher flann` donne de bons résultats au-delà d’environ 5 000 caractéristiques ; `--averaging median` est robuste face à une capture défectueuse, `inlier_weighted` pondère en fonction du nombre de correspondances ; `--lock-scale` projette vers la rotation la plus proche (sans échelle), `--lock-axis` remet à zéro une composante de translation ; `--mask` s&#x27;applique à toutes les caméras (utiliser `--per-cam-override` pour les paramètres propres à chaque caméra, par exemple `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` refuse d&#x27;enregistrer un calibrage dont la valeur RMS de reprojection dépasse le seuil. `--method manual` récupère ses points depuis `--points PATH` : un objet JSON `{"<serial>": [[x, y], ...], ...}` fournissant les coordonnées en pixels des mêmes caractéristiques pour chaque caméra, y compris celles de la caméra de référence (au moins 2 par caméra) ; chacun des `--frames` est calculé à partir de ces points. Lorsqu’un tableau connecté contient les caméras, l’étalonnage utilise les images en direct de ce tableau, et le profil devient également l’alignement du tableau pour ses captures. Sinon, il ouvre un [tableau temporaire](#multi-camera-transient-sessions) : une caméra qui ne fournit aucune image dans l’un des `--frames` échoue à l’étalonnage (sortie `1`, en lui attribuant un nom), et aucun profil n’est enregistré. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Capture une image multibande alignée. `--bit-depth` utilise par défaut l’alignement de la caméra ; `--no-crop` conserve l’image complète (en la complétant par du noir) ; `--interpolation` (par défaut `linear`) et `--border-mode`/`--border-value` (par défaut `constant`/0) contrôlent la déformation CPU — le chemin GPU est bilinéaire dans tous les cas. Ouvre un [tableau transitoire](#multi-camera-transient-sessions) : une caméra qui ne fournit aucune image échoue la capture (sortie `1`, en la nommant), et rien n’est écrit. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Flux d’images multibandes alignées (mêmes paramètres de warp que `align-apply`), `--fps` par défaut `3`. Ouvre un [tableau transitoire](#multi-camera-transient-sessions) : un groupe d’images pour lequel une caméra n’a rien fourni n’est pas enregistré, et trois d’affilée mettent fin au flux. Si un groupe a été ignoré, l’exécution se termine en affichant `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` et `Error: …`, puis se termine par `1`. Un flux qui se termine par une autre erreur affiche également `Error: …` et se termine par `1`. |
| `lattice align-info --profile PATH [--json]` | Afficher les détails du profil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Modifier l&#x27;ordre des couches. |

#### Index / Mathématiques de la végétation

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` associe les symboles d&#x27;un préréglage aux bandes** (NDVI : `red`, `nir`), un `--channel SYMBOL=BAND` pour chacun.

- **`--live` : uniquement pour un symbole auquel la pile ne peut pas répondre.** Comme pour `--input`, un symbole d’un préréglage que vous omettez prend la bande de la pile (les bandes des caméras activées du profil) dont le nom commence par celui-ci. Cette vérification est effectuée avant l’ouverture de toute caméra : un symbole auquel aucune bande, ou plusieurs bandes, ne répond est alors refusé ; en les nommant, on quitte `1`. Un `--formula` personnalisé nécessite un `--channel` pour chaque symbole.
- **Les produits `--live` indiquent à qui ils appartiennent.** Les `index.tif` et `index_LUT.png` contiennent les informations relatives à la caméra de la pile (marque, modèle, numéro de série), son heure de capture (exposition de la caméra de référence, UTC) et son objectif, et tous deux désignent l&#x27;index comme leur bande unique (`XMP-Camera:BandName` `NDVI` ; le fichier PNG couleur désigne l&#x27;index, et non ses canaux de couleur), tout comme le fait un produit `--input`. Un index issu des bandes de plusieurs caméras désigne chacune d’entre elles, et son `Model` est `LATTICE` — voir [Fichiers créés à partir de plusieurs caméras](#files-made-from-several-cameras).
- **`--input` : uniquement pour un symbole auquel le fichier ne peut pas répondre.** Un symbole que vous omettez correspond à la bande du fichier dont le nom commence par ce symbole : `red` prend `Red_660` ou `Red_660_RGN`, `nir` prend `NIR_850`. Les noms proviennent du fichier lui-même : la liste des bandes propre à la pile ou, sur une acquisition de radiance / réflectance Chloros ou un produit `process`, ses balises `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Ainsi, `lattice index --input <reflectance capture> --preset NDVI` n’a pas besoin de `--channel`.
- **Deux bandes correspondant à un même symbole ne sont pas distinguées** (un empilement à 9 caméras comporte deux bandes NIR) : nommez celle-ci. Un symbole auquel rien ne correspond est rejeté avec `channel_map missing entries`, puis `1`.**Les noms de bandes dans une pile** indiquent le filtre de la caméra : les bandes d’une caméra M3C sont `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` sur un RGN ; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` sur un OCN ; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` sur un RGB ; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` sur un NGB — et la bande unique de la caméra M3Mbande unique de la caméra M3M est `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` répertorie les bandes d’un profil.

Jeu complet d’indicateurs : `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (répétables), `--capture-level raw|debayered|radiance|reflectance|unknown` (remplace le niveau de capture enregistré dans le fichier TIFF source ; par défaut : lu dans les métadonnées du fichier TIFF, en recourant par défaut à la valeur `Chloros:ProcessingLevel` d’un fichier Chloros ; ainsi, une capture de réflectance indique `reflectance`), `--output PATH` / `-o` (hors ligne : un chemin d’accès ; avec une extension d’image, les fichiers sont nommés à partir de leur racine — `-o ndvi_FRGN.png` écrit `ndvi_FRGN.tif` et `ndvi_FRGN_LUT.png` — et en l’absence d’extension, il s’agit d’un préfixe, `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png` ; par défaut à côté de l’entrée, comme `<input name>_<preset>` ; avec `--live` et sans `-o`, un dossier `output/` sous le dossier à partir duquel vous exécutez la commande), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Avec `--live`, les curseurs d’alignement s’appliquent également : `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (cible d’exposition automatique des caméras, voir les règles relatives aux transitoires), `--serials SN1,SN2,…`. `--live` ouvre un [tableau de transitoires](#multi-camera-transient-sessions) : une caméra qui ne fournit aucune image fait échouer l&#x27;exécution (sortie `1`, en la nommant), et aucune pile ni aucun index n’est écrit. Elle ouvre `--serials` (la première déclenche le déclencheur), sinon les caméras du profil dont l’entrée `layer_order` est activée, en commençant par sa référence — la référence même si ses bandes sont désactivées, puisque les autres sont alignées sur celle-ci et déclenche le déclencheur. Une caméra dont les bandes sont désactivées n’est jamais calibrée ; ainsi, une caméra RGB désactivée dans le profil ne fait plus échouer un indice de réflectance (`no-absolute-radiance-scale`). Un profil qui ne nomme aucune caméra ouvre toutes les caméras du réseau, comme auparavant.

Chaque formulaire s’exécute sur le backend ; celui-ci doit donc être en cours d’exécution : lancez-le depuis l’interface graphique d’Chloros, ou à l’aide de la commande affichée par l’CLI lorsqu’elle ne parvient pas à y accéder. Hors ligne, le backend lit `--input` et écrit lui-même le fichier TIFF en virgule flottante et le fichier PNG colorisé, de sorte qu’un `--input` ou `-o` est résolu par rapport au dossier à partir duquel vous exécutez la commande (voir [Conventions](#conventions)). Les deux produits hors ligne contiennent les informations d’enregistrement de l’appareil photo de l’entrée, tout comme le fait une exportation d’index `process` : marque / modèle / numéro de série, GPS, balises de l’objectif, heure de capture (celle de l’entrée elle-même ; lorsque ses données EXIF n’en indiquent aucune, l’instant d’exposition qu’elle a enregistré ou l’horodatage de son nom de fichier, en UTC) et l’identifiant d’étalonnage MAPIR — jamais l’étalonnage raw-DN, qui ne s’applique pas à un index — ainsi que `Chloros:ProcessingLevel=index` et l’expression de l’index ; le TIFF à virgule flottante désigne une bande, le préréglage. `--list-presets` et `--list-gradients` affichent les tables de préréglages et de dégradés du backend. Une formule rejetée par le backend, ou une bande `--channel` que le TIFF ne possède pas, quitte `1` en indiquant la raison fournie par le backend.

> **Les symboles `--channel` sont sensibles à la casse.** Le côté « symbole » doit correspondre exactement aux noms de canaux du préréglage (les préréglages utilisent des minuscules, par exemple NDVI = `red`,`nir` — vérifiez `--list-presets`), et la partie « bande » doit correspondre à un nom de bande dans la pile alignée (ou être un index de bande à partir de 0 en mode hors ligne). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` fonctionne ; `--channel RED=660` échoue avec une erreur `channel_map missing entries` .

#### `lattice index` — Masques de pile multibande

À l’étape `--capture-level radiance` ou `reflectance`, le démixage 3×3 de chaque caméra peut limiter une bande à 0 (voir [Masque de limitation de décomposition](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Une pile multibande contient les bandes de plusieurs caméras, et chaque caméra limite sa propre bande ; ainsi, `--live --save-multiband` écrit **un masque par caméra source** à côté de la pile :

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Chaque masque est un `uint8` monobande sur la grille de pixels de la **pile** (après le même alignement, la même déformation et le même recadrage), avec les bits propres à cette caméra : bit 0 = la bande de son canal R, bit 1 = G, bit 2 = B. Une bande de la pile issue du canal *c* d’une caméra est limitée là où le bit *c* de cette caméra (R = 0, G = 1, B = 2) est activé.
- La provenance intégrée de chaque bande indique le nom du masque de sa caméra (`unmix_clamp_mask`, par rapport au dossier de la pile, ou `none`) et le bit à lire (`unmix_clamp_mask_bit`).
- Une caméra dont le « unmix » n’a rien écrêté ne génère aucun fichier ; une caméra mono ou RGB, qui n’effectue pas de « unmix », n’en génère pas non plus. L’interface CLI affiche chaque masque sous la pile (`clamp mask SN…: …`) ; la réponse les répertorie sous la forme `clamp_masks`.
- Le fichier d&#x27;index TIFF/PNG de la même exécution est calculé à partir de la même pile, sur la même grille ; les masques le décrivent donc également. `--capture-level raw` n’effectue pas de décomposition et n’écrit aucun masque.
- La pile et l’index se trouvent directement dans `--output` ; les masques sont donc placés dans le dossier `clamp_masks/`, qui appartient à `--output` — jamais au-dessus du dossier que vous avez nommé. Ce dossier contient différents types de fichiers (pile, index, rendu colorisé) : sélectionnez-en les fichiers, ne transmettez pas le dossier entier au logiciel de photogrammétrie.

#### Connexions persistantes (Smart-Prep, flux équivalent à l’interface graphique)

Ces commandes maintiennent les caméras ouvertes dans le pool du backend entre les invocations de l’CLI.

| Sous-commande | Objectif |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Ajoute une caméra au pool (caméra unique, pas de tableau). Les [options communes](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` et `--packet-size` (`standard` = 1 500, `jumbo` = 9 000, ou en octets ; sondes `auto`) sont appliquées à la connexion (un commutateur `--exposure` ou `--gain` désactive le mode automatique). Elle revient dès que le flux de la caméra a démarré (en attendant environ 15 s pour cela, puis 10 s pour son premier tampon complet) et affiche `stream: streaming`, ou `stream: no_frames -- connected, no frames yet: …` lorsque le flux n’a rien transmis — pour une caméra utilisant des paquets jumbo, avec la recommandation de se reconnecter à l’aide de `--packet-size standard`. Pendant qu’une caméra se connecte (une première connexion télécharge son étalonnage, environ 70 s), une autre commande nécessitant une caméra attend celle-ci pendant 5 s au maximum, puis échoue avec un code HTTP503 `CAMERA_POOL_BUSY` indiquant la caméra en cours de connexion ; relancez-la dans un instant. `lattice status` et la liste des caméras n’attendent jamais : une caméra en cours de connexion s’affiche comme étant en cours de connexion, avec sa phase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Libération. La caméra est rétablie telle qu’elle a été détectée lors de la connexion (câblage du déclencheur, PTP, paramètres) ; `--keep-settings` enregistre ses paramètres dans sa propre mémoire à la place, `--clear-settings` réinitialise cette mémoire aux paramètres d’usine (voir [Mémoire de la caméra](#camera-memory----keep-settings----clear-settings)). Une caméra faisant partie d’un ensemble libère l’ensemble complet. |
| `lattice cam-list` | Liste des caméras du pool. |
| **`lattice array-connect`**|**Connecter un réseau synchronisé persistant (point d’entrée recommandé).** Exécute l’intégralité du flux de préparation intelligente via l’interface graphique. Environ une minute pour 9 caméras ; juste après la mise sous tension des caméras, cela peut en prendre plusieurs. Pendant l’attente, il affiche chaque étape atteinte par le backend, sous la forme `    <s> s: <step>` (par exemple `    41 s: open SN:214000535 (4 of 9)`), et le journal du backend comporte une ligne de chronométrage par étape (`[array-connect <id>] <step>: <s> s …`) ainsi qu’une dernière ligne indiquant les étapes les plus lentes. Il attend jusqu’à 600 s. Si la connexion n’est pas établie dans ce délai, le backend l’annule : chaque caméra qu’il a ouverte est remise dans l’état où elle a été trouvée, puis fermée ; la commande affiche `Error:` avec l’étape atteinte, puis se termine par `1`, et aucun réseau n’est connecté (code HTTP504 `CLIENT_DEADLINE_PASSED`). La combinaison Ctrl+C annule la connexion de la même manière (`Interrupted: …`) ; si le réseau était déjà connecté, la ligne l’indique et mentionne le code `array-disconnect` qui le déconnecte. Une commande interrompue pendant qu’elle attend (`kill -9`) laisse la connexion en attente d’annulation lorsque le backend n’a pas reçu de réponse de sa part depuis 30 s. Un tableau qui est actif mais qui ne dispose pas encore de groupe de trames synchronisées est tout de même signalé comme connecté, avec son flux identifié par `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Libération d’un tableau. Chaque caméra est laissée en fonctionnement libre (`TriggerMode` `Off`), avec tous les autres paramètres tels qu’ils étaient lors de l’établissement de la connexion — ses réglages de déclenchement et de ligne de synchronisation (le`TriggerSource` revient à son état initial, sa ligne de synchronisation redevient une entrée), son PTP et les réglages de la caméra modifiés par la session — ainsi, une commande mono-caméra sur un membre s’applique immédiatement. `--keep-settings` enregistre les réglages de chaque camératels qu’ils ont été laissés par la session dans la mémoire propre à la caméra ; `--clear-settings` réinitialise cette mémoire aux paramètres d’usine. Voir [Mémoire de la caméra](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Liste des matrices connectées. |
| `lattice array-status [--array-id ID]` | FPS en direct, PTP, dernière erreur. `ptp_enabled` n’est égal à `True` que lorsque tous les membres indiquent que PTP est activé ; `ptp_off:` désigne un membre qui ne le fait pas. La ligne `state:` lit `restarting (applying settings)` lors d’un redémarrage des paramètres, `recovering (…)` lorsqu’un membre ayant subi une coupure d’alimentation est réactivé (voir la note ci-dessous), `stalled (…)` lorsque la boucle de capture s&#x27;exécute mais ne publie aucun groupe de trames, et `stopped (…)` une fois que la boucle de capture a abandonné (reconnecter la matrice). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Une capture synchronisée à partir de la matrice en direct — Unique / Continue / Par intervalle / Rapide. **Par défaut : `all`**(un fichier par type d’exportation applicable par caméra),**aligné** comme l’est la capture de l’application (`--no-aligned` pour les fichiers natifs). Les caméras ignorées (par ex. les RGBs exclues de la radiance/réflectance) sont signalées avec `Skipped: SN:<serial> (<reason>)` ; la lecture DAQ de la capture utilisée est enregistrée en parallèle, une capture en format brut uniquement est incluse, et le tout est signalé avec `DAQ: <path>` ([Le DAQ qui lit une capture enregistre](#le-daq-qui-lit-une-capture-enregistre)). Voir [Modes de capture, enregistreurs et retraitement hors ligne](#modes-de-capture-enregistreurs--retraitement-hors-ligne). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Enregistrer la vue combinée en temps réel sous forme de vidéo/GIF (qualité de surveillance ; sans fenêtre d&#x27;interface graphique ouverte, le backend affiche l&#x27;NDVI du réseau, ou `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Rafale Bayer brute à haute fréquence d’images (qualité analyse ; retraitement hors ligne). À l’instar de `array-record` et `array-capture --continuous`, elle effectue un check-in auprès du backend à chaque interrogation : elle est interrompue sans être arrêtée (`kill -9`), le backend l&#x27;arrête 30 s après la dernière vérification (`stop_reason` `client_gone` ; le manifeste de la rafale est finalisé et la construction de son index combiné est lancée, comme lors d’un arrêt) et le réseau reste connecté. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Retraitement d’une rafale brute enregistrée en vidéo(s) calibrée(s). |

> **Une caméra qui subit une coupure d’alimentation ou une perte de liaison se reconnecte d’elle-même.** Lorsqu’un membre cesse de répondre (un cycle d’alimentation du commutateur redémarre toutes les caméras qu’il alimente), le réseau l’attend, le reconnecte dès qu’il répond à nouveau, et lui restitue les paramètres définis par `array-connect` : PTP, ligne de déclenchement et de synchronisation, cadence réseau, format de pixel et taille d’image, ainsi que l’exposition automatique. Elle reprend alors la diffusion sans nouveau `array-connect`. Pendant ce temps, `array-status` affiche `state: recovering (...)` et `last_error` indique quelle caméra est en attente ; ensuite, il affiche `re-opened:`. Une caméra qui a redémarré revient avec le PTP désactivé ; ainsi, jusqu’à ce qu’elle soit rouverte, `ptp_enabled` correspond à `False` et `ptp_off:` lui attribue un nom.

> **Une caméra retenue n’est jamais ouverte une deuxième fois.** Une deuxième ouverture arrête le flux de la caméra retenue, réécrit ses paramètres et peut provoquer un plantage du backend. Ainsi, sur une caméra détenue par le pool (l’interface graphique ou `cam-connect`), un élément de tableau, ou une caméra détenue par une tâche d’arrière-plan (un `lattice continuous` en cours d’exécution, `multi-stream` ou `align-stream`, qu’il s’agisse de la sienne ou de celle d’un autre client), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, untentative unique `white-balance` et `presets --apply` renvoient un refus avec un code HTTP 409 `CAMERA_HELD`, en indiquant le titulaire et la commande de libération (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, ou pour une tâche, son identifiant et son initiateur), puis se termine `1`. Il en va de même pour une caméra qu’une autre commande utilise actuellement (une commande `multi-test` ou `multi-capture` ouvrant ses caméras, une autre commande ponctuelle) ; pour chaque commande mentionnée ici, le message d’erreur indique d’attendre que cette commande se termine. `lattice power`, sans `--sleep`, parcourt le descripteur propre au détenteur lorsqu’il en existe un (le pool, un tableau, la caméra ouverte d’un travail), et est refusé si une caméra est déjà utilisée par une autre commande. Chaque réponse `CAMERA_HELD` contient le code à la fois dans `code` et `error_code`. Pour les commandes qui ouvrent toutes les caméras, voir [Multi-caméras (sessions transitoires)](#multi-camera-transient-sessions).

##### Options `array-connect`

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Détection automatique de toutes les caméras LATTICE (nécessite ≥2) | La première caméra en série est le MAÎTRE. Si ce paramètre est omis, la détection se limite aux modèles LATTICE (M3C / M3M) et les connecte tous. |
| `--line {Line0,Line2,Line3}` | `Line2` | Ligne de synchronisation GPIO. |
| `--target-fps F` | auto | Fréquence de déclenchement du maître. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Remplacer le sélecteur de niveau. |
| `--wire-ceiling-mbps MB_PER_S` | détecté automatiquement | **Bande passante physique soutenue de l’hôte, en Mo/s — valeur à laquelle dépend l’allocation de l’ensemble de la matrice.** Réduisez-le lorsque le réseau signale des trames GVSP corrompues : la valeur par défaut est dérivée du débit de liaison annoncé par la carte réseau, qui surestime les adaptateurs USB, les voies PCIe étroites et les structures partagées très sollicitées. Cette valeur est conservée dans le bloc de capture du réseau du projet ; ainsi, une réouverture ou une reconnexion via la CLI ou le SDK la rétablit. Voir [État de santé du réseau de cartes](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | aucun (capteur complet) | Regroupement matériel (toujours « Average »). |
| `--width N` / `--height N` / `--pixel-format FMT` | capteur complet ; format par caméra | Une taille d’image ou un format de pixels explicite pour chaque caméra, au lieu du capteur complet et du choix du format par caméra (Mono12 / BayerRG12). |
| `--no-recommend` | désactivé | Ignorer l’étape d’analyse du réseau (chaque caméra conserve alors sa taille actuelle au lieu d’être réglée sur le capteur complet). |
| `--no-ptp` | désactivé | Désactiver le PTP (les horodatages entre caméras ne sont alors **pas** comparables). |
| `--packed {auto,on,off}` | `auto` | Diffuser des trames 12 bits compressées et les décompresser sur cet ordinateur — plus de trames par seconde à la résolution maximale du capteur, données identiques. `auto` ne compresse que lorsque cela est utile et que cet ordinateur en a les moyens ; `off` ne compresse jamais. Voir [Transport 12 bits compressé](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Taille de paquet GVSP pour chaque caméra du réseau. `auto` permet à chaque caméra d’explorer le réseau ; `jumbo` = 9 000, `standard` = 1 500, ou une taille en octets le bloque. Avant la version 1.3.0, `array-connect` acceptait cet indicateur mais l&#x27;ignorait. |

> **Capteur complet par défaut.** En l&#x27;absence de `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect`, chaque caméra est réglée sur le capteur complet — sans
> binning, ni recadrage de la zone d’intérêt (ROI) — et laisse la fréquence d’images s’adapter à ce que cet ordinateur et
> le réseau peuvent fournir, plutôt que de réduire la taille des images pour maintenir la fréquence. La
> ligne d’analyse du réseau indique la fréquence prévue, et la ligne suivante répertorie
> les options plus rapides (binning ou zone d’intérêt plus petite) avec leurs fréquences prévues ; passez
> `--binning` ou `--width`/`--height` pour en sélectionner une. La commande « Connect » n’effectue jamais de regroupement de pixels ni de recadrage
> de sa propre initiative.

##### Transport 12 bits compressé (`--packed`)

Lorsque le capteur fonctionne à pleine capacité, la fréquence d’images d’un réseau est déterminée par la vitesse à laquelle chaque caméra peut envoyer
ses images. Les formats 12 bits (`BayerRG12`, `Mono12`) sont généralement transmis sous forme de mots de 16 bits ;
le transport **compressé** envoie les mêmes 12 bits en 1,5 octet par pixel et
les décompresse sur cet ordinateur, ce qui réduit de 25 % le temps d’envoi de chaque image —
soit environ **33 % d’images par seconde en plus à pleine capacité du capteur** (mesure : 10,7 → 14,0 images/seconde
sur un réseau de 9 caméras). **Les données enregistrées sont identiques** : mêmes valeurs, mêmes
fichiers 16 bits, mêmes métadonnées.

- `auto` (par défaut) compresse uniquement lorsque cela est utile **et** que cet ordinateur dispose de
  suffisamment de ressources pour la décompression. À la connexion, le backend mesure sa propre vitesse de décompression, le nombre de cœurs,
  la charge CPU actuelle et la bande passante réseau disponible ; ainsi, un ordinateur de bureau disposant de cœurs inutilisés compresse
  à pleine capacité du capteur, tandis qu’un ordinateur occupé ou à faible puissance diffuse les données sans compression. Les petites zones d’intérêt (ROI) et les images regroupées (binned frames) ne gagnent
  peu.
- `off` ne compresse jamais — utilisez-le pour désactiver la compression lors d’un diagnostic, ou sur un
  ordinateur qui doit consacrer toute sa puissance CPU à d’autres tâches.
- `on` compresse toujours les formats 12 bits.

La sortie « connect » affiche la décision et sa justification (« compressed 12-bit: on (auto :
projection 11,0 -&gt; 14,4 images par seconde, …)`); `/api/camera/array/status` signale
`packed_transport` et `packed_decision`. Pour définir une valeur par défaut à l’échelle de la machine
au lieu de passer l’indicateur à chaque fois, définissez `CHLOROS_ARRAY_PACKED12=off` (ou
`on`) dans l’environnement du backend — un `--packed on|off` explicite prévaut toutefois.
Les formats 8 bits et 16 bits ne sont jamais compressés.

### Un seul dossier de projet, tous modes confondus

Un dossier de projet contient les paramètres de chaque périphérique : `cameras.json` — ceux de chaque caméra (`cameras[].settings`, pour une seule caméra et pour chaque élément du réseau) ainsi que les paramètres communs à l’ensemble du réseau (`arrays[].capture` : format, ROI, binning, fréquence de déclenchement, budget de câblage) — et `sensors.json`, ceux de chaque capteur de lumière (temps d’intégration, moyenne par image, limite). L’application, l’CLI, le SDK et le serveur MCP lisent et écrivent tous les mêmes fichiers, de la même manière :

- **Un périphérique connecté dans le cadre d’un projet y est enregistré et y consigne chaque modification.** « Dans le cadre d’un projet » signifie qu’un projet est ouvert en arrière-plan (celui de l’application), ou — depuis le SDK — via un fichier `ChlorosProject`, qui désigne son propre dossier. Une caméra, un réseau de capteurs ou un capteur de lumière que le projet ne contient pas encore est ajouté à ses fichiers lorsqu’il se connecte (`lattice cam-connect`, `array-connect`, `daq pool-connect`, l’application, le SDK, `chloros-mcp`), avec ses paramètres. Toute modification apportée à n&#x27;importe quel mode (`lattice exposure`, `gain`, `configure --set`, `resolution`, leset de la fréquence de déclenchement, la limite d’un capteur de lumière, les commandes de l’application) est enregistrée dans les fichiers de ce projet quelques instants plus tard. Une exposition, un gain ou une cible AE définis sur une caméra sont enregistrés en tant que « maintenus » ; ainsi, la réouverture de la caméra les conserve.
- **L&#x27;ouverture d&#x27;un projet configure ses périphériques en fonction de celui-ci**, quel que soit le mode : une caméra, un réseau ou un capteur de lumière connecté sans paramètres propres reprend les paramètres enregistrés du projet (`lattice cam-connect` / `array-connect` avec un projet ouvert, `chloros-cli project connect`, `open_project().connect_all()`, réouverture de l’application). Un réseau connecté sous un nouvel identifiant met à jour le réseau que le projet a enregistré avec les mêmes caméras ; un réseau comportant d’autres caméras est ajouté à côté.
- **L’enregistrement effectué par l’application elle-même ne remplace jamais les paramètres en temps réel d’un périphérique** par une ancienne copie de ceux-ci, et conserve les modifications par élément enregistrées par un appelant CLI ou SDK.
- **Le format de pixels que vous définissez est celui que le projet conserve.** Un format défini dans les paramètres du tableau, via `--pixel-format` ou `connect_array(pixel_format=)`, y compris le format 8 bits, est enregistré et réutilisé à l’identique. Si aucun format n’est défini, chaque caméra se connecte avec son format par défaut de 12 bits (`BayerRG12`, `Mono12` sur un capteur monochrome). Le nom d’un format correspond à sa profondeur de bits ; la famille est celle du capteur, de sorte qu’une caméra couleur n’est jamais configurée en format mono ni une caméra mono en format Bayer, et la ligne `[PixelFormat <serial>] ... written` l’indique lorsqu’un format a été mappé. Les exportations d’une capture 8-bit sont au format 8 bits, y compris la réflectance : elles sont redimensionnées et marquées de son code `PixelScale`.
- Un périphérique connecté sans être associé à un projet n’enregistre rien.
- **Un enregistrement suit le projet créé après lui ; une session ouverte le laisse avec son projet.**Lorsqu’un projet est créé — un SDK `create_project()`, le `chloros-mcp` de `create_project`, ou le**Nouveau projet**de l’application — ou dupliqué (l’option**Dupliquer**de l’application, qui ouvre la copie) alors qu’un enregistrement est encore en cours d’écriture dans le projet qui était ouvert auparavant, l’enregistrement se poursuit et ce qu’il écrit se retrouve dans le nouveau projet, au même emplacement à l’intérieur de celui-ci (ci-dessous). L&#x27;ouverture d&#x27;un projet existant — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, via la fonction**Ouvrir un projet** de l’application — laisse l’enregistrement en cours dans le projet dans lequel il est enregistré : il s’achève à cet endroit, et ses fichiers sont ajoutés à la liste des fichiers de ce projet (l’application arrête tout enregistrement avant d’ouvrir un projet).
  - `array-record` et `array-burst` : la vidéo, ou le dossier « burst » contenant toutes les images, y est déplacé dans son intégralité lorsque l’enregistrement s’arrête (c’est `--duration`, Ctrl+C). Le fichier `Burst dir` généré est le nouveau, et les fichiers `--build` et `array-build-video` s’appuient sur celui-ci. Si le disque refuse le déplacement, l’enregistrement reste à l’emplacement où il a été écrit ; lejournal de fin indique pourquoi, tout comme la réponse HTTP de l’arrêt (`relocation_error`). Les images brutes d’une rafale arrêtée et le fichier `.daq` sont ajoutés à la liste des fichiers du projet dans lequel elle aboutit ; une vidéo (un `array-record`, vidéo générée à partir d’une rafale) n’est pas ajoutée à la liste des fichiers d’un projet.
  - `array-capture --continuous` : toutes les captures effectuées après le changement sont enregistrées dans le nouveau projet ; celles enregistrées auparavant restent à leur emplacement d’origine.
  - `daq pool-record` et `project sensor log` (ainsi que l&#x27;enregistrement par le capteur de lumière intégré à l&#x27;application) : le fichier `.daq` est déplacé vers le dossier du nouveau projetdossier `light_sensor` de ce projet lorsque l’enregistrement s’arrête, et s’ajoute à la liste des fichiers de ce projet.

  Un enregistrement dans un dossier situé en dehors du projet ouvert (un `-o` situé ailleurs) reste à sa place, tandis que les fichiers `lattice continuous` et `multi-stream` conservent le dossier qui leur a été attribué.
- **Dans l’application, les captures effectuées après la création d’un nouveau projet y sont intégrées.** La création ou la duplication d’un projet maintient toutes les sessions de capture et tous les enregistrements en cours, et chaque capture effectuée par la suite (Capturer tout, Démarrer la capture) est enregistrée dans le nouveau projet et s’ajoute à sa liste de fichiers. L’ouverture d’un projet existant dans l’application les arrête tous au préalable.
- **Un projet n’est pas déplacé pendant qu’il enregistre.**Le déplacement du projet ouvert vers un autre répertoire de travail (via**Paramètres → Répertoire de travail** de l’application ; aucune commande CLI ne permet de déplacer un projet, contrairement à un client HTTP effectuant une requête vers `/api/move-project`) est refusé avec le code HTTP 409 `PROJECT_RECORDING` tant qu’un enregistrement de matrice ou une rafale, une capture continue de matrice, une tâche `lattice continuous` / `multi-stream` ou un enregistrement par capteur de lumière y écrit : `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` L’application refuse de la même manière, avant de demander un dossier, alors que ses propres fonctions « Capture All », « Start Capturing » ou un enregistrement par capteur de lumière sont en cours d’exécution.

Ceci est distinct de l’état dans lequel les caméras sont laissées lors de la déconnexion (voir [Mémoire de la caméra](#camera-memory----keep-settings----clear-settings)) : un projet configure lui-même ses caméras à son ouverture.

### Mémoire de la caméra — `--keep-settings` / `--clear-settings`

Lorsqu’un ensemble est connecté, ses caméras reprennent les paramètres de l’ensemble, que vous pouvez ensuite modifier. Lorsque Chloros libère les caméras — `array-disconnect`, fin d’une commande transitoire (`multi-capture` sans tableau, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), l&#x27;arrêt de `lattice continuous`, la fin d&#x27;un `lattice capture` d&#x27;une caméra qu&#x27;il ne gère pas — il remet chacun d&#x27;entre eux dans l&#x27;état où il se trouvait avant la connexion : son câblage de déclenchement et de ligne de synchronisation, ainsi que les paramètres PTP et de la caméra (la liste complète se trouve sous [Multi-caméras (sessions transitoires)](#multi-camera-transient-sessions)).

Pour utiliser les caméras en dehors d’un projet, deux indicateurs sur n’importe laquelle de ces commandes écrivent dans la **mémoire propre** de la caméra — celle avec laquelle elle démarre, quel que soit le logiciel :

- **`--keep-settings`** enregistre dans la mémoire de chaque caméra les paramètres tels qu’ils y figuraient à la fin de la session (UserSet2, qui constitue le jeu de paramètres de démarrage). Les caméras continuent de fonctionner en mode libre (`TriggerMode` `Off`), le câblage de la ligne de synchronisation restant tel quel. Une caméra sans UserSet2 enregistre dans UserSet1 ; cet ordinateur s’en souvient, de sorte que le système d’auto-réparation du réseau d’Chloros — ce qui réinitialise un UserSet1 s&#x27;étant écarté de la référence de Chloros — ne le modifie pas.
- **`--clear-settings`** réinitialise la mémoire de chaque caméra aux paramètres d&#x27;usine : la référence de Chloros, celle que la fonction d&#x27;auto-réparation du réseau intègre — les paramètres d’usine par défaut avec `TriggerMode`, `Off`, `TriggerSource` et `Line2`, exposition automatique, la balance des blancs calibrée d’une caméra RGB conservée — dans UserSet1, qui constitue le jeu de paramètres de démarrage.
- Les deux ne peuvent pas être combinés. Chaque caméra dispose d’une ligne : `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` ou `SN:…: camera memory reset to the factory settings.` ; la réponse `camera_memory` est identique pour chaque caméra.

L’application ne propose pas ces indicateurs.

**Chaque sortie rétablit les caméras dans leur état initial.**Le même rétablissement « tel qu’il a été trouvé » s’effectue sur chaque chemin de sortie, et pas seulement pour les commandes ci-dessus : `cam-disconnect` (et la caméra**X**de l’application) ; le**X** de l’application sur un tableau ou sur la ligne de n’importe quel élément du tableau (un tableau ne peut pas s’exécuter sans élément, donc le tableau entier est libéré) ; la fermeture de l’application (elle accorde au backend jusqu’à 20 s pour remettre en place chaque caméra, tableau et le processus `lattice continuous` en cours d’exécution avant de l’arrêter ; une commande encore en cours d’exécution à ce moment-là — une commande transitoire ouvrant ses caméras ou attendant leurs horloges PTP, une commande ponctuelle `lattice capture`, `power`, `white-balance` ou `presets --apply` — s&#x27;arrête à son prochain point de sécurité, remet d&#x27;abord ses caméras en place, pendant 8 s maximum, et renvoie une réponse HTTP 503 `SHUTTING_DOWN` ; une connexion de type « array » toujours en cours — `lattice array-connect`, la connexion de l’application — s’arrête avant sa prochaine caméra et referme les caméras qu’elle a ouvertes, pendant 10 s maximum, et renvoie une réponse HTTP 503 `SHUTTING_DOWN` ; et une requête arrivant pendant l’arrêt n’ouvre rien : HTTP 503 `SHUTTING_DOWN`) ; et une commande CLI qui est interrompue ou dont le terminal se ferme (SIGTERM / SIGHUP arrêtent sa tâche — et remet ses caméras en place — avant de quitter `143` / `129`, comme Ctrl+C). Un processus `lattice continuous`, `multi-stream` ou `align-stream` tué net (SIGKILL), qui ne peut rien arrêter, voit sa tâche interrompue et ses caméras remises à zéro par le backend 30 s après son dernier enregistrement. Un backend qui plante ou qui est forcé de s’arrêter ne peut rien réinitialiser ; un redémarrage ramène chaque caméra à sa mémoire, et la prochaine connexion à Chloros reconfigure une caméra qu’elle trouve activée.**Un projet configure lui-même ses caméras.** Lorsqu’un réseau se connecte alors qu’un projet est ouvert, chaque membre récupère les paramètres que le projet a enregistrés pour lui — une exposition, un gain ou une cible AE qui lui est attribuée (considérée comme la vôtre, de sorte que l’exposition automatique ne la modifie pas) et son mode de balance des blancs (correction de Chloros ; la balance des blancs propre à la caméra reste désactivée) — quel que soit le contenu de la mémoire de la caméra.

### Smart-AE / Smart-Capture

Les réseaux LATTICE exécutent l’exposition automatique (AE) en continu en arrière-plan dès qu’ils sont connectés, mais une scène nouvellement cadrée met un instant à converger. `array-capture --smart` est la **solution pratique** : il attend que l’exposition automatique se stabilise sur toutes les caméras du réseau, puis déclenche la capture. Utilisez-le lorsque vous changez de scène en cours de session.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

La politique de stabilisation est prudente par défaut : délai d’expiration de 5 s, fenêtre de stabilité de 1,5 s, tolérance de variation d’exposition de ±5 %. Réglez ces paramètres via le SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) si vous avez besoin d’un comportement différent de celui de l’automatisation.

#### Fonctionnement de l’exposition automatique

- **L’exposition d’abord, le gain ensuite.** Lorsque le gain est en mode Auto, il reste à 0 dB jusqu’à ce que l’exposition ait atteint la valeur maximale autorisée.
- **Une exposition manuelle laisse le gain en mode « Auto »** jusqu’à ce que vous modifiiez le gain ou que vous le régliez en mode manuel. De retour en mode « Auto », la limite du gain est la même qu’auparavant.
- **Les petites zones surexposées n’assombrissent pas la scène.** Une réflexion spéculaire représentant jusqu’à 1 % du cadre est ignorée, et sur les panneaux d’une cible d’étalonnage connue, jusqu’à 0,5 % d’un panneau.
- **Le plafond d’exposition du DAQ ne s’applique que lorsqu’une cible d’étalonnage est visible.** Il empêche un panneau blanc d’être écrêté ; lorsqu’aucun panneau n’est visible, la caméra s&#x27;expose en fonction de la scène.
- **La lumière constante reste stable, et les variations de lumière se stabilisent en quelques secondes.** En cas de lumière constante, l&#x27;exposition et sa limite restent inchangées, et après un changement important de luminosité, l&#x27;image se stabilise en quelques secondes. Un écrêtage ne limite plus définitivement l&#x27;exposition, et un changement de zone d&#x27;intérêt (ROI) permet de retrouver l&#x27;exposition adéquate.
- **La cible de luminosité reste à portée.** Elle n’augmente pas lorsque la caméra est à sa limite d’exposition et que son gain ne peut plus augmenter ; ainsi, un retour de lumière ne provoque pas de surexposition de l’image.
- **Une caméra dont l’exposition automatique cesse de réagir est maintenue au niveau correspondant à la luminosité ambiante.** En plein soleil, une caméra pourrait passer à son temps d’exposition le plus court et y rester avec une image sombre, alors que tous ses paramètres affichent des valeurs normales. Chloros surveille l’image, maintient cette caméra au niveau qu’elle a conservé pour cette luminosité, et la relâche lorsque la luminosité change. Un capteur de lumière sombre ou couvert ne perturbe pas ce fonctionnement.

### Niveaux d’exportation des captures (valeur par défaut de `all`)

À partir de cette version, `lattice capture`, `lattice multi-capture` et `lattice array-capture` **sont par défaut réglés sur `--processing all`** — un fichier enregistré par type d’exportation qui s’applique à chaque caméra, ce qui correspond au comportement de l’option « Tout capturer » de l’interface graphique. Les niveaux sont les suivants :

| Niveau | Sortie | S’applique à |
| --- | --- | --- |
| `raw` | Bayer monocanal (caméras monochromes : la bande unique) provenant directement du capteur. | Toutes les caméras. |
| `debayered` | Couleur à 3 canaux, enregistrée dans l’ordre des canaux **RGB**(un fichier TIFF avec interprétation photométrique RGB ; les fichiers PNG/JPEG s’ouvrent également en RGB) : le décalage de noir (DSNU) du pack d’étalonnage est soustrait du fichier brut, puis le dématriçage sensible aux contours d’OpenCV— sans table de conversion linéaire (LUT), sans correction de champ plat ni démixage (caméras monochromes : 1 canal en niveaux de gris). Les images débayérisées en mémoire renvoyées par le SDK Python (`grab()`, `frame_stream()`) sont au format**BGR**, selon l’ordre d’OpenCV ; seul le fichier enregistré est au format RGB. | Toutes les caméras. |
| `radiance` | float32 W/m²/sr/nm via la chaîne radiométrique complète. | Multispectrales (M3C/M3M) uniquement — **ignoré pour les caméras à filtre RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatible Pix4D. | Multispectral uniquement, et **uniquement lorsqu’un DAQ est associé et que la caméra est calibrée** ; sinon ignoré. |
| `display` | Chaîne complète de prévisualisation via l’interface graphique (CCM + WB + gamma selon le profil de la caméra). Enregistrée dans un dossier `preview/`. `lattice capture` accepte également le nom `preview` pour celle-ci ; `array-capture` / `multi-capture` n&#x27;acceptent que `display`. | Toutes les caméras. |

Indiquez un seul niveau pour n&#x27;en enregistrer qu&#x27;un seul (`--processing debayered`). Lorsque vous demandez `all`, les niveaux qui ne s&#x27;appliquent pas à une came donnée sont ignorés (et signalés), sans générer d’erreur — une caméra non connectée ou non calibrée reçoit tout de même `raw` / `debayered` / `preview`.

#### Enregistrement de la mesure DAQ d’une capture

Chaque fois qu’un DAQ est associé à la caméra ou au réseau, ou qu’il est mis en pool, la mesure de rayonnement descendant utilisée pour une capture est enregistrée dans un fichier **`.daq`** situé dans le sous-dossier `daq/` du dossier des captures, avec son `.match.json` (écart entre la lecture et l’exposition) et son `.cameras.json` (à qui appartient la lecture), et est signalée sur une ligne `DAQ:`. Cela vaut pour tous les niveaux, y compris les captures brutes ou débayérisées, de sorte que toute capture peut être retraité ultérieurement en radiance, en réflectance ou en indice. Il s’agit du même fichier, avec le même nom et les mêmes fichiers sidecar, que la capture provienne de l’interface graphique, de la CLI ou du SDK. En l’absence totale de mesure DAQ, il n’y a pas de `.daq`, et un `multi-capture` sans matrice connectée (solution de secours transitoire) n’en enregistre aucune.

| Indicateur | Effet |
| --- | --- |
| `--no-daq` | Ne pas enregistrer la lecture sur une capture dont les niveaux ne l’utilisent pas (`raw`, `debayered`). Un niveau `radiance`, `reflectance` ou `display` l&#x27;écrit tout de même, car la réimportation a besoin de la lecture avec laquelle le produit a été fabriqué. |
| `--daq-tolerance-ms MS` | Écart maximal par rapport à l’exposition pour que la mesure soit encore considérée comme dans les tolérances (valeur par défaut : 500). Seul le verdict change (les `.match.json`, `in_tolerance` et `tolerance_ms`, ainsi que la réflectance `DlsInTolerance`), jamais la mesure utilisée. Voir [Une mesure DAQ éloignée de l’exposition](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | L&#x27;ancienne méthode pour demander le `.daq` lors d&#x27;une capture en format brut uniquement. C&#x27;est désormais le comportement par défaut, et les codes `lattice capture` et `array-capture` acceptent toujours cet indicateur (ce qui n’est pas le cas de `multi-capture`). `--no-daq` a la priorité sur celui-ci. |

Quel DAQ : celui lié à la caméra ou au réseau de capteurs (sélection du capteur de lumière dans l’interface graphique, liaison enregistrée dans un projet, ou `chloros-cli daq pool-bind`), ou bien le plus récent dont dispose le backend. Une rafale brute (`array-burst`) conserve son propre fichier `.daq` par lecture distincte dans le dossier « burst », comme auparavant.

**Les formats PNG et JPEG contiennent 8 bits par canal ; ils n’utilisent donc que les fichiers `display` / `preview` et les produits d’index.** Les fichiers `--format png` ou `jpg` dont le niveau dépasse leur capacité sont refusés avant toute capture ; le niveau est alors indiqué, ainsi que le format TIFF qui le prend en charge :

- `radiance` (float32) et `reflectance` (uint16, `32768` = 1,0) — dans tous les cas, `--processing all` (valeur par défaut) avec `--format png`/`jpg` est donc également refusé : utilisez `--format tiff` ou `--levels display`.
- `raw` et `debayered` provenant d’une caméra diffusant plus de 8 bits (les trames M3C / Mono12 M3M à 12 bits) — l’CLI ne peut pas détecter la profondeur de bits de la caméra, ce qui fait que le backend les refuse (HTTP 400). La trame uniquetrame unique `lattice capture --processing debayered` capture une trame de 8 bits pour les formats PNG/JPEG et n’est pas refusée, tout comme un `multi-capture` sans tableau persistant (débayérisé à 8 bits).

Chaque route de capture répond de la même manière (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`) : `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, ainsi que les routes HTTP qui se trouvent derrière elles.

### À quoi ressemble un dossier de captures

Chaque type d’exportation est placé dans son **propre sous-dossier**sous `-o` ; ainsi, une capture à plusieurs niveaux ne mélange jamais les types. Le schéma illustre la**capture par défaut `all` d’une caméra par `lattice capture`** — plusieurs niveaux par caméra, de sorte que chaque fichier porte un suffixe `_<level>` :

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Variations des noms de fichiers :

- **Champ « caméra » — numéro de série, puis filtre :** chaque fichier de capture porte le nom de la caméra qui l’a enregistré sous la forme `SN<serial>-<filter>`, où `<filter>` correspond à la partie « filtre » du modèle de la caméra : `SN261902142-F450` pour une `M3M-L87-F450`, `SN213800234-FRGN` pour une `M3C-L87-FRGN`, `SN214701292-FRGB` pour une caméra RGB. Le filtre est ajouté au numéro de série, comme dans `-`, et non par `_` ; ainsi, un nom comporte les mêmes champs séparés par « `_` » qu’auparavant (préfixe, date, heure, millisecondes, caméra, niveau). Une caméra dont le modèle ne peut pas être lu (DeviceUserID effacé) est nommée simplement `SN<serial>`. Les captures issues de versions antérieures ne comportent pas de filtre dans leur nom ; Chloros lit les deux.
- **Préfixe :** `lattice capture` nomme les fichiers `capture_…` ; `array-capture`, `multi-capture` avec un réseau connecté, et `project capture` / `burst` / `stream` d’un tableau les nomme `sync_<ts>_SN<serial>-<filter>…`.
- **Suffixe de niveau uniquement en cas de plusieurs niveaux.**Une caméra qui enregistre**plusieurs**niveaux dans une seule capture (`all` ou `--levels a,b`) ajoute `_<level>`. Une caméra qui enregistre**un** niveau conserve le nom simple, sans suffixe — `--processing reflectance` écrit `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — afin que les scripts écrits pour les captures à un seul niveau continuent de fonctionner. Le niveau est toujours indiqué par le dossier.
- **Les masques de limitation suivent le nom du produit :** `<product name>_clampmask.tif`. Une capture de réflectance à un seul niveau génère `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif` ; une capture `all` génère `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **L&#x27;extension** suit `--format` : la valeur par défaut `tiff` donne `.tiff`, `--format tif` donne `.tif` (les masques de limitation sont toujours `.tif`).
- **Une exception :** `lattice capture --processing debayered` (ou `--levels debayered`) prend un chemin d&#x27;accès rapide à une seuleet écrit `capture_<ts>_SN<serial>-<filter>.tiff` directement dans `-o` — sans sous-dossier.
- **Les flux et les rafales** contiennent le même champ : `lattice continuous` écrit `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` et une rafale brute `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` correspond à l’horodatage de la capture et `<serial>` au numéro de série de la caméra ; ainsi, un groupe synchronisé partage un
horodatage commun à toutes les caméras. **Notez cette seule asymétrie :** le niveau `display` est stocké dans un dossier
nommé `preview/`, tandis que les fichiers à plusieurs niveaux conservent `_display` dans le nom — le dossier et le suffixe diffèrent
pour ce niveau uniquement. Les niveaux inconnus sont placés par défaut dans un dossier portant leur propre nom, et si le sous-dossier
ne peut pas être créé, le fichier est écrit à la racine de sortie plutôt que d’être perdu.

**Retraitement d’un dossier de captures :**pointez `chloros-cli process` vers la**racine des captures**
(`output/`). `process` n’importe normalement que le dossier que vous nommez, mais lorsque ce dossier ne contient aucune
d’images et comporte des sous-dossiers, il explore automatiquement l’arborescence — ainsi, les sous-dossiers de niveau et le
sous-dossier `daq/` sont tous récupérés en une seule fois. Chaque niveau d’une capture est importé sous forme d’une seule image, les
autres niveaux étant disponibles en tant que modes, plutôt que sous forme d’une image par niveau.

Nommer directement un **sous-dossier de niveau** (par exemple `output/raw/`) fonctionne également. Cela laisse `daq/`
de côté ; vous devez donc copier ou indiquer la lecture DAQ à côté lorsque vousdérivez un produit radiométrique
à partir de `raw/` — sinon, la correspondance d’horodatage n’aura aucun élément de référence.

**Le traitement commence toujours à partir de `raw`.** Au sein de chaque capture, l’image brute constitue la source du pipeline ;
`debayered`, `radiance`, `reflectance` et `preview` apparaissent comme des modes d’affichage mais ne sont jamais réinjectés
dans le pipeline. Le retraitement d’un produit dérivé réappliquerait la vignette, le CCM et les
calculs de radiance déjà intégrés dans ses pixels ; c’est pourquoi Chloros préfère renoncer plutôt que de
procéder à un double traitement. Deux conséquences à connaître :

- Les rendus `index/` et `composite/` ne sont **jamais** traités. Ce sont des sorties, pas des captures —
  un rendu LUT NDVI ne comporte aucune interprétation significative de la radiance.
- Un dossier de captures exporté **sans** `raw` (par exemple `array-capture --processing reflectance`) ne constitue
  pas de source de pipeline valide. Ces captures s’importent et s’affichent normalement, mais `process` les ignore
  et l’indique :

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Si vous avez réellement besoin de faire passer un produit dérivé — une session de hub capturée avec
  `demosaic` activé, ou un dossier hérité — `--input-level {raw,debayered,processed}` force le point d’entrée
  et passe outre l’ignorance. Cet indicateur est une échappatoire délibérée ; `auto` (la valeur par défaut)
  ne traite jamais une capture dépourvue de données brutes.

**Balises d’étalonnage : sur `raw` uniquement.** Le fichier `raw` d’une capture contient le paquet d’étalonnage de la caméra : la charge utile `CalibrationJSON` et les balises compatibles MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), qui décrivent les valeurs brutes du capteur. Tous les autres fichiers de chaque niveau (`debayered`, `display`, `radiance`, `reflectance`, le rendu `index/`) ainsi que chaque trame `lattice continuous` / `multi-stream` porte l’identité de la caméra (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) ainsi que l’enregistrement propre à l’image, `XMP-mapir:FrameJSON` (exposition, gain, température, format des pixels, heure de capture, correspondance DAQ), car chaque produit `process` porte cette identité. `process` trouve l&#x27;étalonnage d&#x27;un tel fichier grâce à son numéro de série et `CalBundleSha256` dans le cache d&#x27;étalonnage d&#x27;Chloros, en téléchargeant le pack une seule fois si cet ordinateur ne le possède pas.

#### Fichiers créés à partir de plusieurs caméras

Un fichier créé à partir des bandes de plusieurs caméras — un empilement multibande (`align-apply`, `align-stream`, `index --live --save-multiband`), un index en temps réel issu de plusieurs caméras (`index.tif`, `index_LUT.png`), une image `composite/` issue d’une capture en matrice — mentionne toutes les caméras qui y ont contribué :

- **Listes par caméra, classées par bande** (la composition : couche d’arrière-plan, puis de premier plan ; une couche d’index, les caméras dont proviennent ses bandes) : `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (le pack d’étalonnage par lequel sont passées les bandes de chaque caméra, sinon — une pile `align-apply` / `align-stream`, dont aucune bande n’a été traitée — le pack de la caméra que Chloros a mis en cache, celui dont le nom figure dans ses fichiers de capture), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — des listes parallèles, dont l’élément *n* de chacune décrit la même caméra ; `-` lorsque la caméra n’a aucune valeur (pas de pack d’étalonnage, objectif inconnu). Chaque appareil photo correspond à **un** élément : lorsque son pack comporte plusieurs valeurs pour un champ — les certificats de lampe halogène d’un pack —, celles-ci sont reliées par `;` (`F-1479-1;F-1480-1;F-1481-1`), jamais par une virgule, de sorte qu’une liste affiche un élément par appareil photo, quelle que soit la manière dont elle est imprimée.
- **EXIF `SerialNumber`** contient les mêmes numéros de série, séparés par des virgules (`214000535,214000551`).
- Il ne comporte **aucune** identité propre à une caméra (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …) : ces numéros identifient un appareil photo et son ensemble, et `process` permettrait de déterminer l’étalonnage du fichier à partir de ceux-ci. Un index calculé à partir des bandes de plusieurs appareils photo (`index.tif`, `index_LUT.png`) comporte des données EXIF `Model` `LATTICE` (le rig), et non le modèle d’une seule caméra `LATT-…`.

Un fichier contenant les bandes d’**une seule** caméra — un fichier TIFF par bande (`--per-band`), une image `align-apply --per-camera`, une pile ou un index en temps réel d’une seule caméra — porte les balises d’identité propres à cette caméra, comme tout autre fichier de capture. La pile de `align-apply`, ses fichiers par bande et par caméra, ainsi que chaque pile de `align-stream` et ses fichiers par bande, comportent l’heure de capture de leur groupe (l’exposition de la caméra de référence, en UTC), et une image `--per-camera` d’une caméra mono correspond à un seul plan, au format `--bit-depth`, tout comme la pile.**Orientation de la caméra (inversion / rotation).** Le réglage d’orientation d’une caméra (les options « Mirror H » /
Mirror V / rotation) de l’interface graphique détermine ce qui est enregistré lors d’une capture :

- **`raw` et `debayered` sont enregistrés exactement tels que le capteur les a lus**, quel que soit le réglage. Une
  mosaïque de Bayer pivotée ou inversée présenterait un motif de filtres de couleur différent, et les cartes d’étalonnage
  (décalage sombre, champ plat) se rapportent à la grille propre au capteur.
- **`preview`/`display`, `radiance`, `reflectance` et le rendu `index/` sont orientés** — pivotés
  une seule fois, après tous les calculs d’étalonnage, de sorte qu’ils s’ouvrent dans le sens dans lequel l’appareil photo a été cadré. Leurs
  fichiers `clamp_masks/…_clampmask.tif` se trouvent sur la même grille.
- **Chaque fichier de capture enregistre le réglage** au format XMP : `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, dans le sens des aiguilles d’une montre, après
  les rotations) et `ChlorosOrientationApplied` — `false` sur les données brutes et débayérisées (pixels natifs),
  `true` sur les produits orientés. `ChlorosProcessing` répertorie les opérations appliquées aux
  pixels de ce fichier (`none` lorsqu’il est natif). Cela inclut chaque image transmise par `lattice continuous` (les
  paramètres propres à la caméra, non appliqués) et chaque image de `lattice multi-stream` (l’identité : ses
  images constitutives sont natives, comme celles d’une capture en tableau). Les piles `lattice align-stream` / `align-apply`
  enregistrent les paramètres de la caméra maître, non appliqués : leurs bandes sont déformées pour s’adapter à l’image de la caméra maître
  mais ne sont pas pivotées.
- **`process` applique l’orientation enregistrée** lorsqu’il traite une image brute : décalage sombre, champ plat
  , dématriçage et démixage dans les coordonnées du capteur d’abord, puis la déformation d’alignement du réseau, et enfin l’
  orientation en dernier — de sorte que les produits d’aperçu, de radiance, de réflectance et d’indice correspondent exactement à
  ceux de la capture en direct, et enregistrent `ChlorosOrientationApplied=true`. Les produits de secours non calibrés
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) sont orientés de la même manière, après
  le champ plat. Le produit débayérisé conserve son orientation d’origine.
- **La visionneuse d’images d’Chloros affiche un fichier brut ou débayérisé pivoté selon son enregistrement** (affichage uniquement —
  le fichier n’est pas modifié), afin qu’il corresponde à ses produits ; la vignette, la vue en taille réelle, l’
  affichage du curseur et le bac à sable Index/LUT utilisent tous la même rotation. Un fichier sans enregistrement, ainsi qu’
  un produit (déjà orienté), s’affichent tels qu’ils ont été enregistrés. Une exportation vers le bac à sable enregistre l’orientation que ses
  pixels portent (`ChlorosOrientationApplied=true` ; l’identité d’une couche sans enregistrement).
- **Les captures sans enregistrement sont traitées en format natif** (comportement inchangé). Avant la version 1.3.0, une
  capture à caméra unique avec un réglage de retournement ou de rotation faisait elle-même pivoter ses fichiers bruts et débayérisés ;
  Chloros, ne pouvant pas distinguer un tel fichier brut déjà- à partir d’un fichier natif et ne fait aucune supposition ; il
  le traite donc tel qu’il a été enregistré. Les cartes d’étalonnage se trouvent sur la grille propre au capteur ; pour une telle
  capture, utilisez-en une réalisée avec la version 1.3.0 lorsque son étalonnage est important.
- **Par défaut, les tableaux ne font pivoter aucun fichier composant** (le warp d’alignement et le fichier XMP `Alignment*` se trouvent dans
  les cadres propres aux caméras, non pivotés) et enregistrent l’identité. `array-capture --orient` fait pivoter les
  produits de chaque image, après la transformation d’alignement : selon l’orientation du **maître** au moment où ils ont été
  transformés dans son repère (afin que les images restent co-enregistrées), ou bien selon celle de leur propre caméra. Chaque
  fichier enregistre l’orientation du repère dans lequel se trouvent ses propres pixels — un fichier brut, celui de sa propre caméra —
  et un fichier brut contenant la transformation d’alignement enregistre également `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation` : celles du fichier maître, pour un produit aligné
  dans le cadre de la caméra principale. `process` fait pivoter chaque produit selon celui qui correspond à ses pixels :
  celui aligné lorsqu’il déforme le fichier brut (`--array-alignment`, la valeur par défaut — les
  ), celui propre à la caméra lorsqu’il ne le fait pas (`--no-array-alignment` — ceux de la
  capture en direct non alignée). La même option existe pour `array-burst`, `project capture` / `burst` /
  `stream`, les recettes et l’SDK.

### Captures ignorées dans les réseaux de filtres mixtes

Lorsque vous mélangez des caméras RGB et multispectrales dans un même réseau, `array-capture --processing radiance` (ou `reflectance`) enregistre les images multispectrales et **ignore** les caméras RGB — la radiance par pixel Bayer n’a pas de sens pour un capteur à large bande. L&#x27;CLI affiche explicitement chaque fichier enregistré (avec son niveau d&#x27;exportation), chaque `.daq` écrit et chaque omission, de sorte que le nombre de fichiers n&#x27;est pas surprenant :

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Les jetons indiquant la raison de l’ignorance suivent le format `<level>-not-applicable-to-rgb-cam`. Une caméra dont le pack d’étalonnage n’est pas chargé ignore les fichiers `radiance` / `reflectance` avec `<level>-needs-calibration (connect the camera so its cal pack loads)`, et en l’absence de lecture de réflectance DAQ utilisable, les sauts se produisent avec `reflectance-skipped-no-dls`. La réflectance peut également être ignorée avec `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, et avec `dls-uncalibrated-band-<nm>` lorsque la bande se situe en grande partie en dehors de la plage radiométriquement étalonnée du capteur de lumière du DAQ (~374–974 nm) — parmi les références disponibles à la vente, seule la F988, dont le parcours pris en charge correspond au flux de travail avec panneau de réflectance.

Utilisez `--processing debayered` (ou `display`) pour inclure toutes les caméras quel que soit le type de filtre, ou la valeur par défaut `all` pour obtenir en une seule fois tous les niveaux applicables par caméra.

### Une lecture DAQ éloignée de l’exposition

Chaque valeur `.daq` enregistrée lors d’une capture correspond à la lecture DAQ associée à cette exposition, et la valeur `.match.json` qui l’accompagne indique l’écart entre les deux (`gap_ms`) et si cet écart se situe dans la tolérance (`in_tolerance`, 500 ms sauf si la capture a défini `--daq-tolerance-ms`). Une valeur hors de cette plage est tout de même utilisée, de sorte qu’aucune image n’est perdue, et elle est identifiée : le fichier XMP du fichier de réflectance contient les données `DlsInTolerance=False` et `DlsMatchGapS`. `lattice capture`, `multi-capture` et `array-capture` l’affichent également sous le `.daq` concerné, avec l’erreur de flux du DAQlorsqu’on explique :

**Comment une lecture est associée à une exposition.** Une capture attend la première lecture du DAQ après l’exposition, afin de pouvoir fusionner les lectures situées de part et d’autre de celle-ci (`method` `interpolated`). Un DAQ-E envoie une lecture toutes les 0,4 s à la lumière du jour et toutes les 1 à 2 s à l’intérieur ; l’attente ajoute donc environ une fois et demie ces intervalles à la commande, et non à l’exposition. On n’attend pas un DAQ qui a cessé d’envoyer des données. Une lecture-E correspond à la moyenne de la luminosité sur l’intervalle écoulé depuis sa lecture précédente ; elle est donc placée au milieu de cet intervalle : `gap_ms` correspond à la distance mesurée entre l’exposition et ce point médian, tandis que `bracket_stamps_ns` indique les horodatages propres aux lectures. `array-capture --continuous` attend sans ralentir la capture : ses fichiers bruts sont écrits immédiatement et leurs `.daq` une fois la lecture reçue. Avec `--interval`, chaque passage attend comme s’il s’agissait d’une seule capture.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Lorsqu’aucun fichier de réflectance n’a été écrit à partir de cette lecture, la première ligne se termine plutôt par `saved anyway; its .match.json says in_tolerance: false`.

La capture se termine tout de même par `0` : tous les fichiers ont été écrits et indiquent leur contenu. Un DAQ dont le flux est bloqué continue de fournir sa dernière lecture ; vérifiez-la donc avec `chloros-cli daq pool-list` avant de vous fier à la réflectance de cette capture. Le backend reconnecte un DAQ-E en pool qui a cessé de fournir des données (voir [Un DAQ-E qui cesse de fournir des données](#a-daq-e-that-stops-delivering)). Une lecture hors tolérance n’affiche que sa ligne `DAQ:`, sauf s’il s’agit d’une lecture conservée (voir ci-dessous).

**Une lecture conservée après la déconnexion de son DAQ.** Lorsqu’aucun DAQ ne peut fournir de lecture pour une exposition (le DAQ lié n’est pas disponible, ou aucun DAQ du pool n’en possède, par exemple après `daq pool-disconnect`), la capture réutilise la dernière mesure dont disposait Chloros, et sa ligne `.match.json` décrit cette mesure par rapport à cette exposition : `method`, `retained`, `gap_ms` de cette exposition aux échantillons à partir desquels la mesure a été effectuée, `in_tolerance` correspondant à cet écart, et `retained_from_epoch_ns`, l’exposition à laquelle la mesure a d’abord été associée. Une capture effectuée longtemps après la déconnexion du DAQ affiche donc les lignes ci-dessus, et en dessous une ligne supplémentaire, qui s’affiche même lorsque la mesure se situe encore dans les limites de tolérance :

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Sa valeur `.daq` indique toujours le DAQ qui a effectué la lecture, ainsi que l’étalonnage et le capuchon utilisés lors de cette lecture ; ainsi, une réimportation permet de savoir quel DAQ, quel étalonnage et quel capuchon l’ont produite.

### Fichiers dont l’écriture échoue

Une caméra dont le fichier ne peut pas être enregistré — disque plein, erreur de droits d’accès — n’empêche pas l’enregistrement des fichiers des autres caméras. Chaque commande de capture (`lattice capture`, `multi-capture`, `array-capture` ; `project capture`, `burst`, `stream` ; ainsi que les commandes `capture`, `burst` et `stream` d’une recette) affiche chaque fichier qui n’a pas pu être enregistré sur stderr, puis **quitte `1`** même si le reste a été enregistré :

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Tous les fichiers associés à une capture sont pris en compte, y compris le seulimage débayérisée `lattice capture --levels debayered`. Une lecture `.daq` dont le fichier sidecar de provenance `.match.json` n’a pas pu être écrit est répertoriée sous le nom `[match]` (le fichier `.daq` lui-même a été enregistré).

Lorsque le disque est plein, une capture répétée (`array-capture --continuous` / `--interval`, `project burst` / `stream`, une recette `burst` / `stream`) **s&#x27;arrête à ce passage** au lieu de ne rien écrire sur le même disque pour le reste de l&#x27;exécution. Toute autre défaillance est signalée à chaque passage où elle se produit, et l&#x27;exécution se poursuit jusqu&#x27;à la fin, puis se termine par `1`. `array-capture --continuous` (la capture continue du backend) ne répertorie aucun fichier, y compris ceux qui ont échoué : il les comptabilise dans ses lignes de progression et de résumé (`N failed`, `N file(s) failed to write.`) et quitte `1` — voir [Mode continu](#continuous-mode---continuous).

Les enregistreurs — `array-burst` et `array-record` — signalent leurs propres échecs. `array-burst` s’arrête de lui-même lorsque le disque est plein, ou juste avant qu’il ne le soit. Les deux continuent à fonctionner malgré toute autre écriture ayant échoué. Lorsque l’un d’eux signale une erreur, celle-ci s’affiche après les compteurs, et la commande **quitte `1`** :

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` à la place de `Disk full:` signifie que la rafale s’est arrêtée avant d’avoir rempli le disque. Les trames enregistrées avant l’arrêt sont conservées. `array-burst --build` ignore la compilation lorsque l’espace est épuisé, car les vidéos seraient enregistrées sur le même disque. Libérez d’abord de l’espace, puis exécutez `array-build-video --burst-dir …`. Après toute autre erreur, la compilation se poursuit.

Un `array-record` qui n’a écrit aucune image constitue également un échec d’enregistrement (`Recorder error: no-frames: the combined view published no frame to record …`, quitter `1`) — voir [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Modes de capture, enregistreurs et retraitement hors ligne

Tous ces éléments fonctionnent sur un **tableau persistant** (exécutez d’abord `array-connect`). Ils reflètent le panneau de capture de l’interface graphique.

### Modes `array-capture`

`array-capture` est une commande unique proposant quatre modes d’obturation ainsi qu’un ensemble d’options d’exportation :

| Mode | Indicateur | Comportement |
| --- | --- | --- |
| **Unique** *(par défaut)* | (aucun) | Un groupe de captures synchronisées, puis sortie. |
| **Continu** | `--continuous` | Captures successives jusqu’à ce que `Ctrl+C`, `--count N` capture, `--duration S`, ou jusqu’à ce que le disque soit plein. Exécute la capture continue propre au backend (le mode « Continu » de l’interface graphique) — voir [Mode continu](#continuous-mode---continuous) ci-dessous. |
| **Intervalle** | `--interval S` | Un passage toutes les `S` secondes (mesurées à partir du début de chaque passage), mêmes limites. |
| **Le plus rapide** | `--fastest` | Brute+ la lecture DAQ attribuée + le composite d’indice combiné ; ignore les calculs de radiance/réflectance/affichage pour que l’image s’affiche rapidement. Implique `--processing raw --force-daq`. Retraiter ultérieurement les données enregistrées `.daq` pour obtenir des produits calibrés. |

Options d’exportation (à combiner avec n’importe quel mode ; toutes partagent le point de terminaison GUI/SDK) :

| Indicateur | Effet |
| --- | --- |
| `--array-id ID` | Le tableau à capturer (à omettre lorsqu’un seul est connecté). |
| `-o, --output DIR` | Dossier de sortie (par défaut `output`, dans le dossier à partir duquel vous exécutez la commande). |
| `--format {tiff,tif,png,jpg,jpeg}` | Format de fichier (par défaut : `tiff`). Les formats PNG/JPEG n&#x27;acceptent que `display` et le rendu d&#x27;index ; voir [Niveaux d&#x27;exportation de capture](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Niveau d&#x27;exportation unique (`raw`, `debayered`, `radiance`, `reflectance`, `display`), ou `all` (par défaut). |
| `--levels L1,L2,…` | Sous-ensemble explicite de types d&#x27;exportation (par ex. `raw,radiance,reflectance`) ; **remplace `--processing`**. |
| `--aligned` / `--no-aligned` | **Par défaut : aligné, comme la capture du même tableau par l’application Chloros**(son paramètre de capture**Aligné** par tableau, activé sauf si vous le désactivez). L’exportation non brute de chaque membre (`debayered`, `display`, `radiance`, `reflectance`, le rendu d’index) est transposée dans le cadre de la caméra principale et recadrée à la zone de chevauchement commune des membres ; ainsi, les fichiers sont alignés au pixel près et présentent la taille, le recadrage et les balises `Alignment*` identiques à ceux de l’application — plus petits que le capteur (ou de la zone d’intérêt) en raison des décalages des membres ; dans le cadre d’une zone d’intérêt, la transformation est recalibrée sur la fenêtre. `XMP-mapir:AlignmentJSON` indique `"applied": true` et fournit `crop_roi`. Le fichier brut reste non déformé à sa taille native mais contient la transformation dans ses métadonnées (`"applied": false`). `--no-aligned` enregistre chaque fichier à sa taille native, sans déformation. `--aligned` correspond à la valeur par défaut. Que la déformation soit appliquée ou non, chaque fichier membre enregistre la transformation `Alignment*` de sa caméra dès que le tableau dispose d’un profil d’alignement — à l’exception d’une caméra que le profil ne parvient pas à aligner (omise ou forcée à l’identité en raison d’un échec de résolution), et de la caméra de référence lorsqu’aucune autre caméra n’est alignée : celles-ci ne comportent aucune balise `Alignment*`. En l’absence de profil sur le réseau, la capture est enregistrée non alignée, et l’CLI affiche la raison sur une ligne `Alignment:`, comme elle le fait pour tous les autres cas où la déformation n’a pas pu être appliquée (éléments à des résolutions différentes, matrice non synchronisée par câble, image sur laquelle le profil ne peut pas être mappé). L’alignement automatique de la matrice crée le profil une fois que l’exposition est stabilisée ; `align-calibrate` en crée un à la demande. |
| `--orient` | Convertir les **produits**de chaque élément — `display`, `radiance`, `reflectance`, le rendu `index/` et le `composite/` — en fonction de l’orientation de la caméra, après la déformation d’alignement : un produit déformé dans le cadre du maître (par défaut) selon l’orientation du**orientation du master**, afin que les éléments restent co-enregistrés ; un fichier non aligné (`--no-aligned`, ou une capture à laquelle la transformation n’a pas pu être appliquée) selon celle de sa propre caméra.**Par défaut : désactivé — chaque fichier élément reste natif**, comme c’est toujours le cas pour les captures de tableaux. `raw` et `debayered` restent au format natif dans les deux cas. Chaque fichier enregistre l’orientation de ses produits (voir [Orientation de la caméra](#à-quoi-ressemble-un-dossier-de-captures)); le fichier XMP `Alignment*` conserve la transformation du cadre natif, appliquée avant l’orientation, de sorte que `process` (déformation, puis orientation) reproduit le produit en temps réel. |
| `--index` / `--no-index` | Enregistrer / ignorer la superposition de l’indice de végétation par caméra lorsqu’elle est configurée. Par défaut : l’afficher. |
| `--no-daq` | Ignorer le fichier sidecar `.daq` lorsqu’aucun niveau sélectionné n’utilise le DAQ (capture en format brut uniquement). Par défaut, la lecture est enregistrée, ce qui permet de retraiter les images en réflectance/indice hors ligne. `--force-daq` est toujours accepté (valeur par défaut). |
| `--daq-tolerance-ms MS` | Tolérance de correspondance DAQ pour ces captures (par défaut 500 ms). |
| `--smart` | Attendre que l’AE se stabilise sur toutes les caméras avant le déclenchement (voir [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compression des pixels au format TIFF. `none` (par défaut depuis la version 1.3.0 ; la version 1.2.0 écrivait `deflate`) = non compressé — les pixels du capteur sont enregistrés exactement tels qu’ils sont lus, environ 6,3 Mo par image en pleine résolution, et le débit soutenu le plus rapide ; `deflate` = zlib L1 sans perte + prédicteur horizontal, environ 4,1 Mo par image, plus lent. Les deux formats sont sans perte et s’affichent de manière identique à l’importation. |

> **TIFF à écriture unique + le modèle à débit soutenu.**Chaque fichier est écrit**une seule fois**: les pixels, le paquet XMP, l’IFD0 (Marque/Modèle/Numéro de série de l’appareil photo) et les balises EXIF `ExposureTime` / `SerialNumber` en un seul passage, sans réécriture ultérieure. `--continuous` capture des groupes synchronisés consécutifs — chaque capture prend le groupe suivant le précédent, y compris celui publié alors que la capture était encore en cours d’enregistrement — et fonctionne donc au débit du tableau. DEFLATE bloque le GIL de Python, de sorte que les écritures compressées ne sont**pas**parallélisées entre les threads d’écriture propres à chaque caméra ; une capture soutenue en pleine résolution à la fréquence du capteur nécessite des écritures non compressées (par défaut)**et** d’un disque de classe NVMe (~600 Mo/s en débit soutenu pour un réseau de 9 caméras à environ 10 images par seconde). Le même paramètre est accessible sous le nom `compression` sur `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Mode continu (`--continuous`)

`array-capture --continuous` (sans `--interval` ni `--smart`) confie l&#x27;exécution à la **capture en continu propre au backend** — la session dans laquelle s&#x27;exécute le mode Continu de l&#x27;interface graphique. Le backend enregistre lui-même chaque groupe synchronisé, avec deux captures en cours et aucune requête de la part de l’CLI entre les groupes ; il parvient ainsi à suivre le rythme d’un réseau rapide où une requête par capture prend du retard (mesuré à 73–77 captures/s sur un réseau de 9 caméras avec une zone d’intérêt de 512 pixels). Chaque option d’exportation ci-dessus s’applique à chaque capture.

- **Limites :** `--count N` s’arrête après N captures (une capture = un groupe synchronisé : les fichiers de chaque membre); `--duration S` après S secondes. En l’absence de ces deux paramètres, il s’exécute jusqu’à `Ctrl+C`, jusqu’à ce que le disque soit plein, qu’une erreur survienne ou que le réseau de caméras soit interrompu. `Ctrl+C` arrête la session du backend, puis affiche le résumé.
- **Progression** toutes les 2 s, avec `, N failed` et `, N groups skipped` ajoutés lorsqu’ils sont non nuls (un groupe ignoré est un groupe publié par la matrice pour lequel aucune capture n’a été effectuée) :

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Pas de liste par fichier.** L&#x27;exécution se termine par une ligne de résumé — captures, taux, fichiers et dossier de sortie — suivie de `N file(s) failed to write.` / `N group(s) skipped.` lorsque la valeur est différente de zéro :

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **S&#x27;arrête** lorsque le disque est plein avec `Continuous capture stopped: the drive is full.`, et en cas d&#x27;erreur ou de suppression de la matrice avec `Continuous capture stopped: <reason>.` (les deux sur stderr).
- **Code de sortie :** `0` lorsque les captures ont été enregistrées et que tous les fichiers ont été écrits, y compris `Ctrl+C` ; `1` si l&#x27;écriture d&#x27;un fichier a échoué, en cas de disque plein, d’erreur, de suppression du tableau ou d’absence totale de capture.
- **Microsoft Defender :** lorsque Defender utilise un cœur de processeur ou plus pour analyser les fichiers écrits par la capture, l’exécution affiche une fois, sur stderr : `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.`. Consultez les [Paramètres système](#windows-exe).
- Une seule capture continue par matrice à la fois ; une deuxième est refusée avec le code `a continuous capture is already running for this array`. Via HTTP, la session est `POST /api/camera/array/continuous/start` (le corps `/api/camera/array/capture` plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` et `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, avec ou sans `--continuous`) et**`--continuous --smart`** conservent la boucle par requête : une requête de capture par passage — à chaque moment programmé pour « Interval », une fois que l’AE s’est stabilisé pour `--smart` — les fichiers de chaque passage étant répertoriés comme en mode « Single », une ligne `Done — N capture pass(es).` à la fin, et `--count N` comptant les passages.

### `array-record` — vidéo/GIF à index combiné (qualité surveillance)

Enregistre la **vue en direct de l’index combiné** du tableau dans un fichier `.avi` (et éventuellement un fichier `.gif`). Lorsque la vue combinée de l’interface graphique est ouverte, elle enregistre cette vue telle qu’elle s’affiche. Si aucune vue n’est ouverte, le backend génère lui-même la vue : l’indice que vous transmettez avec `--expr`, ou par défaut le NDVI du tableau, une fois celui-ci aligné. Le NDVI par défaut utilise les bandes rouge et NIR les plus proches de 660 nm et 850 nm, comme le fait la vue en direct. Une vue de l’interface graphique ouverte pendant l’enregistrement est enregistrée tant qu’elle reste ouverte. Il interroge la progression toutes les 2 s et s’arrête sur `--duration`, `Ctrl+C`, ou lorsque l’enregistreur se termine de lui-même.

Un enregistrement qui n’écrit aucune image échoue : l’enregistreur signale `no-frames: …` (le capteur est peut-être encore en cours d’alignement ou de stabilisation de son exposition) et la commande **se termine sur `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--array-id ID` | matrice uniquement | Matrice cible (à omettre si une seule est connectée). |
| `-o, --output DIR` | `output` | Répertoire de sortie. Un chemin relatif se trouve dans le dossier à partir duquel exécutez la commande. |
| `--fps F` | `10` | Fréquence d&#x27;images d&#x27;enregistrement. |
| `--duration S` | jusqu’à Ctrl+C | Arrêt automatique après `S` secondes. |
| `--gif` | désactivé | Enregistrer également un GIF animé. |
| `--gif-only` | désactivé | Créer uniquement un GIF (pas de `.avi`). |
| `--expr EXPR` | l&#x27;NDVI du tableau | Expression d&#x27;index à enregistrer lorsqu&#x27;aucune vue GUI n&#x27;est ouverte, dans les noms de bandes du tableau (par ex. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'` ; voir [noms de bandes](#index--vegetation-maths)). Un nom de couleur court (`NIR`, `Red`) ne fonctionne que si une bande du tableau le possède. Une vue de l’interface graphique ouverte est enregistrée telle qu’elle s’affiche. |

### `array-burst` — rafale Bayer brute à haute fréquence d’images (qualité analyse)

Lit directement le tampon du groupe synchronisé de la boucle de capture — **aucune chaîne d’étalonnage, aucun exiftool, aucun affichage en direct requis** — ce qui lui permet de fonctionner à la fréquence de capture maximale de l’appareil photo. Écrit les images brutes + un manifeste par image + un fichier `.daq` par lecture DLS distincte sous `<output>/bursts/<base>/`. Re-traitez hors ligne (commande suivante), ou passez `--build` pour le faire immédiatement à l&#x27;arrêt.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--array-id ID` | tableau uniquement | Tableau cible. |
| `-o, --output DIR` | `output` | Répertoire de sortie (les données en rafale sont stockées dans `<DIR>/bursts/<base>/`). |
| `--duration S` | jusqu’à Ctrl+C | Arrêt automatique après `S` secondes. |
| `--max-frames N` | illimité | Arrêt automatiqueaprès `N` images brutes. |
| `--orient` | désactivé | Les produits générés par une compilation à partir de cette rafale sont orientés en fonction de l’orientation de la caméra — les produits par caméra sont orientés selon celle de leur propre caméra, et l’index combiné selon celle de la caméra maître — comme le fait `array-capture --orient`. Les images brutes restent au format natif ; le `manifest.json` de la rafale enregistre l’orientation et l’option de chaque élément. |
| `--build` | désactivé | Après l’arrêt, retraiter immédiatement la rafale (identique à `array-build-video`). |
| `--products …` | `combined:index` | Avec `--build` : quelle(s) vidéo(s) générer (voir ci-dessous). |
| `--fps F` | `10` | Avec `--build` : nombre d&#x27;images par seconde (fps) de la vidéo de sortie. |
| `--save-tiffs` | désactivé | Avec `--build` : enregistrer également des fichiers TIFF calibrés image par image (chacun avec son [masque de limitation unmix](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) lorsqu’un pixel est écrêté ; le résultat de la compilation les répertorie sous le nom `clamp_masks`). |
| `--gif` | désactivé | Avec `--build` : écrit également un ou plusieurs GIF animés. |

### `array-build-video` — retraite hors ligne d’une rafale enregistrée

Aligne temporellement chaque image brute sur la lecture `.daq` enregistrée la plus proche et la fait passer par la **même chaîne de radiance / réflectance / indice que le pipeline d’importation**, générant ainsi une ou plusieurs vidéos.

`--products` est une liste séparée par des virgules d’éléments `kind:level`, où `kind` ∈ `per_cam` | `combined` et `level` ∈ `radiance` | `reflectance` | `index`. Un `level` seul (sans `kind:`) prend par défaut la valeur `per_cam`. La valeur par défaut est `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Indicateur | Valeur par défaut | Description |
| --- | --- | --- |
| `--burst-dir DIR` | (obligatoire) | Chemin d&#x27;accès au dossier « burst » (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`, comme ci-dessus. |
| `--fps F` | `10` | Nombre d&#x27;images par seconde (fps) de la vidéo de sortie. |
| `--save-tiffs` | désactivé | Enregistrer également des fichiers TIFF calibrés image par image en même temps que la ou les vidéos. |
| `--gif` | désactivé | Écrire également des GIF animés. |

**Une vidéo ne contient pas de masque de limitation** (un flux d’images, 8 bits). À la place, le résultat de la compilation contient `clamp_fractions` : pour chaque produit vidéo, la part de pixels de chaque image que le démixage a confinée dans la bande de son canal R / G / B — le produit `Chloros:Band{R,G,B}UnmixClampFraction` de radiance/réflectance de cette image qui serait apposé (pour l’indice combiné, celui de chaque caméra composant le groupe). L&#x27;exécution affiche la pire image par produit sur une ligne `Clamp:`. `--save-tiffs` ajoute les masques par pixel pour les fichiers TIFF, dans le fichier `clamp_masks/<level>/` situé à côté du dossier `<level>/` contenant les fichiers TIFF.**Une compilation interrompue ne laisse aucune vidéo incomplète.** Chaque vidéo est enregistrée sous le nom `<name>.partial.avi` et ne prend son nom définitif qu’une fois terminée ; ainsi, une ancienne vidéo portant ce nom reste en place jusqu’à ce que sa version de remplacement soit complète. Relancez la compilation pour remplacer un fichier `.partial.avi` resté en suspens.

L&#x27;exécution affiche chaque vidéo qu&#x27;elle a enregistrée, puis tout produit `Skipped:` (une caméra/un niveau demandé — ou l&#x27;indice combiné — qui n&#x27;a produit aucune vidéo, avec la raison — par exemple, absence de pack d&#x27;étalonnage, aucune lecture DLS pour la réflectance, ou aucune paire de bandes rouge/NIR pour l’indice combiné) ainsi que tout fichier `Note:`. Une caméra RGB pour laquelle une radiance ou une réflectance a été demandée reçoit à la place sa vidéo d’affichage, avec une balance des blancs si le pack d’étalonnage de la caméra est disponible ; une caméra fonctionnant sans celui-ci est mentionnée dans un `Note:`. Elle **renvoie une valeur non nulle** lorsqu’un produit demandé a été ignoré ou n’a pas été généré comme demandé (une expression d’indice configurée n’est pas évaluée hors ligne) ; les vidéos qui ont été générées sont tout de même écrites et répertoriées.

> **Choisissez l’enregistreur adapté.** `array-record` est de *niveau surveillance* — il enregistre le composite en direct rendu : la vue combinée de l’interface graphique, ou, si aucune interface graphique n’est ouverte, la vue NDVI (ou `--expr`). `array-burst` → `array-build-video` est de *niveau analyse* — il enregistre les données brutes du capteur à plein débit et reconstitue ensuite des vidéos calibrées de radiance, de réflectance et d’indice, sans nécessiter de vue en direct.

### Caméras mono (M3M) à bande unique

La gamme **M3M**est la version monochrome de la gamme Bayer**M3C**: un filtre d’interférence à bande étroite par caméra (`M3M-<lens>-F<wavelength>`, par exemple `M3M-L87-F685`), le capteur fournit donc une**seule bande en niveaux de gris**, sans mosaïque de Bayer. Il n’y a pas de démosaïquage à effectuer, pas de diaphonie inter-canaux à démêler, ni de balance des blancs à régler — tout le processus de traitement des couleurs pour l’affichage en RGB ne s’applique tout simplement pas.

Ce que cela signifie sur la CLI :

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**détectent une caméra monochrome et**passent à la ligne suivante avec un message d’une ligne** au lieu d’imposer des paramètres inutiles. Ils fonctionnent toujours normalement avec une caméra M3C RGB/Bayer dans la même session.
- **Un format de pixels Bayer correspond à sa profondeur.** Un M3M ne propose que des formats mono ; ainsi, un format Bayer issu de `--pixel-format` ou `--preset` définit le format mono de la même profondeur : `BayerRG8` et `--preset high_speed` donnent `Mono8` ; `BayerRG12`, `high_quality` et `triggered` donnent `Mono12` ; un nom « packed » reste « packed » (`BayerRG12p` donne `Mono12p`). Les fichiers et `lattice format` désignent le format Mono. Un tableau mélangeant des caméras M3C et M3M adopte un format de la même manière : Bayer sur chaque M3C, sa profondeur Mono sur chaque M3M. Un nom correspond à sa profondeur de bits : l’inverse est également vrai, ainsi, une M3C n’est jamais placée sur un format Mono (`--pixel-format Mono12` correspond à `BayerRG12` sur une M3C, quelle que soit la caméra qui sert de maître au réseau), et `lattice format --set` est mappé de la même manière et génère le format qu’il a défini.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fonctionnent toujours — la radiance et la réflectance sont des cartes radiométriques *par bande* et sont parfaitement bien définies pour une seule bande. Les images monochromes comportent une matrice de réponse du capteur **d’identité** (pas de décomposition 3×3), de sorte que le plan passe à travers les calculs d’étalonnage sans être modifié.
- **Une seule caméra monochrome ne peut pas produire d’indice de végétation.**Les indices NDVI, NDRE, etc. nécessitent au moins deux bandes (par exemple, Red + NIR). Pour obtenir un indice à partir d’un matériel monochrome, pointez**plusieurs** caméras M3M sur différentes longueurs d’onde, alignez-les en une seule pile multibande, puis calculez l’indice à partir de *celle-ci* :

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` ouvre les caméras lui-même (une [session transitoire](#multi-camera-transient-sessions)), il ne faut donc rien connecter dans un premier temps : `--serials`, sinon les caméras activées dans le profil (voir [Index / Mathématiques de la végétation](#index--vegetation-maths)). Tant qu’Chloros détient une caméra (l’interface graphique, `cam-connect`, `array-connect`) la demande est refusée avec un code HTTP 409 `CAMERA_HELD` et le programme se ferme `1` ; libérez-les avec `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

Les symboles `--channel` doivent correspondre **exactement** aux noms des canaux du préréglage (sensible à la casse ; les NDVI sont en minuscules `red`, `nir` — voir `--list-presets`), et le nom de la bande doit correspondre à une bande de la pile alignée (le mode hors ligne accepte également les indices de bande à partir de 0, par exemple `--channel red=0 --channel nir=1`).

Le discriminant dans toute la pile est le jeton `M3M` dans la chaîne de modèle (il n’apparaît jamais dans une chaîne `M3C`), présenté à l’interface graphique/au SDK sous la forme `is_mono`.

---

## Configuration et réglage de la carte réseau de l’hôte (réseaux de caméras LATTICE)

Les caméras LATTICE transmettent le flux GVSP via la carte Ethernet de l’hôte ; ainsi, pour les réseaux de caméras multiples, le **pilote**et la**taille de l’anneau de réception** sont tout aussi importants que le débit de la liaison. Des paramètres incorrects se traduisent par l’apparition d’une erreur `FRAMES WILL DROP` / `Reduce ROI to enable` dans le panneau « Paramètres du réseau de caméras » (ainsi que dans `lattice network-analysis` / le SDK `analyze_array_network()`), même lorsque les caméras elles-mêmes fonctionnent correctement.

### Adaptateurs USB 10 GbE — Realtek RTL8157 (« Realtek USB 10GbE Family Controller »)

Les paramètres ci-dessous correspondent aux propriétés du pilote Windows. Sous Linux, un adaptateur USB est également soumis à une limite (400 Mo/s sur x86_64, 200 Mo/s sur NVIDIA Jetson) (voir [Modèle d’fps et de rafales des matrices](#array-fps--burst-model)), et les tampons de réception des sockets du noyau ont une importance pour chaque adaptateur — voir [Hôtes Linux](#linux-hosts--socket-receive-buffers).

| Élément | Valeur requise | Pourquoi c’est important |
| --- | --- | --- |
| **Version du pilote**|**≥ v10.67 (janvier 2026)**, INF `rtump64x64sta.inf` | L’ancien pilote**2016**(v10.65, `rtump64x64.inf`) gère mal la mise hors tension et les bogues avec**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**lors de l’arrêt, du redémarrage ou de la mise en veille. La transition se bloque (délai d’attente d’environ 5 minutes), l’utilisateur procède à un arrêt forcé, et ces arrêts non corrects répétés**corrompent le référentiel WMI**(PowerShell et les outils commencent à échouer avec `Invalid class`) et**bloquent la pile USB** lors du démarrage suivant (la carte réseau ne s’active pas ; les périphériques USB cessent d’être répertoriés). Effectuez une mise à jour depuis realtek.com (ou auprès du fournisseur de la clé) avant de compter sur des redémarrages propres. |
| **Tampons de réception**— mot-clé `ReceiveBufferLen` |**256**(maximum du pilote) | L&#x27;anneau de réception de la carte réseau. La valeur par défaut du pilote,**32**, ne laisse qu’environ 0,26 Mo d’anneau utilisable — bien trop peu pour une rafale multi-caméras —, ce qui fait que le panneau de configuration signale `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` et bloque les connexions. À**256**, le ring est vaste (**environ 13,5 Mo mesurés sur l’hôte 10 GbE du laboratoire**), ce qui offre au pipeline de réception une réelle marge de manœuvre pour les rafales GVSP multi-caméras. (La décision quant à savoir si une configuration donnée *établit* effectivement une connexion repose sur deux vérifications : le contrôle d’**admissibilité tenant compte du drain**et le contrôle de**sursouscription agrégée** — et non d’une simple comparaison brute entre la rafale et l’anneau ; voir [Modèle de fps et de rafales de la matrice](#array-fps--burst-model).) |
| **URB de réception**— mot-clé `PendingReceives` |**64** (max.) | Blocs de requêtes USB en transit ; à augmenter parallèlement aux tampons de réception pour l’absorption des rafales. |
| **Trame Jumbo** — mot-clé `*JumboPacket` | **9014** | Nécessaire pour les paquets GVSP de 9 000 octets (6 fois moins de paquets par trame qu’avec 1 500 octets). |

> ⚠️ **Une mise à jour du pilote de la carte réseau RÉINITIALISE ces propriétés avancées à leurs valeurs par défaut.**Après avoir mis à jour ou remplacé le pilote de la carte,**réappliquez** `ReceiveBufferLen=256` et `PendingReceives=64`, sinon le panneau de la matrice se bloquera à nouveau même si « rien n’a changé au niveau du matériel ». C’est la cause n° 1 pour laquelle un équipement qui fonctionnait auparavant refuse soudainement de se connecter.

Appliquez ces commandes depuis un PowerShell **avec des privilèges élevés** (remplacez par le nom de votre carte, par exemple `"Ethernet 5"`) :

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` concerne les adaptateurs USB 10 GbE.** Il détecte désormais le type d’adaptateur et règle le mot-clé « receive-ring » approprié : `*ReceiveBuffers`→2048 pour les cartes réseau PCIe (Intel I219, etc.), ou `ReceiveBufferLen`→256 + `PendingReceives`→64 pour le contrôleur **USB** 10 GbE de Realtek (qui n’expose pas `*ReceiveBuffers`). Les valeurs cibles sont limitées à la valeur maximale signalée par chaque pilote (`NumericParameterMaxValue`), de sorte qu’aucune valeur hors plage ne soit jamais écrite. Exécutez cette commande depuis un terminal **avec des privilèges élevés** ; comme pour tout réglage basé sur le registre, la modification prend effet après un redémarrage de la carte réseau ou un redémarrage du système. Les commandes manuelles `Set-NetAdapterAdvancedProperty` ci-dessus restent une bonne alternative : elles s’appliquent à la volée (en reconnectant la carte réseau) sans nécessiter de redémarrage.

### Machines Linux
— tampons de réception des sockets

Sous Linux
, y compris sur NVIDIA Jetson, les tampons de réception des sockets du noyau ont autant d’importance que l’anneau de réception de la carte réseau. Avec les valeurs par défaut du noyau (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 octets), une rafale de paquets GVSP provenant d’une caméra provoque un débordement du socket de flux avant qu’Chloros
’ait pu les lire. Les paquets sont perdus au niveau du noyau alors que la carte réseau elle-même ne signale aucune perte, et un réseau de caméras perd des groupes d’images et réduit sa fréquence d’images. Mesuré sur un ensemble de 4 caméras connectées à un adaptateur USB 10 GbE, l’augmentation de ces deux valeurs à 32 Mo a fait passer le débit de l’ensemble de 4,96 à 7,52 images par seconde après la connexion, et le débit de capture brute continue de 4,7 à 7,6 captures par seconde, sans aucune perte de paquets au niveau du noyau.

- **Le fichier .deb les configure.** L’installation du paquet écrit les valeurs `/etc/sysctl.d/60-chloros-gige.conf` (les deux à `33554432`) et les applique immédiatement. Une valeur déjà définie à un niveau supérieur n’est pas modifiée.
- **`lattice network` les vérifie.** Lorsque l’une ou l’autre de ces valeurs est inférieure, `socket receive buffers too small for GigE Vision (…)` apparaît dans la liste *Problèmes détectés*, et le journal du backend affiche le même avertissement lors de la première connexion d’une caméra ou d’un réseau de caméras.
- **Pour les augmenter manuellement** (jusqu’au prochain redémarrage ; le fichier du paquet les applique à chaque démarrage) :

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` les augmente et les maintient également, mais uniquement lorsque le backend s’exécute en tant que root. Un backend s’exécutant sous votre compte utilisateur (lancé avec `chloros-backend`, ou [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) ne peut pas modifier les paramètres du noyau et affiche à la place la ligne `sudo sysctl`.

**Le réglage automatique des cartes réseau ne concerne que les cartes connectées.** Lors de la première connexion d’une caméra, un backend s’exécutant en tant que root règle les cartes réseau filaires dont la taille de l’anneau de réception est faible. Désormais, il ne règle que les adaptateurs disposant d’une liaison ; ainsi, un port intégré débranché conserve son MTU, et les tampons de réception ne sont pas modifiés. `lattice network` ne signale plus le MTU d’un adaptateur débranchécomme un problème. `lattice network`, `--fix`, `lattice probe` et l’optimisation des adaptateurs ne prennent en compte que les ports Ethernet filaires, et non le bus CAN d’un Jetson (`can0`) ni les liaisons en mode périphérique USB (`usb0`, `usb1`) qu’il propose à un PC hôte ; une carte Ethernet USB pour caméras est toutefois prise en compte.

### Notions de base sur le réseau (tous les liens LATTICE)

- **Adressage :** adresse locale de liaison `169.254.0.0/16` (GigE Vision LLA). L’hôte utilise une adresse statique `169.254.x.x/16` ; les caméras et le DAQ-E s’attribuent automatiquement des adresses dans la même plage. Aucun DHCP ni passerelle n’est requis.
- **Taille des paquets :**privilégiez les paquets jumbo (9 000), mais laissez la détection automatique la déterminer — elle contourne la limite ICMP de 1 500 octets de la caméra via une sonde GVSP, de sorte qu’elle aboutit à la taille jumbo partout où la ligne la prend réellement en charge.**Chaque caméra est mesurée une fois par session du backend, puis à nouveau lorsque sa liaison change (autre adaptateur, autre débit de liaison ou coupure de liaison, comme lors du remplacement d’un commutateur) ; les reconnexions réutilisent le résultat. Après un changement derrière un commutateur resté opérationnel, redémarrez le backend pour effectuer une nouvelle mesure.**Une première connexion peut parfois prendre quelques minutes lors de cette vérification, et le journal l’indique lorsque c’est le cas. N’utilisez le pin `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` que si vous en savez plus que la sonde, et privilégiez une configuration par commande plutôt qu’une configuration permanente : un pin contourne la sonde, donc si le chemin ne peut pas réellement supporter 9000,**chaque** capture expire avec `SC_ERR_TIMEOUT -1011` (voir [Variables d’environnement](#environment-variables)).
- **La taille de l’anneau RX évolue en fonction de `ReceiveBufferLen` :**à la valeur par défaut `32`, l’anneau utilisable est d’environ 0,26 Mo (trop petit pour toute rafale multi-caméras) ; à la valeur maximale `256`, il est grand (environ 13,5 Mo mesurés sur l’hôte 10 GbE du laboratoire), ce qui offre une réelle marge de manœuvre. La possibilité de connecter une configuration est alors déterminée par le contrôle d’admissibilité tenant compte de la consommation**et** par le contrôle de sursouscription agrégée ci-dessous — et non par une simple comparaison brute entre la rafale et l’anneau.

### Modèle de fps et de rafales de la matrice

Comment lire le panneau « Array Settings » (et `lattice analyze-array` / `analyze_array_network` de l’SDK
) :

- **La rafale est cumulée par caméra selon le format de pixels réel de chaque caméra.**Les caméras mono**M3M**diffusent en**Mono12 (2 bits/px)**, ou, avec `--pixel-format`, au format mono de leur profondeur (`BayerRG8` correspond à `Mono8`, 1 bit/pixel) ; les caméras Bayer**M3C**transmettent en 8 ou 12 bits (le micrologiciel M3C émet silencieusement du BayerRG12 même lorsque du BayerRG8 est demandé). Ainsi, une image en pleine résolution provenant de 4 caméras pèse**environ 12,6 Mo si toutes sont en 8 bits, mais environ 25 Mo avec trois caméras mono en 12 bits**. La projection détermine le format de chaque caméra à partir de son modèle (cache d’identité), de sorte que la rafale corresponde à ce que le câble transporte réellement — et non à une hypothèse « taille unique » de BayerRG8.
- **Un adaptateur Ethernet USB est limité à 200 Mo/s, quelles que soient ses spécifications nominales.** Le tableau d’efficacité qui convertit un débit de liaison en un débit soutenu est dérivé du PCIe ; une carte réseau USB annonce son débit de liaison *Ethernet*, mais est limitée par le bus USB et son pilote. Une clé USB 10 GbE affichait auparavant un débit d’environ 1 063 Mo/s « en continu » — un chiffre qui n’a jamais été vérifié — et la régulation de débit qui en résultait corrompait 6 à 18 % des trames tout en affichant un nombre d’images par seconde cible satisfaisant. Les cartes réseau connectées via USB sont désormais limitées de manière absolue : **200 Mo/s** sous Windows
et sur NVIDIA Jetson, **400 Mo/s** sous Linux
x86_64 (sous Linux
, une carte connectée à un port USB 2.0 est limitée à **30 Mo/s**, et la fiche technique recommande de la brancher sur un port USB 3) (la limite est imposée par le bus USB et le pilote de l’hôte ; elle n’est donc pas proportionnelle aux spécifications techniques. Une carte USB 1 GbE atteint environ 80 Mo/s et n’est pas affectée). Sous Linux
, l’adaptateur est identifié via sysfs (en recourant aux identifiants `ethtool -i` et `bus-info`). Chaque chiffre est mesuré : sous Windows
, le même ensemble de 4 caméras a corrompu 6 à 10 % des images au-delà de 200 Mo/s ; sous Linux
x86_64, la même carte a fonctionné sans problème à tous les plafonds jusqu’à 480 Mo/s (273 Mo/s fournis, la limite propre aux caméras). Sur NVIDIA Jetson, ce même adaptateur n’a fonctionné sans perte qu’à 200 Mo/s et a perdu des images de manière intermittente au-delà de ce débit. `wire_ceiling_source` dans le fichier de capacités l’indique explicitement, et `nic_is_usb` le signale. Vous pouvez passer outre dans les deux cas avec `--wire-ceiling-mbps`.
- **L’admittance tient compte du drain, et non de la différence entre rafale complète et anneau.** Une rafale simultanée doit uniquement tenir dans le *backlog transitoire* = `max(0, Σ per-cam arrival − host drain) × emit_window`, et non dans la rafale entière. Sur une structure hôte rapide / caméras lentes (un **PCIe**hôte 10G + 4 caméras 1 GbE : arrivée ≈ 320 Mo/s, vidange ≈ 1 063 Mo/s), l’hôte évacue plus vite que les caméras ne se remplissent, le backlog est ≈ 0 ; ainsi, l’émission simulée en pleine résolution**est acceptée**même si la rafale de 25 Mo dépasse l’anneau de 13,5 Mo. Placez ces mêmes quatre caméras derrière un adaptateur**USB** 10 GbE sous Windows
, et le débit de sortie est de 200 Mo/s, et non de 1 063 — l’arrivée le dépasse, et la perte se manifeste sous forme de trames corrompues plutôt que par une baisse de la fréquence d’images. Sur un hôte 1 GbE, le seuil DLThr de 31,25 Mo/s des caméras fait que le débit d’arrivée dépasse le débit de sortie → le système **bloque** correctement (pour *cette* catégorie de blocage, réduisez le ROI ou utilisez un binning ≥ 2). L’admission est l’une des **deux** portes de connexion — l’autre est le contrôle de sursouscription agrégée ci-dessous.
- **Le nombre d’images par seconde (fps) prévu correspond au cycle de la caméra la plus lente.**En émission simultanée, chaque caméra envoie sa trame en même temps ; le réseau fonctionne donc au rythme du plus long entre le temps de lecture du capteur et le temps d’émission de son élément le plus lent (octets de la trame ÷ son débit d’émission : la**liaison d’accès** de la caméra, 1 GbE ≈ 80 Mo/s, ou sa part du plafond de bande passante de l’hôte lorsque celui-ci est inférieur). Chaque exposition chevauche la lecture de l’image précédente ; l’exposition n’entre donc pas dans le cycle : l’exposition automatique occupe la période définie par le débit, puis augmente le gain, et une scène sombre ne réduit plus le débit (ce qui était le cas dans la version 1.2.0). Seule une exposition manuelle plus longue que cette période ralentit le déclenchement (voir [Exposition sur un élément du réseau](#exposure-on-an-array-member)). L’ajout de caméras ne ralentit pas le réseau tant que la part de bande passante de chacune d’entre elles ne diminue pas. En émission série (`slip-emit-and-capture`), les temps d’émission des caméras s’additionnent, et l’exposition de chaque élément est prise en compte dans l’échelonnement. Mesure effectuée avec un capteur à pleine capacité, en 12 bits, sur un réseau de 9 caméras avec un hôte PCIe 10 GbE : 10,7 images par seconde, ou 14,0 images par seconde avec un [transport compressé](#packed-12-bit-transport---packed).
- **Après une capture continue, la projection indique la fréquence atteinte.** Une capture de 30 s ou plus à une fréquence automatique enregistre la fréquence à laquelle le système s’est stabilisé pour cette taille de réseau sur cet ordinateur ; les brefs essais du contrôleur de fréquence à une cadence plus rapide sont ignorés. Dès lors, la ligne `array-connect` et le panneau « Paramètres du tableau » indiquent ce débit lorsqu’il est inférieur à celui du modèle, en précisant ce qui l’a limité : `clock-bound` (le tableau a fonctionné au débit auquel il s’est stabilisé, la boucle de capture attendant son signal d’horloge de déclenchement), `loop-bound` (la boucle de capture n’avait plus de temps), `capture-bound` (la capture a pris du retard et a sauté des groupes) ou `measured` (rien ne précise laquelle). La valeur du modèle se trouve dans `wire_fps_max`. Le tableau atteint toujours la fréquence maximale supportée par la liaison.
- **Un réseau fonctionne sur n’importe quel réseau — c’est la fréquence d’images qui fait la différence.**Chaque caméra reçoit une part égale du**plafond de la ligne sans risque de collision**(`sustained × sim_emit_factor`), synchronisée avec GevSCPD, et toutes continuent de déclencher à la même fréquence. Sur une liaison lente, la part est faible et la fréquence d’images diminue en conséquence : 8 caméras 12 bits en pleine résolution sur un réseau 1 GbE sans trames jumbo obtiennent 52 / 8 = 6,5 Mo/s chacune, soit environ**1 image par seconde**(environ 1,3 image par seconde compressée), alors qu’un hôte 10G les fait fonctionner à un débit de 11 à 14 images par seconde. Si les caméras ne peuvent pas être synchronisées à un rythme aussi lent — connectez-vous et lisez la plage GevSCPD de chaque caméra pour le vérifier —, le réseau passe en mode d’émission**série**(`slip-emit-and-capture`) : les caméras se relaient sur la ligne à la vitesse maximale de la liaison, déclenchées les unes après les autres, de sorte que chaque trame arrive toujours mais que les éléments ne sont**pas exposés simultanément**. `analyze-array` et la sortie « connect » indiquent le mode ; définissez `--force-tier sim-capture-sim-emit` sur « pin » pour refuser le repli. La taille de trame n’intervient pas dans ce partage ; ainsi,**le regroupement de pixels et une zone d’intérêt (ROI) plus petite augmentent la fréquence d’images mais ne changent rien à la capacité de l’array à s’adapter**. Seul un câble incapable de transporter le tableau, même en série, est**surchargé**— par exemple, lorsqu’il y a bien plus de caméras qu’un câble ne peut en prendre en charge, ou que de nombreuses autres caméras y transmettent des flux — et dans ce cas, le symptôme est une perte de paquets GVSP, et non un débit réduit : `analyze-array` remet à zéro les valeurs d’images par seconde (fps) réalisables et affiche `** OVER-SUBSCRIBED**` ; la connexion se poursuit avec un avertissement sonore, et une connexion HTTP
avec `pin_resolution: true` (mode strict) **est refusée**. Les solutions consistent à réduire le nombre de caméras, à utiliser des trames jumbo de bout en bout ou à utiliser une carte réseau plus rapide. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` ramène le refus au niveau d’un avertissement sonore pour les tests de performance — voir [Variables d’environnement](#environment-variables).

### État de la baie — quel sous-système perd des trames

Le code `GET /api/camera/array/<array_id>/capability` d’une baie connectée comporte un bloc
`health` en temps réel, réévalué sur une fenêtre glissante de **10 secondes**. Il ventile les pertes de trames
en deux causes nécessitant des correctifs opposés, plutôt que de signaler un seul taux « incomplet »
qui n’en identifie aucune :

| Champ | Signification | Sous-système concerné |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (par port série) | La trame **est arrivée mais était structurellement incorrecte**— perte de paquet GVSP. |**Réseau** : bande passante, cadencement, anneau de réception de la carte réseau, MTU |
| `never_arrived_rate_pct` (par numéro de série) | La trame **n’est jamais arrivée**— la caméra n’a pas déclenché, ou rien n’en est sorti. |**Déclenchement / synchronisation** : câble M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Taux le plus défavorable pour chaque caméra. | — |
| `per_cam_rate_pct` | Taux combiné d’incomplétude par caméra (les deux causes confondues). | — |
| `stable_for_seconds` | Durée pendant laquelle chaque caméra est restée en dessous de 0,01 %. | — |

Au-delà de 5 %, le backend enregistre une ligne `[array-health <id>] WARN` indiquant la répartition — lors du
premier dépassement, lors d’un changement de niveau de gravité, une fois par minute tant que la situation persiste, et une fois lorsqu’
elle se résout. La partie défectueuse écrit « `[gvsp-corrupt <SN>]` » dans le fichier de journal du backend lors du
premier événement par caméra et par motif, puis un résumé toutes les 60 s ; le panneau de journalisation de l’application affiche à la place la
ligne `[array-health <id>]` à la place. Chaque évaluation est toujours consignée dans le fichier journal du backend ;
les compteurs avancent à chaque tampon, indépendamment de ce qui est affiché.

Le même enregistrement indique le nombre sur lequel repose toute l’allocation :

| Champ | Signification |
| --- | --- |
| `wire_ceiling_mbps` | Le budget de bande passante soutenu de l’hôte actuellement en vigueur, en Mo/s. |
| `wire_ceiling_source` | D&#x27;où provient ce chiffre, en mots — par exemple `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (sous Linux
x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` lorsque `--wire-ceiling-mbps` (ou le champ **Budget de bande passante** de l’interface graphique) le définit. |
| `nic_is_usb` | `true` pour une carte Ethernet USB — voir la limite USB ci-dessus. |

**Interprétation :** une valeur non nulle pour `gvsp_corrupt_rate_pct` avec `never_arrived_rate_pct` à 0
signifie que le déclenchement et la synchronisation du câble sont parfaits et que 100 % de la perte se situe sur le chemin réseau
— réduisez la valeur de `--wire-ceiling-mbps` et reconnectez-vous. La situation inverse indique plutôt un problème au niveau du
câble de synchronisation ou de la ligne de déclenchement.

> **`--target-fps` n’est pas le paramètre déterminant pour les trames corrompues.** Le rythme GevSCPD est défini
> une seule fois à la connexion ; ainsi, réduire la fréquence de déclenchement modifie le rapport cyclique et non le
> débit de rafales à émission simultanée. Une réduction mesurée de 5× de la demande n’a apporté aucune amélioration ;
> la réduction du plafond de la ligne de 240 à 200 Mo/s a fait passer ce même système de 10,4 %
> de trames corrompues à 0,00 %.

> **Un réseau en fonctionnement ne peut pas se replanifier lui-même sur le micrologiciel LATTICE (M3C / M3M).** Il
> ne peut pas résoudre ce problème de lui-même ; déconnectez-le puis reconnectez-le afin que le sélecteur de temps de connexion puisse
> se replanifier avec le nouveau plafond.

### Symptôme → solution

| Symptôme (Paramètres de la matrice / connexion / `analyze_array_network`) | Cause | Solution |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` réinitialisé à 32 (généralement après une mise à jour du pilote) | Définir `ReceiveBufferLen`→256, `PendingReceives`→64 ; rouvrir le panneau (redémarrer le backend s’il a mis en cache l’ancienne taille de l’anneau) |
| Blocage lors du redémarrage/de l’arrêt ; par la suite, erreurs WMI `Invalid class`, la carte réseau ne s&#x27;active pas, les périphériques USB sont absents | Ancien pilote Realtek USB 10 GbE de 2016 → écran bleu (BSOD) `0x9F` → arrêts forcés | Mettez à jour le pilote de la carte réseau vers la version v10.67 (2026), puis réappliquez les paramètres de l&#x27;anneau de réception ci-dessus |
| La matrice se connecte à la pleine capacité du capteur mais avec une faible fréquence d’images | Le mode « Auto » conserve la pleine capacité du capteur et laisse la fréquence chuter au niveau de ce que la liaison peut supporter | Choisissez l’une des options les plus rapides parmi celles répertoriées dans la liste des sorties de connexion (`--binning 2` / `4`, ou un `--width`/`--height`), activez les trames jumbo ou mettez à niveau la liaison |
| L’Array signale un nombre d’images par seconde (fps) cible correct mais n’en fournit qu’une fraction ; `health.gvsp_corrupt_rate_pct` différent de zéro, `never_arrived_rate_pct` 0 | Le budget de bande passante estimé de l’hôte est supérieur à ce qu’il peut réellement supporter (cas typique avec une carte Ethernet USB, une voie PCIe étroite ou une structure partagée) | Se reconnecter avec une valeur `--wire-ceiling-mbps` inférieure et revérifier l’état de santé du bloc. **Pas** `--target-fps` — La régulation GevSCPD est verrouillée lors de la connexion |
| Linux
: un tableau perd des groupes de trames et réduit son débit alors que la carte réseau ne signale aucune perte ; `lattice network` indique `socket receive buffers too small` | Les tampons de réception des sockets du noyau sont à leur valeur par défaut, ce qui entraîne la perte de paquets de la caméra au niveau du noyau | Augmentez la valeur de `net.core.rmem_max` / `rmem_default` à 32 Mo — voir [Hôtes Linux
](#linux-hosts--socket-receive-buffers). Le fichier .deb effectue cette opération lors de l’installation |
| Caméras manquantes dans les groupes publiés ; `health.never_arrived_rate_pct` différent de zéro, `gvsp_corrupt_rate_pct` 0 | Déclencheur / chemin de synchronisation — les caméras ne se déclenchent pas, ce n’est pas un problème réseau | Vérifiez le câble de synchronisation M8 et `--line` ; assurez-vous que chaque élément est activé (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` dépassé dans `analyze-array`, ou refus de connexion en mode strict (`pin_resolution` ; `array over-subscribes the wire`) | Le câble ne peut pas acheminer le flux, quelle que soit la cadence atteinte par les caméras, simultanément ou à tour de rôle — il y a bien plus de caméras que le câble ne peut en prendre en charge, ou d’autres caméras diffusent déjà sur ce câble | Moins de caméras, des trames jumbo de bout en bout, ou une carte réseau plus rapide. **Le ROI/binning n’aidera PAS** (le partage est indépendant de la taille des trames). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` prend le pas en environnement de test (accepte la perte de paquets) |
| Le réseau de caméras se connecte via `slip-emit-and-capture` (série) sur un réseau lent | Les caméras ne peuvent pas être synchronisées à un rythme suffisamment lent pour partager le câble simultanément ; elles fonctionnent donc à tour de rôle ; les trames ne sont pas exposées au même instant | Des trames jumbo ou une carte réseau plus rapide rétablissent la capture simultanée ; `--force-tier sim-capture-sim-emit` insiste pour le faire (et perd des paquets à la place) |

---

## `chloros-cli daq`

Commandes du capteur spectral. Les commandes `pool-*` sont des clients HTTP
« légers » qui pilotent le capteur via le pool persistant du backend ; elles nécessitent donc un backend en cours d’exécution (voir [Démarrer le backend](#start-the-backend)). Le backend gère le transport ; ainsi, les scripts CLI
et SDK
partagent un même descripteur actif au lieu de se disputer le port série. `pool-discover` est le seul moyen de récupérer l&#x27;adresse MAC BLE d&#x27;un DAQ-M.

> **`chloros-cli daq --help`** (ou `-h`, ou `daq help`) répertorie les sous-commandes `pool-*` (sous Linux
, ainsi que les commandes matérielles directes incluses dans la compilation), et `chloros-cli daq <subcommand> --help` (ou `daq help <subcommand>`) présente les options de chacune d’entre elles. Elles couvrent la connexion, stream, l&#x27;enregistrement de fichiers `.daq` calibrés, l&#x27;échange de profils de capacité et l&#x27;association d&#x27;un DAQ à une caméra ; pour un DAQ-E, elle permet également de configurer ses paramètres réseau, le PTP, de le redémarrer et de mettre à jour son micrologiciel. Le DAQ peut également être piloté depuis Python
avec `chloros_sdk.connect_daq_sensor()`, qui utilise le même chemin mutualisé.
>
> **Utilisez les commandes `pool-*` sur toutes les plateformes.** Elles se comportent de la même manière sous Windows
, Linux
et Jetson. Sous Windows
, `daq discover` et `daq list` sont des alias de `pool-discover`, et toute autre commande `daq` qui n’est pas `pool-*`, fait sortir `1` avec `DAQ direct-hardware commands require the full daq package`. Sous Linux
et Jetson, les commandes `daq discover`, `daq list` et les autres commandes `pool-*` et `daq` qui ne sont pas de type `daq discover` constituent les commandes matérielles directes: elles ouvrent le capteur dans le processus propre à l’CLI
, en dehors du pool du backend (donc pas sur un capteur géré par le backend, l’interface graphique ou un script), et `daq discover` y accepte `--only` et `--timeout` mais pas `--json`. Cette référence ne documente que les commandes `pool-*`.

### Workflow de première connexion au capteur DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Référence `pool-*`

| Sous-commande | Objectif |
| --- | --- |
| `daq pool-connect` (smart-detect) | Ouvrir un capteur dans le pool de backend. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U sur un port série spécifique. |
| `daq pool-connect --ble` | DAQ-M via BLE, adresse MAC détectée automatiquement. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M via BLE sur une adresse MAC connue (implique `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet avec un nom d’hôte ou une adresse IP connue (implique `--eth`). |
| `daq pool-connect --eth` | DAQ-E via Ethernet, hôte détecté automatiquement (mDNS + repli sur ARP ; fonctionne à partir d&#x27;un cache ARP vide sous Windows
et Linux
). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Réglage de la fenêtre d’intégration / de l’état AE. |
| `daq pool-connect --no-stream` | Se connecter mais ne pas encore démarrer le streaming (reprendre avec `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil de correction Cap. La valeur par défaut au niveau du backend est `sunshine_cosine` ; `none` correspond à un capteur nu. Un [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) l&#x27;ignore (diffuseur fixe). **Les capteurs sont livrés nus ; vous devez donc nommer le capuchon que vous avez installé :** la valeur par défaut est enregistrée comme étant (`cap_id_source=auto_default`) et le traitement émet un avertissement à ce sujet, car un DAQ-U nu soumis à la correction solaire affiche une valeur environ 30 fois supérieure à la normale. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Analysez chaque transport à la recherche de capteurs auxquels vous pourriez vous connecter, sans vous connecter. **C&#x27;est ainsi que vous trouvez l&#x27;adresse MAC BLE d&#x27;un DAQ-M.** `--timeout` s&#x27;applique par transport (par défaut 5 s, 1-20). Les capteurs déjà ouverts dans le pool ne sont pas répertoriés — un DAQ-M connecté cesse de s&#x27;annoncer — ; utilisez donc `pool-list` pour ceux-ci. `usb` répertorie uniquement les périphériques série USB : les ports série propres à l’ordinateur (`COM1`, `/dev/ttyS*` sous Linux
) ne peuvent pas être des capteurs et sont donc ignorés. |
| `daq pool-list` | Affiche tous les capteurs du pool du backend. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Associe une caméra ou un tableau à un DAQ, comme le fait la sélection de capteur de lumière de l’interface graphique : ses captures utilisent, font correspondre et enregistrent les mesures de ce DAQ. `SENSOR` est un identifiant de capteur provenant de `pool-list` ou l’adresse à laquelle le DAQ est accessible. `--clear` (pas `SENSOR`) supprime l&#x27;association, et les captures reviennent alors au DAQ le plus récent. La liaison dure jusqu’au redémarrage du backend, ou jusqu’à ce que cette caméra, ce tableau ou ce DAQ soit déconnecté : une déconnexion (ici, depuis le SDK
ou dans l’application) rompt toutes les liaisons avec le matériel déconnecté, quel qu’en soit l’auteur, y compris celles des éléments d’un tableau avec celui-ci. La caméra doit être connectée, figurer dans le projet ouvert, ou avoir été ouverte auparavant sur cet ordinateur, et le tableau doit être connecté ou figurer dans le projet ouvert ; tout autre numéro de série ou identifiant est rejeté par son nom et rien n’est lié. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Libération — quelle que soit la personne ayant ouvert le capteur ; cela libère donc également un capteur laissé dans le pool par un script qui a planté (tout enregistrement en cours sur ce capteur est d’abord terminé). Un capteur qui ne se trouve pas dans le pool n’a rien à libérer : il affiche `ID: already released (not in the pool)` et se ferme `0` ; une libération refusée ou non confirmée par le backend se termine par `1`. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Les N trames de spectre les plus récentes (1 par défaut). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reprendre / mettre en pause la diffusion en continu. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Démarrer / arrêter un enregistrement .daq. `-o` est défini par défaut sur `~/Documents/DAQ Live View` sur l&#x27;ordinateur du backend (un chemin relatif se trouve dans le dossier à partir duquel vous exécutez la commande) ; `--duration` (`-d`, par défaut `0` = jusqu’à `--stop`). `--confirm-after S` émet un avertissement lorsqu’aucune mesure n’est arrivée S secondes après le démarrage (valeur par défaut : 20 ; 0 désactive cette fonction) ; avec `--duration`, `--heartbeat S` affiche le nombre de mesures toutes les S secondes et émet un avertissement lorsque ce nombre cesse d’augmenter (valeur par défaut : 30 ; 0 désactive cette fonction). Un enregistrement sur un DAQ-E qui cesse de transmettre se poursuit dans le même fichier une fois que le backend a rétabli la connexion, et `--stop` répertorie les interruptions ([Un DAQ-E qui cesse de transmettre](#a-daq-e-that-stops-delivering)). La combinaison Ctrl+C met fin prématurément à un enregistrement `--duration` et le signale ; sans `--duration`, l’enregistrement continue après Ctrl+C (quitter `130`) jusqu’à `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Permet de changer de profil de correction de capacité pendant l’exécution. `--cap-id` est obligatoire ; `--cap-id none` supprime la correction de capacité (capteur nu). Un DAQ-E-S conserve `as_recorded` quel que soit le paramètre défini. |
| `daq pool-enable-ptp --sensor-id ID` | Réactive le PTP pour un DAQ-E dont le PTP est désactivé. Le paramètre est enregistré sur l’appareil, de sorte qu’un appareil laissé désactivé reste désactivé — ses horodatages ne sont pas synchronisés par PTP — jusqu’à ce que cette commande soit exécutée. `pool-list` signale un tel appareil (`!! PTP is switched OFF`) et affiche cette commande pour celui-ci. Chloros
ne réactive jamais un appareil de lui-même, car la désactivation peut être intentionnelle. |
| `daq pool-info --sensor-id ID [--json]` | Paramètres propres à un DAQ-E : son micrologiciel (et si cette version de Chloros
en propose un plus récent), son adresse et la manière dont il l’a obtenue (DHCP / link-local / statique), les paramètres réseau enregistrés jusqu’à ce que le capteur les exécute (ceux enregistrés avec `--no-reboot` inclus), le flux à la mise sous tension et le PTP. Sur un capteur dont le flux n’atteint pas cet ordinateur, il lit les données du capteur lors d’une nouvelle connexion et le signale. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Configurer un DAQ-E en DHCP (`--auto`) ou à une adresse statique (`--mask`, par défaut `255.255.255.0`). Le capteur redémarre pour appliquer ces paramètres et se reconnecte avec les paramètres qu’il avait auparavant, en attendant jusqu’à `--timeout` secondes (par défaut 45) ; `--no-reboot` se contente de l&#x27;enregistrer. Voir [Paramètres du dispositif DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Indique si un DAQ-E commence la transmission dès la mise sous tension sans logiciel connecté (enregistré sur le capteur). Chloros
lance la transmission de son propre chef dans tous les cas. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Redémarrer un DAQ-E et le reconnecter avec ses paramètres précédents (attente jusqu’à `--timeout`, par défaut 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Flasher le micrologiciel DAQ-E inclus dans cette version de Chloros
sur un capteur fonctionnant sous une version antérieure, puis le reconnecter (environ une minute ; ne pas le mettre hors tension ; `--timeout` 45 s par défaut pour qu’il se reconnecte). Un capteur déjà à jour n’est pas modifié. |

### Un DAQ-E qui cesse de transmettre

Un DAQ-E du pool qui ne transmet aucun spectre pendant 20 secondes est reconnecté par le backend. Cela s’effectue par un redémarrage du commutateur qui alimente le capteur : le capteur redémarre avec lui et ne retransmet pas tant qu’on ne lui demande pas de le faire (sauf si `pool-set-auto-stream --on` est activé).

- **Il vérifie d’abord qu’il s’agit bien du même capteur.** Il interroge l’adresse que le capteur occupait, sans rien modifier, pour obtenir le nom, l’adresse MAC et le numéro de série du capteur. Il interroge également sous le nom `daq-e-<id>.local` du capteur, mais ne fait confiance à cette réponse que si elle provient de la propre carte réseau (adresse MAC) du capteur. Aucun autre répondeur n’est reconnecté ni pris en compte.
- **Il se reconnecte en conservant les paramètres de la session :** temps d’intégration, moyennage des images, exposition automatique et profil de capture tels qu’ils étaient, et non les valeurs par défaut à la mise sous tension du capteur. Un capteur dont le PTP était activé et qui est revenu avec cette fonction désactivée est réactivé. Ensuite, il redémarre le flux et le récepteur de l’ordinateur qui lui est dédié.
- **Il effectue de nouvelles tentatives à des intervalles croissants :** immédiatement, puis après 5, 10, 20 et 40 secondes, puis toutes les 60 secondes tant que le capteur est indisponible. À chaque tentative, le capteur est simplement interrogé sur son identité. Un capteur qui répond mais qui n’envoie toujours rien après trois tentatives de reconnexion est laissé tel quel et signalé (`auto-reconnect GAVE UP`). `pool-stream --start` effectue une nouvelle tentative.
- **`pool-list` indique où il en est.** Tant que le capteur reste silencieux, sa ligne `STREAM FAULT` indique l’état d’avancement de la reconnexion. La ligne disparaît avec le premier spectre, puis le capteur affiche `reconnected automatically N time(s)`.
- **L&#x27;enregistrement se poursuit dans le même `.daq`.** Le fichier marque chaque interruption par deux lignes : `event_type` 5 au niveau du dernier spectre avant l&#x27;interruption et 6 au niveau du premier après celle-ci, chacune comportant uniquement un horodatage. `pool-record --duration` affiche `!! STREAM LOST …` pendant l’absence du capteur et `stream resumed after …` à son retour. `pool-record --stop` répertorie chaque interruption et émet un avertissement, comme il le fait pour un fichier vide (un enregistrement ne contenant aucune mesure quitte `1`).
- **Un enregistrement conserve un seul étalonnage.** Si le capteur revient avec un étalonnage différent alors qu’un enregistrement est en cours, le reste de l’enregistrement conserve celui avec lequel il a commencé, et l’avertissement d’arrêt l’indique. `pool-disconnect` + `pool-connect` adopte la nouvelle.
- **Vos commandes prévalent.** Un capteur que vous avez désactivé avec `pool-disconnect`, ou mis en pause avec `pool-stream --stop`, n’est jamais reconnecté, même si une reconnexion est déjà en cours.
- **`pool-info` sur un capteur silencieux** lit le capteur lors d’une nouvelle connexion et ajoute `!! stream NOT reaching this computer`. Un capteur qui ne répond pas est immédiatement signalé. `pool-list` n’affiche plus d’état PTP pour un capteur qui a cessé de répondre.
- Les DAQ-U et DAQ-M ne sont pas pris en charge. Un DAQ-M se reconnecte automatiquement via Bluetooth, et un DAQ-U débranché de l’USB nécessite à nouveau `pool-connect`.

### DAQ-E-S — diffuseur fixe et capteur d’inclinaison

Le **DAQ-E-S** est un DAQ-E présentant deux différences importantes pour le logiciel. Tout le reste — détection, connexion, PTP, enregistrement, paramètres de l’appareil ci-dessous — fonctionne exactement comme pour un DAQ-E.**Son diffuseur est fixe, il ne nécessite donc aucune correction de cap.** Le diffuseur de type « Sunshine » est intégré et était déjà présent sur l’appareil lors de la mesure de son étalonnage en usine ; cet étalonnage indique donc déjà l’irradiance descendante réelle. Chloros
définit son cap sur `as_recorded` (sans profil) quelle que soit la demande : un cap nommé spécialement pour lui (`sunshine_cosine`, `fov_*`, `none`) est ignoré et génère un message d’erreur, et ses enregistrements stockent `cap_id_source=model`, de sorte que l’avertissement de limite présumée ne se déclenche jamais pour celui-ci. N’appliquez pas non plus manuellement de limite à ses données : une limite `sunshine_cosine` au maximum donnerait une valeur environ 11 fois trop élevée.

**Il est équipé d’un accéléromètre à 3 axes.** La réponse cosinus du diffuseur de lumière solaire n’est valable que lorsqu’il est à l’horizontale ; ainsi, un capteur incliné renvoie un rayonnement solaire erroné — et la réflectance hérite de cette erreur — sans autre symptôme. Chaque lecture du DAQ-E-S comporte donc son orientation :

| Champ | Signification |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Angles en degrés. `None` lorsque le micrologiciel marque l’angle comme indéfini (chute libre, un axe mort). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Accélération en milli-g. Une valeur `mag_mg` très éloignée de ~1 000 indique que l’appareil était en mouvement et que les angles ne sont pas significatifs pour cet échantillon. |
| `cal_applied` | Activé : les axes et les angles sont dans le référentiel du diffuseur (l&#x27;étalonnage de montage de l&#x27;appareil est appliqué). Désactivé : référentiel brut de la carte, pouvant afficher jusqu&#x27;à environ 2,3° d&#x27;inclinaison qui n&#x27;est pas réelle. |
| `sample_age_ms` | Âge de l&#x27;échantillon de l&#x27;accéléromètre au moment de l&#x27;envoi du spectre. |
| `temp_raw` | **Tendance** non calibrée de la température de la puce — différences uniquement, jamais une lecture du thermomètre. |

`daq pool-latest --json` les renvoie sous la clé `imu` de chaque trame (`null` sur un DAQ-U, DAQ-M ou sur un DAQ-E standard). Un fichier `.daq` enregistré à partir d’un DAQ-E-S stocke les mêmes valeurs dans ses colonnes `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) à côté de chaque spectre, ce qui permet au lecteur de distinguer un nuage d’un trépied renversé. Sur tous les autres modèles, ces colonnes sont vides.

Le flux d’attitude est rapporté séparément : `daq pool-list` affiche `!! ATTITUDE STREAM FAULT (spectra unaffected)` lorsqu’il est hors service, ainsi que le nombre de défaillances dont il s’est remis par la suite. Une défaillance à cet endroit n’entraîne jamais de lacune dans l’enregistrement spectral. `daq pool-discover` répertorie l’appareil sous le nom `DAQ-E-S` (ou `DAQ-E` s’il ne répond pas au contrôle d’identité dans un délai d’environ 2 s).

### Paramètres de l’appareil DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` sans `--gateway` n&#x27;enregistre aucune passerelle** (toute passerelle définie auparavant est supprimée), ce qui convient à un réseau de caméras sans routeur.
- **Une adresse statique doit exister sur le réseau de cet ordinateur.** Chloros
refuse toute adresse dont le sous-réseau n’inclut pas l’adresse de cet ordinateur sur le réseau du capteur : après le redémarrage, rien ne pourrait atteindre le capteur pour rétablir la configuration initiale. N’utilisez `--force` que si cet ordinateur doit être réadressé ou acheminé vers le nouveau sous-réseau.
- **Un redémarrage est refusé pendant que le capteur enregistre.** Arrêtez d’abord l’enregistrement (`pool-record --stop`). Un enregistrement lancé juste avant un redémarrage est interrompu par celui-ci, et les codes `pool-set-network`, `pool-reboot` et `pool-update-firmware` se terminent par une ligne `Warning:` indiquant le nom du fichier d’enregistrement (même si le capteur ne revient pas par la suite).
- **Après un redémarrage, Chloros
ne reconnecte que le capteur lui-même.** Il attend à l&#x27;endroit où se trouvait le capteur — et après `--auto` également sous son nom `daq-e-<id>.local` — une réponse contenant le nom du capteur, son nom d’hôte et son modèle, à partir de l’adresse MAC de la carte réseau qu’il possédait, et le reconnecte à l’adresse qui a répondu. Une réponse sous ce nom n’est acceptée qu’avec cette adresse MAC ; ainsi, depuis un ordinateur qui accède au capteur via un routeur, ou depuis WSL2 dans son réseau NAT par défaut, un capteur que le DHCP a déplacé vers une nouvelle adresse n’est pas reconnecté : la commande l’indique et affiche la commande `pool-connect --eth-host <its address> …` qui le ramène avec les réglages de cap et d’exposition qu’il avait.
- **`pool-update-firmware` refuse un capteur dont il ne peut pas lire la version du micrologiciel.** Un capteur qui met plus de temps que le délai d’attente pour répondre après le flash n’est pas défectueux : dès qu’il répond, il est reconnecté avec les paramètres du nouveau micrologiciel renseignés, et la commande signale qu’il a été lent.
- **Une mise à jour du micrologiciel nécessite une connexion TCP entrante.** Le capteur se reconnecte à cet ordinateur pour récupérer l’image. Chloros
écoute sur la carte réseau qui achemine vers le capteur (sur toutes les cartes uniquement lorsqu’il ne peut pas déterminer laquelle) et ne sert que l’adresse propre au capteur ; toute autre connexion est fermée sans être traitée. Sous Windows
, la règle de pare-feu `Chloros Backend (TCP-In)` de l’installateur autorise l’accès (voir [Windows
(.exe)](#windows-exe)) ; un pare-feu qui la bloque empêche la mise à jour avec le code d&#x27;erreur `… device never connected back on TCP <port> …`.
- **Sous WSL2, utilisez la mise en réseau en miroir** (`networkingMode=mirrored` dans `%UserProfile%\.wslconfig`). En mode NAT par défaut, l’adresse de cet ordinateur sur le réseau du capteur est celle de WSL lui-même ; par conséquent, toute adresse statique est refusée (à l’exception de `--force`), le flux en direct n’atteint pas WSL et la mise à jour du micrologiciel ne peut pas aboutir, car le capteur se reconnecte à l’ordinateur pour recevoir l’image.
- **L’identifiant du capteur et le nom d’hôte ne changent pas.** Quelle que soit son adresse, un DAQ-E reste `daq-e-<id>` et `daq-e-<id>.local`, de sorte que `--eth-host daq-e-<id>.local` continue de le détecter.
- **L&#x27;interface graphique conserve les mêmes paramètres** : onglet « Capteurs de lumière », paramètres du capteur (roue dentée), « Réseau ». L&#x27;interface graphique maintient sa propre connexion avec un capteur ; par conséquent, si vous modifiez les paramètres d&#x27;un capteur avec lequel l&#x27;interface graphique est connectée depuis l&#x27;interface graphique : un redémarrage depuis la CLI
met fin à la session de l&#x27;interface graphique avec celui-ci.
- Les équivalents SDK
sont `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` et `update_firmware()`.

---

## `chloros-cli project`

Ouvrir, se connecter à et piloter un projet Chloros
enregistré (un dossier contenant `cameras.json` + `sensors.json` + `project.json`). Tout passe par le backend, de sorte que l’interface graphique et la CLI
produisent un état matériel identique.

### Référence des sous-commandes

| Sous-commande | Objectif |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Affiche le manifeste des périphériques du projet (caméras, réseaux de caméras, capteurs), le lit à partir de ses fichiers et vérifie ses fichiers source (voir [Un projet déplacé](#a-moved-project---relink)). Elle n’ouvre pas le projet dans le backend : le projet qui y est ouvert, ainsi que ses périphériques et ses enregistrements, restent inchangés. Quitte `1` si aucun des fichiers source du projet ne peut être trouvé. |
| `project devices PATH [--reconnect]` | Liste ou relance de la découverte. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Connexion de chaque caméra / réseau de caméras / capteur enregistré. Chaque réseau de caméras se connecte comme le fait `lattice array-connect` : jusqu’à 600 s, ses étapes étant affichées sur stderr (`[array <id>]    <s> s: <step>` ; stdout correspond au rapport JSON
), et un réseau non connecté dans ce délai est annulé — ses caméras sont remises dans leur état initial — et signalé sous `errors`, sans jamais rester connecté. `--packed` définit le transport 12 bits compressé de chaque réseau (par défaut `auto` ; un projet n’enregistre aucun paramètre par réseau ; voir [Transport 12 bits compressé](#packed-12-bit-transport---packed)). Chaque capteur de lumière est ouvert dans le pool DAQ du backend à son adresse enregistrée, avec son temps d’intégration, sa moyenne par trame, son exposition automatique et sa limite enregistrés : un capteur pour lequel aucune limite n’a été choisie se voit attribuer `sunshine_cosine`, un capteur enregistré « nu » (`none`) reste « nu », et un DAQ-E-S ne prend jamais de limite. Le panneau DAQ de l’interface graphique ne fait pas partie du pool : fermez d’abord un capteur à cet endroit. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Capture unique à partir d’une caméra ou d’un réseau de caméras nommé (`--prefix` par défaut, `capture`). `--orient` (matrices) : la matrice enregistre les données brutes, qui restent au format natif — cela indique que les produits créés par `process` à partir de celles-ci doivent être orientés, comme `array-capture --orient`. Par défaut : natif. `project burst` / `stream` et les éléments `capture` / `burst` / `stream` (`orient: true`) d’une recette utilisent la même option. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Salve de N images provenant d’une caméra ou d’un réseau nommé (`-n/--count` par défaut 5 ; `-i/--interval` en secondes entre les images, par défaut 0). Les rafales de réseaux de caméras suppriment les doublons des groupes synchronisés répétés (contrôle de périméité) afin qu’un réseau à cycle partiel ne puisse pas renvoyer N copies d’une même image ; affiche les résultats à chaque itération. Une capture qui échoue en cours d’exécution interrompt la rafale et quitte `1`, en indiquant le nombre de trames enregistrées auparavant. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Capture sur disque en boucle : `--fps F` capture une seconde (3 par défaut), une image par capture à partir d’une caméra, un groupe d’images synchronisées à partir d’un tableau (avec le même mécanisme de surveillance de l’obsolescence que `project burst`), jusqu’à ce que `-n N` capture (par défaut `0` = jusqu’à ce que l’on appuie sur Ctrl+C, ce qui affiche le nombre d’images enregistrées et quitte `0`, ou `1` lorsqu’aucune image n’a été enregistrée ou qu’un fichier a échoué ; un Ctrl+C avant que `-n N` ne capture les données quitte `130`). Une capture qui échoue en cours d&#x27;exécution interrompt le flux et quitte `1`, en indiquant le nombre d&#x27;images enregistrées avant l&#x27;échec. `--poll-interval` est obsolète et ignoré : il est toutefois toujours accepté afin que les anciens scripts puissent s&#x27;exécuter, avec une note sur stderr. |
| `project sensor read PATH NAME [--json]` | Dernière trame du spectre. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Enregistrement d&#x27;un fichier .daq pour `--seconds` (10 par défaut). Un enregistrement ne contenant aucune mesure est signalé sur stderr et provoque la sortie de `1`. |
| `project run PATH RECIPE.yaml` | Exécute une recette de capture YAML/JSON
. `--dry-run` effectue une validation sans exécution. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcule l’alignement d’un tableau — voir [le tableau des options ci-dessous](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Afficher le profil d’alignement actuel. |
| `project align clear PATH NAME` | Supprimer le profil mis en cache. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Décaler la transformation d’un esclave (par défaut `0`, `0`, `0`, `1.0` : aucun changement). |
| `project align export PATH NAME --to FILE` | Enregistrer le profil au format JSON
. |
| `project align import PATH NAME --from FILE [--no-validate]` | Charger un profil enregistré. |

#### Options `project align calibrate`

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Méthode d&#x27;alignement. **Ces orthographes diffèrent de celles de `lattice align-calibrate`**, qui accepte les formes abrégées `orb` / `akaze` / `phase` ; ces deux commandes ne sont pas interchangeables pour cet indicateur. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformer le modèle pour l&#x27;ajuster. |
| `--frames N` | `1` | Instaurer une moyenne des instantanés d&#x27;images synchronisés moyenne. |
| `--reference SN` | la caméra maître | Numéro de série de la caméra de référence ; tous les autres membres sont déformés pour s’aligner sur celle-ci. |
| `--max-features N` | `5000` | Limite du nombre de caractéristiques ORB. |
| `--ratio-threshold F` | `0.75` | Test du. |
| `--ransac-threshold-px F` | `3.0` | Seuil des points intérieurs de RANSAC. |
| `--min-matches N` | `15` | **Seuil de qualité** — rejeter la solution si le nombre de correspondances valides est inférieur à cette valeur. |
| `--max-reproj-err-px F` | `4.0` | **Seuil de qualité** — rejeter la solution si l&#x27;erreur de reprojection RMS dépasse cette valeur. |
| `--checkerboard RxC` | — | Géométrie de la carte pour `--method checkerboard`, par exemple `9x6`. |
| `--name PROFILE` | vide | Nom du profil intégré dans le fichier JSON
enregistré. **Il ne s&#x27;agit pas du nom du tableau** — celui-ci correspond à la position `NAME`. |

Ces deux contrôles de qualité expliquent pourquoi un calibrage peut réussir la résolution tout en
refusant d’être enregistré : un profil qui échouerait à l’un ou l’autre de ces contrôles entraînerait silencieusement un mauvais alignement de chaque
capture ultérieure ; il est donc refusé plutôt que conservé.

### Un projet déplacé (`--relink`)

Le fichier `project.json` d’un projet nomme chaque fichier source — chaque image et chaque enregistrement du capteur de lumière
— par son chemin d’accès complet. Si vous déplacez le dossier du projet, ou l’ensemble du dossier « projects »,
vers un autre disque, ces chemins d’accès continuent de pointer vers l’ancien emplacement.

L’ouverture d’un projet corrige ce qu’il peut de lui-même : un fichier qui se trouvait dans le
dossier du projet est recherché dans le dossier où se trouve désormais le projet, et chaque chemin d’accès qui y est trouvé
est réorienté et enregistré (`project.json` est d’abord sauvegardé sous le nom
`project.json.before-relink-<date>-<time>`). Ce processus se produit à chaque ouverture — celle de l’application,
`chloros-cli project open`, `chloros-cli process` dans un projet existant, ceux du SDK

et ceux de `chloros-mcp`. Un chemin n’est redirigé que vers un fichier qui s’y trouve.

Les fichiers qui se trouvaient en dehors du dossier du projet — une session Hub, une importation via « Ajouter un dossier » —
ne peuvent pas être trouvés de cette manière. Indiquez le nom du dossier dans lequel ils se trouvent actuellement :

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` prend en compte le dossier dans lequel les fichiers ont été placés, ou n’importe quel dossier situé au-dessus : chaque dossier de fichier manquant
est recherché sous celui-ci en fonction des noms de dossiers qui le terminent
(`Hub Projects/<session>/raw/<serial>`, puis `<session>/raw/<serial>`, …), et les fichiers de chaque
dossier sont extraits de l’emplacement qui en contient le plus. `project open` affiche
ce qui a été réorienté, le nombre de fichiers source trouvés et les dossiers dans lesquels se trouvaient les autres
. Il en manque encore : un avertissement, sortie `0` — le projet s’ouvre avec le reste. Aucun
trouvé : une erreur indiquant les noms de ces dossiers et `--relink`, sortie `1` ; le backend
refuse d’ouvrir un tel projet pour `process` ou `project devices --reconnect` (409 HTTP
`PROJECT_SOURCES_MISSING`).

### Exemples

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Langage DSL de recette

`project run RECIPE.yaml` accepte un fichier YAML ou JSON décrivant une séquence d&#x27;actions :

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Actions prises en charge : `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` récupère les clés `cameras.json`, `settings`, `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (les anciens noms `exposure_time`, `exposure_auto`, `gain`, `gain_auto` sont acceptés comme alias). Une exposition, un gain ou une valeur cible définis de cette manière sont considérés comme les vôtres : le contrôle automatique de l’exposition d’Chloros (y compris l’exposition mesurée par le DAQ) ne les modifie pas. Une exposition manuelle laisse le gain tel quel : en mode automatique, l’appareil photo continue d’ajuster le gain (sur toute sa plage) en fonction de la luminosité cible jusqu’à ce que vous définissiez `gain_db` ou `auto_gain: "Off"`. Une commande `auto_exposure: "Continuous"` ultérieure rétablit l’exposition automatique, et `target_brightness_auto: true` rétablit la cible de Chloros après avoir maintenu une cible fixe. L’action `burst` prend en paramètre `name` (obligatoire), `count` (valeur par défaut : 5), `interval` (en secondes, valeur par défaut 0), `output`, `format`, et `settings` (même configuration par caméra que `apply`) ; les rafales en réseau utilisent le même chien de garde de groupe fraîchement synchronisé que `project burst`.

Un `capture`, `burst` ou `stream`, la valeur par défaut de `output` est `output`. Un chemin relatif `output` — la valeur par défaut incluse — est résolue par rapport au dossier à partir duquel vous exécutez `chloros-cli project run`, et non par rapport au dossier du fichier de recette ou au dossier du projet (dans la version 1.2.0, il se trouvait dans le dossier d’installation du backend).

Exécutez-le :

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Chaque exécution est d’abord vérifiée par rapport au projet.**Chaque action doit avoir comme clé exactement l’un des verbes ci-dessus — `- capture: {name: FrontLeft}`, et non `- action: capture` ou `- type: capture` — avec un `name`, le projet dispose de `cameras.json` / `sensors.json`, d’une caméra ou d’un tableau pour `apply` / `capture` / `burst` / `stream` et un capteur de lumière pour `sensor` ; un `stream` nécessite que `count` soit &gt; 0 et un `wait` une durée. Une recette présentant l’une de ces erreurs n’exécute**aucune** action (`[recipe] error: …`, `[recipe] not run: …`, sortie `1`), même avec `stop_on_error: false`. Les touches qu’une étape n’utilise pas sont un `[recipe] warning:`, tout comme chaque matrice ou capteur de lumière piloté par la recette qui n’est pas connecté pour le moment. Une recette ne connecte jamais de périphériques : exécutez d’abord `chloros-cli project connect PATH`. `--dry-run` affiche la recette, puis les mêmes erreurs et avertissements, et `[recipe] valid: …` / `NOT valid: …`, n’exécute aucune commande et quitte `1` lorsque la recette contient une erreur.

---

## Variables d’environnement

| Variable | Effet |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Remplace l&#x27;URL du backend (par défaut `http://127.0.0.1:5000`) — **prise en compte uniquement par les familles de commandes `lattice`, `project` et `daq pool-*`** (`lattice network-analysis` utilise plutôt `--backend-url`). Les commandes principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixent la valeur de `http://127.0.0.1:<port>` et ignorent cette variable (la valeur IPv4 littérale contourne la pénalité d&#x27;Windows `localhost`→`::1` d&#x27;environ 2 s par requête), de sorte qu&#x27;elles ciblent toujours la machine locale. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | À définir dans l’environnement du backend. `1` atténue le refus de sursouscription — une connexion de type « array » avec `pin_resolution`, ou une [commande multi-caméras transitoire](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — en un avertissement fort, acceptant la perte de paquets GVSP. À des fins de test uniquement — voir [Modèle de fps et de rafales du tableau](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | À définir dans l’environnement du backend : `on` ou `off` définit par défaut le [transport 12 bits compressé](#packed-12-bit-transport---packed) pour chaque tableau (valeur par défaut : `auto`). Une valeur explicite `--packed` prévaut toujours. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (ou `true` / `yes`) désactive la [vérification automatique des mises à jour](#chloros-cli-update) de l&#x27;CLI Linux de Linux au démarrage. |
| `CHLOROS_CLI_MODE` | Définie par l’CLI-même ; indique au backend d’activer le traitement parallèle. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ignore la sonde de repli GVSP (résultats ICMP uniquement). **Cela désactive les paquets jumbo, cela ne se contente pas de réduire le volume des messages du journal** — la caméra ne répond aux pings DF que jusqu’à 1 500 sur chaque chemin, ce test est donc le seul moyen de détecter les paquets jumbo. Gain d’environ 1 s par caméra et par connexion ; coût d’environ 1,45 fois la bande passante maximale si le réseau *pouvait* prendre en charge les paquets jumbo. Le SDK affiche un avertissement lorsque vous activez cette option. |
| `CHLOROS_GVSP_SESSION_REUSE` | À définir dans l’environnement du backend : `0` mesure la taille des paquets à chaque connexion au lieu d’une seule fois par caméra et par session de backend. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixe la taille des paquets GVSP à N octets ; ignore complètement la détection. Préférez la configuration par commande (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) à un réglage permanent : une taille fixe empêche l’adaptation au réseau en amont, et fixer la valeur à 9 000 sur un chemin incapable de transporter des paquets jumbo provoque un **timeout**à**chaque** capture avec `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | À définir dans l’environnement du backend : le nombre maximal de secondes pour une [commande multi-caméras transitoire](#multi-camera-transient-sessions), ou un `lattice capture` d’une caméra qui ne conserve rien, passe à régler l’exposition de chaque caméra à partir de la scène avant de conserver une image (par défaut `5`). `0` désactive cette fonctionnalité : l&#x27;exposition automatique propre à chaque caméra s&#x27;applique alors dès la première image, en partant de sa limite d&#x27;exposition. |
| `CHLOROS_LOG_DIR` | À définir dans l’environnement du backend : le dossier dans lequel est enregistré son journal détaillé. Valeur par défaut `%LOCALAPPDATA%\Chloros\logs` sous Windows, `~/.cache/chloros/logs` sous Linux (`$XDG_CACHE_HOME/chloros/logs` lorsque celui-ci est défini). Le service racine du fichier .deb définit `/var/log/chloros`, accessible en lecture par le groupe `adm` — voir [Exécuter le backend en tant que service](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Emplacement des fichiers temporaires. L&#x27;interface CLI utilise automatiquement `/mnt/ssd/tmp` s&#x27;il est présent. |

---

## Codes de sortie

| Code | Signification |
| --- | --- |
| `0` | Succès. |
| `1` | Échec générique (la plupart des erreurs de sous-commandes). Une commande de capture quitte également `1` lorsque l&#x27;écriture d&#x27;un fichier demandé a échoué, même si les autres ont été enregistrés, et `array-burst` / `array-record` et `1` se terminent lorsque l’enregistreur signale une erreur — voir [Fichiers dont l’écriture a échoué](#files-that-fail-to-write). `process` quitte également `1` lorsque les exportations d’une exécution n’ont pas pu être vérifiées, ou lorsqu’une image a été perdue, bien que les autres produits aient été enregistrés. |
| `2` | Erreur d’argument (indicateur inconnu ou manquant ; voir également `time-sync set-priority` sans aucune des deux priorités). |
| `130` | Interrompu par Ctrl+C : toute commande qu&#x27;il interrompt — `lattice`, `project`, `daq` et `process` (`process` une fois que le backend a arrêté l&#x27;exécution, voir [Remarques sur `process`](#notes)). Une commande dont l’arrêt normal se fait par Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` et `project stream` avec `-n 0` — se termine en fonction de ce qu’elle a effectué : `0` une fois l&#x27;écriture terminée, `1` en cas d&#x27;échec. |
| `143` / `129` | Interrompu par SIGTERM / SIGHUP (128 + le signal), après l&#x27;exécution de la procédure de nettoyage de la commande : un travail en cours s&#x27;est arrêté et ses caméras ont été remises en place, un traitement en arrière-plan s&#x27;est arrêté. Un `lattice continuous` ou tout autre enregistrement dont l&#x27;arrêt normal se fait par Ctrl+C quitte également `143`, et non `0`. |

`lattice network-analysis` quitte `1` pour les verdicts `needs_force_slip` et `error`, et `selftest` quitte `1` à moins que les sept vérifications ne soient réussies.

---

## Conseils de dépannage

- **« Connexion requise »** → Exécutez `chloros-cli login EMAIL PASSWORD` une fois sur cet ordinateur.
- **« Le dossier du projet est trop profond pour cet ordinateur »** → Les chemins d’accès longs de Windows sont désactivés sur cette machine (paramètre par défaut de Windows) ; ainsi, aucun chemin ne doit dépasser 260 caractères, et les produits se trouvent jusqu’à environ 130 caractères en dessous du dossier du projet. Aucun traitement n’a été effectué. Le message indique le dossier de projet le plus long compatible : utilisez-en un plus court (`-o` ou `chloros-cli set-project-folder`), ou activez les chemins d’accès longs de Windows et redémarrez Chloros. Le programme d&#x27;installation Windows propose cette option sous le nom « Activer les chemins d&#x27;accès longs Windows », cochée par défaut ; sinon, utilisez la stratégie de groupe : *Configuration de l&#x27;ordinateur &gt; Modèles d&#x27;administration &gt; Système &gt; Système de fichiers &gt; Activer les chemins d&#x27;accès longs Win32*.
- **« backend unreachable »** → Lancez l&#x27;application de bureau Chloros (Windows), ou démarrez le backend vous-même : `"C:\Program Files\Chloros\backend\chloros-backend.exe"` sous Windows (dans le dossier `backend` de votre dossier d&#x27;installation ; il ne se trouve pas dans `PATH`), `chloros-backend` sous Linux. Ou vérifiez `CHLOROS_BACKEND_URL` en cas d’accès à distance.
- **Les commandes `lattice` échouent avec le message « Pilotes de caméra LATTICE introuvables »** → Le backend n’a pas pu charger le runtime de caméra LATTICE avec lequel il est installé. Toutes les commandes `lattice` s’exécutent sur le backend ; l’CLI-même ne contient aucun pilote de caméra. Réinstallez Chloros : le programme d’installation Windows et le fichier .deb installent le runtime à côté du backend.
- **« Array connect » / « Array Settings » affiche « FRAMES WILL DROP » ou « Reduce ROI to enable »** → La taille de l’anneau de réception de la carte réseau de l’hôte est trop petite (elle est généralement réinitialisée à 32 après une mise à jour du pilote de la carte réseau). Voir [Configuration et réglage de la carte réseau hôte](#host-nic-setup--tuning-lattice-arrays) — configurez `ReceiveBufferLen=256`, `PendingReceives=64`.
- **La machine se bloque au redémarrage/à l’arrêt, puis WMI `Invalid class` / la carte réseau ne s’active pas / les périphériques USB sont manquants** → Pilote obsolète de l’adaptateur USB 10 GbE provoquant `DRIVER_POWER_STATE_FAILURE` (écran bleu `0x9F`). Mettez à jour le pilote de la carte — voir [Configuration et optimisation de la carte réseau hôte](#host-nic-setup--tuning-lattice-arrays).
- **Sous Linux : les captures appartiennent à l’utilisateur root, vous ne pouvez donc pas les supprimer, et le journal du backend ne se trouve pas dans votre `~/.cache`** → Le backend s’exécute en tant que root via `chloros-backend.service`. Passez à `chloros-backend@<user>.service`, qui l’exécute en votre nom — voir [Exécution du backend en tant que service](#running-the-backend-as-a-service-headless). Jusqu’à ce moment-là, le journal du service root se trouve dans `/var/log/chloros` (accessible au groupe `adm`).
- **Sous Linux : une unité de backend est `failed` avec le message « Une autre instance est déjà en cours d’exécution » (code de sortie 98)** → Un autre backend occupe déjà le port : un seul peut s’exécuter à la fois. Si deux unités de backend sont activées, conservez-en une : `sudo systemctl disable --now <unit>` pour celle que vous n’utilisez pas — voir [Exécuter le backend en tant que service](#running-the-backend-as-a-service-headless). Si seule cette unité est activée, un backend démarré en dehors de systemd (par `chloros-backend`, `chloros-cli` ou un script SDK) le détient : arrêtez ce backend, puis exécutez `sudo systemctl start <unit>`.
- **Avertissement concernant la mémoire swap sur Jetson** → Ajoutez une mémoire swap sur fichier ; l’CLI affiche les commandes exactes `fallocate` / `swapon`.
- **« backend not reachable » (backend inaccessible) provenant d’une commande `lattice`, `project` ou `daq`** → aucun backend n’est en cours d’exécution : ces commandes n’en lancent jamais. Exécutez la commande `chloros-cli backend status`, puis consultez la section [Démarrer le backend](#start-the-backend).
- **« `daq net ...` n&#x27;est pas disponible dans Chloros »** (Linux et Jetson) → Certains paramètres propres au DAQ-E sont réservés au service MAPIR. Les paramètres réseau, les redémarrages et les mises à jour du micrologiciel sont gérés par les commandes `pool-set-network`, `pool-reboot` et `pool-update-firmware` ; contactez le support MAPIR pour le reste.
- **« Les commandes matérielles directes du DAQ nécessitent le package daq complet »** (Windows) → Utilisez `pool-*`, qui pilote le même capteur via le backend, ou `chloros_sdk.connect_daq_sensor()` depuis Python.

---

## Voir aussi

- [Référence du SDK Python](sdk-reference.md) — équivalent programmatique de chaque commande CLI.
- [Capteurs de lumière DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) dans le manuel Chloros — les capteurs, leurs capacités et leur plage d&#x27;étalonnage, l&#x27;enregistrement, la mise en réseau DAQ-E.
- [Caméras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) dans le manuel Chloros — connexion, paramètres des caméras, modes de capture, matrices.
- Documentation en ligne : `https://mapir.gitbook.io/chloros/cli`
