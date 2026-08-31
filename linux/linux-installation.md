# Installation de Linux

Chloros est distribué pour Linux sous forme de paquets `.deb` qui installent CLI et le serveur backend. Le paquet Python SDK est un paquet pip distinct (également inclus dans `.deb` sous la forme d’un fichier wheel dont la version correspond).

Les noms de fichiers des paquets indiquent la version et l’architecture : `chloros_1.2.0_amd64.deb` pour x86_64, et `chloros_1.2.0_arm64_jp6.deb` pour les versions JetPack 6 de Jetson. Remplacez le fichier que vous avez effectivement téléchargé dans les commandes ci-dessous.

***

## Linux amd64 (x86_64)

### Configuration requise

| Exigence | Minimum | Recommandé |
| --- | --- | --- |
| **Distribution** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processeur** | x86_64 (Intel/AMD) | Intel Core i7 ou supérieur |
| **Mémoire (RAM)** | 8 Go | 16 Go ou plus |
| **Carte graphique** | Aucune (traitement par le processeur) | Carte graphique NVIDIA avec au moins 4 Go de VRAM (12 Go ou plus pour débloquer `GPU_PARALLEL`, 7 Go ou plus pour désactiver Texture Aware sur le chemin d&#x27;image unique) |
| **Stockage** | 2 Go d&#x27;espace libre | SSD avec au moins 10 Go d&#x27;espace libre |
| **Python** | Python 3.7+ (pour SDK) | Python 3.10+ |

> **Ubuntu 20.04 et Debian 11 ne sont pas pris en charge.** La liste des dépendances de `.deb` est
> dérivée de ce à quoi le backend Chloros est effectivement lié, ce qui inclut
> `libc6 (>= 2.34)`. Focal et Bullseye intègrent tous deux glibc 2.31 ; par conséquent, `apt` refuse
> catégoriquement l&#x27;installation plutôt que de la laisser échouer plus tard lors de l&#x27;exécution.

### Installation

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` ne résout pas les dépendances. S’il signale des paquets manquants, `sudo apt-get install -f` (ou `sudo apt --fix-broken install`) termine l&#x27;installation — il s&#x27;agit d&#x27;un processus normal, et non d&#x27;une erreur.
{% endhint %}

Vérifiez l’installation :



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Configuration requise

| Exigence | Minimum | Recommandé |
| --- | --- | --- |
| **Plateforme** | NVIDIA Jetson avec JetPack 6 | Jetson Orin NX 16 Go ou AGX Orin |
| **JetPack** | JetPack 6.x | Dernière version de JetPack 6 |
| **Mémoire (RAM)** | 8 Go (partagée entre GPU et CPU) | 16 Go+ partagés (12 Go+ est le seuil pour les workers GPU parallèles) |
| **Stockage** | 2 Go d’espace libre | SSD NVMe avec 10 Go+ d’espace libre |
| **Python** | Python 3.7+ (pour SDK) | Python 3.10+ |

### Installation

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Même configuration que le `.deb` pour amd64, avec une version CUDA optimisée pour les Jetson Orin / Orin NX / Orin Nano. Pour en savoir plus sur la mémoire, la gestion thermique et le comportement en environnement de production des Jetson, consultez le [Guide NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Installation de Python et SDK (tous les Linux)

SDK est un client purement Python HTTP pour le backend ; par conséquent, le même paquet fonctionne sur amd64 et arm64. Deux sources :**Depuis PyPI** — la version stable publiée :

```bash
pip install chloros-sdk
```

**À partir du fichier wheel fourni** — compatibilité garantie avec le backend que vous venez d’installer (utilisez cette option si votre version est plus récente que celle de PyPI) :

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Les distributions conformes à la PEP 668** (Ubuntu 23.10+, Debian 12+) refusent les installations pip à l&#x27;échelle du système. Utilisez `pip install --user …`, un environnement virtuel ou `sudo pip install --break-system-packages …`. Le programme d&#x27;installation du paquet n&#x27;installe jamais automatiquement SDK dans votre système Python — ce choix vous appartient.
{% endhint %}

Options supplémentaires :

| Option | Commande | Ajoute |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pour la diffusion en direct de la progression |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pour le transport BLE (DAQ-M) |

Vérifiez le fichier SDK :

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Le fichier `.deb` installe les fichiers Chloros, CLI et le backend. Le Python SDK communique avec ce backend via un réseau local HTTP API (`http://127.0.0.1:5000`) et le démarre automatiquement en cas de besoin. Utilisez toujours l’adresse IPv4 littérale plutôt que `localhost` — `localhost` pouvant être résolu en `::1` et prendre environ deux secondes par requête.
{% endhint %}

***

## Première configuration

### 1. Se connecter

L&#x27;accès à CLI et SDK nécessite un forfait payant Chloros+ (**Copper** ou supérieur), appliquée côté serveur : un appelant déconnecté obtient `401 AUTH_REQUIRED`, et un appelant du niveau gratuit (Iron) obtient `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Les identifiants sont mis en cache dans `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Vous devez vous reconnecter après chaque installation ou mise à jour.** Le script `prerm` du package efface délibérément `~/.chloros/user_session.json` et la licence mise en cache pour chaque utilisateur de la machine, afin qu&#x27;une nouvelle version revalide toujours la licence au lieu de se fier à un cache obsolète.
{% endhint %}

### 2. Vérifier l&#x27;état de votre licence

```bash
chloros-cli status
```

`chloros-cli status` fonctionne sur tous les niveaux (y compris la version gratuite), ce qui vous permet de toujours comprendre pourquoi l&#x27;accès est disponible ou non.

### 3. Lancer les diagnostics système

```bash
chloros-cli selftest
```

Sept vérifications s’exécutent dans l’ordre, et la commande renvoie un code de sortie différent de zéro si l’une d’entre elles échoue :

| # | Vérification | Ce qu’elle vérifie |
| --- | --- | --- |
| 1 | **Version** | CLI indique sa version (`v1.2.0`). |
| 2 | **Port disponible** | Le port 5000 est libre, *ou* un backend Chloros en bon état y a déjà répondu (ce qui est considéré comme une réussite). |
| 3 | **Démarrage du backend** | Le binaire du backend se lance. |
| 4 | **Test API (`/api/test`)** | Le backend répond `status: ok`. |
| 5 | **Informations système** | Affiche `GPU: <name>, CUDA: <bool>, PyTorch: <version>` à partir de `/api/system-info`. |
| 6 | **Modèles de débruitage** | Trouve les modèles `*.pth.enc` (sur Linux : `/usr/lib/chloros/models`). |
| 7 | **CUDA + Dénoyauteur**| La fonction « Texture Aware » est effectivement utilisable — nécessite CUDA**et** au moins un fichier de modèle. |

L&#x27;exécution se termine avec `N/7 checks passed`, en répertoriant les éventuels échecs par nom.

### 4. Traiter votre premier ensemble de données

```bash
chloros-cli process ~/datasets/flight001
```

***

## Fichiers et répertoires

### Par utilisateur

Chloros conserve ses identifiants et sa configuration CLI dans un répertoire unique multiplateforme, **`~/.chloros/`** (sur Windows, `%USERPROFILE%\.chloros\`). Deux caches spécifiques à Linux respectent en revanche les conventions XDG — ceux-ci prennent en compte les paramètres `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` lorsqu’ils sont définis.

| Chemin | Objectif |
| --- | --- |
| `~/.chloros/user_session.json` | Cache de session de connexion créé par `chloros-cli login` (effacé à chaque installation ou mise à jour de paquet) |
| `~/.chloros/working_directory.txt` | Remplacement du dossier de projet par défaut (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Préférence de langue CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Paramètre de langue partagé avec l&#x27;interface graphique Windows — un `language` a ici priorité sur `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache d’une heure pour la vérification des mises à jour au démarrage de Linux/Jetson |
| `~/.chloros/backend.log` | Journal du backend lorsque celui-ci a été lancé par le CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Paquets de calibration LATTICE mis en cache par caméra, indexés par numéro de série et hachage du bundle |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Remplacements facultatifs par l’utilisateur pour les profils de correction de capacité DAQ |
| `~/.config/chloros/system_config.json` | Profil matériel mis en cache issu de l’adaptation dynamique des calculs (Dynamic Compute Adaptation) — le supprimer pour forcer une nouvelle détection du matériel |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Journaux du serveur backend, un fichier par lancement |
| `~/Chloros Projects/` | Dossier de projet par défaut lorsqu’aucune redéfinition n’est définie |

### À l’échelle du système

| Chemin | Objectif |
| --- | --- |
| `/usr/bin/chloros-cli` | Script d&#x27;encapsulation — définit `LD_LIBRARY_PATH` pour les bibliothèques natives intégrées, puis exécute le binaire réel |
| `/usr/bin/chloros-backend` | Script d&#x27;encapsulation — identique, avec en plus `CHLOROS_PRODUCTION=1` afin que le module d&#x27;authentification du backend ne puisse jamais se désactiver silencieusement |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Les binaires compilés |
| `/usr/lib/chloros/arena_runtime/` | Environnement d’exécution Arena SDK requis par les caméras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modèles de débruitage chiffrés utilisés par le débayeur « Texture Aware » |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK : roue correspondant exactement à cette version |
| `/usr/lib/chloros/exiftool` | exiftool intégré (liée par un lien symbolique à `/usr/local/bin/exiftool` uniquement si aucun exiftool système n&#x27;existe) |
| `/etc/chloros/update.conf` | Mise à jour de lade la configuration du canal lue par `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Configure `net.ipv4.ip_unprivileged_port_start = 319` afin que le backend puisse se lier aux ports PTP sans droits root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Dirige le chargeur dynamique vers `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Accorde à l&#x27;utilisateur connecté l&#x27;accès au pont série USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Service backend toujours actif (installé, **non activé**) |
| `/usr/share/applications/chloros-cli.desktop` | Entrée de menu d’application « Chloros CLI » permettant d’ouvrir un terminal |

## Emplacement de l&#x27;exécutable du backend

Les composants CLI et SDK détectent automatiquement le backend :

| Composant | Chemin d&#x27;accès |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Remplacez le chemin d’accès au backend à l’aide du drapeau `--backend-exe` CLI ou du paramètre de constructeur `backend_exe` SDK, et le port à l’aide de `--port` (par défaut `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` pointe vers les familles de commandes **`lattice`**,**`project`**, et**`daq pool-*`** vers un backend distant. Les commandes principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) l&#x27;ignorent délibérément et ciblent toujours `http://127.0.0.1:<port>`.
{% endhint %}

***

## Caméras LATTICE et capteurs de lumière DAQ sur Linux

Les familles de commandes « live-hardware » fonctionnent toutes sur Linux (amd64 et Jetson) :

* **`chloros-cli lattice`** — détecte, se connecte, configure et capture à partir des caméras LATTICE et des matrices synchronisées. Le modèle `.deb` intègre le runtime Arena SDK dont elles ont besoin et l’enregistre auprès du chargeur dynamique.
* **`chloros-cli daq pool-*`** — se connecter aux capteurs de lumière DAQ-U/M/E via le pool de backends, diffuser des spectres calibrés et enregistrer des fichiers `.daq`. La version compilée CLI ne contient que la famille `pool-*` : `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — exécute un projet enregistré (ses caméras, ses capteurs et ses paramètres de traitement) en mode sans interface graphique.
* **`chloros-cli time-sync`** — inspecter le « grandmaster » PTP sur lequel le backend Chloros s&#x27;exécute pour les caméras LATTICE et les capteurs DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` est requis par `pool-latest`, `pool-stream`, `pool-record` et `pool-set-cap` ; `pool-list` affiche les identifiants actuellement disponibles dans le pool.

{% hint style="info" %}
**Privilégiez `--eth-host` pour la première connexion DAQ-E sur une machine multi-homed.** La détection automatique parcourt le mDNS et peut ne pas détecter l&#x27;interface du capteur en raison d&#x27;un cache ARP vide ; ainsi, la première connexion `pool-connect --eth` après le démarrage peut échouer même si le capteur fonctionne parfaitement. En indiquant l&#x27;adresse IP ou le nom d&#x27;hôte du capteur, la détection est entièrement contournée.
{% endhint %}

**Les autorisations série DAQ-U** sont gérées par la règle udev installée (`uaccess` + groupe `dialout`). Si un capteur déjà branché reste inaccessible, rechargez les règles ou rebranchez-le :

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consultez la [référence CLI](../CLI.md) pour connaître l’ensemble complet des commandes.

### PTP toujours actif pour les hôtes sans interface graphique

Lors de la première installation, l&#x27;unité systemd `chloros-backend.service` est générée mais **n&#x27;est pas activée**. Sur un Jetson sans écran ou un serveur devant maintenir la synchronisation temporelle PTP en fonctionnement continu pour les capteurs DAQ-E et les caméras LATTICE, activez-la :

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Sans cela, le PTP ne fonctionne que lorsque le backend Chloros est en cours d’exécution — c’est-à-dire pendant une session CLI/SDK active.

L’unité lie le backend à `127.0.0.1:5000` (paramètres d’environnement `CHLOROS_HOST` / `CHLOROS_PORT` au sein de l’unité ; à remplacer par `sudo systemctl edit chloros-backend.service`) et le redémarre en cas d’échec après 5 secondes.

**Comment PTP obtient ses ports.** PTP utilise les ports UDP 319/320, tous deux situés en dessous du seuil normal de 1 024 pour les ports privilégiés. Le paquet `postinst` écrit `/etc/sysctl.d/60-chloros-ptp.conf` avec `net.ipv4.ip_unprivileged_port_start = 319`, ce qui permet au backend de s’y lier tout en s’exécutant sous votre compte utilisateur. Il applique également le fichier `setcap cap_net_bind_service,cap_net_raw=+ep` au binaire du backend par mesure de sécurité — c’est pourquoi `libcap2-bin` est une dépendance déclarée du paquet.***

## Exemples de scripts Bash

{% hint style="info" %}
**Codes de sortie adaptés aux scripts.**`chloros-cli process` renvoie `0` en cas de réussite et**une valeur non nulle en cas d’échec — y compris lors d’une exécution ayant demandé des produits d’imagerie mais n’en ayant généré aucun** (il affiche `Processing finished but wrote no image products.` et indique le nom du dossier du projet ainsi que les causes habituelles). Les exécutions réussies signalent le nombre de produits d’image générés (`Image products written: N`). Codes de sortie : `0` (succès), `1` (échec), `2` (erreur d&#x27;argument), `130` (interruption).
{% endhint %}

### Traitement de plusieurs jeux de données

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Traitement avec des paramètres personnalisés

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Les valeurs `--format` valides sont au nombre de quatre exactement, et elles contiennent des espaces — veillez à toujours les mettre entre guillemets :

| Valeur `--format` | Dossier de sortie |
| --- | --- |
| `TIFF (16-bit)` *(par défaut)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepte `standard` (par défaut) ou `texture-aware` (Chloros+).

### Traitement automatisé avec Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Exemple

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Dépannage

### CLI introuvable après l&#x27;installation

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Autorisation refusée

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### « setcap failed » pendant l&#x27;installation

Le fichier `.deb` applique `cap_net_bind_service` à `/usr/lib/chloros/chloros-backend` afin de pouvoir lier les ports PTP 319/320 sans droits root. Si `libcap2-bin` manquait au moment de l&#x27;installation, l&#x27;appel est ignoré. Installez-le et réinstallez le paquet :

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP ne démarre pas / Impossible de lier le port 319

Vérifiez que le seuil des ports non privilégiés a bien été abaissé, et réappliquez-le pour le démarrage en cours si ce n’est pas le cas :

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Vérifiez ensuite le grand maître :

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### « Pilotes de caméra LATTICE introuvables »

Le runtime Arena SDK ne peut pas être résolu. Vérifiez que la configuration du chargeur écrite par le paquet est présente et à jour :

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Échec du démarrage du backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Les journaux du backend relatifs à l’échec du démarrage se trouvent dans `~/.cache/chloros/logs/`.

### CUDA non détecté

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` signale la même chose en une seule ligne : `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Bibliothèques partagées manquantes

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Démarrage lent sur les systèmes à carte SD

Les binaires compilés s’extraient automatiquement dans un répertoire temporaire à chaque lancement. Si `/mnt/ssd/tmp` existe, Chloros l’utilise automatiquement ; sinon, configurez `TMPDIR` sur un système de fichiers rapide :

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Mise à jour de Chloros sur Linux

La commande `update` est réservée à Linux/Jetson. Elle vérifie la version publiée dans le canal de mise à jour configuré à `/etc/chloros/update.conf` et propose de télécharger et d&#x27;installer la version correspondante `.deb` :

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Sur Linux/Jetson, la commande CLI effectue également une vérification de mise à jour non bloquante à chaque démarrage (résultat mis en cache pendant une heure dans `~/.chloros/update_cache.json`) et affiche `Update available: vX.Y.Z` lorsqu’une version plus récente est disponible. Vos paramètres et vos projets sont conservés après la mise à jour ; vous devrez vous reconnecter par la suite.

## Désinstallation

```bash
sudo apt remove chloros
```

La désinstallation arrête `chloros-backend.service`, rétablit la limite par défaut des ports non privilégiés (1024), supprime le lien symbolique vers l’outil exiftool fourni et la configuration du chargeur Arena, et efface les identifiants mis en cache. Vos projets et vos fichiers de données `~/.chloros/` ne sont pas modifiés.

***

## Étapes suivantes

* [Guide NVIDIA Jetson](nvidia-jetson-guide.md) — Optimisation et déploiement spécifiques à Jetson
* [CLI : Ligne de commande](../CLI.md) — le guide CLI
* [API : Python SDK](../api-python-sdk.md) — le guide SDK
* [Référence CLI](../reference/cli-reference.md) et [Référence SDK](../reference/sdk-reference.md) — listes exhaustives des commandes/API pour la version 1.2.0
* [Adaptation dynamique des ressources de calcul](../processing-architecture/dynamic-compute-adaptation.md) — comment Chloros s&#x27;adapte à votre matériel
