# Référence de la commande « Chloros » CLI

**Version :**

1.2.0**Date de création :**29/07/2026 19:19 ·**Date de révision :** 30/08/2026**Public visé :** Optimisé pour une utilisation par les modèles de langage de grande envergure (LLM) ; lisible par l’homme.**Portée :** Toutes les sous-commandes de `chloros-cli` destinées aux utilisateurs, avec leurs options et des exemples à copier-coller.

Ce document constitue la référence complète de l’outil en ligne de commande `chloros-cli` fourni avec MAPIR Chloros. Il est volontairement exhaustif afin qu’un LLM (ou un humain) puisse composer n’importe quel workflow pris en charge à partir des listes ci-dessous sans avoir à inspecter le code source.

Si vous souhaitez uniquement consulter les points essentiels, rendez-vous à :
- [Guide de démarrage rapide en cinq minutes](#five-minute-quickstart)
- [Workflow de première connexion de la caméra LATTICE](#lattice-camera-first-connect-workflow)
- [Workflow de première connexion d&#x27;un capteur DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modes de capture, enregistreurs et retraitement hors ligne](#capture-modes-recorders--offline-reprocess)

---

## Conventions

- Toutes les commandes sont préfixées par `chloros-cli`. Sur Windows, le binaire est `chloros-cli.exe` ; sur Linux /Jetson, il s&#x27;agit de `chloros-cli`.
- Les arguments facultatifs sont indiqués sous la forme `--flag`. Les arguments positionnels obligatoires sont indiqués sans crochets.
- Lorsqu’une valeur par défaut est fournie, le fait d’omettre l’indicateur utilise cette valeur.
- L’CLI est un client d’HTTPs léger s’appuyant sur le backend Chloros (serveur Flask sur `127.0.0.1:5000`). Le backend est lancé automatiquement par la plupart des commandes. `CHLOROS_BACKEND_URL=<url>` redirige les familles de commandes **`lattice`**,**`project`**et**`daq pool-*`** vers un backend distant — les commandes principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) bloquent délibérément `http://127.0.0.1:<port>` et l’ignorent (la valeur littérale IPv4 évite l’Windows’ `localhost`→`::1`, ce qui entraîne une pénalité d’environ 2 secondes par requête). Voir [Variables d’environnement](#environment-variables).
- Une connexion avec un compte Chloros+ est requise pour tous les appels vers SDK / CLI (exécutez `chloros-cli login` une fois par machine ; mis en cache dans `~/.chloros/`).
- Les exemples utilisent les chemins d&#x27;accès Linux ; sur Windows, remplacez `/home/user/...` par `C:/Users/.../...`.

---

## Synopsis de haut niveau

```
chloros-cli [global options] COMMAND [command options]
```

### Options globales

| Indicateur | Description |
| --- | --- |
| `--backend-exe PATH` | Remplacer l&#x27;exécutable du backend détecté automatiquement. |
| `--port N` | Port d&#x27;HTTPs du backend (par défaut : `5000`). |
| `-v, --verbose` | Activer le mode de sortie détaillé. |
| `--restart` | Redémarrer de force le backend (tue tout processus `backend_server.py` en cours d&#x27;exécution). |
| `--version` | Afficher la version (`Chloros CLI 1.2.0`). |
| `--help` | Afficher l&#x27;aide de haut. |

### Index des commandes

| Commande | Objectif |
| --- | --- |
| [`process`](#chloros-cli-process) | Traite de bout en bout un dossier de captures Survey3 ou LATTICE. |
| [`login`](#chloros-cli-login) | Authentifier cette machine avec un compte Chloros+. |
| [`logout`](#chloros-cli-logout) | Effacer les identifiants mis en cache. |
| [`status`](#chloros-cli-status) | Afficher l&#x27;état actuel de la licence / de l&#x27;authentification. |
| [`export-status`](#chloros-cli-export-status) | Afficher la progression en temps réel de l’exportation Thread-4 pendant l’exécution de `process`. |
| [`language`](#chloros-cli-language) | Définir ou répertorier la langue d&#x27;affichage d&#x27;CLI (38 langues prises en charge). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Dossier de projet par défaut (partagé avec l&#x27;interface graphique). |
| [`update`](#chloros-cli-update) | Recherche et installation des mises à jour d&#x27;CLIs (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostics système + tests de fonctionnement. |
| [`time-sync`](#chloros-cli-time-sync) | État / contrôle du maître PTP. |
| [`lattice`](#chloros-cli-lattice) | Contrôle et capture de la caméra LATTICE (plus de 45 sous-commandes). |
| [`daq`](#chloros-cli-daq) | Contrôle des capteurs spectraux DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Ouvrir et piloter un projet «Chloros» enregistré (caméras + DAQ). |

---

## Installation

`chloros-cli` est fourni dans le programme d’installation de bureau d’Chloros sur toutes les plateformes prises en charge — il n’y a pas de téléchargement séparé CLI. L’installation du package de la plateforme ajoute `chloros-cli` à votre `PATH` aux côtés de l’application de bureau et du binaire backend qu&#x27;il pilote.

Derniers téléchargements : [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Le programme d’installation fournit également des scripts de lancement pratiques (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) qui ouvrent unà l’CLI ; leur fonctionnement est décrit dans le [Guide de l’utilisateur d’CLI](../CLI.md) et n’est pas répété ici.

### Windows (.exe)

1. Téléchargez le programme d’installation d’Windows depuis la page de téléchargement.
2. Exécutez `Chloros-Setup-x.y.z.exe` et suivez les instructions de l’assistant. Le chemin d’installation par défaut est `C:\Program Files\Chloros\` (le fichier « CLI » est placé dans le répertoire `C:\Program Files\Chloros\cli\`, que le programme d’installation ajoute à la variable PATH).
3. Ouvrez un nouveau terminal (`cmd.exe`, PowerShell ou le terminal d’Windows) afin que le fichier `PATH` mis à jour soit pris en compte.

```powershell
chloros-cli --version
```

Le programme d’installation ajoute automatiquement `chloros-cli.exe` à votre système `PATH` et intègre le runtime Arena SDK nécessaire aux caméras LATTICE.

### Linux amd64 (.deb)

Pour Ubuntu 22.04 LTS ou plus récent / postes de travail x86_64 basés sur Debian.

> **Ubuntu 20.04 n&#x27;est pas pris en charge.** La liste des dépendances du paquet est dérivée de
> ce à quoi le backend est réellement lié, et cela inclut `libc6 (>= 2.34)` ;
> Focal fournit glibc 2.31. `apt` refuse l’installation plutôt que de laisser celle-ci échouer lors de l’
> exécution.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Le fichier .deb installe :
- `chloros-cli` vers `/usr/bin/chloros-cli`
- Le backend compilé vers `/usr/lib/chloros/chloros-backend`
- Le runtime Arena SDK (pour les caméras LATTICE)
- Les modèles de débruitage, les ensembles d’étalonnage et la configuration du canal de mise à jour

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Même structure que le fichier .deb amd64, avec une version CUDA optimisée pour Jetson Orin / Orin NX / Orin Nano.

### Authentification unique par machine

Chaque plateforme nécessite une connexion unique via Chloros+ avant que les appels SDK / CLI ne fonctionnent :

```bash
chloros-cli login user@example.com 'YourPassword'
```

Les identifiants sont mis en cache dans `~/.chloros/user_session.json`.

### Vérification de l’installation

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Un abonnement Chloros+ est requis.**L’CLI nécessite un forfait Chloros+ actif.**Copper**est le niveau d’entrée Chloros+ — tous les niveaux payants Chloros+ donnent accès à CLI / SDK ; seul le niveau gratuit**Iron** n’y donne pas accès. (Correspondance des identifiants de forfait : `0`=Iron/gratuit, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Passez à un forfait supérieur sur [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Ce seuil est appliqué par le backend, et pas seulement par l’CLI : une requête comportant les indicateurs SDK / CLI sans forfait payant est rejetée avec le code `403 PLAN_UPGRADE_REQUIRED`, qu’elle provienne de `chloros-cli`, de l’Python SDK ou d’un client HTTP développé manuellement. Un utilisateur déconnecté reçoit à la place le code d’erreur `401 AUTH_REQUIRED`. L’accès fonctionne hors ligne pendant la période de grâce du forfait (30 jours pour les forfaits mensuels, jusqu’à expiration pour les forfaits annuels) et s’interrompt à l’expiration de cette période ; `chloros-cli status` continue de fonctionner afin que la raison soit visible (il s’agit de la seule route SDK / CLI exemptée du contrôle d’accès par niveau — `GET /api/license-status`).

---

## Démarrage rapide en cinq minutes

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Traiter un dossier d’images via l’ensemble du pipeline Chloros (détection de cible → étalonnage → vignette → réflectance → exportation de l’index).

### Synopsis

```
chloros-cli process INPUT [OPTIONS]
```

### Arguments de position

| Argument | Description |
| --- | --- |
| `INPUT` | Chemin d&#x27;accès au dossier d&#x27;entrée contenant les fichiers `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ou `.dng`. |

### Options communes

| Option | Valeur par défaut | Description |
| --- | --- | --- |
| `-o, --output PATH` | un nouveau dossier horodaté sous le chemin d’accès par défaut de votre projet (`~/Chloros Projects` sauf configuration contraire) | Dossier de projet à créer ou à réutiliser. Si le dossier contient déjà un fichier `project.json`, un dossier frère `_1`/`_2` est créé au lieu de le remplacer. |
| `-n, --project-name NAME` | auto (horodatage) | Nom du projet. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utilise un débayériseur neuronal de type Chloros+ ; plus lent mais de meilleure qualité. |
| `--vignette / --no-vignette` | `--vignette` | Correction de la vignettage. |
| `--reflectance / --no-reflectance` | `--reflectance` | Étalonnage de la réflectance (utilise une mire si elle est détectée, NIST par-série pour LATTICE). Pour LATTICE multispectral, cela fait également office de commutateur de **produit** de réflectance — voir [Commutateurs d’exportation par produit](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | désactivé | Appliquer les corrections GNSS PPK à partir des fichiers sidecar. |
| `--exposure-pin-1 MODEL` | désactivé | Fixer le modèle « pin-1 » d’un dispositif à double caméra «Survey3». |
| `--exposure-pin-2 MODEL` | désactivé | Fixer le modèle « pin-2 ». |
| `--recal-interval SECONDS` | 0 | Forcer la réexécution des calculs d&#x27;étalonnage toutes les N secondes de temps de capture. |
| `--timezone-offset HOURS` | local | Remplacer le décalage horaire intégré dans les métadonnées de sortie. |
| `--format FORMAT` | `TIFF (16-bit)` | L&#x27;une des valeurs suivantes : `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | aucun | Indices de végétation (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forcer le point d’entrée du pipeline pour les fichiers TIFF LATTICE (Survey3 .raw n’est pas concerné). Également la porte de secours qui permet de traiter une capture **sans fichier raw** — voir [À quoi ressemble un dossier de captures](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | activé | Génère le produit débayérisé linéaire (`Debayered_Images`). Voir [Commutateurs d’exportation par produit](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | activé | Émet l’aperçu à l’écran (`Preview_Images`) : RGB = balance des blancs (illuminant DAQ si disponible, sinon « gray-world ») + gamma ; multispec = étirement en fausses couleurs. |
| `--radiance / --no-radiance` | activé | Émet la radiance en float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Référence pour le produit de réflectance LATTICE : `auto` = la cible dans le cadre ayant passé le contrôle qualité (QA) est la référence absolue, repli sur le rayonnement descendant DAQ (ρ = π·L/E) ; `target` = strict (pas de substitution DAQ) ; `daq` = DAQ faisant autorité. Voir [Commutateurs d’exportation par produit](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | aucun | Répertoire des balayages de réflectance **mesurés** par unité (`<serial>.csv`) ; en cas d’échec, utilisation des spectres T3/T4P en cas d’échec. |
| `--array-alignment / --no-array-alignment` | activé | Matrices LATTICE : applique l’alignement module à module enregistré dans le fichier XMP `Chloros:Alignment*` de chaque capture à chaque produit traité (débayérisation / aperçu / radiance / réflectance / indice). Aucune opération pour les images ne comportant pas ces balises. |
| `--array-alignment-crop / --no-array-alignment-crop` | recadrage | Recadre les exportations alignées sur la zone de chevauchement commune du réseau afin que tous les modules partagent une même empreinte ; `--no-…` conserve la zone de capture complète du capteur (remplissage noir en dehors de la source). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Rééchantillonnage pour la déformation due à l’alignement. `nearest` préserve les valeurs DN exactes de la source (pas de mélange inter-pixels des valeurs radiométriques). |

### Options de détection de cibles

| Indicateur | Description |
| --- | --- |
| `--min-target-size PIXELS` | Taille minimale de la cible du panneau (px) pour le détecteur. |
| `--target-clustering 0-100` | Sensibilité du regroupement. |
| `--target / --targets` | Traiter le dossier d&#x27;entrée comme contenant uniquement des panneaux-cibles (ignorer la détection des relevés). |

### Exemples

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

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

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Commutateurs d’exportation par produit (LATTICE multispectral)

Le traitement LATTICE se décompose en **tous les produits applicables en un seul passage**. Les quatre commutateurs par type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sont tous**activés par défaut** ; utilisez le formulaire `--no-<type>` pour en désactiver un. Les caméras principales RGB n’émettent jamais que des données débayérisées + aperçu (pas de radiance/réflectance par bande) ; par conséquent, les options `--radiance` et `--reflectance` n’ont aucun effet sur celles-ci. Les commutateurs sont ignorés pour les caméras Survey3 `.raw` (qui suivent la norme de réflectance/chemin cible). *(L’ancien indicateur `--radiometric-output {reflectance,radiance,sensor-response}` a été **supprimé** et remplacé par ces commutateurs ; il n’y a plus de niveau `sensor-response`.)*

| Produit | Sortie | DAQ descendante nécessaire ? |
| --- | --- | --- |
| `--debayered` | Démosaïquage linéaire (`Debayered_Images`). | Non |
| `--preview` | Aperçu à l&#x27;écran (`Preview_Images`) : RGB = WB + gamma ; multispec = étirement en fausses couleurs. | Non |
| `--radiance` | float32 W/m²/sr/nm issu de la chaîne radiométrique complète (`Radiance_Images`). | N° |
| `--reflectance` | uint16 de la réflectance ρ (`32768` = 1,0), prêt pour Pix4D. | **Oui**, sauf si une cible dans le cadre ayant passé le contrôle qualité (QA) l’ancre (voir ci-dessous). |

`--reflectance-source` sélectionne la référence de réflectance :**`auto`**(par défaut) fait d’une cible dans le cadre ayant passé le contrôle qualité la**référence absolue**— les chaînes de lignes empiriques ancrées sur la cible sont comparées sur des panneaux mis de côté et la valeur gagnante mesurée est appliquée — en revenant à la division descendante du DAQ (ρ = π·L/E) lorsqu’aucune cible n’est présente ou que le contrôle qualité échoue ;**`target`**est strict (pas de substitution par le DAQ) ;**`daq`**opte pour le comportement faisant autorité du DAQ. La géométrie de la cible (ArUco / ROI fixe / bande) provient de la configuration de cible du projet ; `--target-reflectance-dir DIR` conserve les**balayages** par unité (`<serial>.csv`) recherchés à l’aide du numéro de série ou du QR de l’unité cible, avec les spectres T3/T4P nominaux comme solution de secours.

Le chemin de réflectance DAQ résout automatiquement le **flux descendant apparié à l’horodatage**à partir d’un**`.daq`**(DAQ-U/M/E)**ou d’un fichier `.csv` natif DAQ-M**trouvé avec les images. Si un ensemble d’étalonnages par caméra ou par DAQ n’est pas mis en cache localement, le pipeline**le récupère automatiquement depuis AWS** lors de la première utilisation (nécessite une connexion Internet une seule fois ; cenregistré sous `~/.chloros/`).

#### Lecture des pixels de réflectance (Pix4D / Metashape / vos propres scripts)

La réflectance est stockée sous forme de DN entier, et **le DN correspondant à ρ = 1,0 dépend de la caméra source**:

| Source | ρ = 1,0 correspond à | Comment le déterminer |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (marge jusqu’à ρ 2,0) | La balise XMP `Chloros:PixelScale=32768` est apposée sur le fichier. |
| Survey3 | `65535` (limité à ρ 1,0) | Pas de balises XMP « `Chloros:*` » — cette absence *est* le signal. |

**Lisez la valeur `Chloros:PixelScale` et divisez par celle-ci** plutôt que de supposer une constante. La balise est définie dans le domaine uint16, elle reste donc inchangée `32768` dans tous les formats de sortie qui effectuent un redimensionnement — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` et `TIFF (32-bit, Percent)` sont tous auto-descriptifs (il faut d’abord renormaliser le type de données stocké en uint16 : ×257 à partir de 8 bits, ×65535 à partir de float).

> **Un cas ne comporte pas d’échelle, par conception.** Lorsqu’une capture provenant d’une source 8 bits (BayerRG8) est écrite au format 8 bits TIFF, le pipeline *retranche* à la plage 0..255 au lieu de procéder à un redimensionnement ; ainsi, toute valeur supérieure à ρ≈0,008 est ramenée à 255 et aucune échelle ne décrit le fichier. Chloros omet délibérément à la fois le tuple `Chloros:PixelScale` et le tuple `MicaSense:RadiometricCalibration`, et en explique la raison dans le journal. **Si la balise est absente d’un fichier de réflectance LATTICE, ne supposez pas l’existence d’une échelle — réexportez en 16 bits ou 32 bits** plutôt que de diviser des pixels qui n’ont jamais été divisibles.

#### Données EXIF conservées lors de l’exportation

`process` copie le **bloc GPS et son ExifIFD** de la capture source sur chaque produit ; ainsi, une
exportation conserve `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` et
`CameraSerialNumber` en plus du géoréférencement.

**`FocalLength` n’est pas facultatif pour la photogrammétrie.** Pix4D calcule la distance d’échantillonnage au sol (GSD) à partir de la
distance focale et de l’altitude ; en l’absence de cette balise, il utilise une échelle totalement erronée. Lors d’un
vol au-dessus d’une orangeraie comprenant 49 prises de vue, l’absence de cette balise a transformé un site de 411 m × 160 m en une reconstruction
de 47,8 km × 13 km — une ortho de 455 MP composée en grande partie de « nodata », ce qui a ensuite été interprété comme un problème de mosaïquage et
un problème de BigTIFF avant que quiconque ne vérifie la GSD. Si votre orthophoto présente une échelle invraisemblable,
lancez d’abord `exiftool -FocalLength` sur le produit exporté.

La copie n’est délibérément **pas** `-all:all` : les balises structurelles d’IFD0 perturbent la sortie de LATTICE lorsqu’elles
sont copiées, et `ExifImageWidth` / `ExifImageHeight` sont exclus car ils décrivent la
capture *source* — une exportation ayant subi un redimensionnement comporterait sinon des dimensions
contradictoires avec sa propre trame. Le fichier XMP est écrit directement plutôt que copié, car ExifTool
ignore les balises XMP issues d’une même invocation lorsque le bloc XMP est copié (ce qui supprimerait les balises d’MAPIR
).

### Emplacement des fichiers de sortie

Les fichiers sont enregistrés **dans le dossier du projet, regroupés par appareil photo puis par format de fichier** :

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

Le dossier de l’appareil photo est `LATT-<sensor>-<lens>-F<filter>` pour LATTICE (correspondant aux données EXIF de la capture
`Model`) et `<model>_<filter>` pour Survey3 — deux caméras partageant un capteur et un filtre mais dont les
objectifs diffèrent conservent des arborescences distinctes, car le vignetage, le champ de vision et la distorsion varient. Le format
du dossier suit `--format` : `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` pour
`TIFF (32-bit, Percent)`X.

> **Chaque produit exporté conserve le nom du fichier SOURCE.** Une exportation de radiance de
> `capture_…_raw.tif` s&#x27;appelle toujours `capture_…_raw.tif` — elle se trouve simplement dans
> `tiff32/Radiance_Images/`. **C&#x27;est le dossier qui identifie le produit, pas le nom de fichier**, donc une recherche globale
> sur `*radiance*.tif` ne donne aucun résultat ; il faut plutôt effectuer une correspondance sur le répertoire.

### Enregistrements du capteur de lumière — calibrés `.daq` + `.csv`

`process` gère également les enregistrements `.daq` présents dans votre dossier d’entrée, et il n’a **pas**
besoin d’images pour cela : un DAQ-U / DAQ-M / DAQ-E utilisé seul constitue une
capture complète, et un dossier ne contenant que des fichiers `.daq` est une entrée valide.

Un DAQ peut être enregistré **sans** son étalonnage — c’est ce que font par défaut les
enregistreurs publics
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) : ils écrivent les comptages bruts des capteurs et horodatent le fichier afin que
Chloros récupère l’étalonnage d’usine de ce capteur **par numéro de série** (d’abord dans le cache local,
puis dans le cloud MAPIR) et l’applique. `process` réécrit le résultat :

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Le fichier `.csv` contient une ligne par mesure : horodatage UTC, temps d’intégration, puissance totale,
lux photopique/scotopique, PPFD (et sa répartition bleu/vert/rouge), longueur d’onde de crête, puis le
spectre complet sur la grille de longueurs d’onde propre au capteur. Le fichier `.daq` est réimporté sans être
étalonné une seconde fois.

En cas de réussite, l’exécution génère le fichier `Light-sensor products written: N (calibrated .daq + .csv)`.
La partie entre parenthèses décrit ce qui a été effectivement écrit ; on lit donc
`(RAW COUNTS — this sensor has no calibration bundle)` pour un capteur sans bundle et
`(N calibrated, M raw counts)` pour un dossier contenant les deux. Les en-têtes propres au backend,
`[DAQ-EXPORT]` et `[RUN-SUMMARY]`, tirent leur libellé de la même manière — aucun des
les trois ne peut qualifier une exportation brute de « calibrée ».

Un enregistrement DAQ-U / DAQ-M / DAQ-E dont le bundle d’étalonnage ne peut être récupéré — vous êtes
hors ligne, ou ce capteur n’a pas d’étalonnage enregistré — est **ignoré avec une raison indiquée** sur une
ligne `[DAQ-EXPORT]`, et n’est jamais écrit sous forme de fichier « étalonné » contenant des comptages bruts.
Connectez-vous à Internet et relancez l’opération. La raison est celle que le lecteur a effectivement
déterminée pour ce fichier (schéma illisible, absence de fichier d’étalonnage, erreur d’écriture), et le résumé de l’exécution
répertorie des motifs **distincts** — vingt fichiers ignorés pour une seule cause sont comptabilisés comme une
seule cause, et non comme vingt répétitions de celle-ci.

#### Exportation des enregistrements DAQ-A sous forme de comptes bruts

La gamme **DAQ-A** est antérieure au système de bundle par numéro de série et ne dispose d’aucun bundle d’étalonnage
à récupérer — elle est plutôt étalonnée sur le terrain par rapport à une cible de réflectance, ce qui
explique pourquoi elle n’en a jamais eu besoin. Le rejet de ces enregistrements les a privés de tout moyen d’obtenir leurs
chiffres, c’est pourquoi ils sont exportés sous un **nom différent** :

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Un nom de fichier différent plutôt qu’un indicateur à l’intérieur du fichier, car l’information doit rester intacte
lorsqu’elle est transmise par e-mail sous forme de simple nom. L’en-tête `.csv` indique
`raw spectral sensor counts (NOT irradiance)` et signale que les valeurs sont comparables
**au sein** du fichier — ce qui correspond exactement à l’usage que leur fait l’étalonnage par cible — et
non entre les capteurs. Les colonnes photométriques dépendantes de la puissance (puissance totale, lux photopiques et
scotopiques, PPFD) sont écrites **NULL** plutôt que d’être intégrées à partir des comptages, et le résumé de la
série indique `RAW COUNTS` ; ainsi, les données « exportées » dans un journal ne peuvent pas être interprétées comme de l’irradiance.

Les enregistrements hérités **v1.01 / v1.02** (générés par un DAQ-A-SD) ne comportent pas d’époque par lecture,
mais uniquement l’heure d’écriture du fichier. Le module de correspondance image↔rayonnement descendant continue de les rejeter — la correspondance d’une
trame à une heure d’écriture entraînerait une erreur invisible — mais l’exportateur les lit, et
l’CSV affiche « `clock=daq_created_on` » ; le produit indique donc sur quelle horloge il se base.

### Remarques

- `process` détecte automatiquement si votre dossier est de type « Survey3 », « LATTICE » ou mixte.
- La progression est diffusée via les Server-Sent Events ; l’CLI affiche en temps réel la progression par thread (Détection, Analyse, Traitement, Exportation).
- Pour lesLinux/Jetson, l’CLI vérifie l’espace d’échange et peut afficher un avertissement avant le traitement de dossiers volumineux. Le débayérisation sensible aux textures applique également automatiquement une limite de fréquence du GPU sur les Jetson à faible consommation (Nano, Orin Nano).
- En cas de réussite, l’exécution indique le nombre de produits d’image qu’elle a enregistrés (`Image products written: N`).

#### Une exécution qui n’écrit aucune image échoue

Si vous avez demandé des produits et que l’exécution n’en a écrit **aucun** — uniquement `project.json` et
`calibration_data.json` —, `process` considère cela comme un échec : il affiche
`Processing finished but wrote no image products.` et **sort avec un code de sortie différent de zéro**, ce qui permet à un script de
le détecter. Le message indique le nom du dossier du projet et les causes habituelles :

- le dossier d’entrée n’a pas été reconnu comme une capture (vérifiez la configuration et `--input-level`), ou
- tous les produits demandés ont été ignorés car inapplicables à ces caméras (par exemple, demande de
  radiance/réflectance à partir de caméras RGB uniquement).

Relancez le script avec `--verbose` et vérifiez dans le journal du backend la présence des lignes `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
qui expliquent les sauts par caméra qui, autrement, n’apparaîtraient pas dans la sortie de l’CLI.

Une exécution délibérée en mode « métadonnées uniquement » — tous les produits désactivés et pas de `--indices` — est tout de même un
**succès**, car une sortie d’images vide est le résultat attendu dans ce cas.

Il en va de même pour un **léger**un exécution utilisant uniquement le capteur** : un dossier d’enregistrements `.daq` ne contient, par définition, aucune image à exporter
, et l’exécution est évaluée sur la base des fichiers `.daq` / `.csv` calibrés qu’elle a créés à la place.

---

## `chloros-cli login`

Authentifiez cette machine avec un compte cloud Chloros+. Les identifiants sont mis en cache de manière sécurisée dans `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemples

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (en supprimant ou en dupliquant certaines parties du du mot de passe). En cas de code d’erreur 401, l’CLIe effectue automatiquement une nouvelle tentative en ajoutant à nouveau `$$`, puis en utilisant la moitié du mot de passe sans doublons ; si la nouvelle tentative aboutit, elle vous connecte et affiche la syntaxe correcte avec des guillemets simples à utiliser la prochaine fois.

> **Utilisation sans interface graphique/par script : l’absence de session mise en cache implique un invite interactive, et non un échec rapide.** Toute commande de lancement dede lancement de processus (`process`, `status`, `export-status`, `time-sync`, …) exécutée sans licence/session mise en cache affiche une invite interactive `Email:` / `Password:` sur stdin avant de poursuivre. Une tâche en mode silencieux sans session mise en cache se bloquera donc en attendant une entrée — exécutez `chloros-cli login EMAIL PASSWORD` une fois par machine avant de planifier des tâches sans interface graphique.

---

## `chloros-cli logout`

Efface la session mise en cache et force une nouvelle connexion lors du prochain appel.

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

Interroge la progression en temps réel de l&#x27;exportation Thread-4. Peut être appelé en toute sécurité **pendant** l&#x27;exécution de `process` depuis un autre shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Définit la langue d’affichage de l’CLI (38 langues prises en charge, y compris les langues CJK, RTL et indiennes). Règle par défaut l’affichage en anglais sur les consoles héritées qui ne peuvent pas afficher les caractères.

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

---

## `chloros-cli update`

Linux/ Jetson uniquement. Vérifie `version_url` à partir de `/etc/chloros/update.conf` et propose de télécharger et d’installer le fichier `.deb` correspondant.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Sur Linux / Jetson, l’CLI effectue également une **vérification automatique des mises à jour à chaque démarrage** (non bloquante, ne retarde jamais la commande) : il lit `/etc/chloros/update.conf`, met le résultat en cache pendant 1 heure dans `~/.chloros/update_cache.json`, et affiche `Update available: vX.Y.Z / Run: chloros-cli update` lorsqu’une version plus récente existe. Ignore silencieusement toute erreur et les commandes de type « Windows ».

---

## `chloros-cli selftest`

Exécute un test de fonctionnement en 7 étapes : version, disponibilité des ports, démarrage du backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), présence du modèle de débruitage, disponibilité de CUDA et du débruiteur.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

État et contrôle du grand maître PTP. L’hôte « Chloros » exécute le grand maître PTP ; les caméras LATTICE et les unités DAQ-E lui sont asservies pour la synchronisation horodaté entre appareils.

| Sous-commande | Description |
| --- | --- |
| `status` | Afficher l’état du grand maître, les priorités BMCA et l’identité de l’horloge. |
| `peers` | Lister les esclaves détectés via Delay_Req (caméras + capteurs DAQ-E). |
| `cameras` | État de santé PTP par caméra (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Redémarrer le processus grandmaster. |
| `set-priority --priority1 N --priority2 N` | Remplacer les priorités BMCA. |

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

Contrôle des caméras LATTICE. Chaque sous-commande transite par le backend Chloros ; ce dernier gère le pool de caméras, de sorte que les appels CLI suivants réutilisent le même descripteur ouvert.

### Options communes (partagées par la plupart des sous-commandes)

| Indicateur | Description |
| --- | --- |
| `-d, --device N` | Index de la caméra (par défaut : 0). |
| `-s, --serial SN` | Numéro de série spécifique ; remplace `--device`. |
| `--serials SN1,SN2,…` | Numéros de série séparés par des virgules pour le fonctionnement multi-caméras. |
| `--all` | Fonctionner sur toutes les caméras détectées. |
| `--exposure US` | Temps d&#x27;exposition en microsecondes. |
| `--gain DB` | Gain en dB. |
| `--pixel-format FMT` | Par exemple : `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimensions de l’image. |
| `--preset {default,high_quality,high_speed,triggered}` | Appliquer un préréglage. Tous fonctionnent en mode libre, sauf `triggered`, qui arme la caméra pour un front matériel sur la ligne 2 — en l’absence de signal sur cette ligne, elle attendra indéfiniment plutôt que de capturer une image. |
| `-o, --output DIR` | Répertoire de sortie (par défaut : `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Taille des paquets GVSP. `auto` exécute des sondes ICMP+GVSP ; `jumbo` = 9000 ; `standard` = 1500. |

### Workflow de première connexion de la caméra LATTICE

```bash
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
#      - Auto-shrink frame size to fit the wire
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
| `lattice info` | Lister les caméras connectées (fabricant, modèle, numéro de série, adresse IP, adresse MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analyser le système hôte pour une configuration optimale des caméras. `--no-discover` ignore la détection des caméras (plus rapide, analyse par carte réseau uniquement). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Vérifier/corriger les paramètres de la carte réseau ; estimer la bande passante et le nombre d&#x27;images par seconde. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Stable-schéma de capacités réseau du backend + recommandation de tableau (renvoie `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` conserve la résolution demandée mais plafonne le nombre d’images par seconde cible — lire `recommended.recommended_target_fps` et le transmettre comme cible de connexion ; considérez cela comme un succès, et non comme une erreur. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analyse de scénarios sans ouvrir les caméras. **`--n-active` correspond au nombre total de caméras sur le réseau, et pas seulement à celui de ce tableau**— augmentez-le lorsque des caméras autonomes diffusent simultanément, ou lorsque le budget réseau est calculé par rapport à une demande qui sous-estime leur nombre (par défaut : `len(--models)`). Affiche toujours les lignes agrégées `Wire budget:` (Mo/s demandés par rapport au plafond sans risque de collision) et `Max cameras:`, et signale `** OVER-SUBSCRIBED**` lorsque le réseau est surchargé — voir [FPS du réseau et modèle de rafale](#array-fps--burst-model). |
| `lattice gpu` | Afficher l’état du GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Vérifier ou mettre à jour le micrologiciel de la caméra. La sélection locale `.fwa` est verrouillée : le fichier dans `firmware/<MODEL_PREFIX>/` correspondant à celui de la version `MIN_FIRMWARE_VERSION` est flashé lorsqu’il est présent (la version la plus récente sert uniquement de solution de secours) ; ainsi, une image du fournisseur plus récente stockée sur le disque reste inactive jusqu’à ce que ce pin soit modifié — les versions plus récentes sont délibérément déployées sur les appareils via le manifeste AWS signé, ce qui est préférable lorsqu’elles sont plus récentes. |
| `lattice presets [--apply NAME]` | Lister ou appliquer les préréglages de la caméra. |
| `lattice status` | État en temps réel de la caméra. |

#### Capture

| Sous-commande | Objectif |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Image unique. **Enregistre tous les types d’exportation par défaut** (`--processing all`) ; voir [Niveaux d’exportation de capture](#capture-export-levels-the-all-default). `--levels` enregistre un sous-ensemble explicite (remplace `--processing`) ; `--force-daq` écrit la valeur DAQ attribuée en tant que fichier « sidecar » `.daq`, même lors d’une capture en mode brut uniquement. `--jpeg-quality` = JPEG qualité 1–100 (par défaut 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Enregistrement sur disque jusqu’à la combinaison Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Aperçu MJPEG en direct via un navigateur. `--ae-damping` définit l’amortissement de l’exposition automatique (0,4–100). |

#### Réglage du capteur

| Sous-commande | Objectif |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Lecture/écriture de n&#x27;importe quel nœud GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Exposition et AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Gain et gain automatique. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Zone d’intérêt (ROI) du capteur et regroupement de pixels. |
| `lattice format [--set FMT] [--list]` | Format des pixels. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Déclenchement matériel/logiciel. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (aucun indicateur = balance des blancs en une seule prise) | Opérations de balance des blancs. RGB / Caméras Bayer uniquement ; opération nulle (ignorée) sur les M3M monochromes. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Pipeline de couleurs de l’affichage «RGB». `natural` (par défaut) correspond au traitement en direct le plus économique ; `enhanced` ajoute la suppression des franges + la vibrance + le contraste local CLAHE pour obtenir un rendu « hub-parity » complet à un coût de traitement par image environ deux fois plus élevé, ce qui se traduit par une fréquence d’images **en direct** — les captures enregistrées bénéficient toujours du traitement complet dans les deux cas. RGB / Caméras Bayer uniquement ; ignoré sur la M3M mono. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Afficher la saturation/le contraste (caméras avec filtre RGB). Ignoré sur la M3M mono. |
| `lattice filter [--set NAME] [--list]` | Définit le modèle de filtre de la caméra (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Mesure de la puissance et des nœuds thermiques ; activation/désactivation du mode veille à faible consommation. |

#### Étalonnage et capteurs

| Sous-commande | Objectif |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Étalonnage à partir d’une cible de réflectance. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Commandes intégrées de l’|
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Appliquer une correction de vignettage aux images existantes. |

#### Multicaméra (sessions transitoires)

| Sous-commande | Objectif |
| --- | --- |
| `lattice multi-info` | Lister toutes les caméras ayant un rôle de synchronisation. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Une image synchronisée par caméra. Enregistre **tous les types d’exportation par défaut**lorsqu’un tableau persistant est connecté ; le mode de secours transitoire sans tableau est**uniquement débayérisée** (exécutez d’abord `array-connect` pour le reste). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Diffusion d’images synchronisées (transitoire). |
| `lattice multi-test [--count N]` | Test de synchronisation GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Détection automatique du câblage maître/esclave GPIO. |

#### Alignement

| Sous-commande | Objectif |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus les paramètres de détection/correspondance `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, les paramètres RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinaison multi-images `[--averaging mean\|median\|inlier_weighted]`, contraintes géométriques `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restriction spatiale `[--roi X0,Y0,X1,Y1] [--mask PATH]` et remplacements par esclave `[--per-cam-override SN:KEY=VALUE]` (répétables) | Calcul du profil d’alignement à partir de caméras en temps réel. `--prefilter` utilise par défaut `gradient` (carte des contours ; correspond à l’aligneur de l’interface graphique/du tableau — les contours sont conservés d’une bande spectrale à l’autre). `--matcher flann` est efficace à partir d’environ 5 000 caractéristiques ; `--averaging median` est robuste face à une capture défectueuse, `inlier_weighted` pondère en fonction du nombre de correspondances ; `--lock-scale` effectue une projection vers la rotation la plus proche (sans échelle), `--lock-axis` remet à zéro une composante de translation ; `--mask` s&#x27;applique à toutes les caméras (utiliser `--per-cam-override` pour les paramètres par caméra, par exemple `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` refuse d’enregistrer un étalonnage dont la valeur RMS de reprojection dépasse le seuil. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Capture une image multibande alignée. `--bit-depth` s&#x27;aligne par défaut sur la caméra ; `--no-crop` conserve l&#x27;image complète (en la complétant par du noir) ; `--interpolation` (par défaut `linear`) et `--border-mode`/`--border-value` (par défaut `constant`/0) contrôlent la déformation par le CPU — le chemin GPU est bilinéaire dans tous les cas. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Trames multibandes alignées sur le flux (mêmes paramètres de warp que `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Afficher les détails du profil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Modifier l’ordre des calques. |

#### Index / Mathématiques de la végétation

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Ensemble complet d’options : `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (répétable), `--capture-level raw|debayered|radiance|reflectance|unknown` (remplace le niveau de capture enregistré dans l’TIFF source ; par défaut : lu à partir des métadonnées de l’TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Avec `--live`, les boutons de déformation d’alignement s’appliquent également : `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Les symboles `--channel` sont sensibles à la casse.** Le côté « symbole » doit correspondre exactement aux noms des canaux du préréglage (les préréglages utilisent des minuscules, par exemple : NDVI = `red`, `nir` — voir `--list-presets`), et la partie « bande » doit correspondre à un nom de bande dans la pile alignée (ou être un index de bande à partir de 0 en mode hors ligne). `--channel red=Red_660 --channel nir=NIR_850` fonctionne ; `--channel RED=660` échoue avec une erreur `channel_map missing entries`.

#### Connexions persistantes (Smart-Prep, flux équivalent à l’interface graphique)

Ces commandes maintiennent les caméras ouvertes dans le pool du backend d’un appel à l’autre de la fonction «CLI».

| Sous-commande | Objectif |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Ajouter une caméra au pool (caméra unique, pas de réseau). |
| `lattice cam-disconnect [--serial SN] [--all]` | Libérer. |
| `lattice cam-list` | Lister les caméras du pool. |
| **`lattice array-connect`**|**Connecter un ensemble synchronisé persistant (point d’entrée recommandé).** Exécute l’intégralité du flux de préparation intelligente via l’interface graphique. |
| `lattice array-disconnect [--array-id ID] [--all]` | Libérer un ensemble. |
| `lattice array-list` | Lister les ensembles connectés. |
| `lattice array-status [--array-id ID]` | FPS en direct, PTP, dernière erreur. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Une capture synchronisée à partir du réseau en direct — Unique / Continue / Par intervalle / La plus rapide. **Par défaut : `all`** (un fichier par type d’exportation applicable et par caméra). Les caméras ignorées (par exemple, celles RGB exclues de la radiance/réflectance) sont signalées avec `Skipped: SN:<serial> (<reason>)` ; la mesure DAQ utilisée pour la réflectance est enregistrée parallèlement et signalée avec `DAQ: <path>`. Voir [Modes de capture, Enregistreurs et retraitement hors ligne](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Enregistre la vue en direct de l’indice combiné au format vidéo/GIF (qualité de surveillance ; nécessite que le flux combiné soit ouvert). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Rafale Bayer brute à haute fréquence d’images (qualité analyse ; retraitement hors ligne). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Retraitement d’une rafale brute enregistrée en vidéo(s) calibrée(s). |

##### Options `array-connect`

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Détection automatique de toutes les caméras LATTICE (nécessite ≥ 2) | La première série correspond au MAÎTRE. Si cette option est omise, la détection se limite aux modèles LATTICE (`TRI032*`) et les connecte toutes. |
| `--line {Line0,Line2,Line3}` | `Line2` | Ligne de synchronisation GPIO. |
| `--target-fps F` | auto | Fréquence de déclenchement du maître. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Remplacer le sélecteur de niveau. |
| `--wire-ceiling-mbps MB_PER_S` | détecté automatiquement | **Le budget de bande passante soutenu de l’hôte, en Mo/s — la valeur à laquelle repose l’allocation de l’ensemble de la matrice.** Réduisez-le lorsque la matrice signale des trames corrompues par le protocole GVSP : la valeur automatique est dérivée du débit de liaison annoncé par la carte réseau, ce qui surestime les adaptateurs USB, les voies PCIe à faible bande passante et les structures partagées très sollicitées. Cette valeur est conservée dans le bloc de capture de la matrice du projet ; une réouverture / CLI / SDK. Voir [État de santé du réseau de cartes](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Regroupement matériel. |
| `--no-recommend` | désactivé | Sauterl&#x27;étape d&#x27;analyse du réseau. |
| `--no-ptp` | désactivé | Désactiver le PTP (les horodatages entre caméras ne sont alors **pas** comparables). |

### Smart-AE / Smart-Capture

Les matrices LATTICE exécutent l’AE en continu en arrière-plan dès qu’elles sont connectées, mais une scène nouvellement cadrée met un certain temps à converger. `array-capture --smart` est la **solution pratique intégrée** : elle attend que l’exposition automatique se stabilise sur toutes les caméras du réseau, puis déclenche la capture. Utilisez-la lorsque vous changez de scène en cours de session.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

La politique de stabilisation est prudente par défaut : délai d’expiration de 5 s, fenêtre de stabilité de 1,5 s, tolérance de variation d’exposition de ±5 %. Ajustez ces paramètres via l’SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) si vous avez besoin d’un comportement différent de celui de l’automatisation.

### Niveaux d’exportation des captures (valeur par défaut : `all`)

À partir de cette version, `lattice capture`, `lattice multi-capture` et `lattice array-capture` **sont définis par défaut sur `--processing all`** — un fichier enregistré par type d’exportation qui s’applique à chaque caméra, ce qui correspond au comportement « Capturer tout » de l’interface graphique. Les niveaux sont les suivants :

| Niveau | Sortie | S’applique à |
| --- | --- | --- |
| `raw` | Bayer monocanal (caméras monochromes : la bande unique) provenant directement du capteur. | Toutes les caméras. |
| `debayered` | Démosaïquage BGR à 3 canaux (caméras monochromes : 1 canal en niveaux de gris). | Toutes les caméras. |
| `radiance` | float32 W/m²/sr/nm via la chaîne radiométrique complète. | Multispectrales (M3C/M3M) uniquement — **ignoré pour les caméras à filtre d’RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatible Pix4D. | Multispectral uniquement, et **uniquement lorsqu’un DAQ est associé et que la caméra est calibrée** ; sinon ignoré. |
| `preview` / `display` | Chaîne complète de prévisualisation via l’interface graphique (CCM + WB + gamma selon le profil de la caméra). `lattice capture` nomme ceci `preview` ; `array-capture`/`multi-capture` utilisent `display`. | Toutes les caméras. |

Passez un seul niveau pour n&#x27;en enregistrer qu&#x27;un seul (`--processing debayered`). Lorsque vous demandez `all`, les niveaux qui ne s’appliquent pas à une came donnée sont ignorés (et signalés), sans générer d’erreur — une caméra non connectée ou non calibrée reçoit tout de même `raw` / `debayered` / `preview`.

Pour toute image de réflectance, la valeur de rayonnement descendant du DAQ effectivement utilisée est enregistrée dans un **`.daq`** situé à côté de l’image (afin que la capture puisse être retraité ultérieurement) et indiquée sur une ligne `DAQ:`.

### À quoi ressemble un dossier de captures

Chaque type d’exportation est placé dans son **propre sous-dossier** sous `-o`, de sorte qu’une capture à plusieurs niveaux ne mélange jamais les types :

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` correspond à l’horodatage de la capture et `<serial>` au numéro de série de la caméra ; ainsi, un groupe synchronisé partage un
horodatage commun à toutes les caméras. **Notez toutefois une asymétrie :** le niveau `display` est stocké dans un dossier
nommé `preview/`, tandis que les fichiers eux-mêmes conservent `_display` dans le nom — le dossier et le suffixe diffèrent
pour ce niveau uniquement. Les niveaux inconnus sont placés par défaut dans un dossier portant leur propre nom, et si le sous-dossier
ne peut pas être créé, le fichier est écrit à la racine de sortie plutôt que d’être perdu.

**Retraitement d’un dossier de captures :**pointez `chloros-cli process` vers la**racine des captures**
(`output/`). `process` n&#x27;importe normalement que le dossier que vous nommez, mais lorsque ce dossier ne contient aucune
d’images et qu’il comporte des sous-dossiers, il explore automatiquement l’arborescence — ainsi, les sous-dossiers de niveau racine et la
racine `.daq` sont tous récupérés en une seule fois. Chaque niveau d’une capture est importé sous forme d’une seule image, les
autres niveaux étant disponibles en tant que modes, plutôt que sous forme d’une image par niveau.

Nommer directement un **sous-dossier de niveau** (par exemple `output/raw/`) fonctionne également. Cela laisse la racine
`.daq` ; veillez donc à copier ou à indiquer la lecture DAQ parallèlement lorsque vous dérivez à nouveau un produit radiométrique
à partir de `raw/` — sinon, la correspondance d’horodatage n’aura aucun élément de référence.

**Le traitement commence toujours à partir de `raw`.** Au sein de chaque capture, l’image brute constitue la source du pipeline ;
`debayered`, `radiance`, `reflectance` et `preview` apparaissent comme des modes d’affichage mais ne sont jamais renvoyés
vers le pipeline. Le retraitement d’un produit dérivé réappliquerait la vignette, le CCM et
les calculs de radiance déjà intégrés à ses pixels ; c’est pourquoi « Chloros » les rejette plutôt que de
procéder à un double traitement. Deux conséquences à connaître :

- Les rendus `index/` et `composite/` ne sont **jamais** traités. Ce sont des sorties, pas des captures —
  un rendu LUT « NDVI » n’a aucune interprétation significative en termes de radiance.
- Un dossier de captures exporté **sans** `raw` (par exemple `array-capture --processing reflectance`) ne contient
  aucune source valide pour le pipeline. Ces captures s’importent et s’affichent normalement, mais `process` les ignore
  et l’indique :

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Si vous avez réellement besoin de faire passer un produit dérivé — une session de hub capturée avec
  `demosaic` activé, ou un dossier hérité —, `--input-level {raw,debayered,processed}` force le point d’entrée
  et passe outre l’ignoration. Cet indicateur constitue une échappatoire délibérée ; `auto` (la valeur par défaut)
  ne traite jamais une capture dépourvue de données brutes.

### Captures ignorées dans lesfiltres

Lorsque vous mélangez des caméras « RGB » et multispectrales au sein d’un même réseau, `array-capture --processing radiance` (ou `reflectance`) enregistre les images multispectrales et **ignore** les caméras « RGB » — carla radiance « -Bayer » n’a pas de sens pour un capteur à large bande. Le fichier « CLI » répertorie explicitement chaque fichier enregistré (avec son niveau d’exportation), chaque instance de `.daq` écrite et chaque image ignorée, ce qui explique le nombre de fichiers :

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Les jetons indiquant la raison du saut suivent le format `<level>-not-applicable-to-rgb-cam`. La réflectance peut également être ignorée avec `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ainsi qu’avec `dls-uncalibrated-band-<nm>` lorsque la bande se situe en grande partie en dehors de la plage radiométriquement étalonnée du capteur de lumière DAQ (~374–974 nm) — parmi les références commercialisées, seule la F988, dont le parcours pris en charge est le flux de travail avec panneau de réflectance.

Utilisez `--processing debayered` (ou `display`) pour inclure toutes les caméras, quel que soit le type de filtre, ou la valeur par défaut `all` pour obtenir d&#x27;un seul coup tous les niveaux applicables par caméra.

---

## Modes de capture, enregistreurs et retraitement hors ligne

Ces éléments fonctionnent tous sur un **tableau persistant** (exécutez d’abord `array-connect`). Ils reproduisent le panneau de capture de l’interface graphique.

### Modes `array-capture`

`array-capture` est une commande unique proposant quatre modes d’obturation ainsi qu’un ensemble de commutateurs d’exportation :

| Mode | Indicateur | Comportement |
| --- | --- | --- |
| **Unique** *(par défaut)* | (aucun) | Un seul groupe de capture synchronisé, puis sortie. |
| **Continu** | `--continuous` | Passes successives jusqu’à `Ctrl+C`, `--count N` ou `--duration S`. |
| **Intervalle** | `--interval S` | Un passage toutes les `S` secondes (mesurées à partir du début de chaque passage), mêmes limites. |
| **Le plus rapide** | `--fastest` | Bruts uniquement + la lecture DAQ attribuée + le composite d’indice combiné ; ignore les calculs de radiance/réflectance/affichage afin que l’image s’affiche rapidement. Implique `--processing raw --force-daq`. Retraiter ultérieurement les `.daq` en produits calibrés ultérieurement. |

Options d’exportation (à combiner avec n’importe quel mode ; toutes partagent l’interface graphique et le point de fin d’SDK) :

| Indicateur | Effet |
| --- | --- |
| `--processing LEVEL` | Niveau d’exportation unique, ou `all` (par défaut). |
| `--levels L1,L2,…` | Sous-ensemble explicite de types d’exportation (par ex. `raw,radiance,reflectance`) ; **remplace `--processing`**. |
| `--aligned` / `--no-aligned` | Aligne l’exportation non brute de chaque élément sur le [profil d’alignement](#alignment) du tableau (co-enregistré). Les données brutes ne sont pas alignées mais portent la transformation dans les métadonnées. Revient à un alignement non défini (avec un avertissement) si le tableau n’a pas de profil. |
| `--index` / `--no-index` | Enregistrer / ignorer la superposition de l’indice de végétation par caméra lorsqu’elle est configurée. Par défaut : l’afficher. |
| `--force-daq` | Enregistrer la lecture DAQ/DLS attribuée sous forme de fichier sidecar `.daq` même lorsqu’aucun niveau sélectionné n’en a besoin (par exemple, une capture « raw » uniquement), afin que les images puissent être retraitées hors ligne en réflectance/indice. |
| `--smart` | Attendre que l&#x27;AE se stabilise sur toutes les caméras avant de déclencher (voir [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compression des pixels «TIFF». `deflate` (par défaut) = zlib L1 sans perte + prédicteur horizontal, environ 4,1 Mo par image en pleine résolution ; `none` = non compressé, écriture environ 5 fois plus rapide à environ 6,3 Mo par image — à utiliser pour un débit soutenu maximal lorsque l&#x27;espace disque le permet. Les deux sont sans perte et s&#x27;affichent de manière identique à l&#x27;importation. |

> **TIFF d&#x27;écriture unique + le modèle à débit soutenu.**Les captures sont écrites en**un seul**passage dans un fichier TIFF contenant les pixels + XMP + IFD0 Marque/Modèle (mesuré sur Mono12 en pleine résolution : 36 ms compressées / 6,5 ms non compressées, contre environ 148 ms pour l’ancienne méthode « écriture puis réécriture avec ExifTool ») ; la seule tâche ExifTool restante (peaufinage du sous-IFD EXIF) s’exécute sur un worker asynchrone en arrière-plan, et une image est complète et prête à être importée même si ce worker ne s’exécute jamais. Notez que la compression DEFLATE maintient le GIL de l’Python, de sorte que les écritures compressées ne sont**pas**parallélisées entre les threads d’écriture propres à chaque caméra — une capture soutenue en pleine résolution avec 8 caméras à la cadence du capteur (~10,4 images par seconde) nécessite `--compression none`**et** un disque de classe NVMe (~500 Mo/s d’écritures soutenues). Le même paramètre est accessible sous le nom `compression` sur `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — vidéo/GIF à index combiné (qualité surveillance)

Enregistre tout ce que la **vue en direct à index combiné** sur un fichier `.avi` (et éventuellement un fichier `.gif`). Comme il exploite le flux composite en direct, le flux combiné doit être ouvert (par exemple, lorsque le tableau est prévisualisé dans l’interface graphique) pour que les images puissent être capturées. Il interroge la progression toutes les 2 s et s’arrête sur `--duration`, `Ctrl+C`, ou lorsque l’enregistreur se termine automatiquement.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--array-id ID` | matrice uniquement | Matrice cible (à omettre si une seule est connectée). |
| `-o, --output DIR` | `output` | Répertoire de sortie (local au backend). |
| `--fps F` | `10` | Fréquence d&#x27;images d&#x27;enregistrement. |
| `--duration S` | jusqu&#x27;à Ctrl+C | Arrêt automatique après `S` secondes. |
| `--gif` | désactivé | Enregistrer également un GIF animé. |
| `--gif-only` | désactivé | Enregistrer uniquement un GIF (pas de `.avi`). |

### `array-burst` — rafale raw-Bayer à haute fréquence d’images (qualité analyse)

Lit directement le tampon de groupe synchronisé de la boucle de capture — **aucune chaîne d’étalonnage, aucun exiftool, aucun affichage en direct requis** — ce qui permet de fonctionner à la fréquence de capture maximale de l&#x27;appareil. Écrit les images brutes + un manifeste par image + un `.daq` par lecture DLS distincte sous `<output>/bursts/<base>/`. Retraiter hors ligne (commande suivante), ou passez `--build` pour que le traitement s&#x27;effectue immédiatement à l’arrêt.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--array-id ID` | tableau uniquement | Tableau cible. |
| `-o, --output DIR` | `output` | Répertoire de sortie (la rafale est placée dans `<DIR>/bursts/<base>/`). |
| `--duration S` | jusqu’à Ctrl+C | Arrêt automatique après `S` secondes. |
| `--max-frames N` | illimité | Arrêt automatique après `N` trames brutes. |
| `--build` | désactivé | Après l&#x27;arrêt, retraiter immédiatement la rafale (identique à `array-build-video`). |
| `--products …` | `combined:index` | Avec `--build` : vidéo(s) à générer (voir ci-dessous). |
| `--fps F` | `10` | Avec `--build` : nombre d’images par seconde (fps) de la vidéo de sortie. |
| `--save-tiffs` | désactivé | Avec `--build` : enregistrer également des fichiers TIFF calibrés image par image. |
| `--gif` | désactivé | Avec `--build` : enregistrer également des GIF animés. |

### `array-build-video` — retraiter hors ligne une rafale enregistrée

Aligne temporellement chaque image brute sur la lecture `.daq` enregistrée la plus proche et la fait passer par la **même chaîne de radiance / réflectance / indice que le pipeline d’importation**, générant ainsi une ou plusieurs vidéos.

`--products` est une liste séparée par des virgules d’éléments `kind:level`, où `kind` ∈ `per_cam` | `combined` et `level` ∈ `radiance` | `reflectance` | `index`. Un `level` nu (sans `kind:`) prend par défaut la valeur `per_cam`. La valeur par défaut est `combined:index`.

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
| `--products …` | `combined:index` | Liste `kind:level`, comme ci-dessus. |
| `--fps F` | `10` | Nombre d&#x27;images par seconde (fps) de la vidéo de sortie. |
| `--save-tiffs` | désactivé | Enregistrer également des fichiers TIFF calibrés image par image en même temps que la vidéo(). |
| `--gif` | désactivé | Enregistrer également des GIF animés. |

> **Choisissez l&#x27;enregistreur adapté.** `array-record` est de *niveau surveillance* : il capture le signal composite en direct tel qu&#x27;il s&#x27;affiche et nécessite que le flux soit ouvert. `array-burst` → `array-build-video` est de *niveau analyse* — il enregistre les données brutes du capteur à plein débit et reconstitue ensuite des vidéos calibrées en radiance, réflectance et indice, sans nécessiter de visualisation en direct.

### Caméras mono (M3M) à bande unique

La gamme **M3M**est la version monochrome de la gamme Bayer**M3C**: un filtre d’interférence à bande étroite par caméra (`M3M-<lens>-F<wavelength>`, par exemple `M3M-L87-F685`), de sorte que le capteur fournit une**seule bande en niveaux de gris** sans mosaïque de Bayer. Il n’y a pas de démosaïquage à effectuer, pas de diaphonie entre canaux à démêler, ni de balance des blancs à régler — tout le processus de traitement des couleurs «RGB-display» ne s’applique tout simplement pas.

Ce que cela signifie pour l’CLI :

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**détectent une caméra monochrome et**sautent cette étape avec un message d’une ligne** au lieu d’appliquer des paramètres inutiles. Ils fonctionnent toujours normalement avec une caméra RGB /Bayer M3C dans la même session.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fonctionnent toujours — la radiance et la réflectance sont des cartes radiométriques *par bande* et sont parfaitement bien définies pour une seule bande. Les portent une matrice de réponse du capteur **identique** (pas de démixage 3×3), de sorte que le plan passe à travers les calculs d’étalonnage sans être modifié.
- **Une seule caméra monochrome ne peut pas produire d’indice de végétation.**NDVI / NDRE /etc. nécessitent au moins deux bandes (par exemple Red + NIR). Pour obtenir un indice à partir d’un matériel monochrome, pointez**plusieurs** caméras M3M sur différentes longueurs d’onde, alignez-les en une seule pile multibande, puis calculez l’indice à partir de *celle-ci* :

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

Les symboles `--channel` doivent correspondre **exactement** aux noms des canaux du préréglage* (sensible à la casse ; les NDVI sont en minuscules `red`,`nir` — voir `--list-presets`), et le nom de la bande désigne une bande dans la pile alignée (le mode hors ligne accepte également les indices de bande à partir de 0, parpar exemple `--channel red=0 --channel nir=1`).

Le discriminant dans toute la pile est le jeton `M3M` dans la chaîne de modèle (il n’apparaît jamais dans une chaîne `M3C`), affiché dans l’interface graphique/SDK sous la forme `is_mono`.

---

## Configuration et réglage de la carte réseau de l’hôte (matrices LATTICE)

Les caméras LATTICE transmettent le protocole GVSP via la carte Ethernet de l’hôte, c’est pourquoi, pour les réseaux de caméras multiples, le **pilote**de la carte et la**taille de l’anneau de réception** sont tout aussi importants que le débit de la liaison. Des paramètres incorrects se traduisent par une porte `FRAMES WILL DROP` / `Reduce ROI to enable` dans le panneau « Paramètres du réseau » (et dans `lattice network-analysis` / `analyze_array_network()` de l’SDK), même lorsque les caméras elles-mêmes fonctionnent correctement.

### Cartes USB 10 GbE — Realtek RTL8157 (« Realtek USB 10GbE Family Controller »)

| Élément | Valeur requise | Pourquoi c&#x27;est important |
| --- | --- | --- |
| **Version du pilote**|**≥ v10.67 (janvier 2026)**, INF `rtump64x64sta.inf` | L&#x27;ancien pilote**de 2016**(v10.65, `rtump64x64.inf`) gère mal la mise hors tension et provoque des « bugchecks » avec**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**lors de l’arrêt, du redémarrage ou de la mise en veille. La transition se bloque (~5 min ), l’utilisateur procède à un arrêt forcé, et ces arrêts non contrôlés répétés**corrompent le référentiel WMI**(PowerShell et les outils commencent à rencontrer des erreurs avec `Invalid class`) et**bloquent la pile USB** au démarrage suivant (la carte réseau ne s’active pas ; les périphériques USB cessent d’être détectés). Effectuez une mise à jour depuis realtek.com (ou auprès du fournisseur de la clé) avant de compter sur des redémarrages « propres ». |
| **Tampons de réception**— mot-clé `ReceiveBufferLen` |**256**(maximum du pilote) | L&#x27;anneau RX de la carte réseau. La valeur par défaut du pilote,**32**, ne laisse qu&#x27;environ 0,26 Mo d&#x27;anneau utilisable — bien trop petite pour une rafale multi-caméras — ce qui fait que le panneau de configuration de la matrice signale `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` et bloque les connexions. À**256**, la boucle est grande (**environ 13,5 Mo mesurés sur l’hôte 10 GbE du laboratoire**), ce qui donne au pipeline de réception une réelle marge de manœuvre pour les rafales multi-cam GVSP. (La capacité d’une configuration donnée à *établir une connexion* est déterminée par deux vérifications : la vérification d’admission **sensible au drainage**et la vérification**d’agrégation sur) — et non d’une simple comparaison brute entre la rafale et l’anneau ; voir [Modèle de fps et de rafales de l’array](#array-fps--burst-model).) |
| **URB de réception**— mot-clé `PendingReceives` |**64** (max.) | Blocs de requêtes USB en cours de transmission ; à augmenter parallèlement aux tampons de réception pour l’absorption des rafales. |
| **Trame jumbo** — mot-clé `*JumboPacket` | **9014** | Nécessaire pour les paquets GVSP de 9 000 octets (6 fois moins de paquets par trame qu’avec 1 500). |

> ⚠️ **Une mise à jour du pilote de la carte réseau RÉINITIALISE ces propriétés avancées à leurs valeurs par défaut.**Après avoir mis à jour ou remplacé le pilote de la carte,**réappliquez** `ReceiveBufferLen=256` et `PendingReceives=64`, sinon le panneau de la baie se verrouillera à nouveau même si « rien n’a changé au niveau du matériel ». C’est la cause n° 1 pour laquelle un système qui fonctionnait auparavant refuse soudainement de se connecter.

Appliquez ces paramètres depuis un **PowerShell** (remplacez par le nom de votre carte, par exemple `"Ethernet 5"`) :

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` concerne les adaptateurs USB 10 GbE.** Il détecte désormais le type d’adaptateur et ajuste le mot-clé « receive-ring » approprié : `*ReceiveBuffers`→2048 pour les cartes réseau PCIe (Intel I219, etc.), ou `ReceiveBufferLen`→256 + `PendingReceives`→64 pour le contrôleur **USB** 10 GbE de Realtek (qui n’expose pas `*ReceiveBuffers`). Les valeurs cibles sont limitées à la valeur maximale signalée par chaque pilote (`NumericParameterMaxValue`), ce qui empêche toute écriture d’une valeur hors plage. Exécutez cette commande depuis un terminal **privilégié** ; comme pour tout réglage basé sur le registre, la modification prend effet après un redémarrage de la carte réseau ou un redémarrage du système. Les commandes manuelles `Set-NetAdapterAdvancedProperty` ci-dessus restent une bonne alternative — elles s’appliquent à la volée (en reliant la carte réseau) sans redémarrage.

### Notions de base sur le réseau (tous les liens LATTICE)

- **Adressage :** adresse locale de liaison `169.254.0.0/16` (GigE Vision LLA). L’hôte utilise une adresse statique `169.254.x.x/16` ; les caméras et le DAQ-E s’attribuent automatiquement une adresse dans la même plage. Aucun DHCP ni passerelle n’est nécessaire.
- **Taille des paquets :**de préférence jumbo (9 000), mais laissez la détection automatique la déterminer — elleeffectue des mesures à chaque connexion et dépasse déjà la limite ICMP de 1 500 octets de la caméra via une sonde GVSP ; elle opte donc pour la taille « jumbo » partout où la ligne le permet réellement. Ne forcez pas la valeur avec `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` que si vous en savez plus que la sonde, et privilégiez lacommande ponctuelle plutôt que permanente : une configuration fixe contourne la détection, donc si le chemin ne peut pas réellement supporter 9 000,**chaque** capture aboutit à un délai d’expiration avec `SC_ERR_TIMEOUT -1011` (voir [Variables d’environnement](#environment-variables)).
- **La taille de l’anneau RX évolue avec `ReceiveBufferLen` :**à la valeur par défaut `32`, l’anneau utilisable est d’environ 0,26 Mo (trop petit pour toute rafale multi-caméras) ; à la valeur maximale `256`, il est vaste (environ 13,5 Mo mesurés sur l’hôte 10 GbE du laboratoire), ce qui offre une réelle marge de manœuvre. La possibilité de connexion d’une configuration est alors déterminée par le contrôle d’admissibilité tenant compte de la consommation**et** par le contrôle de sursouscription agrégée ci-dessous — et non par une simple comparaison brute entre le débit en rafale et la capacité de l’anneau.

### Modèle de fps et de débit en rafale du réseau de cartes

Comment lire le panneau « Array Settings » (et `lattice analyze-array` / le `analyze_array_network` de l’SDK) :

- **La rafale est additionnée par caméra selon leformat de pixels réel de chaque caméra.**Les caméras mono**M3M**transmettent en**Mono12 (2 bits/px)**; les caméras Bayer**M3C**transmettent en 8 ou 12 bits (le modèle TRI032S émet silencieusement du BayerRG12 même lorsque du BayerRG8 est demandé). Ainsi, une image en pleine résolution de 4- image en pleine résolution provenant de 4 caméras pèse**environ 12,6 Mo si toutes sont en 8 bits, mais environ 25 Mo avec trois caméras mono 12 bits**. La projection détermine le format de chaque caméra à partir de son modèle (cache d’identité), de sorte que la rafale correspond à ce que le câble transporte réellement — et non à une- hypothèse de taille BayerRG8.
- **Un adaptateur Ethernet USB est plafonné à 200 Mo/s, quelles que soient ses spécifications techniques.** Le tableau d’efficacité qui convertit un débit de liaison en un débit soutenu est dérivé du PCIe ; une carte réseau USB annonce son *débit Ethernet* mais est limité par le bus USB et son pilote. Une clé USB 10 GbE affichait autrefois un débit « soutenu » d’environ 1 063 Mo/s — un chiffre qui n’a jamais été vérifié — et la régulation de débit qui en résultait corrompait 6 à 18 % des images tout en continuant à indiquer un nombre d’images par seconde (fps) cible satisfaisant. Les cartes réseau connectées via USB sont désormais plafonnées à **200 Mo/s** en valeur absolue (la limite est imposée par le bus, elle n’évolue donc pas en fonction de la spécification nominale ; une carte USB 1 GbE atteint environ 80 Mo/s et n’est pas affectée). L’entrée `wire_ceiling_source` dans l’enregistrement des capacités l’indique explicitement, et `nic_is_usb` le signale. Vous pouvez passer outre cette limitation dans les deux cas avec `--wire-ceiling-mbps`.
- **L’admission tient compte du drain, et non de la distinction entre rafale complète et anneau.** Une rafale simultanée doit uniquement s’adapter au *retard transitoire* = `max(0, Σ per-cam arrival − host drain) × emit_window`, et non à la rafale entière. Sur une architecture hôte rapide / cam lente- (un hôte **PCIe**10G + 4 caméras 1 GbE : arrivée ≈ 320 Mo/s, débit de sortie ≈ 1 063 Mo/s), l’hôte évacue les données plus vite que les caméras ne les remplissent, le backlog est ≈ 0, donc l’émission simulée en pleine résolution**est autorisée**même si la rafale de 25 Mo dépasse l’anneau de 13,5 Mo. Placez ces mêmes quatre caméras derrière un adaptateur**USB**10 GbE et le débit de sortie est de 200 Mo/s, et non de 1 063 — l’arrivée dépasse le débit de vidange, et la perte se manifeste par des images corrompues plutôt que par une fréquence d’images réduite. Sur un hôte 1 GbE, le seuil DLThr de 31,25 Mo/s des caméras fait que l’arrivée dépasse le débit de vidange → le système**bloque** correctement (pour *cette* catégorie de blocage, réduisez la zone d’intérêt ou utilisez un binning ≥ 2). L’admissibilité est l’une des **deux** conditions de connexion — l’autre est le contrôle global de sursouscription ci-dessous.
- **Le nombre d’images par seconde (fps) prévu correspond à un plafond prudent en mode de récupération série.**La boucle de récupération de l’hôte extrait actuellement le tampon de chaque caméra**série**(~une fenêtre d’émission par caméra chacune), de sorte que le cycle est limité par `max(readout+emit, N × emit)`, l’émission par caméra étant plafonnée au**lien d’accès**de la caméra (1 GbE ≈ 80 Mo/s), et non à la liaison montante de l’hôte. Pour uncaméras en pleine résolution 12 bits, le**débit est d’environ 2,8 images par seconde**, ce qui correspond aux mesures de ~2,7 à 3,0. Le nombre d’images par seconde est délibérément**indépendant de l’exposition**, de sorte que, dans les scènes sombres, le débit réel peut descendre légèrement en dessous du plafond à mesure que l’exposition s’allonge. La récupération en série est le véritable limiteur de fps ; sa parallélisation ferait remonter le plafond vers le débit d’émission unique.
- **La sursouscription agrégée constitue un obstacle majeur à la connexion.**L’allocation de bande passante par caméra est plafonnée à**8 Mo/s**(`ARRAY_PER_CAM_FLOOR_BPS`) ; ainsi, une fois ce seuil atteint, la demande agrégée (`per_cam × N`) peut dépasser le**plafond de sécurité anti-collision du réseau**(`sustained × sim_emit_factor`). En pratique, les plafondssur 1 GbE :**6 caméras à 1 500 MTU, 9 avec des trames jumbo**. Ce plafond dépend uniquement de la bande passante physique et du seuil minimal — il est**indépendant de la taille des trames**, donc**le regroupement et une zone d’intérêt (ROI) plus petite NE SONT D’AUCUNE AIDE** (ils réduisent le nombre d’octets par *trame*, pas le nombre d’octets par *seconde* régulé par le GevSCPD) ; les seules solutions sont de réduire le nombre de caméras, d’utiliser des trames jumbo de bout en bout ou d’utiliser une carte réseau plus rapide. Le symptôme serait une perte de paquets GVSP, et non une réduction progressive du nombre d’images par seconde, c’est pourquoi `analyze-array` remet à zéro les valeurs d’images par seconde réalisables et affiche `**OVER-SUBSCRIBED**`, ainsi que `array-connect` avec une résolution fixe **refuse de se connecter** (sinon, le « walk-down » regroupe les images par lots, ce qui ne résout pas non plus ce type de blocage). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` ramène ce refus au niveau d’un avertissement sonore pour les tests en laboratoire — voir [Variables d’environnement](#environment-variables).

### État de santé du réseau de caméras — quel sous-système perd des trames

Le `GET /api/camera/array/<array_id>/capability` d’un tableau connecté contient un bloc
`health` actif, réévalué sur une fenêtre glissante de **10 secondes**. Il ventile la perte de trames
en deux causes nécessitant des corrections opposées, plutôt que de signaler un seul taux « incomplet »
qui n’en identifie aucune :

| Champ | Signification | Sous-système concerné |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (par port série) | La trame **est arrivée mais était structurellement incorrecte**— perte de paquets GVSP. |**Réseau** : budget de bande passante, rythme de transmission, anneau RX de la carte réseau, MTU |
| `never_arrived_rate_pct` (par port série) | La trame **n&#x27;est jamais arrivée**— la caméra ne s&#x27;est pas déclenchée, ou rien n&#x27;en est sorti. |**Déclenchement / synchronisation** : câble M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Le taux le plus faible de chaque caméra. | — |
| `per_cam_rate_pct` | Taux combiné d’incomplétude par caméra (les deux causes confondues). | — |
| `stable_for_seconds` | Durée pendant laquelle chaque caméra est restée en dessous de 0,01 %. | — |

Au-delà de 5 %, le backend enregistre une ligne `[array-health <id>] WARN` indiquant la division — lors de la
premier dépassement, lors d’un changement de niveau de gravité, une fois par minute tant que la situation persiste, et une fois lorsqu’
elle se résout. La moitié défectueuse affiche `[gvsp-corrupt <SN>]` lors du premier dépassement par caméra et
la raison, puis un récapitulatif toutes les 60 s. Chaque évaluation est toujours consignée dans le fichier journal du backend ;
les compteurs avancent à chaque tampon, indépendamment de ce qui est affiché.

Le même enregistrement indique le nombre sur lequel repose l’allocation totale :

| Champ | Signification |
| --- | --- |
| `wire_ceiling_mbps` | Le budget de bande passante soutenu de l&#x27;hôte actuellement en vigueur, Mo/s. |
| `wire_ceiling_source` | Origine de ce chiffre, en mots — par exemple `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` lorsque `--wire-ceiling-mbps` (ou le champ **Budget de câble** de l’interface graphique) le définit. |
| `nic_is_usb` | `true` pour un adaptateur Ethernet USB — voir la limite de 200 Mo/s ci-dessus. |

**Interprétation :** une valeur non nulle pour `gvsp_corrupt_rate_pct` avec `never_arrived_rate_pct` à 0
signifie que le déclenchement et la synchronisation du câble sont parfaits et que 100 % de la perte se situe au niveau du
— réduisez la valeur de `--wire-ceiling-mbps` et reconnectez-vous. Le schéma inverse indique plutôt un problème au niveau du
câble de synchronisation ou de la ligne de déclenchement.

> **`--target-fps` n’est pas le paramètre responsable des trames corrompues.** Le rythme de transmission GevSCPD est défini
> une seule fois lors de la connexion ; par conséquent, réduire la fréquence de déclenchement modifie le rapport cyclique et non le
> débit de rafales à émission simultanée. Une réduction mesurée de 5× de la demande n’a apporté aucune amélioration ;
> abaisser le plafond de débit de 240 à 200 Mo/s a fait passer le taux de trames corrompues de ce même équipement de 10,4 %
> à 0,00 %.

> **La réduction automatique en cours de transmission n’est pas disponible sur le micrologiciel TRI032S.** Une matrice en cours d’exécution
> ne peut pas corriger cela elle-même ; déconnectez-la puis reconnectez-la afin que le sélecteur de connexion puisse
> replanifier avec le nouveau plafond.

### Symptôme → solution

| Symptôme (Paramètres de la matrice / connexion / `analyze_array_network`) | Cause | Solution |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` réinitialisé à 32 (généralement après une mise à jour du pilote) | Définissez `ReceiveBufferLen`→256, `PendingReceives`→64 ; rouvrir le panneau (redémarrer le backend s’il a mis en cache l’ancienne taille de l’anneau) |
| Le redémarrage/l&#x27;arrêt se bloque ; par la suite, erreurs WMI `Invalid class`, la carte réseau ne s’active pas, les clés USB sont absentes | Ancien pilote Realtek USB 10 GbE de 2016 → écran bleu `0x9F` → coupures d’alimentation forcées- | Mettre à jour le pilote de la carte vers la version ≥ v10.67 (2026), puis réappliquer les paramètres de ring de réception ci-dessus |
| La connexion aboutit mais renvoie une résolution inférieure à la résolution native | Smart-prep a automatiquement-réduit automatiquement la trame pour l&#x27;adapter à la ligne | Mettre à niveau la liaison / accepter la réduction / `--force-tier slip-emit-and-capture` |
| La matrice signale un nombre d’images par seconde (fps) cible correct mais n’en fournit qu’une fraction ; `health.gvsp_corrupt_rate_pct` différent de zéro, `never_arrived_rate_pct` égal à 0 | Le budget de ligne déduit par l’hôte-indique ce qu’il supporte réellement (cas courant sur une carte Ethernet USB, une voie PCIe étroite ou une structure partagée) | Reconnectez-vous avec une valeur `--wire-ceiling-mbps` inférieure et vérifiez à nouveau le bloc d’intégrité. **Pas** `--target-fps` — Le rythme GevSCPD est fixé lors de la connexion |
| Caméras manquantes dans les groupes publiés ; `health.never_arrived_rate_pct` différent de zéro, `gvsp_corrupt_rate_pct` 0 | Chemin de déclenchement / synchronisation — les caméras ne se déclenchent pas, ce n’est pas un problème réseau | Vérifiez le câble de synchronisation M8 et `--line` ; vérifiez que chaque élément est activé (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` dépassé dans `analyze-array`, ou refus de connexion avec résolution bloquée (`array over-subscribes the wire`) | La demande totale par caméra (8 Mo/s minimum × N caméras) dépasse le plafond de bande passante sans collision — 6 caméras en pleine résolution sur 1 GbE à 1 500 MTU, 9 avec des trames jumbo | Moins de caméras, des trames jumbo de bout en bout ou une carte réseau plus rapide. **Le ROI/binning n&#x27;aidera PAS** (le plafond est indépendant de la taille des trames). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` passe outre sur le banc d&#x27;essai (accepte la perte de paquets) |

---

## `chloros-cli daq`

Commandes du capteur spectral. Deux catégories :
- **`pool-*`**— clients « thin » d’HTTPs qui pilotent le capteur via le pool persistant du backend.**Il s’agit du chemin pris en charge, et du seul présent dans l’CLI fournie.** Le backend gère le transport ; ainsi, l’interface graphique, les scripts CLI et SDK partagent tous un même descripteur actif au lieu de se disputer le port série.
- **Tout le reste**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — accès direct au matériel, documenté ci-dessous par souci d’exhaustivité. Ces éléments nécessitent le paquet `daq` Python, qui n’est**n’est inclus dans aucun artefact livré** : l’CLIe compilée l’exclut (`scripts/Build-CLI.ps1` définit `--nofollow-import-to=daq`, et les transports `pyserial` / `bleak` / `zeroconf` ne le contiennent pas), et le paquet PyPI SDK ne le contient pas non plus. Ils ne fonctionnent qu’à partir d’un checkout du code source ; considérez-les donc comme une voie de développement interne à MAPIR plutôt que comme quelque chose à utiliser.
- **`discover` / `list`** se situent à mi-chemin entre les deux : il s’agit de commandes matérielles directes issues d’un checkout du code source, mais sur une version livrée, elles se rabattent sur `pool-discover` et le backend effectue l’analyse. L’analyse fonctionne donc partout — ce qui est important car c’est le seul moyen de connaître l’adresse MAC BLE d’un DAQ-M.

> **`chloros-cli daq --help`** (ainsi que `-h` / `help`) répertorie les sous-commandes de `pool-*` — l’aide est délibérément redirigée vers le client du pool afin de refléter les commandes réellement exécutées. Si vous invoquez une sous-commande de matériel direct sur une version livrée, celle-ci se termine par une erreur explicite indiquant le paquet manquant et vous renvoyant vers `pool-*` ; rien ne se passe en silence. (`discover` / `list` font exception — elles redirigent vers `pool-discover` et fonctionnent tout simplement.)
>
> **Tout ce dont un client a besoin est accessible via `pool-*`** — se connecter, diffuser en continu, enregistrer des fichiers `.daq` calibrés et échanger des profils de capteurs. Le DAQ peut également être piloté depuis Python avec `chloros_sdk.connect_daq_sensor()`, qui utilise le même chemin partagé.

### Workflow de première connexion au capteur DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Référence `pool-*`

| Sous-commande | Objectif |
| --- | --- |
| `daq pool-connect` (smart-detect) | Ouvrir un capteur dans le pool du backend. |
| `daq pool-connect --port PORT` | DAQ-U sur un port série spécifique. |
| `daq pool-connect --ble` | DAQ-M via BLE, adresse MAC détectée automatiquement. |
| `daq pool-connect --mac MAC` | DAQ-M via BLE sur une adresse MAC connue (implique `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet sur un hôte connu. |
| `daq pool-connect --eth` | DAQ-E via Ethernet, hôte détecté automatiquement (mDNS + repli sur ARP ; fonctionne avec un cache ARP vide sur Windows et Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Réglage de la fenêtre d’intégration / de l’état AE. |
| `daq pool-connect --no-stream` | Connexion sans démarrage immédiat de la transmission (reprise avec `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil de correction de capacité. La valeur par défaut au niveau du backend est `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Analyser chaque transport à la recherche de capteurs auxquels vous pourriez vous connecter, sans vous connecter. **C&#x27;est permet de trouver l’adresse MAC BLE d’un DAQ-M.** `daq discover` / `daq list` sont automatiquement redirigés vers cette page dans les versions livrées. Les capteurs déjà ouverts dans le pool ne sont pas répertoriés — un DAQ-M connecté cesse de diffuser son annonce — ; utilisez donc `pool-list` pour ceux-ci. |
| `daq pool-list` | Afficher tous les capteurs du pool du backend. |
| `daq pool-disconnect --sensor-id ID [--all]` | Libérer. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Trames du spectre N les plus récentes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reprendre / mettre en pause le streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Démarrer / arrêter un enregistrement .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Changer le profil de correction de la capacité en cours d’exécution. |

### Sous-commandes matérielles directes (uniquement dans le code source — non incluses dans les versions livrées)

> Répertoriées par souci d’exhaustivité. Elles nécessitent le paquet `daq` « Python » ainsi que les commandes `pyserial` / `bleak` / `zeroconf`, dont aucune n’est fournie dans la version compilée CLI ni sur PyPI SDK — elles ne s’exécutent qu’à partir d’une extraction du code source MAPIR. **Si vous utilisez une version publiée Chloros, utilisez plutôt les commandes `pool-*` ci-dessus** ; elles couvrent la connexion, la diffusion en continu, l’enregistrement et la sélection des capteurs.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Ouvrir, se connecter à et piloter un projet Chloros enregistré (un dossier contenant `cameras.json` + `sensors.json` + `project.json`). Tout passe par le backend, ce qui garantit que l’interface graphique et CLI reflètent l’état identique du matériel.

### Référence des sous-commandes

| Sous-commande | Objectif |
| --- | --- |
| `project open PATH` | Afficher le manifeste des périphériques du projet (caméras, réseaux de caméras, capteurs). |
| `project devices PATH [--reconnect]` | Afficher la liste ou relancer la détection. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Connecter toutes les caméras / réseaux de caméras / capteurs enregistrés. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Capture unique à partir d’une caméra ou d’un ensemble nommé. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Rafale de N images à partir d’une caméra ou d’un réseau spécifié (`-n/--count` : valeur par défaut 5 ; `-i/--interval` : intervalle en secondes entre les images, valeur par défaut 0). Les rafales de réseaux suppriment les doublons des groupes synchronisés (surveillance de l&#x27;obsolescence) ; ainsi, un réseau à cycle partiel ne peut pas renvoyer N copies d’une même image ; affiche les résultats par itération. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Transfert du flux versdisque via une tâche de backend. `--poll-interval` = nombre de secondes entre les interrogations `/stats` (valeur par défaut : 2,0). |
| `project sensor read PATH NAME [--json]` | Dernière trame de spectre. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Enregistrer un fichier .daq. |
| `project run PATH RECIPE.yaml` | Exécuter une recette de capture YAML/JSON. `--dry-run` valide sans exécuter. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcule l&#x27;alignement pour un tableau — voir [le tableau des options ci-dessous](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Afficher le profil d’alignement actuel. |
| `project align clear PATH NAME` | Supprimer le profil mis en cache. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Décaler la transformation d’un esclave. |
| `project align export PATH NAME --to FILE` | Enregistrer le profil dans JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Charger un profil enregistré. |

#### Options `project align calibrate`

| Indicateur | Par défaut | Description |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Méthode d&#x27;alignement. **Ces orthographes diffèrent de `lattice align-calibrate`**, qui utilise les formes abrégées `orb` / `akaze` / `phase` ; les deux commandes ne sont pas interchangeables pour cet indicateur. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformer le modèle pour l’ajuster. |
| `--frames N` | `1` | Synchroniser les instantanés d’images pour obtenir une moyenne. |
| `--reference SN` | la caméra maître | Numéro de série de la caméra de référence ; tous les autres membres sont déformés pour s&#x27;aligner sur celle-ci. |
| `--max-features N` | `5000` | Limite du nombre de caractéristiques ORB. |
| `--ratio-threshold F` | `0.75` | Test du . |
| `--ransac-threshold-px F` | `3.0` | Seuil des points intérieurs de RANSAC. |
| `--min-matches N` | `15` | **Seuil de qualité** — rejeter la solution si le nombre de correspondances valides est inférieur à cette valeur. |
| `--max-reproj-err-px F` | `4.0` | **Seuil de qualité** — rejeter la solution si l&#x27;erreur de reprojection RMS dépasse cette valeur. |
| `--checkerboard RxC` | — | Géométrie de la carte pour `--method checkerboard`, par exemple `9x6`. |
| `--name PROFILE` | vide | Nom du profil intégré dans l’JSON enregistré. **Il ne s’agit pas du nom du tableau** — celui-ci correspond à la position `NAME`. |

Ces deux contrôles de qualité expliquent pourquoi un calibrage peut réussir la résolution tout en
refuser d’être enregistré : un profil qui échouerait à l’un ou l’autre de ces contrôles entraînerait silencieusement un mauvais alignement de toutes les
captures suivantes ; il est donc rejeté plutôt que conservé.

### Exemples

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

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

`project run RECIPE.yaml` accepte un fichier YAML ou JSON décrivant une séquence d’actions :

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
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
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

Actions prises en charge : `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. L’action `burst` requiert les paramètres `name` (obligatoire), `count` (valeur par défaut : 5), `interval` (en secondes, valeur par défaut 0), `output`, `format` et `settings` (identique à laconfiguration de la caméra, comme `apply`) ; les rafales de capteurs utilisent le même chien de garde de groupe fraîchement synchronisé que `project burst`.

Exécutez la commande suivante :

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Variables d’environnement

| Variable | Effet |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Remplace l’URL du backend (par défaut `http://127.0.0.1:5000`) — **prise en compte uniquement par les familles de commandes `lattice`, `project` et `daq pool-*`.** Les commandes principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) connectent la broche `http://127.0.0.1:<port>` et ignorent cette variable (le littéral IPv4 contourne la pénalité d&#x27;environ 2 s par requête liée à l&#x27;Windows `localhost`→`::1`), de sorte qu’elles ciblent toujours la machine locale. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` ramène le refus de connexion dû à la sursouscription du tableau (agrégé par-cam &gt; plafond de bande passante sans collision avec `pin_resolution`) en un « avertissement sonore et poursuite », acceptant la perte de paquets GVSP. À usage de benchmark uniquement — voir [Modèle de fps et de rafales de la matrice](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Définie par l’CLIe lui-même ; indique au backend d’activer le traitement parallèle. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ignore la sonde de secours GVSP (résultats ICMP uniquement). **Cela désactive les paquets jumbo, ça ne se contente pas simplement de réduire le volume des messages de log** — la caméra ne répond aux pings DF que jusqu’à 1 500 sur chaque chemin, cette sonde est donc la seule capable de détecter les paquets jumbo. Gain d’environ 1 s par caméra et par connexion ; coût d’environ 1,45× la bande passante maximale si le réseau *pouvait* prendre en charge les paquets jumbo. L’SDK vous avertit lorsque vous activez cette option. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixe la taille des paquets GVSP à N octets ; ignore complètement la détection. Préférez l’utilisation de lacommande (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) plutôt que de le définir de manière permanente : une taille fixe empêche l’adaptation au réseau en amont, et le fait de fixer la valeur à 9 000 sur un chemin incapable de prendre en charge les paquets « jumbo » entraîne un délai d’expiration de **chaque** capture avec `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Remplace le répertoire d’extraction « onefile » de Nuitka. L’CLI utilise automatiquement `/mnt/ssd/tmp` s’il est présent. |

---

## Codes de sortie

| Code | Signification |
| --- | --- |
| `0` | Succès. |
| `1` | Échec générique (la plupart des erreurs de sous-commandes). |
| `2` | Erreur d’argument. |
| `130` | Interrompu par Ctrl+C. |

---

## Conseils de dépannage

- **« Connexion requise »** → Exécutez `chloros-cli login EMAIL PASSWORD` une fois sur cette machine.
- **« Backend inaccessible »** → Lancez l’application de bureau Chloros, ou exécutez directement le binaire du backend (`chloros-backend`), ou vérifiez `CHLOROS_BACKEND_URL` s’il s’agit d’un accès à distance.
- **Les commandes `lattice` échouent avec le message « Pilotes de caméra LATTICE introuvables »** → Le runtime d’SDK Arena n’est pas installé ; l’CLI est fourni avec `win32api` intégré sur Windows, mais le runtime C fait partie du programme d’installation graphique.
- **La fenêtre « Array connect / Array Settings » affiche « FRAMES WILL DROP » ou « Reduce ROI to enable »** → La taille de l’anneau de réception de la carte réseau hôte est trop petite (elle est généralement réinitialisée à 32 après une mise à jour du pilote de la carte réseau). Voir [Configuration et réglage de la carte réseau hôte](#host-nic-setup--tuning-lattice-arrays) — configurer `ReceiveBufferLen=256`, `PendingReceives=64`.
- **La machine se bloque au redémarrage/à l’arrêt, puis WMI `Invalid class` / la carte réseau ne s’active pas / les clés USB sont manquantes** → Pilote obsolète de la carte USB 10 GbE provoquant `DRIVER_POWER_STATE_FAILURE` (écran bleu `0x9F`). Mettez à jour le pilote de la carte — voir [Configuration et optimisation de la carte réseau hôte](#host-nic-setup--tuning-lattice-arrays).
- **Avertissement concernant la zone d’échange (swap) sur Jetson** → Ajoutez une zone d’échange sur fichier ; la commande `CLI` affiche les commandes exactes ``fallocate`` / ``swapon``.
- **Commandes directes DAQ manquantes** → Attendu : le paquet `chloros-cli` fourni exclut délibérément le paquet `daq`, de sorte que seul `pool-*` est présent (l’SDK PyPI ne le propose pas non plus). Utilisez `pool-*`, qui pilote le même capteur via le backend, ou `chloros_sdk.connect_daq_sensor()` disponible sur Python.

---

## Voir aussi

- [Python Référence SDK](sdk-reference.md) — équivalent programmatique de chaque commande CLI.
- [Guide des capteurs DAQ](../daq/README.md) — câblage et étalonnage spécifiques aux capteurs.
- Documentation en ligne : `https://mapir.gitbook.io/chloros/cli`
