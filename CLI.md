# CLI : Ligne de commande

> **Référence complète :**[Référence CLI](reference/cli-reference.md) répertorie**tous les indicateurs de chaque sous-commande** et est optimisée pour les assistants IA — collez son URL dans votre assistant et demandez une commande fonctionnelle : `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Astuce pour les outils d'IA :** n'importe quelle page de ce manuel est disponible au format Markdown brut en ajoutant `.md` à son URL (par exemple `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), et `https://mapir.gitbook.io/chloros/llms.txt` indexe l’intégralité du manuel pour une utilisation par les modèles de langage de grande envergure (LLM).

> **Vous souhaitez que votre assistant exécute Chloros lui-même ?** `chloros-mcp`, un serveur MCP installé avec le Python SDK, fonctionne avec Claude Desktop, Claude Code, Cursor et d’autres clients MCP : voir [Utilisation de Chloros avec des assistants IA](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Qu'est-ce que l'CLI ?

`chloros-cli` est l'interface en ligne de commande du même moteur de traitement que celui utilisé par l'application de bureau Chloros. Il s'agit d'un client HTTP léger connecté au backend Chloros (un serveur local sur `127.0.0.1:5000`) — la plupart des commandes lancent automatiquement le backend, de sorte qu'un simple appel `chloros-cli process …` suffit à un script. Le backend reste alors en cours d’exécution pour la commande suivante : il s’arrête sur `chloros-cli logout` (à moins que l’application ne soit ouverte ou qu’une exécution ou une compilation soit en cours), sur `chloros-cli backend stop`, ou de lui-même après 30 minutes d’inactivité (absence de client, d’exécution ou d’appareil connecté). `chloros-cli backend status` l’affiche.

Il fonctionne sous **Windows 10/11 (x64)**et**Linux (x86_64, et NVIDIA Jetson arm64 sur JetPack 6)**, dans n’importe quel terminal, sans interface graphique requise. Vérifiez votre installation avec :

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Aperçu des familles de commandes :

* **Traitement et compte** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 langues — voir [Langues prises en charge](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Linux/Jetson uniquement)
* **Matériel en service** — `lattice` (contrôle de caméra LATTICE, plus de 45 sous-commandes), `daq pool-*` (capteurs de lumière DAQ), `time-sync` (PTP)
* **Automatisation** — `project` (exécuter un projet Chloros enregistré en mode sans interface graphique, y compris les recettes de capture YAML)

Options globales à connaître : `--port N` (port du backend, par défaut `5000`), `-v/--verbose`, `--restart` (redémarrage forcé du backend), `--backend-exe PATH`. Consultez la [Référence de l'interface CLI](reference/cli-reference.md) pour la liste complète.

***

## Installation

L'CLI **est fournie avec le programme d'installation de Chloros** sur toutes les plateformes — il n'existe pas de téléchargement séparé de l'CLI. Téléchargez le programme d'installation depuis la page [Téléchargement](download.md).

### Windows


Le programme d’installation place la CLI à l’emplacement suivant :

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

et ajoute ce dossier une seule fois à votre système `PATH` (une mise à jour supprime également les entrées en double laissées par les programmes d’installation précédents) — **ouvrez un nouveau terminal**après l’installation afin que le fichier `PATH` mis à jour soit pris en compte. Le programme d’installation place également des scripts de lancement (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) à la racine de l’installation, ainsi qu’un**raccourci Chloros CLI** dans le menu Démarrer, qui ouvrent chacun un terminal avec `chloros-cli` prêt à l’emploi.

### Linux


Installez `.deb` pour votre architecture à l’aide de `apt` (et non `dpkg -i`, qui ignore les prérequis recommandés pour le SDK Python de Python) :

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Cela installe `chloros-cli` à `/usr/bin/chloros-cli` (déjà présents dans `PATH`), le backend et les programmes CLI décompressés dans `/usr/lib/chloros/backend/` et `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` est un lanceur qui démarre le backend), le runtime Arena SDK dont le backend a besoin pour les caméras LATTICE, et le Python SDK pour le système `python3`. Voir [Installation sous Linux](linux/linux-installation.md) pour plus de détails.

### Vérification

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Connexion et licence

L'accès à la CLI (et au SDK Python Python) nécessite un **forfait Chloros+ payant**— tous les forfaits payants l'incluent ; le forfait gratuit ne le propose pas. Cette restriction est appliquée**côté serveur** par le backend, et non par le binaire de la CLI : un appel effectué sans être connecté est rejeté avec le code `401 AUTH_REQUIRED`, et un appel effectué après connexion sur le forfait gratuit est rejeté avec le code `403 PLAN_UPGRADE_REQUIRED`, qu’il provienne de `chloros-cli`, du SDK ou d’un client HTTP développé manuellement. Effectuez la mise à niveau à l’adresse [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Connectez-vous **une seule fois par machine** :

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Mots de passe contenant des caractères spéciaux**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` est altéré par le shell ; l’interface CLI détecte ce problème lors d’un code d’erreur 401 et effectue automatiquement une nouvelle tentative, mais l’utilisation de guillemets simples permet d’éviter complètement ce problème).
{% endhint %}

La session est mise en cache dans `~/.chloros/user_session.json` et continue de fonctionner hors ligne pendant la période de grâce du forfait (30 jours pour les forfaits mensuels, jusqu’à expiration pour les forfaits annuels). `chloros-cli status` fonctionne même sans forfait payant, ce qui permet de toujours voir la raison du refus.

{% hint style="danger" %}
**Vous planifiez une tâche en mode headless ? Connectez-vous d’abord.**Une commande de lancement de backend (`process`, `status`, `export-status`, …) exécutée**sans session mise en cache**ne signale pas rapidement l’échec — elle bascule vers une invite interactive `Email:` / `Password:` sur stdin. Une tâche cron ou une étape d’intégration continue (CI) exécutée en mode silencieux**restera donc bloquée en attendant une entrée**. Exécutez `chloros-cli login EMAIL 'PASSWORD'` une fois sur la machine avant de planifier quoi que ce soit.
{% endhint %}

***

## Votre première exécution de traitement

Pointez `process` vers un dossier de captures — il détecte automatiquement Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng`, ou un mélange des deux :

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Les flux de progression s’affichent en temps réel pour chaque thread du pipeline (Détection, Analyse, Traitement, Exportation), et une exécution réussie se termine par l’affichage du nombre de produits d’image générés : `Image products written: N`, suivi de `(plus M clamp masks)` lorsque l'exécution a également généré des [masques de clamp](output-image-formats.md#clamp-masks) (ceux-ci ne sont pas comptabilisés comme des produits).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Si un backend Chloros est déjà en cours d’exécution (par exemple celui de l’application de bureau), `process` l’utilise et y ouvre son projet. Il refuse toute opération tant que l’application est ouverte sur un projet qu’elle a elle-même ouvert : il faut soit lancer le processus depuis l’application, soit fermer ce projet. Si une exécution précédente est encore en cours, il attend jusqu’à 15 secondes avant de refuser. `--restart` force le démarrage d’un nouveau backend. Sous Linux, un backend exécuté par un service systemd n’est jamais redémarré : `process` l’utilise tel quel, et là où il le redémarrerait normalement (`--restart`, ou une importation n’ayant rien trouvé), il affiche `sudo systemctl restart <unit>` et se ferme `1`.

### Où sont enregistrées les sorties

`process` écrit dans un **dossier de projet**, et non dans votre dossier d’entrée :

* En l’absence de `-o` : le projet est créé dans votre dossier de projet par défaut (partagé avec l’interface graphique ; gérez-le avec `get-project-folder` / `set-project-folder` ; si aucun n’est défini, `Documents/MAPIR/Chloros Projects` dans votre dossier utilisateur), nommé par `-n/--project-name` ou un horodatage (`YYYYMMDD_HHMMSS`) lorsqu’il est omis.
* Avec `-o PATH` : ce dossier **est** le dossier du projet. S’il contient déjà un fichier `project.json`, un fichier `_1`/`_2`… est créé à la place, au lieu de le remplacer. Un chemin relatif `PATH` est calculé par rapport au dossier à partir duquel vous exécutez `chloros-cli`.

{% hint style="info" %}
**Chemins relatifs (modifiés dans la version 1.3.0).** Chaque chemin d’accès ouvert par le backend — `-o`/`--output` lors de n’importe quelle commande (y compris la commande par défaut `output` des commandes de capture), `--input`, `--burst-dir`, `--profile`, `--save` et `--mask` — est résolu par rapport au dossier à partir duquel vous exécutez `chloros-cli` ; ainsi, `-o captures/` écrit dans `<current folder>/captures/`. Dans la version 1.2.0, le backend les résolvait dans son propre dossier d’installation, ce qui entraînait un échec avec le message « Accès refusé ». Avec un backend distant (`CHLOROS_BACKEND_URL`), les chemins d’accès sont envoyés tels qu’ils ont été saisis et sont résolus sur cette machine.
{% endhint %}

{% hint style="warning" %}
**Windows : veillez à ce que le dossier de projet soit court, sauf si les chemins d’accès longs sont activés.** Lorsque les chemins d’accès longs sont désactivés sous Windows (configuration par défaut de Windows), aucun chemin d’accès ne doit dépasser 260 caractères, et les produits doivent se trouver jusqu’à environ 130 caractères en dessous du dossier de projet. `process` vérifie cela avant de démarrer : un dossier de projet trop profond est refusé avec le message **« Le dossier de projet est trop profond pour cet ordinateur »**, le dossier de projet le plus long compatible est indiqué, ainsi que la procédure pour activer les chemins d’accès longs — aucune opération n’est effectuée. Utilisez une version plus courte de `-o` (ou `set-project-folder`), ou activez les chemins d’accès longs de Windows (option **Activer les chemins d’accès longs de Windows** du programme d’installation, cochée par défaut) et redémarrez Chloros.
{% endhint %}

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

Le dossier de la caméra est `LATT-<sensor>-<lens>-F<filter>` pour LATTICE (correspondant à l’EXIF de la capture `Model`) et `<model>_<filter>` (par exemple `Survey3N_RGN`) pour Survey3. Le dossier de format suit `--format` : `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` pour `TIFF (32-bit, Percent)`.

Pour une caméra multispectrale M3C, un dossier `clamp_masks/` peut également apparaître dans un dossier de format, à côté des dossiers de produits (par exemple `tiff16/clamp_masks/Reflectance_Calibrated_Images/` et `tiff32/clamp_masks/Radiance_Images/`). Il contient les masques par pixel des produits de radiance et de réflectance pour lesquels le démixage de bande a limité une valeur négative à zéro — voir [Masques de limitation](output-image-formats.md#clamp-masks). Aucun dossier de produit ne contient jamais de masque.

{% hint style="info" %}
**Chaque produit exporté conserve le nom du fichier SOURCE.**Une exportation de radiance de `capture_..._raw.tif` s’appelle toujours `capture_..._raw.tif` — elle se trouve simplement dans `tiff32/Radiance_Images/`.**C'est le dossier qui identifie le produit, pas le nom de fichier** ; utilisez donc un motif générique pour le répertoire, et non pour le suffixe `*radiance*`.
{% endhint %}

### Les options que vous utiliserez concrètement

| Option | Par défaut | Fonction |
| --- | --- | --- |
| `-o, --output PATH` | dossier de projet par défaut | Emplacement du dossier de projet (voir ci-dessus). |
| `-n, --project-name NAME` | horodatage | Nom du projet. |
| `--format FMT` | `TIFF (16-bit)` | L'une des valeurs suivantes : `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)` ou `JPG (8-bit)`. Chloros ne procède pas à une conversion vers un format supérieur : sur toutes les images 8 bits, un format plus profond est enregistré sous la forme `TIFF (8-bit)`, et cela est indiqué une seule fois. |
| `--indices NAME [NAME ...]` | aucun | Indices de végétation à exporter (voir [Indices de végétation](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = débayérisation neuronale, plus lente, qualité maximale (Chloros+, nécessite un GPU CUDA : en l’absence de celui-ci, le traitement utilise `standard` ; refusé pour les captures LATTICE, qui utilisent toujours `standard`). |
| `--vignette / --no-vignette` | activé | Correction de la vignettage. |
| `--reflectance / --no-reflectance` | activé | Étalonnage de la réflectance ; pour LATTICE, cela correspond également à l'activation/désactivation du produit de réflectance. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forcer le point d'entrée du pipeline pour les fichiers TIFF LATTICE. |

Pour tout le reste — réglage de la détection de cible, PPK, repères d’exposition, indicateurs d’alignement de matrice —, consultez la [section `process` de la référence CLI](reference/cli-reference.md).

***

## Choix des éléments à exporter (produits LATTICE)

Le traitement LATTICE se répartit sur **tous les produits concernés en un seul passage**. Les quatre commutateurs par produit sont tous**activés par défaut** ; utilisez le formulaire `--no-` pour en désactiver un :

| Commutateur | Produit |
| --- | --- |
| `--debayered` | Démosaïquage linéaire → `Debayered_Images/` : suppression du décalage sombre du pack d’étalonnage, puis dématriçage sensible aux contours, sans correction de linéarité, champ plat ni démixage — le même fichier qu’une capture enregistre à son niveau `debayered`. Conservé dans l’orientation native du capteur (enregistrée dans XMP). |
| `--preview` | Aperçu à l'écran (balance des blancs + gamma ; étirement en fausses couleurs pour le multispectral) → `Preview_Images/` |
| `--radiance` | radiance float32, W/m²/sr/nm → `Radiance_Images/` (toujours `tiff32/`) |
| `--reflectance` | uint16 réflectance, compatible Pix4D → `Reflectance_Calibrated_Images/` |

Les caméras principales RGB n émettent jamais que des données débayérisées + aperçu — la radiance/réflectance par bande n’a pas de sens pour un capteur à large bande, ces commutateurs sont donc sans effet pour elles. Survey3 `.raw` ignore les commutateurs et suit le chemin standard de réflectance/cible.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (par défaut `auto`) sélectionne la référence de réflectance : `auto` fait d’une [cible d’étalonnage](calibration-targets.md) présente dans le cadre et ayant passé le contrôle qualité la référence absolue, et se rabat sur la division descendante du capteur de lumière DAQ (ρ = π·L/E) en l’absence de cible (une cible ayant passé le contrôle qualité avec un indicateur entre en concurrence avec le DAQ sur le score mesuré) ; `target` est strict (pas de substitution par le DAQ) ; `daq` fait autorité sur le DAQ. Des balayages de cibles mesurés par unité peuvent être fournis avec `--target-reflectance-dir`.

{% hint style="info" %}
**Lecture des pixels de réflectance :**la valeur DN correspondant à ρ = 1,0 est**par source** — les fichiers LATTICE intègrent la balise `Chloros:PixelScale=32768` dans le fichier XMP ; les fichiers Survey3 utilisent la valeur 65535 et ne comportent pas de balise `Chloros:PixelScale` (ils comportent toutefois d’autres balises de traitement `Chloros:*`). Lisez la balise et divisez par sa valeur plutôt que de supposer une constante. Vous trouverez plus de détails ainsi que le seul cas limite délibéré sans échelle dans la [Référence CLI](reference/cli-reference.md).
{% endhint %}

**Le traitement commence toujours à partir de `raw`.** Les produits dérivés (exportations débayérisées, de radiance ou de réflectance) ne sont jamais réintroduits dans le pipeline — les réimporter et les traiter reviendrait à appliquer deux fois les calculs d’étalonnage ; c’est pourquoi Chloros les ignore et le signale. `--input-level` est la porte de secours prévue pour les cas où vous avez réellement besoin de forcer un point d’entrée.

***

## Lorsqu’une exécution échoue

À partir de la version 1.2.0, `process` signale clairement son échec au lieu de « réussir » sans rien afficher :

* Une exécution qui **a demandé des produits mais n’en a écrit aucun**— uniquement `project.json` et `calibration_data.json` — affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie différent de zéro**, ce qui permet aux scripts de le détecter. Causes habituelles : le dossier d’entrée n’a pas été reconnu comme une capture (vérifiez la configuration et `--input-level`), ou tous les produits demandés étaient inapplicables pour ces caméras (par exemple, demander la radiance/réflectance à partir de caméras RGB uniquement).
* Une **exécution délibérée avec uniquement les métadonnées** (tous les produits désactivés, pas de `--indices`) est tout de même considérée comme réussie — une sortie d’image vide est le résultat attendu dans ce cas.
* Relancez l’exécution avec `--verbose` et vérifiez le journal du backend à la recherche des lignes `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, qui expliquent les omissions par caméra.
* Sous Windows, lorsque les chemins d’accès longs sont désactivés, un dossier de projet dont la profondeur dépasse la limite de 260 caractères est refusé **avant** le début du traitement ; le dossier de projet le plus long pouvant être pris en charge est alors utilisé (voir [Où sont enregistrés les fichiers de sortie](#where-the-outputs-land)).
* Les commandes de capture (`lattice capture`, `lattice array-capture` et les autres familles de capture) répertorient tous les fichiers qu’elles n’ont pas pu écrire et se terminent par le code `1` ; les captures répétées s’arrêtent lorsque le disque est plein.

Codes de sortie : `0` succès · `1` échec générique · `2` erreur d’argument · `130` interrompu par Ctrl+C.

***

## Indices de végétation

Lancez `--indices` avec un ou plusieurs noms prédéfinis ; chaque indice est placé dans son propre dossier `<INDEX>_Index_Images/` :

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

Les 22 noms prédéfinis acceptés par `process --indices` :

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Un nom qui ne figure pas encore dans le projet est ajouté à celui-ci. Un nom qui n’est pas un préréglage, ou que les bandes des caméras ne peuvent pas générer, est ignoré et un avertissement l’indique. Si l’CLI ne parvient pas à lire la liste d’index actuelle du projet, elle laisse la liste inchangée et affiche un avertissement plutôt que de la remplacer.

{% hint style="warning" %}
**Il existe trois listes d’index — ne les confondez pas.**Le menu déroulant « Paramètres du projet » de l’interface graphique comporte 27 formules (ajoute `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — ces cinq formules sont réservées à l’interface graphique et ne sont**pas** valables pour `--indices`). La commande `lattice index --preset` (en temps réel/hors ligne) utilise sa propre liste distincte de 22 préréglages. Les formules et les calculs de bandes sont documentés dans [Formules d'indices multispectraux](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Capteurs de lumière DAQ : un aperçu rapide

La famille `daq pool-*` pilote les capteurs spectraux DAQ MAPIR (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) via le pool persistant du backend — l’interface graphique, la CLI et le SDK partagent tous un même descripteur en temps réel. **`pool-*` est le chemin DAQ pris en charge dans l’CLI fournie**; les autres sous-commandes `daq` auxquelles vous pourriez voir référence sont une interface interne à MAPIR en lecture seule et se terminent par une erreur explicite vous redirigeant vers `pool-*`.

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

`pool-record` sans `--duration` s'exécute jusqu'à `pool-record --stop` ; le répertoire de sortie par défaut est `~/Documents/DAQ Live View/` **sur la machine du backend**. Le profil de correction de la capacité est choisi au moment de la connexion (`--cap-id`, valeur par défaut du backend : `sunshine_cosine`) et peut être modifié à la volée à l’aide de `pool-set-cap` — les profils de correction de la capacité et la plage d'étalonnage du capteur sont abordés dans les chapitres consacrés à l'acquisition de données (DAQ) de ce manuel.

{% hint style="warning" %}
**DAQ-E sur un hôte à plusieurs cartes réseau :** la première détection automatique `pool-connect --eth` après le démarrage peut échouer même si le capteur est en bon état. `--eth-host <ip-or-hostname>` est la version fiable — utilisez-la chaque fois que la détection échoue.
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

Parallèlement : `chloros-cli time-sync` fournit des informations sur le « grand maître » PTP exécuté par l’hôte Chloros (les caméras LATTICE et les capteurs DAQ-E lui sont asservis pour la synchronisation temporelle entre appareils), et `chloros-cli project` ouvre un projet Chloros enregistré et pilote ses caméras, ses réseaux de capteurs et ses capteurs en mode « headless » — y compris les recettes de capture YAML scriptées.

Ces trois familles (`lattice`, `project`, `daq pool-*`) sont également les seules à prendre en charge `CHLOROS_BACKEND_URL` pour piloter un backend **à distance** ; les commandes principales ciblent toujours la machine locale.

Des guides pas à pas complets sont disponibles dans les chapitres consacrés à LATTICE de ce manuel ; chaque option figure dans la [Référence CLI](reference/cli-reference.md).

***

## Dépannage : les 5 principaux problèmes

| Symptôme | Solution |
| --- | --- |
| `Login required`, ou une tâche planifiée se bloque à l’invite `Email:` | Exécutez `chloros-cli login EMAIL 'PASSWORD'` une fois sur cette machine — les commandes sans session mise en cache s’exécutent de manière interactive au lieu d’échouer immédiatement. |
| `backend unreachable` | Lancez l’application de bureau Chloros, ou démarrez le backend vous-même : `"C:\Program Files\Chloros\backend\chloros-backend.exe"` sous Windows (dans le dossier `backend` de votre dossier d’installation ; il ne se trouve pas dans `PATH`), `chloros-backend` sous Linux. `chloros-cli backend status` indique si un service est en cours d’exécution ; `lattice`/`project`/`daq pool-*` n’en lancent jamais. Si vous les pointez vers un backend distant, vérifiez `CHLOROS_BACKEND_URL`. |
| Connexion au tableau bloquée : `FRAMES WILL DROP` / `Reduce ROI to enable` | Réinitialisation par défaut de l'anneau de réception de la carte réseau de l'hôte — la cause n° 1 pour laquelle un équipement qui fonctionnait auparavant refuse de se connecter, généralement après une mise à jour du pilote de la carte réseau. Exécutez `chloros-cli lattice network --fix` depuis un terminal **avec des privilèges élevés** (ou définissez `ReceiveBufferLen=256`, `PendingReceives=64`) ; consultez la section *Configuration et réglage de la carte réseau de l’hôte* du guide de référence. |
| La sous-commande `daq` se ferme : « nécessite le package DAQ complet… » | Utilisez les commandes `daq pool-*`, qui pilotent le même capteur via le backend : connexion, transmission en continu, enregistrement, sélection de la capacité, et pour un DAQ-E, ses paramètres réseau, un redémarrage et les mises à jour du micrologiciel (ou `chloros_sdk.connect_daq_sensor()` depuis Python). |
| Jetson affiche un avertissement concernant la mémoire virtuelle avant l’ouverture de dossiers volumineux | Ajoutez une mémoire virtuelle basée sur un fichier — l’CLI affiche les commandes exactes `fallocate`/`swapon` à exécuter. |

***

## Obtenir de l’aide

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Tous les indicateurs, toutes les sous-commandes :** [Référence de la CLI](reference/cli-reference.md)
* **Équivalent Python :** [Python SDK](api-python-sdk.md) et la [Référence du SDK](reference/sdk-reference.md)
* **Assistance :** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
