# Référence du SDK Chloros Python

**Version :**

1.3.0**Générée le :**29/07/2026 à 19 h 19 ·**Révisée le :** 06/11/2026**Package :** `chloros-sdk` (PyPI)**Public visé :** Optimisé pour une utilisation avec les grands modèles de langage (LLM) ; lisible par l’homme.**Portée :** Toutes les classes, fonctions et aides publiques exposées par `import chloros_sdk`, avec des exemples copiables-collables couvrant le traitement d’images, le contrôle d’une seule caméra, les tableaux synchronisés, les capteurs DAQ et l’automatisation de projets, ainsi que le serveur `chloros-mcp` pour les assistants IA.

Si vous souhaitez uniquement consulter les points clés, rendez-vous à :
- [Installation et démarrage rapide](#installation)
- [Smart-Connect pour les réseaux de caméras LATTICE](#smart-connect-for-lattice-cameras)
- [Sessions de capteurs DAQ](#daq-sensor-sessions)
- [Utilitaires de caméra](#camera-utilities)
- [Matrices transitoires, profils d’alignement et indices](#transient-arrays-alignment-profiles--indices)
- [Dossier de projet et recettes](#project-folder--recipes)
- [Automatisation des projets](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Images en direct dans votre propre pipeline](#live-frames-in-your-own-pipeline)
- [Serveur MCP (assistants IA)](#mcp-server-ai-assistants)

> **Modifications apportées dans la version 1.3.0** (ce que détecte un script 1.2.0 existant).
>
> **Modifications qu'un script 1.2.0 devra peut-être prendre en compte :**
>
> - **Une version que le backend ne confirme pas génère désormais une exception.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` sur un tableau ouvert par la session, `DAQSensorSession.disconnect()` / `force_disconnect()` sur un capteur qu’elle a ouvert, et les erreurs `ChlorosProject.disconnect_all()` déclenchent l’erreur `ChlorosConnectError` lorsque le backend refuse la libération ou ne la confirme pas à temps, car le périphérique peut encore être occupé. Dans la version 1.2.0, ils se terminaient sans message. La sortie d’un bloc `with` ne déclenche toujours pas d’erreur.
> - **Un périphérique emprunté n’est jamais libéré.** `with attach_array(...)` et un `disconnect()` explicite sur celui-ci laissent le tableau en cours d’exécution (dans la version 1.2.0, quitter le bloc le libérait, même lorsque l’interface graphique en était propriétaire); appelez `force_disconnect()` lorsque vous souhaitez y mettre fin. De même, un `connect_daq_sensor()` qui a trouvé le capteur déjà présent dans le pool, y compris la détection intelligente avec indicateur à zéro, le laisse ouvert sur `disconnect()` et à la sortie de son bloc `with`.
> - **Les formats PNG et JPEG n'acceptent que les produits `display` et indexés.** `capture()` avec `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) et un niveau `radiance` ou `reflectance`, ou `processing="all"`, déclenche `ValueError` avant toute requête ; un `raw` / `debayered` de 12 bits est refusé par le backend. Voir [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` ouvre ce capteur ou déclenche** `ChlorosConnectError` en répertoriant ce qu’il a trouvé ; il ne se rabat jamais sur un autre capteur. Voir [Transport épinglé](#pinned-transport).
> - **`frame_stream()` déclenche une exception au lieu de se mettre en silence :** `FrameFetchError`, en cas de refus, répéterait lors du tick suivant (une lecture de réflectance sans aucune acquisition de données incluse), après 60 s sans trame et après 60 s sans nouveau déclencheur. Voir [Un déclencheur par lecture](#one-trigger-per-read--group_id).
> - **Un délai d’attente de lecture ou une connexion interrompue déclenche `ChlorosConnectError`** à chaque appel smart-connect et `analyze_array_network()`, là où la version 1.2.0 déclenchait le délai d’attente sous-jacent ou l’erreur de connexion.
> - **`connect_array()` attend 600 s par défaut (`timeout`, 180 s dans la version 1.2.0), et une connexion qu’il abandonne est annulée.** L’appel envoie son `timeout` au backend, qui remet chaque caméra dans l’état où elle a été détectée et n’établit aucune connexion lorsque celle-ci n’est pas établie à temps ; l’appel génère une erreur `ChlorosConnectError` (504 HTTP `CLIENT_DEADLINE_PASSED`). Dans la version 1.2.0, une connexion établie juste après la mise sous tension des caméras pouvait se prolonger au-delà de l’appel et connecter le tableau après que celui-ci ait déclenché l’exception. `connect_all()` attend chaque tableau enregistré de la même manière (120 s dans la version 1.2.0). Voir [Signature `connect_array()`](#connect_array-signature).
> - **`connect_array()` conserve l’ensemble des capteurs.** En l’absence de `width` / `height` / `binning`, chaque caméra est configurée sur le capteur complet et la fréquence d’images chute au niveau fourni par l’hôte et le réseau ; la version 1.2.0 a réduit ou regroupé les images pour maintenir la fréquence. Voir [Matrice synchronisée](#synchronized-array--arraysession-smart-prep).
> - **Le format de pixels Bayer sur une caméra M3M (mono) correspond à sa profondeur.** Une caméra M3M ne propose que des formats mono, et les préréglages désignent les formats Bayer : dans la version 1.2.0, la caméra refusait ce format et conservait le sien, tandis que `settings.pixel_format` désignait le format Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, un `CameraSettings(pixel_format=...)`, un `connect_camera(preset=...)` et un tableauformat partagé définissent désormais le format Mono de même profondeur sur un M3M : `"BayerRG8"` et `high_speed` donnent `Mono8` ; `"BayerRG12"`, `high_quality` et `triggered` donnent `Mono12` ; `cam.settings.pixel_format` lui attribue un nom. Il en va de même dans l’autre sens : un M3C n’est jamais enregistré au format Mono, donc `"Mono12"` — à partir de n’importe lequel de ces éléments, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` ou le format enregistré d’un réseau — donne `BayerRG12` sur un M3C, quelle que soit la caméra qui sert de maître au réseau. Un `CameraSettings` que vous transmettez conserve son nom ; ainsi, le même objet continue de demander à chaque caméra cette profondeur au sein de sa propre famille. `connect_camera(preset=...)` conserve le format du préréglage (la version 1.2.0 règle la caméra sur 12 bits après l’application du préréglage), et une caméra ouverte sur un préréglage n’écrit plus les données qu’elle a capturées dans `PRESETS`. Tout autre format refusé par une caméra (un format absent de sa liste `PixelFormat`, tel que `RGB8` sur un M3M ou `BayerRG16` sur un micrologiciel qui ne le prend pas en charge) ne reçoit pas non plus de nom distinct : après la connexion d’un `LatticeCamera`, `apply_settings()` ou `set_format()`, `cam.settings.pixel_format` et `packed_transport` indiquent le format de flux de la caméra, tandis qu’une ligne imprimée précise le format qu’elle a refusé.
> - **`analyze_array_network()` indique à quelle connexion cela s’applique :** `pixel_format=None` (anciennement `"BayerRG8"`) désigne les formats définis par la connexion (Mono12 / BayerRG12), et `packed="auto"` indique le mode de transport compressé ; ainsi, le nombre d’images par seconde (fps) indiqué peut différer de 1.2.0'. Un `pixel_format` que vous transmettez projette un M3M à cette profondeur. Voir [Aide à l’analyse réseau](#network-analysis-helper).
> - **`CameraSession.capture()` renvoie un `CaptureResult`** (toujours une liste de dictionnaires de trames) contenant `.skipped`, `.failed`, `.daq_files` et `.daq_matches`. Un résultat `ArraySession.capture()` contient également `.alignment` (indiquant si la déformation d’alignement a été appliquée, et pourquoi pas) et `.oriented`.
> - **Les captures de tableaux sont alignées par défaut, tout comme celles de l’application.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` et `ArrayHandle.capture()` / `capture_smart()` déforment les fichiers `debayered`, `display`, `radiance`, `reflectance` et les fichiers d’index dans l’image de la caméra principale, puis les recadre à la zone de chevauchement des membres (plus petite que le capteur en raison des décalages des caméras, `XMP-mapir:AlignmentJSON` `"applied": true`), de sorte qu’ils correspondent aux fichiers capturés par l’application Chloros pour la même scène ; `raw` reste au format natif. Auparavant, une capture sans `aligned=True` enregistrait chaque fichier à sa taille native, sans correction de distorsion. `aligned=False` conserve ce comportement. Les fichiers `chloros-mcp``capture_array` et `start_capture_run` adoptent la même valeur par défaut. Voir [Méthodes `ArraySession`](#arraysession-methods).
> - **`connect_all()` désigne un backend qui refuse le SDK.** Un backend en cours d’exécution qui est déconnecté ou souscrit à une formule inférieure à Copper est signalé sous `errors` avec sa propre raison (HTTP 401 / 403), et non comme « aucun backend Chloros n'est en cours d'exécution » ; chaque appel `connect_all()` vérifie à nouveau le backend, de sorte que l'appeler après la connexion fonctionne sur le même objet de projet.
> - **Un `output_dir` relatif fonctionne avec un backend local.** Les chemins relatifs sont convertis en chemins absolus par rapport au répertoire de travail de votre script avant d’être envoyés. Dans la version 1.2.0, le backend les résolvait dans son propre dossier d’installation, ce qui entraînait un échec avec le message « Accès refusé ».
> - **`numpy` est désormais une exigence** du SDK (les appels live-frame renvoient des tableaux numpy).
> - **Un nom de niveau d’exportation qui n’est pas « one » est refusé.** `processing=` et `levels=` prennent en charge `raw`, `debayered`, `radiance`, `reflectance` ou `display` (ou un alias tel que `preview` ; `processing=` ainsi que `"all"`). Tout autre nom est refusé avant que la caméra ne soit activée (400 HTTP `UNKNOWN_LEVEL` : `capture()` déclenche `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`) ; la version 1.2.0 a enregistré ou renvoyé `raw` pour cela.
> - **La prise en charge des textures est refusée pour les captures LATTICE.** Sur un projet contenant des captures LATTICE, `configure(debayer="Texture Aware (Slow, Highest Quality)")` déclenche `ChlorosConfigurationError`, tout comme `process()` après un `configure()` qui a demandé avant l’importation. Une valeur « Texture Aware » enregistrée précédemment par le projet est définie sur « Standard », avec un code d’erreur `UserWarning`. Les captures LATTICE bénéficient toujours du dématriçage standard ; la version 1.2.0 a donc accepté ce paramètre et l’a ignoré ; l’application Chloros ne le propose pas non plus pour ces captures. Voir [Valeurs prises en charge](#supported-values).
> - **Les caméras reviennent à leur état d’origine — sauf en cas de `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, la fin de chaque fonction transitoire (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) ainsi que l’arrêt de `start_camera_continuous()` rétablissent chaque caméra dans son état initial : son câblage de déclenchement et de ligne de synchronisation, son PTP, ainsi que tous les paramètres de la caméra modifiés au cours de la session (exposition et gain, la cible AE, les limites et l’amortissement, la balance des blancs, le niveau de noir, le format de pixel, le binning et la zone d’intérêt (ROI), les inversions, la fréquence d’images). `keep_settings=True`, quant à lui, enregistre les paramètres de chaque caméra dans sa propre mémoire, afin qu’elle démarre avec ces paramètres ; `clear_settings=True` réinitialise cette mémoire aux paramètres d’usine. Dans les deux cas, les caméras continuent de fonctionner en mode autonome. Un projet configure ses caméras à partir de ses propres paramètres enregistrés lorsqu’il les connecte. Voir [Mémoire de la caméra](#camera-memory--keep_settings--clear_settings).
> - **Tous les modes enregistrent un projet de la même manière.** Toute modification apportée à une caméra, un réseau de caméras ou un capteur de luminosité connecté à un projet — depuis le SDK, l’application, la CLI ou le serveur MCP — est enregistrée dans le fichier `cameras.json` / `sensors.json` de ce projet, et l’ouverture du projet dans n’importe quel mode configure les périphériques à partir de celui-ci ; un `ChlorosProject` nomme son propre dossier vers le backend. Auparavant, seule l’application enregistrait le projet. Voir [Un enregistrement de projet, tous modes confondus](#one-project-record-every-mode).
> - **Les fichiers créés à partir de plusieurs caméras mentionnent chacune d’entre elles.** Une pile multibande, un index en temps réel constitué des bandes de plusieurs caméras et une image de capture matricielle `composite/` contiennent le numéro de série, le modèle et le pack d'étalonnage de chaque caméra source sous forme de listes (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), tandis que les listes EXIF `SerialNumber` répertorient les numéros de série séparés par des virgules. Auparavant, elles ne comportaient aucune information sur les caméras, ou uniquement le numéro de série de la première caméra. Voir [Fichiers créés à partir de plusieurs caméras](#files-made-from-several-cameras).
> - **Seul un fichier `raw` contient les balises d’étalonnage de la caméra.** L'ensemble compatible MicaSense (`BlackLevel`, `RadiometricCalibration`, les ensembles parcanal, les ensembles `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`) ainsi que la charge utile `CalibrationJSON` décrivent les valeurs brutes du capteur ; ils sont donc intégrés uniquement au fichier de capture `raw`. Tous les autres fichiers — les fichiers de capture`debayered`, `display`, `radiance`, `reflectance` ainsi que les fichiers d’index, les trames d’enregistrement continu et de flux multi-caméras— ainsi que tous les fichiers de produit `process()` — porte l’identifiant de la caméra (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) et, lors d’une capture, les paramètres propres à l’image (`XMP-mapir:FrameJSON` : exposition, gain, température, format des pixels, heure de capture). Dans la version 1.2.0, ils contenaient également l'ensemble de données brutes, qu'un outil (Pix4D, Metashape) appliquait aux pixels ne correspondant pas à des valeurs brutes. Le traitement détermine l'étalonnage d'un tel fichier à partir de son numéro de série et de `CalBundleSha256`.
>
> **Autres modifications :**
>
> - **La fréquence d’images automatique d’un réseau s’adapte à ce que celui-ci fournit.** Une fréquence qui perd plus de 1 % de ses déclenchements est réduite, après évaluation sur plusieurs observations prolongées plutôt que sur une seule, et la fréquence à partir de laquelle elle a été réduite est réessayée de temps à autre. Une fréquence que vous définissez vous-même reste inchangée.
> - **L’exposition automatique est plus stable.** Elle se maintient en cas de luminosité constante, se stabilise en quelques secondes après un changement, ignore les petites zones surexposées et rétablit le fonctionnement d’une caméra qui cesse de répondre en plein soleil. Voir [Comment fonctionne l’exposition automatique](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` ne comptabilise que vos images et vos enregistrements.** Un dossier contenant le fichier `camera_cal/` d’un projet ne voit plus ces fichiers d’étalonnage comptabilisés comme des images et des enregistrements, et un projet qui les avait déjà intégrés les supprime.
> - **`configure(radiometric_output=...)` s’applique.** Il définit désormais les paramètres des produits `Export radiance` / `Export reflectance` du projet ; auparavant, les paramètres propres à chaque projet prévalaient sur ceux-ci, de sorte que `radiance`, `sensor-response` et `all` exportaient tous la radiance et la réflectance.
> - **`test_array_sync()` démarre à partir de l’état de mise sous tension des caméras.** Il active le PTP des caméras pour la session, attend que leurs horloges se synchronisent et se stabilisent avant son premier déclenchement (`ptp_lock`) — chaque caméra reste verrouillée pendant 120 s, soit environ 2 minutes à compter de la mise sous tension (`ptp_settle_s=` le configure), car une horloge qui vient de se synchroniser présente un décalage pouvant atteindre environ 200 µs pendant environ 2 minutes — et rétablit le PTP de chaque caméra tel qu’il l’a trouvé. Une caméra encore verrouillée depuis un test précédent n’est pas attendue à nouveau. Auparavant, il désactivait le PTP sur un réseau à commutateur unique où aucun DAQ-E ne répondait, et considérait l’horloge libre de chaque caméra comme une horloge PTP qui ne se verrouillerait pas (« PTP non verrouillé », à chaque exécution).
> - **Un enregistrement `.daq` indique si son capuchon a été sélectionné ou supposé.** Les DAQ-U, DAQ-M et DAQ-E sont livrés sans capuchon, et Chloros ne peut pas détecter si un capuchon est installé ; ainsi, un capteur connecté sans capuchon spécifié reçoit par défaut le nom `sunshine_cosine`, ce qui correspond à une *supposition*. Chaque enregistrement Chloros (schéma 1.30) stocke désormais `cap_id_source` à côté de `cap_id` : `operator` lorsque vous avez nommé un capuchon (`none` inclus), `auto_default` lorsqu’il s’agissait de la valeur par défaut. Le traitement d’un enregistrement dont le condensateur a été supposé, avec le condensateur DAQ du projet laissé sur « Auto », génère un fichier `[daq-dls] WARNING` portant le nom du fichier : un DAQ-U nu, après correction « sunshine », affiche une valeur environ 30 fois trop élevée. **Nommez le condensateur que vous avez installé** lors de la connexion (`connect_daq_sensor(cap_id=...)` ou `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, ou le menu déroulant des condensateurs de l’interface graphique), ou définissez le condensateur DAQ du projet, et l’avertissement disparaîtra.
> - **Plus de fenêtres de console pip sous Windows.** Sur un PC équipé de Python, le backend et chacun de ses image workers exécutaient `pip` une fois au démarrage, et chacun ouvrait une fenêtre de console. Ce n’est plus le cas.
> - **Les enregistrements `.daq` en direct résistent à une coupure de courant.**Un enregistrement est écrit avec un journal et une synchronisation complète du disque à chaque validation, en arrière-plan du capteur ; ainsi, un disque lent retarde le fichier, mais jamais les mesures. Un enregistrement interrompu en cours de validation (coupure de courant, arrêt forcé) peut laisser un fichier `<name>.daq-journal` à côté :**copiez les deux ensemble.**Chloros effectue une restauration du journal lors de la première lecture du fichier ; l’enregistrement s’ouvre donc avec toutes les données validées avant l’arrêt (seules les 3 dernières secondes environ sont perdues) ; dans la version 1.2.0, une coupure de courant au mauvais moment pouvait rendre l’intégralité du fichier illisible (« l’image disque de la base de données est mal formée »).
> - **`DAQSensorSession.record_start()` sur un disque très sollicité ne bloque plus le capteur** : les mesures sont mises en file d’attente et écrites sur un thread dédié, et celles qui n’ont pas pu être écrites sont comptabilisées dans `status()` de `record_errors` (avec `record_last_error`).
> - **`LatticeCamera` direct :**une image sans exposition/gain par bloc indique désormais l’exposition et le gain**réels** de la caméra (auparavant, la solution de secours consistait à lire une valeur mise en cache qui pouvait rester figée pendant tout un cycle d’exposition automatique), et les versions `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` appliquent le même filtrage d’entrée de 5 µs dont bénéficient les esclaves câblés d’Chloros (sans filtrage, un esclave déclenchait des images sans qu’aucun signal de déclenchement ne soit envoyé).
> - **Les caméras déclenchées par logiciel ne perdent plus un déclenchement sur deux.** Une caméra en mode de déclenchement logiciel met désormais en file d’attente un déclenchement qui arrive pendant sa lecture (ou Chloros attend qu’elle soit prête), comme le faisaient déjà les caméras synchronisées par câble.
> - **`discover_daq_sensors()` désigne un DAQ-E-S `DAQ-E-S`.** Son `model` était `DAQ-E` pour chaque capteur Ethernet ; celui qui ne répond pas au contrôle d’identité dans un délai d’environ 2 s reste répertorié, sous le nom de `DAQ-E`.
> - **Nouveau : `chloros-mcp`, un serveur MCP pour les assistants IA.** Claude Desktop, Claude Code, Cursor et d’autres clients MCP peuvent traiter des images (créer un projet, importer des captures, choisir les paramètres, traiter, générer un rapport sur les résultats de l’exécution) et piloter les caméras LATTICE, les réseaux de caméras et les capteurs de lumière DAQ : connexion, prévisualisation, réglage de l’exposition et des couleurs, capture, enregistrement. Ses outils couvrent les fonctionnalités de ce SDK. Il s’installe avec le SDK, et le fichier .deb pour Linux le place dans `/usr/bin`. Voir [Serveur MCP (assistants IA)](#mcp-server-ai-assistants).
> - **Nouveau : utilitaires pour caméras**, les commandes `chloros-cli lattice` sous forme d’appels au SDK : profil de couleur, saturation / contraste, hautes lumières, balance des blancs (en une seule prise, automatique, rapports fixes), préréglages, modèles de filtres, état et mise à jour du micrologiciel, alimentation (détection, veille, réveil), étalonnage de la cible, adaptateurs réseau (vérification, estimation, correction), correction du vignetage sur les fichiers, lecture DLS et détection de l’hôte. Voir [Utilitaires de la caméra](#camera-utilities).
> - **Nouveau : le reste de l’CLI sous forme d’appels SDK**, de sorte que l’CLI, le SDK et `chloros-mcp` offrent les mêmes fonctionnalités (l’CLI ne conserve que la connexion, le provisionnement d’un DAQ-E et les serveurs DAQ directs, pour des raisons de sécurité) : caméras ouvertes en une seule commande (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), les fichiers de profil d’alignement (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), des index (`index_catalog()`, `compute_index()`, `capture_live_index()`), le « grandmaster » PTP (`time_sync_status()` et ses dérivés), le dossier du projet (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) et les recettes (`run_project_recipe()`, le programme d’exécution propre à l’CLI). Pour les capteurs de lumière : `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` et `light_sensor_reflectance()` (réflectance ambiante et sur les objets ainsi que les indices de deux capteurs) ; pour une caméra : `CameraSession.status()` et `capture_repeated()` ; pour un réseau : `ArraySession.capability()` et `set_member_settings()` (exposition, gain, etc. d'un élément). Voir [Matrices transitoires, profils d’alignement et indices](#transient-arrays-alignment-profiles--indices), [Dossier de projet et recettes](#project-folder--recipes) et [Sessions de capteurs DAQ](#daq-sensor-sessions).
> - **Amélioration des produits d'indices LATTICE** (NDVI et autres indices) issus de `process()`, dans tous les modes de traitement.
> - **Caméras LATTICE OCN : amélioration des produits de radiance, de réflectance et d’indices en plein soleil.** Une correction de flotte est appliquée après le démixage des bandes — en temps réel, dans les captures et dans `process()` ; les caméras RGN restent inchangées. Voir [Sortie radiométrique](#radiometric-output-lattice-multispectral-pipeline).
> - **Chaque capture enregistre la valeur DAQ utilisée, comme le fait l’interface graphique.** `CameraSession.capture()` et `ArraySession.capture()` / `continuous()` écrivent le fichier `.daq` et son fichier `.match.json` chaque fois qu’un DAQ est lié ou mis en pool, y compris une capture « brute » ; dans la version 1.2.0, une capture brute ou débayérisée n’en enregistrait qu’une seule avec `force_daq=True`. `daq=False` désactive cette fonctionnalité et `daq_tolerance_ms=` définit la tolérance de correspondance ; les deux résultats indiquent ce qui a été enregistré dans `.daq_files` et `.daq_matches`. Une capture attend désormais la lecture suivante du DAQ afin de pouvoir fusionner les lectures de part et d’autre de l’exposition. Voir [La lecture du DAQ enregistrée par une capture](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` contient des images en temps réel et une capture continue côté serveur.** `grab()` et `frame_stream()` renvoient des images numpy issues d’un seul déclenchement (la même lecture que les API `ArrayHandle.grab()` / `frame_stream()` du projet), et `continuous()` enregistre chaque groupe synchronisé dans le backend et renvoie un [`ContinuousHandle`](#continuoushandle). Voir [Tables en temps réel dans votre propre pipeline](#live-frames-in-your-own-pipeline).
> - **Un tableau `grab()` est associé à un déclencheur** (`ArrayHandle.grab()` et le nouveau `ArraySession.grab()`). Dans la version 1.2.0, 12 des 20`ArrayHandle.grab()` à quatre caméras chevauchaient deux déclencheurs. Chaque image porte désormais `group_id`, et un tableau `frame_stream()` génère chaque déclencheur une seule fois. Voir [Un déclencheur par lecture](#one-trigger-per-read--group_id).
> - **Images par bande et alignées :** `grab()` / `frame_stream()` prennent `band=`, `align=`, `bit_depth=` et `refl_max=` sur les descripteurs de caméra et de matrice. Voir [Exporter une seule bande](#exporting-a-single-band--band).
> - **Nouvelles méthodes et nouveaux arguments `ArraySession` :** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (avec `CaptureResult.group_seq` et `.failed`), `continuous()`, `connect_array(packed=...)` ; `capture_repeated()` enchaîne les groupes consécutifs et s'arrête lorsque le disque est plein.
> - **`ArraySession.record()` enregistre sans interface graphique ouverte.** Lorsque la vue combinée de l'interface graphique est fermée, le backend génère la vue lui-même : leNDVI du tableau, ou l’index que vous transmettez comme `expr=`. `RecorderHandle.stop()` déclenche une exception `ChlorosConnectError` pour un enregistrement ayant échoué, c’est-à-dire qui n’a écrit aucune image, et conserve le résultat de l’arrêt sur `.result` (laisser un bloc `with` ne déclenche pas d’exception : vérifiez `rec.result`). Voir [`RecorderHandle`](#recorderhandle).
> - **Nouvelles méthodes `DAQSensorSession` :** `enable_ptp()`, `force_disconnect()`, et pour les paramètres propres à un DAQ-E, `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` et `update_firmware()`. `record_stop()` renvoie `warning` et `gaps`. Un redémarrage ayant clôturé un enregistrement le nomme, dans sa réponse ou, en cas d’échec du redémarrage, sur `ChlorosConnectError` (`recording_closed` ; voir [Paramètres du périphérique DAQ-E](#daq-e-device-settings)). Voir [Méthodes `DAQSensorSession`](#daqsensorsession-methods).
> - **Un DAQ-E en pool qui cesse de transmettre est reconnecté par le backend**, avec ses paramètres d’origine, et un enregistrement en cours sur celui-ci se poursuit dans le même `.daq`, l’interruption étant marquée dans le fichier ; `record_stop()` renvoie les interruptions sous la forme `gaps`. Dans la version 1.2.0, le capteur restait silencieux jusqu’à ce qu’il soit déconnecté puis reconnecté, ce qui mettait fin à l’enregistrement. Voir la section [Un DAQ-E qui cesse de fonctionner](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` fonctionne à partir d’un SDK installé.** Le programme d’installation (etet PyPI) contient désormais les chargeurs de projet dont il a besoin ; auparavant, `open_project()` déclenchait `ImportError` en nommant `project_cameras` / `project_sensors` dans tout SDK installé via pip. `connect_all()` connecte les caméras, les matrices et les capteurs de lumière enregistrés via un backend en cours d’exécution (les capteurs de lumière via son pool DAQ), et un périphérique enregistré qu’il ne peut pas détecter est répertorié sous `errors` avec la raison indiquée, au lieu d’être ignoré sans explication. Un tableau enregistré qui est déjà actif dans le backend (celui de l’interface graphique, ou celles de `chloros-cli project connect`) est connecté et reste en fonctionnement, alors qu’auparavant cela était signalé comme une erreur (« La caméra  fait<sn> déjà partie du réseau <id>»). Voir [Méthodes `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` et `process_lattice_capture()` détectent Chloros sous Windows.** Elles recherchent l’emplacement où le programme d’installation place le backend (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` par défaut, un dossier personnalisé indiqué dans les propres données de l’installateur), et se connectent à un backend déjà en cours d’exécution sans en rechercher un autre. Dans la version 1.2.0, ils généraient l’erreur « Backend Chloros introuvable » sur une machine installée, à moins que vous n’ayez passé `backend_exe=`.
> - **Un backend qui se ferme au démarrage est immédiatement signalé.** Lorsque `ChlorosLocal()` ou un appel smart-connect lance un backend qui se ferme avant de répondre, l’appel génère en quelques secondes l’erreur `ChlorosBackendError` pour `ChlorosLocal` et `ChlorosConnectError` pour les appels smart--connect, avec le code de sortie, la commande qu’il a exécutée et les dernières lignes affichées par le backend. La version 1.2.0 attendait l’expiration du délai de démarrage de 60 s sans en indiquer la raison. Voir [Démarrage automatique du backend](#backend-auto-start).
> - **Le backend continue de fonctionner entre deux scripts.** Quitter un bloc `with ChlorosLocal(...)`, le retour de `process_folder()` / `process_lattice_capture()` et la sortie de `chloros-mcp` n’arrêtent plus un backend lancé par le SDK ; le script suivant le réutilise. Il s'arrête sur `logout()` (à moins que l'application Chloros soit ouverte ou qu'une exécution soit en cours), sur `shutdown_backend()`, ou de lui-même après 30 minutes si rien ne l'utilise (`CHLOROS_BACKEND_IDLE_MIN` dans l’environnement du backend modifie ce comportement ; `0` : jamais).
> - **Un backend lancé par le SDK est invité à s’arrêter, il n’est pas forcé à se fermer.** `shutdown_backend()` permet au backend de libérer ses caméras et ses capteurs de lumière et de terminer tout enregistrement ; il n’est arrêté que s’il est toujours en cours d’exécution 10 secondes plus tard. La version 1.2.0 l’arrêtait immédiatement (sous Windows, un arrêt forcé), ce qui pouvait rendre inopérant un capteur de lumière DAQ connecté à ce backend. Un backend que le SDK détectait comme étant en cours d’exécution n’était toujours pas affecté.
> - **Sous Linux, le démarrage automatique n’exécute jamais un `backend_server.py` à partir du dossier courant.** En l’absence de backend installé, un script lancé dans un dossier en contenant un l’exécutait ; désormais, seul le backend installé est lancé (ou, pour `ChlorosLocal`, celui que `backend_exe=` désigne).
> - **`configure(indices=...)`**ajoute chaque préréglage au projet correspondant aux bandes de sa caméra, y compris un nouveau projet, et affiche un avertissement (`UserWarning`) pour tout nom qu’il ne peut pas ajouter.**`import_images()`** signale les enregistrements `.daq` sous le nom `scan_count` et n’affiche plus d’avertissement concernant un dossier de capture LATTICE.
> - **`read_image_audit_tags()` fonctionne à partir d’une installation via pip.** Il lit lui-même les métadonnées de l’image ; dans la version 1.2.0, il renvoyait `{}` sans les composants internes de l’installation sur le bureau dans le chemin d’accès à Python.
> - **`capture()`, juste après `connect_array()`**, attend jusqu’à 20 s pour le premier groupe d’images du tableau au lieu d’afficher l’erreur HTTP 503 « Array still warming up » (Le tableau est encore en phase de préchauffage).
> - **`status()` signale un blocage.** Le nouveau champ `stalled` est à True pendant que la boucle de capture s’exécute mais ne publie rien, et `actual_fps` est alors égal à 0. Dans la version 1.2.0, `actual_fps` conservait sa dernière valeur pendant un blocage ; ainsi, un tableau bloqué affichait `actual_fps > 0` tandis que les captures échouaient avec le code d’erreur 503 « Aucun nouveau groupe d’images synchronisées ».
> - **Moins d’ensembles de trames incomplets et d’erreurs 503 lors de la capture :** l’entrée de déclenchement de chaque caméra esclave est désormais débarrassée des parasites.
> - **Transport 12 bits compressé** (`connect_array(packed="auto")`) augmente la fréquence d’images de l’ensemble de capteurs d’environ un tiers. Voir [Transport 12 bits compressé](#packed-12-bit-transport-packed).
> - **Exposition de la matrice sous une zone d’intérêt (ROI) :** une petite zone d’intérêt ne limite plus l’exposition d’un élément à la période de fonctionnement libre du capteur (4,9 ms à 768 × 304). Voir [Exposition des éléments de la matrice](#exposure-on-array-members).
> - **L’exposition automatique ne réduit plus la fréquence d’images automatique d’un réseau.** Un réseau synchronisé par câble fonctionne à la fréquence autorisée par son réseau et la lecture des caméras ; l’exposition automatique remplit la période correspondant à cette fréquence, puis augmente le gain. Une exposition manuelle plus longue que cette période ralentit le déclenchement pour s’y adapter au lieu d’être refusée. Voir [Exposition sur les éléments du réseau](#exposure-on-array-members).
> - **`read_nodes()`**ajoute `frame_values` et `notes` pendant que l’exposition automatique est active (voir [`read_nodes()` en exposition automatique](#read_nodes-under-auto-exposure--frame_values-and-notes)). Lorsque la caméra refuse une exposition ou un gain,**`set_settings()`** indique la plage qu’elle accepte et réactive le mode automatique qu’elle avait désactivé pour cette valeur ; `set_settings(target_brightness_auto=True)` est un nouveau code.
> - **`orient=`** sur `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` et `ArrayHandle.capture()` / `capture_smart()` fait pivoter les produits en fonction de l'orientation de la caméra. Voir [Méthodes `ArraySession`](#arraysession-methods).
> - **`DLS()` nécessite le `mac_address` propre au capteur.** Il n'y a plus de MAC par défaut (il s'agissait auparavant de l'adresse d'une unité) ; l'appel de `DLS()` sans MAC déclenche `DLSError`. Voir [les amplitudes de bande DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Le programme d'installation Windows** propose les options « Activer les chemins d'accès longs sous Windows » et « Exclure le backend Chloros de l'analyse par Microsoft Defender », toutes deux cochées par défaut. Voir [Étape 1](#step-1--install-the-chloros-platform-package).
> - **Le fichier .deb pour Linux** installe le Python SDK pour le système `python3` (auparavant, il se contentait de placer le fichier wheel sur le disque), augmente la taille des tampons de réception des sockets du noyau dont les flux des caméras ont besoin, et ajoute un service backend s'exécutant sous votre compte utilisateur. Le paquet Jetson est `chloros_1.3.0_arm64_jp6.deb`, pour JetPack 6, avec PyTorch CUDA de JetPack. Voir [Étape 1](#step-1--install-the-chloros-platform-package) et [Étape 2](#linux-deb).

---

## L’architecture en 60 secondes

Le SDK est une fine couche Python recouvrant le backend Chloros (le même serveur Flask que celui utilisé par l’interface graphique de bureau et la CLI). Pour l’automatisation, vous importez `chloros_sdk` et appelez des méthodes de haut niveau ; en arrière-plan, chaque appel se transforme en requête HTTP vers le backend local sur le port 5000 — `http://127.0.0.1:5000/api/...` (et non `localhost`, qui redirige d’abord vers `::1` sous Windows et coûte environ 2 s par requête vers un backend IPv4 uniquement). Le backend gère le parc matériel — caméras, capteurs DAQ, profils d’alignement, tampons d’images — ce qui permet aux scripts SDK de coexister avec l’interface graphique sans se disputer les ports série ou la bande passante des cartes réseau.

Vous utiliserez trois interfaces :

1. **`ChlorosLocal` + fonctions libres** (`process_folder`, `process_lattice_capture`) — Pipeline de traitement d’images. Traitez l’intégralité d’un dossier (étalonnage / débayérisation / exportation d’index) à partir d’un seul appel Python.
2. **Maniement Smart-Connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Ouvrez une session backend persistante pour le matériel en temps réel. Même flux « smart-prep » que l’interface graphique : sondage réseau, sélection automatique du niveau, PTP, initialisation AE, configuration du déclencheur GPIO.
3. **`ChlorosProject` / `open_project`** — Charge un projet enregistré (dossier contenant `cameras.json` + `sensors.json` + `project.json`), connecte simultanément ses caméras, ses matrices et ses capteurs de lumière, et déclenche les captures à l’aide de descripteurs nommés.

Les surfaces 1 et 2 **lancent automatiquement un backend local** si aucun n’est déjà à l’écoute (le même binaire fourni que celui lancé par l’interface graphique ou la CLI) — ainsi, un simple script fonctionne depuis un shell vierge sans que vous ayez à démarrer un backend au préalable : `ChlorosLocal` (ainsi que `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` et `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` et `analyze_array_network()` ne démarrent jamais : ils ont besoin d’un backend en cours d’exécution. Passez `auto_start_backend=False` pour désactiver cette fonctionnalité (par exemple, lorsque l'on pointe vers un backend distant, qui n'est jamais lancé). Voir [Démarrage automatique du backend](#backend-auto-start). La Surface 3 se comporte différemment : `open_project()` n’accepte aucun paramètre `auto_start_backend`, et `connect_all()` ne lance jamais de backend — il interroge `http://127.0.0.1:5000` une fois et, si aucune réponse n’est reçue, revient au contrôle direct (sans backend) du périphérique via `lattice_sdk`. Cela nécessite le runtime de l’SDK Arena et OpenCV (`cv2`) importable dans votre Python, ce que la roue du SDK n’installe pas ; sans eux, chaque caméra et chaque tableau enregistrés sont signalés comme une erreur, avec la raison correspondante. Les capteurs de lumière d’un projet ne disposent d’aucun chemin sans backend : en l’absence de backend en cours d’exécution, chaque capteur enregistré est signalé comme une erreur, avec la raison correspondante (voir [`SensorHandle`](#sensorhandle)). Seuls `proj.process()` et `stream(..., overlays=True)` construisent de manière différée un `ChlorosLocal()` (qui s’exécute automatiquement au démarrage).

Ces trois éléments sont auth: exécutez `chloros-cli login` une fois sur la machine, ou connectez-vous via l’interface graphique du bureau. L'exception levée par un appel sans session valide dépend de l'interface (voir [Exceptions](#exceptions)) : `ChlorosAuthenticationError` à partir de `ChlorosLocal` (et `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` provenant des appels smart-connect et de leurs sessions, `status: "error"` avec une exception `error_code` provenant de `analyze_array_network()`, et une entrée sous `errors` dans le rapport `connect_all()`.

Configuration requise :
- Python 3.7+ (conformément aux spécifications du paquet ; développé/testé sur la version 3.10)
- Chloros Desktop installé localement (le binaire du backend est fourni dans le programme d’installation)
- Compte Chloros+ actif sur **n’importe quel niveau payant de Chloros+ (Copper ou supérieur)**; le niveau gratuit**Iron**ne donne pas accès au SDK ni à la CLI. Cette restriction est appliquée**côté serveur**** : chaque requête marquée SDK/CLI doit comporter à la fois une session active et un forfait payant. Un forfait inférieur à Copper génère le code d'erreur `403` avec `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` provenant des appels `ChlorosLocal`, `ChlorosConnectError` pour les appels smart-connect). Un appelant déconnecté reçoit à la place les codes `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` issu de `ChlorosLocal`, `ChlorosConnectError` issu des appels Smart-Connect) — ces deux codes sont distincts car la réexécution de `chloros-cli login` corrige l’erreur `401` mais ne peut pas corriger l’erreur `403`, qui nécessite une mise à niveau du forfait.
- L'utilisation hors ligne est prise en charge pendant la période de grâce du forfait : le niveau d'accès est lu à partir du cache de validation du serveur (5 min) ou du cache de licence signée et liée à la machine (30 jours pour les forfaits mensuels, jusqu'à l'expiration de l'abonnement pour les forfaits annuels). À l’expiration de ce délai de grâce, le forfait passe en version gratuite et l’accès au SDK et à la CLI est interrompu jusqu’à ce que la machine puisse se connecter au serveur au moins une fois. L’erreur `chloros-cli status` (`GET /api/license-status`) reste accessible dans le niveau gratuit afin que la raison soit visible.
- Windows 10/11 64 bits, **Ubuntu 22.04 LTS ou version plus récente**, ou Jetson (JetPack 6). Ubuntu 20.04 n’est**pas** pris en charge : les dépendances de `.deb` découlent des bibliothèques auxquelles le backend est lié, y compris `libc6 (>= 2.35)`, et Focal intègre glibc 2.31.

---

## Installation

Le Python SDK est une fine couche Python superposée au backend Chloros. À moins que vous n’utilisiez un backend sur une autre machine (voir [Mode backend à distance](#remote-backend-mode-pip-only-host-via-tunnel)), vous devez disposer du **paquet Chloros pour bureau installé localement** (programme d’installation Windows ou Linux `.deb`), qui inclut les capteurs de lumière DAQ. Ce paquet fournit le binaire du backend, le runtime Arena SDK pour les caméras LATTICE, ainsi que les ensembles d’étalonnage.

Derniers téléchargements : [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Étape 1 — Installer le paquet de la plateforme Chloros

#### Windows (.exe)

1. Téléchargez `Chloros Installer 1.3.0.exe` depuis la page de téléchargement.
2. Exécutez le programme d’installation et suivez les instructions de l’assistant. Le chemin d’installation par défaut est `C:\Program Files\Chloros\`.
3. Lancez Chloros au moins une fois et connectez-vous avec votre compte Chloros+.

Le programme d’installation autorise le backend (`chloros-backend.exe`, rien d’autre) de passer à travers le pare-feu Windows sur chaque profil réseau : entrées UDP, pour les flux des caméras, PTP, mDNS et le flux DAQ-E, et entrées TCP provenant uniquement du sous-réseau local, pour la connexion qu’un DAQ-E établit vers l’ordinateur lors d’une mise à jour du micrologiciel ([`update_firmware()`](#daq-e-device-settings)). Le programme de désinstallation supprime ces deux règles. L’API HTTP HTTP du backend reste sur `127.0.0.1` dans tous les cas.

Le groupe **Paramètres système** de l’assistant comporte deux options, toutes deux cochées par défaut :

| Option | Fonction | Désactivation de l’installation silencieuse |
| --- | --- | --- |
| **Activer les chemins d’accès longs sous Windows** | Permet aux dossiers de projet d’être situés à un niveau plus profond que la limite de 260 caractères imposée par Windows. Lorsque les chemins d’accès longs sont désactivés (paramètre par défaut de Windows), un dossier de projet situé à un niveau trop profond pour ses produits est refusé avant tout traitement — voir [Dépannage](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Exclure le backend Chloros de l’analyse par Microsoft Defender** | Empêche l’analyse en temps réelde chaque fichier généré par une capture de caméra. À des taux de capture élevés, l’analyse coûte environ 30 % du débit : mesuré avec une zone d’intérêt (ROI) de 512 sur un réseau de 9 caméras, la capture RAW en continu a atteint 52,7 captures/s avec l’analyse et 77,7 captures/s avec le backend exclu. | `/MERGETASKS="!defenderexclusion"` |

Une capture continue signale la charge de Defender sous le nom `defender_cores` (voir [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Installez avec `apt install ./…`, et non `dpkg -i` : apt installe également les dépendances du SDK Python, que le paquet recommande plutôt qu’il n’exige (voir [Étape 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Pour Jetson Orin / Orin NX / Orin Nano sous JetPack 6 (L4T R36). Son PyTorch est la version CUDA de JetPack 6 ; le traitement GPU s’effectue donc sur le GPU de l’Orin ; il utilise les bibliothèques CUDA et cuDNN installées par JetPack 6 (`sudo apt install nvidia-jetpack` sur une image qui ne les contient pas).

#### Linux : tampons de réception et service backend

Les deux paquets .deb augmentent à 32 Mo les tampons de réception des sockets du noyau nécessaires aux flux des caméras LATTICE (`/etc/sysctl.d/60-chloros-gige.conf`). Avec les paramètres par défaut du noyau, les paquets de la caméra sont rejetés par le noyau et un tableau perd des images ainsi que sa fréquence d’images ; `chloros-cli lattice network` signale les tampons lorsqu’ils sont insuffisants. Consultez la section [Hôtes Linux — tampons de réception des sockets](cli-reference.md#linux-hosts--socket-receive-buffers) de la référence CLI.

Sur une machine sans interface graphique, exécutez le backend en tant que service **sous votre compte utilisateur**, afin qu’un script puisse en détecter un en cours d’exécution (`connect_all()` ne démarre jamais de backend) :

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Le backend s’exécute alors en tant que vous : les captures qu’il écrit vous appartiennent, et il utilise votre `chloros-cli login`. Le `chloros-backend.service` de root est toujours installé pour les machines qui l’ont activé, mais chaque fichier écrit par un backend root appartient à root. Consultez la section [Exécution du backend en tant que service](cli-reference.md#running-the-backend-as-a-service-headless) de la référence de l’CLI.

Un backend exécuté par le service est partagé et n’est jamais arrêté par un client : les scripts SDK et `chloros-cli` l’utilisent tel qu’ils le trouvent (`ChlorosLocal.shutdown_backend()` n'arrête qu'un backend que `ChlorosLocal` a lui-même démarré), et `GET /api/status` nomme l'unité `service`. Redémarrez-le avec `sudo systemctl restart chloros-backend@$USER.service`. La mise à jour du paquet arrête les services backend pendant son installation, de sorte qu’un script en cours d’exécution perd ses caméras et ses capteurs, puis redémarre tous ceux qui sont activés ; la désinstallation du paquet les arrête et désactive ceux qui sont activés. Un seul backend peut s’exécuter à la fois, donc n’activez qu’une seule unité : une deuxième refusera de démarrer et restera `failed` (code de sortie 98, pas de redémarrage), et son message `systemctl status` en explique la raison. Pour changer d’unité, désactivez celle que vous n’utilisez pas.</id></sn>

### Étape 2 — Installer le Python SDK

**Le programme d'installation de Chloros inclut un fichier « wheel » SDK correspondant.** Chaque programme d'installation Windows et chaque fichier .deb Linux place sur le disque un fichier `chloros_sdk-X.Y.Z-py3-none-any.whl` qui correspond exactement à la version de l'interface graphique, de la CLI ou du backend. Vous n'avez pas besoin de consulter PyPI pour rester à jour.

#### Windows

Le programme d’installation exécute automatiquement `pip install` à partir du fichier wheel fourni en utilisant votre Python système (le lanceur `py.exe` est privilégié, mais le système se rabat sur `python -m pip`). Aucune intervention n’est nécessaire : `import chloros_sdk` fonctionne dans votre environnement Python une fois l’installation réussie. Si Python n’est pas installé sur la machine, le programme d’installation ignore cette étape en silence et l’interface graphique ainsi que la CLI continuent de fonctionner.

#### Linux (.deb)

Les deux paquets .deb (x86_64 et Jetson) installent le SDK pour le système `python3` ; ainsi, `import chloros_sdk` fonctionne après l’installation, sans pip et sans connexion réseau. Les paquets (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` et les chargeurs de projet) sont placés dans `/usr/lib/python3/dist-packages`, et la commande `chloros-mcp` de l’SDK (le [serveur MCP](#mcp-server-ai-assistants)) dans `/usr/bin`, tous appartenant au paquet : une mise à jour les remplace et la suppression du paquet les supprime.

Les prérequis propres au SDK, `python3-numpy` et `python3-requests`, sont **recommandés, mais pas obligatoires** (l’interface graphique, la CLI et le backend sont autonomes) : `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` sur un Jetson) les installe, contrairement à `dpkg -i`. Le résultat de l’installation indique la version du SDK que `python3` importe, toute dépendance manquante, ainsi qu’un fichier `chloros_sdk` situé ailleurs dans le chemin d’accès de `python3` qui remplace la copie du paquet (comme celui installé précédemment avec `pip install --user`).

Dans un virtualenv, qui n’a pas accès aux paquets du système, installez-y le fichier wheel fourni :

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pour les déploiements Jetson en mode « air-gapped », cette opération s’effectue entièrement hors ligne : le SDK et le fichier wheel se trouvent déjà sur le disque.

#### PyPI public

Pour les hôtes utilisant uniquement pip (aucun paquet de bureau Chloros installé ; le SDK pilote alors un backend sur une autre machine, voir [Mode backend distant](#remote-backend-mode-pip-only-host-via-tunnel)) :

```bash
pip install chloros-sdk
```

PyPI propose la dernière version stable. Veillez à ce que la version du SDK corresponde à celle du backend Chloros qu’il pilote (voir [Gestion des versions et compatibilité](#versioning--compatibility)). L’utilitaire DAQ-M `DLS` nécessite également `bleak` : `pip install "chloros-sdk[camera]"` (ou `pip install bleak`).

#### Vérification

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Abonnement Chloros+ requis.** Tous les appels au SDK nécessitent une connexion Chloros+ active. Exécutez `chloros-cli login user@example.com 'YourPassword'` une fois par machine ; les identifiants sont mis en cache dans `~/.chloros/`.

### Ai-je besoin du package Desktop ?

Le package pip seul **ne suffit pas** pour la plupart des workflows. Voici ce dont chaque interface SDK a besoin :

| Interface SDK | A-t-elle besoin du package Desktop ? | Pourquoi |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Oui**(pour en démarrer un)**/ Non** (un backend déjà accessible) | Utiliser un backend en cours d’exécution (celui de l’application de bureau ou celui d’un autre script lancé) ; sinon, démarrer automatiquement celui qui est installé : `/usr/lib/chloros/chloros-backend` (sous Linux) ou `<install folder>\backend\chloros-backend.exe` (sous Windows ; `C:\Program Files\Chloros\backend\` par défaut, un dossier d’installation personnalisé est lu à partir des informations de l’installateur). Passez `backend_exe=` pour le nommer vous-même. |
| `chloros-mcp` ([serveur MCP](#mcp-server-ai-assistants)) | **Oui**(pour en démarrer un)**/ Non** (un backend est déjà accessible) | Basé sur `ChlorosLocal`, donc identique à la ligne ci-dessus. |
| `read_image_audit_tags` | **Non** | Lit les métadonnées propres à l’image traitée (bibliothèque standard Python uniquement). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Oui**(local)**/ Non**(à distance) | Clients HTTP purs via le backend. Backend local → paquet de bureau requis. Backend distant → `backend_url=`**via un tunnel** (voir Mode backend distant — les backends fournis se lient uniquement en boucle locale). |
| `ChlorosProject` / `open_project` | **Oui** | Accède aux projets enregistrés via le backend. |
| Classes LATTICE directes (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Oui** | Nécessite le runtime Arena SDK, fourni dans le package de bureau, ainsi qu’OpenCV (`cv2`) dans votre environnement Python, que ni le package de bureau ni le SDK n’installent (`pip install opencv-python`). Sinon, `CAMERA_AVAILABLE` correspond à `False` lors de l’importation. `DLS` nécessite également `bleak`. |

### Mode « Remote-Backend » (hôte pip uniquement, via un tunnel)

> **Le backend installé n'est pas accessible via le réseau local.** Il écoute uniquement sur
> la boucle de retour (`127.0.0.1` / `::1`), donc `backend_url="http://<lan-ip>:5000"`
> **ne peut pas atteindre un Chloros installé**. Pour piloter un backend sur une autre
> machine, redirigez vous-même son port de bouclage et pointez le SDKvers le
> tunnel :

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Les hôtes sans interface graphique / CI / robotiques peuvent conserver une machine avec l’installation complète du bureau comme « serveur Chloros » et `pip install chloros-sdk` partout ailleurs — mais le transport entre eux s’effectue via le tunnel configuré par l’utilisateur ci-dessus, et non via une URL LAN directe.

> **`ChlorosLocal` sur un hôte « pip-only ».** `ChlorosLocal(backend_url=BACKEND)` se connecte à un backend qu’il peut atteindre sans installation de bureau : il recherche un binaire de backend local uniquement lorsqu’il doit en LANCER un (aucune réponse et `auto_start_backend=True`). Jusqu’à la version 1.2.0, il recherchait d’abord le binaire et générait l’erreur `ChlorosBackendError` (« Backend Chloros introuvable… ») sur un hôte utilisant uniquement pip, même si un backend était accessible. Les dossiers que vous transmettez (`process_folder`, `import_images`) sont vérifiés sur cette machine et lus par le backend ; ainsi, avec un backend distant, ils doivent désigner le même dossier sur les deux machines, par exemple un lecteur partagé.

### Flux de travail DAQ uniquement (hôte pip-only)

`pip install chloros-sdk` ne peut pas, à lui seul, piloter un capteur DAQ : un capteur DAQ est toujours piloté via un backend Chloros avec `connect_daq_sensor()` :

- **Capteur sur cette machine :** installez ici le package Chloros pour ordinateur de bureau. `connect_daq_sensor()` démarre son backend si aucun n’est en cours d’exécution.
- **Capteur sur une autre machine :** cette machine a besoin du package de bureau ; accédez à son backend via le tunnel en [mode backend distant](#remote-backend-mode-pip-only-host-via-tunnel). Le backend ouvre le capteur sur sa propre machine, de sorte qu’un DAQ-U soit branché sur cette machine, qu’un DAQ-M se trouve à portée Bluetooth de celle-ci et qu’un DAQ-E soit sur un réseau accessible par celle-ci.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Dans tous les cas, le backend applique le plan « Chloros+ » : connectez-vous sur la machine qui exécute le backend (`chloros-cli login` ou l’application de bureau) avec n’importe quel forfait payant (Copper ou supérieur). Un backend déconnecté et un forfait inférieur à Copper génèrent tous deux un `ChlorosConnectError` indiquant la raison du backend (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Guide de démarrage rapide

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Index de l'API de premier niveau

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Traitement d’images — `ChlorosLocal`

Classe principale du pipeline. Lance le backend lors de sa construction (si aucun ne répond et `auto_start_backend=True`), crée et configure des projets, surveille la progression et renvoie des résumés après exécution.

### Constructeur

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Méthodes

| Méthode | Description |
| --- | --- |
| `create_project(project_name, camera=None)` | Crée un nouveau projet (éventuellement avec un modèle de caméra tel que `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Ouvre un projet enregistré (son dossier ou son nom dans le dossier des projets) et en fait le projet ouvert. Les chemins d’accès d’un projet déplacé sont réorientés vers le dossier dans lequel il se trouve désormais ; `relink` indique le nom du dossier dans lequel se trouvent désormais ses autres fichiers manquants. Renvoie la réponse du backend avec `source_files` ; lance une exception `ChlorosProjectFilesMissingError` si aucun fichier ne peut être trouvé. Voir **Un projet déplacé** ci-dessous. |
| `import_images(folder_path, recursive=False)` | Importe des images RAW/TIF/JPG/DNG **et `.daq` des enregistrements de capteurs de lumière**. Renvoie `count` (images) et `scan_count` (enregistrements). Émet un avertissement uniquement si le dossier ne contient ni l'un ni l'autre. |
| `export_light_sensor(daq=True, csv=True)` | Écrit les fichiers calibrés `.daq` + `.csv` pour chaque enregistrement du capteur de lumière du projet, dans `<project>/Light Sensor/`. Voir [Enregistrements du capteur de lumière](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Définit les curseurs de traitement. `indices` sont des noms de préréglages (`NDVI`, `GNDVI`, `OSAVI`, …), chacun étant ajouté au projet pour les bandes de sa caméra ; un nom qui n’est pas un préréglage, ou que les bandes ne permettent pas de créer (`NDRE` nécessite les bandes RE et NIR, qu’aucune caméra ne fournit à elle seule), reçoit un nom de type `UserWarning` et aucune image d’index. `daq_log_path` désigne un enregistrement par capteur de lumière (un fichier `.daq` ou `.csv`, ou un dossier contenant ces fichiers) auquel chaque image est comparée, tant dans LATTICE que dans Survey3, au lieu des enregistrements importés avec les images ; si un chemin d’accès est manquant ou ne contient aucun enregistrement, les enregistrements importés sont utilisés. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Exécute le pipeline. Renvoie `{"status": "complete", "async": False}`, ainsi qu’une clé `summary` lorsque le backend en fournit une — voir [Résumé et conseils post-exécution](#post-run-summary--hints) — `export_check_failure` lorsque les exportations de l’exécution n’ont pas pu être vérifiées, et `image_failure` lorsqu’une image a été perdue. `wait=False` renvoie `{"status": "started", "async": True}` simultanément. |
| `get_config()` / `get_status()` / `status()` | Vérifier l’état du backend. |
| `logout()` | Déconnexion : efface les identifiants mis en cache et libère l'emplacement de périphérique de cette machine (`device_deregistration` dans le résultat). Un cycle de traitement en cours n’est pas interrompu : il se termine dans le mode dans lequel il a démarré (`run_in_progress: true` et `run_mode` dans le résultat), et le cycle suivant utilise le traitement standard. |
| `shutdown_backend()` | Arrête le backend si cette instance l’a lancé ; un backend détecté en cours d’exécution n’est pas interrompu. Le fait de laisser un bloc `with` ou de soumettre l’instance au ramasse-miettes ne l’arrête pas : le backend continue de fonctionner pour le script suivant et s’arrête de lui-même au bout de 30 minutes s’il n’est utilisé par rien. Le backend est invité à s’arrêter ; il libère donc ses caméras et ses capteurs de lumière et termine d’abord tout enregistrement en cours ; il n’est arrêté que s’il est toujours en cours d’exécution 10 secondes plus tard. |
| `discover_cameras()` | Détecter les caméras LATTICE **via le backend de cette instance** (`/api/camera/discover`). Renvoie une liste de dictionnaires (`serial`, `model`, `ip`, …) — de même structure que celle affichée par l’interface graphique ou la CLI. Liste vide si aucune caméra n’est trouvée ou si le backend est inaccessible. |
| `camera_capture(output_dir, format="tiff", **settings)` | Capture une seule image**via le backend**(lancée automatiquement par ce descripteur) afin qu’elle bénéficie de la même préparation que dans l’interface graphique ou la CLI (12 bits par défaut, réutilisation du pool, métadonnées de calibrage intégrées). Résoudre la cible avec `serial=` ou `device_index=` ; transmettez `exposure`/`gain`/`pixel_format`/`preset` sous la forme de `**settings`. Renvoie le dictionnaire de métadonnées hérité (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Génère des images de prévisualisation composées par superposition à partir d’une caméra mutualisée — client MJPEG léger via la route `/api/camera/<serial>/stream-annotated` du backend (zèbre / grille / réticule / histogramme / peaking / spot dessinés côté serveur). `decode=True` génère des tableaux BGR et nécessite OpenCV (`cv2`), que le SDK n’installe pas ; `False` génère des octets JPEG bruts. Également accessible par projet sous le nom `ChlorosProject.stream(overlays=True)`. |

**Une exécution en cours conserve son projet.** Tant qu’une exécution de Processing est en cours — celle de ce script, de l’application, de `chloros-cli process` ou de `chloros-mcp` —, le backend refuse de créer, d’ouvrir, d’importer dans un projet ou de modifier les paramètres de traitement d’un projet (409 HTTP `PROCESSING_IN_PROGRESS`) : `create_project()` et `import_images()` déclenchent `ChlorosProcessingError`, et `configure()` déclenche `ChlorosConfigurationError`, tandis que `process()`, s’il est dans un mode différent de celui de l’exécution en cours, déclenche `ChlorosProcessingError` (une exécution conserve le mode dans lequel elle a démarré), chacune avec le message du backend. Une capture enregistrée dans le projet s’y ajoute alors à la fin de l’exécution (c’est ce qu’indique la réponse `project_add`). Attendez que l’exécution se termine (`get_status()`), puis continuez.**Un projet déplacé.** Le fichier `project.json` d’un projet nomme chaque fichier source (chaque image et chaque enregistrement du capteur de lumière) par son chemin d’accès complet ; ainsi, le déplacement du dossier du projet — ou de l’ensemble du dossier « projects » vers un autre disque — fait que ces fichiers continuent de faire référence à l’ancien emplacement. Chaque ouverture corrige ce qu’elle peut de manière autonome : un fichier qui se trouvait dans le dossier du projet est recherché dans le dossier où se trouve désormais le projet, et chaque chemin d’accès qui y est trouvé est redirigé et enregistré (`project.json` est d’abord sauvegardé sous le nom `project.json.before-relink-<date>-<time>`). Un chemin n’est jamais redirigé vers un fichier qui n’existe pas à cet emplacement. Les fichiers qui se trouvaient en dehors du dossier du projet — une session Hub, une importation via « Ajouter un dossier » — ne sont détectés qu’avec `relink=<the folder they are in now>` (ou tout dossier situé au-dessus). `open_project()` renvoie `source_files` : `total`, `found`, `missing`, `missing_roots` (les dossiers dans lesquels se trouvaient les fichiers manquants), `missing_outside_project` et `relink` (ce qui a été redirigé, d’où vers où). Un projet comportant quelques fichiers manquants s’ouvre avec le reste ; un projet où aucun fichier n’a été trouvé ne s’ouvre pas, et `open_project()` génère l'erreur `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit` ; HTTP 409 `PROJECT_SOURCES_MISSING`). Les codes `project open --relink` et `chloros-mcp` de l’CLI`open_project` (`relink_folder`) de l’interface CLI font de même.

À utiliser comme gestionnaire de contexte pour garantir le nettoyage :

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Enregistrements du capteur de lumière — étalonnés `.daq` + `.csv`

Un DAQ-U / DAQ-M / DAQ-E peut être enregistré **sans** son ensemble d’étalonnage. C’est
ce que font par défaut les enregistreurs publics [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) le font par défaut : ils enregistrent les valeurs brutes des capteurs et horodatent le
fichier afin que Chloros récupère l’étalonnage d’usine de ce capteur **par numéro de série** — d’abord dans le cache local, puis via le cloud MAPIR — et l’applique lors de l’importation.

Chloros réécrit le résultat sous la forme de deux produits par enregistrement, sous
`<project>/Light Sensor/` :

| Produit | Description |
| --- | --- |
| `<name>_calibrated.daq` | L’archive rétraitable — même schéma qu’un enregistrement en direct, déclarant désormais le bundle qui l’a produite. Sa réimportation **ne**provoque**pas** un deuxième étalonnage. |
| `<name>_calibrated.csv` | Irradiance spectrale en W/m²/nm sur la grille de longueurs d’onde propre au capteur, une ligne par lecture, plus des colonnes photométriques (puissance totale, lux photopiques/scotopiques, PPFD et sa répartition bleu/vert/rouge, longueur d’onde de pic). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Capteurs sans ensemble de données uniquement (DAQ-A).** Nombre brut de comptes spectraux du capteur — *et non* l’irradiance. Voir ci-dessous. |

`<name>` correspond au nom de fichier de l’enregistrement. Deux enregistrements situés dans des dossiers différents peuvent partager
une même valeur de mesure par capture (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) enregistrée à la
même milliseconde par deux caméras — et chacune obtient tout de même ses propres résultats : la première conserve
`<name>`, la suivante ajoute les dossiers qui la distinguent **après** le nom
(`<name>__<capture>_daq_calibrated.daq`), de sorte que la date et l’heure de l’enregistrement restent en tête.

`process()` effectue cette exportation comme l’une de ses étapes. Il ne nécessite **pas** d’images :
un capteur de lumière utilisé seul constitue un flux de travail à part entière, et un tel projet ne comporte aucune
image par définition.

**Les enregistrements DAQ-A s’exportent sous forme de comptes bruts.** La gamme DAQ-A est antérieure au système de
bundles par numéro de série et ne dispose d’aucun bundle à récupérer — elle est plutôt étalonnée sur le terrain par rapport à une
cible de réflectance, ce qui explique pourquoi elle n’en a jamais eu besoin. Ces enregistrements s’exportent
sous une racine `_raw` plutôt que `_calibrated` : il s’agit d’un nom de fichier différent plutôt que d’un indicateur
à l’intérieur du fichier, car l’information doit rester lisible lorsqu’elle est transmise par e-mail sous forme de nom nu. L’
en-tête `.csv` indique `raw spectral sensor counts (NOT irradiance)` et signale que les
valeurs sont comparables **au sein** du fichier — ce qui correspond exactement à l’objectif de l’étalonnage par cible
— et non entre différents capteurs. Les colonnes photométriques dépendantes de la puissance (puissance totale,
lux photopiques/scotopiques, PPFD) renvoient **NULL** au lieu d’être calculées à partir des comptages intégrés.

Un DAQ-U / DAQ-M / DAQ-E dont le bundle n’a tout simplement pas pu être récupéré est tout de même **ignoré**,
et non écrit en format brut : dans ce cas, le bundle existe bel et bien et « se reconnecter et retraiter » est un conseil avisé.

Les enregistrements hérités **v1.01 / v1.02** (générés par un DAQ-A-SD) ne comportent pas d’époque par lecture,
mais uniquement l’heure d’écriture du fichier. Le module de correspondance image↔flux descendant les rejette toujours — la mise en correspondance d’une
trame à une heure d’écriture serait une erreur invisible — mais l’exportateur les lit, et le
fichier CSV affiche `clock=daq_created_on`, ce qui permet au produit d’indiquer de quelle horloge il s’agit.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Un enregistrement dont le fichier d’étalonnage ne peut être récupéré (hors ligne, ou capteur sans
étalonnage enregistré) est signalé sous `skipped` **avec la raison**. Il n’est jamais
enregistré en tant que fichier « étalonné » contenant des comptages bruts — connectez-vous à Internet et
relancez l’opération, et l’exportation s’achèvera.

### Rappel de progression

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Résumé et conseils après l’exécution

Une fois l’opération terminée, `process()` récupère `GET /api/processing-summary` et joint le corps en tant que `result["summary"]`. La récupération s’effectue au mieux et ne bloque jamais un retour réussi — si le résumé n’est pas disponible, `process()` revient à la forme simple `{"status": "complete", "async": False}`. Chaque entrée de `summary["hints"]` — des phrases complètes proposant des mesures correctives, par exemple pourquoi une exécution n’a produit aucun résultat — est également réémise sous forme de Python `UserWarning`, de sorte que les exécutions sans résultat s’autodiagnostiquent même si vous n’inspectez jamais le dictionnaire :

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` correspond à la partie lisible par machine :

| Clé | Ce qu’elle comptabilise |
| --- | --- |
| `models` | Groupes de caméras dans l’exécution. |
| `images_in_groups` | Images sources de ces groupes. |
| `targets_found` | Cibles de réflectance détectées. |
| `images_calibrated` | Images pour lesquelles la session a généré un produit de réflectance. |
| `images_failed` | Images ayant échoué et n’ayant pas été exportées. |
| `exported_files` | **Fichiers de produits d’image générés par la session** : tous les produits (débayérisation, aperçu, radiance, réflectance, indice, rendus de cibles), dans tous les formats. |
| `reflectance_files` | Parmi ces produits de réflectance (`Reflectance_Calibrated_Images`), qu’ils aient été calibrés par une cible ou par l’irradiance du capteur de lumière (DAQ). Une exécution sans cible qui a tout de même généré des données de réflectance l’indique dans une remarque ; seule une exécution n’ayant généré aucune donnée de réflectance signale que ses sorties de réflectance sont manquantes. |
| `clamp_masks_written` | Les [masques de limitation de décomposition](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) indiqués à côté d’eux — ce sont des compagnons, pas des produits, ils ne figurent donc pas dans `exported_files` (l’CLI les comptabilise séparément de la même manière). |
| `captures_left_out` | Images brutes dans le dossier `captures` du projet que cette exécution n’a pas capturées. Une exécution attend, dès son démarrage, les données capturées précédemment ; ce nombre est donc égal à 0, sauf si une capture a été enregistrée pendant l’exécution ou si le délai d’attente a expiré. `summary["captures_left_out_note"]` indique comment les traiter. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Enregistrements du capteur de lumière, comptabilisés séparément à dessein — ils proviennent d’une étape différente et existent pour des exécutions ne comportant aucune image ; les inclure donnerait l’impression qu’une exécution « DAQ uniquement » a exporté des images. |

À côté de celles-ci : `summary["output_dirs"]` (chaque répertoire dans lequel des données ont été écrites),
`summary["light_sensor_export"]`, `summary["stopped"]` (vrai lorsque l’utilisateur a interrompu la
série, afin que les comptages partiels ne soient pas interprétés comme une exécution terminée ayant produit un résultat insuffisant), et
`summary["groups"]` (la ventilation par groupe).

`exported_files` est enregistré par le pipeline **au fur et à mesure de l’écriture**, et non pas extrait a posteriori des
objets image du projet. Les stratégies parallèles et GPU créent leurs propres objets image
(dans des sous-processus de travail pour les chemins GPU) ; ainsi, l’ancienne analyse signalait
`0 file(s) written` pour chaque exécution de ce type, puis émettait l’indication « zéro exportation » — sur des exécutions
où tout avait pourtant fonctionné. Si vous créez un script en fonction de ce nombre, une exécution parallèle réussie
affiche désormais un compteur non nul.

Les rapports d’omissions du capteur de lumière indiquent la raison réellement établie par le lecteur pour chaque fichier — un
schéma illisible, un bundle manquant, une erreur d’écriture — **dédupliquée** ; ainsi, vingt fichiers
ignorés pour une même cause sont comptabilisés comme une seule cause plutôt que comme vingt répétitions de celle-ci.

> **`process()` ne se déclenche pas lorsqu’une exécution ne produit aucune image.** C’est l’un des deux points sur lesquels le SDK et
> la CLI divergent délibérément : `chloros-cli process` traite « des produits ont été demandés, aucun n’a été
> écrit » comme un échec et se termine avec un code de sortie non nul, tandis que le SDK revient normalement et signale la
> cette condition via `summary` / hints. Si votre pipeline doit s’arrêter en cas d’exécution vide, vérifiez-le
> vous-même — inspectez `summary` (ou comptez les fichiers dans le dossier du projet) plutôt que de vous fier à
> l’absence d’exception. Les causes habituelles sont un dossier d’entrée qui n’a pas été reconnu comme une
> capture et des produits ignorés car inapplicables aux caméras présentes (par exemple, la radiance provenant de caméras RGB uniquement
>).
>
> `process()` ne se déclenche pas non plus lorsque les exportations d’une exécution n’ont pas pu être vérifiées : il affiche un avertissement et définit
> `result["export_check_failure"]`. Les produits sont enregistrés. Pour interrompre le pipeline à ce stade, vérifiez cette clé. Il en va de même pour une exécution qui
> a perdu une image : elle affiche un avertissement et active `result["image_failure"]`, ce qui fait que `chloros-cli process`
> renvoie une valeur non nulle à la sortie.

### Fonctions pratiques

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Chaque exécution s’effectue au sein de son propre `with ChlorosLocal(...)` ; un backend qu’elle a dû lancer continue de s’exécuter lorsqu’elle revient, pour l’appel suivant, et s’arrête automatiquement au bout de 30 minutes si personne ne l’utilise. Aucune d’entre elles n’accepte les paramètres `auto_start_backend`, `radiometric_output` ou `daq_log_path` ; utilisez `ChlorosLocal` et `configure()` pour ces derniers. `process_lattice_capture()` n’accepte ni `camera`, ni `debayer`, ni `ppk` : il ignore la détection des panneaux et utilise le débayeur standard.

### Valeurs prises en charge

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Sortie radiométrique (pipeline multispectral LATTICE)

Le niveau d'exportation multispectral LATTICE (M3C/M3M) du pipeline `process` — `reflectance` (par défaut), `radiance`, `sensor-response` ou `all` (chaque mode applicable par image) — définit les **« Exportation de la radiance »**et**« Exportation de la réflectance »**** : `radiance` exporte uniquement la radiance, `reflectance` uniquement la réflectance, `sensor-response` aucune des deux (uniquement les produits débayérisés et d’aperçu), `all` les deux. Les produits débayérisés et de prévisualisation ne peuvent pas être modifiés au niveau de ce paramètre. `configure()` dispose d’un mot-clé dédié à cet effet :

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

La solution de secours avancée — l'écriture des produits via `custom_settings` — fonctionne toujours, mais n'oubliez pas qu'elle remplace l'ensemble du bloc de paramètres (voir l’avertissement ci-dessous). Écrivez les clés `"Export <product>"` : les paramètres propres à un projet `"Export radiance"` / `"Export reflectance"` d’un projet l’emportent sur l’ancienne clé `"Radiometric output"` ; cette dernière, à elle seule, ne modifie donc rien.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (par défaut) divise la radiance de la caméra par le **flux descendant DAQ correspondant à l’horodatage**, déterminé automatiquement à partir d’un fichier `.daq` (DAQ-U/M/E) enregistré**ou d’un fichier `.csv`**natif DAQ-M** trouvé avec l’imagerie ; tout ensemble d’étalonnage par caméra ou par DAQ manquant localement est**récupéré automatiquement depuis AWS** lors de la première utilisation. L’CLI expose cette fonctionnalité conformément àcommutateurs de type produit sur `chloros-cli process` : `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**À qui appartient cette lecture ?**Une lecture enregistrée par une capture `.daq` correspond à la lecture de la ou des caméras qu’elle décrit — nommées dans un fichier sidecar `<file>.daq.cameras.json` qui l’accompagne (captures uniques, synchronisées et en tableau ; une rafale brute nomme la lecture de chaque élément dans son manifeste) — et n’est associée**qu’aux images provenant de ces caméras**. Une caméra autonome sans lecture propre n’emprunte donc plus le DAQ d’une autre caméra ; liez-lui un DAQ, ou enregistrez-en un. L’enregistrement propre à un DAQ (`chloros-cli daq pool-record`, ou un enregistrement lancé depuis l’interface graphique), ou tout fichier `.daq` dépourvu de ce fichier d’accompagnement (y compris les anciennes captures), continue de fonctionner avec toutes les caméras.**Caméras OCN exposées à la lumière du soleil.** Chaque pack d’étalonnage OCN est généré sous un éclairage à bande étroite, avec une LED par bande, alors que la lumière du soleil mélange les bandes. Chloros 1.3.0 applique une correction globale basée sur la lumière du soleil après le démixage des bandes, ce qui améliore les produits de radiance, de réflectance et d’indice OCN en plein soleil. Elle s’applique partout où la séparation des bandes a lieu — `process()`, les captures, `grab()` / `frame_stream()` et la vue en direct — pour les packs OCN qui la prennent en charge ; les caméras RGN restent inchangées. Les images OCN capturées avec une version antérieure en bénéficient également lorsqu’elles sont traitées sur un ordinateur conservant l’étalonnage utilisé lors de leur prise de vue (Chloros conserve l’étalonnage d’une caméra une fois celle-ci connectée) ; dans le cas contraire, elles sont traitées comme auparavant. Les valeurs de radiance, de réflectance et d’indice OCN diffèrent donc de celles de la version 1.2.0.

> `custom_settings` **remplace** l’intégralité du bloc de paramètres calculés (il contourne, de par sa conception, les autres mots-clés et la validation de `configure()`). Lorsque vous l’utilisez, incluez toutes les clés `Project Settings` qui vous intéressent, comme dans l’exemple ci-dessus.

#### Lecture des pixels d’indice (l’NDVI et les autres indices de végétation)

**Un raster d’indices entier correspond à un étirement du contraste par image, et non à une carte de valeurs comprises entre −1 et 1.** Les produits d’indices `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` et `JPG (8-bit)` — y compris le fichier TIFF 8 bits dans lequel une capture 8 bits est enregistrée — stockent

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

où `IndexMin` / `IndexMax` sont **les valeurs d’index finies la plus basse et la plus haute propres à cette image**. DN 0 n’est pas −1, et la pleine échelle n’est pas +1.

Le Chloros appose des timbres aux deux extrémités afin que la correspondance reste inversible :

| Balise | Signification |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — la convention ci-dessous |
| `Chloros:IndexMin` | la valeur d’index stockée en tant que DN 0 |
| `Chloros:IndexMax` | la valeur d’index stockée comme échelle complète |

Lisez-les et inversez-les, en normalisant d'abord le DN stocké entre 0 et 1 par rapport à la valeur maximale du type de données (dtype) stocké, afin qu'une seule expression couvre tous les formats d'entiers :

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

La lecture ne nécessite pas `-config pix4d.config` — exiftool lit ces propriétés XMP sans aide. La configuration n’est requise que pour les *écrire*.

> **`IndexMin == IndexMax`** signifie que chaque pixel de cette image contenait la même valeur d’index. La trame est remplie à mi-échelle et il n’y a rien par quoi diviser — prenez la valeur extrême.

> **L’absence de balises `Chloros:Index*` signifie qu’aucun mappage n’est spécifié.** Une exportation LUT est un rendu colorisé dont les pixels sont des couleurs en dégradé indexées sur la fenêtre de seuil de votre LUT, et une image ne comportant aucun pixel d’indice fini n’a pas de plage à indiquer. Ne présumez pas d’un mappage dans l’un ou l’autre cas — la même règle s’applique qu’en cas d’absence de balise `PixelScale`.**Comme l’étirement s’applique à chaque image, deux images d’un même vol ne sont pas comparables en termes de DN**— chacune possède ses propres valeurs extrêmes ; ainsi, un même indice NDVI correspond à des DN différents selon les images. Pour les analyses quantitatives portant sur l’ensemble d’un vol (statistiques zonales, seuils fixes, séries chronologiques), exportez**`TIFF (32-bit, Percent)`**: les rasters d’indice y sont enregistrés sous forme de**valeurs d’indice elles-mêmes** de type float32, sans rééchelonnage ni balises nécessaires. Malgré le nom du format, les produits d’indice ne sont pas multipliés par 100 — cet échelle ne s’applique qu’à la réflectance de type entier.

#### Masque de limitation du démixage — pixels sur lesquels une bande a été limitée

Le démixage spectral 3×3 peut rendre une bande négative — lorsque la lumière contient à peine cette bande, ou lorsque la scène se situe en dehors de ce que décrit la matrice de démixage de la caméra. Comme il ne s’agit pas d’une valeur réelle, l’exportation la fixe à 0 — et la vue en direct affiche ce même 0. Une bande peut être écrêtée sur quelques pixels ou sur la quasi-totalité d’entre eux ; le masque enregistre l’une ou l’autre situation. `Chloros:Band{R,G,B}UnmixClampFraction` indique la proportion de l’image concernée ; le masque d’écrêtage précise **quels pixels** :

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— dans un dossier `clamp_masks/`**à côté** du dossier du produit, jamais à l’intérieur de celui-ci (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — pour chaque produit de radiance, de réflectance et d’indice dans lequel au moins un pixel a été écrêté. `uint8` monobande, sur la grille de pixels propre au produit (orientation, zone d’intérêt, regroupement de pixels et correction d’alignement du tableau inclus). Chaque masque porte le nom de la caméra de son produit (`Make` / `Model` / numéro de série), l’heure de capture et les balises d’objectif, et répertorie les bandes que ses bits représentent (`XMP-Camera:BandName`, bit 0 en premier) ; ses bits `ImageDescription` et `Software` l’identifient comme un masque de limitation, et il ne comporte aucun étalonnage.
- **Bits** : bit 0 = la bande du canal R du produit (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Un bit activé signifie que la valeur du pixel dans cette bande a été limitée à 0 — considérez-le, ainsi que tout indice dérivé de celui-ci, comme peu fiable.
- **`Chloros:UnmixClampMask`** est apposé sur chaque produit ayant fait l’objet d’un « unmix » : le chemin d’accès au masque par rapport au dossier du produit (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, toujours séparé par `/`-séparé par un espace — ajoutez-le au dossier de l’image pour ouvrir le masque), ou `none` lorsqu’aucun pixel n’a été limité. L’absence de balise signifie que le produit n’a jamais subi de démixage (caméras RGB ou monochromes, niveaux bruts/débayérisés).
- Les résultats de la capture le contiennent sous la forme **`clamp_mask`** de l’image (le chemin d’accès au masque, ou `None` lorsqu’aucun pixel n’a été écrêté) — dans les résultats `CameraSession.capture()`, les images `ArraySession.capture()`dans les images `CaptureResult` et les réponses de capture HTTP. La version hors ligne répertorie ses masques sous la forme `clamp_masks`.
- La pile multibande d’un index en direct (`/api/camera/cli-array-index` avec `save_multiband`) reçoit un masque par caméra source, dans le dossier de sortie `clamp_masks/` — voir la référence CLI`lattice index`. Une vidéo ne comporte pas de masque : le résultat de la compilation hors ligne indique à la place la fraction de clamp de chaque image.

Comme aucun dossier de produit ne contient de masque, à quelque niveau que ce soit, un dossier de produit peut être transmis tel quel à un logiciel de photogrammétrie — sous-dossiers compris — et celui-ci ne prend en compte que les images. Transmettez-lui le dossier de produit, et non le dossier situé au-dessus. La fonction d’importation d’Chloros ignore les masques.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect pour les caméras LATTICE

Sessions backend persistantes pour le matériel en temps réel. Mêmes points de terminaison que ceux utilisés par l’interface graphique, ce qui garantit un comportement identique entre le SDK, la CLI et l’interface graphique.

### Caméra unique — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

L’exposition, le gain et la cible définis ici sont conservés tels quels : le contrôle automatique de l’exposition de Chloros (y compris l’exposition mesurée par le DAQ) ne les modifie pas. `cam.set_settings(exposure_auto="Continuous")` (ou `gain_auto=`) restitue l’exposition (gain), et `cam.set_settings(target_brightness_auto=True)` restitue la cible au système de ciblage automatique d’Chloros.

#### Signature `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Il revient une fois le flux lancé.** Le backend attend, dans une plage limitée (environ 15 s pour que le flux démarre, puis 10 s pour son premier tampon complet — et non pour la première image d'aperçu, qui suit environ 1,5 s plus tard), donc un `status()` apparaît juste après celui qui indique `streaming`. Le `connect_stream` de la session correspond au bloc `stream` de la réponse : `state` `streaming`, `no_frames` (le flux a démarré mais n’a rien transmis — c’est ce qu’indique le bloc `note` et, pour une caméra utilisant des paquets jumbo, de se reconnecter avec une taille de paquet de 1 500 octets), `starting` ou `start_failed`, avec `waited_s`. `None` pour une caméra déjà connectée. Un aperçu (`get_camera_preview`, `/api/camera/jpeg/<serial>`) demandée juste après un délai d’attente limité (4 s) pour la première image de prévisualisation de la caméra, au lieu de répondre « Aucune image disponible pour l'instant » ; une caméra dont le flux n'a encore rien transmis (`no_frames`, ou activée pour un déclencheur matériel) répond toujours immédiatement.
- **La taille des paquets est à définir par vos soins.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` ouvre la caméra avec des paquets de 1 500 octets, sans test préalable (`chloros-cli lattice cam-connect --packet-size standard` envoie la même chose).
- **Une seule connexion à la fois est autorisée dans le pool de caméras** : l’ouverture, la vérification de la taille des paquets, un premier téléchargement de calibrage (environ 70 s pour une caméra nouvellement connectée à cet ordinateur), le démarrage du flux. Une requête nécessitant une caméra entre-temps (`read_nodes`, `set_settings`, une capture, une deuxième connexion) l’attend au maximum 5 s, puis échoue avec un code HTTP 503 `CAMERA_POOL_BUSY`, en indiquant le nom de la caméra qui se connecte (`ChlorosConnectError`) ; demande à nouveau dans un instant. Les listes n’attendent jamais : `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` et `CameraSession.status()` répondent immédiatement. Une caméra en cours de connexion est répertoriée sous la réponse du pool `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — et `elapsed_s`), et son `status()` indique `connecting: True` avec son `phase` ; les autres caméras sont lues en temps réel lorsque le pool est libre, sinon la réponse provient de leur dernière lecture (`live: False`).

#### Méthodes `CameraSession`

| Méthode | Description |
| --- | --- |
| `status(timeout=15.0)` | L'état en direct de la caméra, tel que lu par `chloros-cli lattice status` : `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` ou `stopped`), `device` (modèle, numéro de série, micrologiciel, température, `is_mono`) et ses paramètres actuels. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lit les nœuds GenICam ; renvoie `{nodes, errors, enums, device, was_in_pool, in_array_id}`, ainsi que `frame_values` et `notes` pendant l'exécution de l'exposition automatique (voir ci-dessous). |
| `set_settings(**kwargs)` | Écrit dans les nœuds par nom convivial (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Un `exposure_time` (`gain`) correspond à une exposition manuelle (gain) : écrit sans `exposure_auto` (`gain_auto`), il faut d’abord désactiver le mode automatique afin que la valeur ne soit pas rejetée par un mode automatique en cours d’exécution. Le résultat, `applied`, conserve la valeur enregistrée par la caméra ; une écriture refusée par la caméra se trouve dans `errors` et n’est pas conservée. L’erreur indique la plage que la caméra accepte désormais (une valeur maximale `ExposureTime` dépend de la fréquence d’images de la caméra), et un mode automatique qui avait été désactivé uniquement pour la valeur refusée est réactivé ; `restored` le répertorie (par ex. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Capture d’une **seule** image. Renvoie un `CaptureResult` : une liste de dictionnaires de métadonnées d’images (un par niveau enregistré), avec `.skipped`, `.failed`, `.daq_files` et `.daq_matches`. (La capture en rafale/multi-images a été supprimée — appelez `capture()` en boucle si vous avez besoin d’une série.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Captures consécutives (`interval_s=0`) ou à intervalle régulier à partir de cette caméra, pour les captures `count` et/ou `duration_s` (au moins une, pour que la procédure se termine), comme le fait `ArraySession.capture_repeated()` pour un tableau. Renvoie la liste des `CaptureResult` ; l'opération s'arrête après une capture signalant un disque plein. `on_capture(i, result)` s'exécute après chacune d'elles ; les autres mots-clés sont redirigés vers `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Libération du pool, la caméra est rétablie telle qu’elle a été trouvée lors de la connexion (câblage du déclencheur, PTP, paramètres); `keep_settings=True` enregistre plutôt ses paramètres dans sa propre mémoire, `clear_settings=True` réinitialise cette mémoire (voir [Mémoire de la caméra](#camera-memory--keep_settings--clear_settings)) ; renvoie `camera_memory` lorsque l’une ou l’autre de ces opérations est demandée. Une caméra membre d’un tableau libère l’intégralité du tableau. Opération sans effet si nous nous sommes connectés à une session déjà ouverte. Renvoie une valeur dès que le backend confirme la libération, ou lorsqu’il n’y a rien à libérer (la caméra a déjà disparu ou aucun backend n’est en cours d’exécution); une libération ayant expiré ou refusée par le backend génère l’erreur `ChlorosConnectError`, car la caméra peut encore être occupée et la connexion suivante entrerait en conflit avec elle. La sortie du bloc `with` ne génère jamais d’erreur. |

Attributs : `serial`, `device` (l'enregistrement de découverte de la caméra) et `backend_url`.

Contrôles d’exportation `capture()` (même modèle que le tableau + interface graphique) :

- `processing` / `levels` — `processing="all"` enregistre tous les types d’exportation applicables ; `levels=["raw","radiance"]` n’enregistre que ceux-ci (remplace `processing`). Omettez les deux pour utiliser la valeur par défaut du backend.
- `daq` — la lecture DAQ de la capture utilisée est enregistrée par défaut sous forme de fichier sidecar `.daq`, incluant une capture brute uniquement, afin que l’image puisse être retraité ultérieurement en réflectance/indice. `daq=False` l'ignore lorsqu'aucun niveau ne l'utilise. `force_daq=True` est l'ancienne orthographe de la valeur par défaut. Pas de DAQ, pas de `.daq`.
- `daq_tolerance_ms` — écart maximal par rapport à l’exposition pour que la lecture reste dans les tolérances (valeur par défaut : 500). Seul le verdict change.

#### `read_nodes()` en exposition automatique — `frame_values` et `notes`

Pendant l’exécution de `ExposureAuto` (`GainAuto`), le nœud `ExposureTime` (`Gain`) relit la **dernière valeur écrite**, et non ce que fait la caméra à ce moment-là. `nodes` conserve toujours cette valeur de nœud. Lorsque vous interrogez `ExposureTime` ou `Gain` sur une caméra connectée dont le mode automatique n’est pas `Off`, la réponse ajoute :

- `frame_values` — la valeur contenue dans la dernière image de la caméra, sous le nom du nœud (`ExposureTime` en µs, `Gain` en dB) ;
- `notes` — une phrase par nœud de ce type indiquant les deux valeurs.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Les deux clés sont absentes lorsque le mode automatique est `Off` (le nœud correspond alors à la) et pour une caméra qui n’est pas connectée au backend. Pour un tableau, `ArraySession.status()["member_ae"]` fournit simultanément les valeurs d’image de chaque élément.

#### Plafond d’exposition mesuré par DAQ (HTTP)

Lorsqu’un capteur de lumière DAQ est connecté, Chloros limite l’exposition × le gain d’une caméra en fonction de la lumière du jour mesurée, afin qu’un panneau de référence blanc ne soit pas écrêté. Ce plafond ne s’applique que lorsqu’une cible d’étalonnage se trouve dans le champ de vision de cette caméra (apparue dans son cadre au cours des 10 dernières secondes) ; en l’absence de cible dans le champ de vision, c’est l’exposition automatique habituelle de la caméra, avec sa propre protection contre les surexpositions, qui prend le relais, afin que le reste de la scène ne soit pas assombri plus que nécessaire. Il n’existe pas d’enveloppe SDK ; configurez ou modifiez ce paramètre par caméra à l’aide des codes `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (le corps de la requête POST `{"enabled": false}` désactive cette fonctionnalité pour cette caméra). La réponse contient les codes `enabled`, `cap_us` (la limite maximale en temps réel, `None` lorsqu’elle est inactive, y compris lorsqu’aucune cible d’étalonnage n’est visible), `k_exp_g` (la constante utilisée pour calculer le plafond) et :

| Champ | Signification |
| --- | --- |
| `k_exp_g_source` | Constante utilisée : `camera` — celle de cette caméra, calculée à partir du point de laboratoire de son pack d’étalonnage (caméras RGB) ; `fleet` — la constante de la flotte, utilisée lorsque le pack ne comporte pas de point de référence ; `override` — une valeur de remplacement définie au niveau du backend. |
| `k_exp_g_fleet` | La constante de la flotte, à titre de comparaison. |

### Matrice synchronisée — `ArraySession` (Smart-Prep)

`connect_array` est **le point d’entrée recommandé** pour les configurations à plusieurs caméras. Il exécute en arrière-plan l’intégralité du flux Smart-Prep via l’interface graphique :

1. **Analyse du réseau**(`/api/camera/array/recommend`) — configure le**capteur complet** (pas de binning, pas de recadrage de la zone d’intérêt) et laisse la fréquence d’images s’adapter à ce que l’hôte et le réseau peuvent fournir, en fonction des formats de pixels et du mode de compression que Connect utilisera. Les images ne sont jamais réduites pour maintenir la fréquence : utilisez un `width` / `height` ou `binning=2` / `4` pour privilégier la fréquence d’images au détriment de la résolution (le point de terminaison`alternatives` répertorie ces options avec leurs fps prévus).
2. **Sélection automatique du niveau** — `sim-capture-sim-emit` (toutes les caméras exposent au même déclenchement, l’émission est cadencée pour partager la ligne, la fréquence d’images diminuant à mesure que le partage se réduit) sur tout réseau où les caméras peuvent être cadencées pour partager la ligne ; `slip-emit-and-capture` (les caméras se relaient sur la ligne, sans exposition simultanée) uniquement lorsqu’elles ne le peuvent pas.
3. **PTP activé**par défaut — les horodatages entre caméras s’alignent sur une horloge partagée à**~1 ms**près, le [maître PTP](#ptp-grandmaster--time_sync_status) du backend. L’exposition simultanée est assurée par le déclencheur matériel M8 (**< 100 µs** entre les modules), et non par le PTP : le PTP aligne les *horodatages*, pas les expositions.
4. **Sélection automatique du format de pixel par caméra** — mono → `Mono12`, couleur → `BayerRG12`.
5. **Initialisation de l'exposition automatique (AE)** — enregistrement de l'état actuel de l'AE de chaque caméra afin que la connexion ne réinitialise pas l'exposition en cours de prise de vue.
6. **Configuration du déclenchement GPIO** — `connect_array` arme toutes les caméras (`TriggerMode=On`, `TriggerSource=Line2`) afin que l’impulsion du maîtrepuisse piloter les esclaves via le câble M8. Cette étape ne concerne que les réseaux de caméras : une caméra isolée, activée avec `LatticeCamera`, fonctionne alors en mode libre. La désactivation du réseau (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) ramène toutes les caméras en mode libre (`TriggerMode=Off`), tout le reste restant tel que détecté lors de la connexion : les réglages du déclencheur et de la ligne de synchronisation (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource` ; la ligne de synchronisation de la caméra maître réinitialisée en entrée), PTP, ainsi que les réglages de la caméra modifiés au cours de la session — sauf si `keep_settings=True` / `clear_settings=True` (voir [Mémoire de la caméra](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attributs : `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` et `packed_reason` (les quatre derniers correspondent à `None` sur un descripteur provenant de `attach_array`), `connect_stream` et `backend_url`. `connect_stream` correspond au bloc `stream` de la réponse à la connexion, la forme de la connexion unique plus `members` (`{serial: {state, why?}}`) : `streaming` une fois que le premier groupe synchronisé du tableau est arrivé (le backend l'attend, dans une limite d'environ 10 s), ou `no_frames` avec l’échec de récupération le plus récent de chaque membre comme `why` et, pour les paquets jumbo, la valeur de repli de 1 500 octets dans l’`note`. `None` sur un descripteur `attach_array`.

#### Signature `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` correspond à la durée pendant laquelle l’appel attend la connexion ; cette valeur est envoyée au
backend en tant que délai limite. Si la connexion n’est pas établie à ce moment-là, l’opération est annulée : chaque caméra
ouverte est rétablie dans l’état où elle a été trouvée et fermée, aucun tableau n’est connecté, et l’
appel génère une erreur `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, indiquant l’
étape atteinte par la connexion). Ainsi, un appel qui a levé une exception ne laisse jamais de réseau
connecté derrière elle. Un réseau de 9 caméras se connecte en environ une minute ; juste après
la mise sous tension des caméras, cela peut prendre plusieurs minutes. Un réseau qui est opérationnel mais qui ne dispose pas encore d’un
groupe d’images synchronisées est renvoyé, avec le code `connect_stream["state"]`
`no_frames`. Le journal du backend comporte une ligne de chronométrage par étape de connexion
(`[array-connect <id>] <step>: <s> s …`) et une dernière ligne indiquant l’étape la plus lente.

Valeurs `force_tier` :
- `"sim-capture-sim-emit"` — véritable simultanéité (toutes les caméras se déclenchent sur le même front d’horloge).
- `"sim-capture-ftd-stagger"` — décalage flexible dans le domaine temporel (les caméras émettent à des instants légèrement décalés afin que les paquets soient sérialisés sur le câble).
- `"slip-emit-and-capture"` — capture séquentielle par caméra (pas de synchronisation temporelle). Sélectionnée automatiquement uniquement lorsque les caméras ne peuvent pas être ralenties suffisamment pour partager le bus simultanément.

`wire_ceiling_mbps` remplace le **budget de bande passante soutenu de l’hôte** en Mo/s — le seul
chiffre dont dépend l’allocation de l’ensemble du réseau. Laissez la valeur `None` pour utiliser la valeur détectée automatiquement. Réduisez-la lorsque le réseau signale des trames GVSP corrompues : la valeur automatique est dérivée
du débit de liaison annoncé par la carte réseau, ce qui surestime les adaptateurs USB, les voies PCIe étroites et
les structures partagées très sollicitées — et cette surestimation se traduit par des trames corrompues plutôt que par une
liaison visiblement lente. La valeur est conservée dans le bloc de capture de la matrice du projet ; ainsi, une
réouverture ou un `connect_array` ultérieur la rétablit comme n’importe quel autre paramètre de la matrice.
Voir [État de santé de la matrice](#array-health--which-subsystem-is-losing-frames).

#### Transport compressé 12 bits (`packed=`)

Lorsque tous les capteurs sont actifs, la fréquence d’images d’un réseau est déterminée par la vitesse à laquelle chaque caméra peut envoyer
ses images. Les formats 12 bits (`BayerRG12`, `Mono12`) sont généralement transmis sous forme de mots de 16 bits ;
le transport compressé envoie les mêmes 12 bits en 1,5 octet par pixel et
les décompresse sur l’hôte, ce qui réduit de 25 % le temps d’envoi de chaque image — soit environ
**33 % d’images par seconde en plus à pleine capacité du capteur** (mesure : 10,7 → 14,0 images par seconde sur un
réseau de 9 caméras). **Les données enregistrées sont identiques** : mêmes valeurs, mêmes fichiers 16 bits,
mêmes métadonnées.

- `packed="auto"` (par défaut) compresse uniquement lorsque cela est utile **et** que l’hôte peut
  se permettre le décompressage : à la connexion, le backend mesure sa propre vitesse de décompressage,
  le nombre de cœurs CPU, la charge actuelle et le budget réseau. Un ordinateur de bureau disposant de cœurs inutilisés
  compresse à pleine capacité du capteur ; un hôte occupé ou à faible consommation (ordinateurs monocarte,
  contrôleurs embarqués) diffuse les données non compressées. Les petites zones d’intérêt (ROI) et les images regroupées n’apportent que peu de gains.
- `packed="off"` (ou `False`) ne compresse jamais — pour écarter cette option lors du diagnostic,
  ou sur un hôte qui doit réserver son processeur à d’autres tâches.
- `packed="on"` (ou `True`) compresse toujours les formats 12 bits.
- Toute autre valeur déclenche `ValueError`. Les formats 8 bits et 16 bits ne sont jamais
  compressés.

`analyze_array_network(..., packed="auto")` prend la même décision,
son fps correspond donc à la fréquence à laquelle le tableau s’exécutera (la réponse contient
`packed_transport` et `packed_reason`) ; sa valeur par défaut `pixel_format` correspond à celle
appliquée par « connect » (Mono12 / BayerRG12) plutôt qu’à 8 bits.

La décision et sa justification figurent dans la session :

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` applique la même dérogation à tous les
tableaux d’un projet (valeur par défaut `None`, ce qui signifie `"auto"` : un projet n’enregistre aucun
paramètre de compression par tableau).
Pour une valeur par défaut à l’échelle de la machine, définissez `CHLOROS_ARRAY_PACKED12=off` (ou `on`) dans l’
environnement du backend ; une valeur explicite `packed="on"` / `"off"` prévaut toujours.
`/api/camera/array/status` signale `packed_transport` et `packed_decision`.

#### Fréquences d’images attendues

Fréquences d’images mesurées pour le réseau de capteurs (synchronisation matérielle, 12 bits ; capteur complet = 2048 × 1536). Utilisez-les pour vérifier la cohérence de `status()["actual_fps"]` ; `analyze_array_network()` projette le débit pour votre propre hôte et votre réseau.

| Configuration | Fréquence d'images du réseau de caméras |
| --- | --- |
| Capteur complet, carte réseau hôte PCIe 10 GbE, transport compressé (`packed="auto"` sur un ordinateur de bureau) | ~14 images par seconde |
| Capteur complet, carte réseau hôte PCIe 10 GbE, transport non compressé (`packed="off"`) | ~10,5 images par seconde |
| Capteur complet, 8 caméras sur un adaptateur USB 10 GbE | ~3,8 images par seconde |
| Capteur complet, liaison hôte 1 GbE | 1,2–1,7 image par seconde |
| Binning 2 × 2 (`binning=2`), carte réseau hôte 10 GbE | ~30–33 images par seconde |

- Sur une liaison PCIe 10 GbE, ce sont les **caméras** qui constituent la limite en mode « capteur complet », et non le réseau : 4 et 8 caméras fonctionnent à la même cadence.
- **`target_fps` correspond à la cadence de déclenchement, et le réseau de caméras ne fonctionne jamais plus vite.** `target_fps=1` fait fonctionner le réseau de caméras à 1 image par seconde ; `None` (la valeur par défaut) le fait fonctionner à la fréquence maximale prise en charge par l’hôte et le réseau. Toute fréquence supérieure à cette valeur est ramenée à celle-ci (voir [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` de `4294` dans `status()` n’est pas un débit de liaison.** Cela signifie que Windows n’a pas pu lire la vitesse de la carte (le pilote a renvoyé la plus grande valeur 32 bits), ce qui est courant avec les cartes Ethernet USB. Le débit utilisable d’une carte USB sous Windows est plafonné à 200 Mo/s dans tous les cas (voir [État de l’array](#array-health--which-subsystem-is-losing-frames)) ; pour en optimiser un, consultez la section « Adaptateurs USB 10 GbE » de la référence CLI sous [Configuration et optimisation de la carte réseau hôte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Une limite USB s’applique également sous Linux : 400 Mo/s sur x86_64, 200 Mo/s sur NVIDIA Jetson.** La carte est identifiée via sysfs (en repli sur `ethtool -i`) ; ainsi, une carte USB 5 GbE ou 10 GbE est limitée à cette valeur maximale plutôt qu’à ses performances nominales de 500 / ~1 060 Mo/s. Le chiffre pour x86_64 a été mesuré sous Linux : la même carte, qui corrompt les trames au-delà de 200 Mo/s sous Windows, a permis de faire fonctionner sans problème un réseau de 4 caméras à toutes les limites jusqu’à 480 Mo/s. Sur NVIDIA Jetson, ce même adaptateur n’a fonctionné sans erreur qu’à 200 Mo/s et a perdu des trames de manière intermittente au-delà de ce seuil. Si votre hôte absorbe davantage de trames sans `gvsp_corrupt`, augmentez-le avec `wire_ceiling_mbps`. Sous Linux, un adaptateur branché sur un port USB 2.0 est limité à 30 Mo/s ; branchez-le sur un port USB 3.

#### Sursouscription (le seuil minimal par caméra)

Un réseau de caméras fonctionne sur n’importe quel réseau ; c’est la fréquence d’images qui fait la différence. Le rythme de simulation d’émission alloue à chaque caméra une part égale du débit disponible sans risque de collision, et toutes continuent de déclencher à la même fréquence ; ainsi, sur une liaison lente, la part — et la fréquence d’images — est faible : 8 camérassur un réseau 1 GbE sans trames jumbo, obtiennent 6,5 Mo/s chacune, soit environ **1 image par seconde**(environ 1,3 image par seconde en compression). La part est plafonnée au débit le plus lent auquel les caméras peuvent être synchronisées (`per_cam_floor_bps` : lu à partir de la plage GevSCPD de chaque caméra lors de la connexion, et jamais en dessous de 1 Mo/s). Si la liaison nécessite un débit inférieur, la connexion bascule vers l’émission**série**(`slip-emit-and-capture`) : les caméras s’alternent au débit maximal de la liaison, de sorte que chaque trame arrive mais que les éléments**ne sont pas exposés simultanément**— le niveau de réponse et les bloqueurs (`SERIAL: ...`) l’indiquent ; transmettez `force_tier="sim-capture-sim-emit"` pour refuser ce repli. La taille de trame n’entre pas en ligne de compte dans le partage, de sorte que**le regroupement et les zones d’intérêt (ROI) augmentent la fréquence d’images mais ne changent rien au fait qu’un tableau tienne ou non**. Seule une liaison incapable de transporter le tableau à n’importe quel rythme, que ce soit simultanément ou à tour de rôle, est**sursouscrite** — le mode de défaillance est la perte de paquets GVSP, et non une fréquence d’images plus faible. Solutions : moins de caméras, des trames jumbo de bout en bout ou une carte réseau plus rapide.

- Les réponses `analyze_array_network()` et `/api/camera/array/connect` contiennent les codes `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` et `per_cam_floor_bps`. Lorsque `oversubscribed` est vrai, la projection **remet à zéro les champs fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) plutôt que de signaler une fréquence trompeuse, lente mais fonctionnelle.
- `POST /api/camera/array/connect` accepte un paramètre de corps `pin_resolution` (**HTTP uniquement — ce n’est pas un kwarg du SDK**; `connect_array` ne l'expose pas). Il s'agit d'un mode strict : une connexion en sursouscription avec `pin_resolution` défini est**catégoriquement refusée** avec une erreur indiquant toutes les solutions possibles. Sans ce paramètre, la connexion se poursuit avec un avertissement indiquant que le tableau perdra des paquets. Dans les deux cas, la connexion conserve la taille de trame demandée — elle ne regroupe ni ne tronque jamais les données de son propre chef.
- À des fins de test uniquement : définissez `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` dans l’environnement du backend pour ramener le refus au niveau d’un avertissement sonore — vous établissez la connexion malgré tout et acceptez la perte de paquets.

#### État de la matrice — quel sous-système perd des trames

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) transporte un bloc `health` actif sur une
matrice connectée, réévalué sur une **fenêtre glissante de 10 secondes** . Il ventile la perte de trames
en deux causes nécessitant des correctifs opposés, au lieu d’un taux « incomplet » unique qui
n’en identifie aucune :

| Champ | Signification | Sous-système concerné |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (par numéro de série) | L’image **est arrivée mais était structurellement incorrecte**— perte de paquets GVSP. |**Réseau** : bande passante, cadencement, anneau de réception de la carte réseau, MTU |
| `never_arrived_rate_pct` (par numéro de série) | La trame **n’est jamais arrivée**— la caméra ne s’est pas déclenchée, ou rien n’en est sorti. |**Déclencheur / synchronisation** : câble M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Taux le plus bas par caméra pour chacune. | — |
| `per_cam_rate_pct` | Taux combiné d’incomplétude par caméra (les deux causes réunies). | — |
| `stable_for_seconds` | Durée pendant laquelle chaque caméra est restée en dessous de 0,01 %. | — |

Parallèlement à `health`, ce même enregistrement indique le nombre de fois où l'allocation totale est restée en attente :

| Champ | Signification |
| --- | --- |
| `wire_ceiling_mbps` | Budget de bande passante en vigueur pour l'hôte, en Mo/s. |
| `wire_ceiling_source` | Origine de ce chiffre, en mots — par exemple `USB-capped 200 MB/s (was theoretical 1062; …)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` lorsque `wire_ceiling_mbps=` l'a défini. |
| `nic_is_usb` | `true` pour un adaptateur Ethernet USB. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interprétation :** une valeur non nulle pour `gvsp_corrupt_rate_pct` avec `never_arrived_rate_pct` à 0 signifie que
le déclenchement et la synchronisation du câble sont parfaits et que 100 % de la perte se situe sur le chemin réseau — réduisez
`wire_ceiling_mbps` et reconnectez-vous. Le schéma inverse indique plutôt un problème au niveau du câble de synchronisation ou de la
ligne de déclenchement.

> **`target_fps` n’est pas le paramètre déterminant pour les trames corrompues.** Le rythme GevSCPD est défini une seule fois lors de la
> connexion ; ainsi, réduire la fréquence de déclenchement modifie le rapport cyclique et non le
> débit de rafales à émission simultanée. Une réduction mesurée de 5× de la demande n’a apporté aucune amélioration, tandis que
> la réduction du plafond de la liaison de 240 à 200 Mo/s a fait passer le taux de trames corrompues de ce même système de 10,4 % à
> 0,00 %.

> **Un réseau en cours d’exécution ne peut pas se replanifier automatiquement sur le micrologiciel LATTICE (M3C / M3M).** Il ne peut pas
> résoudre ce problème de lui-même ; déconnectez-le puis reconnectez-le afin que le sélecteur de temps de connexion se réorganise en fonction
> du nouveau plafond.

Une **carte Ethernet USB est plafonnée à 200 Mo/s** par la sonde, quelle que soit sa
nom : le tableau d’efficacité qui convertit un débit de liaison en un débit soutenu est
dérivé de la norme PCIe, et une carte réseau USB annonce son débit de liaison Ethernet tout en étant limitée par le
bus USB et son pilote. Cette limite est absolue, et non relative — une carte USB 1 GbE
atteint environ 80 Mo/s et n’est pas affectée. Elle s’applique sous Windows et Linux (y compris Jetson) ;
voir [Débits d’images attendus](#expected-frame-rates) pour connaître les conditions de mesure.

#### Méthodes `ArraySession`

| Méthode | Description |
| --- | --- |
| `status(timeout=10.0)` | État en temps réel du tableau en cours d’exécution : `target_fps` / `target_fps_auto` (la fréquence de déclenchement et si le contrôleur de fréquence la gère), `target_fps_clamped_reason` (raison pour laquelle une fréquence verrouillée est maintenue à un niveau inférieur, sinon `None`), `actual_fps` (mesuré ; 0 une fois que la boucle de capture a abandonné ou s'est bloquée), `recommended_fps`, `streaming` (Faux une fois que la boucle de capture a abandonné — vérifiez ceci, pas seulement `last_error` ; reste à « True » après un redémarrage des paramètres, ce que signale `restarting`), `stalled` (à « True » tant que la boucle de capture s’exécute mais n’a publié aucun groupe d’images depuis environ dix périodes d’images, soit 2 à 30 s ; `last_error` ne l'indique qu'après 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` uniquement lorsque le réseau était connecté via PTP **et** chaque membre a lu `PtpEnable` en mode « on » ; ce n’est pas la même chose que « verrouillé », ce que mesure `ptp_epoch`), `ptp_configured` (ce que la connexion a demandé), `ptp_off_members` / `ptp_unread_members` (éléments ayant lu en dernier `PtpEnable` désactivé / n’ayant pas répondu), `member_recovery` (`None`, ou un membre ayant subi une coupure d’alimentation ou une perte de liaison : `state`, `waiting` bien qu’il ne réponde pas, `reopening`, puis `recovered`, avec `members` et `why` — le réseau le réintègre lorsqu’il répond à nouveau et lui restitue son PTP de connexion, le câblage du déclencheur et de la ligne de synchronisation, la cadence et les réglages de la caméra, puis rediffuse sans nouveau `connect_array`), et `ptp_epoch` (mesuré), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` signifie qu’Windows n’a pas pu lire la vitesse de la carte — voir [Fréquences d’images attendues](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (indiquant si la chaîne de radiance par bande est utilisable et quel élément l’a interrompue), et `band_clamp` — tandis qu’un aperçu en direct affiche le tableau : la proportion de pixels éclairés que le démixage a limitée à 0, par élément (`members` : `{serial: [band0, band1, band2]}` dans l'ordre des bandes R, G, B de la caméra) et, par bande, ce que montre le composite (`composite` : `{band name: share}`), avec `warn`, la proportion que l’interface graphique signale comme « données de bande à faible confiance » (mesurées environ une fois par seconde ; `None` lorsqu’aucun affichage en direct n’est actif), et `member_ae` — pour chaque élément, l’exposition (`exposure_us`) et le gain (`gain_db`) de sa dernière image publiée, tirés de l’image elle-même (`read_nodes` de `ExposureTime` / `Gain` renvoie la dernière valeur écrite pendant l’exécution de l’exposition automatique, et non la valeur actuelle). |
| `set_member_settings(serial, **kwargs)` | Écrit les paramètres d’une caméra membre, tels que `CameraSession.set_settings()` les récupère (exposition, gain, luminosité cible, etc.) ; la caméra membre reste dans le tableau. Déclenche une exception `ChlorosConnectError` si la série n’est pas un membre. Voir [Exposition sur les membres du tableau](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Ce sur quoi la fonction « smart-prep » s’est mise d’accord (le niveau de capture, la taille d’image, `pixel_format`), le bloc `health` en direct et le budget de bande passante de l’hôte. Voir [État de santé de la baie](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Modifie le taux de déclenchement pendant le fonctionnement de la baie, sans reconnexion. Une valeur le fixe (le contrôleur de taux ne le modifie pas) ; `None` le remet en mode automatique. Renvoie `{requested, target_fps, target_fps_auto, clamped}` — `clamped` est True lorsque la fréquence a été réduite à un niveau compatible avec la capacité de l’hôte et du réseau. Voir [Capture par intervalles pour les levés par drone](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Une image de chaque caméra, **toutes issues d’un seul déclenchement**, sous forme de tableaux numpy (aucun fichier n’est écrit). Renvoie un `GrabResult` (un `dict`): `{serial: array}`, ou `{serial: (array, metadata)}` avec `with_metadata=True`. `.group_id` est le déclencheur commun à toutes les images ; `.errors` associe chaque caméra omise à la raison de cette omission (une caméra qui ne peut pas être prise en charge à ce déclencheur est omise et n’est jamais remplacée par une autre). `processing` correspond à `raw` / `debayered` / `radiance` / `reflectance` / `display`. La lecture est identique à celle de l’API du projet [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — une implémentation partagée — donc tous les mots-clés qui s’y trouvent (`band`, `align`, `bit_depth`, `refl_max`) fonctionne ici. Nécessite `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Itérateur fournissant un résultat `grab()` par tick (arguments-clés uniquement), cadencé à `fps`, limité par `duration` (en secondes) et/ou `count` (`None` = illimité). **Chaque ensemble renvoyé constitue un nouveau déclencheur :** un tick dont le déclencheur le plus récent est l’ensemble qu’il a déjà généré attend le prochain déclencheur au lieu de se répéter, de sorte que le flux s’exécute à la fréquence la plus lente entre `fps` et la fréquence d’images propre au tableau. Déclenche `FrameFetchError` plutôt que de se mettre en silence : immédiatement lorsque toutes les caméras refusent, pour une raison quelconque, que le tick suivant se répète ; après 60 s de ticks sans image ; et après 60 s sans nouveau déclencheur (« aucun nouveau déclencheur depuis 60 s »). Même implémentation que `ArrayHandle.frame_stream()`. Voir [Images en direct dans votre propre pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Lance la capture continue propre au backend (mode « Continu » de l’interface graphique) → [`ContinuousHandle`](#continuoushandle). Enregistre **chaque** groupe synchronisé publié par le tableau, comme le ferait `capture()` (mêmes contrôles d’exportation), jusqu’à ce qu’il soit arrêté ou jusqu’à ce que `max_captures` / `duration_s` (`None` = jusqu’à ce qu’il soit arrêté). Génère une exception `ChlorosConnectError` si un processus est déjà en cours d’exécution pour ce tableau. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Un groupe de capture synchronisé. Renvoie un `CaptureResult` (liste de dictionnaires de trames + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Passez un résultat précédent`group_seq` comme `after_seq` pour obtenir le groupe synchronisé suivant après celui-ci plutôt que d’attendre un nouveau. Commandes d’exportation ci-dessous. |
| `capture(..., smart=True)` | **Capture intelligente** — attend que l’AE se stabilise sur toutes les caméras, puis déclenche la capture. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Capture la plus rapide : données brutes uniquement + la lecture DAQ attribuée (+ l’index combiné libre). Correspond au bouton « Capture la plus rapide » de l’interface graphique. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Unique / Continue / Par intervalle dans une boucle limitée. Renvoie `list[CaptureResult]`.**Nécessite `count` et/ou `duration_s`** pour se terminer (le SDK ne dispose pas de Ctrl+C). Se termine également prématurément, après le passage, lorsqu’un passage signale un disque plein (une entrée `.failed` avec `disk_full`) ; ce passage constitue le dernier résultat. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Lancer l’enregistrement de la vue combinée en direct sous forme d’index vers une vidéo/un GIF → `RecorderHandle`. La vidéo est au format Motion JPEG `.avi` (pas besoin de ffmpeg). En l’absence de fenêtre graphique ouverte, le backend effectue le rendu : `expr` (une expression d’index dans les noms de bandes du tableau, par exemple `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), ou l’NDVI du tableau par défaut. Un enregistreur composite par matrice. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Lancer une rafale Bayer brute à haute fréquence d’images → `RecorderHandle`. Retraiter hors ligne avec `build_video()`. `orient=True` : les produits générés par `build_video()` sont orientés en fonction de l’orientation de la caméra (les produits par caméra selon celle de leur propre caméra, l’index combiné selon celui de la caméra maître) ; les images brutes restent dans leur format d’origine et le manifeste de la rafale enregistre l’orientation de chaque élément. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Retraitement hors ligne d’une rafale brute enregistrée en une ou plusieurs vidéos calibrées. Bloque jusqu’à la fin (`wait=True`) et renvoie le résultat de la compilation : `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — voir [le résultat de la compilation](#the-build_video-result) ci-dessous. |
| `build_video_status(job_id, timeout=15.0)` | Interroger une tâche de compilation hors ligne : `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Libérer l'ensemble du tableau — **si cette session l'a ouvert** (`connect_array`) : il s'agit alors de `force_disconnect()`, et une exception `ChlorosConnectError` est levée lorsque la libération n'est pas confirmée (la sortie du bloc `with` ne déclenche jamais d’erreur). Sur un descripteur provenant de `attach_array`, il laisse le réseau en fonctionnement et affiche une ligne sur stderr l’indiquant : le réseau peut appartenir à l’interface graphique ou à un autre script. Chaque caméra est laissée en fonctionnement libre et dans l’état où la connexion l’a trouvée ; `keep_settings=True` enregistre ses paramètres dans la mémoire propre à la caméra, `clear_settings=True` réinitialise cette mémoire aux paramètres d’usine (voir [Mémoire de la caméra](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Libère l’ensemble du réseau, quel que soit l’utilisateur qui l’a ouvert. À utiliser sur un descripteur associé lorsque vous souhaitez mettre fin au réseau (par exemple, pour le reconnecter avec des paramètres différents). Renvoie une valeur dès que le backend confirme la libération, ou lorsqu’il n’y a rien à libérer (le tableau a déjà disparu ou aucun backend n’est en cours d’exécution) ; une libération ayant expiré ou refusée par le backend déclenche `ChlorosConnectError`. Avec l’un ou l’autre indicateur, il renvoie ce que la mémoire de chaque caméra a reçu (`camera_memory`), sinon `None`. |

Commandes d’exportation `capture()` (même point de terminaison que celui utilisé par l’interface graphique et la CLI) :

- `processing` / `levels` — `processing="all"` (ou `levels=["raw","radiance",…]`) enregistre tous les types d’exportation applicables par caméra ; une seule valeur `processing` enregistre uniquement ce niveau.
- `aligned` — **activé par défaut (`None`), car la capture du même réseau par l’application Chloros est**(son paramètre de capture**« Aligné »** par matrice). L’exportation non brute de chaque élément est alignée sur l’image de la caméra principale via le [profil d’alignement](#array-alignment) et recadrée à la zone de chevauchement commune des éléments ; ainsi, les fichiers sont alignés au pixel près et présentent la taille, le recadrage et les balises `Alignment*` identiques à ceux de l’application (dans une région d’intérêt [ROI], la transformation est rebasée sur la fenêtre); `XMP-mapir:AlignmentJSON` indique `"applied": true` avec son `crop_roi`. Les fichiers bruts restent non déformés à leur taille native mais conservent la transformation dans leurs métadonnées. `aligned=False` enregistre chaque fichier à sa taille native, non déformé ; `aligned=True` correspond au comportement par défaut. `continuous()`, `capture(smart=True)`, `capture_repeated()` et les options au niveau du projet `ArrayHandle.capture()` / `capture_smart()` adoptent la même valeur par défaut. Que la déformation soit appliquée ou non, chaque fichier membre enregistre la transformation `Alignment*` de sa caméra dès que le tableau dispose d’un profil d’alignement — à l’exception d’une caméra que le profil n’aligne pas (omise ou forcée à l’identité en raison d’un échec de résolution), ainsi que la caméra de référence lorsqu’aucune autre caméra n’est alignée : celles-ci ne comportent aucune balise `Alignment*`. En l’absence de profil sur le tableau, la capture est enregistrée non alignée, et le résultat l’indique : `result.alignment` correspond à `{"requested": True, "applied": False, "warnings": [...]}`, avec un avertissement par raison pour laquelle la déformation n’a pas pu être appliquée (absence de profil, éléments à des résolutions différentes, tableau non, une image sur laquelle le profil ne peut pas être mappé). `applied` correspond à `True` uniquement lorsque toutes les exportations ayant demandé la correction de déformation l’ont obtenue. L’alignement automatique du réseau crée le profil une fois que l’exposition s’est stabilisée ; `ArrayHandle.calibrate_alignment()` / la CLI `align-calibrate` en créent un à la demande. `result.oriented` indique si `orient=True` a inversé les produits.
- `render_index=False` — ignore la superposition de l’indice de végétation par caméra ; par défaut, celle-ci est rendue là où elle a été configurée.
- `daq` / `daq_tolerance_ms` — la lecture DAQ est enregistrée par défaut sous forme de fichier « sidecar » `.daq`, même si aucun niveau sélectionné n’en a besoin ; `daq=False` l’ignore alors, et `daq_tolerance_ms` définit la tolérance de correspondance (500 par défaut). `force_daq=True` est l’ancienne orthographe de la valeur par défaut. Voir [La lecture DAQ enregistrée par une capture](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — active les **produits**de chaque membre (`display`, `radiance`, `reflectance`, l’index par caméra et le composite) par l’orientation de la caméra, après la déformation d’alignement : par l’orientation de la**caméra principale**au moment où ils ont été alignés dans son repère (valeur par défaut `aligned` — les fichiers membres restent co-enregistrés), sinon par celle de leur propre caméra.**Par défaut : chaque fichier de membre en format natif**, comme c’est toujours le cas pour les captures en tableau ; `raw` et `debayered` restent en format natif dans tous les cas. Chaque fichier enregistre l’orientation du cadre dans lequel se trouvent ses pixels (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — un fichier brut, celui de sa propre caméra), et le fichier XMP `Alignment*` conserve la transformation du cadre natif, appliquée avant l’orientation ; un fichier RAW contenant cette transformation enregistre également `ChlorosAligned*` (celle du cadre de référence), c’est-à-dire l’orientation d’un produit transposé dans le cadre de référence. Le traitement fait pivoter chaque produit selon la transformation propre à ses pixels : celle de l’alignement lorsqu’il transpose le fichier RAW (alignement de tableaux activé), celle de l’appareil photo lui-même lorsqu’il ne le fait pas. `capture_repeated()` et `capture_fastest()` la laissent passer ; les fonctions `ArrayHandle.capture()` / `capture_smart()` au niveau du projet la prennent également en compte.

**Compression TIFF (paramètre HTTP uniquement) :** `ArraySession.capture()` n’envoie pas de clé `compression`, donc la valeur par défaut du backend s’applique — `POST /api/camera/array/capture` lit un paramètre de corps `compression`, `"none"` par défaut : non compressé, les pixels du capteur tels qu’ils sont lus (~6,3 Mo par image en pleine résolution). `"deflate"` (zlib L1 sans perte + prédicteur horizontal, environ 4,1 Mo/image) écrit des fichiers plus petits mais plus lentement — les deux sont sans perte et se lisent de manière identique à l’importation. Le SDK ne propose aucun argument de ligne de commande pour cela ; la solution de contournement est `chloros-cli lattice array-capture --compression deflate` ou le HTTP brut. DEFLATE bloque le GIL de Python, de sorte que les écritures compressées ne peuvent pas être parallélisées entre les threads d’écriture par caméra — une capture en pleine résolution soutenue à la fréquence du capteur nécessite le format non compressé par défaut. Détails : [Référence CLI → array-capture](cli-reference.md).**Groupes consécutifs (protocole HTTP) :** le point de terminaison renvoie `group_seq` ; une boucle qui le renvoie sous la forme `after_seq` lors de sa requête suivante obtient le groupe synchronisé suivant — y compris celui publié pendant l’enregistrement de la capture précédente — au lieu d’attendre un groupe publié après la requête, et `skipped_groups` signale tout groupe manqué par la boucle. `capture_repeated`, `--continuous` de l’CLI et le mode Continu de l’interface graphique fonctionnent tous ainsi ; les passages d’intervalle restent à jour à chaque moment prévu.**Remplacements d’exportation par membre (HTTP uniquement) :**le même point de terminaison accepte également `exclude_serials` (liste — suppression de membres de l’ensemble enregistré ; le tableau continue de se déclencher comme un groupe synchronisé et les membres exclus sont renvoyés dans `excluded`), `serial_levels` (remplacements au niveau de chaque caméra `{serial: [level tokens]}`) et `serial_index` (remplacements de superposition d’index par caméra `{serial: bool}`). Il s’agit de paramètres de corps de parité de l’interface graphique et**pas encore de paramètres clés-valeur de l’SDK** ; les membres absents des tables sont remplacés par les valeurs globales du tableau `levels` / `render_index`.

##### Inspection des caméras ignorées — `CaptureResult.skipped`

`ArraySession.capture()` renvoie un `CaptureResult`, qui est une sous-classe de `list` : parcourez-le, indexez-le, `len()` — tous les modèles existants continuent de fonctionner. Le nouveau code peut inspecter l’attribut `.skipped` pour voir quelles caméras ont été exclues et pourquoi. Le cas le plus courant concerne les caméras RGB dans un réseau à filtres mixtes lorsque vous demandez `processing="radiance"` ou `"reflectance"` — la radiance par pixel Bayer n’a aucun sens pour un capteur à large bande ; le backend ignore donc ces caméras plutôt que de produire des résultats absurdes.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Les jetons de motif suivent le modèle `<level>-not-applicable-to-rgb-cam` (une entrée par niveau ignoré, chacune comportant `level`). Une caméra dont le pack d’étalonnage n’est pas chargé ignore `radiance` / `reflectance` avec `<level>-needs-calibration (connect the camera so its cal pack loads)`, et, en l’absence de lecture DAQ utilisable de la réflectance, saute avec `reflectance-skipped-no-dls`. Les autres sauts spécifiques à la réflectance sont `reflectance-skipped-no-fresh-dls` (aucune nouvelle lecture de rayonnement descendant disponible), `reflectance-skipped-bound-daq-unavailable (…)` (le DAQ associé n’a pas pu être atteint), et `dls-uncalibrated-band-<nm>` — la bande se situe en grande partie en dehors de la plage radiométriquement calibrée du capteur de lumière du DAQ (~374–974 nm) ; par conséquent, la division absolue de la réflectance basée sur le DAQ est refusée et l’image bascule de manière évidente vers la réponse du capteur. Parmi les références commercialisées, seule la F988 déclenche ce comportement ; le parcours pris en charge par cette caméra est le flux de travail avec panneau de réflectance.

##### Fichiers qui n’ont pas pu être écrits — `CaptureResult.failed`

Une capture qui n’a pu écrire qu’une partie de ses fichiers renvoie tout de même : la liste contient tous les fichiers qui **ont été** écrits, et `.failed` répertorie les autres, à raison d’un dictionnaire par fichier avec `serial` (`None` pour le composite combiné et le fichier d’accompagnement `.daq` d’un tableau), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, ou `match` — la provenance de `.daq` vers `.match.json`, lorsque `.daq` a été écrit mais que son fichier d’accompagnement ne l’a pas été) et `error`. Sur un disque plein, l’entrée contient également `disk_full: True` ; toute capture ultérieure sur ce disque échouerait de la même manière, ce qui explique pourquoi `capture_repeated()` s’arrête là. Aucune exception n’est levée en cas de capture partielle, et aucun fichier dans `.failed` n’a été laissé sur le disque à moitié écrit. `.failed` est vide lors d’une capture propre.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Le DAQ lisant une capture enregistre — `.daq_files` et `.daq_matches`

Chaque fois qu’un DAQ est associé à la caméra ou au réseau de caméras, ou qu’il est mis en pool, chaque capture enregistre la lecture descendante qu’il a utilisée : un fichier `.daq` dans le sous-dossier `daq/` du dossier de capture, avec ses fichiers `.match.json` et `.cameras.json` (dont elle provient). Cela vaut également pour une capture brute ou débayérisée, et il s’agit du même fichier, que la capture provienne de l’interface graphique, de la CLI ou du SDK. `daq=False` ignore cette étape pour une capture dont les niveaux n’utilisent pas le DAQ (un niveau de type `radiance`, `reflectance` ou `display` l’écrit toujours) ; `daq_tolerance_ms` déplace la ligne de tolérance pour cette capture et jamais la lecture utilisée. Quel DAQ : celui lié à la caméra ou au réseau de capteurs (la sélection du capteur de lumière dans l’interface graphique, une liaison de projet ou `chloros-cli daq pool-bind`), sinon le plus récent dont dispose le backend. L’interface CLI désigne les commandes `--no-daq` et `--daq-tolerance-ms` ([Référence CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Une capture enregistre la valeur du capteur de lumière utilisée sous la forme `.daq`, accompagnée d’un `<file>.daq.match.json` indiquant comment cette valeur a été liée à l’exposition : `method`, `gap_ms` (de l’exposition à la lecture DAQ la plus proche) et `in_tolerance` (dans les 500 ms, ou le `daq_tolerance_ms` de la capture). Le fichier XMP de réflectance contient les mêmes valeurs que `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` et `Chloros:DlsInTolerance`.

Une capture attend la première mesure du DAQ après l’exposition, afin de pouvoir fusionner les mesures situées de part et d’autre de celle-ci (`method` `interpolated`). Un DAQ-E envoie une mesure toutes les 0,4 s à la lumière du jour et toutes les 1 à 2 s à l’intérieur ; ainsi, `capture()` est renvoyé jusqu’à environ un intervalle et demi plus tard ; l’exposition elle-même n’est pas retardée. On n’attend pas un DAQ qui a cessé d’envoyer des données. Une mesure DAQ-E correspond à la moyenne de la luminosité sur l’intervalle écoulé depuis sa mesure précédente ; elle est donc placée au milieu de cet intervalle : `gap_ms` est mesuré par rapport à ce milieu, et la valeur `.match.json`'`bracket_stamps_ns` fournit les horodatages propres aux mesures. `ArraySession.continuous()` attend sans ralentir la capture. Les passages en sens inverse de `capture_repeated()`ne s’attendent pas, elles conservent donc leur cadence et sont associées à la mesure la plus proche avant l’exposition.

Une mesure n’expire jamais, donc aucune trame n’est perdue à cause d’une mesure tardive — elle est plutôt étiquetée :

- **Le flux du DAQ est bloqué** — sa dernière lecture est utilisée, et `gap_ms` indique depuis combien de temps elle date (`in_tolerance: false` : plus de 500 ms).
- **Aucun DAQ à lire**— déconnecté (`disconnect()`, `chloros-cli daq pool-disconnect`), ou le DAQ associé est inaccessible — la dernière lecture de la caméra est réutilisée et associée à**cette** exposition : `method`, `retained`, `gap_ms` issus de cette exposition, `in_tolerance` est recalculée, et `retained_from_epoch_ns` indique l’exposition à laquelle elle a été initialement associée. Une caméra qui n’a jamais fourni de mesure ne reçoit toujours aucune réflectance, et le résultat explique pourquoi (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ou le refus de la capture).

Les fichiers enregistrés par une capture dans le dossier du projet ouvert dans Chloros rejoignent la liste des fichiers de ce projet, comme le fait une capture dans l’application (les fichiers enregistrés ailleurs ne sont pas concernés) : `CameraSession.capture()`, `ArraySession.capture()` (ainsi que `smart=True`) et `capture_cameras_transient()`. Le résultat l'indique dans `.project_add` (`capture_cameras_transient()` : le `project_add` du dictionnaire) -- `held`, `files` et `message` du backend ; `held` correspond à `True` tant qu’un traitement est en cours, et les fichiers s’y ajoutent une fois celui-ci terminé. `None` lorsqu’aucun fichier n’a été ajouté au projet.

`CameraSession.capture()` et `ArraySession.capture()` les renvoient sous la forme de `.daq_files` et `.daq_matches`, tandis que les réponses de capture HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) contiennent `daq_files` et `daq_matches`, à raison d’un par `.daq` : `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` et `stream_fault` — erreur de flux propre au DAQ, consultée uniquement en cas de mesure hors tolérance. Avant de vous fier à la réflectance issue d’une capture, vérifiez `in_tolerance` dans son `.match.json` ou XMP, ou les niveaux [`status()`](#daqsensorsession-methods).

Niveaux `processing` :

| Niveau | Sortie |
| --- | --- |
| `"raw"` | Bayer monocanal (caméras mono : la bande unique) provenant directement du capteur. |
| `"debayered"` *(par défaut du SDK)* | Couleur à 3 canaux : le décalage de noir (DSNU) du pack d'étalonnage soustrait du brut, puis le dématriçage sensible aux contours d'OpenCV — sans table de conversion linéaire (LUT), champ plat ni démixage (caméras monochromes : 1 canal en niveaux de gris). Le fichier TIFF enregistré est dans l’ordre des canaux **RGB**(RGB photométrique, ce qui permet à n’importe quelle visionneuse d’images d’afficher les couleurs réelles) ; les tableaux en mémoire renvoyés par `grab()` / `frame_stream()` sont dans l’ordre**BGR** d’OpenCV — inversez les canaux (`frame[..., ::-1]`) pour comparer le résultat avec un fichier enregistré. `chloros-cli process` écrit le même résultat que `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm via la chaîne radiométrique complète. Multispectral uniquement — les caméras RGBs sont ignorées. |
| `"reflectance"` | uint16 0..32768 (compatible Pix4D) ; nécessite un appairage DAQ en temps réel pour une référence absolue. Multispectral uniquement. |
| `"display"` | Chaîne complète correspondant à l’aperçu de l’interface graphique (CCM + WB + gamma selon le profil de la caméra). |
| `"all"` | **Un fichier par niveau applicable** pour chaque caméra (correspondant à l’option « Capture All » de l’interface graphique / par défaut de la CLI). Le fichier `CaptureResult` renvoyé contient alors un dictionnaire d’images par `(cam, level)`, avec le niveau indiqué dans chaque dictionnaire ; les niveaux non applicables apparaissent dans `.skipped`. La lecture DAQ utilisée pour la capture est enregistrée sous forme de fichier sidecar `.daq`. |

> **Remarque — la valeur par défaut diffère de celle de l’CLI.** `ArraySession.capture()` prend par défaut la valeur `processing="debayered"` ; la commande `chloros-cli lattice array-capture` prend par défaut la valeur `processing="all"`. Transmettez explicitement `processing="all"` depuis le SDK pour reproduire l’enregistrement à plusieurs niveaux de la CLI/GUI.

##### Noms des fichiers de capture de tableaux

Chaque niveau est enregistré dans son propre sous-dossier de `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` pour `display`, `index/`, `composite/`). Chaque fichier porte le nom de sa caméra sous la forme `SN<serial>-<filter>` : le numéro de série, puis la partie « filtre » du modèle de la caméra — `SN214000533-FOCN` pour une `M3C-L87-FOCN`, `SN261902142-F450` pour une `M3M-L87-F450` — associé à `-`, de sorte qu’un nom comporte les mêmes champs séparés par « `_` » qu’auparavant. Une caméra dont le modèle ne peut pas être lu est simplement `SN<serial>`, et les captures issues de versions antérieures ne comportent aucun filtre dans leur nom. Le nom du fichier dépend du nombre de niveaux demandés par un membre :

| Nombre de niveaux demandés | Nom du fichier | Exemple (`reflectance`, par défaut `format="tiff"`) |
| --- | --- | --- |
| **Un niveau** (`processing="reflectance"`, ou un élément unique `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — pas de suffixe de niveau, comme dans les versions précédentes, afin que les scripts existants continuent de fonctionner | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Plusieurs niveaux** (`processing="all"`, ou `levels` avec plusieurs éléments) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

Le [masque de limitation](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) suit le nom du produit, `clamp_masks/<product folder>/<product name>_clampmask.tif` : unecapture de réflectance à un seul niveau génère `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, tandis que `processing="all"` génère `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. La règle relative au suffixe est déterminée pour chaque élément en fonction des niveaux demandés ; ainsi, un niveau ignoré (par exemple, la réflectance sur une caméra RGB) ne la modifie pas. Lisez les chemins d’accès à partir des images renvoyées plutôt que de les construire. Une capture à caméra unique (`CameraSession.capture()`) ajoute toujours le niveau : `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG / JPEG : ne prennent en charge que les produits d'affichage et d'indexation

Un fichier PNG ou JPEG contient 8 bits par canal. Toute demande portant sur un niveau que le format ne peut pas prendre en charge est refusée avant toute capture, avec un message indiquant le niveau et le format TIFF qui le prend en charge :

- **`radiance`**(float32) et**`reflectance`**(uint16, `32768` = 1,0) — ainsi que `processing="all"`, qui les inclut. Le SDK vérifie cela sur les arguments : `capture()` déclenche**`ValueError`** sans requête (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` provenant d’une caméra diffusant plus de 8 bits** (M3C 12 bits, Mono12 M3M) — le backend les refuse, connaissant la profondeur de bits de chaque caméra : Code 400 HTTP avec `error_code` et `FORMAT_CANNOT_HOLD_LEVEL`, généré sous la forme `ChlorosConnectError` (`CameraSession` / `ArraySession`) ou `RuntimeError` (`CameraHandle` / `ArrayHandle`). La capture à un seul niveau `debayered` d’une caméra génère une image 8 bits au format PNG/JPEG et n’est pas refusée.

Enregistrez les données radiométriques au format `format="tiff"` (`ext=".tiff"`) ; utilisez les formats PNG ou JPEG pour `display` et les rendus indexés.

#### Comportement de l’exposition automatique

- **L’exposition d’abord, le gain ensuite.** Lorsque le gain est en mode Auto, il reste à 0 dB jusqu’à ce que l’exposition ait atteint la valeur maximale autorisée.
- **Une exposition manuelle laisse le gain en mode « Auto »** jusqu’à ce que vous modifiiez le gain ou que vous le régliez en mode manuel. De retour en mode « Auto », la limite du gain est la même qu’auparavant.
- **Les petites zones surexposées n’assombrissent pas la scène.** Une réflexion spéculaire représentant jusqu’à 1 % de l’image est ignorée, et dans les panneaux d’une cible d’étalonnage connue, jusqu’à 0,5 % d’un panneau.
- **Le plafond d’exposition du DAQ ne s’applique que lorsqu’une cible d’étalonnage est dans le champ de vision.** Cela empêche un panneau blanc d’être écrêté ; lorsqu’aucun panneau n’est visible, la caméra expose en fonction de la scène.
- **Une lumière constante reste stable, et une lumière changeante se stabilise en quelques secondes.** En cas de lumière constante, l’exposition et sa limite restent inchangées, et après un changement important de luminosité, l’image se stabilise en quelques secondes. Une surexposition ne limite plus définitivement l’exposition, et un changement de zone d’intérêt (ROI) permet de retrouver l’exposition adéquate.
- **La cible de luminosité reste accessible.** Elle n’augmente pas lorsque la caméra est à sa limite d’exposition et que son gain ne peut plus augmenter ; ainsi, un retour de lumière ne provoque pas de surexposition de l’image.
- **Une caméra dont l’exposition automatique cesse de répondre est maintenue au niveau correspondant à la luminosité ambiante.** En plein soleil, une caméra pourrait descendre à son temps d’exposition le plus court et y rester avec une image sombre, alors que tous ses paramètres indiquent une valeur normale. Chloros surveille l’image, maintient cette caméra au niveau qu’elle avait conservé pour cette luminosité, et la relâche lorsque la luminosité change. Un capteur de lumière obscurci ou couvert ne perturbe pas ce fonctionnement.

#### Exposition sur les éléments du réseau

Une caméra LATTICE limite `ExposureTime` à une période de sa fréquence d’images. Pendant qu’un réseau diffuse des données, Chloros exécute chaque élément avec `AcquisitionFrameRateEnable` activé, à une fréquence légèrement supérieure à la fréquence de déclenchement, de sorte que le plafond d’exposition d’un élément se situe un peu en dessous de la période de déclenchement, **dans n’importe quelle zone d’intérêt (ROI)** :

- La période d’image d’un élément correspond à la période de déclenchement moins une marge de 15 % de cette période (au minimum 3 ms, au maximum la moitié de la période), et son plafond d’exposition correspond à cette période d’image moins environ 0,1 ms. Exemple : un déclenchement à 10 images par seconde (100 ms) donne une fréquence de membre d’environ 11,8 images par seconde et un plafond d’exposition d’environ 85 ms.
- **Une durée d’exposition manuelle `ExposureTime` supérieure à ce plafond ralentit le déclenchement pour s’y adapter.** Chloros ouvre d’abord le plafond de ce membre à cette valeur et laisse la fréquence réduite fonctionner pendant quelques périodes de déclenchement (l’écriture peut prendre quelques secondes de plus), puis l’enregistre, et le déclenchement ne se produit pas plus rapidement que ne le permet l’exposition. Sur un tableau à cadence automatique, la cadence précédente est rétablie dès que chaque membre dont l’exposition manuelle l’avait ralentie revient en exposition automatique. Une valeur que la caméra elle-même ne peut pas accepter est toujours refusée, la plage qu’elle accepte étant désormais indiquée dans l’erreur, et l’exposition automatique du membre est rétablie si elle avait été désactivée uniquement pour cette écriture. Avant la version 1.3.0, une petite zone d’intérêt (ROI) plafonnait chaque membre à la période de fonctionnement libre du capteur, quelle que soit la fréquence de déclenchement : 4,9 ms à 768 × 304, de sorte qu’aucun élément ne pouvait être éclairci.
- Chloros ne raccourcit jamais une exposition manuelle pour s’adapter à un déclenchement plus rapide. Une fréquence automatique reste dans les limites autorisées par les expositions manuelles ; une fréquence fixe plus rapide que ne le permet une exposition laisse à ce membre un plafond suffisamment long pour celle-ci, et il n’accepte pas de déclenchements plus rapides que ne le permet son exposition. Réduisez l’exposition, ou la fréquence fixe, lorsque chaque membre doit répondre à chaque déclenchement.
- **Relecture :** la lecture des nœuds d’un élément affiche `AcquisitionFrameRateEnable` True et `AcquisitionFrameRate` à la fréquence de l’élément ; l’activation est désactivée lorsque le réseau s’arrête.
- **Le tableau possède ces deux nœuds. N’écrivez pas `AcquisitionFrameRateEnable` ni `AcquisitionFrameRate` sur un élément du tableau** — réglez plutôt laà l’aide de `target_fps` / `set_target_fps()` à la place.

Écrivez l’exposition d’un élément à l’aide de `arr.set_member_settings(serial, exposure_time=20000)` (les mots-clés de `CameraSession.set_settings()` ; `chloros-cli lattice exposure --serial` a le même effet). Le membre reste dans le tableau. N’ouvrez pas un membre du tableau avec `connect_camera()` : connect refuse une caméra qui se trouve déjà dans un tableau, et la commande `CameraSession` qu’il renverrait supprime la caméra de son tableau lors de la déconnexion. `ArraySession.status()["member_ae"]` signale l’exposition de chaque élémentà partir de sa dernière image.

### Un seul enregistrement de projet, quel que soit le mode

Un dossier de projet contient les paramètres de chaque appareil : `cameras.json` — ceux de chaque caméra (`cameras[].settings`, pour une caméra unique et pour chaque élément du réseau) et ceux applicables à l’ensemble du réseau (`arrays[].capture` : format, ROI, binning, fréquence de déclenchement, budget de câblage) — ainsi que `sensors.json`, les paramètres de chaque capteur de lumière (temps d’intégration, moyenne par image, limite). L’application, l’CLI, le SDK et le serveur MCP lisent et écrivent tous les mêmes fichiers, de la même manière :

- **Un périphérique connecté à un projet y est enregistré et y consigne chaque modification.** « Dans le cadre d’un projet » signifie qu’un projet est ouvert dans le backend (celui de l’application), ou — via un fichier `ChlorosProject` (`open_project()`), qui indique son propre dossier au backend (`project_path`). Une caméra, un réseau de capteurs ou un capteur de lumière que le projet ne contient pas encore est ajouté à ses fichiers lors de sa connexion (`connect_camera()`, `connect_array()`, la connexion du capteur de lumière, l’CLI, l’application, `chloros-mcp`), avec ses paramètres. Un changement de mode (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, les paramètres du réseau, `set_cap()`, l’CLI ou les commandes de l’application) est enregistrée dans les fichiers de ce projet quelques instants plus tard. Une exposition, un gain ou une cible AE définis sur une caméra sont enregistrés comme « maintenus » ; ainsi, la réouverture de la caméra les conserve.
- **L'ouverture d'un projet configure ses périphériques à partir de celui-ci**, quel que soit le mode : une caméra, un réseau ou un capteur de lumière connecté sans paramètres propres se voit attribuer les(`lattice cam-connect` / `array-connect` lorsqu’un projet est ouvert, `chloros-cli project connect`, `open_project().connect_all()`, lors de la réouverture de l’application). Un réseau connecté sous un nouvel identifiant met à jour le réseau que le projet a enregistré avec les mêmes caméras ; un réseau comportant d’autres caméras est ajouté à côté.
- **L’enregistrement effectué par l’application elle-même ne remplace jamais les paramètres en temps réel d’un périphérique** par une ancienne copie de ceux-ci, et conserve les modifications par élément enregistrées par un appelant CLI ou SDK.
- **Le format de pixels que vous définissez est celui que le projet conserve.** Un format défini dans les paramètres du tableau, via `--pixel-format` ou `connect_array(pixel_format=)`, y compris le format 8 bits, est enregistré et réutilisé à l’identique. Si aucun format n’est défini, chaque caméra se connecte avec son format par défaut 12 bits (`BayerRG12`, `Mono12` sur un capteur monochrome). Le nom d’un format correspond à sa profondeur de bits ; la famille correspond à celle du capteur ; ainsi, une caméra couleur n’est jamais configurée sur un format mono, ni une caméra mono sur un format Bayer, et la ligne `[PixelFormat <serial>] ... written` l’indique lorsqu’un format a été mappé. Les exportations d’une capture 8 bits sont en 8 bits, réflectance comprise : redimensionnées et marquées de son `PixelScale`.
- Un périphérique connecté sans projet n’enregistre rien.
- **Un enregistrement suit un projet créé après lui ; une ouverture le laisse avec son projet.**Lorsqu’un projet est créé — `create_project()`, `chloros-mcp`'s `create_project`, ou la fonction**Nouveau projet**de l’application — ou dupliqué (fonction**Dupliquer**de l’application, qui ouvre la copie) alors qu’un enregistrement est encore en cours d’écriture dans le projet qui était ouvert auparavant, l’enregistrement se poursuit et ce qu’il écrit se retrouve dans le nouveau projet, au même emplacement à l’intérieur de celui-ci (ci-dessous). L'ouverture d'un projet existant — `ChlorosProject.process()` (qui ouvre son projet en arrière-plan), `chloros-cli process`, l’option**Ouvrir un projet** de l’application — laisse un enregistrement en cours dans le projet où il est enregistré : il s’achève là, et ses fichiers rejoignent la liste des fichiers de ce projet (l’application arrête tout enregistrement avant d’ouvrir un projet).
  - `ArraySession.record()` et `ArraySession.burst()` : la vidéo, ou le dossier de rafale contenant toutes les images, y est déplacé dans son intégralité lorsque l’enregistrement s’arrête. Le résultat de l’arrêt indique le nouveau chemin d’accès (`out_dir`, `video_path`), et une compilation de la rafale est générée à partir de celui-ci. Si le disque refuse le déplacement, l'enregistrement reste à l'emplacement où il a été écrit, et le résultat l'indique (`relocation_error`). Les images brutes d'une rafale arrêtée et le fichier `.daq` sont ajoutés à la liste des fichiers du projet dans lequel elle aboutit ; la vidéo n'est pas ajoutée à la liste des fichiers d’un projet.
  - `ArraySession.continuous()` : chaque capture effectuée après le changement est enregistrée dans le nouveau projet ; celles enregistrées auparavant restent à leur emplacement d’origine.
  - `DAQSensorSession.record_start()` (et l’enregistrement par le capteur de lumière de l’application elle-même): le fichier `.daq` est déplacé vers le dossier `light_sensor` du nouveau projet lorsque l'enregistrement s'arrête ; `record_stop()` renvoie le nouveau chemin d'accès, et le fichier est ajouté à la liste des fichiers de ce projet.

  Un enregistrement dans un dossier situé en dehors du projet ouvert reste à sa place, et les tâches `start_camera_continuous()` / multi-flux conservent le dossier qui leur a été attribué.
- **Dans l’application, les captures effectuées après la création d’un nouveau projet y sont intégrées.** La création ou la duplication d’un projet maintient toutes les sessions de capture et tous les enregistrements en cours, et chaque capture effectuée par la suite (Capturer tout, Démarrer la capture) est enregistrée dans le nouveau projet et s’ajoute à sa liste de fichiers. L’ouverture d’un projet existant dans l’application les interrompt tous au préalable.
- **Un projet n’est pas déplacé pendant qu’il enregistre.**Le déplacement du projet ouvert vers un autre répertoire de travail (via**Paramètres → Répertoire de travail** de l’application ; aucun appel SDK ne déplace un projet, contrairement à un client HTTP effectuant une requête vers `/api/move-project`) est refusé avec le code HTTP 409 `PROJECT_RECORDING` tant qu’un enregistrement en rafale ou en rafale continue, une capture continue en rafale, une tâche de capture continue / multi-flux d’une caméra ou un enregistrement par capteur de lumière y écrit des données : `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Ceci est distinct de l’état dans lequel les caméras sont laissées lors de la déconnexion (voir [Mémoire de la caméra](#camera-memory--keep_settings--clear_settings)) : un projet configure lui-même ses caméras à son ouverture.

### Mémoire des caméras — `keep_settings` / `clear_settings`

Lorsqu’un réseau est connecté, ses caméras reprennent les paramètres du réseau, et vous pouvez ensuite les modifier. Lorsque Chloros libère les caméras — `ArraySession.disconnect()` / `force_disconnect()`, à la fin d’une fonction transitoire, à l’arrêt de `start_camera_continuous()` —, il rétablit pour chacune d’entre elles l’état dans lequel elle se trouvait avant la connexion : son câblage de déclenchement et de ligne de synchronisation, ainsi que ses paramètres PTP et de caméra — `ExposureAuto` / `GainAuto` (ainsi qu’un réglage manuel de `ExposureTime` / `Gain`), `TargetBrightness` ainsi que les limites et l’amortissement de l’exposition automatique, la balance des blancs, le niveau de noir, `PixelFormat`, le binning et la zone d’intérêt (ROI), le pipeline de couleurs (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, les `PtpSlaveOnly` / `PtpDomainNumber` de PTP, la taille des paquets (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), la cadence (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), les données de bloc (`ChunkModeActive`, chaque `ChunkEnable` sous son `ChunkSelector`), le filtrage des parasites de l'entrée de déclenchement : un tableau fournit la ligne de synchronisation d'un esclave (`LineFilterSelector` / `LineFilterWidth`) ainsi que la fréquence d’acquisition (`AcquisitionFrameRate` uniquement lorsque son activation était activée : lorsqu’elle est désactivée, la fréquence correspond à la valeur maximale autorisée par la cadence, et non à un réglage). Seules les informations différentes sont enregistrées : l’arrêt de la caméra, la cadence et la fréquence d’acquisition avant une exposition nécessitant leur période d’image.

Pour utiliser les caméras en dehors d’un projet, deux arguments écrivent dans la **mémoire propre** de la caméra — celle avec laquelle elle démarre, quel que soit le logiciel (sur `disconnect()` / `force_disconnect()`, toutes les fonctions transitoires, `start_camera_continuous()` et `ChlorosProject.disconnect_all()` ; MCP : les mêmes arguments que pour `disconnect_device` et les outils transitoires ; CLI : `--keep-settings` / `--clear-settings`) :

- **`keep_settings=True`** enregistre les paramètres de chaque caméra tels qu’ils ont été laissés en mémoire à la fin de la session (UserSet2, qui constitue le jeu de paramètres de démarrage). Les caméras continuent de fonctionner en mode libre (`TriggerMode` `Off`), le câblage de la ligne de synchronisation restant tel quel. Une caméra ne disposant pas de UserSet2 enregistre ses paramètres dans UserSet1 ; cet ordinateur en garde la trace, de sorte que le mécanisme d’auto-réparation du réseau d’Chloros — qui réinitialise un UserSet1 s’étant écarté de la configuration de référence d’Chloros — ne la modifie pas.
- **`clear_settings=True`** réinitialise la mémoire de chaque caméra aux paramètres d’usine : la référence de Chloros, celle que la fonction d’auto-réparation de son réseau intègre — paramètres d’usine par défaut avec `TriggerMode`, `Off`, `TriggerSource`, `Line2`, exposition automatique, la balance des blancs calibrée d’une caméra RGB conservée — dans UserSet1, qui constitue le jeu de démarrage.
- Pas les deux (`ValueError` ; le backend renvoie un code 400). La réponse contient `camera_memory`, par caméra `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` ou `{"cleared": bool, "errors": [...]}` — `force_disconnect()` le renvoie ; pour une tâche, il est accompagné de `stop()`.

L’application ne propose pas ces éléments.

**Chaque sortie réinitialise les caméras.**La réinitialisation « telle que trouvée » s’exécute sur chaque chemin de sortie : `CameraSession.disconnect()` (la caméra**X**de l’application), le**X** de l’application sur un tableau ou n’importe quel élément du tableau (le tableau entier est libéré), la fermeture de l’application (le backend dispose de 20 s maximum pour remettre en place toutes les caméras, tous les tableaux et tous les enregistrements continus en cours ; une fonction transitoire ou un appel ponctuel encore en cours s’arrête à son prochain point de sécurité, remet d’abord ses caméras en place, pendant 8 s maximum, puis renvoie une réponse HTTP 503 `SHUTTING_DOWN` ; une fonction `connect_array()` encore en cours d’exécution s’arrête avant sa prochaine caméra et referme les caméras qu’elle a ouvertes, pendant 10 s maximum, puis déclenche `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`) ; et celle qui arrive pendant l’arrêt n’ouvre rien : HTTP 503 `SHUTTING_DOWN`), ainsi qu’une commande `chloros-cli` qui est interrompue ou dont le terminal se ferme (sa tâche est arrêtée et ses caméras sont d’abord remises en place). Un backend qui plante ou qui est forcé de s’arrêter ne peut rien remettre en place ; un redémarrage ramène chaque caméra dans sa mémoire.**Un projet configure lui-même ses caméras.** Lorsqu’un ensemble se connecte via un projet — le projet ouvert de l’application, ou un `ChlorosProject`, qui se nomme lui-même auprès du backend (`project_path`) — chaque membre reçoit les paramètres que le projet a enregistrés pour lui : une exposition, un gain ou une cible AE qui lui est attribuée (considérée comme la vôtre, de sorte que l’exposition automatique ne la modifie pas) et son mode de balance des blancs (correction de Chloros ; la balance des blancs propre à l’appareil photo reste désactivée), quel que soit le contenu de la mémoire de l’appareil.

### Modes de capture et enregistreurs

La surface du tableau reflète le panneau de capture de l’interface graphique : modes d’obturation Unique / Continu / Intervalle / Le plus rapide, plus deux enregistreurs (vidéo composite en direct et rafale RAW → retraitement hors ligne).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**correspond à la boucle « Continu / Intervalle » du SDK. Comme il n’existe pas de commande `Ctrl+C` permettant de l’interrompre depuis un script, vous**devez** passer `count` et/ou `duration_s` (la boucle s’arrête lorsque l’un ou l’autre est atteint). `interval_s` est mesuré à partir du début de chaque passage (comme dans l'interface graphique). Les passages continus (`interval_s=0`) enchaînent `group_seq` → `after_seq` ; il s'agit donc de groupes synchronisés consécutifs au rythme du tableau. Les autres arguments de ligne de commande sont transmis directement à `capture()`.
- **`continuous`**correspond à la capture continue propre au backend — le mode « Continu » de l’interface graphique.**Utilisez-le plutôt que `capture_repeated()` lorsque vous souhaitez que chaque déclenchement soit enregistré sur le disque à une fréquence d’images élevée :** `capture_repeated()` effectue une requête HTTP par groupe à partir de votre script, tandis que `continuous()` maintient deux captures en cours de traitement au sein du backend, sans aller-retour entre les groupes. Mesuré avec une zone d’intérêt (ROI) de 512 sur un réseau de 9 caméras : 73 à 77 captures/s, contre environ 78 pour un réseau. Conservez `capture_repeated()` pour la capture par intervalle (`interval_s`) et pour les passages que vous inspectez un par un. Sous Windows, excluez le backend de Microsoft Defender pour obtenir des débits élevés (voir [Étape 1](#step-1--install-the-chloros-platform-package)).
- **`record`** est de *niveau surveillance* : il enregistre le composite d’index combinés rendu en temps réel. Lorsque la vue combinée de l’interface graphique est ouverte, il enregistre cette vue telle qu’elle s’affiche. Si aucune n’est ouverte, le backend génère lui-même la vue : `expr` si vous en transmettez une, sinon l’NDVI du réseau (les bandes rouges et NIR les plus proches de 660 nm et 850 nm, telles que la vue en direct les détecte), une fois le réseau aligné. Nommez les bandes dans `expr` selon la nomenclature du réseau : `<band>_<filter>` pour une caméra M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` pour une M3M (`Mono_850`) ; un nom de couleur court (`NIR`, `Red`) ne fonctionne que si une bande du tableau en dispose. Une vue de l’interface graphique ouverte pendant l’enregistrement est enregistrée tant qu’elle reste ouverte. Un enregistrement qui n’écrit aucune trame échoue avec les codes d’erreur `error` et `"no-frames: …"` (le tableau peut encore être en train d’aligner ou de stabiliser son exposition), et `stop()` déclenche une exception (voir [`RecorderHandle`](#recorderhandle)). Un enregistreur composite par matrice (déclenche une exception si un autre est déjà en cours d’exécution).
- **`burst` → `build_video`** est de *niveau analyse* : `burst` écrit les images brutes + un manifeste par image + un `.daq` par lecture DLS distincte sous `<output>/bursts/<base>/` à la cadence maximale de la boucle de capture (pas de chaîne, pas d’exiftool, pas de prévisualisation). `build_video` synchronise chaque image avec le `.daq` le plus proche et relance la chaîne de traitement de la radiance et de laréflectance/indice du pipeline d’importation. `products` est une liste de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (par défaut : l’indice combiné). `burst().stop()` lance également automatiquement la construction d’un, renvoyée sous la forme `build_job` dans le résultat d’arrêt.

##### Le résultat `build_video`

| Clé | Signification |
| --- | --- |
| `success` | `True` uniquement lorsque tous les produits demandés ont été générés conformément à la demande : au moins une sortie, rien dans `skipped`, et pas de résultat dans `degraded`. |
| `outputs` | Chemins d’accès des vidéos (et GIF) enregistrées. |
| `errors` | Problèmes, sous forme de texte. |
| `skipped` | Un `{"serial", "level", "reason"}` par produit demandé n’ayant produit **aucune** vidéo — soit en raison d’un problème lié à la caméra (pas de pack d’étalonnage pour cette caméra, pas de lecture DLS pour la réflectance, échec de la chaîne de traitement) ou de l’indice combiné, qui contient `"kind": "combined"`, `serial` et `None` (aucune paire de bandes rouge/NIR parmi les caméras, aucun groupe avec une radiance pour les deux bandes). Toute entrée génère `success` et `False` ; les autres vidéos sont tout de même enregistrées et répertoriées. |
| `substitutions` | Caméras rendues à un niveau différent de celui demandé, et pourquoi — une caméra RGB pour laquelle une radiance/réflectance a été demandée reçoit sa vidéo d’affichage, avec une balance des blancs lorsque le pack d’étalonnage de la caméra est disponible. Ce n’est pas un échec. |
| `notes` | Texte informatif, pas de problème — par exemple, une caméra sans expression d’index spécifique n’a pas de vidéo d’index, ou la vidéo d’affichage d’une caméra RGB n’est pas équilibrée en blanc (pas de pack d’étalonnage, ou un pack sans équilibre des blancs d’usine), avec mention du nom de la caméra. Ce n'est pas un échec. |
| `degraded` | `True` lorsqu'un produit a été défini mais n'est pas celui demandé : l'indice combiné hors ligne est la différence normalisée rouge/NIR intégrée, donc une expression d’indice configurée n’est pas évaluée. Génère `success` `False`. |
| `combined` | Résumé de l’indice combiné : les bandes utilisées, la plage de la table de conversion (LUT), les images générées. |
| `clamp_masks` | Chemins d'accès des [masques de limitation de démixage](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) desimage (`save_tiffs=True`), dans `clamp_masks/<level>/` à côté du dossier `<level>/` des TIFF, un par TIFF dans lequel un pixel a été écrêté. Une vidéo n'en contient aucun. |
| `clamp_fractions` | Ce qu’une vidéo contient à la place d’un masque : un fichier `{"kind", "serial", "level", "frames"}` par produit vidéo (`kind` `"combined"` pour les caméras membres de l’index combiné), `frames` une liste de `[frame, R, G, B]` — la part des pixels de cette image que le démixage a écrêtée dans la bande couverte par son canal R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Renvoyé par `ArraySession.record()` et `ArraySession.burst()`. À utiliser comme gestionnaire de contexte pour s’arrêter automatiquement à la sortie de la portée, ou à piloter manuellement.

| Élément | Description |
| --- | --- |
| `job_id` | ID de la tâche du backend (chaîne). |
| `kind` | `"composite"` (provenant de `record`) ou `"raw"` (provenant de `burst`). |
| `start_stats` | Le dictionnaire renvoyé par l'appel `start`. |
| `result` | `None` pendant l’exécution ; le dictionnaire final du résultat d’arrêt une fois l’enregistrement terminé. |
| `stats(timeout=10.0)` | Statistiques en temps réel de la tâche (images écrites, fps réalisés, temps écoulé). |
| `stop(timeout=60.0)` | Arrêt de l’enregistreur ; renvoie et met en cache le résultat final (`.result` reste `None` si la demande d’arrêt elle-même a échoué). Pour un enregistrement `record()` ayant échoué — c’est-à-dire n’ayant écrit aucune image (`error` `"no-frames: …"`) — il met en cache le résultat sur `.result`, puis déclenche une exception `ChlorosConnectError`. Idempotent (un deuxième appel renvoie le résultat mis en cache, sans déclencher d’exception). Quitter un bloc `with` arrête l'enregistreur sans déclencher d'exception ; vérifiez donc `rec.result.get("error")` après celui-ci. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Renvoyé par `ArraySession.continuous()`. À utiliser comme gestionnaire de contexte — quitter le bloc `with` arrête la capture si elle est toujours en cours — ou contrôlez-la avec `status()` / `wait()` / `stop()`. Une seule capture continue s'exécute par tableau à la fois.

| Élément | Description |
| --- | --- |
| `start_status` | Le dictionnaire d'état renvoyé par l'appel de démarrage. |
| `result` | `None` pendant l’exécution ; le dictionnaire d’état final une fois la capture terminée (défini par `stop()`, ou par un `status()` / `wait()` ayant détecté sa fin). |
| `status(timeout=10.0)` | Dictionnaire d’état en cours (champs ci-dessous). |
| `wait(timeout=None, poll_s=0.5)` | Bloquer jusqu’à ce que la capture se termine d’elle-même — atteinte de `max_captures` ou `duration_s`, tableau vide, disque plein, erreur — et renvoyer l’état final. Avec `timeout`, renvoie le dernier état après ce nombre de secondes, même si l’opération est toujours en cours : voir `running`. |
| `stop(timeout=30.0)` | Arrête la capture et renvoie son état final. Idempotent (un deuxième appel renvoie le résultat mis en cache). |

Champs d’état :

| Champ | Signification |
| --- | --- |
| `running` | Vrai tant que la capture est en cours. |
| `captures` | Groupes synchronisés enregistrés. |
| `files` | Fichiers écrits. |
| `failed_files` | Fichiers qui n’ont pas pu être écrits (comme dans [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Tentatives de capture ayant échoué dans leur ensemble, y compris celles pour lesquelles la capture a été réessayée (erreur HTTP503 alors que la baie n'avait pas de nouveau groupe à temps). |
| `skipped_groups` | Groupes publiés par le tableau qui n’ont pas été enregistrés (la capture a pris du retard). |
| `captures_per_s` | Taux de sauvegarde moyen depuis le début. |
| `elapsed_s` / `started_at` | Nombre de secondes écoulées jusqu’à présent ; heure de début (secondes de l’époque Unix). |
| `last_group_seq` | Dernier groupe synchronisé enregistré. |
| `last_error` | Dernière erreur, ou `None`. |
| `stop_reason` | Une fois terminé : `count` (`max_captures` atteint), `duration` (`duration_s` atteint), `stopped` (`stop()` appelé), `array_gone` (le tableau a été déconnecté), `error` (20 échecs consécutifs, ou une requête rejetée par le backend), `disk_full` (un fichier pour lequel le disque n’avait pas assez d’espace). `None` en cours d’exécution. |
| `max_captures` / `duration_s` | Les limites avec lesquelles il a été lancé (`0` = aucune). |
| `defender_cores` | Nombre de cœurs de processeur utilisés par Microsoft Defender au cours des 10 dernières secondes ; `None` jusqu’à ce qu’une valeur soit mesurée, et toujours `None` sur un hôte sans Defender. |
| `defender_warned` | Vrai dès que la capture a enregistré que Defender la ralentit (à partir d’un cœur). Exclure le backend de l’analyse Defender — voir [Étape 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Trames en temps réel dans votre propre pipeline

Pour transmettre des images synchronisées à votre propre code — apprentissage automatique, visionneuse personnalisée, enregistreur personnel — utilisez `ArraySession.frame_stream()`. Chaque tick génère un `GrabResult` : `{serial: numpy array}` pour chaque caméra, **le tout à partir d’un seul déclencheur**, avec l’identifiant de ce déclencheur sur `.group_id`. Il s’agit de la même implémentation que celle de l’API du projet `ArrayHandle.frame_stream()`, donc tout ce qui se trouve sous [Un déclencheur par lecture](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#alignement-des-flux--align) et [`bit_depth`](#profondeur-de-bit--sacrifier-la-précision-au-profit-de-la-bande-passante) s’appliquent aux deux.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Chaque ensemble généré constitue un nouveau déclencheur.** `frames.group_id` ne se répète jamais au sein d’un flux ; lorsque vous demandez plus de `fps` que la vitesse d’exécution du tableau, le flux attend le déclencheur suivant, de sorte qu’il s’exécute à la vitesse la plus lente des deux.
- **Les caméras manquantes sont nommées, elles ne sont pas remplacées.** Une caméra qui n’a pas pu être prise en charge à ce déclencheur est absente du dictionnaire et répertoriée dans `frames.errors`.
- **Il se termine de manière brutale.** `FrameFetchError` se répète en cas de refus au tick suivant, après 60 s sans image, ou après 60 s sans nouveau déclencheur.**Ce qu’il offre.**`frame_stream()` transfère des images Numpy entières via la connexion HTTP locale et atteint un débit maximal de près de**170 Mo/s**. Cela suffit pour le traitement en temps réel d’images brutes, ou d’un tableau regroupé ou d’un tableau de régions d’intérêt (ROI), mais pas pour chaque déclenchement en pleine résolution — pour conserver**chaque** déclenchement, enregistrez sur le disque avec [`continuous()`](#continuoushandle). Mesures effectuées sur un réseau de quatre caméras (FOCN, FRGN, FRGB, F450) avec transport compressé ; chaque tick correspondait à un ensemble complet d’un déclenchement :

| | Capteur complet (matrice 14,2–14,4 ips) | Regroupement 2 × 2 (matrice ~33 ips) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 séries/s | 22,2 séries/s |
| `frame_stream(processing="debayered")` | 2,3 séries/s | 9,3 séries/s |
| `frame_stream(processing="display")` | 0,8 série/s | 3,0 séries/s |
| MJPEG par caméra, 4 flux en 720p | 13,5 images par seconde chacun ; chaque image correspond à un déclencheur | ~25 images par seconde chacun ; ~45 % des déclencheurs présents sur les 4 |

Utilisez `processing="raw"` pour le débit et le dématriçage de votre côté ; le niveau `debayered` supprime également le décalage sombre du pack d’étalonnage, ce qui n’est pas le cas d’un dématriçage de niveau `raw`.

#### Flux MJPEG par caméra (pas de RTSP)

**Le protocole RTSP n’est pas pris en charge.** L’interface de diffusion est MJPEG sur HTTP (`multipart/x-mixed-replace`, un fichier JPEG par partie, chaque partie avec un `Content-Length`), une URL par caméra :

| URL | Ce qu’elle sert |
| --- | --- |
| `GET /api/camera/stream/<serial>` | Aperçu 8 bits, jusqu’à environ 30 images par seconde. Pour un élément du réseau, `max_h` (valeur par défaut 720, 64–2160) et `jpeg_q` (valeur par défaut 80) définissent la taille et la qualité. |
| `GET /api/camera/<serial>/stream-annotated` | Le même aperçu avec les superpositions de l’interface graphique (zèbre, grille, réticule, histogramme, peaking, posemètre spot). `ChlorosLocal.camera_stream()` le lit pour vous. |
| `GET /api/camera/<serial>/stream-reflectance` | Réflectance de 8 bits par bande — voir [Flux de réflectance en direct](#live-reflectance-stream-mjpeg). |

- **En-têtes de déclenchement.** Pour un élément du tableau, chaque partie des trois flux contient `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` et `X-Chloros-Timestamp-Ns`. Les parties provenant de caméras différentes mais portant le même `X-Chloros-Group-Id` constituent le même déclencheur : collectez les parties et regroupez-les en fonction de cet en-tête pour reconstituer un ensemble. Une caméra autonome n’a pas de groupe : ses parties de prévisualisation ne comportent aucun de ces en-têtes (ses composants de réflectance portent un en-tête `X-Chloros-Group-Id` vide).
- **Le flux de prévisualisation perd des images en cas de charge élevée.** Il diffuse l’image la plus récente et ignore celles qu’il n’a pas pu envoyer à temps ; ainsi, à des fréquences de balayage élevées, tous les déclencheurs n’atteignent pas le flux de chaque caméra (voir le tableau ci-dessus). Pour chaque déclenchement, utilisez `frame_stream()` ou `continuous()`.
- **`/api/camera/stream/<serial>` se met en pause lorsque le flux en direct combiné du réseau est ouvert** (`/api/camera/array/stream/<array_id>` — la vue « réseau » de l’interface graphique l’ouvre). Le flux de chaque caméra envoie alors une image de remplacement « Paused (array stream active) », sans en-têtes `X-Chloros-*`, et rien d’autre jusqu’à la fermeture du flux combiné ; il reprend alors de lui-même. `stream-annotated` et `stream-reflectance` ne se mettent pas en pause. Fermez la vue « Array » de l’interface graphique lorsqu’un script lit les flux de prévisualisation par caméra.
- **Pour les chiffres, et non les images, utilisez `frame_stream()`.**Le format JPEG est avec perte et en 8 bits ; `frame_stream()` renvoie les mêmes résultats sans codec dans le chemin d’accès.

### Capture à intervalles réguliers pour les levés par drone

Un vol de levé nécessite un groupe d’images synchronisées tous les quelques mètres parcourus, et non pas chaque image que la matrice est capable de produire. Deux paramètres contrôlent cela :**`target_fps`**définit la fréquence à laquelle les caméras prennent une image, et**`capture_repeated(interval_s=…)`** définit la fréquence à laquelle une image est enregistrée.**Les caméras continuent de filmer entre deux enregistrements.** Chaque intervalle enregistre le dernier groupe synchronisé ; les images intermédiaires ne sont pas enregistrées. L’exposition automatique s’ajuste à chaque image, de sorte que l’exposition est déjà stabilisée au moment de l’enregistrement. Il n’existe aucun mode dans lequel les caméras s’arrêtent entre deux prises et sont réactivées pour chacune d’entre elles : chaque prise devrait alors attendre que le flux commence et que l’exposition se stabilise.**Fixe la fréquence d’images en fonction des besoins de l’intervalle.**Si tu la laisses en mode automatique, un réseau fonctionne à la fréquence la plus élevée que l’hôte et le réseau peuvent supporter. La fixer à une valeur inférieure réduit la chaleur dégagée par les caméras, la charge réseau et l’utilisation du processeur de l’hôte sans modifier ce qui est enregistré. Choisis une fréquence qui donne**2 à 4 images par intervalle**:

- Chaque enregistrement attend l’image suivante après sa demande ; ainsi, un groupe d’images enregistrées est stocké jusqu’à une période d’image après le moment prévu. Une fréquence plus élevée réduit cet écart.
- L’exposition automatique s’ajuste une fois par image ; ainsi, une fréquence très faible ralentit la réactivité de l’exposition face au passage d’un nuage.

**Calculez l’intervalle à partir du plan de vol :**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Par exemple, une empreinte de 60 m avec un chevauchement avant de 80 % correspond à un espacement de 12 m ; à 10 m/s, cela donne `interval_s=1.2`, et un `target_fps=3` fixe donne entre 3 et 4 images par intervalle. La capture est basée sur le temps : le SDK ne dispose pas de mode déclenché par la distance ou par GPS, il faut donc maintenir une vitesse au sol constante sur les lignes de levé.

**Enregistrez les données brutes à bord de l’aéronef et effectuez l’étalonnage ultérieurement.** `interval_s` est mesuré à partir du début de chaque enregistrement ; ainsi, un enregistrement qui dure plus longtemps que l’intervalle entraîne un démarrage tardif du suivant et une dérive de l’espacement. L'enregistrement des données brutes ainsi que de la lecture du capteur de lumière (`processing="raw"` ; la lecture est enregistrée par défaut) permet de limiter la durée de chaque enregistrement et de garantir que chaque image reste convertible en réflectance par la suite.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fixe le débit dès le début ;**`arr.set_target_fps(fps)`**le modifie en cours de vol sans se reconnecter, et**`arr.set_target_fps(None)`** le ramène en mode automatique. Un débit supérieur à ce que l’hôte et le réseau peuvent supporter est réduit à ce plafond — vérifiez `clamped` dans le résultat — et le débit le plus bas est de 0,1 image par seconde.
- **N’utilisez pas `burst()` pour réduire le nombre d’images.** Il écrit chaque image produite par le tableau ; il suit un `target_fps` fixe, ce n’est donc l’outil approprié que lorsque toutes les images sont souhaitées.
- **`target_fps` ne corrige pas les trames corrompues.** Le réduire diminue le nombre d’images produites, et non la vitesse à laquelle chacune est envoyée ; pour les images corrompues par GVSP, réduisez plutôt la valeur de `wire_ceiling_mbps` (voir [État de santé du réseau de cartes](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Se connecter à un tableau déjà connecté — `attach_array`

Si le tableau est déjà actif (la GUI l’a ouvert, ou une session SDK précédente a appelé `connect_array`), utilisez `attach_array` pour obtenir un descripteur vers celui-ci au lieu de vous reconnecter. `connect_array` renvoie toujours l’erreur « La caméra  <sn>fait déjà partie du réseau <id>», car l’envoi d’une requête POST avec `/array/connect` pour un membre du pool n’est pas idempotent ; `attach_array` lit `/api/camera/array/list` et effectue la correspondance soit par array_id, soit par numéros de série.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Modèle : les scripts SDK qui fonctionnent en co-tenancy avec l’interface graphique de bureau doivent d’abord essayer `attach_array` et se rabattre sur `connect_array` si aucun tableau ne se trouve encore dans le pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Un tableau attaché est emprunté, il n’est pas détenu en propriété.** Un descripteur provenant de `attach_array` ne libère jamais la matrice : ni la sortie du bloc `with`, ni un appel explicite à `disconnect()` ne la libèrent (l’appel explicite affiche une ligne sur stderr indiquant qu’il a laissé le tableau en cours d’exécution). Même règle que pour `CameraSession` / `DAQSensorSession`. Le partage avec l’interface graphique est donc sans risque, avec ou sans `with` :
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Lorsque vous souhaitez réellement y mettre fin — pour vous reconnecter avec un autre `wire_ceiling_mbps`, par exemple — appelez `arr.force_disconnect()`. Une reconnexion après un simple `disconnect()` sur un descripteur attaché échoue avec le message « La caméra  est<sn> déjà dans le tableau <id>», car rien n’a été libéré.

### Aide à l’analyse réseau

Utile avant d’ouvrir le tableau — permet de vérifier si les paramètres proposés sont compatibles :

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Il ne lance jamais de backend : un backend qui ne répond pas déclenche l’erreur `ChlorosConnectError`, tandis qu’une requête refusée est renvoyée comme résultat. Une erreur `packed` déclenche l’erreur `ValueError`.

`status` est l'un des codes suivants : `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (sinon `error` — y compris une requête refusée : connexion non établie, `error_code: AUTH_REQUIRED`, ou un forfait inférieur au seuil minimum du SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` signifie que la résolution demandée ne s’adapte à l’anneau RX qu’à une fréquence de déclenchement plafonnée — conservez la résolution et passez de `target_fps=result["recommended"]["recommended_target_fps"]` à `connect_array` (voir [Exemple 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` n’apparaît que lorsque l’image demandée ne peut être fournie à aucune fréquence d’images ; la connexion ne réduit jamais les images de son propre chef, vous devez donc appliquer `recommended` vous-même.

**Comment interpréter la projection** (même modèle que le panneau « Array Settings » de l’interface graphique) :

- **La rafale (`frame_bytes_total`) est additionnée par caméra au format de pixels réel de chaque caméra.**Les caméras**M3M**mono transmettent en Mono12 (2 bits par pixel), ou, avec un `pixel_format`, le format mono de leur profondeur (`"BayerRG8"` correspond à `Mono8`, 1 bit/pixel) ; ainsi, sans celui-ci, une image en pleine résolution à 4 caméras pèse**environ 25 Mo** avec trois caméras mono, et non les ~12,6 Mo que donnerait une hypothèse de 8 bits partout. Le backend détermine le format de chaque caméra à partir de son modèle.
- **L’admittance (`burst_fits_nic_ring`) tient compte du débit de sortie**, et non pas « rafale complète » par opposition à « anneau » : le mode « sim-emit » s’applique lorsque l’hôte vide l’anneau de réception plus rapidement que les caméras ne le remplissent. Un hôte 10G associé à des caméras 1 GbE**admet** la pleine résolution même lorsque la rafale dépasse la capacité de l’anneau ; un hôte 1 GbE bloque (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` correspond à une limite supérieure prudente pour la récupération en série** — `max(readout+emit, N×emit)` avec l’émission par caméra limitée au lien de caméra 1 GbE, indépendamment de l’exposition. Par exemple, environ 2,8 images par seconde pour un réseau de 4 caméras en pleine résolution 12 bits (ce qui correspond aux valeurs mesurées par le runtime, comprises entre environ 2,7 et 3,0). Modèle complet : [Référence CLI → Modèle de fps et de rafales pour les réseaux de caméras](cli-reference.md#array-fps--burst-model).
- **Après une capture continue, la projection indique la valeur atteinte.** Une capture de 30 s ou plus à une fréquence automatique enregistre la fréquence à laquelle le système s’est stabilisé pour cette taille de réseau sur cet ordinateur ; les brefs essais du contrôleur de fréquence à une cadence plus rapide sont ignorés. À partir de là, la ligne `array-connect`', l’analyse de `connect_array()` et le panneau « Paramètres du tableau » indiquent ce taux lorsqu’il est inférieur à celui du modèle, en précisant ce qui l’a limité dans `cycle_limiter` : `"clock"` (le réseau a fonctionné à la fréquence à laquelle il s’est stabilisé, la boucle de capture attendant son signal d’horloge de déclenchement), `"loop"` (la boucle de capture n’avait plus de temps), `"capture"` (la capture a pris du retard et a sauté des groupes) ou `"measured"` (rien n’indique lequel). La valeur du modèle figure dans `wire_fps_max`. Le tableau continue d’augmenter jusqu’à atteindre la capacité du canal de transmission.
- **La sursouscription (`oversubscribed: true`) signifie que le canal de transmission ne peut pas acheminer le tableau, quelle que soit la cadence atteinte par les caméras** — les champs fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) affichent 0, et le binning ne peut pas y remédier (celui-ci réduit le nombre d’octets par image, et non le nombre d’octets synchronisés par seconde). Une connexion lente ne provoque pas ce phénomène à elle seule : la fréquence d’images diminue à la place, et si les caméras ne peuvent pas être synchronisées à une cadence suffisamment lente, le niveau passe à `slip-emit-and-capture` (série). Les solutions consistent à réduire le nombre de caméras, à utiliser des trames jumbo ou à installer une carte réseau plus rapide ; `max_cams_collision_safe` indique la limite maximale de votre liaison et de vos caméras. La réponse contient également les codes `aggregate_demand_bps`, `collision_safe_ceiling_bps` et `per_cam_floor_bps`. Voir [Sursouscription](#over-subscription-the-per-cam-floor).

### Découverte et répertoriage

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Chacune accepte uniquement des mots-clés (`backend_url=` et `timeout=30.0`) et renvoie une liste de dictionnaires. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) prend également `auto_start_backend=True` ; les appels `list_*` ne lancent jamais de backend.

---

## Smart-AE / Smart-Capture

Les matrices LATTICE exécutent l’AE en continu en arrière-plan dès qu’elles sont connectées, mais une scène nouvellement orientée met un certain temps à converger. **Smart-capture** est la solution pratique intégrée : elle interroge l’exposition de chaque caméra, attend que le réseau soit stable sur une fenêtre, puis déclenche la capture. Elle est équivalente à l’interface graphique : le bouton de capture « smart » de l’application de bureau appelle le même point de terminaison du backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Lorsque vous utilisez `ChlorosProject` (section suivante), vous disposez de plus d’options de réglage :

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

La politique d’exposition automatique « smart-AE » est conservatrice par défaut. Resserrez le paramètre `exposure_tolerance_pct` pour les travaux radiométriques exigeants ; assouplissez-le pour les scènes évoluant rapidement où vous visez simplement une précision « suffisante ».

---

## Sessions de capteurs DAQ

Pool de backend persistant pour les capteurs spectraux (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Reflète la surface de la caméra : détection intelligente, réutilisation du pool, attachement idempotent.

### Détection intelligente (configuration zéro)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Ordre de priorité : Ethernet → BLE → USB. Passez n’importe quelle indication explicite pour verrouiller le transport.

### Transport verrouillé

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` désigne un capteur, et seul ce capteur est ouvert. Il est mis en correspondance avec les capteurs déjà ouverts, puis le nom d’hôte ou l’adresse IP d’un DAQ-E (un DAQ-E que l’analyse du réseau ne détecte que par son adresse IP est interrogé, en lecture seule, pour obtenir son nom, son adresse MAC et son numéro de série, de sorte que n’importe lequel de ces éléments permet également de le localiser), l’adresse MAC BLE d’un DAQ-M, ainsi que l’identifiant qu’un DAQ-U ou un DAQ-M renvoie une fois ouvert (tout autre appareil ouvert à des fins de vérification est refermé). S’il n’est pas détecté, `connect_daq_sensor` déclenche `ChlorosConnectError` en répertoriant ce qui a été trouvé ; avec une adresse également (`port=` / `mac=` / `eth_host=`) correspondant à un capteur différent, une exception est également levée. La fonction ne se rabat jamais sur le premier capteur visible — c’est à cela que sert l’appel avec le drapeau zéro.</id></sn></id></sn>

### Signature `connect_daq_sensor()`

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Un capteur qui se trouve déjà dans le pool est restitué tel quel (emprunté) : les codes `integration_time`, `frame_avg` et `enable_ae` ne s’y appliquent pas. Déconnectez-le d’abord pour l’ouvrir avec vos paramètres. `cap_id` constitue l’exception : sur un capteur emprunté, il est défini avec `set_cap()`. S’il est omis, la valeur par défaut du backend s’applique (voir la remarque sous le tableau des méthodes).

### Méthodes `DAQSensorSession`

| Méthode | Description |
| --- | --- |
| `status(timeout=10.0)` | Résumé des entrées du pool (état de streaming/enregistrement, plage de longueurs d’onde, sha d’étalonnage, temps d’intégration, frame_avg, état AE). Pour un DAQ-E dont le flux s’est interrompu : `reconnect` (lorsque la reconnexion automatique du backend est active : `state`, `attempts`, `next_attempt_in_s`, `last_error` ; `None` tant qu’il diffuse), `stream_lost_s`, `reconnects` et `last_reconnect`, ainsi que `record_gaps` pour l'enregistrement en cours ; `rx_stream_fault` indique l'état de reconnexion jusqu'à l'arrivée du premier spectre. |
| `latest(*, n=1, timeout=10.0)` | Renvoie jusqu’à N trames de spectre les plus récentes (chacune `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Reprendre / mettre en pause la diffusion en continu (le descripteur reste ouvert). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Démarrer l'enregistrement d'un fichier .daq. Renvoie le chemin d'accès au fichier. Refusé pour les DAQ-U/M sans pack d'étalonnage AWS (le DAQ-E fait exception). |
| `record_stop()` | Arrête l'enregistrement. Renvoie `{path, rows, warning, gaps}` ; `warning` correspond à `None` pour un enregistrement correct, sinon le diagnostic du backend (par exemple un fichier vide ou une interruption dans le flux), qui est également affiché sur stderr. `gaps` répertorie chaque segment où le flux n’a rien fourni : `start_utc`, `end_utc` (`None` : toujours perdu à l’arrêt), `duration_s`, `reconnected`, `marked_in_file`. Il est vide en cas d'enregistrement continu. |
| `set_cap(cap_id, timeout=15.0)` | Définit le profil de correction de crête appliqué aux mesures à partir de la suivante : `"sunshine_cosine"`, `"fov_15"` … `"fov_90"`, ou `"none"` (ou `None`) pour un capteur nu. Renvoie le, dont la valeur `cap_id` indique la lecture. Un DAQ-E-S conserve la valeur `as_recorded` quelle que soit la requête (son diffuseur est intégré). Génère la valeur `ChlorosConnectError` pour un profil qui ne se charge pas. CLI : `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | DAQ-E uniquement : réactive le PTP pour un appareil dont le PTP est désactivé (`status()["rx_ptp_enabled"]` correspond à `False` ; le paramètre est enregistré sur l’appareil, qui reste donc désactivé jusqu’à ce qu’il soit réactivé). Renvoie l’état actualisé. Déclenche `ChlorosConnectError` pour un DAQ-U / DAQ-M, ou lorsque l’appareil ne confirme pas. Chloros ne réactive jamais un appareil de lui-même, car la désactivation peut être intentionnelle. CLI : `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | DAQ-E uniquement : les paramètres propres à l’appareil. `firmware` (`running`, `bundled`, `update_available`), `ip` et `ip_mode` (`dhcp` / `linklocal` / `static` : ce qui est actuellement en cours d’exécution), `network` (le paramètre enregistré : `mode`, plus `ip` / `mask` / `gateway` en mode statique), `network_change_pending` (vrai jusqu’à ce que l’appareil exécute le paramètre enregistré, celui-ci incluant `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` correspond à `None` lorsque laversion de l’appareil n’a pas pu être lue. Tant que le flux de l’appareil n’atteint pas cet ordinateur, il est lu lors d’une nouvelle connexion et le dictionnaire ajoute `stream_silent` (`True`), `stream_fault` et `reconnect` ; un appareil qui ne répond pas déclenche l'erreur « CLI : `pool-info` ». |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E uniquement : DHCP (`"auto"`) ou une adresse statique. Redémarre l'unité pour appliquer la modification et la reconnecte ; cette session continue de fonctionner et `address` suit l'unité. `reboot=False` se contente de l'enregistrer. Voir [Paramètres de l’appareil DAQ-E](#daq-e-device-settings). CLI : `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | DAQ-E uniquement : diffusion dès la mise sous tension sans logiciel connecté (enregistré sur l’appareil). Renvoie le paramètre confirmé. CLI : `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | DAQ-E uniquement : redémarre l’appareil et le reconnecte avec ses paramètres d’origine. CLI : `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | DAQ-E uniquement : flashe le micrologiciel fourni avec cette version d'Chloros sur un appareil fonctionnant sous une version antérieure, puis reconnecte-le (environ une minute ; ne pas le mettre hors tension). Renvoie `updated`, `fw_before`, `fw_after`, `bundled` et un `warning` lorsque l’appareil a mis du temps à répondre ou a répondu avec une autre version ; `updated` correspond à `False` pour un appareil déjà à jour. Génère une exception `ChlorosConnectError` pour un périphérique dont la version ne peut pas être lue. CLI : `pool-update-firmware`. |
| `disconnect()` | Libération du pool. Pas d’-op pour les descripteurs attachés mais non possédés. Sur un capteur ouvert par cette session, c’est `force_disconnect()`, et déclenche `ChlorosConnectError` lorsque la libération n’est pas confirmée (la sortie du bloc `with` ne déclenche jamais d’exception). |
| `force_disconnect(timeout=30.0)` | Libération du capteur, quel que soit l’utilisateur qui l’a ouvert — par exemple un capteur laissé dans le pool par un script qui a planté, auquel un nouveau script n’accède qu’en tant que descripteur attaché dont le code `disconnect()` ne fait rien. La fermeture du capteur d’un autre client met fin à tout enregistrement en cours. Retourne une fois que le backend a confirmé la libération, ou lorsqu’il n’y a rien à libérer (le capteur n’est pas dans le pool, ou aucun backend n’est en cours d’exécution) ; une libération ayant expiré ou refusée par le backend déclenche `ChlorosConnectError`. CLI : `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attributs : `sensor_id`, `model`, `transport`, `address` (suit un DAQ-E que `set_network()` a réadressé), `info` et `backend_url`.

> **Nommez le capuchon.** Les DAQ-U, DAQ-M et DAQ-E sont livrés sans condensateur, et Chloros ne peut pas déterminer s’il y en a un : un capteur connecté sans `cap_id` obtient la valeur `sunshine_cosine`, une hypothèse que ses enregistrements indiquent avoir (`cap_id_source=auto_default`). Un DAQ-U nu, sous cette correction, affiche une valeur environ 30 fois trop élevée ; il faut donc passer `cap_id="none"` pour un capteur nu (ou `set_cap("none")`), et le profil du capuchon que vous avez monté dans le cas contraire.

### DAQ-E-S — diffuseur fixe et capteur d’inclinaison

Le **DAQ-E-S** est un DAQ-E présentant deux différences importantes pour le logiciel. Tout le reste — détection, connexion, PTP, enregistrement, paramètres de l’appareil ci-dessous — fonctionne exactement comme pour un DAQ-E.**Son diffuseur est fixe, il ne nécessite donc aucune correction de capuchon.** Le diffuseur de type « Sunshine » est intégré et était déjà en place sur l’appareil lors de son étalonnage en usine ; cet étalonnage indique donc déjà l’irradiance descendante réelle. Chloros définit son cap sur `as_recorded` (sans profil) quelle que soit la demande : un cap portant son nom (`sunshine_cosine`, `fov_*`, `none`) est ignoré et génère un message d’erreur, et ses enregistrements stockent la valeur `cap_id_source=model` ; l’avertissement de « cap supposé » ne se déclenche donc jamais pour cet appareil. N’appliquez pas non plus manuellement de limite à ses données : une limite `sunshine_cosine` au maximum donnerait une valeur environ 11 fois trop élevée.

**Il est équipé d’un accéléromètre à 3 axes.** La réponse cosinus du diffuseur de lumière n’est valable que lorsqu’il est à l’horizontale ; ainsi, un capteur incliné renvoie un irradiance erroné — et la réflectance hérite de cette erreur — sans autre symptôme. Chaque lecture DAQ-E-S comporte donc son orientation :

| Champ | Signification |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Angles en degrés. `None` lorsque le micrologiciel signale un angle indéfini (chute libre, axe inactif). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Accélération en milli-g. `mag_mg` : une valeur éloignée de ~1 000 indique que l’appareil était en mouvement et que les angles ne sont pas significatifs pour cet échantillon. |
| `cal_applied` | Activé : les axes et les angles sont dans le repère du diffuseur (l'étalonnage de montage de l'appareil est appliqué). Désactivé : repère brut de la carte, pouvant afficher jusqu'à environ 2,3° d'inclinaison qui n'est pas réelle. |
| `sample_age_ms` | Âge de l'échantillon de l'accéléromètre au moment de l'envoi du spectre. |
| `temp_raw` | **Tendance** de la température de la puce non calibrée — différences uniquement, jamais une lecture du thermomètre. |

`DAQSensorSession.latest()` les renvoie sous la clé `imu` de chaque trame (`None` sur un DAQ-U, DAQ-M ou DAQ-E standard). Une valeur `.daq` enregistrée à partir d’un DAQ-E-S stocke les mêmes valeurs dans ses colonnes `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) à côté de chaque spectre, ce qui permet au lecteur de distinguer un nuage d’un trépied renversé. Sur tous les autres modèles, ces colonnes sont vides.

Le flux d’attitude rend compte de ses propres champs dans `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), distincts des champs spectraux, de sorte qu’une panne d’accéléromètre ne soit jamais interprétée comme une lacune dans l’enregistrement. `discover_daq_sensors()` et le fichier `model` de la session indiquent `DAQ-E-S`.

### Paramètres du périphérique DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Une adresse statique doit exister sur le réseau de cet ordinateur.** `set_network` génère une erreur `ChlorosConnectError` si le sous-réseau ne comprend pas l’sur le réseau du capteur : après le redémarrage, aucun élément ne pourrait atteindre le capteur pour rétablir la configuration initiale. Ne passez `force=True` que lorsque cet ordinateur sera réadressé ou redirigé vers le nouveau sous-réseau.
- **`set_network`, `reboot` et `update_firmware` redémarrent le capteur** et déclenchent `ChlorosConnectError` pendant qu’il enregistre, et s’il ne revient pas dans un délai de `comeback_timeout` secondes. L’ID du capteur et le nom d’hôte `daq-e-<id>.local` ne changent jamais.
- **Un enregistrement lancé juste avant le redémarrage est clôturé par celui-ci et nommé.** Lorsque le capteur revient, le dictionnaire renvoyé contient `recording_closed` (le chemin d’accès au fichier) et un `warning` l’indiquant. S’il ne revient pas, ou si le transfert du micrologiciel échoue, le `ChlorosConnectError` l’indique à la place : son message se termine par l’avertissement, et `e.recording_closed` / `e.warning` sont activés (ainsi que `None` lorsqu’aucun enregistrement n’a été clôturé). Relancez l’enregistrement une fois que le capteur est de nouveau opérationnel.
- **Seul le capteur lui-même est reconnecté.** Après le redémarrage, l’appareil doit répondre à l’endroit où il se trouvait -- et, après `set_network("auto")`, également sous son nom `.local` -- en indiquant son nom, son nom d’hôte et son modèle, à partir de l’adresse MAC de sa carte réseau ; `address` devient l’adresse IP qui a répondu. Une réponse sous ce nom n’est prise en compte qu’avec cette adresse MAC ; ainsi, depuis un ordinateur qui accède au capteur via un routeur, ou depuis le réseau NAT par défaut de WSL2, un capteur déplacé par le DHCP n’est pas reconnecté (`ChlorosConnectError` ; reconnectez-le à sa nouvelle adresse).
- **Une mise à jour du micrologiciel nécessite que le capteur se reconnecte.** `update_firmware()` invite l’appareil, et celui-ci ouvre une connexion TCP vers cet ordinateur pour récupérer l’image, sur un port choisi par le système d’exploitation. Le backend écoute sur la carte réseau qui communique avec l’appareil (sur toutes les cartes uniquement lorsqu’il ne peut pas déterminer laquelle) et ne dessert que cet appareil : toute autre connexion est refusée (`[DAQ-FW] … refused a firmware connection from <ip>` dans la sortie du backend) tant qu’il reste en attente. Un pare-feu bloquant la connexion fait échouer la mise à jour avec le code `device never connected back on TCP <port>`. Le programme d’installation Windows ajoute la règle correspondante (voir [Étape 1](#windows-exe)) ; sur un autre pare-feu, autorisez le trafic TCP entrant vers le backend depuis le sous-réseau du capteur.
- **Sous WSL2, utilisez la mise en miroir du réseau** (`networkingMode=mirrored` dans `.wslconfig`) : en mode NAT par défaut, toutes les adresses statiques sont refusées (l’adresse propre à WSL ne se trouve pas sur le réseau du capteur), le flux n’atteint pas WSL et la mise à jour du micrologiciel ne peut pas aboutir.
- L’interface graphique maintient sa propre connexion avec un capteur : un redémarrage via un script met fin à la session de l’interface graphique avec ce capteur. L’interface graphique dispose des mêmes paramètres (onglet « Capteurs de lumière », paramètres du capteur, Réseau).

### Découverte — trouver une adresse pour se connecter

`discover_daq_sensors()` analyse les ports USB / BLE / ETH à la recherche de capteurs que vous *pourriez* ouvrir. Il s’agit de l’équivalent DAQ de `discover_lattice_cameras()`, et du seul moyen d’obtenir le **MAC BLE d’un DAQ-M** — un DAQ-E dispose d’un nom d’hôte et un DAQ-U d’un port COM, mais l’adresse MAC n’est ni imprimée sur l’appareil ni répertoriée par le système d’exploitation.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Champ | Description |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / MAC BLE / nom d'hôte — transmettre à `connect_daq_sensor` en tant que `port=` / `mac=` / `eth_host=`. |
| `display` | Étiquette lisible par l'utilisateur. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, ou `None`. Un capteur Ethernet est de type `DAQ-E-S` ou `DAQ-E`, car il s'identifie ainsi lorsqu'on lui demande, en lecture seule, pendant le balayage (`DAQ-E` s’il ne répond pas à temps). Un port série USB affiche `DAQ-U` lorsque sa puce de pont est l’une de celles utilisées par un DAQ-U (FTDI, CH340, CP210x, Prolific) — un indice tiré de la description du port, et non d’une sonde — et `None` pour tout autre périphérique USB, qui figure toujours dans la liste et que vous pouvez donc transmettre sous la forme `port=`. Un port série sans identifiant de fabricant USB et sans puce de pont de ce type (un UART de carte mère tel que `/dev/ttyS*` sous Linux, une liaison série Bluetooth, Intel AMT) ne peut pas être un DAQ-U et n’apparaît pas dans la liste. |
| `extra` | Détails par transport (nom annoncé BLE ; USB `vid`, `pid`, `manufacturer`, `serial_number` ; DAQ-E ip/fw/…). Les valeurs vides sont omises. |

| Paramètre | Valeur par défaut | Description |
| --- | --- | --- |
| `transports` | les trois | Séquence (ou chaîne CSV) limitant le balayage. À spécifier lorsque vous savez ce que vous voulez — le BLE est le maillon faible. |
| `scan_timeout` | 5 | Fenêtre de balayage par transport en secondes ; le backend limite cette valeur entre 1 et 20. |
| `timeout` | 60,0 | Plafond HTTP pour l’ensemble de l’appel (comme ailleurs dans le SDK). |
| `auto_start_backend` | `True` | Lance un backend local si aucun n’est en cours d’exécution. Ne lance jamais de backend pour un `backend_url` distant. |

> **Les capteurs déjà ouverts dans le pool n’apparaissent pas.** Un périphérique BLE connecté cesse de diffuser et un port COM ouvert ne peut pas être détecté ; la découverte répertorie donc ce qui est *disponible pour la connexion*. Un résultat vide juste après avoir connecté un périphérique est normal — utilisez `list_daq_sensors()` pour ce que vous possédez déjà. Les transports dont le balayage ne peut pas s’exécuter (bleak / zeroconf non installés) sont ignorés plutôt que de générer une exception, de sorte qu’une machine sans Bluetooth obtient tout de même ses réponses USB et ETH.

### Liste

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Quel DAQ alimente quelle caméra — `daq_bindings()`

La réflectance d’une capture, ainsi que le fichier `.daq` qu’elle enregistre, utilisent le DAQ associé à cette caméra ou à ce réseau de caméras, ou bien le DAQ le plus récent dont dispose le backend. La sélection du capteur de lumière dans l’interface graphique et le fichier `chloros-cli daq pool-bind` le définissent ; tout comme :

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` renvoie `{"cameras": {serial: b}, "arrays": {array_id: b}}` : chaque `b` contient `binding` (la valeur définie), `source` (`project`, provenant de l’`sensors.json` ou de l’`runtime` du projet), `known` / `known_as` (si la clé « caméra » ou « tableau » désigne un nom connu d’Chloros, et comment : `connected`, `project` — le `cameras.json` du projet ouvert — ou, pour une caméra, `seen` : déjà ouvert sur cet ordinateur ; `known: False` pour une clé qui ne désigne rien) et `resolved` (le capteur de pool auquel il est désormais associé : `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Une liaison à l’exécution remplace celle du projetpour la session ; le fichier de projet n’est pas modifié. La déconnexion d’une caméra, d’un tableau ou d’un DAQ supprime toutes les liaisons associées à ce matériel, quelle que soit la personne qui les a définies (celles des éléments du tableau sont associées à celles du tableau).

`set_daq_bindings()` ne lie qu’une caméra connectée, présente dans le projet ouvert ou déjà ouverte sur cet ordinateur, et un réseau connecté ou présent dans le projet ouvert (ou une touche déjà liée). Toute autre touche déclenche `ChlorosConnectError` en la nommant (`400`, `code`, `UNKNOWN_BINDING_TARGET`, avec les clés `unknown`), et **rien** dans cet appel n’est lié. `None` efface toute clé.

### Réflectance ambiante + objet — `light_sensor_reflectance()`

Deux DAQ mis en commun, l’un orienté vers le ciel (ambiant) et l’autre vers la cible (objet), fournissent la réflectance de la cible et les indices de végétation à partir de leurs dernières mesures : la paire « Ambient + Object » de l’interface graphique, et `chloros-cli daq reflectance` sans avoir à ouvrir les capteurs une deuxième fois. Les deux doivent être connectés et en cours de transmission.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bandes.** La réflectance de chaque bande correspond à l’irradiance intégrée de la bande de l’objetsur celle de l’environnement. `daq_band_presets(filter_code=None)` répertorie les ensembles de bandes : un par filtre LATTICE M3C (`RGN`, `OCN`, `NGB`, le modèle de bandes propre à la caméra ; ainsi, un NDVI DAQ et un NDVI de caméra correspondent à la même grandeur) et `GENERIC` à large bande. Son `suggested` correspond au préréglage `filter_code`, sinon au filtre de la première caméra connectée (`suggested_from` indique lequel).
- **Renvoie** `bands` (par bande : `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (les), `custom` (chacun avec `value`, ou `valid: False` et `error`), `invalid_bands`, `reflectance_type`, `preset` et les deux mesures `timestamps` (`ts_ns`). Une plage que le capteur ambiant n’a pas pu mesurer (hors de sa plage de mesure, au niveau de son seuil de bruit) est considérée comme invalide, avec indication du motif, et tous les indices qui en dépendent sont omis, sans jamais être calculés par rapport à zéro.
- **Apparents ou relatifs.** Les indices dépendants de l’échelle (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) nécessitent un rapport absolu ; ils ne sont donc disponibles que lorsque les deux capteurs sont étalonnés en puissance (chacun dispose d’un `calibration_bundle_sha`) : `"Apparent Reflectance"`. Sinon, le résultat est `"Relative Reflectance"`, avec uniquement les indices invariants d’échelle.
- **Deux grilles de longueurs d’onde différentes** sont comparées au niveau de leur chevauchement : les points du capteur ambiant situés dans la plage de l’objet, le spectre de l’objet interpolé sur ces points.
- **Deux capteurs, jamais un seul.** Attribuer le même nom à un capteur (le même identifiant, quelle que soit l’orthographe acceptée par le pool) génère l’erreur `ChlorosConnectError` : la lecture d’un capteur sur lui-même est de 1,0 sur toutes les bandes, ce qui ne mesure rien. Le backend le refuse également (`POST /api/daq/indices` répond à `400`, `code` à `SAME_SENSOR`, alors que les `ambient_sensor_id` et `object_sensor_id` envoyés par le SDK correspondent à un capteur regroupé, c'est-à-dire son identifiant et son port).

### Partage avec l’CLI et d’autres scripts

Le pool de capteurs DAQ du backend est partagé par tous les clients CLI et SDK présents sur la machine. Si un capteur se trouve déjà dans le pool — ouvert par `chloros-cli daq pool-connect` ou par un autre script —, l’appel de `connect_daq_sensor(port="COM3")` depuis Python renvoie un descripteur emprunté: son `disconnect()` est une opération sans effet, ce qui évite à votre script d’arracher le capteur au client qui l’a ouvert lors de la sortie de l’environnement. Il en va de même pour `sensor_id=` (tapé avec `:` ou `-`, les deux fonctionnent), pour un DAQ-E identifié par son adresse IP alors qu’il a été ouvert par son nom d’hôte (ou l’inverse), ainsi que pour la détection intelligente avec indicateur zéro lorsque le backend répond avec un capteur qu’il détenait déjà — un bloc `with` ne libère jamais qu’un capteur qu’il a lui-même ouvert.

L’interface graphique d’Chloros **ne**fait**pas** partie de ce pool : son panneau DAQ gère sa propre connexion au capteur, de sorte que `connect_daq_sensor` n’emprunte jamais un capteur que l’interface graphique a ouvert. N’utilisez pas simultanément le même capteur depuis l’interface graphique et depuis un script. Un DAQ-U ou un DAQ-M que l’interface graphique détient ne s’ouvre généralement pas, mais un DAQ-E accepte une deuxième session : la connexion du script modifie les paramètres d’acquisition de l’appareil sous l’interface graphique, et son `disconnect()` interrompt le flux de l’appareil, ce qui met fin à la vue en direct de l’interface graphique et à tout enregistrement en cours. Fermez d’abord le capteur dans l’interface graphique.

### Maître PTP — `time_sync_status()`

Le backend exécute le maître PTP dont les caméras LATTICE et les unités DAQ-E tirent leurs horodatages. Il démarre à la demande : `connect_camera()`, `connect_array()`, `connect_daq_sensor()` et `connect_all()` le lancent s’il n’est pas en cours d’exécution — y compris lors d’une connexion qui échoue ensuite, comme lorsque `connect_all()` tente de se connecter à un DAQ-E qui n’existe pas — tout comme le font `test_array_sync()` et chaque appel qui ouvre des caméras pour une commande (`capture_cameras_transient()`, `start_multi_camera_stream()`, les appels d’alignement, `capture_live_index()`).

- **Où il s’annonce.** Sur les cartes réseau filaires connectées aux caméras (y compris une adresse locale de liaison `169.254.x.x`), sinon sur la carte de la route par défaut si celle-ci est filaire. Jamais sur le Wi-Fi ou une carte virtuelle (Tailscale, WSL) : aucune caméra ni aucun DAQ-E ne se trouve derrière celles-ci. En l’absence d’une telle carte active, il fonctionne en annonçant sur rien (`health` `NO_SOCKETS`) et se lie à la carte dès que la prochaine caméra ou le prochain DAQ se connecte.
- **Après une perte de connexion.** Lorsque la carte réseau de la caméra perd sa connexion puis la récupère (en cas de débranchement d’un câble, de remplacement d’un commutateur ou de réénumération d’une carte USB), le « grandmaster » reconstruit ses sockets de lui-même, sans redémarrage, et vérifie à nouveau chaque DAQ-E connecté, en réactivant celui qui était resté bloqué. Il se reconstruit également lorsque toutes les caméras et tous les DAQ-E qu’il desservait restent inactifs pendant 15 s.
- **La connexion est-elle établie ?** Lire `time_sync_status()` (`GET /api/time-sync/status`). `state` indique uniquement que le thread du « grandmaster » s’exécute ; `health` indique si ses paquets sont transmis : `OK` (aucune anomalie détectée), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (certains adaptateurs sont défaillants) ou `PEERS_SILENT` (les caméras et les DAQ-E qu'il desservait ont cessé de répondre), avec une phrase dans `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` ne comptent que les messages qui ont été envoyés ; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` et `last_delay_req_age_s` décrivent ceux qui ne l'ont pas fait, et `rebinds` / `last_rebind` les reconstructions. `sync_sends_slow` compte les synchronisations dont l’envoi a dépassé 200 µs et `sync_send_max_us` est la plus longue ; l’heure d’origine de chaque Follow_Up compte au maximum 200 µs de son envoi. Le grand maître s’exécute dans son propre processus (`own_process`, `process_pid`), avec une priorité élevée sous Windows (`process_priority`), afin qu’un backend très sollicité ne puisse pas retarder les horodatages et synchroniser d’un seul coup l’horloge de toutes les caméras ; si ce processus ne peut pas être lancé, il s’exécute au sein du backend comme auparavant (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Fonction | Ce qu’elle fait |
| --- | --- |
| `time_sync_status()` | État du grand maître, santé, identité d’horloge, priorités et compteurs de messages (ci-dessus). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Les esclaves PTP qu’il détecte (caméras, DAQ-E) : `peers`, chacun des `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | `PtpStatus` de chaque caméra matricielle connectée, `PtpOffsetFromMaster` et `PtpMeanPathDelay` (une caméra dont le micrologiciel ne comporte aucun nœud PTP ne dispose que de son numéro de série). `time-sync cameras`. |
| `restart_time_sync()` | Arrête et redémarre le grand maître ; le PTP de chaque DAQ-E du pool est revérifié après (`daq_ptp_rechecked`). Déclenche `ChlorosConnectError` lorsqu’il ne revient pas. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Ses priorités PTP (0-255, la plus faible l'emporte lors de l'élection du meilleur maître), pour un réseau comportant un autre maître PTP. Nécessite que celui-ci soit en cours d'exécution. `time-sync set-priority`. |

Chacune prend `backend_url=` et `timeout=` et démarre un backend local qui n’est pas en cours d’exécution.

---

## Utilitaires pour caméras

Ce que fait `chloros-cli lattice` au-delà de la connexion et de la capture, en tant que fonctions simples dans `chloros_sdk` (module `chloros_sdk.camera_utils`). Chacune appelle la route du backend utilisée par la commande CLI, avec les règles de la CLI, de sorte que les deux se comportent de la même manière. Elles prennent un numéro de série de caméra, et non une session, ainsi que les mêmes mots-clés `backend_url=` / `timeout=` que les appels smart-connect ; un backend qui ne s’exécute pas est lancé, comme le fait `connect_camera()`. Un refus déclenche `ChlorosConnectError` avec le motif fourni par le backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Fonction | Ce qu’elle fait |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Le profil de couleur de l’écran : `raw`, `linear`, `natural`, `enhanced` ou `custom_temp`, ainsi que sa température de couleur (limitée à 2 000-10 000 K ; lue lorsque le profil est `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Saturation et contraste de l'affichage, 0 à 200 % chacun (100 = inchangé); `reset=True` rétablit les deux valeurs à 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Réduit la luminosité des pixels les plus clairs et les plus blancs de l’affichage : -100..0 (tronqué vers zéro, puis limité ; 0 = désactivé). |
| `camera_white_balance_once(serial, settle_s=None)` | Balance des blancs en une seule prise : la caméra transmet brièvement, converge, puis renvoie `red_ratio` / `blue_ratio` (`ratios_read` à la valeur « False » lorsqu’elle ne peut pas les communiquer). Elle conserve la balance des blancs qu’elle a déterminée ; tous les autres paramètres écrits par son interface d’entrée sont renvoyés tels quels. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (en continu) ou `"off"` ; ou des rapports de balance fixes `red` / `blue`, qui désactivent d’abord le mode automatique. Un rapport refusé par la caméra est répertorié sous `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Les préréglages de la caméra ; lorsqu’on en applique un, la caméra conserve les paramètres indiqués par le nom du préréglage (la réponse est `settings`, car la caméra les lit ensuite, par exemple la fréquence d’images à laquelle elle fonctionne ; `None` pour un champ que le préréglage ne nomme pas), et tous les autres paramètres que son fichier « open » a écrits sont rétablis tels qu’ils ont été trouvés. `"default"` réinitialise la caméra à ses paramètres d'usine par défaut (ses réglages d'usine avec exposition automatique en mode libre, la limite de débit de la liaison et la correction des défauts désactivées, la balance des blancs calibrée d'une caméra RGB conservée), sans les enregistrer dans sa mémoire : la réponse contient `factory_defaults: True` et tous les champs de `settings`. Dans `camera_presets()`, `default` correspond à `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Les modèles de filtre, ainsi que les détails d’un modèle (bandes, nombre de condition de réponse spectrale). Le modèle de filtre d’une caméra est lu à partir de sa chaîne de modèle ; il n’est pas défini manuellement. |
| `camera_firmware_status(serial=None)` | Le micrologiciel de chaque caméra: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` et `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Flashe le micrologiciel de cette version Chloros sur une caméra : réinitialisation d’usine, flashage, redémarrage, réécriture des paramètres de base de Chloros ; l'étalonnage de la caméra est conservé. Durée : plusieurs minutes ; la caméra doit rester sous tension. `force=True` réinstalle le micrologiciel sur une caméra existante. |
| `camera_power_probe(serial)` | Les nœuds d'alimentation et thermiques de la caméra, ainsi que sa température. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Le mode veille à faible consommation de la caméra (elle reste en veille jusqu’à ce qu’elle soit réveillée : sa limite de débit de liaison activée (`DeviceLinkThroughputLimitMode` `On`) à son seuil minimal, le mode d’économie d’énergie activé si la caméra en dispose), avec la température avant et après ; le réveil rétablit la liaison telle qu’elle était au moment de la mise en veille (la limite, son mode et le délai de paquet `GevSCPD`, que la caméra maintient à 0 tant que la limite est activée ; l’enregistrement est conservé sur cet ordinateur, de sorte qu’un réveil effectué ultérieurement par un backend puisse également le retrouver), ou, pour une caméra que cet ordinateur n’a pas mise en veille mais qui semble en veille (sa limite activée au seuil minimum), avec la limite désactivée et un `GevSCPD` laissé à 0 à 80 (réglages d’usine) ; une limite définie au-dessus du seuil minimal est conservée telle quelle. Seul l’état d’alimentation est conservé ; tous les autres paramètres dont la valeur a été modifiée sont rétablis tels qu’ils ont été trouvés. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Capturer des images jusqu’à ce qu’une cible d’étalonnage MAPIR soit dans le champ de vision, puis ajuster les coefficients par canal à partir de ses panneaux : `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Les cartes réseau Ethernet de cet ordinateur pour GigE Vision (MTU, vitesse de liaison, trames jumbo) et tout `issues` ; `estimate=True` ajoute la fréquence d'images et la bande passante dont bénéficieraient les caméras `cameras`. |
| `fix_network_settings()` | Applique les paramètres de la carte GigE Vision à l'adaptateur de caméra de cet ordinateur. Cela modifie les paramètres de l’adaptateur et ne fonctionne que lorsque Chloros est exécuté en tant qu’administrateur ; sinon, le programme s’arrête en affichant le journal de ce qu’il a détecté. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Corrige la vignetture des images d’un dossier, chacune avec la carte de champ plat de son propre objectif : `processed`, `lens_models` (l'objectif pour lequel chaque fichier a été corrigé), `skipped` (avec les raisons). Sans `lens_model`, l'objectif de chaque fichier provient de son modèle EXIF ; un fichier `lens_model` s’applique aux fichiers dont le modèle ne mentionne aucun appareil photo, et un fichier dont le modèle contredit cette information est ignoré. Fichiers uniquement. |
| `read_dls(mac, filter=None, camera_serial=None)` | Une mesure provenant d’un DLS (capteur de lumière descendante Bluetooth) : spectre, l’irradiance par bande et les amplitudes de bande via la réponse propre à l’appareil photo lorsque `camera_serial` est spécifié. `discover_daq_sensors(transports=["ble"])` détecte l’adresse MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Le matériel de l’hôte dédié aux caméras et la configuration qu’il recommande (`report_text`), ainsi que le CPU / GPU / runtime PyTorch du système. |

- **Certaines ont besoin de la caméra pour elles seules.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` et `update_camera_firmware()` ouvrent la caméra. Tant qu’Chloros la détient (connectée dans l’application, avec `connect_camera()`, en tant que membre d’un ensemble, ou actuellement utilisée par une autre commande, comme un test de tableau ou une capture ouvrant ses caméras), le backend renvoie une erreur HTTP409 `CAMERA_HELD` (à la fois dans `code` et `error_code`), indiquant comment le libérer. Les réglages de couleur, les hautes lumières, `camera_power_probe()` et les lectures fonctionnent sur une caméra connectée.
- **La couleur, les hautes lumières et la balance des blancs concernent les caméras RGB.** Sur une caméra mono à bande unique (M3M), les fonctions de réglage ne modifient rien et renvoient `{"skipped": True, "reason": ...}`, tout comme le fait l’CLI. Il s’agit de paramètres d’affichage : la vue en direct et les captures au niveau de l’affichage, jamais la radiance ni la réflectance.

---

## Tableaux transitoires, profils d’alignement et indices

Ce que fait `chloros-cli lattice` en l’absence de connexion persistante, tout comme les fonctions de `chloros_sdk` (module `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` et `index`. Chacune envoie la requête transmise par la commande CLI, avec les paramètres par défaut de cette dernière.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Elles ouvrent elles-mêmes les caméras.** Chaque commande ouvre ses caméras, effectue son travail et les ferme ; elle a donc besoin de caméras qu’Chloros ne détient pas (connectées dans l’application, avec `connect_camera()`, ou en tant qu’élément d’un tableau) ; le backend renvoie un code d’erreur HTTP 409 `CAMERA_HELD` et indique comment en libérer une. Une ouverture temporaire prend un certain temps (environ 40 s pour quelques caméras). Pour les captures répétées, `connect_array()` une seule fois constitue la méthode la plus rapide, et cela fonctionne avec tous les types d’exportation, alors qu’une capture transitoire enregistre une image débayérisée. `calibrate_alignment_profile()` fait exception : sur un réseau connecté de ces caméras, il traite à partir des images propres au réseau (`used_live_array`).
- **`serials`** désigne les caméras à ouvrir, la première déclenchant l’opération ; `None` ouvre toutes les caméras du réseau.
- **`settings`** est un dictionnaire de paramètres de caméra prioritaires, dont les champs sont définis par les indicateurs de l’CLI : `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` et `preset` (un préréglage de caméra par son nom). Une valeur d’exposition ou de gain spécifiée désactive ce contrôle automatique, tout comme le font les indicateurs.
- **`gpu`** : `True` force le warp GPU, `False` le CPU (`--gpu` / `--no-gpu`) ; `None` laisse le choix au backend.
- **`keep_settings`**/**`clear_settings`** (toutes les fonctions qui ouvrent des caméras, valeur par défaut : `False`): à la fin, chaque caméra est réinitialisée à son état initial ; `keep_settings=True` enregistre plutôt les paramètres de la session dans la mémoire propre de la caméra (fonctionnement autonome), `clear_settings=True` réinitialise cette mémoire aux paramètres d'usine — voir [Mémoire de la caméra](#camera-memory--keep_settings--clear_settings). La réponse (pour une tâche, celle de `stop()`) contient `camera_memory`.
- **Les chemins d’accès sont ceux du backend** : les chemins relatifs sont convertis en chemins absolus par rapport à votre répertoire de travail lorsque le backend se trouve sur cette machine. Les fonctions de fichier de profil d’alignement (`alignment_profile_info()`, `reorder_alignment_profile()`) lisent le fichier sur cet ordinateur, tout comme le fait l’CLI.

| Fonction | Description |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Enregistre en continu une caméra jusqu’à ce que l’opération soit arrêtée. Renvoie un `CliJob`. Les images d’une exécution `tiff` constituent le fichier `raw`, un tableau contenant les données de capture de cette caméra (un plan de mosaïque par, non compressées) ; les fichiers `png` et `jpg` contiennent 8 bits ; ces séries correspondent donc à des images démoisaillées destinées à la visualisation. Un DAQ lié ou mutualisé écrit un fichier `.daq` par image dans ledossier `daq/` de la série (`daq_files` dans ses statistiques). Chaque image est écrite une fois avec l’intégralité de ses métadonnées (modèle, heure de capture, objectif, bandes, étalonnage sur une image brute) ; `stop()` renvoie une réponse une fois que toutes les images déjà capturées ont été écrites (jusqu’à 25 s), et ce n’est qu’alors que la caméra est fermée. Une caméra laissée en mode « armed » pour un déclencheur matériel (`TriggerMode` `On`) est réglée en mode libre pour la session (la réponse de démarrage `trigger` indique ce qui a été détecté). À l’arrêt, la caméra est réinitialisée à l’état dans lequel elle a été trouvée — ses paramètres de déclenchement et tous les réglages modifiés pendant la session —, sauf si `keep_settings=True` est utilisé, ce qui enregistre les paramètres de la session dans la mémoire propre de la caméra (mode libre), ou si `clear_settings=True` est utilisé, ce qui réinitialise cette mémoire. Une session qui n’enregistre aucune image se termine automatiquement après 10 s avec `stop_reason` et `no_frames`, la raison étant indiquée dans `last_error`, et sa commande `stop()` renvoie ce motif sous la forme `error`. Les images perdues parce que les périphériques d’écriture n’ont pas pu suivre sont comptabilisées, la commande `drop_reason` en indiquant la raison. Pour une caméra gérée par Chloros, utilisez `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Diffuse des groupes d’images synchronisées à `fps`, par groupes `count` ou jusqu’à ce que la transmission soit arrêtée. Renvoie un `CliJob` (dont `start_reply` contient `n_cams` et le flux `pacing`) ; ses statistiques finales comptabilisent les groupes capturés, écrits et abandonnés, les fichiers que l’éditeur n’a pas pu écrire et les groupes pour lesquels une caméra n’a rien fourni. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Une capture synchronisée, une image débayérisée par caméra : `results` (`filepath`, `serial`, `frame_id`) et `pacing`. Un groupe auquel il manque une caméra s'affiche, en indiquant d'abord le nombre de fichiers enregistrés. `lattice multi-capture` sans matrice connectée. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` captures synchronisées à une exposition fixe : `sync_groups`, `master`, une ligne `group_timestamps` par déclenchement (une caméra ayant manqué un déclenchement est absente de sa ligne) et `sync` : décalage de chaque caméra par rapport à la caméra maître, déclenchement par déclenchement (`offsets_us` : par caméra `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), sur UNE horloge (`clock_domain` : `ptp`, ou `device` — PTP désactivé, horloge de fonctionnement propre à chaque caméra ; son décalage correspond donc au moment de son démarrage et seule la gigue mesure la synchronisation). Un déclencheur dont les horodatages combinent l’horloge de l’appareil et le PTP (PTP désactivé ou encore en cours de synchronisation) est ignoré et comptabilisé (`groups_mixed_domain`), tout comme celui où une horloge PTP est encore à plus de `agree_within_us` de la maître (`groups_clocks_disagree`) ; `notes` l’indiquent. `clock_offset_mean_us` / `clock_offset_jitter_us` correspondent à la moyenne et à la gigue maximales par caméra. `success` est faux, tout comme `error`, lorsqu’un déclenchement a été manqué, lorsque moins de deux déclenchements ont synchronisé toutes les caméras sur une même horloge, ou lorsque les horloges ne concordent jamais (`sync.problems`) ; les lignes sont renvoyées dans tous les cas. Il démarre le grand maître PTP s’il n’est pas en cours d’exécution (`time_sync` indique `grandmaster_running` et `grandmaster_started_by_this_test`), active le PTP de chaque caméra (comme le fait `connect_array()`), et avant le premier déclenchement, attend que l’horloge PTP de chaque caméra se verrouille sur le grand maître et se stabilise : `PtpStatus` : « Slave » sur un maître, avec un écart inférieur à 100 µs par rapport à celui-ci et stable pendant 3 s, et chaque caméra verrouillée de cette manière, sans interruption, pendant 120 s — soit environ 2 minutes après la mise sous tension. Une horloge qui vient de se synchroniser présente un écart pouvant atteindre environ 200 µs pendant environ 2 minutes après avoir affiché « Slave », et le propre `PtpOffsetFromMaster` de la caméra ne le signale pas ; 120 s suffisent pour que les caméras se rapprochent à environ 10 µs d’une horloge stabilisée. Une caméra restée verrouillée depuis un test précédent qui avait laissé le PTP activé, telle que perçue par ce backend, compte à partir de ce verrouillage, de sorte qu’un test effectué juste après un autre n’attend pas à nouveau ; une caméra mise sous tension par le test, ou une caméra détectée avec le PTP activé mais dont ce backend n’a pas vu la synchronisation (allumée ailleurs, ou avant un redémarrage du backend), attend ses 120 s, et dans un ensemble comprenant les deux caméras qui viennent de se synchroniser, celles-ci attendent toujours leur tour. Une caméra qui quitte le mode esclave, dépasse les 100 µs ou suit un autre grand maître pendant 3 lectures consécutives (1,5 s) pendant l’attente recommence son décompte ; une ou deux de ces lectures sont enregistrées (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) et le décompte se poursuit, et un identifiant de grand maître qui ne peut être lu est considéré comme inconnu, et non comme un changement. Le fait que le comptage de toutes les caméras redémarre simultanément (en moins d’une seconde) est dû au décalage temporel du « grandmaster », et non à celui des caméras : le journal du backendindique `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` le répertorie dans `grandmaster_steps`, et un délai d’attente qui expire alors commence son message d’erreur par `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. La limite n’est pas prolongée pour celui-ci. `ptp_settle_s` (0-600 ; `0` en l’absence d’attente de stabilisation) définit le délai de stabilisation pour l’exécution ; toute autre valeur est refusée (code HTTP 400). L'attente est limitée au temps de stabilisation plus 90 s (210 s par défaut, au moins 90 s), et l'appel attend le backend `timeout` s : par défaut 240 plus le temps de stabilisation (360 s), tout comme le fait `test_array_sync` de `chloros-mcp`. `ptp_lock` dispose de `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (durée pendant laquelle chaque caméra est restée verrouillée et stable), `found_settled` (les caméras déjà verrouillées pendant la durée de stabilisation au moment où elles ont été détectées) et, par caméra, `status`, `offsets_ns`, `slave_s` (durée depuis laquelle elle est verrouillée ; « Aucune » : non verrouillée) et `remaining_s` (« Aucune » lorsque le PTP d’aucune caméra ne s’active). Le journal du backend contient une ligne toutes les 5 s indiquant l’état de chaque caméra, depuis combien de temps elle est verrouillée et le temps qu’il lui reste. Chaque caméra s'allume avec le PTP désactivé ; à partir de là, les horloges se synchronisent en environ 10 s. Ensuite, chaque caméra est réinitialisée selon les paramètres relevés lors du test — son PTP, l'exposition automatique et le gain que le test maintient désactivés pendant son exécution (`ExposureAuto` / `GainAuto`, et un mode manuel `ExposureTime` / `Gain`), et les autres, sauf si `keep_settings=True` — c’est-à-dire une caméra dont le PTP reste activé (`PtpEnable=True`, ou un cycle avec `keep_settings=True`) reste verrouillée, et une fois que ce backend l’a vue verrouillée pendant le temps de stabilisation, un test ultérieur ne l’attend pas. Si les horloges ne se stabilisent pas, `success` est faux avant tout déclenchement, et `error` indique la marche à suivre : une caméra qui ne s’est jamais verrouillée (`not_slave`) signifie que le grand maître ne parvient pas à la joindre (`time_sync_status()`) ; celle qui n’a cessé de se déconnecter est identifiée par la durée pendant laquelle elle est restée verrouillée (`remaining_s`) ; les horloges sont encore en phase de convergence (`far`) : vérifiez `time_sync_status()`, et pour qu’elles soient verrouillées avant le test, activez le PTP et laissez-le activé (`PtpEnable=True` sur chaque caméra, ou `keep_settings=True` ; le test rétablit les paramètres qu’il a détectés, donc il reste activé), puis lancez-le. `lattice multi-test`. |
| `detect_array_master(line=None)` | Les caméras sur la ligne de synchronisation et un maître par défaut : `master_serial`, `slave_serials`, ainsi que les caméras `line` détectées. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Quelle gamme de modèles de caméras (noms de produits, `"LATT-M3C-L87-FOCN"`) se retrouverait sur le réseau de cet ordinateur : `tier`, `achievable_fps_max`, le budget de câblage (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` et `fix_suggestions`. Par défaut, les modèles de caméras détectés ; aucune caméra n’est ouverte. Pour les caméras dont vous disposez, avec les paramètres qu’une connexion utiliserait, consultez [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Définir un profil d’alignement à partir d’images en temps réel et l’enregistrer. `method` : `orb`, `akaze`, `phase`, `checkerboard` (avec `checkerboard="9x6"`, les coins intérieurs) ou `manual` (avec `control_points={serial: [[x, y], ...]}`, ceux de la caméra de référence également). `model` : `translation`, `rigid`, `affine` ou `homography`. Les options plus précises correspondent aux indicateurs de l’CLI : `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis` ; `per_cam_overrides` est `{serial: {option: value}}`. `rms_threshold_px` refuse d'enregistrer une solution moins bonne. Renvoie `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (par caméra `reprojection_error`, `num_matches`) et `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Une image multibande alignée sur le profil : `files` (le fichier TIFF multibande avec son `bands` et sa taille, plus un par bande avec `per_band` et un par caméra avec `per_camera`) et `vignette_corrected`. `bands` sélectionne et `order` classe les bandes par nom (liste ou chaîne séparée par des virgules). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Aligne les images multibandes (mêmes options de déformation que `capture_aligned_multiband()`), pour `count` ou jusqu’à ce que l’opération soit arrêtée. Renvoie un `CliJob` ; ses statistiques finales comptent `frames_written` et tout `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Un fichier de profil résumé sous la forme `align-info` l'affiche : `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (par caméra : `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (par caméra : `band_names`, `wavelengths_nm`, `enabled`) ; `profile` correspond au fichier complet. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Modifie le fichier de profil sur place : `order` correspond aux noms de bandes (chacune déplace le calque de sa caméra vers le haut, dans cet ordre), `enable` / `disable` sont des numéros de série. Renvoie le nouveau `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Les préréglages d’index (`presets` : chacun des `name`, `channels`, `formula`) et de couleur `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Un index sur un fichier TIFF multibande aligné : un `preset` (`NDVI`, `NDRE`, ...) ou un `formula` sur les symboles `channel_map` correspond à des bandes (`{"NIR": "NIR_850", "Red": 0}`, par nom ou par index). Un préréglage’symbole `channel_map` omet la bande du fichier dont le nom commence par celui-ci (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), nommée par le fichier lui-même (sa liste de bandes, ou celle d’une capture ou d’un produit Chloros `Chloros:Band{R,G,B}Name`), ainsi, l’NDVI sur une acquisition de réflectance Chloros ne nécessite pas de `channel_map` ; deux bandes correspondant à un même symbole sont rejetées, et non devinées. `capture_level` indique ce que contiennent les bandes : `reflectance` ou `radiance` rendent les valeurs quantitatives ; une pile non calibrée (celle de `align-apply`) fournit des valeurs qualitatives ; `"unknown"` lit ces données à partir du fichier, y compris le fichier `Chloros:ProcessingLevel` Chloros. Renvoie `stats` (la forme, la plage des 1er à 99e centiles, la plage de la table de conversion, la valeur `capture_level` utilisée) ainsi que la valeur `files` inscrite à côté des entrées (`<input stem>_<preset>.tif`, `..._LUT.png`) ou à l’emplacement `output`, un chemin d’accès : s’ils comportent une extension d’image, les fichiers sont nommés à partir de leur racine (`output="D:/out/ndvi_FRGN.png"` génère `ndvi_FRGN.tif` et `ndvi_FRGN_LUT.png`), sans extension, il s'agit d'un préfixe (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`) ; lisez les chemins d’accès à partir de `files`. Ils contiennent les informations d’enregistrement de l’appareil photo source (marque / modèle / numéro de série, GPS, objectif, heure de prise de vue en UTC, l’identifiant d’étalonnage mais jamais son étalonnage raw-DN), `Chloros:ProcessingLevel=index`, ainsi que l’index comme bande unique (`XMP-Camera:BandName`, le fichier TIFF et le fichier PNG LUT). Fichiers uniquement ; pas de caméra. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Capture, étalonnage (`reflectance` ou `radiance` ; `raw` ignore cette étape), aligner et calculer un index en une seule étape ; `save_multiband=True` conserve la pile calibrée, que `compute_index()` pourra récupérer ultérieurement. Renvoie `stats` (avec le `serials` dont il a utilisé les bandes) et le `files`. Elle ouvre `serials` (la première déclenche le déclencheur), sinon les caméras du profil dont l’entrée `layer_order` est activée, la caméra de référence en premier (ouverte même si ses bandes sont désactivées) ; une caméra dont les bandes sont désactivées n’est jamais calibrée. `channel_map` n’est nécessaire que pour un symbole auquel la pile ne peut pas répondre, comme dans `compute_index()` : le symbole d’un préréglage prend la seule bande activée dont le nom commence par celui-ci, vérifiée avant l’ouverture de toute caméra (si aucune ou plusieurs répondent, la requête est refusée, en les nommant) ; un `formula` personnalisé est nécessaire pour chaque symbole. `index.tif` et la LUT contiennent la caméra de la pile, l’heure de capture (l’exposition de la caméra de référence, en UTC) et l’objectif, et tous deux désignent l’index comme leur unique bande (`XMP-Camera:BandName`, par exemple `NDVI` : le fichier PNG couleur désigne l’index, et non ses canaux de couleur). `lattice index --live`. |

### `CliJob`

Le descripteur renvoyé par les trois flux. La tâche s'exécute en arrière-plan et conserve le contrôle de ses caméras jusqu'à ce qu'elle soit arrêtée.

| Élément | Description |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (images enregistrées / écrites, ignorées, `fps`, `elapsed`, échecs d’écriture, `stop_reason` ; pour une tâche continue, également `frames_grabbed` et `frames_queued` — chaque image capturée est écrite, mise en file d’attente, abandonnée ou a échoué — et `drop_reason`, `writers` et `write_ms`, soit le temps moyen nécessaire à l’écriture d’une image) et `output_dir`. `state` correspond à `running`, `ended` (il s’est arrêté de lui-même et n’a pas encore été arrêté), `stopping` (un arrêt est en train de vider sa file d’attente : statistiques en temps réel, et `stopped_by`) ou `finished` (ses statistiques finales, les `error` et `stopped_by` de l’arrêt). L'historique d'un travail arrêté est conservé pendant 10 minutes (les 32 dernières au maximum) ; pour chaque client : un autre script ou `chloros-mcp` lit le même état et les mêmes statistiques finales. Un travail `chloros-cli` a démarré (`lattice continuous`, `multi-stream`, `align-stream`) est arrêté par le backend lorsque l’CLI cesse de se manifester pendant 30 s — tué, sa machine en veille — avec `stop_reason`, `client_gone` et `stopped_by` du backend ; la lecture de ses statistiques à partir d’ici ne le maintient pas actif. Une tâche lancée depuis l’SDK s’exécute jusqu’à ce qu’elle soit arrêtée. Chaque démarrage envoie son `timeout` comme délai du backend (`client_deadline_s`): une caméra qui s’ouvre après celle-ci (celle-ci étant encore en cours de démarrage) est remise en place puis fermée, et aucune tâche n’est lancée ; l’appel génère alors `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Une caméra occupée par une tâche est refusée à toute autre ouverture (`connect_camera()`, un autre démarrage, une capture transitoire) avec un code HTTP 409 `CAMERA_HELD`, indiquant le nom de la tâche et celui de la personne qui l’a lancée. |
| `stop(timeout=30.0)` | L’arrêter et fermer ses caméras : `final_stats`, `output_dir` et `error` lorsque l’arrêt ne s’est pas déroulé correctement. Un deuxième appel renvoie la même réponse, tout comme un arrêt lancé depuis un autre client : celui-ci, envoyé alors que le premier est encore en train de vider sa file d’attente, attend que celui-ci termine (jusqu’à 25 s ; passé ce délai, une exception est levée, et les statistiques finales apparaissent sur `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Attendre qu’il se termine de lui-même (lorsque son `count` est atteint, que le disque est plein ou qu’une erreur survient), puis `stop()` l’interrompre pour collecter les comptes finaux. `timeout` déclenche `TimeoutError` alors qu’il est toujours en cours d’exécution ; un travail sans `count` ne se termine que lorsqu’il est arrêté. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Son identifiant et son type (`continuous`, `multi-stream`, `align-stream`), l'emplacement où il écrit, la réponse de la route de départ et si `stop()` s'est exécuté. |

Utilisez-le comme gestionnaire de contexte pour l'arrêter à la sortie de la portée.

---

## Dossier du projet et recettes

**Le dossier de projet** est le dossier dans lequel les nouveaux projets sont créés ; il est partagé par l’application Chloros, la CLI et le SDK (module `chloros_sdk.system_utils` ; `chloros-cli project-folder`) :

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Si un dossier configuré n'existe plus, le système revient par défaut au dossier par défaut (`Documents/MAPIR/Chloros Projects`), tout comme l'application. Un chemin relatif indiqué à `set_project_folder()` est interprété à partir de votre répertoire de travail lorsque le backend se trouve sur cette machine.

**Une recette** est la liste d’actions au format YAML / JSON que `chloros-cli project run PATH RECIPE` exécute sur un projet enregistré. `run_project_recipe()` l’exécute avec le(installé avec le SDK), de sorte qu’une même recette se comporte de la même manière, qu’elle soit exécutée par :

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** est un fichier `.yaml` / `.yml` / `.json` ou de la recette elle-même : un dictionnaire contenant `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`), ainsi que, de manière facultative, `overrides` et `stop_on_error` (valeur par défaut : true : l'exécution s'arrête à la première action qui échoue). Les actions correspondent au [Recipe DSL](cli-reference.md#recipe-dsl) de la référence CLI ; un `output` relatif est résolu par rapport au répertoire de travail de votre script.
- **Renvoie** `ok` (toutes les actions ont réussi), `actions` (nombre d’actions réussies), `log` (la sortie du programme d'exécution ; `echo=True` l'affiche également pendant son exécution) et `validation`. Un fichier de recette manquant ou impossible à analyser génère l'erreur `ChlorosConnectError`.
- **Vérifié avant son exécution.**`validation` correspond à `{"ok", "errors", "warnings", "steps"}` : chaque action doit avoir pour clé exactement un verbe connu (`{"capture": {"name": ...}}`, et non `{"action": "capture"}` ou `{"type": "capture"}`) désignant une caméra, un tableau ou un capteur dont dispose le projet, et du type approprié. Une recette comportant une erreur n’exécute**aucune** action (`ok` false, les erreurs dans `log`), quelle que soit la valeur de `stop_on_error`. Les clés inutilisées, ainsi que tout tableau ou capteur de lumière piloté par la recette mais qui n’est pas connecté pour le moment, sont `warnings`. Une recette ne connecte pas les périphériques : connectez d’abord le projet (`ChlorosProject.connect_all()`, `chloros-cli project connect`, ou l’outil MCP `connect_project_devices`). `dry_run=True` n’exécute rien et renvoie `{"recipe": ..., "validation": ...}`.
- Une seule recette s'exécute à la fois dans un processus. Une recette YAML nécessite PyYAML.

---

## Automatisation de projet — `ChlorosProject`

Un projet Chloros enregistré est un dossier contenant `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` charge le manifeste (`relink` : le dossier dans lequel se trouvent désormais les fichiers source manquants d’un projet déplacé, utilisé lorsque le backend le charge — `process()` — voir **Un projet déplacé** sous `ChlorosLocal`), et `connect_all` met en ligne toutes les caméras, matrices et capteurs de lumière enregistrés avec leurs paramètres sauvegardés — soit le même état matériel que celui que produirait l’interface graphique. Les capteurs de lumière se connectent via le [pool de capteurs DAQ](#daq-sensor-sessions) ; ils ont donc besoin d’un backend en cours d’exécution (voir [`SensorHandle`](#sensorhandle)).

### Exemple minimal

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Ou en tant que gestionnaire de contexte :

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Méthodes `ChlorosProject`

| Méthode | Description |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Détecte et connecte toutes les caméras, matrices et capteurs de lumière enregistrés. Renvoie un rapport de connexion par classe (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, chacun avec `connected`, `missing` et `errors` ; `arrays` inclut également `alignment`). Si un backend est à l'écoute sur `127.0.0.1:5000`, les caméras et matrices enregistrées sont détectées et connectées via celui-ci. En l'absence de backend, ce processus les détecte et les pilote lui-même via `lattice_sdk`, qui nécessite le runtime de l’SDK Arena et OpenCV (`cv2`) — la roue SDK n’installe ni l’un ni l’autre ; lorsqu’elle n’y parvient pas, chaque caméra et chaque matrice enregistrées sont répertoriées sous `errors` avec la raison correspondante, et `missing` reste vide. Un backend en cours d’exécution mais refusant le SDK (déconnexion : HTTP 401 ; un plan inférieur à Copper : 403) n’est pas considéré comme « aucun backend » : chaque caméra, tableau et capteur de lumière enregistré est répertorié sous `errors` avec la raison fournie par le backend, et rien n’est ouvert dans ce processus. Chaque appel vérifie à nouveau le backend ; ainsi, `connect_all()`, après la connexion ou après le démarrage d’Chloros, se connecte au même objet de projet. Un ensemble enregistré déjà actif dans le backend avec les mêmes caméras (celles de l’interface graphique ou celles de `chloros-cli project connect`) est associé tel quel, sans réapplication de ses paramètres enregistrés, et `disconnect_all()` le laisse fonctionner. Les capteurs de lumière passent toujours par le backend (voir [`SensorHandle`](#sensorhandle)). Il ne lance jamais de backend. |
| `disconnect_all()` | Démonte tout. Chaque périphérique est d’abord libéré ; ensuite, si le backend n’a pas confirmé la libération d’une caméra, d’un tableau ou d’un capteur de lumière (le démontage a expiré ou a été refusé), un `ChlorosConnectError` est attribué à chacun d’entre eux. Un périphérique déjà supprimé, ou pour lequel aucun backend n’est en cours d’exécution, reste silencieux. Un réseau de capteurs ou un capteur de lumière qu’un autre client avait ouvert reste en cours d’exécution. La sortie `with` du projet ne se déclenche jamais. |
| `capture_all(output_dir=".")` | Une image de chaque caméra et réseau de capteurs connectés, et un spectre (`SensorHandle.read()`) provenant de chaque capteur de lumière connecté, capturés tour à tour (caméras, puis matrices, puis capteurs), et non en un seul instant. |
| `stream(camera, overlays=False, fps=10.0)` | Générateur produisant des images BGR `numpy` à partir d’une caméra (ou d’un réseau) nommée. `overlays=False` lit les images dont le descripteur`grab()` au niveau `debayered` : via le backend lorsque le projet est connecté via celui-ci, sinon via une boucle de capture `lattice_sdk` directe (les tableaux génèrent des dictionnaires `{serial: frame}`). `overlays=True` achemine via `ChlorosLocal.camera_stream()` → le flux MJPEG `/api/camera/<serial>/stream-annotated` du backend, le bloc `ui.overlay` enregistré par la caméra étant transmis sous forme de paramètres de requête. Nécessite le mode backend, OpenCV (`cv2`) et une **caméra autonome** : une caméra en mode direct déclenche `RuntimeError` (le backend ne peut pas récupérer une caméra appartenant à ce processus) et un tableau déclenche `NotImplementedError` (superpose les images composites par caméra — diffuse un élément par son nom). Équivalent en une seule étape : `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Exécute l’alignement sur chaque tableau actuellement connecté. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Exécuter le pipeline d’étalonnage / d’indexation sur les images du projet (enveloppe `ChlorosLocal.process` ; ces quatre-là sont les **seuls** arguments clés acceptés — `indices=` etc. génère une exception `TypeError` ; définit les indices via `ChlorosLocal.configure()`). Construit de manière différée un `ChlorosLocal()`, qui lance automatiquement un backend. |

Attributs :
- `proj.cameras` — `Dict[str, CameraHandle]` indexé par nom ET numéro de série.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexé par nom ET array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexé par nom ET slot_id.
- `proj.config` — Dictionnaire `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Niveaux de traitement.** `capture()`, `grab()` et `frame_stream()` utilisent tous le même jeton `processing`, et la chaîne est cumulative : chaque niveau exécute tout ce qui se trouve au-dessus de lui :

| Niveau | Sortie | Remarques |
| --- | --- | --- |
| `raw` | Bayer 1 canal, natif du capteur | Pas de dématriçage. Les superpositions ne sont pas disponibles à ce niveau. |
| `debayered` | Couleur à 3 canaux (**par défaut** pour `CameraHandle` et tous les `grab()` / `frame_stream()` ; `ArrayHandle.capture()`, `capture_smart()` et `stream()` sont par défaut configurés sur `raw`) : BGR en mémoire (`grab()` / `frame_stream()`), enregistré au format RGB par `capture()` | En mode backend, lors d’une capture : le décalage de noir (DSNU) du pack d’étalonnage appliqué au fichier brut, puis le dématriçage sensible aux contours d’OpenCV — sans table de conversion linéaire (LUT), champ plat ni démixage. Le seul niveau qui fonctionne sans mode backend est celui du simple dématriçage bilinéaire de la bibliothèque de la caméra (sans décalage de noir). Le fichier TIFF enregistré est au format RGB photométrique ; il faut donc inverser les canaux d’une image en mémoire (`frame[..., ::-1]`) pour la comparer à un fichier. |
| `radiance` | float32, W/m²/sr/nm, **un plan par bande** | Chaîne radiométrique complète : dématriçage + démixage 3×3 (multispec) + DSNU + champ plat + échelle NIST, l'exposition et le gain ayant été soustraits pour que les valeurs soient absolues. |
| `reflectance` | uint16, 32768 = 1,0 (ou uint8, 255 = 1,0 — voir `bit_depth`), **un plan par bande** | Radiance divisée par l'irradiance descendante (ρ = π·L/E). Nécessite une lecture DLS/DAQ — voir la note ci-dessous. |
| `display` | 8 bits, similaire au sRGB | Rendu équivalent à celui de l'interface graphique : CCM + balance des blancs + gamma via le profil de couleur actif de la caméra. |

En mode direct (sans backend), seul `debayered` est disponible : `grab()`, à un autre niveau, déclenche
`NotImplementedError`, `capture()` ignore `processing`, et `frame_stream()` déclenche
`RuntimeError` à chaque niveau. `reflectance` a besoin d’une lecture descendante utilisable — le point final de la trame extrait
automatiquement le DAQ regroupé vers l’emplacement DLS de la caméra, mais en l’absence de DAQ associé, la chaîne refuse la
sortie de réflectance et indique clairement la rétrogradation dans les métadonnées renvoyées plutôt que de
renvoyer un produit de qualité inférieure.

> **Échelle DN de réflectance — ne pas la coder en dur.** La réflectance LATTICE utilise `32768` = ρ 1,0 et indique
> XMP `Chloros:PixelScale=32768` ; la réflectance Survey3 utilise `65535` = ρ 1,0 et ne comporte aucune
> balises `Chloros:*`. Lisez la balise et divisez par sa valeur. Elle est définie dans le domaine uint16, elle reste donc
> `32768` pour tous les formats qui effectuent un redimensionnement (TIFF 16 bits, TIFF 8 bits, PNG/JPG 8 bits, pourcentage 32 bits)
> — normalisez d’abord le type de données stocké en uint16 (×257 à partir de 8 bits, ×65535 à partir de float). La seule
> exception concerne un fichier issu d’une version antérieure à la 1.3.0 : un TIFF 8 bits était alors *découpé*, et non redimensionné, donc
> aucune échelle ne le décrit, et il ne contient ni tuple `PixelScale` ni MicaSense. Traitez une balise manquante sur un
> fichier de réflectance LATTICE comme « aucune échelle valide », et non comme une valeur par défaut.

**Balises d’étalonnage : sur `raw` uniquement.** Le fichier `raw` d’une capture contient le paquet d’étalonnage de la caméra : la charge utile `CalibrationJSON` et les balises compatibles MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), qui décrivent les valeurs brutes du capteur. Les fichiers de tous les autres niveaux (`debayered`, `display`, `radiance`, `reflectance`, le rendu `index/`) ainsi que chaque image issue d’un enregistrement continu ou d’un flux multi-caméras comporte l’identifiant de la caméra (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, …) ainsi que les données propres à l’image, `XMP-mapir:FrameJSON` (exposition, gain, température, format de pixel, heure de capture, correspondance DAQ), car chaque produit `process()` porte cette identité. Le traitement retrouve l’étalonnage d’un tel fichier grâce à son numéro de série et à `CalBundleSha256` dans le cache d’étalonnage d’Chloros, en téléchargeant le pack une seule fois si cet ordinateur ne le possède pas.

#### Fichiers créés à partir de plusieurs caméras

Un fichier créé à partir des bandes de plusieurs caméras — un empilement multibande (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), un index en temps réel créé à partir de plusieurs caméras (`index.tif`, `index_LUT.png`), une image `composite/` issue d’une capture en réseau — mentionne toutes les caméras qui y ont contribué :

- **Par-caméra, par ordre de bande** (la composition : couche d’arrière-plan, puis de premier plan ; une couche d’index, les caméras dont proviennent ses bandes) : `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (le pack d’étalonnage par lequel sont passées les bandes de chaque caméra, sinon — une pile `capture_aligned_multiband()` / `start_aligned_stream()`, dont les bandes ne passent par aucun — le pack de la caméra que Chloros a mis en cache, celui dont le nom correspond à celui de ses fichiers de capture), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — des listes parallèles, dont l’élément *n* de chacune décrit la même caméra ; `-` où une caméra n’a aucune valeur (pas de pack d’étalonnage, un objectif inconnu). Chaque caméra correspond à **un** élément : lorsque son pack comporte plusieurs valeurs pour un champ — les certificats de lampe halogène d’un pack — celles-ci sont reliées par `;` (`F-1479-1;F-1480-1;F-1481-1`), jamais par une virgule ; ainsi, une liste affiche un élément par appareil photo, quelle que soit la manière dont elle est imprimée.
- **EXIF `SerialNumber`** contient les mêmes numéros de série, séparés par des virgules (`214000535,214000551`).
- Il ne contient **aucune** identité de caméra individuelle (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …) : celles-ci désignent une caméra et son pack, et le traitement permettrait de déterminer l’étalonnage du fichier à partir de ces informations. Un index calculé à partir des bandes de plusieurs caméras (`index.tif`, `index_LUT.png`) comporte les données EXIF `Model` `LATTICE` (le dispositif), et non le modèle `LATT-…` d’une seule caméra.

Un fichier contenant les bandes d’**une seule** caméra — un TIFF par bande, une image `capture_aligned_multiband(per_camera=True)`, une pile ou un index en temps réel d’une seule caméra — porte les balises d’identité propres à cette caméra, comme c’est le cas pour tout autre fichier de capture. La pile de `capture_aligned_multiband()`, ses fichiers par bande et par caméra, ainsi que chaque pile de `start_aligned_stream()` et ses fichiers par bande, comportent l’heure de capture de leur groupe (l’exposition de la caméra de référence, en UTC), et une image `per_camera` d’une caméra monochrome est un plan, à l’instar de la pile `bit_depth`.**Orientation de la caméra (retournement / rotation).** Les images issues de `grab()` / `frame_stream()` sont toujours dans l’
orientation native de la caméra, à tous les niveaux. Les captures enregistrées respectent le réglage d’orientation de la caméra
(options « Mirror H » / « Mirror V » / rotation de l’interface graphique) :

- **Les fichiers `raw` et `debayered` correspondent exactement à la lecture du capteur**, quel que soit le réglage — une mosaïque de Bayer
  pivotée ou retournée correspondrait à un motif de filtres de couleur différent, et les cartes d’étalonnage sont
  sur la grille propre au capteur.
- **Les fichiers `display`, `radiance`, `reflectance` et les fichiers d’index sont orientés** — pivotés une fois, après tous les
  calculs d’étalonnage. Leurs masques de limitation se trouvent sur la même grille.
- **Chaque fichier de capture enregistre le réglage** dans le fichier XMP : `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, dans le sens des aiguilles d’une montre, après
  les retournements) et `ChlorosOrientationApplied` — `false` sur les données brutes et débayérisées, `true` sur les
  produits orientés. `ChlorosProcessing` répertorie les opérations appliquées aux pixels de ce fichier (`none` lorsqu’il est
  au format natif). Cela inclut chaque image d’un enregistrement continu (réglage propre à la caméra,
  non appliqué) et d’un flux multi-caméras (l’identité : ses images constitutives sont natives, comme celles d’une
  capture matricielle).
- **Le traitement d’un projet applique l’orientation enregistrée** aux produits qu’il génère à partir d’un
  (calibrage dans les coordonnées du capteur d’abord, déformation d’alignement du réseau ensuite, orientation en dernier),
  afin qu’ils correspondent aux produits de la capture en direct — y compris les produits de secours non calibrés (réponse du capteur / correction de vignettage) ; son produit débayérisé reste natif.
- **La visionneuse d’images Chloros affiche un fichier brut ou débayérisé dont l’orientation a été modifiée par son enregistrement** — à des fins d’affichage uniquement,
  le fichier n’est pas modifié ; un fichier sans enregistrement est affiché tel qu’il a été stocké.
- **Les captures sans enregistrement sont traitées comme natives.** Avant la version 1.3.0, une capture à caméra unique
  convertissait elle-même ses fichiers bruts et débayérisés ; Chloros ne peut pas distinguer un tel fichier brut d’un fichier natif et
  ne fait aucune supposition.
- **Les captures en matrice conservent par défaut chaque fichier membre au format natif** et enregistrent son identité ;
  `ArraySession.capture(orient=True)` traite leurs produits (voir les commandes de capture de matrice).

#### Sortie par bande et matrices mixtes

Aux niveaux `radiance` et `reflectance`, une caméra renvoie **un plan par bande de cette
caméra** — trois pour une M3C (ses trois bandes non mélangées, ordre des canaux BGR), un pour une M3M (son
filtre à bande étroite unique). Les plans dupliqués de la caméra mono sont regroupés, de sorte qu’un **réseau mixte
M3C/M3M produit des configurations de matrice différentes pour chaque série**. Consultez la clé `bands` dans les métadonnées de l’image
pour connaître la nature de chaque plan ; ne la déduisez pas du nombre de canaux :

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Vérifiez toujours `hdr["processing_level"] == "reflectance"` avant d’utiliser ces valeurs. Une rétrogradation
(pas de lecture DAQ, pas de pack d’étalonnage, configuration de bandes inconnue) est signalée dans `processing_level` et
`radiometric_refused` plutôt que signalée comme élevée — le réseau que vous récupérez reste une trame valide, mais
ce n’est pas le produit que vous avez demandé. Une capture enregistrée qui a été rétrogradée porte le même code dans son XMP
que `Chloros:RadiometricRefused` (par exemple `no-band-layout`, `no-dls-snapshot`).

#### Un déclenchement par lecture — `group_id`

**Les éléments du réseau sont fournis à partir du groupe d’images synchronisées du réseau, et non à partir du
tampon de capture propre à chaque caméra.** Les caméras d’un réseau synchronisé par matériel exposent ensemble, mais les N lectures par caméra
ne sont pas *échantillonnées* ensemble — la lecture de la dernière image mise en tampon de chaque caméra chevauche les, et une seule image perdue désynchronise l’ensemble sans que cela ne se remarque.

La fourniture à partir du groupe est nécessaire mais pas suffisante en soi : chaque requête
capture *le dernier* groupe à son propre instant, et la boucle de capture peut publier entre-temps.
D’après des mesures effectuées sur un ensemble de 4 caméras en direct, **12 lectures parallèles sur 20 chevauchaient deux groupes** — toujours
exactement adjacents. La lecture est donc **fixée** :

1. Chaque image porte `group_id`, le compteur de publication monotone du réseau.
2. `grab()` lit la caméra maître (ou, si celle-ci ne peut pas répondre, la caméra suivante capable de le faire),
   puis fixe les caméras restantes à l’identifiant de groupe sur lequel elle s’est posée (`?group_id=N` sur le point de terminaison HTTP).
3. Le backend conserve les groupes récents pendant **au moins une demi-seconde** ; ainsi, un groupe déjà
   supplanté par des publications ultérieures continue d’être desservi — suffisamment longtemps pour permettre le, même sur un réseau complet de 9 caméras à capteur intégral. Une attribution que le backend ne
   conserve plus est refusée avec les codes `409` et `group_aged_out: true`, plutôt que d’être traitée silencieusement
   par un déclencheur différent — `grab()` relit l’ensemble.

Nouvelle mesure avec épinglage : **20 lectures sur 20 ont renvoyé un seul déclencheur.** `grab()` s’en charge pour vous ;
vous n’avez besoin de `group_id=` que lorsque vous interrogez directement le point de terminaison HTTP.

- Une caméra qui ne peut pas être desservie au sein du groupe épinglé — soit parce qu’elle en est absente, soit parce qu’elle refuse la requête
  (un `band` qu’elle ne possède pas, un niveau qu’elle ne peut pas produire) — est **omise plutôt que remplacée a posteriori**
  par un autre déclencheur, et les autres caméras sont toujours renvoyées.
- Le dictionnaire renvoyé par `grab()` est un `GrabResult` (une sous-classe simple de `dict` ;
  `from chloros_sdk import GrabResult`) : `.errors` associe chaque
  numéro de série omis à l’exception levée lors de sa lecture, et `.group_id` correspond au groupe partagé. Un
  `.errors` vide signifie que toutes les caméras font partie de ce groupe.
- Une caméra autonome n’appartient à aucun groupe, signale `group_id: None` et **rejette** une broche `group_id=`
  plutôt que de l’ignorer.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Le même compteur est publié sur **chaque flux MJPEG par caméra** d’un membre du réseau — l’
aperçu (`/api/camera/stream/<serial>`), l’aperçu annoté (`/stream-annotated`) et le
flux de réflectance (`/stream-reflectance`) — sous le nom `X-Chloros-Group-Id` sur chaque partie, en plus de
`X-Chloros-Frame-Id` et `X-Chloros-Timestamp-Ns`. Les flux sont continus plutôt que
ponctuels ; ils sont donc mis en correspondance en collectant les parties et en les regroupant sur cet en-tête — il n’y a pas de
broche à configurer. Une caméra autonome n’appartient à aucun groupe. Voir
[Flux MJPEG par caméra](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` ne se met jamais en veille.** Une requête qui échoue pour une raison pouvant se résoudre d’elle-même
est ignorée — un flux de caméra ne renvoie rien pour celle-ci, un flux de tableau renvoie les caméras qui ont
répondu (éventuellement aucune). Ces raisons sont les suivantes : pas encore d’image ; le thread de capture de la caméra s’est mis en pause pendant une
seconde ou deux (`404 not streaming`, auquel le backend répond chaque fois qu’une autre caméra se connecte
ou se déconnecte, ou qu’un changement de paramètre rééquilibre le pool); un redémarrage du backend ou `5xx` ; l’alignement
est encore en cours de stabilisation (`409 … retry shortly`) ; et un refus de réflectance car le DAQ n’a pas encore de mesure
(`no-dls-snapshot`, une mesure périmée ou vide). Un refus au tick suivant ne ferait que se répéter — un
`band` inconnu, `bit_depth=8` sur la radiance, un pack de calibration sans échelle de radiance, un refus de réflectance
avec **aucun DAQ du tout** (aucun lié à la caméra ou à son réseau, aucun connecté au
backend : le `dls_source` du refus est `"none"`), tout autre `4xx` —
génère un `chloros_sdk.FrameFetchError` (un `RuntimeError`, transportant `.status` et `.body`)
simultanément. Il en va de même pour **60 secondes** de ticks sautés sans trame entre eux, quelle que soit la valeur de `fps` — suffisamment
long pour laisser le temps à l’alignement de se stabiliser et au backend de redémarrer, mais suffisamment court pour qu’une caméra
a bel et bien disparu mette fin au flux. Cette erreur d’abandon est également un `FrameFetchError`, tant pour une caméra que pour
un réseau de caméras, l’exception de la dernière lecture étant `.errors` (et, pour une caméra, sous la forme `__cause__`). Pour
un tableau, « refusé » signifie que toutes les caméras ont été refusées ; l’exception `.errors` contient la
raison de chaque caméra.

**Un tableau `frame_stream()` génère chaque déclencheur une seule fois** (de même que `ArrayHandle` et `ArraySession`). Un
tick dont l’ensemble le plus récent correspond au déclencheur qu’il a déjà restitué attend le déclencheur suivant au lieu de
restituer à nouveau le même ensemble ; le flux s’exécute donc à la cadence la plus lente entre `fps` et la cadence propre au tableau, et les ensembles consécutifs ne partagent jamais un `group_id`. Un tableau qui cesse de publier génère
`FrameFetchError` (« aucun nouveau déclencheur depuis 60 s ») plutôt que de répéter son dernier ensemble. Sans cela, un
flux demandant plus que ce que le tableau publiait — ou démarrant juste après la connexion — générait
des répétitions : lors d’un test, 20 séries comportaient 11 identifiants de groupe distincts.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportation d’une seule bande — `band`

`grab()` / `frame_stream()` et `band="<name>"` permettent d’exporter une seule bande d’une caméra sous la forme d’un
tableau `(H, W, 1)` monocanal — de même forme que celle produite par une caméra monochrome ; la sortie à une seule bande est donc
traitée de manière identique, qu’elle provienne d’un M3M ou de la division d’un M3C. La bande rouge d’un FRGN :

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Les orthographes acceptées sont le nom complet de la configuration (`Red_660`, `NIR_850`, `Green_550`) ou son
prénom (`red`, `nir`, `green`), sans distinction de majuscules/minuscules. Un nom inconnu est rejeté et la liste des noms disponibles
s’affiche.

> **La sélection s’effectue par nom, et non par numéro de canal — c’est délibéré.** « Canal 1 » correspond à `Green_550` dans
> l’ordre des canaux BGR du réseau, mais à *rouge* dans l’ordre des lettres du code du filtre(`RGN`), et les deux
> ne concordent pas. Choisir le mauvais donne une trame parfaitement plausible mais de la mauvaise bande, que
> rien en aval ne peut détecter. C’est pour cette raison qu’il n’y a pas d’index numérique.

#### Alignement des flux — `align`

`align=True` transpose l’image dans le cadre de référence commun de son tableau. Chaque caméra est transposée
selon la **même**résolution de profil et la**même** zone d’intérêt (ROI) de recadrage, de sorte que les flux par caméra ouverts indépendamment
soient **mutuellement alignés** — vous n’avez pas besoin de les demander ensemble :

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Exigences et comportement :

- La caméra doit faire partie d’un réseau **synchronisé au niveau matériel** (câble de synchronisation M8) disposant d’un profil d’alignement
  — l’alignement suppose que les caméras soient exposées simultanément, ce qu’un réseau fonctionnant en mode logiciel ou
  en mode libre ne le permet pas. Refusé avec `409` dans le cas contraire.
- **Refusé, jamais désaligné en silence.** Le chemin de capture peut enregistrer des données désalignées avec un avertissement
  car un fichier conserve ses balises de provenance et peut être réenregistré ultérieurement ; un flux en direct ne dispose pas
  d’une telle seconde chance, donc l’absence de profil constitue une erreur plutôt qu’un repli.
- **La demande d’alignement le déclenche.** Demander `align` sur un réseau sans profil lance le
  calibrage et renvoie `409` avec ce qu’il attend (par ex. *« en attente que l’exposition
  se stabilise à 214000551 (1/3) »*) — réessayez et cela réussira, généralement en quelques dizaines de secondes.
  C’est important en mode sans interface graphique : l’alignement automatique au moment de la connexion abandonne après 30 s, et toute tentative
  effectuée pendant ce délai est ignorée tant que l’AE intelligent est encore en cours de stabilisation ; ainsi, un tableau dont l’exposition
  converge lentement se retrouvait auparavant sans profil pour toute la session et sans moyen d’en obtenir
  un sans ouvrir la vue combinée du bureau. `alignment_status` signale également
  `ae_align_holdouts`, en indiquant les caméras encore en phase de stabilisation et leur progression.
- Attribuez à chaque flux d’un même ensemble le **même `max_h`**, sinon leurs recadrages se retrouveront sur des grilles de pixels
  différentes.
- L’alignement est appliqué *avant* l’encodage de la profondeur de bits, de sorte que la déformation interpole des valeurs en pleine précision.

#### `bit_depth` — compromis entre précision et bande passante

`grab()` et `frame_stream()` acceptent `bit_depth=16` (par défaut) ou `bit_depth=8` :

| `bit_depth` | Codage de la réflectance | Coût |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, marge jusqu’à ρ 2,0 | Précision totale. |
| `8` | uint8 couvrant `0..refl_max` (ρ par défaut 1,0) | Moitié des octets ; quantification d’environ 0,004 ρ au plafond par défaut. Les valeurs supérieures au plafond sont écrêtées — voir ci-dessous. |

##### Le plafond de 8 bits — `refl_max`

`refl_max` définit la valeur de ρ que couvre la plage de 8 bits, ainsi `pixel_scale = 255 / refl_max` et ρ reste
`pixel / pixel_scale`. `grab()` et `frame_stream()` le considèrent comme un mot-clé à la fois sur la caméra et
des descripteurs de tableau (`?refl_max=` sur le point de terminaison HTTP). Augmentez-la pour gagner de la marge pour les reflets spéculaires,
par paliers proportionnellement plus grossiers :

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | pas |
| --- | --- | --- |
| `1.0` (par défaut) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **L'exposition automatique ne peut pas corriger un écrêtage 8 bits, et tenter de le faire ne fait qu'empirer la qualité des données.**
> ρ = π·L/E, l’exposition et le gain étant éliminés du calcul ; ρ est donc une propriété de la scène, et non des
> réglages de l’appareil photo — mesuré sur un capteur en temps réel, diviser par deux l’exposition a déplacé la moyenne de ρ de 1,3 %. Augmenter
> l'exposition pour « adapter » la plage entraîne au contraire une saturation des pixels au niveau du **capteur**, ce qui
> sous-estime L et fausse ρ vers le **bas** sans signaler d'erreur (mesure : ρ max. 1,203 → 0,985 à une
> exposition doublée, soit une baisse moyenne de −23 %). Choisissez la plage avec `refl_max` ; empêcher le capteur d’entrer en
> saturation, c’est justement le rôle de l’exposition automatique (AE), et cela protège la *validité* de ρ plutôt que sa plage.

**Lisez `reflectance_clipped_fraction`, et non l’indicateur.** `reflectance_clipped_at_1_0` se déclenche sur un
seul pixel — sur une scène réelle, il a signalé `True` alors que la fraction était `0.000001`, soit un pixel
sur un million. C’est la fraction qui vous indique s’il faut relever le plafond.

**`bit_depth=8` concerne uniquement la réflectance.** Une requête pour `radiance` est rejetée plutôt que
satisfaite : la réflectance a une plage fixe comprise entre 0 et 1 qui résiste à un encodage sur 8 bits, tandis que la radiance n’a pas
de plage fixe ; pour l’adapter à 8 bits, il faut donc un étirement par centile à chaque image, dont la correspondance
change à chaque image — il s’agit d’une image, pas d’une mesure.

#### Flux de réflectance en direct (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` fournit une réflectance de 8 bits par bande sous la forme
`multipart/x-mixed-replace` — un fichier M3C au format JPEG à 3 canaux, un fichier M3M en niveaux de gris. Paramètres de requête :
`fps` (valeur par défaut : 10), `jpeg_q` (valeur par défaut : 95), `max_h` (0 = natif), `band` (une bande nommée, comme
ci-dessus), `align` (`1` à co-enregistrer — mêmes règles que pour `align` du SDK), `refl_max` (le plafond de codage 8 bits, comme ci-dessus). Chaque partie comporte
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` et `X-Chloros-Reflectance-Clipped`.
**Les parties provenant de caméras différentes et portant le même `X-Chloros-Group-Id` correspondent au même déclencheur** —
c’est ainsi que plusieurs flux par caméra d’un même réseau sont réassemblés, et cela est garanti
côté serveur car les éléments du réseau sont fournis à partir du groupe de trames synchronisées (voir
[Un déclencheur par lecture](#un-déclencheur-par-lecture--group_id)). Il renvoie `409` lorsque la réflectance n’est
pas disponible au moment de la connexion et met fin au flux si elle devient indisponible en cours d’exécution ; ainsi, un flux bloqué
signifie « plus de réflectance » plutôt que « toujours correct, juste statique ».

> **Le format JPEG est avec perte — ces pixels constituent une mesure.** Les erreurs de compression se répercutent directement sur les
> valeurs de réflectance et sont plus marquées au niveau des bords, là où se trouvent les limites entre les matériaux. La qualité
> est fixée par défaut à 95 pour cette raison. Si vous avez besoin que les chiffres restent intacts, utilisez
> `grab(processing="reflectance")` / `frame_stream(...)`, qui renvoient le même résultat sans
> codec dans le chemin d’accès (et `bit_depth=8` si la contrainte porte sur le coût de la connexion et non sur le codec).

> **Les données EXIF sont conservées lors de l’exportation.** `process()` copie le bloc GPS de la capture source
> **et son ExifIFD** sur chaque produit ; les exportations contiennent donc `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` et `CameraSerialNumber`, ainsi que le
> géoréférencement. C’est à partir de `FocalLength` que Pix4D calcule la distance d’échantillonnage au sol — sans cela,
> la reconstruction se retrouve à une échelle complètement erronée (dans un cas concret, un site de 411 m
> a été transformé en un site de 47,8 km). La copie n’est délibérément pas `-all:all` : les balises structurelles d’IFD0 perturbent
> la sortie de LATTICE, et `ExifImageWidth`/`Height` sont exclus car ils décrivent la
> capture source plutôt que le raster exporté.

Sous-indicateurs de la phase de capture (s'appliquent aux niveaux radiométriques — `radiance`, `reflectance`, `display`) :

| Indicateur | Par défaut | Signification |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + champ plat + démixage 3x3 + échelle radiométrique NIST. |
| `apply_white_balance` | `True` | Table de conversion (LUT) de balance des blancs. Prise en compte du DLS lorsqu’un DAQ est associé à la caméra. |
| `apply_index` | `False` | Évaluation de l'indice de végétation. |
| `index_expression` | `None` | Remplacer la formule. Sivide → active automatiquement l’indice. |
| `annotated` | `False` | Superposition des décorations de l’interface graphique (zèbre/grille/crête). Non disponible pour `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **La valeur de retour est un `chloros_sdk.CapturePaths`, de type `CapturePathMap`, et non `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` correspond à `Dict[str, Union[str, List[str]]]` : un chemin unique
> `processing` attribue un chemin à chaque numéro de série, tandis que `processing="all"` lui fournit la
> **liste ordonnée** de tous les produits enregistrés pour cet appareil photo. `ArrayHandle.capture()` n’accepte
> aucun `levels=`, et il ignore les arguments-clés qu’il ne connaît pas (`levels`, `daq`,
> `render_index`, `force_daq`) ; ne transmettez donc que ceux indiqués ici (`aligned=False`
> enregistre les fichiers natifs, comme sur `ArraySession.capture()`) ; utilisez
> `ArraySession.capture()` pour ces contrôles. Un composite combiné en direct, s’il était diffusé en continu, arrive sous la clé supplémentaire
> `"combined"` plutôt que sous un numéro de série. Le code qui suppose `str` plante sur la
> forme de liste sans que le vérificateur de types ne s’y oppose — l’annotation indiquait `Dict[str, str]`
> pendant un certain temps après la publication de la forme liste, ce qui explique l’existence de l’alias. Normalisez
> lorsque vous souhaitez la forme plate :
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Ce qui ne pouvait pas être écrit : `.failed`.** La carte renvoyée par `capture()` est une
> `chloros_sdk.CapturePaths` — une simple sous-classe de `dict`, donc tout ce qui précède fonctionne
> sans modification — qui contient également `.failed` : les fichiers que le backend n’a pas pu écrire,
> les mêmes entrées que [`CaptureResult.failed`](#fichiers-qui-n’ont-pas-pu-être-écrits--échec-de-la-capture)
> (`serial`, `level`, `error` et `disk_full: True` sur un disque plein). Une
> capture partielle renvoie tout de même tous les chemins qui ont été écrits ; sur un disque plein, la carte est
> vide et le fichier `.failed` en explique la raison. `.skipped` répertorie les sauts par caméra et
> `.group_seq` le groupe synchronisé, et `.alignment` / `.oriented` correspondent à ceux
> du backend, comme sur `CaptureResult`. `capture_smart()` les renvoie sous la forme
> `result["frames"].failed` et `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Alignement des tableaux

`ArrayHandle` expose l’intégralité de la surface d’alignement. Par défaut, les profils sont liés à la session — appelez explicitement `export_alignment()` pour les persister.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alignement à la connexion

`connect_all(align=...)` permet d’aligner automatiquement chaque tableau à la connexion :

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Si non spécifié, le système utilise par défaut `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

Un capteur de lumière enregistré (DAQ-U, DAQ-M ou DAQ-E), ouvert par `connect_all()` dans le [pool de capteurs DAQ](#daq-sensor-sessions) — le même pool que celui utilisé par `connect_daq_sensor()` et `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` attend le **prochain** spectre du capteur — celui déjà mis en mémoire tampon au moment de l'appel ne compte pas — et renvoie `(spectrum, is_saturated, integration_time, x, y, z)`, ou `None` si aucun n’arrive dans les 5 s. Un flux mis en pause est repris. `spectrum` est exprimé en W/m²/nm lorsque le capteur est étalonné.
- `.sensor` correspond à la [`DAQSensorSession`](#daq-sensor-sessions) du capteur ; pour tout le reste : `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` et `.settings` constituent l'entrée sauvegardée à partir de `sensors.json`.

**Ce que `connect_all` restaure.** Chaque capteur est ouvert à son adresse enregistrée — son port COM, son adresse Bluetooth ou son hôte DAQ-E — et, lorsque le projet a enregistré l’identifiant du capteur, il doit s’agir de ce capteur : une autre unité à cette adresse est refusée, car non connectée. Son temps d’intégration, sa moyenne par trame et son exposition automatique enregistrés sont appliqués, ainsi que sa correction de cap :

| Capteur enregistré | Connecté avec |
| --- | --- |
| Un profil de capteur (par ex. `fov_30`) | Ce profil. |
| Aucun profil sélectionné | `sunshine_cosine`, le profil par défaut de connexion, enregistré comme supposé (`cap_id_source=auto_default`), ce qui déclenche un avertissement lors du traitement. Les capteurs sont livrés nus : choisissez le capuchon qui est monté. |
| `none` — sélectionné comme **Aucun (capteur nu)** dans l’interface graphique | Aucune correction de capuchon. |

Un DAQ-E-S n’accepte aucun profil de capuchon, quel que soit celui qui a été enregistré : son diffuseur fait partie de son étalonnage (voir [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Un projet enregistré par une version antérieure d’Chloros avait enregistré un capteur nu comme option par défaut ; il se reconnecte donc avec le `sunshine_cosine` ; sélectionnez à nouveau **Aucun (capteur nu)** dans l’interface graphique et enregistrez le projet.

Un capteur qu’un autre client a déjà ouvert dans le pool (via l’CLI ou un autre script) est connecté tel quel, sans réapplication des paramètres, et `disconnect_all()` le laisse en fonctionnement ; un capteur ouvert par `connect_all()` est libéré. Comme pour `connect_daq_sensor()`, l’interface graphique Chloros **ne**fait**pas** partie du pool : fermez un capteur dans l’interface graphique avant de connecter le projet à partir d’un script.**Configuration requise et rapport.** Les capteurs de lumière nécessitent un backend Chloros en cours d’exécution et connecté à un forfait Chloros+ payant (Copper ou supérieur), et `connect_all()` n’en lance jamais. En l’absence de backend en cours d’exécution, tous les capteurs enregistrés sont répertoriés sous `report["sensors"]["errors"]`, avec une mention à cet effet. Un capteur que le backend ne peut pas ouvrir y est également répertorié, avec la raison indiquée par le backend ; `report["sensors"]["missing"]` reste vide pour les capteurs de lumière.

---

## Matériel direct (sans backend)

Si vous souhaitez ne dépendre d’aucun backend pour les caméras LATTICE (CI, robots sans interface graphique, systèmes embarqués), importez directement `lattice_sdk` — il est réexporté par `chloros_sdk`. Remarque concernant `CAMERA_AVAILABLE` : `lattice_sdk` figure dans la roue SDK, mais nécessite la présence du runtime Arena SDK et d’OpenCV (`cv2`), et la roue n’installe ni l’un ni l’autre. Les capteurs de lumière passent par le backend — [`connect_daq_sensor()`](#daq-sensor-sessions) ou le composant `proj.sensors` d’un projet — à une exception près, l’assistant de magnitude de bande DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), qui nécessite également `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Préréglages et déclenchement

Trois des quatre préréglages fonctionnent en **mode libre** : la caméra expose en continu et un
`capture()` renvoie l’image suivante. `triggered` fait exception : il arme l’
appareil photo en attendant un front matériel sur la ligne 2, il ne capture donc rien tant qu’il n’en détecte pas.

| Préréglage | Déclencheur | À utiliser lorsque |
| --- | --- | --- |
| `default` | fonctionnement libre | usage général |
| `high_speed` | fonctionnement libre | 8 bits, limite à 60 images par seconde, exposition courte |
| `high_quality` | fonctionnement libre | 12 bits, pas de limite d’images par seconde — le choix habituel pour les photos |
| `triggered` | **armé, ligne 2** | la caméra est raccordée à un câble de synchronisation M8 et est déclenchée par un autre dispositif |

Si vous choisissez `triggered` (ou si vous configurez vous-même `trigger_mode="On"`) sans que rien
ne commande la ligne 2, chaque `capture()` expirera — ce qui est normal, puisque vous avez demandé
à la caméra d’attendre. Le SDK explique ce phénomène lorsqu’il se produit ; voir
[SC_ERR_TIMEOUT pendant la capture](#direct-hardware-backend-free).

> **Remarque — Les messages « GVSP probe » / `SC_ERR_TIMEOUT -1011` lors de la connexion ne sont pas des erreurs.**
> Lors de la connexion, le SDK tente de négocier des**trames jumbo** (paquets GVSP de 9 000 octets) pour un débit plus élevé. Sur une liaison réseau directe point à point (par exemple, une adresse `169.254.x.x` locale), le réseau ne peut généralement pas prendre en charge les trames jumbo ; cette tentative aboutit donc à un délai d’expiration et génère des lignes de journal telles que :
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Il s’agit du **mécanisme de repli prévu** : le SDK revient automatiquement aux paquets standard de 1 500 octets et la caméra continue de se connecter normalement (les lignes `[chunk-enable …]` qui suivent font partie de la séquence de connexion normale). La capture fonctionne toujours.
>
> Vous pouvez ignorer cette sonde, mais **elle ne sert pas seulement à faire taire les journaux — elle désactive les trames jumbo.** La caméra ne répond aux pings « Don't-Fragment » que jusqu’à 1 500 octets, quelle que soit la qualité de votre réseau ; le test de ping seul ne peut donc jamais détecter les trames jumbo ; cette sonde est la seule à pouvoir le faire. Si vous la désactivez, la caméra utilisera indéfiniment des paquets standard de 1 500 octets, sur n'importe quel réseau :
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Cela ne vaut la peine que sur un réseau dont vous *savez* qu'il ne prend pas en charge les paquets « jumbo », où cela permet de gagner environ une seconde de temps de connexion par caméra. Comme il s’agit d’un véritable compromis et non d’une simple modification cosmétique, le SDK l’indique désormais lorsque vous l’utilisez :
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Ne le modifiez pas sauf si vous avez une bonne raison.**Si cette option reste activée, chaque caméra est mesurée une fois par session backend, puis à nouveau lorsque sa liaison change (autre adaptateur, autre débit de liaison ou coupure de liaison, comme lors du remplacement d’un commutateur) ; les reconnexions réutilisent le résultat.**Après un changement derrière un commutateur resté opérationnel, redémarrez le backend pour effectuer une nouvelle mesure.** Une première connexion peut parfois prendre quelques minutes lors de cette vérification, et le journal l’indique lorsque c’est le cas.
>
> Si vous *souhaitez* bénéficier du débit jumbo, activez le mode jumbo de bout en bout (MTU de la carte réseau à 9 000 + un commutateur qui les laisse passer), ou fixez-la avec `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` lorsque vous savez que la liaison le prend en charge — bien qu’il soit préférable d’utiliser `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` par commande plutôt que de la définir de manière permanente, car une taille fixe contourne la détection et empêche l’adaptation au réseau en amont. **Tous les** périphériques du chemin doivent prendre en charge les paquets jumbo — y compris tout répartiteur ou injecteur PoE, ce qui est généralement la raison pour laquelle une configuration par ailleurs compatible avec les paquets jumbo ne peut pas les acheminer.

> **`SC_ERR_TIMEOUT -1011` pendant `capture()` / `grab*()` est un problème différent — il s’agit là d’une véritable erreur.**
> La remarque ci-dessus concerne uniquement le code `-1011` enregistré par la**sonde de temps de connexion**. La même erreur générée par une**capture** signifie que la caméra s’est correctement connectée mais n’envoie aucune image :
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Le signe révélateur est une caméra dont le canal de *contrôle* fonctionne correctement — la détection fonctionne, les réglages et les écritures `[chunk-enable …]` aboutissent tous — alors que *chaque* image expire.
>
> **La cause habituelle est que la caméra est configurée pour un déclenchement matériel.** Avec les codes `trigger_mode="On"` et `trigger_source="Line2"`, la caméra n’émet absolument rien jusqu’à ce qu’un front d’onde électrique arrive sur le câble de synchronisation M8. Si aucun câble ne pilote cette ligne, chaque capture attend indéfiniment. La caméra n’est pas défectueuse et le réseau fonctionne correctement — elle fait exactement ce qu’on lui a demandé.
>
> Les préréglages `CameraSettings()`, `default`, `high_speed` et `high_quality` sont des préréglages en mode libre, et une capture qui expire alors qu’elle est activée affiche une explication au lieu d’afficher simplement « `-1011` ». `PRESETS["triggered"]` active la ligne 2, conformément à la conception.
>
> Pour forcer n'importe quelle caméra à fonctionner en mode libre :
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Si le délai d'expiration persiste avec `trigger_mode="Off"`, cela signifie que la caméra ne transmet réellement pas de données — envoyez-nous le journal et `ip link show`.

#### Profils de couleur (aperçu en direct RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` permet de sélectionner le profil de couleur de l'écran pour l'**aperçu en direct** sur les caméras RGB (les caméras multispec ignorent ce paramètre) :

| Profil | Signification |
| --- | --- |
| `raw` | Contourner entièrement la chaîne radiométrique. |
| `linear` | DSNU + flat + WB, pas de CCM, pas de gamma. |
| `natural` | Linéaire + CCM mesuré + gamma sRGB, avec uniquement la finition « cheap » (lissage de la chrominance + désaturation des hautes lumières) — le réglage par défaut réaliste. |
| `enhanced` | `natural` plus la finition « hub-parity » complète (suppression des franges, vibrance, contraste local CLAHE). Un rendu plus riche pour environ **le double du coût de finition par image**, d’où une fréquence d’images LIVE plus faible. |
| `custom_temp` | `natural` mais balance des blancs verrouillée sur `custom_cct_k` Kelvin (DLS ignoré ; limité à 2 000–10 000 K côté backend). |

Le profil est un régulateur de vitesse/rendu **réservé à l’aperçu en direct** : les captures enregistrées bénéficient toujours d’un rendu riche et complet, quel que soit le profil sélectionné ; ainsi, choisir `natural` pour gagner du temps par image ne réduit pas la qualité du fichier enregistré sur le disque. Un profil inconnu active `ValueError` ; lorsqu’un backend chloros est accessible, la modification lui est également transmise via POST afin que la prochaine image d’aperçu en tienne compte (les utilisateurs de l’SDK direct sans backend bénéficient tout de même de la mise à jour des paramètres).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Points forts (affichage RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` extrait les pixels les plus lumineux et les plus blancs de l’**affichage** sur les caméras RGB — cela corrige le problème d’une surface blanche brillante qui s’affiche comme une tache blanche uniforme. `value` correspond à `-100`..`0` (limité); `0` est désactivé, et le rendu correspond alors exactement à ce qu’il serait sans ce paramètre. Des réglages plus élevés réduisent le blanc de l’écran (à environ 60 % à `-100`) et élargissent la courbe de tonalité, ce qui permet à une surface que la luminosité automatique de l’aperçu avait poussée vers le blanc pur de retrouver ses détails.

Il s’agit d’un réglage d’affichage, mais contrairement à la saturation et au contraste (qui ne modifient que l’aperçu en direct), il modifie également ce que vous enregistrez : l’aperçu en direct et les captures **au niveau de l’affichage** s’y conforment, de sorte que ces fichiers sont également plus sombres (ils portent la valeur `Chloros:DisplayHighlights`) ; les exportations de radiance et de réflectance ne le font jamais, et cette option ne permet pas de récupérer les données du capteur qui ont été écrêtées — réduisez l’exposition pour cela. Les caméras multispec et mono l’ignorent. La valeur est stockée dans `settings.highlights` et envoyée via POST à un backend chloros accessible, exactement comme `set_color_profile` ; l’équivalent en CLI est `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Caméras mono (M3M) et `Calibration`

Une caméra mono **M3M** (`M3M-<lens>-F<wavelength>`) est monobande : un seul plan en niveaux de gris, pas de mosaïque de Bayer, pas de matrice de diaphonie spectrale 3×3. `Calibration` la reconnaît et expose un indicateur `is_mono`. La réflectance s'applique toujours sous forme de carte radiométrique par bande (la décomposition est la matrice identité), mais les calculs multibandes sur une seule caméra donnent des résultats valides plutôt que des valeurs absurdes :

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Pour calculer un indice de végétation à partir d’un matériel monochrome, combinez plusieurs caméras M3M à différentes longueurs d’onde en une pile multibande alignée (voir [Alignement de la matrice](#array-alignment)) et calculez l’indice sur l’ensemble de cette pile plutôt que sur une seule caméra.

#### Magnitudes des bandes DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` se connecte à un DAQ-M via Bluetooth grâce à sa propre adresse MAC BLE. Il s’agit de la seule classe de capteur de lumière qui communique directement avec le capteur plutôt que via le backend ; elle nécessite donc les éléments requis par les classes de caméras directes (`CAMERA_AVAILABLE` : le runtime de l’SDK Arena et OpenCV), ainsi que `bleak` pour le Bluetooth (`pip install "chloros-sdk[camera]"` ou `pip install bleak` ; sans cela, `DLS` déclenche `DLSError`). Un DAQ-M connecté ailleurs (le pool du backend, un projet, l’interface graphique) cesse de se signaler ; il faut donc d’abord le libérer à cet endroit. **`mac_address` est obligatoire** : il n’y a pas de valeur par défaut, car chaque unité possède la sienne, et `DLS()`, en l’absence de cette valeur, déclenche `DLSError` avant toute tentative de connexion. Rechercher l’adresse MAC avec `discover_daq_sensors(transports=["ble"])` (voir [Détection](#discovery--finding-an-address-to-connect-with)) ou `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` intègre un spectre DAQ-M (la dernière lecture lorsque `spectrum` est `None` ; `DLSError` s’il n’y en a pas encore) à travers les réponses des trois canaux d’une caméra et renvoie `{"red": …, "green": …, "nir": …}` — les clés désignent les canaux dans cet ordre ; avec les courbes d’une caméra RGB, le troisième (`nir`) correspond au canal bleu.

- Sans `spectral_response`, il utilise la courbe de la flotte du modèle de filtre (celle avec laquelle le `filter_model` a été construit), qui a été mesurée sur un seul appareil.
- `spectral_response` utilise plutôt les courbes mesurées **propres** à la caméra : un dictionnaire `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, tel qu’il est stocké dans le pack d’étalonnage de cette caméra (`color_correction.spectral_response_smoothed` — caméras RGB ; les packs multispectraux ne contiennent pas ce type de courbe). En l’absence de dictionnaire utilisable, le système revient à la courbe de la flotte.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

La commande `chloros-cli lattice dls --camera SERIAL` de l’CLI effectue la même opération via le backend, en recherchant la courbe de la caméra dans son pack d’étalonnage chargé ou mis en cache, et indique quelle courbe a été utilisée. Sur HTTP, cela correspond à `POST /api/dls/cli-read` avec `mac` (obligatoire : sans cela, la réponse est un code HTTP400, `code` `MAC_REQUIRED`) et `camera_serial` dans le corps ; le `band_magnitudes_source` de la réponse est `camera <serial>` lorsque la courbe propre à cette caméra a été utilisée, ou `model <filter>` pour la courbe de la flotte (une caméra sans courbe : pas de RGB, ou aucun pack chargé ou mise en cache).

---

## Exceptions

Interceptez la classe de base pour gérer « tout ce qui a mal tourné dans Chloros » :

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` et `ChlorosConfigurationError` sont des attributs de `chloros_sdk` (et de `chloros_sdk.exceptions`) mais pas dans `__all__` ; par conséquent, `from chloros_sdk import *` ne les importe pas : importez-les par leur nom. La famille `ChlorosError` est dérivée de `ChlorosLocal` et des fonctions qui en découlent (`process_folder()`, `process_lattice_capture()`) ; la surface Smart-Connect hérite de `ChlorosConnectError`.

Hiérarchie :

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exemples de bout en bout

### 1. Traiter un dossier avec une barre de progression personnalisée

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matrice LATTICE en temps réel → Réflectance + référence DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campagne de capture axée sur un projet

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Flux d’images multi-caméras → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de capture « headless » en mode matériel direct (sans backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Test de compatibilité avant la connexion d’un réseau de 4 caméras

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Équivalent d’une recette de capture (Python pur)

`run_project_recipe()` exécute une recette CLI telle quelle (voir [Dossier du projet et recettes](#project-folder--recipes)). Écrites en Python, les étapes sont les suivantes :

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Serveur MCP (assistants IA)

`chloros-mcp` permet à un assistant IA utilisant le [Model Context Protocol](https://modelcontextprotocol.io) (MCP), tel que Claude Desktop, Claude Code ou Cursor, de piloter Chloros : traiter des images (créer un projet, importer un dossier de captures, choisir les paramètres, effectuer le traitement, générer un rapport sur les résultats exportés) et de piloter les caméras LATTICE, les réseaux de caméras et les capteurs de lumière DAQ (connexion, prévisualisation, réglage de l’exposition et des couleurs, capture, enregistrement, mise à jour du micrologiciel). Ses outils couvrent les fonctionnalités de ce SDK. Vous formulez votre demande en langage naturel et l’assistant appelle les outils du serveur :

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Le serveur repose sur ce SDK : les outils de traitement appellent [`ChlorosLocal`](#image-processing--chloroslocal) et les outils matériels les [appels smart-connect](#smart-connect-for-lattice-cameras), les [sessions DAQ](#daq-sensor-sessions), `ChlorosProject` et les [utilitaires de caméra](#camera-utilities), qui communiquent tous avec le même backend que celui utilisé par l’application Chloros. Ses requêtes portent le code `X-Chloros-Client: sdk`, de sorte que le backend applique les vérifications de connexion et de forfait du SDK, ainsi que les règles associées (la fonctionnalité « Texture Aware » est refusée pour les captures LATTICE, par exemple).

### Installation

Le serveur est fourni dans le paquet `chloros-sdk` ; il n’y a rien d’autre à installer. Le SDK installe une commande `chloros-mcp`, qui est identique à `python -m chloros_sdk.mcp_server` :

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows :** le programme d’installation de Chloros installe le SDK via pip dans votre environnement Python, ce qui place `chloros-mcp.exe` dans le dossier `Scripts` de cet environnement Python.
- **Sous Linux (.deb) :** le paquet installe le SDK pour le système `python3` et place la commande `chloros-mcp` dans `/usr/bin`, de sorte qu’elle se trouve dans le répertoire `PATH`. Dans un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` ajoute `chloros-mcp` au répertoire `bin` du virtualenv.

Le serveur utilise uniquement la bibliothèque standard fournie avec le SDK (Python 3.7+). Un client MCP le lance et communique avec lui via stdin/stdout ; vous ne devez pas l'exécuter vous-même, sauf avec `--help`.

### Configuration du client

**Claude Desktop :**ajoutez le serveur à `claude_desktop_config.json` (sous Windows, `%APPDATA%\Claude\claude_desktop_config.json` ;**Paramètres > Développeur > Modifier la configuration** permet d'y accéder), puis redémarrez Claude Desktop :

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code :**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor :** la même entrée `mcpServers` que pour Claude Desktop, dans `~/.cursor/mcp.json` (pour tous les projets ; `%USERPROFILE%\.cursor\mcp.json` sous Windows) ou `.cursor/mcp.json` dans un projet.

Les options se placent dans `args`, ou après la commande pour Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Pour utiliser un SDK d’une version particulière à la place de celui de `chloros-mcp` sur le `PATH`, exécutez le module avec ce Python (son chemin d'accès complet sous la forme `command` s'il ne s'agit pas du premier `python3` sur le `PATH`) :

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Un client démarre le serveur sans votre shell ; par conséquent, s’il signale que le serveur n’a pas pu démarrer, `chloros-mcp` ne se trouve probablement pas sur le `PATH` que le client voit. Indiquez plutôt le chemin d’accès complet (`where chloros-mcp` sous Windows, `which chloros-mcp` sous Linux, où le fichier .deb se trouve à l’emplacement `/usr/bin/chloros-mcp`), ou exécutez le module avec la version de Python qui intègre le SDK.

### Options

| Option | Par défaut | Description |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Le backend Chloros à utiliser. |
| `--no-auto-start` | désactivé | Ne jamais démarrer de backend : utiliser un backend déjà en cours d’exécution, tel que celui de l’application Chloros, le service de backend Linux, ou un tunnel vers le (voir [Mode backend distant](#remote-backend-mode-pip-only-host-via-tunnel) ; les chemins d’accès aux dossiers doivent alors désigner le même dossier sur les deux machines). |
| `--backend-exe PATH` | là où Chloros l’installe | Le backend à démarrer lorsqu’aucun n’est en cours d’exécution, pour un Chloros installé à un emplacement que le SDK ne prend pas en compte. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` ou `ERROR`. Les journaux sont envoyés vers stderr, que la plupart des clients affichent dans leur journal MCP ; stdout ne contient que les messages de protocole. |

### Outils

Les arguments de chaque outil sont ceux de l’appel SDK qu’il encapsule ; le client répertorie chacun d’entre eux avec son schéma complet. Les chemins d’accès sont absolus.

**Traitement**

| Outil | Appel SDK | Fonction |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Crée un projet et en fait le projet ouvert, sur lequel agissent les outils de traitement. Un nom simple crée le dossier dans le dossier de projet par défaut ; un chemin absolu le crée à cet emplacement. |
| `open_project` | `ChlorosLocal.open_project` | Ouvre un projet enregistré et en fait le projet ouvert. Les chemins d'accès d'un projet déplacé sont réorientés vers le dossier dans lequel il se trouve désormais ; `relink_folder` (absolu) indique le nom du dossier dans lequel se trouvent désormais ses autres fichiers manquants. Renvoie `source_files` et `image_count`. Aucun fichier trouvé : une erreur indiquant leur emplacement et nommant `relink_folder` ; rien n’est ouvert. |
| `import_images` | `import_images` | Importe les paires `.RAW`+`.JPG` de Survey3, les captures LATTICE `.tif` et des enregistrements DAQ `.daq` ; renvoie `count` (images) et `scan_count` (enregistrements). |
| `configure_processing` | `configure` | Modifie les paramètres transmis et aucun autre (valeurs : [Valeurs prises en charge](#supported-values)) ; `custom_settings` envoie l'intégralité de la configuration telle qu'elle est fournie. |
| `get_project_settings` | `get_config` | Lit les paramètres de traitement, d'exportation et d'indexation enregistrés du projet ouvert. |
| `start_processing` | `process(wait=False)` | Lance le traitement du projet ouvert et revient immédiatement. |
| `get_processing_status` | `get_status` | Signale `state` (`idle`, `starting`, `processing` ou `finished`) ainsi que la progression ; une fois terminé, le rapport de l'exécution indique `summary`, `export_check_failure` lorsque les exportations n'ont pas pu être vérifiées, et `image_failure` lorsque des images ont été perdues : les autres produits sont enregistrés ; signalez l'exécution comme ayant échoué. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Crée un projet pour un dossier, l'importe, configure et lance l'exécution en un seul appel (`lattice: true` pour les acquisitions LATTICE). Lorsqu’une exécution est en cours, elle est interrompue avant toute création, et le backend refuse alors également toute modification de projet depuis n’importe quel outil. |
| `export_light_sensor` | `export_light_sensor` | Écrit les fichiers calibrés `.daq` et `.csv` pour les enregistrements du capteur de lumière du projet sans traiter les images, en récupérant l’étalonnage d’usine depuis MAPIR Cloud lorsqu’il n’est pas mis en cache (voir [Enregistrements du capteur de lumière](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Lit les métadonnées d'audit Chloros d'une image traitée (source, étalonnage, étapes appliquées). |

**Caméras et réseaux de caméras**

| Outil | Appel SDK | Fonction |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Ce qui est actuellement connecté, sur n'importe quel client, chacun avec `connected_here` ; ainsi que les tâches en cours d'exécution sur le serveur. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Caméras et capteurs de lumière pouvant être connectés. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | État en temps réel d’une caméra, d’un réseau de caméras (avec ses capacités et son état de santé) ou d’un capteur de lumière. |
| `analyze_array_network` | `analyze_array_network` | Capacités de cet hôte et de ce réseau pour un réseau de caméras proposé. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Connecter une caméra ou un ensemble synchronisé (maître en premier). |
| `connect_project_devices` | `open_project(...).connect_all` | Connecter tous les appareils enregistrés dans un projet sauvegardé, avec leurs paramètres. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Déconnecter une caméra, un réseau ou un capteur de lumière (voir Comportement). `keep_settings: true` sur un réseau enregistre les paramètres de chaque caméra dans sa propre mémoire, `clear_settings: true` réinitialise cette mémoire (son résultat porte la référence `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Les périphériques du projet, tous ensemble. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Lire ou écrire les paramètres de la caméra, par nom convivial ou par nom de nœud GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Fixer la fréquence d'un tableau ou rétablir la fréquence automatique. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Une capture (`fastest` et `wait_for_exposure` sont `capture_fastest` et la prise de vue à exposition stabilisée). En l’absence de `output_dir`, il enregistre dans le dossier ouvert du projetdossier `captures` du projet ouvert, et les fichiers sont intégrés au projet (mis en attente jusqu’à la fin d’un cycle de traitement) ; la réponse `project_add` l’indique. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Captures en continu ou par intervalles, sous forme de tâche. |
| `start_array_recording` | `ArraySession.record` / `burst` | Une vidéo à index combiné ou une rafale brute, sous forme de tâche. |
| `build_burst_video` | `ArraySession.build_video` | Retraiter une rafale pour obtenir une vidéo calibrée, sous forme de tâche. |
| `get_job_status` / `stop_job` | | Suivre ou arrêter une tâche. |
| `get_camera_preview` | | Aperçu en direct de chaque caméra, renvoyé sous forme d’image. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Une image (ou un ensemble synchronisé) à un niveau de traitement, sans enregistrement : statistiques par plan et fichier PNG correspondant pour chacune. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Alignement d'un tableau. |

**Capteurs de lumière**

| Outil | Appel SDK | Fonction |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Connecter un DAQ-U, DAQ-M ou DAQ-E (`cap_id` définit la correction de la limite). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Les dernières mesures : horodatage, saturation, plage spectrale et pic, ainsi que le spectre moyenné par intervalles de longueur d'onde (`full_spectrum` pour chaque point). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Démarrer ou arrêter le flux. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Enregistrer un fichier `.daq` en tant que tâche ; `duration_s` l'arrête automatiquement. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | La correction de la coiffe (`none` pour un capteur nu). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Quel capteur fournit la réflectance de chaque caméra et de chaque réseau ? |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Les ensembles de bandes, ainsi que la réflectance ambiante et celle de l'objet et les indices de deux capteurs en flux continu. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Paramètres propres à un DAQ-E, enregistrés dans l'appareil. |

**Utilitaires de la caméra** ([Utilitaires de la caméra](#camera-utilities))

| Outil | Appel SDK | Fonction |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` et leurs méthodes de définition | Couleur d’affichage d’une caméra RGB : profil et température de couleur, saturation / contraste, hautes lumières. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Balance des blancs en une seule prise, ou mode auto / désactivé / rapports fixes. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Préréglages de la caméra. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Modèles de filtres, ou un par nom. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Versions du micrologiciel et flash (une tâche par défaut). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | État d'alimentation et thermique ; veille ou réveil. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Coefficients provenant d’une cible d’étalonnage à l’écran. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Les adaptateurs réseau de la caméra, une estimation et la correction (Administrateur uniquement). |
| `apply_vignette_correction` | `apply_vignette_correction` | Corriger le vignetage d'un dossier d'images. |
| `read_dls` | `read_dls` | Une lecture DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Matériel et environnement d'exécution de l'hôte. |

**Caméras transitoires, profils d’alignement et indices** ([Matrices transitoires, profils d’alignement et indices](#transient-arrays-alignment-profiles--indices)). Chaque outil de cette section qui ouvre des caméras utilise `keep_settings` / `clear_settings` (valeur par défaut : false) : voir [Mémoire de la caméra](#camera-memory--keep_settings--clear_settings).

| Outil | Appel SDK | Fonction |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Enregistrer une caméra ou diffuser plusieurs caméras de manière synchronisée, sous forme de tâche (`duration_s` l'arrête automatiquement). |
| `capture_cameras_transient` | `capture_cameras_transient` | Une capture synchronisée sans connexion à un réseau de caméras. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Vérification de la synchronisation du déclenchement et identification des caméras présentes sur la ligne de synchronisation. |
| `analyze_array_models` | `analyze_array_models` | Ce qu'un ensemble de ces modèles obtiendrait sur ce réseau. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Résoudre et enregistrer un fichier de profil d’alignement (`options` contient les options de résolution plus avancées). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Une image multibande alignée, ou un flux de ces images sous forme de tâche. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Le contenu d’un fichier de profil : l’ordre des bandes et les caméras activées. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Préréglages d’index et dégradés ; index d’un fichier TIFF multibande ; capture et indexation en une seule étape. |

**Synchronisation horaire, dossier de projet et recettes** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Dossier de projet et recettes](#project-folder--recipes))

| Outil | Appel SDK | Fonction |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | L'état de santé du maître PTP, ses pairs et l'état PTP de chaque caméra matricielle. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Le redémarrer ; définir ses priorités. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Le dossier dans lequel les nouveaux projets sont créés. |
| `run_project_recipe` | `run_project_recipe` | Exécute la recette d’un projet enregistré, sous forme de tâche avec son journal. Vérification préalable : une recette comportant une erreur est refusée et rien n'est exécuté ; un avertissement est émis concernant les tableaux / capteurs non connectés (la connexion n'est pas établie : exécuter d'abord `connect_project_devices`). `dry_run` renvoie le résultat de la vérification et n'exécute rien. |

Il n’existe aucun outil permettant d’ouvrir un projet enregistré pour le traitement : chaque tâche de traitement commence par `create_project` (ou `process_folder`).

### Prérequis

- **Chloros installé sur le même ordinateur.** Le backend Chloros effectue le traitement. Le serveur peut également utiliser un backend accessible via un tunnel (`--backend-url` avec `--no-auto-start`, voir [Options](#options)).
- **Être connecté sur cet ordinateur**, dans l’application Chloros ou via `chloros-cli login`. Le serveur ne peut pas se connecter à votre place : un outil appelé alors que vous êtes déconnecté renvoie le message « Non connecté à Chloros » avec la raison fournie par le backend, et l’assistant vous en informe.
- **N'importe quel niveau payant d'Chloros+ (Copper ou supérieur).** Le serveur nécessite un accès SDK, et le niveau gratuit Iron n'en dispose pas : un outil appelé sur le niveau gratuit répond « Non disponible sur ce forfait Chloros » en indiquant la raison fournie par le backend.
- Python 3.7 ou version plus récente avec le SDK, et un client MCP capable de démarrer des serveurs locaux (stdio).

### Comportement

- **Le backend.** Le lancement du serveur ou la liste de ses outils ne déclenche aucune action ; c’est le premier appel d’outil qui établit la connexion. Si l’application Chloros (ou le service backend Linux) est en cours d’exécution, le serveur utilise son backend et le laisse en marche. Si aucun backend n’est en cours d’exécution, le serveur démarre celui qui est installé, sans fenêtre. Lorsque le serveur se ferme (vous quittez le client ou celui-ci redémarre le serveur), il arrête les tâches de capture et les enregistrements du capteur de lumière qu’il avait lancés et libère les périphériques auxquels il était connecté ; le backend continue de fonctionner, de sorte qu’un traitement inachevé se poursuit jusqu’à son terme, puis il s’arrête de lui-même au bout de 30 minutes s’il n’est utilisé par rien. Avec `--no-auto-start`, le serveur ne lance jamais de backend : un outil appelé alors qu’aucun n’est en cours d’exécution vous invite à ouvrir l’application Chloros, et l’appel suivant, une fois que vous l’avez fait, établit la connexion.
- **Un seul projet ouvert.** Le backend dispose d’un seul projet ouvert, partagé avec l’application Chloros lorsque celle-ci est ouverte ; ne travaillez donc pas sur un autre projet dans l’application pendant que l’assistant est en cours d’exécution.
- **Seuls les paramètres transmis changent.** `configure_processing` conserve chaque paramètre qui ne lui est pas fourni à sa valeur enregistrée (le SDK `configure()` réinitialise le débayérage, la correction de vignettage, l’étalonnage de la réflectance, le PPK et le format d’exportation à leurs valeurs par défaut lorsqu’ils ne sont pas fournis ; le serveur les récupère à partir du projet). S’il ne parvient pas à lire les paramètres enregistrés du projet, il ne modifie rien et l’indique, sauf si les cinq paramètres sont fournis. `indices` remplace la liste d’index du projet. Le résultat répertorie ce qui a été envoyé sous la forme `settings_sent`, et désigne sous `warnings` tout index que les bandes de la caméra ne peuvent pas générer.
- **Le traitement s'effectue selon le principe « démarrage puis interrogation ».** `start_processing` renvoie une réponse dès le début de l'exécution, qui prend ensuite de quelques minutes à plusieurs heures, et l’assistant enchaîne avec `get_processing_status`. `state` correspond à `idle` (aucun traitement en cours), `starting` (le backend n’a pas encore signalé l’exécution, pendant 20 s maximum), `processing` ou `finished`, avec le [résumé](#post-run-summary--hints) : ce qu’elle a exporté, ainsi que des indications lorsqu’elle a exporté moins que prévu (aucune image, aucune cible d’étalonnage trouvée). Les produits sont enregistrés dans le dossier du projet, comme depuis l’application. `start_processing` est refusé lorsqu’une exécution est en cours, et un backend qui cesse de répondre est signalé comme une erreur, et non comme une exécution terminée.
- **Ce que voit l’assistant.** Les avertissements du SDK apparaissent dans le résultat de l’outil sous `warnings`, et les échecs (déconnexion, plan, chemin d’accès incorrect) sous forme de texte d’erreur qu’il peut lire et vous transmettre. Une liste comportant plus de 20 éléments est tronquée aux 20 premiers, sa longueur totale étant indiquée sous la référence `<key>_total`. Les chemins d’accès doivent être absolus.
- **Les appareils sont partagés.** Les caméras, les matrices et les capteurs de lumière se trouvent dans le backend ; ainsi, l’assistant peut en piloter un via l’application Chloros, et les modifications apportées s’affichent dans l’application. `disconnect_device` affiche ce à quoi le serveur est connecté ; un périphérique auquel un autre client est connecté (l’application, `chloros-cli`, un script, un autre `chloros-mcp`) n’est libéré qu’avec `force: true`, qui correspond au `force_disconnect` du SDK, et le refus indique le nom de celui qui l’a connecté lorsque le backend le connaît ; un appareil que le backend ne gère pas du tout est signalé comme non connecté (`released: false`, `connected: false`, rien à signaler). Un enregistrement ou un flux exécuté par le backend génère sa vue `get_job_status` avec `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (groupes d’images pour un flux multicaméra) — ainsi qu’une ligne `summary` (« 220 images écrites, 55 perdues, 0 échec : … ») pendant son exécution, puis, une fois terminé, ses chiffres finaux ; la liste fournie par un autre client (`other_clients_jobs`) contient les mêmes informations (`list_cameras()`, `list_arrays()` et `list_daq_sensors()` indiquent respectivement les`connected_by` : `client`, `label`, `name`, `at`). Chaque `chloros-mcp` s’identifie auprès du backend sous le nom de `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, envoyé sous le nom de `X-Chloros-Client-Label` ; tout script SDK peut le définir). L'arrêt de l'enregistrement du capteur de lumière d'un autre client nécessite également `force`.
- **Une tâche de longue durée est considérée comme un travail.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, et toute capture, résolution ou indexation demandée (`background`) renvoie immédiatement un `job_id` ; `get_job_status` suit et `stop_job` met fin . Une exécution se poursuit jusqu’à ce qu’elle soit arrêtée, sauf si vous lui transmettez `max_captures`, `duration_s` (ou, pour une rafale, `max_frames`); `min_free_gb` refuse de démarrer lorsque le disque dispose de trop peu d’espace libre. Aucune limite n’est appliquée à moins que vous n’en définissiez une. Un enregistrement ou un flux exécuté par le backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) conserve l’identifiant de tâche du backend, de sorte qu’un autre `chloros-mcp` — lancé ultérieurement — le retrouve grâce à cet identifiant : `get_job_status` le suit (et répertorie des tâches telles que `other_clients_jobs`, à partir de `GET /api/camera/cli-jobs`, chacune avec `started_by`), et `stop_job` ne l’arrête qu’avec `force: true`. Tout autre travail (qu'il s'agisse d'une capture, d'un enregistrement en tableau ou en rafale, ou d'une capture en arrière-plan) s'exécute dans le serveur `chloros-mcp` qui l'a lancé, et seul ce serveur en a connaissance : un autre serveur qui l’a demandé l’indique. Un processus arrêté lit `stopping` pendant qu’il vide sa file d’attente, puis `finished` avec ses décomptes finaux, pendant 10 minutes et à partir de n’importe quel `chloros-mcp` : celui qui l’a lancé perçoit de la même manière l’arrêt forcé d’un autre. `get_job_status` n’attend jamais un arrêt : pendant un arrêt (le sien `stop_job`, celui d’un autre clientou l’arrêt qui libère les caméras d’une tâche qui s’est terminée d’elle-même), il répond immédiatement avec `stopping` et les comptes en temps réel, et `stop_job` d’une tâche en cours d’arrêt l’indique. Une tâche terminée en explique la raison dans `stop_reason` : sa propre raison (`no_frames`, `disk_full`, `member_missing`, ou, pour une exécution continue en série, `count`, `duration`, `array_gone`), `limit reached` uniquement lorsque ses compteurs indiquent son `count` (ou le `max_frames` d’une rafale, celui d’un enregistrement`duration_s`) est atteint, `error`, `stopped` (avec `stopped_by`) lorsqu'un client l'a arrêté, sinon `finished`.
- **Où sont enregistrées les captures.** Dans le dossier `captures` du projet ouvert, où l’application les enregistre, sauf si `output_dir` est spécifié.
- **Images.** `get_camera_preview` et `grab_frames` renvoient des images que l’assistant peut consulter, en plus de leur format JSON. Les images `grab_frames` sont ajustées en termes de contraste pour faciliter la visualisation ; leurs statistiques correspondent aux valeurs réelles.
- **Lorsque le serveur se ferme**, les tâches qu’il a lancées s’arrêtent et tout enregistrement par capteur de lumière qu’il a démarré est interrompu. Si le serveur a lancé le backend, il déconnecte également les périphériques auxquels il était connecté avant d’arrêter le backend ; dans le backend de l’application, ceux-ci restent connectés.
- **Protocole.** stdio uniquement (pas de point de terminaison HTTP), outils uniquement (pas de ressources ni d’invites), résultats sous forme de texte et d’images, versions du protocole MCP du 05/11/2024 au 25/11/2025.**Ce qui n’est pas inclus.** La connexion ou la déconnexion (le compte est le vôtre ; le serveur ne peut pas se connecter), et l’arrêt d’un backend que le serveur n’a pas démarré. Tout le reste de ce que ce SDK effectue en matière de traitement et de matériel dispose d’un outil dédié. (L’approvisionnement d’un DAQ-E — son nom, son jeton d’authentification, mot de passe OTA, exigence de signature, réinitialisation d’usine — ainsi que les serveurs DAQ directs relèvent exclusivement de `chloros-cli`, également dans le SDK : un nom ou un jeton erroné bloque toute installation de Chloros sur l’appareil.)

---

## Démarrage automatique du backend

Les points d’entrée smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` et `discover_daq_sensors` — sont des clients HTTP légers qui partent du principe qu’un backend est à l’écoute sur `127.0.0.1:5000` (l’URL par défaut de l’interface Smart-Connect). Lorsque l’interface graphique ou la CLI est déjà en cours d’exécution, un serveur est déjà en service. À partir d’un simple script, il se peut qu’il n’y en ait pas — ces fonctions **lancent donc automatiquement le binaire du backend fourni** (en mode sans fenêtre, de la même manière que le fait `ChlorosLocal`) avant leur premier appel, puis attendent jusqu’à 60 s qu’il se lance. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` et `analyze_array_network()` n’en lancent jamais.

Règles :

- **Seule une URL locale est jamais créée.** Un `backend_url` pointant vers `localhost` / `127.0.0.1` / `[::1]` est éligible ; tout autre hôte est considéré comme appartenant à quelqu’un d’autre et n’est jamais lancé.
- **Un backend qui se ferme avant d’avoir répondu met fin à l’attente.** L’appel renvoie immédiatement le code de sortie, la commande qu’il a exécutée et les dernières lignes affichées par le backend, au lieu d’attendre les 60 s. Le code de sortie 98 constitue l’exception : un autre backend détient déjà le port ou le fichier de verrouillage (celui de l’application Chloros, du service backend ou d’un autre script encore en cours de démarrage), de sorte que l’appel continue d’attendre que celui-ci réponde. `ChlorosLocal` se comporte de la même manière.
- **Le backend reste en cours d’exécution pour être réutilisé** (comme dans l’CLI) — il n’y a pas d’arrêt implicite lorsque votre script se termine. La réexécution du script réutilise le backend actif.
- **Désactivez cette fonctionnalité avec `auto_start_backend=False`** sur n’importe lequel de ces appels (par exemple, lorsque vous pointez vers un backend distant ou que vous gérez vous-même le cycle de vie du backend).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Si le binaire fourni ne peut être localisé ou lancé, l’appel HTTP suivant génère une exception `ChlorosConnectError` exploitable et **adaptée à la plateforme**, plutôt qu’une simple trace de refus de connexion — sous Windows, elle vous redirige vers l’application de bureau ou une commande `chloros-cli` ; sous Linux (sans interface graphique), elle vous redirige vers une commande `chloros-cli` ou vers le fichier `.deb`.

---

## Environnement et en-têtes

Le SDK marque chaque appel HTTP vers le backend avec `X-Chloros-Client: sdk`. Le backend applique les règles de licence du SDK/CLI (connexion **et** abonnement Chloros+ payant requis) plutôt que le parcours gratuit de l’interface graphique. Ce paramètre est défini automatiquement lors de l’importation — vous n’avez rien à faire.

L’en-tête est ajouté à chaque appel `requests` / `urllib` dont l’URL contient `localhost` ou `127.0.0.1`, sur n’importe quel port — y compris vos propres services locaux — et remplace une valeur que vous avez définie vous-même. Les appels vers d’autres hôtes ne sont pas modifiés.

Pour remplacer l’URL du backend, transmettez `backend_url=` (`ChlorosLocal` accepte `backend_url=` ou `api_url=` ; `process_folder()` et `process_lattice_capture()` n'acceptent que `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un backend installé n'écoute que sur la boucle de retour, de sorte qu'un `backend_url` du réseau local ne peut pas l'atteindre ; voir [Mode backend distant](#remote-backend-mode-pip-only-host-via-tunnel) pour le modèle de tunnel.)

---

## Gestion des versions et compatibilité

- La version du SDK est indiquée par `chloros_sdk.__version__`.
- Le SDK lie le comportement aux broches de la version du backend fournie. L'association d'un SDK plus ancien avec un backend plus récent fonctionne généralement (points de terminaison compatibles en aval), mais l'association d'un SDK plus récent avec un backend plus ancien peut entraîner des erreurs `404` sur les nouveaux points de terminaison — mettez à jour l'application de bureau en conséquence.
- L’interface Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) et le point de terminaison d’analyse réseau renvoient des schémas JSON stables ; les nouveaux champs s’ajoutent aux existants.

---

## Conseils de dépannage

- **`ChlorosAuthenticationError: Login required`** (provenant de `ChlorosLocal` ; les appels Smart-Connect génèrent l’erreur `ChlorosConnectError: Login required. …` pour la même raison) → Exécutez `chloros-cli login EMAIL PASSWORD` une fois sur cette machine, ou connectez-vous via l'application de bureau Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (ou `Failed to create project: …` issu de `create_project()`) → Les chemins d’accès longs de Windows sont désactivés sur cette machine (paramètre par défaut de Windows), aucun chemin d’accès ne peut donc dépasser 260 caractères, et les produits se trouvent jusqu’à environ 130 caractères en dessous du dossier de projet. Aucun traitement n’a été effectué. Le message indique le dossier de projet le plus long compatible : utilisez-en un plus court, ou activez les chemins d’accès longs de Windows et redémarrez Chloros. Le programme d’installation Windows propose cette option sous le nom « Activer les chemins d’accès longs Windows », cochée par défaut ; sinon, utilisez la stratégie de groupe : *Configuration de l’ordinateur > Modèles d’administration > Système > Système de fichiers > Activer les chemins d’accès longs Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → La connexion intelligente (smart-connect) déclenche le démarrage automatique d’un backend local ; ce message n’apparaît donc que lorsque le binaire fourni est introuvable ou ne peut être lancé (par exemple, un hôte pipsans package de bureau). Le message dépend de la plateforme : sous Windows, ouvrez l’application de bureau ou exécutez n’importe quelle commande `chloros-cli` ; sous Linux, exécutez une commande `chloros-cli` (il n’existe pas d’interface graphique) ou installez `.deb`. Pour un backend distant, passez `backend_url=` (et `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** lors de l’importation → Échec du chargement de `lattice_sdk` : le runtime de l’SDK Arena, ou OpenCV (`cv2`), ne peut pas être importé dans cette version d’Python. La surface pilotée par le backend (`connect_*`, projetée via un backend, `ChlorosLocal`) fonctionne toujours.
- **L'Array se connecte en utilisant l'intégralité du capteur mais avec une faible fréquence d'images**→ Conformément à la conception : le mode « auto » conserve l'intégralité du capteur et laisse la fréquence d'images chuter jusqu'à la capacité de la liaison. Passez `binning=2` / `4` ou un `width` / `height` plus petit pour obtenir davantage d’images par seconde, ou activez les trames jumbo / mettez à niveau la liaison. Le binning et la ROI ne permettent**pas** de pallier la sursouscription agrégée (`oversubscribed: true`, champs fps à 0) : le nombre trop élevé de caméras pour la liaison ne peut pas être résolu par le binning ou le ROI — réduisez le nombre de caméras, activez les trames jumbo ou passez à une carte réseau plus rapide (voir [Sursouscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` signale que l'anneau de réception de la carte réseau est très petit (~0,26 Mo) / les portes de connexion affichent « FRAMES WILL DROP »** → L'anneau de réception de la carte réseau hôte est à sa valeur par défaut (souvent réinitialisé à 32 après une mise à jour du pilote de la carte réseau). Sur une carte Realtek USB 10 GbE, définissez `ReceiveBufferLen=256` et `PendingReceives=64` (avec des droits élevés), puis redémarrez le backend afin qu’il relise le ring. Procédure complète : [Référence CLI → Configuration et réglage de la carte réseau hôte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **L'hôte se bloque au redémarrage/à l'arrêt, suivi d'erreurs WMI `Invalid class` / la carte réseau ne s'active pas** → Pilote USB 10 GbE obsolète à l’origine de l’erreur `DRIVER_POWER_STATE_FAILURE` (écran bleu `0x9F`). Mettez à jour le pilote de la carte réseau vers une version récente (≥ 2026) et réappliquez les paramètres du ring de réception. Voir [Référence CLI → Configuration et réglage de la carte réseau hôte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Échec de la capture de la matrice avec le code HTTP503 « Aucun nouveau groupe de trames synchronisé »** → La matrice n’a publié aucun nouveau groupe de trames pendant le délai d’attente. Vérifiez `arr.status()` : `stalled: True` (ou `streaming: False`) avec `actual_fps` : une valeur de 0 signifie que la boucle de capture a cessé de publier — déconnectez puis reconnectez le réseau. Si `stalled` est False et que `actual_fps` est supérieur à 0, réessayez. Juste après `connect_array()`, une capture attend déjà depuis 20 s pour le premier groupe.
- **`status()` affiche `link_speed_mbps: 4294`** → Windows n’a pas pu lire la vitesse de la carte ; cela est courant avec les cartes Ethernet USB et ne constitue pas un dysfonctionnement. Voir [Débits de trames attendus](#expected-frame-rates).
- **`update_firmware()` échoue avec `device never connected back on TCP <port>`** → Un pare-feu a bloqué la connexion du DAQ-E vers cet ordinateur. Le programme d'installation Windows ajoute la règle (voir [Paramètres du périphérique DAQ-E](#daq-e-device-settings)) ; avec un autre pare-feu, autorisez le trafic TCP entrant vers le backend depuis le sous-réseau du capteur.
- **Réflectance refusée** → Un DAQ actif doit être associé à la caméra (ou au réseau de caméras) pour obtenir une réflectance à échelle absolue. Effectuez l'association via l'interface graphique ou utilisez `processing="radiance"` (W/m²/sr/nm), qui ne nécessite pas de capteur associé.
- **La capture avec `smart=True` prend plus de temps que prévu** → La convergence AE dépend de la dynamique de la scène. Avec `ArrayHandle.capture_smart()`, élargissez `exposure_tolerance_pct`, réduisez la durée de `stability_window_s` ou diminuez la valeur de `settle_timeout_s` pour obtenir un déclenchement plus rapide et moins stabilisé.

---

## Voir aussi

- [Référence CLI](cli-reference.md) — chaque sous-commande CLI correspond à un appel SDK.
- [Capteurs de lumière DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) dans le manuel Chloros — les capteurs, leurs plages de mesure et leurs plages calibrées, l'enregistrement, la mise en réseau DAQ-E.
- [Caméras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) dans le manuel Chloros — connexion, paramètres des caméras, modes de capture, matrices.
- Documentation en ligne : `https://mapir.gitbook.io/chloros/api-python-sdk`
