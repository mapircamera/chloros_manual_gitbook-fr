# CLI : Ligne de commande

> **Référence complète :**[CLI Référence](reference/cli-reference.md) documente**chaque option de chaque sous-commande** et est optimisée pour les assistants IA — collez son URL dans votre assistant et demandez une commande fonctionnelle : `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Astuce pour les outils d’IA :** n’importe quelle page de ce manuel est disponible au format Markdown brut en ajoutant `.md` à son URL (par exemple `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), et `https://mapir.gitbook.io/chloros/llms.txt` indexe l’intégralité du manuel pour une utilisation par un LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Qu’est-ce que l’CLI
?

`chloros-cli` est l’interface en ligne de commande du même moteur de traitement que celui utilisé par l’application de bureauChloros
. Il s’agit d’un client d’HTTP
s léger s’appuyant sur le backendChloros
(un serveur local sur `127.0.0.1:5000`) — la plupart des commandes lancent automatiquement le backend, de sorte qu’un seul appel à `chloros-cli process …` suffit pour un script.

Elle fonctionne sous **Windows
10/11 (x64)**et**Linux
(x86_64, et NVIDIA Jetson arm64 sur JetPack 6)**, dans n’importe quel terminal, sans interface graphique requise. Vérifiez votre installation avec :

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Aperçu des familles de commandes :

* **Traitement et compte** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 langues — voir [Langues prises en charge](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (uniquement pourLinux
/Jetson)
* **Matériel en temps réel** — `lattice` (contrôle de caméra LATTICE, plus de 45 sous-commandes), `daq pool-*` (capteurs de lumière DAQ), `time-sync` (PTP)
* **Automatisation** — `project` (exécution en mode sans interface graphique d’un projetChloros
enregistré, y compris les recettes de capture YAML)

Options globales à connaître : `--port N` (port du backend, par défaut `5000`), `-v/--verbose`, `--restart` (redémarrage forcé du backend), `--backend-exe PATH`. Consultez la [RéférenceCLI
](reference/cli-reference.md) pour la liste complète.

***

## Installation

L’CLI
**est inclus dans le programme d’installation d’Chloros** sur toutes les plateformes — il n’existe pas de téléchargement séparé d’CLI
. Téléchargez le programme d’installation depuis la page [Téléchargement](download.md).

###Windows


Le programme d’installation place l’CLI
à l’emplacement suivant :

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

et ajoute ce dossier à votre système `PATH` — **ouvrez un nouveau terminal**après l&#x27;installation afin que le fichier `PATH` mis à jour soit pris en compte. Le programme d’installation place également des scripts de lancement (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) dans le répertoire racine de l’installation, ainsi qu’un**raccourci dans le menu Démarrer (ChlorosCLI
)** raccourci dans le menu Démarrer, qui ouvrent chacun un terminal avec `chloros-cli` prêt à l&#x27;emploi.

###Linux


Installez la version `.deb` adaptée à votre architecture :

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Cela installe `chloros-cli` à `/usr/bin/chloros-cli` (déjà sur `PATH`) et le backend vers `/usr/lib/chloros/chloros-backend`, ainsi que le runtime ArenaSDK
nécessaire aux caméras LATTICE. Voir [Installation deLinux
](linux/linux-installation.md) pour plus de détails.

### Vérification

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Connexion et licence

