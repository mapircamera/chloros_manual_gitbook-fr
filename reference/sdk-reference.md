# Chloros Python SDK Référence

**Version :**

1.2.0**Date de création :**29/07/2026 19h19 ·**Date de révision :** 30/08/2026**Package :** `chloros-sdk` (PyPI)**Public visé :** Optimisé pour une utilisation par les grands modèles de langage (LLM) ; lisible par l&#x27;humain.**Portée :** Toutes les classes, fonctions et aides publiques exposées par `import chloros_sdk`, avec des exemples copiables-collables couvrant le traitement d’images, le contrôle d’une seule caméra, les tableaux synchronisés, les capteurs DAQ et l’automatisation de projets.

Si vous souhaitez uniquement consulter les points clés, rendez-vous directement à :
- [Installation et démarrage rapide](#installation)
- [Smart-Connect pour les réseaux de caméras LATTICE](#smart-connect-for-lattice-cameras)
- [Sessions de capteurs DAQ](#daq-sensor-sessions)
- [Automatisation de projet](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## L’architecture en 60 secondes

L’SDK est une fine couche d’Pythons qui recouvre le backend Chloros (le même serveur Flask que celui utilisé par l’interface graphique de bureau et CLI). Pour l’automatisation, vous importez `chloros_sdk` et appelez des méthodes de haut niveau ; en arrière-plan, chaque appel se transforme en requête HTTP adressée au backend local sur le port 5000 — `http://127.0.0.1:5000/api/...` (et non `localhost`, qui est d’abord résolu en `::1` sur Windows et coûte environ 2 s par requête vers un backend IPv4 uniquement). Le backend gère le parc matériel — caméras, capteurs DAQ, profils d’alignement, tampons d’images — ce qui permet aux scripts SDK de coexister avec l’interface graphique sans se disputer les ports série ni la bande passante des cartes réseau.

Vous utiliserez trois interfaces :

1. **`ChlorosLocal` + fonctions libres** (`process_folder`, `process_lattice_capture`) — Pipeline de traitement d’images. Traitez un dossier entier (étalonnage / débayérisation / exportation d’index) à partir d’un seul appel à la fonction `Python`.
2. **Maniestures Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Ouvrez une session backend persistante pour le matériel en temps réel. Même flux « smart-prep » que l’interface graphique : sonde réseau, sélection automatique du niveau, PTP, initialisation AE, configuration du déclencheur GPIO.
3. **`ChlorosProject` / `open_project`** — Chargement d’un projet enregistré (dossier contenant `cameras.json` + `sensors.json` + `project.json`), connexion simultanée de tous les éléments, et effectuez des captures à l’aide de descripteurs nommés.

Les interfaces 1 et 2 **lancent automatiquement un backend local** s’il n’y en a pas déjà un à l’écoute (le même binaire fourni que celui lancé par l’interface graphique ou CLI) — ainsi, un simple script fonctionne depuis un shell vierge sans que vous ayez à démarrer un backend au préalable. Passez `auto_start_backend=False` pour désactiver cette fonctionnalité (par exemple lorsque vous pointez vers un backend distant, qui n’est jamais lancé). Voir [Démarrage automatique du backend](#backend-auto-start). Surface 3 se comporte différemment : `open_project()` n’accepte aucun paramètre `auto_start_backend`, et `connect_all()` ne lance jamais de backend — il interroge `http://127.0.0.1:5000` une fois et, si rien ne répond, revient silencieusement au contrôle direct (sans backend) du périphérique `lattice_sdk`. Seuls `proj.process()` et `stream(..., overlays=True)` créent de manière différée un `ChlorosLocal()` (qui se lance automatiquement).

Les trois sont en mode auth: lancez `chloros-cli login` une fois sur la machine, ou connectez-vous via l’interface graphique du bureau. Les appels à SDK sans session valide génèrent l’erreur `ChlorosAuthenticationError`.

Configuration requise :
- Python 3.7+ (tel que déclaré par le paquet ; développé/testé sur la version 3.10)
- Chloros Desktop installé localement (le binaire du backend est fourni dans le programme d’installation)
- Connexion active à Chloros+. Le seuil d’accès à SDK / CLI est le niveau **Copper**ou supérieur (Copper / Bronze / Silver / Gold) ; le niveau gratuit**Iron**ne donne pas accès à SDK / CLI. Cette restriction est appliquée**côté serveur** : toute requête comportant le drapeau SDK / CLI doit être associée à la fois à une session active et à un abonnement payant, sinon le backend renvoie `403` avec `error_code: PLAN_UPGRADE_REQUIRED` (affiché sous la forme `ChlorosLicenseError` par `ChlorosLocal`, et sous la forme `ChlorosConnectError` par les aides `connect_*`). Un appelant déconnecté obtient à la place `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — ces deux codes sont distincts car la réexécution de `chloros-cli login`X corrige le premier mais ne peut pas corriger le second.
- L’utilisation hors ligne est prise en charge pendant la période de grâce du forfait : le niveau d’accès est lu à partir du(5 min) ou dans le cache des licences signées et liées à la machine (30 jours pour les forfaits mensuels, jusqu’à l’expiration de l’abonnement pour les forfaits annuels). À l’expiration de ce délai de grâce, le forfait passe en version gratuite et l’accès à SDK / CLI est interrompu jusqu’à ce que la machine puisse se connecter au serveur au moins une fois. `chloros-cli status` (`GET /api/license-status`) reste accessible dans le cadre du niveau gratuit ; la raison est donc évidente : il s’agit de la seule route SDK / CLI exemptée de la restriction de niveau.
- Windows 10/11 64 bits, **Ubuntu 22.04 LTS ou version plus récente**, ou Jetson (JetPack 6). Ubuntu 20.04 n’est**pas** pris en charge : les dépendances de `.deb` découlent des bibliothèques auxquelles le backend est lié, y compris `libc6 (>= 2.34)`, et Focal est livré avec glibc 2.31.

---

## Installation

L’Python SDK est une fine couche d’Pythons superposée au backend Chloros. Pour tout ce qui va au-delà de quelques workflows de simple acquisition de données (DAQ), vous devez **installer localement le paquet de bureau Chloros** (programme d’installation Windows ou Linux `.deb`) — c’qui fournit le binaire du backend, le runtime Arena SDK pour les caméras LATTICE, ainsi que les ensembles d’étalonnage.

Derniers téléchargements : [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Étape 1 — Installer le package de la plateforme Chloros

#### Windows (.exe)

1. Téléchargez `Chloros-Setup-x.y.z.exe` depuis la page de téléchargement.
2. Lancez le programme d’installation et suivez les instructions de l’assistant. Le chemin d’installation par défaut est `C:\Program Files\MAPIR\Chloros\`.
3. Lancez Chloros au moins une fois et connectez-vous avec votre compte Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Étape 2 — Installez le Python SDK

**Le programme d&#x27;installation de Chloros fournit un module « wheel » SDK correspondant.** Chaque programme d’installation de Windows et chaque fichier .deb de Linux installe sur le disque un fichier `chloros_sdk-X.Y.Z-py3-none-any.whl` qui correspond exactement à la version de l’interface graphique / CLI / du backend. Vous n’avez pas besoin de consulter PyPI pour rester à jour.

#### Windows

Le programme d’installation exécute automatiquement`pip install` sur le fichier wheel fourni en utilisant l’Python de votre système (le lanceur `py.exe` est privilégié, mais le système se rabat sur `python -m pip` si nécessaire). Aucune intervention n’est requise : `import chloros_sdk` fonctionne dans votre environnement Python une fois l’installation réussie. Si aucune version d’Python n’est présente sur la machine, le programme d’installation ignore cette étape en silence et l’interface graphique ainsi que CLI continuent de fonctionner.

#### Linux (.deb)

Le fichier .deb place le « wheel » à l’emplacement `/usr/lib/chloros/sdk/`. Le fichier `postinst` affiche la commande exacte — les distributions conformes à la norme PEP 668 refusent par défaut les écritures globales via pip, nous ne procédons donc pas à une installation automatique :

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pour les déploiements Jetson en mode « air-gapped », le processus est entièrement hors ligne — le paquet wheel se trouve déjà sur le disque.

#### PyPI public

Pour les hôtes utilisant uniquement pip (aucun paquet de bureau « Chloros » installé ; workflows avec backend distant ou DAQ uniquement) :

```bash
pip install chloros-sdk
```

PyPI est mis à jour lors des builds de l’installateur en version de publication, de sorte que le fichier wheel publié correspond à la dernière version stable. Les builds de développement (par exemple `1.1.4.dev1`) ne sont fournis que via le fichier wheel intégré à l’installateur.

#### Vérifier

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Abonnement à Chloros+ requis.** Tous les appels à SDK nécessitent un identifiant actif sur Chloros+. Exécutez `chloros-cli login user@example.com 'YourPassword'` une fois par machine ; les identifiants sont mis en cache dans `~/.chloros/`.

### Ai-je besoin du pack Desktop ?

Le pack pip seul **ne suffit pas** pour la plupart des workflows. Voici ce dont chaque surface d’SDKs a besoin :

| Surface d’SDK | A-t-elle besoin du pack Desktop ? | Pourquoi |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Oui** | Lance automatiquement le binaire du backend sur `/usr/lib/chloros/chloros-backend` (Linux) ou `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Oui**(local)**/ Non**(à distance) | Clients en mode « pure HTTP » via le backend. Backend local → package de bureau requis. Backend distant → `backend_url=`**via un tunnel** (voir Mode « Remote-Backend » — les backends fournis se lient uniquement en boucle locale). |
| `ChlorosProject` / `open_project` | **Oui** | Accès aux projets enregistrés via le backend. |
| Classes LATTICE directes (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Oui** | Nécessite le runtime natif Arena SDK fourni dans le package de bureau. Sinon, `CAMERA_AVAILABLE` correspond à `False` lors de l&#x27;importation. |
| Classes DAQ directes (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Non** | Interfaçage pur (Python) via pyserial/bleak/zeroconf. Un environnement utilisant uniquement pip peut piloter les DAQ de bout en bout. |

### Mode « Remote-Backend » (hôte pip uniquement, via un tunnel)

> **Le backend fourni n’est pas accessible via le réseau local.** Les versions
> de production ne prennent en charge que le mode loopback (les deux familles de loopback) et refusent catégoriquement le
> seul mode non-loopback (`CHLOROS_CLOUD_MODE`), donc
> `backend_url="http://<lan-ip>:5000"` **ne peut pas fonctionner avec un
> Chloros** installé — ce schéma n’a jamais fonctionné qu’avec un backend source/dev
> . Pour piloter un backend sur une autre machine, redirigez son port de bouclage
> et pointez l’SDKe vers le tunnel :

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Les hôtes sans interface graphique / CI / robotiques peuvent conserver une machine avec l’installation complète du bureau comme « serveur Chloros » et utiliser `pip install chloros-sdk` partout ailleurs — mais le transport entre eux se fait via le tunnel configuré par l’utilisateur ci-dessus, et non via une conURL LAN directe.

> **Limitation connue — `ChlorosLocal` ne prend pas uniquement en charge pip.** `ChlorosLocal(backend_url=BACKEND)` résout actuellement un binaire de backend local dans son constructeur *avant* d’interroger l’URL et génère l’erreur `ChlorosBackendError` (« Chloros backend not found… ») lorsqu’aucun paquet de bureau n’est installé — même si un backend distant est accessible. Seule l’interface « smart-connect » ci-dessus (`connect_camera` / `connect_array` / `connect_daq_sensor`, ainsi que `analyze_array_network` et les aides `list_*` / `discover_*`) fonctionne à partir d’un hôte utilisant uniquement pip.

### Flux de travail DAQ uniquement (hôte pip uniquement)

Si vous n&#x27;avez besoin que de capteurs DAQ et que vous n&#x27;utilisez pas les caméras LATTICE ni le traitement d&#x27;images, le paquet pip est autonome :

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Aucun backend, aucun fichier .deb, aucune connexion via Chloros+ n&#x27;est requise pour les opérations DAQ en accès direct au matériel.

---

## Démarrage rapide

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
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

## Index de l&#x27;APIe de premier niveau

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
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Traitement d’images — `ChlorosLocal`

Classe principale du pipeline. Lance le backend lors de la première utilisation, crée et configure les projets, surveille la progression et renvoie des résumés après exécution.

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
| `import_images(folder_path, recursive=False)` | Importe des images RAW/TIF/JPG/DNG **et `.daq` des enregistrements de**. Renvoie `count` (images) et `scan_count` (enregistrements). Affiche un avertissement uniquement si le dossier ne contient ni l’un ni l’autre. |
| `export_light_sensor(daq=True, csv=True)` | Écrit les données calibrées `.daq` + `.csv` pour chaque enregistrement du capteur de lumière du projet, dans le fichier `<project>/Light Sensor/`. Voir [Enregistrements du capteur de lumière](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Définit les paramètres de traitement. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Lancer le pipeline. Renvoie `{"status": "complete", "async": False}`, ainsi qu&#x27;une clé `summary` lorsque le backend en fournit une — voir [Résumé et conseils après exécution](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Inspecter l’état du backend. |
| `logout()` | Effacer les identifiants mis en cache. |
| `shutdown_backend()` | Arrêter le backend (si SDK -a été lancé). |
| `discover_cameras()` | Détecter les caméras LATTICE **via le backend de cette instance** (`/api/camera/discover`). Renvoie une liste de dictionnaires (`serial`, `model`, `ip`, …) — de même structure que celle affichée par l’interface graphique/CLI. Liste vide si aucune caméra n’est trouvée ou si le backend est inaccessible. |
| `camera_capture(output_dir, format="tiff", **settings)` | Capture d’une seule image**via le backend**(démarrage automatique par ce descripteur) afin qu’elle bénéficie de la même préparation que l’interface graphique/CLI (12 bits par défaut, réutilisation du pool, métadonnées de calibrage intégrées). Définir la cible avec `serial=` ou `device_index=` ; transmettre `exposure`/`gain`/`pixel_format`/`preset` sous la forme `**settings`. Renvoie le dictionnaire de métadonnées hérité (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Superposition de rendement- images de prévisualisation composites provenant d’une caméra mutualisée — client MJPEG léger via la route `/api/camera/<serial>/stream-annotated` du backend (zèbre / grille / réticule / histogramme / peaking / point tracé côté serveur). `decode=True` renvoie des tableaux BGR ; `False` renvoie des octets bruts JPEG. Également accessible par projet sous le nom `ChlorosProject.stream(overlays=True)`. |

À utiliser comme gestionnaire de contexte pour garantir le nettoyage :

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Enregistrements de capteurs de lumière — calibrés `.daq` + `.csv`

Un DAQ-U / DAQ-M / DAQ-E peut être enregistré **sans** son ensemble d’étalonnage. C’est
ce que font par défaut les enregistreurs publics [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) : ils enregistrent les valeurs brutes des capteurs et horodatent le
fichier afin que Chloros récupère l&#x27;étalonnage d&#x27;usine de ce capteur **par numéro de série** — d’abord dans le cache local,
puis dans le cloud MAPIR — et l’applique lors de l’importation.

Chloros réécrit le résultat sous la forme de deux produits par enregistrement, sous
`<project>/Light Sensor/` :

| Produit | Description |
| --- | --- |
| `<name>_calibrated.daq` | L’archive rétraitable — même schéma qu’un enregistrement en direct, déclarant désormais le bundle qui l’a produite. Sa réimportation **ne**provoque**pas** un deuxième étalonnage. |
| `<name>_calibrated.csv` | Irradiance spectrale en W/m²/nm sur la grille de longueurs d’onde propre au capteur, une ligne par lecture, plus des colonnes photométriques (puissance totale, lux photopiques/scotopiques, PPFD et sa décomposition en bleu/vert/rouge, longueur d’onde de crête). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Capteurs sans ensemble de données uniquement (DAQ-A).** Compteurs spectraux bruts du capteur — *pas* l’irradiance. Voir ci-dessous. |

`process()` effectue cette exportation dans le cadre de l’une de ses étapes. Il ne nécessite **pas** d’images :
un capteur de lumière utilisé seul constitue un flux de travail à part entière, et un tel projet ne comporte aucune
image par définition.

**Les enregistrements DAQ-A s’exportent sous forme de comptes bruts.** La gamme DAQ-A est antérieure au
et ne dispose d’aucun bundle à récupérer — elle est plutôt étalonnée sur le terrain par rapport à une
cible de réflectance, ce qui explique pourquoi elle n’en a jamais eu besoin. Ces enregistrements s’exportent
sous une racine `_raw` plutôt que `_calibrated` : un nom de fichier différent plutôt qu’un indicateur
à l’intérieur du fichier, car l’information doit rester intacte lors de l’envoi par e-mail sous forme de nom nu. L’
en-tête `.csv` indique `raw spectral sensor counts (NOT irradiance)` et précise que les
valeurs sont comparables **au sein** du fichier — ce qui correspond exactement à l’usage que leur fait l’étalonnage basé sur la cible
— et non entre différents capteurs. Les colonnes photométriques dépendantes de la puissance (puissance totale,
lux photopiques/scotopiques, PPFD) renvoient **NULL** au lieu d’être calculées à partir des comptages.

Un DAQ-U / DAQ-M / DAQ-E dont le bundle n’a tout simplement pas pu être récupéré est toujours **ignoré**,
et non écrit au format brut : dans ce cas, le bundle existe et « se reconnecter et retraiter » est un conseil avisé.

Les enregistrements hérités **v1.01 / v1.02** (un DAQ-A-SD les écrit) ne comportent pas d’époque par lecture,
seulement l’heure d’écriture du fichier. Le module de correspondance image↔flux descendant les refuse toujours — faire correspondre une
trame à une heure d’écriture serait une erreur invisible — mais l’exportateur les lit, et le
fichier CSV affiche `clock=daq_created_on`, ce qui permet au produit d’indiquer sur quelle horloge il se trouve.

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

Un enregistrement dont le jeu de données d’étalonnage ne peut pas être récupéré (hors ligne, ou capteur sans
données d’étalonnage enregistrées) est signalé sous `skipped` **avec la raison**. Il n’est jamais
enregistré en tant que fichier « étalonné » contenant des comptages bruts — connectez-vous à Internet et
relancez l’opération, et l’exportation s’achèvera.

### Rappels de progression

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

Une fois l’opération terminée, `process()` récupère `GET /api/processing-summary` et joint le corps en tant que `result["summary"]`. La récupération s’effectue au mieux et ne bloque jamais un retour réussi — si le résumé n’est pas disponible, `process()` revient à la structure simple `{"status": "complete", "async": False}`. Chaque entrée de `summary["hints"]` — des phrases complètes indiquant la solution proposée, par exemple pourquoi une exécution n’a produit aucun résultat — est égalementémise sous la forme d’un `UserWarning` de type «Python » ; ainsi, les exécutions sans résultat permettent un autodiagnostic même si vous n’inspectez jamais le dictionnaire :

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
| `models` | Groupes de caméras dans la session. |
| `images_in_groups` | Images sources de ces groupes. |
| `targets_found` | Cibles de réflectance détectées. |
| `images_calibrated` | Images utilisées pour l&#x27;étalonnage de la série. |
| `exported_files` | **Fichiers de produits d&#x27;image générés par la série.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Enregistrements du capteur de lumière, comptabilisés séparément à dessein — ils proviennent d’une étape différente et existent pour des sessions ne comportant aucune image ; les inclure donnerait l’impression qu’une session exclusivement de collecte de données (DAQ) a exporté des images. |

À côté d’eux : `summary["output_dirs"]` (chaque répertoire dans lequel des données ont été écrites),
`summary["light_sensor_export"]`, `summary["stopped"]` (vrai lorsque l’utilisateur a interrompu l’
exécution, afin que les comptages partiels ne soient pas interprétés comme une exécution terminée ayant produit moins que prévu), et
`summary["groups"]` (la ventilation par groupe).

`exported_files` est enregistré par le pipeline **au fur et à mesure de l’écriture**, et non pas extrait par la suite à partir des
objets image du projet. Les stratégies parallèles et GPU créent leurs propres objets image
(dans des sous-processus de travail pour les chemins GPU) ; ainsi, l’ancienne analyse signalait
`0 file(s) written` pour chaque exécution de ce type, puis émettait l’indication « zéro exportation » — lors d’exécutions
où tout avait pourtant fonctionné. Si vous créez un script en fonction de ce nombre, une exécution parallèle réussie
renvoie désormais un compteur non nul.

Les sauts de capteur de lumière signalent la raison réellement établie par le lecteur pour chaque fichier — un
schéma illisible, un bundle manquant, une erreur d’écriture — **dédupliquée** ; ainsi, vingt fichiers
ignorés pour une même cause sont comptabilisés comme une seule cause plutôt que vingt répétitions de celle-ci.

> **`process()` ne se déclenche pas lorsqu’une exécution ne produit aucune image.** C’est le seul point sur lequel SDK et
> CLI diffèrent délibérément : `chloros-cli process` traite « des produits ont été demandés, aucun n’a été
> écrit » comme un échec et se termine avec un code de sortie non nul, tandis que l’SDK se termine normalement et signale la
> condition via `summary` / hints. Si votre pipeline doit s’arrêter lors d’une exécution vide, vérifiez-le
> vous-même — inspectez `summary` (ou comptez les fichiers dans le dossier du projet) plutôt que de vous fier à
> l’absence d’exception. Les causes habituelles sont un dossier d’entrée qui n’a pas été reconnu comme une
> capture et des produits ignorés car inapplicables aux caméras présentes (par exemple, la radiance provenant de caméras RGB -only
>).

### Fonctions pratiques

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
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

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Valeurs prises en charge

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
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

Le niveau d’exportation multispectral LATTICE (M3C/M3M) du pipeline `process` — `reflectance` (par défaut), `radiance`, `sensor-response` ou `all` (chaque mode applicable par image) — correspond au paramètre de traitement **«Sortie radiométrique»** du projet. `configure()` dispose d’un mot-clé dédié :

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

La solution de secours avancée — l&#x27;écriture de la clé `"Radiometric output"` du projet via `custom_settings` — fonctionne toujours, mais n&#x27;oubliez pas qu&#x27;elle remplace l&#x27;intégralité du bloc de paramètres (voir l&#x27;avertissement ci-dessous) :

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (valeur par défaut) divise la radiance de la caméra par le **flux descendant DAQ aligné sur l’horodatage**, déterminé automatiquement à partir d’un fichier `.daq` (DAQ-U/M/E)**ou d’un `.csv` natif DAQ-M**trouvé avec l’imagerie ; tout ensemble d’étalonnage par caméra ou par DAQ manquant localement est**récupéré automatiquement depuis AWS** lors de la première utilisation. L’CLI expose cela sous forme de boutons de sélection par type de produit sur `chloros-cli process` : `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **remplace** l&#x27;intégralité du bloc de paramètres calculés (il contourne, de par sa conception, les autres mots-clés et la validation de `configure()`). Lorsque vous l&#x27;utilisez, incluez toutes les clés `Project Settings` qui vous intéressent, comme dans l&#x27;exemple ci-dessus.

---

## Smart-Connect pour les caméras LATTICE

Sessions backend persistantes pour le matériel en direct. Mêmes points de terminaison que ceux utilisés par l’interface graphique, ce qui garantit un comportement identique sur SDK / CLI / l’interface graphique.

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

#### `CameraSession` Méthodes

| Méthode | Description |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lit les nœuds GenICam ; renvoie `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Écriture de nœuds par nom convivial (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Capture une **seule** image. Renvoie une liste à un élément contenant des dictionnaires de métadonnées d’image. (La capture en rafale/multi-images a été supprimée — appelez `capture()` en boucle si vous avez besoin d’une série.) |
| `disconnect()` | Libération du pool. Aucuneopération si nous étions connectés à une session déjà ouverte. |

Commandes d’exportation `capture()` (même modèle que le tableau + l’interface graphique) :

- `processing` / `levels` — `processing="all"` enregistre tous les types d’exportation applicables ; `levels=["raw","radiance"]` n’enregistre que ceux-ci (remplace `processing`). Omettez les deux pour utiliser la valeur par défaut du backend.
- `force_daq=True` — enregistre la lecture DAQ/DLS attribuée sous forme de fichier « sidecar » `.daq`, même lors d’une capture en format brut uniquement, afin que l’image puisse être retraité ultérieurement en réflectance/indice. Aucune opération si aucun DAQ n’est associé.

### Matrice synchronisée — `ArraySession` (Smart-Prep)

`connect_array` est **le point d’entrée recommandé** pour les configurations multi-caméras. Il exécute en arrière-plan l’intégralité du flux Smart-Prep via l’interface graphique :

1. **Analyse du réseau** (`/api/camera/array/recommend`) — détermine la taille de trame maximale pouvant s’adapter au niveau sim-emit sans perte de trames.
2. **Sélection automatique du niveau** — `sim-capture-sim-emit` si la liaison réseau le permet ; sinon, `sim-capture-ftd-stagger` ou `slip-emit-and-capture`.
3. **Réduction automatique**— réduit silencieusement la taille des trames / augmente le binning lorsque le câble ne peut pas supporter la résolution demandée.**Ce filet de sécurité ne couvre pas la sursouscription globale** : un nombre trop élevé de caméras pour le câble ne peut pas être corrigé en réduisant la taille des trames — voir [Sursouscription](#over-subscription-the-per-cam-floor).
4. **PTP activé** par défaut — lesentre les caméras sont comparables à la microseconde près.
5. **Sélection automatique du format de pixel par caméra** — caméras à RGB → `BayerRG8`, multispec → `BayerRG12`.
6. **Initialisation AE** — enregistrel&#x27;état AE actuel de chaque caméra afin que la connexion ne réinitialise pas l&#x27;exposition en cours de fonctionnement.
7. **Configuration du déclenchement GPIO** — `connect_array` active toutes les caméras (`TriggerMode=On`, `TriggerSource=Line2`) afin que l&#x27;impulsion de la caméra maître commande les caméras esclaves via le câble M8. Il s&#x27;agit d&#x27;une: une caméra unique ouverte avec `LatticeCamera` fonctionne en mode libre à la place.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Signature `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Valeurs `force_tier` :
- `"sim-capture-sim-emit"` — simultanéité réelle (toutes les caméras se déclenchent sur le même front d’horloge).
- `"sim-capture-ftd-stagger"` — décalage flexible dans le domaine temporel (les caméras émettent à des instants légèrement décalés afin que les paquets soient sérialisés sur le réseau).
- `"slip-emit-and-capture"` — capture séquentielle par came (pas de synchronisation temporelle ; seule option possible lorsqu’aucune taille de trame ne convient à la synchronisation).

`wire_ceiling_mbps` remplace le **budget de bande passante soutenu de l’hôte** en Mo/s — le
chiffre sur lequel repose toute l’allocation de la matrice. Laissez la valeur par défaut `None` pour utiliser la valeur détectée automatiquement
. Réduisez-la lorsque la matrice signale des trames corrompues par GVSP : la valeur automatique est dérivée
du débit de liaison annoncé par la carte réseau, qui surestime les adaptateurs USB, les voies PCIe étroites et
les structures partagées très sollicitées — et cette surestimation se traduit par des trames corrompues plutôt que par une
liaison visiblement lente. La valeur est enregistrée dans le bloc de capture de la matrice du projet, de sorte qu’une
réouverture ou une commande `connect_array` ultérieure la rétablit comme n’importe quel autre paramètre de la matrice.
Voir [État de santé de la matrice](#array-health--which-subsystem-is-losing-frames).

#### Sursouscription (seuil minimal par caméra)

Le rythme d’émission simulée alloue à chaque caméra une part du budget de bande passante sans collision, avec un seuil minimal de **8 Mo/s par caméra**(`per_cam_floor_bps`). Une fois que `N × floor` dépasse le plafond de sécurité anti-collision, le réseau**sursouscrit la bande passante**— le mode de défaillance est la perte de paquets GVSP, et non une fréquence d’images réduite — et il n’existe aucun remède au niveau de la taille des trames :**le regroupement et la zone d’intérêt (ROI) réduisent le nombre d’octets par trame, et non le nombre d’octets par seconde régulé**que la vérification agrégée compare. Plafonds pratiques en pleine résolution sur un hôte 1 GbE :**6 caméras à 1 500 MTU, dont 9 avec des trames jumbo** (`max_cams_collision_safe` dans la réponse d’analyse indique le seuil maximal pour votre liaison). Solutions : moins de caméras, des trames jumbo de bout en bout, ou une carte réseau plus rapide.

- Les réponses `analyze_array_network()` et `/api/camera/array/connect` contiennent les codes `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` et `per_cam_floor_bps`. Lorsque `oversubscribed` est vrai, la projection **remet à zéro les champs fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) plutôt que de signaler une fréquence trompeuse, lente mais fonctionnelle.
- `POST /api/camera/array/connect` accepte un paramètre de corps `pin_resolution` (**uniquement « HTTP » — et non un argument kwarg d’SDK**; `connect_array` ne l’expose pas). Le « pinning » supprime le filet de sécurité du « walk-down » de regroupement, de sorte qu’une connexion sursouscrite avec `pin_resolution` défini est**catégoriquement refusée** avec une erreur indiquant toutes les solutions possibles. Sans « pinning », la connexion se poursuit avec le « walk-down », mais avertit que la réduction ne peut pas effacer l’agrégat.
- Solution de secours en environnement de test : définissez `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` dans l’environnement du backend pour ramener le refus à un avertissement sonore — vous vous connectez quand même et acceptez la perte de paquets.

#### État de la matrice — quel sous-système perd des trames

`GET /api/camera/array/<array_id>/capability` contient un bloc `health` actif sur une
matrice connectée, réévalué sur une fenêtre glissante de **10 secondes**. Il répartit la perte de trames
en deux causes nécessitant des correctifs opposés, au lieu d’un taux « incomplet » unique qui
n’en identifie aucune :

| Champ | Signification | Sous-système concerné |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (par numéro de série) | L’image **est arrivée mais était structurellement incorrecte**— perte de paquets GVSP. |**Réseau** : bande passante, cadencement, anneau de réception de la carte réseau, MTU |
| `never_arrived_rate_pct` (par numéro de série) | La trame **n&#x27;est jamais arrivée**— la caméra ne s&#x27;est pas déclenchée, ou rien n&#x27;en est sorti. |**Déclenchement / synchronisation** : câble M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Taux le plus bas de chaque caméra. | — |
| `per_cam_rate_pct` | Taux combiné d’incomplétude par caméra (les deux causes réunies). | — |
| `stable_for_seconds` | Durée pendant laquelle chaque caméra est restée en dessous de 0,01 %. | — |

Parallèlement à `health`, ce même enregistrement indique la valeur sur laquelle repose l&#x27;ensemble de l&#x27;allocation :

| Champ | Signification |
| --- | --- |
| `wire_ceiling_mbps` | Budget de bande passante en vigueur pour l&#x27;hôte, en Mo/s. |
| `wire_ceiling_source` | Origine de ce chiffre, en toutes lettres — par exemple `USB-capped 200 MB/s (was theoretical 1062; …)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` lorsque `wire_ceiling_mbps=` l&#x27;a défini. |
| `nic_is_usb` | `true` pour une carte Ethernet USB. |

Il n&#x27;existe pas d&#x27;enveloppe «SDK» pour ce point de terminaison — lisez-le directement :

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Lecture :** une valeur `gvsp_corrupt_rate_pct` non nulle avec `never_arrived_rate_pct` à 0 signifie que
que le déclenchement et la synchronisation du câble sont parfaits et que 100 % des pertes se situent sur le chemin réseau — réduisez la valeur
`wire_ceiling_mbps` et reconnectez-vous. Le schéma inverse indique plutôt un problème au niveau du câble de synchronisation ou de la
ligne de déclenchement.

> **`target_fps` n’est pas le paramètre responsable des trames corrompues.** Le rythme GevSCPD est défini une seule fois lors de la
> connexion, donc réduire la fréquence de déclenchement modifie le rapport cyclique et non le
> débit de rafales à émission simultanée. Une réduction mesurée de 5 fois la demande n’a apporté aucune amélioration, tandis que
> l’abaissement du plafond de la liaison de 240 à 200 Mo/s a fait passer ce même équipement de 10,4 % de trames corrompues à
> 0,00 %.

> **La réduction automatique en cours de transmission n’est pas disponible sur le micrologiciel TRI032S.** Une matrice en cours d’exécution ne peut pas
> résoudre ce problème d’elle-même ; déconnectez puis reconnectez afin que le sélecteur de temps de connexion se réorganise en fonction
> du nouveau plafond.

Une **carte Ethernet USB est plafonnée à 200 Mo/s** par la sonde, quelles que soient ses
caractéristiques techniques : le tableau d’efficacité qui convertit un débit de liaison en un débit soutenu est
dérivé du PCIe, et une carte réseau USB annonce son débit de liaison Ethernet tout en étant limitée par le
bus USB et de son pilote. La limite est absolue, et non relative : une carte USB 1 GbE
atteint environ 80 Mo/s et n’est pas affectée.

#### Méthodes `ArraySession`

| Méthode | Description |
| --- | --- |
| `status(timeout=10.0)` | `{fps, ptp, frame_count, last_error, …}` en temps réel. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Un groupe de capture synchronisé. Renvoie un `CaptureResult` (liste de dictionnaires de trames + `.skipped`). Commandes d’exportation ci-dessous. |
| `capture(..., smart=True)` | **Capture intelligente** — attend que l’AE se stabilise sur toutes les caméras, puis se déclenche. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Capture la plus rapide : données brutes uniquement + la lecture DAQ attribuée (+ l&#x27;index combiné libre). Correspond au bouton « Capture la plus rapide » de l&#x27;interface graphique. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Capture unique / continue / par intervalle dans une boucle bornée. Renvoie `list[CaptureResult]`.**Nécessite `count` ouou `duration_s`** pour qu’elle se termine (l’SDKe ne dispose pas de Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Lance l’enregistrement de la vue en direct de l’index combiné au format vidéo/GIF → `RecorderHandle`. Un enregistreur composite par matrice. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Lancer une rafale Bayer brute à haute fréquence d’images → `RecorderHandle`. Retraiter hors ligne avec `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Retraiter hors ligne une rafale RAW enregistrée pour obtenir une ou plusieurs vidéos calibrées. Bloque jusqu’à la fin (`wait=True`) et renvoie `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Interroger une tâche de compilation hors ligne : `{running, result, error, burst_dir}`. |
| `disconnect()` | Libère l&#x27;ensemble du tableau. |

Commandes d&#x27;exportation `capture()` (même point de terminaison que celui utilisé par l&#x27;interface graphique/CLI) :

- `processing` / `levels` — `processing="all"` (ou `levels=["raw","radiance",…]`) enregistre chaque type d’exportation applicable par caméra ; une seule valeur `processing` enregistre uniquement ce niveau.
- `aligned=True` — aligne toutes les exportations non brutes des membres sur le [profil d’alignement](#array-alignment) (co-enregistré) ; les données brutes ne sont pas alignées mais intègrent la transformation dans les métadonnées. Le système revient par défaut à un alignement non défini (avec un avertissement affiché dans le champ `alignment` du résultat) si le tableau ne dispose d’aucun profil.
- `render_index=False` — ignore la superposition de l’indice de végétation par caméra ; par défaut, celle-ci est rendue là où elle a été configurée.
- `force_daq=True` — enregistre la lecture DAQ/DLS attribuée sous forme de fichier `.daq`, même si aucun niveau sélectionné ne le nécessite.

**CompressionTIFF (commande « HTTP » uniquement) :** `ArraySession.capture()` n’envoie pas de clé `compression`, donc la valeur par défaut du backend s’applique — `POST /api/camera/array/capture` lit un paramètre de corps `compression`, `"deflate"` par défaut (zlib L1 sans perte + prédicteur horizontal, ~4,1 Mo par image en pleine résolution). `"none"` écrit en format non compressé (~6,3 Mo/image) avec une**vitesse d’écriture environ 5 fois plus rapide** — les deux sont sans perte et se lisent de manière identique à l’importation. L’SDK ne fournit aucun kwarg pour cela ; la solution de secours est `chloros-cli lattice array-capture --compression none` ou l’HTTP brute. DEFLATE détient également le GIL d’Python, donc les écritures compressées ne peuvent pas être parallélisées entre les— une capture soutenue en pleine résolution sur 8 caméras à la fréquence du capteur nécessite `compression: "none"`. Détails : [CLI Référence → capture en tableau](cli-reference.md).**Remplacements d’exportation par membre (uniquement HTTP) :**le même point de terminaison accepte également `exclude_serials` (liste — supprimer des membres de l’ensemble enregistré ; le tableau continue de se comporter comme un groupe synchronisé unique et les membres exclus sont renvoyés dans `excluded`), `serial_levels` (remplacements au niveau de chaque caméra `{serial: [level tokens]}`), et `serial_index` (remplacements d’`{serial: bool}` par superposition d’index par caméra). Il s’agit de paramètres de corps en parité avec l’interface graphique et**pas encore d’arguments clés-valeur SDK** ; les éléments absents des tables utilisent par défaut les paramètres `levels` / `render_index` applicables à l&#x27;ensemble du tableau.

##### Inspection des caméras ignorées — `CaptureResult.skipped`

`ArraySession.capture()` renvoie un `CaptureResult`, qui est une sous-classe de `list`: parcourez-la, indexez-la, appliquez-lui `len()` — tous les modèles existants continuent de fonctionner. Le nouveau code peut inspecter l’attribut `.skipped` pour voir quelles caméras ont été exclues et pourquoi. Le cas le plus courant concerne les camérRGBes dans un réseau de filtres mixtes lorsque vous demandez des données `processing="radiance"` ou `"reflectance"` — la radiance par pixel Bayer n’a pas de sens pour un capteur à large bande ; le backend ignore donc ces caméras plutôt que de produire des résultats absurdes.

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

Les jetons de justification suivent le modèle `<level>-not-applicable-to-rgb-cam` (une entrée par niveau ignoré, chacune comportant `level`). Les sauts spécifiques à la réflectance sont `reflectance-skipped-no-fresh-dls` (aucune nouvelle mesure de rayonnement descendant disponible), `reflectance-skipped-bound-daq-unavailable (…)` (le DAQ associé n’a pas pu être atteint), et `dls-uncalibrated-band-<nm>` — la bande se situe en grande partie en dehors de la plage radiométriquement calibrée du capteur de lumière du DAQ (~374–974 nm) ; par conséquent, la division absolue de la réflectance basée sur le DAQ est refusée et l’image revient systématiquement à la réponse du capteur. Parmi les références commercialisées, seule la F988 déclenche ce comportement ; le parcours pris en charge par cette caméra est le flux de travail avec panneau de réflectance.

Niveaux `processing` :

| Niveau | Sortie |
| --- | --- |
| `"raw"` | Bayer monocanal (caméras monochromes : bande unique) provenant directement du capteur. |
| `"debayered"` *(par défaut SDK)* | BGR à 3 canaux via un dématriçage bilinéaire (caméras monochromes : 1 canal en niveaux de gris). |
| `"radiance"` | float32 W/m²/sr/nm via la chaîne radiométrique complète. Multispectral uniquement — les caméras RGBs sont ignorées. |
| `"reflectance"` | uint16 0..32768 (compatible Pix4D) ; nécessite un appairage DAQ en temps réel pour une référence absolue. Multispectral uniquement. |
| `"display"` | Chaîne complète correspondant à l’aperçu de l’interface graphique (CCM + WB + gamma selon le profil de la caméra). |
| `"all"` | **Un fichier par niveau applicable** pour chaque caméra (correspondant au paramètre par défaut de l’interface graphique «Capture All » / CLI par défaut). Le fichier `CaptureResult` renvoyé contient alors un dictionnaire d’images par `(cam, level)`, avec le niveau dans chaque dictionnaire ; les niveaux inapplicables apparaissent dans `.skipped`. La utilisée pour toute trame de réflectance est enregistrée sous forme de fichier d’accompagnement `.daq`. |

> **Remarque — la valeur par défaut diffère de celle de l’CLI.** `ArraySession.capture()` prend par défaut la valeur de `processing="debayered"` ; la commande `chloros-cli lattice array-capture` prend par défaut la valeur de `processing="all"`. Passez `processing="all"` explicitement depuis l’SDK pour reproduire la sauvegarde à plusieurs niveaux de l’CLI/GUI.

### Modes de capture et enregistreurs

La surface de la matrice reproduit le panneau de capture de l’interface graphique : modes d’obturation unique / Continu / Intervalle / Obturateur le plus rapide, ainsi que deux enregistreurs (vidéo composite en direct et rafale brute → retraitement hors ligne).

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

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**correspond à la boucle « Continu / Intervalle » de l’SDK. Comme il n’existe pas de `Ctrl+C` permettant de l’interrompre à partir d’un script, vous**devez** passer `count` et/ou `duration_s` (elle s’arrête dès que l’un ou l’autre est atteint). `interval_s` est mesuré à partir du début de chaque passage (conformément à l’interface graphique). Les autres arguments de ligne de commande sont transmis directement à `capture()`.
- **`record`** est de *niveau surveillance* : il capture le flux composite d’index combinés en temps réel tel qu’il s’affiche ; le flux combiné doit donc être ouvert pour que les images puissent être enregistrées. Un enregistreur composite par tableau (lance une exception si un autre est déjà en cours d’exécution).
- **`burst` → `build_video`** est de *niveau analyse* : `burst` écrit les images brutes + un-image + un fichier `.daq` par lecture DLS distincte sous `<output>/bursts/<base>/` à la vitesse maximale de la boucle de capture (pas de chaîne, pas d’exiftool, pas de visualisation en direct). `build_video` synchronise chaque image avec le `.daq` le plus proche et relance la chaîne de radiance/réflectance/indice du pipeline d’importation. `products` est une liste de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (par défaut : l&#x27;indice combiné). `burst().stop()` lance également automatiquementgénère une version de l’indice combiné « au mieux », renvoyée sous la forme `build_job` dans le résultat d’arrêt.

#### `RecorderHandle`

Renvoyé par `ArraySession.record()` et `ArraySession.burst()`. Utilisez-le comme gestionnaire de contexte pour arrêter automatiquement le processus à la sortie de la portée, ou contrôlez-le manuellement.

| Membre | Description |
| --- | --- |
| `job_id` | ID de la tâche backend (chaîne). |
| `kind` | `"composite"` (provenant de `record`) ou `"raw"` (provenant de `burst`). |
| `start_stats` | Le dictionnaire renvoyé par l&#x27;appel `start`. |
| `result` | `None` pendant l&#x27;exécution ; le dictionnaire final des résultats d&#x27;arrêt une fois l&#x27;opération terminée. |
| `stats(timeout=10.0)` | Statistiques en temps réel du travail (nombre d’images écrites, nombre d’images par seconde réel, temps écoulé). |
| `stop(timeout=60.0)` | Arrête l’enregistreur ; renvoie et met en cache le résultat final. Idempotent (un deuxième appel renvoie le résultat mis en cache). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Connexion à un— `attach_array`

Si le tableau est déjà actif (l’interface graphique l’a ouvert, ou une session précédente de SDK a appelé `connect_array`), utilisez `attach_array` pour obtenir un descripteur vers celui-ci à la place de le-connexion. `connect_array` renvoie toujours l’erreur « La caméra  fait<sn> déjà partie du tableau <id>» dans cette situation, car l’envoi d’une requête POST à `/array/connect` pour un élément du pool n’est pas idempotent ; `attach_array` lit `/api/camera/array/list` et effectue la correspondance soit par array_id, soit par numéros de série.

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

Modèle : SDK Les scripts partageant l’environnement avec l’interface graphique de bureau doivent d’abord essayer `attach_array`, puis se rabattre sur `connect_array` si aucun tableau n’est encore présent dans le pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Important — la sortie du gestionnaire de contexte entraîne bel et bien une déconnexion.**`ArraySession.disconnect()` envoie toujours un POST à `/array/disconnect` ; il n’existe pas de garde « attached-not-owned » comme c’est le cas pour `CameraSession` / `DAQSensorSession`. Si vous partagez l’environnement avec l’interface graphique et que vous ne souhaitez pas détruire le tableau à la sortie de la portée,**n’utilisez pas le bloc `with`** — conservez le descripteur dans une variable normale et ignorez le XPR expliciteOTX000486 :
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Aide à l’analyse réseau

Utile avant d’ouvrir le tableau — permet de vérifier si les paramètres proposés sont adaptés :

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
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` est l’un des éléments suivants : `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (sinon `error`). `auto_capped_fps` signifie que la résolution demandée ne s&#x27;adapte à l&#x27;anneau RX qu&#x27;avec une fréquence de déclenchement plafonnée — conservez la résolution et passez de `target_fps=result["recommended"]["recommended_target_fps"]` à `connect_array` (voir [Exemple 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Comment interpréter la projection** (même modèle que le panneau « Array Settings » de l’interface graphique) :

- **La rafale (`frame_bytes_total`) est additionnée par caméra au format de pixels réel de chaque caméra.**Les caméras mono**M3M**transmettent en flux Mono12 (2 octets/px) quel que soit le paramètre `pixel_format` que vous transmettez ; ainsi, une image en pleine résolution à 4 caméras pèse**environ 25 Mo** avec trois caméras mono, et non les ~12,6 Mo que l’hypothèse d’un format entièrement 8 bits laisserait supposer. Le backend détermine le format de chaque caméra à partir de son modèle.
- **L’admittance (`burst_fits_nic_ring`) est sensible au débit**sensible au débit de sortie**, et non pas « rafale entière » contre « anneau » : la simulation d’émission s’applique lorsque l’hôte vide l’anneau de réception plus rapidement que les caméras ne le remplissent. Un hôte 10G + des caméras 1 GbE**admettent** la pleine résolution même lorsque la rafale dépasse la capacité de l’anneau ; un hôte 1 GbE bloque (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` correspond à une limite supérieure prudente pour la récupération en série** — `max(readout+emit, N×emit)` avec l’émission par caméra limitée au Camera Link 1 GbE, indépendamment de l’exposition. Par exemple, environ 2,8 images par seconde pour un réseau de 4 caméras en pleine résolution 12 bits (ce qui correspond aux valeurs mesurées en exécution, comprises entre ~2,7 et 3,0). Modèle complet : [CLI Référence → Modèle de fps et de rafales pour les réseaux de caméras](cli-reference.md#array-fps--burst-model).
- **La sursouscription (`oversubscribed: true`) signifie que la somme des limites minimales par caméra (N ×) dépasse la limite maximale garantissant l’absence de collision** — les champs de fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) affichent 0, et la réduction automatique ou le regroupement ne permettent pas de résoudre le problème (ces opérations réduisent le nombre d’octets par trame, et non le nombre d’octets par seconde). Les solutions consistent à réduire le nombre de caméras, à utiliser des trames jumbo ou à installer une carte réseau plus rapide ; `max_cams_collision_safe` indique le plafond (6 caméras en pleine résolution sur 1 GbE à 1 500 MTU, 9 avec des trames jumbo). La réponse contient également les codes `aggregate_demand_bps`, `collision_safe_ceiling_bps` et `per_cam_floor_bps` (8 Mo/s). Voir [Sursouscription](#over-subscription-the-per-cam-floor).

### Découverte et répertoriage

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Les matrices LATTICE exécutent une exposition automatique (AE) en continu en arrière-plan dès qu’elles sont connectées, mais une scène nouvellement cadrée met un instant à converger. **Smart-capture** est la solution pratique intégrée : elle interroge l’exposition de chaque caméra, attend que la matrice soit stable sur une fenêtre, puis déclenche la capture. Elle est l’équivalent de l’interface graphique : le bouton de capture « intelligente » de l’application de bureau appelle le même point de terminaison backend.

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

Lorsque vous utilisez `ChlorosProject` (section suivante), vous disposez de plus de paramètres :

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

La politique d&#x27;exposition automatique « intelligente » est conservatrice par défaut. Resserrez le paramètre `exposure_tolerance_pct` pour les travaux radiométriques exigeants ; assouplissez-le pour les scènes évoluant rapidement où vous souhaitez simplement un résultat « suffisamment proche ».

---

## Sessions de capteurs DAQ

Pool de serveurs backend persistant pour les capteurs spectraux (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Reproduit le fonctionnement de la caméra : détection intelligente, réutilisation du pool, connexion idempotente.

### Détection intelligente (configuration automatique)

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

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Méthodes `DAQSensorSession`

| Méthode | Description |
| --- | --- |
| `status(timeout=10.0)` | Résumé d’une entrée du pool (état de streaming/enregistrement, plage de longueurs d’onde, SHA d’étalonnage, temps d’intégration, frame_avg, état AE). |
| `latest(n=1, timeout=10.0)` | Renvoie jusqu’à N trames de spectre les plus récentes. |
| `stream_start()` / `stream_stop()` | Reprendre / mettre en pause le streaming (le descripteur reste ouvert). |
| `record_start(output_dir=None, device_name=None)` | Démarrer l&#x27;enregistrement d&#x27;un fichier .daq. Renvoie le chemin d&#x27;accès au fichier. Échoue pour les DAQ-U/M sans pack d&#x27;étalonnage AWS (le DAQ-E est exempté). |
| `record_stop()` | Arrête l&#x27;enregistrement. Renvoie `{path, rows}`. |
| `disconnect()` | Libération du pool. Aucune opération pour les descripteurs attachés mais non détenus. |

> **Les profils de correction de crête (`cap_id`) ne sont pas un paramètre d’SDK.** `connect_daq_sensor()` / `DAQSensorSession` n’exposent aucun paramètre `cap_id` ni aucune méthode `set_cap`. Sélectionnez un profil de correction de plafond de flotte via l’CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) ou les routes d’HTTPs `/api/daq` du backend (`/api/daq/connect` et `/api/daq/<id>/cap-id` acceptent `cap_id`).

### Découverte — trouver une adresse pour se connecter

`discover_daq_sensors()` analyse les ports USB / BLE / ETH à la recherche de capteurs que vous *pourriez* ouvrir. Il s&#x27;agit de l&#x27;équivalent DAQ de `discover_lattice_cameras()`, et du seul moyen d&#x27;obtenir l&#x27;adresse MAC BLE d&#x27;un **DAQ-M** — un DAQ-E dispose d’un nom d’hôte et un DAQ-U d’un port COM, mais l’adresse MAC n’est ni imprimée sur l’appareil ni répertoriée par le système d’exploitation.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Champ | Description |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / MAC BLE / nom d&#x27;hôte — à transmettre à `connect_daq_sensor` sous la forme `port=` / `mac=` / `eth_host=`. |
| `display` | Étiquette lisible par l&#x27;utilisateur. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, ou `None` pour un port que l&#x27;analyse ne parvient pas à identifier (les adaptateurs série USB sont impossibles à distinguer sans sonde ; les éléments inconnus sont donc affichés plutôt que masqués). |
| `extra` | Détails par transport (nom annoncé BLE, fabricant USB, adresse IP/firmware DAQ-E/…). Les valeurs vides sont omises. |

| Paramètre | Valeur par défaut | Description |
| --- | --- | --- |
| `transports` | les trois | Séquence (ou chaîne CSV) limitant l’analyse. À spécifier lorsque vous savez ce que vous voulez — le BLE est le maillon faible. |
| `scan_timeout` | 5 | Fenêtre de balayage par transport en secondes ; le backend limite cette valeur à 1–20. |
| `timeout` | 60,0 | Plafond « HTTP » pour l’ensemble de l’appel (comme ailleurs dans l’SDK). |
| `auto_start_backend` | `True` | Lance un backend local si aucun n’est en cours d’exécution. Ne se lance jamais pour un `backend_url` distant. |

> **Les capteurs déjà ouverts dans le pool n’apparaissent pas.** Un périphérique BLE connecté cesse de diffuser ses informations et un port COM ouvert ne peut pas être sondé ; la découverte répertorie donc ce qui est *disponible pour la connexion*. Il est normal d’obtenir un résultat vide juste après avoir connecté un périphérique — utilisez `list_daq_sensors()` pour ce que vous possédez déjà. Les transports dont l’analyse ne peut pas s’exécuter (absence d’installation de bleak / zeroconf) sont ignorés plutôt que de générer une exception ; ainsi, une machine sans Bluetooth obtient tout de même ses réponses USB et ETH.

### Liste

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Co-location avec l’interface graphique / CLI

Si l’interface graphique a déjà un capteur ouvert, l’appel de `connect_daq_sensor(port="COM3")` depuis Python renvoie un descripteur marqué `already_connected=True`. Le `disconnect()` de la session est alors une opération sans effet (no-op), ce qui permet à votre script SDK de ne pas le capteur de l’interface graphique à la fermeture de Scope.

### Classes matérielles directes (sans backend)

`daq_sdk` est réexporté par `chloros_sdk`, ce qui vous permet également de piloter les capteurs de bout en bout en cours de traitement sans backend :

> **Disponibilité :**`daq_sdk` est fourni avec l’installation de bureau d’Chloros,**mais pas** avec le paquet PyPI — `pip install chloros-sdk` vous fournit `lattice_sdk` mais omet `chloros_sdk.DAQ_AVAILABLE == False`. Vérifiez ce paramètre avant d’utiliser ces classes ; sur un hôte utilisant uniquement pip, pilotez le capteur via [`connect_daq_sensor()`](#daq-sensor-sessions), qui ne nécessite aucune bibliothèque de transport locale.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Privilégiez le chemin smart-connect (`connect_daq_sensor`) lorsque vous souhaitez un partage de propriété avec l’interface graphique ; utilisez les classes directes pour les scripts sans interface graphique qui possèdent le capteur en exclusivité.

---

## Automatisation de projet — `ChlorosProject`

Un projet Chloros enregistré est un dossier contenant `cameras.json` + `sensors.json` + `project.json`. `open_project` charge le manifeste, et `connect_all` met en ligne tous les périphériques enregistrés avec leurs paramètres enregistrés — soit le même état matériel que celui que produirait l’interface graphique.

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

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
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
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Détecte et connecte tous les périphériques enregistrés. Renvoie un rapport de connexion par classe. Utilise un backend en cours d’exécution lorsqu’un tel backend est à l’écoute sur `127.0.0.1:5000` ; sinon, bascule silencieusement vers le contrôle direct (sans backend) du périphérique `lattice_sdk` — il ne lance jamais de backend. |
| `disconnect_all()` | Déconnecte tout. |
| `capture_all(output_dir=".")` | Une image de chaque caméra + tableau + spectre de chaque capteur. |
| `stream(camera, overlays=False, fps=10.0)` | Générateur produisant des images BGR `numpy` à partir d’une caméra nommée (ou d’un réseau). `overlays=False` est une boucle de capture directe `lattice_sdk` (les réseaux génèrent des dictionnaires `{serial: frame}`). `overlays=True` achemine via `ChlorosLocal.camera_stream()` → le flux MJPEG `/api/camera/<serial>/stream-annotated` du backend, le bloc `ui.overlay` enregistré de la caméra étant transmis en tant que paramètres de requête. Nécessite le mode backend et une **caméra autonome** : une caméra en mode direct génère une exception `RuntimeError` (le backend ne peut pas récupérer une caméra appartenant à ce processus) et un tableau génère `NotImplementedError` (superpose les flux composites par caméra — diffuse un élément par son nom). Équivalent en une seule opération : `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Exécute l’alignement sur chaque matrice actuellement connectée. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Exécute le pipeline d&#x27;étalonnage / d&#x27;indexation sur les images du projet (englobant `ChlorosLocal.process` ; ces quatre-là sont les **seuls** arguments clés acceptés — `indices=` etc.. déclenche une exception `TypeError` ; définit les indices via `ChlorosLocal.configure()`). Construit de manière différée un `ChlorosLocal()`, qui démarre automatiquement un backend. |

Attributs :
- `proj.cameras` — `Dict[str, CameraHandle]` indexé par nom ET numéro de série.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexé par nom ET array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexé par nom ET slot_id.
- `proj.config` — `project.json["config"]` dict.

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

**Niveaux de traitement.** `capture()`, `grab()` et `frame_stream()` prennent tous le même jeton `processing`,
et la chaîne est cumulative — chaque niveau exécute tout ce qui se trouve au-dessus de lui :

| Niveau | Sortie | Remarques |
| --- | --- | --- |
| `raw` | Bayer 1 canal, natif du capteur | Pas de démosaïquage. Les superpositions ne sont pas disponibles à ce niveau. |
| `debayered` | BGR 3 canaux (**par défaut**) | Démosaïquage. Seul niveau fonctionnant sans mode backend. |
| `radiance` | float32, W/m²/sr/nm | Chaîne radiométrique complète : dématriçage + démixage 3×3 (multispec) + DSNU + champ plat + échelle NIST, avec l&#x27;exposition × le gain divisés pour que les valeurs soient absolues. |
| `reflectance` | uint16, 32768 = 1,0 | Radiance divisée par l’irradiance descendante (ρ = π·L/E). Nécessite une lecture DLS/DAQ — voir la note ci-dessous. |
| `display` | 8 bits, de type sRGB | Rendu équivalent à celui de l&#x27;interface graphique : CCM + balance des blancs + gamma via le profil de couleur actif de la caméra. |

Toute valeur autre que `debayered` nécessite le mode backend ; une caméra en mode direct génère
`NotImplementedError`. `reflectance` nécessite une lecture d&#x27;irradiance descendante exploitable — le point de fin de l&#x27;image extrait
automatiquement le DAQ regroupé dans l&#x27;, mais en l’absence de DAQ lié, la chaîne refuse la
sortie de réflectance et indique clairement la rétrogradation dans les métadonnées renvoyées plutôt que de
restituer silencieusement un produit de qualité inférieure.

> **Échelle DN de réflectance — ne pas la coder en dur.** La réflectance LATTICE utilise `32768` = ρ 1,0 et marque
> XMP `Chloros:PixelScale=32768` ; la réflectanceSurvey3 la réflectance utilise `65535` = ρ 1,0 et ne comporte aucune
> balise `Chloros:*`. Lisez la balise et divisez par cette valeur. Elle est définie dans le domaine uint16, elle reste donc
> `32768` pour tous les formats qui effectuent un redimensionnement (TIFF 16 bits, PNG /JPG 8 bits, pourcentage 32 bits) — normalisez
> d’abord le type de données stocké en uint16 (×257 à partir de 8 bits, ×65535 à partir de float). Seule exception :
> une capture provenant d’une source 8 bits et enregistrée au format 8 bits TIFF est *écrêtée*, et non redimensionnée ; aucun facteur d’échelle ne la décrit
> — Chloros omet entièrement `PixelScale` et le tuple MicaSense dans ce cas. Considérez une balise manquante
> dans un fichier de réflectance LATTICE comme « aucune échelle valide », et non comme une valeur par défaut.

> **Les données EXIF sont conservées lors de l’exportation.** `process()` copie lebloc GPS de la capture source
> **et son ExifIFD** sur chaque produit ; les exportations contiennent donc `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` et `CameraSerialNumber`, ainsi que le
> géoréférencement. C’est à partir de `FocalLength` que Pix4D calcule la distance d’échantillonnage au sol — sans lui
> la reconstruction revient à une échelle complètement erronée (un cas mesuré a transformé un site de 411 m
> en un site de 47,8 km). La copie n’est délibérément pas `-all:all` : les balises structurelles d’IFD0 perturbent
> la sortie LATTICE, et `ExifImageWidth`/`Height` sont exclus car ils décrivent la
> capture source plutôt que le raster exporté.

Sous-indicateurs de la phase de capture (s&#x27;appliquent aux niveaux radiométriques — `radiance`, `reflectance`, `display`) :

| Indicateur | Par défaut | Signification |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + champ plat-champ + démixage 3x3 + échelle radiométrique NIST. |
| `apply_white_balance` | `True` | LUT WB. Prise en compte du DLS lorsqu’un DAQ est associé à la caméra. |
| `apply_index` | `False` | Évaluation de l&#x27;indice de végétation. |
| `index_expression` | `None` | Formule de remplacement. Non vide →-active l&#x27;indice. |
| `annotated` | `False` | Superposition des éléments décoratifs de l&#x27;interface graphique (zébrure/grille/crête). Non disponible pour `raw`. |

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

> **Le type de retour est `CapturePathMap`, et non `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` correspond à `Dict[str, Union[str, List[str]]]` : un chemin à un seul niveau
> `processing` attribue un chemin à chaque numéro de série, tandis qu’une structure à plusieurs niveaux (`"all"`, ou une
> liste explicite `levels`) lui fournit la **liste ordonnée** de tous les produits enregistrés pour cette
> caméra. Un composite combiné en direct, s&#x27;il était diffusé en continu, arrive sous la clé supplémentaire
> `"combined"` plutôt que sous un numéro de série. Un code qui suppose `str` plante sur la
> forme liste sans qu’aucun vérificateur de types ne s’y oppose — l’annotation indiquait `Dict[str, str]`
> pendant un certain temps après la mise en production de la forme liste, ce qui explique l’existence de l’alias. Normalisez
> lorsque vous le souhaitez la forme plate :
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Alignement des tableaux

`ArrayHandle` expose la surface d’alignement complète. Les profils sont par défaut limités à la session — appelez explicitement `export_alignment()` pour les persister.

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

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alignement au moment de la connexion

`connect_all(align=...)` permet d’aligner automatiquement chaque tableau lors de la connexion :

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

Récourt à `project.json["config"]["auto_align_on_connect"]` si non spécifié.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Matériel direct (sans backend)

Lorsque vous souhaitez une dépendance nulle vis-à-vis du backend (CI, robots sans interface graphique, embarqué), importez directement `lattice_sdk` et `daq_sdk` — ces deux éléments sont réexportés par `chloros_sdk`. Protection sur `CAMERA_AVAILABLE` / `DAQ_AVAILABLE` : `lattice_sdk` se trouve dans le paquet PyPI (mais nécessite la présence du runtime Arena SDK), tandis que `daq_sdk` n’est fourni qu’avec l’installation pour ordinateur de bureau.

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

##### Préréglages et déclencheur

Trois des quatre préréglages fonctionnent en **mode libre** : la caméra expose en continu et un
`capture()` renvoie l’image suivante. `triggered` fait exception : il arme la
appareil photo pour qu’il détecte un front matériel sur la ligne 2 ; il ne capture donc rien tant qu’un front n’est pas détecté.

| Préréglage | Déclencheur | À utiliser lorsque |
| --- | --- | --- |
| `default` | fonctionnement libre | usage général |
| `high_speed` | fonctionnement libre | 8bits, limite à 60 images par seconde, exposition courte |
| `high_quality` | fonctionnement libre | 12 bits, pas de limite d’images par seconde — le choix habituel pour les photos |
| `triggered` | **armé, ligne 2** | l’appareil photo est branché à un câble de synchronisation M8 et un autre élément le déclenche |

Si vous choisissez `triggered` (ou si vous configurez vous-même `trigger_mode="On"`) sans qu’aucun
signal ne soit présent sur la ligne 2, chaque `capture()` expirera — ce qui est normal, puisque vous avez demandé
à la caméra d’attendre. L’SDK explique ce phénomène lorsqu’il se produit ; voir
[SC_ERR_TIMEOUT pendant la capture](#direct-hardware-backend-free).

> **Remarque — Les messages « GVSP probe » / `SC_ERR_TIMEOUT -1011` lors de la connexion ne sont pas des erreurs.**&gt; Lors de la connexion, l’SDK tente de négocier des**trames jumbo** (paquets GVSP de 9 000 octets) pour un débit plus élevé. Sur une liaison réseau directe point à point (par exemple, une adresse `169.254.x.x` locale au niveau de la liaison), le réseau ne peut généralement pas acheminer de trames jumbo ; cette tentative aboutit donc à un délai d’expiration et génère des lignes de journal telles que :
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Il s’agit de la **solution de repli prévue** : l’SDKe revient automatiquement aux paquets standard de 1 500 octets et la caméra continue de se connecter normalement (les lignes `[chunk-enable …]` qui suivent font partie de la séquence de connexion normale). La capture fonctionne toujours.
>
> Vous pouvez ignorer cette sonde, mais **elle ne sert pas seulement à désactiver les messages de journalisation — elle désactive les trames jumbo.** La caméra ne répond aux pings « Don&#x27;t-Fragment » que jusqu’à 1 500 octets, quelle que soit la qualité de votre réseau ; le test de ping seul ne permet donc jamais de détecter les trames jumbo ; ce test de vérification est le seul à le permettre. Désactivez-le et la caméra utilisera indéfiniment des paquets standard de 1 500 octets, sur n’importe quel réseau :
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Cela ne vaut la peine que sur un réseau dont vous *savez* qu’il ne prend pas en charge les paquets « jumbo », où cela permet de gagner environ une seconde de temps de connexion par caméra. Comme il s’agit d’un véritable compromis et non d’une simple modification cosmétique, l’SDKe le précise désormais lorsque vous l’utilisez :
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Ne touchez pas à ce paramètre sauf si vous avez une bonne raison.** Si cette option reste activée, chaque connexion réévalue le réseau dont vous disposez réellement : branchez-vous sur un commutateur prenant en charge les paquets « jumbo » et la connexion suivante détectera automatiquement ce mode, sans aucune configuration ni redémarrage.
>
> Si vous *souhaitez* bénéficier du débit « jumbo », activez le mode « jumbo » de bout en bout (MTU de la carte réseau à 9 000 + un commutateur qui les transmet), ou fixez-la avec `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` lorsque vous savez que la liaison la prend en charge — même s’il est préférable d’utiliser `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` par commande plutôt que de la définir de manière permanente, car une taille fixe ignore la détection et empêche l’adaptation au réseau en amont. **Tous les** périphériques sur le chemin doivent prendre en charge les paquets jumbo — y compris tout répartiteur ou injecteur PoE, ce qui explique généralement pourquoi une configuration par ailleurs compatible avec les paquets jumbo ne parvient pas à les acheminer.

> **`SC_ERR_TIMEOUT -1011` pendant `capture()` / `grab*()` est un problème différent — celui-ci est une véritable erreur.**&gt; La remarque ci-dessus concerne uniquement l’erreur `-1011` enregistrée par la**sonde connect-time**. La même erreur générée par une**capture** signifie que la caméra s’est correctement connectée mais n’envoie aucune image :
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Le signe révélateur est une caméra dont le canal de *contrôle* fonctionne correctement — la détection fonctionne, les réglages et les écritures `[chunk-enable …]` aboutissent tous — alors que *toutes* les images dépassent le délai d’attente.
>
> **La cause habituelle est que la caméra est configurée pour un déclenchement matériel.** Avec les codes `trigger_mode="On"` et `trigger_source="Line2"`, la caméra n’émet absolument rien tant qu’un front d’impulsion n’arrive pas sur le câble de synchronisation M8. Si aucun câble ne transmet ce signal, chaque capture attend indéfiniment. La caméra n’est pas défectueuse et le réseau fonctionne correctement — elle fait exactement ce qu’on lui a demandé.
>
> Les codes `CameraSettings()` et `default` / `high_speed` / `high_quality` définissent le mode libre, et une capture qui expire alors qu’elle est activée s’explique d’elle-même au lieu d’afficher simplement `-1011`. `PRESETS["triggered"]` active la ligne 2, conformément à la conception.
>
> Pour forcer n&#x27;importe quelle caméra à fonctionner en mode libre :
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Si le délai d&#x27;expiration persiste avec `trigger_mode="Off"`, cela signifie que la caméra ne transmet réellement pas de données — envoyez-nous le journal et `ip link show`.

#### Profils de couleurs (aperçu en direct RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` permet de sélectionner le profil de couleur d’affichage pour l’**aperçu en direct** sur les caméras RGB (les caméras multispec ignorent ce paramètre) :

| Profil | Signification |
| --- | --- |
| `raw` | Contourner entièrement la chaîne radiométrique. |
| `linear` | DSNU + flat + WB, pas de CCM, pas de gamma. |
| `natural` | Linéaire + CCM mesuré + gamma sRGB, avec uniquement le rendu bas de gamme (lissage de la chrominance + désaturation des hautes lumières) — le réglage par défaut réaliste. |
| `enhanced` | `natural` plus la finition complète « hub-parity » (suppression des franges, vibrance, contraste local CLAHE). Un rendu plus riche pour environ **le double du coût de traitement par image**, d’où une fréquence d’images en temps réel plus faible. |
| `custom_temp` | `natural` mais balance des blancs verrouillée sur `custom_cct_k` Kelvin (DLS ignoré ; limité à 2 000–10 000 K côté backend). |

Le profil est un **prévisionnel en direct** : les captures enregistrées bénéficient toujours d’un rendu riche et complet, quel que soit le profil sélectionné ; ainsi, choisir `natural` pour gagner du temps par image ne réduit pas la qualité du résultat final sur le disque. Un profil inconnu augmente la valeur de `ValueError` ; lorsqu’ un backend chloros est accessible, la modification y est également envoyée via POST afin que l’image d’aperçu suivante en tienne compte (les utilisateurs de direct-SDKs sans backend bénéficient tout de même de la modification des paramètres).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Caméras mono (M3M) et `Calibration`

Une caméra mono **M3M** (`M3M-<lens>-F<wavelength>`) est monobande : un seul plan en niveaux de gris, pas de mosaïque de Bayer, pas de matrice de diaphonie spectrale 3×3. `Calibration` la reconnaît et expose un indicateur `is_mono`. La réflectance s&#x27;applique toujours sous forme de(la décomposition correspond à la matrice identité), mais les calculs multibandes sur une seule caméra génèrent des résultats valides plutôt que des valeurs absurdes :

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

Pour construire un indice de végétation à partir d’un matériel monochrome, combinez plusieurs caméras M3M à différentes longueurs d’onde en une pile multibande alignée (voir [Alignement de matrices](#array-alignment)) et calculez l’indice sur l’ensemble de cette pile plutôt que sur une seule caméra.

Mode direct DAQ :

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Clés acceptées par `apply_sensor_settings`**— à savoir exactement `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E ; obsolète, remplacé par `cap_id`), `filter_model` (DAQ-M) et `cap_id` (tous les types de DAQ ; `None`/`""`/`"none"` = capteur nu, sans correction de capacité). Les clés inconnues sont**ignorées sans message d&#x27;erreur** — par exemple, `{"integration_time": 64}` ne fait rien (il doit s’agir de `integration_time_ms`). Renvoie `{"applied": [...], "errors": {...}}` et ne lève jamais d’exception.

`chloros_sdk` réexporte uniquement la surface principale utilisée ci-dessus. L’API publique complète `daq_sdk` (22 noms) ajoute les éléments suivants — importez-les directement depuis `daq_sdk` :

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

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

> `ChlorosAuthenticationError` et `ChlorosConfigurationError` sont exportés au niveau supérieur avec les autres ; ils peuvent également être importés depuis `chloros_sdk.exceptions`, comme indiqué.

Hiérarchie :

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

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

### 1. Traitement d’un dossier avec une barre de progression personnalisée

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
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
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

### 4. Flux d&#x27;images multi-caméras → Pipeline NumPy

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

### 5. Script de capture « headless » en mode direct-matériel (sans backend)

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

### 6. Test des capacités avant la connexion d’un ensemble de 4 caméras

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
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Équivalent d’une recette de capture (Python pur)

Le langage DSL de recettes de l’CLI dispose d’un équivalent direct en Python :

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

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

## Démarrage automatique du backend

Les points d’entrée smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` et `discover_lattice_cameras` — sont des clients «HTTP» légers qui partent du principe qu’un backend est à l’écoute sur `127.0.0.1:5000` (URL par défaut de la surface Smart-Connect). Lorsque l’interface graphique ou le service CLI est déjà en cours d’exécution, c’est le cas. À partir d’un script nu, il se peut qu’il n’y en ait pas — ces fonctions **lancent donc automatiquement le binaire du backend fourni** (sans fenêtre, de la même manière que le fait `ChlorosLocal`) avant leur premier appel, puis attendent jusqu’à `backend_startup_timeout` qu’il soit opérationnel.

Règles :

- **Seul un URL local est jamais lancé.** Un `backend_url` pointant vers `localhost` / `127.0.0.1` / `[::1]` est éligible ; tout autre hôte est considéré comme appartenant à quelqu’un d’autre et n’est jamais créé.
- **Le backend reste en cours d’exécution pour être réutilisé** (comme l’CLI) — il n’y a pas d’arrêt implicite lorsque votre script se termine. La réexécution du script réutilise le backend actif.
- **Vous pouvez désactiver cette fonctionnalité avec `auto_start_backend=False`** sur n’importe lequel de ces appels (par exemple, lorsque vous avez pointé vers un backend distant, ou si vous gérez vous-même le cycle de vie du backend).

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

Si le binaire fourni ne peut être localisé ou démarré, l’appel HTTP qui suit déclenche une exception exploitable, **adaptée à la plateforme** `ChlorosConnectError` plutôt qu&#x27;une simple trace de connexion— sur Windows, elle vous redirige vers l’application de bureau ou une commande `chloros-cli` ; sur Linux (sans interface graphique), elle vous redirige vers une commande `chloros-cli` ou vers `.deb`.

---

## Environnement et en-têtes

L’SDK marque chaque appel au backend HTTP avec `X-Chloros-Client: sdk`. Le backend applique les règles de licence SDK / CLI (connexion **et** un forfait payant Chloros+ est requis) plutôt que la formule gratuite de l’interface graphique. Ce paramètre est défini automatiquement lors de l’importation — vous n’avez rien à faire.

`http://localhost` et `http://127.0.0.1` sont détectés comme backend local. Les appels vers d’autres hôtes (par exemple, votre propre service d’analyse) ne sont pas modifiés.

Remplacez l’URL du backend en passant `backend_url=` (ou `api_url=` sur `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un `backend_url` non-loopback n&#x27;atteint qu&#x27;un backend source/dev — les backends fournis se lient uniquement en loopback ; voir Mode backend distant pour le schéma de tunnel.)

---

## Gestion des versions et compatibilité

- La version « SDK » est exposée sous le nom `chloros_sdk.__version__`.
- L’SDKe le comportement des broches à celui de la version du backend fournie. La combinaison d’un SDK plus ancien avec un backend plus récent fonctionne généralement (points de terminaison compatibles en aval), mais la combinaison d’un SDK plus récent avec un backend plus ancien peut entraîner des erreurs `404` sur les nouveaux points de terminaison — mettez à jour l’application de bureau pour qu’elle soit compatible.
- L’interface Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) et le point de terminaison d’analyse réseau renvoient des schémas stables JSON ; les nouveaux champs sont cumulatifs.

---

## Conseils de dépannage

- **`ChlorosAuthenticationError: Login required`** → Exécutez `chloros-cli login EMAIL PASSWORD` une fois sur cette machine, ou connectez-vous via l’application de bureau Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Les appels Smart-Connect lancent automatiquement un backend local ; ce message n’apparaît donc que lorsque le binaire fourni est introuvable ou ne peut pas être lancé (par exemple, sur un hôte pipsans package de bureau). Le message s’adapte à la plateforme : sur Windows, ouvrez l’application de bureau ou exécutez n’importe quelle commande `chloros-cli` ; sur Linux, exécutez une commande `chloros-cli` (aucune interface graphique n’existe) ou installez le fichier `.deb`. Pour un backend distant, transmettez `backend_url=` (et `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** lors de l&#x27;importation → Échec du chargement de `lattice_sdk` (généralement, les DLL d&#x27;exécution d&#x27;SDKs Arena ne sont pas installées). La surface hors caméra fonctionne toujours.
- **La connexion de l&#x27;array renvoie une résolution inférieure à la résolution native**→ La fonction « smart-prep » du backendréduit automatiquement la taille de l&#x27;image pour l&#x27;adapter à la bande passante. Utilisez `analyze_array_network()` pour en comprendre la raison, puis mettez à niveau la liaison, acceptez la réduction ou passez à `force_tier="slip-emit-and-capture"` pour une capture séquentielle. Le mécanisme de sécurité de réduction**ne**couvre**pas** la sursouscription agrégée (`oversubscribed: true`, champs fps à 0) : un nombre trop élevé de caméras pour la liaison ne peut pas être résolu par le regroupement de trames/la zone d’intérêt — réduisez le nombre de caméras, activez les trames jumbo ou passez à une carte réseau plus rapide (voir [Sursouscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` signale que l&#x27;anneau de réception de la carte réseau est très petit (~0,26 Mo) / les portes de connexion affichent « FRAMES WILL DROP »** → L&#x27;anneau de réception de la carte réseau de l&#x27;hôte est à sa valeur par défaut (souvent réinitialisé à 32 après une mise à jour du pilote de la carte réseau). Sur une carte Realtek USB 10 GbE, définissez les paramètres `ReceiveBufferLen=256` et `PendingReceives=64` (valeurs élevées), puis redémarrez le backend afin qu’il relise la file d’attente. Procédure complète : [Référence CLI → Configuration et réglage de la carte réseau hôte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **L&#x27;hôte se bloque au redémarrage/à l&#x27;arrêt, suivi d&#x27;erreurs WMI `Invalid class` / la carte réseau ne s&#x27;active pas** → Pilote USB 10 GbE obsolète provoquant `DRIVER_POWER_STATE_FAILURE` (écran bleu `0x9F`). Mettez à jour le pilote de la carte réseau vers une version récente (≥ 2026) et réappliquez les paramètres du receive-ring. Voir [Référence CLI → Configuration et réglage de la carte réseau hôte](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Réflectance refusée** → Un système d’acquisition de données (DAQ) actif doit être associé à la caméra (ou au réseau de caméras) pour obtenir une réflectance à échelle absolue. Effectuez l&#x27;association via l&#x27;interface graphique ou utilisez `processing="radiance"` (W/m²/sr/nm), qui ne nécessite pas de capteur associé.
- **La capture `smart=True` prend plus de temps que prévu** → La convergence AE dépend de la dynamique de la scène ; réduisez la valeur `exposure_tolerance_pct` ou raccourcissez `stability_window_s` si vous souhaitez un déclenchement plus rapide (mais moins stable).

---

## Voir aussi

- [Référence CLI](cli-reference.md) — chaque sous-commande CLI correspond à un appel SDK.
- [Guide des capteurs DAQ](../daq/README.md) — règles de câblage, d’étalonnage et d’enregistrement spécifiques aux capteurs.
- Documentation en ligne : `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