CLI
(etPython
SDK
) : l’accès nécessite un **forfait payantChloros
+**— tous les forfaits payants l’incluent ; le forfait gratuit ne le permet pas. Cette restriction est appliquée**côté serveur** par le backend, et non par le binaireCLI
: un appel effectué sans connexion est rejeté avec le code `401 AUTH_REQUIRED`, et un appel effectué après connexion sur la version gratuite par le code d’erreur `403 PLAN_UPGRADE_REQUIRED`, qu’il provienne de `chloros-cli`, de l’SDK
ou d’un clientHTTP
développé sur mesure. Mise à niveau à l&#x27;adresse [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Connectez-vous **une seule fois par machine** :

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Mots de passe contenant des caractères spéciaux**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` est altéré par le shell ; l’CLI
le détecte lors d’un code d’erreur 401 et effectue automatiquement une nouvelle tentative, mais l’utilisation de guillemets simples permet d’éviter complètement ce problème).
{% endhint %}

La session est mise en cache dans `~/.chloros/user_session.json` et continue de fonctionner hors ligne pendant la période de grâce du forfait (30 jours pour les forfaits mensuels, jusqu’à expiration pour les forfaits annuels). `chloros-cli status` fonctionne même sans forfait payant, ce qui permet de toujours voir la raison d’un refus.

{% hint style="danger" %}
**Vous planifiez une tâche en mode headless ? Connectez-vous d’abord.**Une commande de création de backend (`process`, `status`, `export-status`, …) exécutée**sans session mise en cache**ne génère pas de message d’échec immédiat : elle bascule vers une invite interactive `Email:` / `Password:` sur stdin. Une tâche cron ou une étape d’intégration continue (CI) exécutée en mode silencieux**restera donc bloquée en attendant une entrée**. Exécutez `chloros-cli login EMAIL 'PASSWORD'` une fois sur la machine avant de planifier quoi que ce soit.
{% endhint %}

***

## Votre première exécution de traitement

Pointez `process` vers un dossier de captures — il détecte automatiquementSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng`, ou un mélange des deux :

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Les flux de progression s’affichent en temps réel pour chaque thread du pipeline (détection, analyse, traitement, exportation), et une exécution réussie se termine par un rapport indiquant le nombre de produits d’images générés (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Emplacement des fichiers de sortie

`process` enregistre les fichiers dans un **dossier de projet**, et non dans votre dossier d&#x27;entrée :

* En l’absence de `-o` : le projet est créé dans votre dossier de projet par défaut (partagé avec l’interface graphique ; gérez-le avec `get-project-folder` / `set-project-folder`, solution de secours `~/Chloros Projects`), nommé par `-n/--project-name` ou un horodatage (`YYYYMMDD_HHMMSS`) lorsqu&#x27;il est omis.
* Avec `-o PATH` : ce dossier **est** le dossier du projet. S’il contient déjà un fichier `project.json`, un fichier frère portant le suffixe `_1`/`_2`… est créé au lieu de le remplacer.

Au sein du projet, les produits sont regroupés **par appareil photo, puis par format de fichier** :

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Le dossier de l’appareil photo est `LATT-<sensor>-<lens>-F<filter>` pour LATTICE (correspondant à l’EXIF de la capture `Model`) et `<model>_<filter>` (par exemple `Survey3N_RGN`) pourSurvey3
. Le dossier de format suit `--format` : `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` pour `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Chaque produit exporté conserve le nom du fichier SOURCE.**Une exportation Radiance de `capture_..._raw.tif` s’appelle toujours `capture_..._raw.tif` — elle se trouve simplement dans `tiff32/Radiance_Images/`.**C’est le dossier qui identifie le produit, pas le nom de fichier** ; utilisez donc un filtre générique pour le répertoire, et non pour le suffixe `*radiance*`.
{% endhint %}

### Les options que vous utiliserez concrètement

| Option | Par défaut | Fonction |
| --- | --- | --- |
| `-o, --output PATH` | dossier de projet par défaut | Emplacement du dossier de projet (voir ci-dessus). |
| `-n, --project-name NAME` | horodatage | Nom du projet. |
| `--format FMT` | `TIFF (16-bit)` | L&#x27;un des suivants : `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)` ou `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | aucun | Indices de végétation à exporter (voir [Indices de végétation](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = débayérisation neuronale, plus lente, qualité optimale (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | activé | Correction de la vignette. |
| `--reflectance / --no-reflectance` | activé | Étalonnage de la réflectance ; pour LATTICE, cela correspond également à l&#x27;activation/désactivation du produit de réflectance. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forcer le point d&#x27;entrée du pipeline pour les fichiers TIFF LATTICE. |

Pour tout le reste — réglage de la détection de cibles, PPK, repères d’exposition, indicateurs d’alignement de matrices —, consultez la [section `process` de la référence «CLI
»](reference/cli-reference.md).

***

## Choix des éléments à exporter (produits LATTICE)

Le traitement LATTICE se répartit sur **tous les produits concernés en un seul passage**. Les quatre commutateurs par produit sont tous**activés par défaut** ; utilisez le formulaire `--no-` pour en désactiver un :

| Commutateur | Produit |
| --- | --- |
| `--debayered` | Démosaïquage linéaire → `Debayered_Images/` |
| `--preview` | Aperçu à l’écran (balance des blancs + gamma ; étirement en fausses couleurs pour le multispectral) → `Preview_Images/` |
| `--radiance` | radiance en float32, W/m²/sr/nm → `Radiance_Images/` (toujours `tiff32/`) |
| `--reflectance` | uint16 réflectance, compatible Pix4D → `Reflectance_Calibrated_Images/` |

RGB
Les caméras « master » n’émettent jamais que des données débayérisées + aperçu — la radiance/réflectance par bande n’a pas de sens pour un capteur à large bande, ces commutateurs n’ont donc aucun effet sur elles.Survey3
`.raw` ignore les commutateurs et suit le chemin standard de réflectance/cible.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (par défaut `auto`) sélectionne la référence de réflectance : `auto` crée une [cible d’étalonnage](calibration-targets.md) intégrée à l’image et conforme aux critères de contrôle qualité comme référence absolue et se rabat sur la division descendante du capteur de lumière DAQ (ρ = π·L/E) en l’absence de cible ; `target` est strict (pas de substitution DAQ) ; `daq` fait autorité sur le DAQ. Des balayages de cibles mesurés par unité peuvent être fournis avec `--target-reflectance-dir`.

{% hint style="info" %}
**Lecture des pixels de réflectance :**la valeur DN correspondant à ρ = 1,0 est**par source** — Les fichiers LATTICE intègrent la balise `Chloros:PixelScale=32768` dans le fichier XMP ; les fichiers «Survey3
» utilisent la valeur 65535 (et ne comportent pas de balises `Chloros:*`). Lisez la balise et divisez par cette valeur plutôt que de supposer une constante. Vous trouverez plus de détails et le seul cas limite délibéré sans échelle dans la [RéférenceCLI
](reference/cli-reference.md).
{% endhint %}

**Le traitement commence toujours à partir de `raw`.** Les produits dérivés (exportations débayérisées, de radiance ou de réflectance) ne sont jamais réintroduits dans le pipeline — les réimporter et les traiter reviendrait à appliquer deux fois les calculs d’étalonnage ; c’est pourquoiChloros
les ignore et l’indique clairement. `--input-level` constitue une échappatoire prévue pour les cas où vous avez réellement besoin de forcer un point d’entrée.

***

## En cas d’échec d’une exécution

À partir de la version 1.2.0, `process` signale clairement l’échec au lieu de « réussir » sans rien afficher :

* Une exécution qui **a demandé des produits mais n&#x27;en a écrit aucun**— uniquement `project.json` et `calibration_data.json` — affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie différent de zéro**, ce qui permet aux scripts de le détecter. Causes habituelles : le dossier d’entrée n’a pas été reconnu comme une capture (vérifiez la configuration et `--input-level`), ou tous les produits demandés étaient inapplicables pour ces caméras (par exemple, demande de radiance/réflectance à partir de caméras uniquement de type «RGB
»).
* Une **exécution délibérée avec uniquement les métadonnées** (tous les produits désactivés, pas de `--indices`) est tout de même considérée comme réussie — une sortie d&#x27;image vide est le résultat attendu dans ce cas.
* Relancez l’opération avec `--verbose` et consultez le journal du backend pour repérer les lignes `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, qui expliquent les sauts par caméra.

Codes de sortie : `0` : succès · `1` : échec générique · `2` : erreur d’argument · `130` : interruption par Ctrl+C.

***

## Indices de végétation

Lancez `--indices` avec un ou plusieurs noms de préréglages ; chaque indice est placé dans son propre dossier `<INDEX>_Index_Images/` :

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

Les 22 noms prédéfinis acceptés par `process --indices` :

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Il existe trois listes d’index — ne les confondez pas.**Le menu déroulant « Paramètres du projet » de l’interface graphique comporte 27 formules (ajoute `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — ces cinq-là sont réservées à l’interface graphique et ne sont**pas** valides pour `--indices`). La commande `lattice index --preset` (en temps réel/hors ligne) utilise sa propre liste distincte de 22 préréglages. Les formules et les calculs de bandes sont documentés dans [Formules d’indices multispectraux](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Capteurs de lumière DAQ : un aperçu rapide

La famille `daq pool-*` pilote les capteurs spectraux DAQ d’MAPIR
(DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) via le pool persistant du backend — l’interface graphique,CLI
etSDK
partagent tous un même descripteur actif. **`pool-*` est le chemin DAQ pris en charge dans l’CLI
fournie** ; les autres sous-commandes `daq` auxquelles vous pourriez voir référence sont une surface interne àMAPIR
, réservée à la source, et se terminent par une erreur explicite vous redirigeant vers `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` sans `--duration` s&#x27;exécute jusqu&#x27;à `pool-record --stop` ; le répertoire de sortie par défaut est `~/Documents/DAQ Live View/` **sur la machine du backend**. Le profil de correction de la capacité est choisi au moment de la connexion (`--cap-id`, valeur par défaut du backend : `sunshine_cosine`) et peut être modifié à la volée à l&#x27;aide de `pool-set-cap` — les profils de correction de cap et la plage d&#x27;étalonnage du capteur sont abordés dans les chapitres consacrés à l&#x27;acquisition de données (DAQ) de ce manuel.

{% hint style="warning" %}
**DAQ-E sur un hôte à plusieurs cartes réseau :** la première détection automatique de `pool-connect --eth` après le démarrage peut échouer même si le capteur est en bon état. `--eth-host <ip-or-hostname>` est la version fiable — utilisez-la chaque fois que la détection échoue.
{% endhint %}

***

## Caméras LATTICE, PTP et automatisation de projets

La famille `lattice` (plus de 45 sous-commandes) couvre l’ensemble du fonctionnement des caméras LATTICE : détection, captures ponctuelles, matrices synchronisées persistantes avec le flux de connexion « smart-prep » de l’interface graphique, prévisualisation en direct dans le navigateur, alignement, calculs d’index et diagnostics de la carte réseau de l’hôte. Un aperçu :

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Parallèlement : `chloros-cli time-sync` fournit des rapports sur le « grand maître » PTP exécuté par l’hôteChloros
(les caméras LATTICE et les capteurs DAQ-E s’y synchronisent en tant qu’esclaves pour l’horodatage inter-appareils), et `chloros-cli project` ouvre un projetChloros
enregistré et pilote ses caméras, ses matrices et ses capteurs en mode headless — y compris les recettes de capture YAML scriptées.

Ces trois familles (`lattice`, `project`, `daq pool-*`) sont également les seules à prendre en charge `CHLOROS_BACKEND_URL` pour piloter un backend **à distance** ; les commandes principales ciblent toujours la machine locale.

Des guides pas à pas complets sont disponibles dans les chapitres consacrés à LATTICE de ce manuel ; chaque indicateur est répertorié dans la [RéférenceCLI
](reference/cli-reference.md).

***

## Dépannage : les 5 principaux problèmes

| Symptôme | Solution |
| --- | --- |
| `Login required`, ou une tâche planifiée se bloque à l’invite `Email:` | Exécutez `chloros-cli login EMAIL 'PASSWORD'` une fois sur cette machine — les commandes sans session mise en cache s’exécutent de manière interactive au lieu d’échouer immédiatement. |
| `backend unreachable` | Lancez l’application de bureauChloros
ou exécutez directement le binaire backend (`chloros-backend`). Si vous pointez `lattice`/`project`/`daq pool-*` vers un backend distant, vérifiez `CHLOROS_BACKEND_URL`. |
| Connexion au tableau bloquée : `FRAMES WILL DROP` / `Reduce ROI to enable` | Réinitialisation par défaut de l’anneau de réception de la carte réseau de l’hôte — la cause n° 1 pour laquelle un système qui fonctionnait auparavant refuse de se connecter, généralement après une mise à jour du pilote de la carte réseau. Exécutez la commande `chloros-cli lattice network --fix` depuis un terminal **avec des privilèges élevés** (ou définissez `ReceiveBufferLen=256`, `PendingReceives=64`) ; consultez la section *Configuration et réglage de la carte réseau de l’hôte* du guide de référence. |
| La sous-commande `daq` renvoie le message : « nécessite le package daq complet… » | Ce comportement est normal sur les versions livrées — la version compiléeCLI
ne fournit que la famille de commandes `daq pool-*`, qui couvre la connexion, le flux, l&#x27;enregistrement et la sélection des capteurs. Utilisez `pool-*` (ou `chloros_sdk.connect_daq_sensor()` disponible surPython
). |
| Jetson affiche un avertissement concernant la mémoire virtuelle avant l’ouverture de dossiers volumineux | Ajoutez une mémoire virtuelle sur fichier — l’CLI
affiche les commandes `fallocate`/`swapon` exactes à exécuter. |

***

## Obtenir de l’aide

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Chaque indicateur, chaque sous-commande :** [CLI
Référence](reference/cli-reference.md)
* **ÉquivalentPython
:** [Python
SDK
](api-python-sdk.md) et la [SDK
Référence](reference/sdk-reference.md)
* **Assistance :** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
