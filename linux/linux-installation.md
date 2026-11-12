# Installation sous Linux

Chloros est distribué pour Linux sous forme de paquets `.deb` qui installent l'interface CLI, le serveur backend et le SDK Python pour Python (pour le système `python3`, ainsi qu'un fichier « wheel » adapté à la version pour les environnements virtuels).

Les noms de fichiers des paquets indiquent la version et l’architecture : `chloros_1.3.0_amd64.deb` pour x86_64, et `chloros_1.3.0_arm64_jp6.deb` pour arm64 (NVIDIA Jetson, JetPack 6). Remplacez le nom du fichier que vous avez effectivement téléchargé dans les commandes ci-dessous.

{% hint style="info" %}
**Nouveauté de la version 1.3.0 : les programmes sont fournis décompressés.** Le backend et l’CLI sont installés sous forme de dossiers, `/usr/lib/chloros/backend/` et `/usr/lib/chloros/cli/`, au lieu de fichiers auto-extractibles individuels. Ils ne se décompressent plus dans un dossier temporaire à chaque lancement, ce qui leur permet de démarrer beaucoup plus rapidement (sur x86_64, le backend mettait auparavant environ 100 secondes à se décompresser à chaque démarrage) et n’ont plus besoin de plusieurs Go d’espace libre dans `/tmp`. La version amd64 `.deb` a également environ la moitié de sa taille initiale.
{% endhint %}

***

## Linux amd64 (x86_64)

### Configuration requise

| Exigence | Minimum | Recommandé |
| --- | --- | --- |
| **Distribution** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processeur** | x86_64 (Intel/AMD) | Intel Core i7 ou supérieur |
| **Mémoire (RAM)** | 8 Go | 16 Go ou plus |
| **Carte graphique** | Aucune (traitement par le processeur) | Carte graphique NVIDIA avec au moins 4 Go de VRAM (12 Go ou plus pour débloquer `GPU_PARALLEL`, 7 Go ou plus pour désactiver Texture Aware sur le chemin d’image unique) |
| **Stockage** | 2 Go d’espace libre | SSD avec au moins 10 Go d’espace libre |
| **Python** | Python 3.7 ou version ultérieure (pour le SDK) | Python 3.10 ou version ultérieure |

> **Ubuntu 20.04 et Debian 11 ne sont pas pris en charge.** La liste des dépendances de `.deb` est
> dérivée de ce à quoi le backend Chloros est réellement lié, et cela inclut une
> version de `libc6` plus récente que la glibc 2.31 fournie par les versions « focal » et « bullseye » ; par conséquent, `apt` refuse
> catégoriquement l’installation plutôt que de la laisser échouer ultérieurement lors de l’exécution.

### Installation

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Utilisez `apt install ./…`, et non `dpkg -i`.** `apt` résout les dépendances du paquet et installe également ce qu’il recommande : les prérequis du SDK Python, `python3-numpy` et `python3-requests`. `dpkg -i` n'installe ni l'un ni l'autre. L’interface CLI et le backend sont autonomes et fonctionnent sans ces éléments, mais `import chloros_sdk` en a besoin ; la sortie de l’installation indique ceux qui manquent. Conservez le fichier `./`, afin que `apt` installe le fichier plutôt que de rechercher un paquet portant ce nom. Le paquet déclare également `libgl1`, que l’OpenCV fourni charge et qui peut manquer dans une image de serveur minimale.
{% endhint %}

Vérifiez l’installation :

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Configuration requise

| Exigence | Minimum | Recommandé |
| --- | --- | --- |
| **Plateforme** | NVIDIA Jetson avec JetPack 6 | Jetson Orin NX 16 Go ou AGX Orin |
| **JetPack** | JetPack 6.x | Dernière version de JetPack 6 |
| **Mémoire (RAM)** | 8 Go (partagée entre GPU et CPU) | 16 Go+ partagés (12 Go+ constituent le seuil pour le traitement parallèle) |
| **Stockage** | 2 Go d’espace libre | SSD NVMe avec 10 Go+ d’espace libre |
| **Python** | Python 3.7+ (pour le SDK) | Python 3.10+ |

### Installation

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Même configuration que l’amd64 `.deb` (Python SDK, tampons de réception et services backend inclus), pour Jetson Orin / Orin NX / Orin Nano sur **JetPack 6** (L4T R36). Son PyTorch est la version CUDA de JetPack 6 pour Orin ; ainsi, le traitement GPU, y compris le débayériseur Texture Aware, s'exécute sur le GPU du Jetson. Le paquet ne contient pas CUDA en soi : il utilise les bibliothèques CUDA et cuDNN installées par JetPack 6 (`sudo apt install nvidia-jetpack` sur une image qui ne les contient pas). Lorsqu’aucun GPU CUDA n’est disponible, le traitement s’effectue sur le CPU et `--debayer texture-aware` utilise le débayeur standard (le backend enregistre une ligne `[DEBAYER] … needs a CUDA GPU …` expliquant pourquoi). Pour en savoir plus sur le comportement du Jetson en matière de mémoire, de gestion thermique et de déploiement sur le terrain, consultez le [Guide NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Installation du Python SDK (tous les systèmes Linux)**Le `.deb` installe le SDK pour vous.** Les deux paquets (amd64 et Jetson) l’installent pour le système `python3` ; ainsi, `import chloros_sdk` fonctionne immédiatement après l’installation, sans pip ni connexion réseau. Ses paquets (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` et les chargeurs de projet) sont placés dans `/usr/lib/python3/dist-packages`, qui appartient au paquet : une mise à jour les remplace, et la suppression du paquet les supprime. La version correspond toujours à l’interface CLI et au backend que vous avez installés.

Les prérequis propres au SDK, `python3-numpy` et `python3-requests`, sont **recommandés, mais pas obligatoires** par le package : `sudo apt install ./chloros_<version>_<arch>.deb` les installe, contrairement à `dpkg -i`. La sortie d’installation indique la version du SDK que `python3` importe, toute dépendance manquante, ainsi que tout fichier `chloros_sdk` présent ailleurs dans le chemin d’accès de `python3` qui remplace la copie du paquet (comme un fichier installé précédemment avec `pip install --user`).**Dans un virtualenv**, qui n’a pas accès aux paquets du système, installez-y le fichier wheel fourni (entièrement hors ligne) :

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Depuis PyPI** — uniquement pour une machine sur laquelle aucun paquet Chloros n’est installé, et dont le SDK pilote un backend sur une autre machine (voir [Mode backend distant](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)) :

```bash
pip install chloros-sdk
```

Options supplémentaires, pour une installation via pip (dans un virtualenv ou depuis PyPI) :

| Option | Commande | Ajoute |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pour la diffusion en direct de la progression |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pour le transport BLE (DAQ-M) |

Vérifiez le SDK :

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Le SDK Python Python communique avec le backend que `.deb` installe via une HTTP API locale (`http://127.0.0.1:5000`). `ChlorosLocal` et les aides à la connexion le lancent automatiquement lorsque cela est nécessaire ; le `connect_all()` d’un projet enregistré ne le fait jamais, donc, sur une machine sans interface graphique, exécutez le backend en tant que [service](#always-on-ptp-for-headless-hosts). Utilisez toujours l’adresse IPv4 littérale plutôt que `localhost` — `localhost` peut se résoudre en `::1` et prendre environ deux secondes par requête.
{% endhint %}

***

## Première configuration

### 1. Connexion

L'accès à l'CLI et au SDK nécessite un forfait Chloros+ payant (**Copper** ou supérieur), contrôlé côté serveur : un appelant déconnecté obtient `401 AUTH_REQUIRED`, et un appelant du forfait gratuit (Iron) obtient `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Les identifiants sont mis en cache dans `~/.chloros/user_session.json`.

{% hint style="info" %}
**Les mises à niveau conservent vos identifiants ; la désinstallation les efface.** L’installation d’une version plus récente de `.deb` par-dessus une installation existante (y compris via `chloros-cli update`) conserve `~/.chloros/user_session.json` et la licence mise en cache, ce qui permet aux tâches CLI/SDK sans interface graphique de continuer à fonctionner. La désinstallation du paquet (`sudo apt remove chloros`) efface les identifiants enregistrés et le cache de licence pour chaque utilisateur de la machine ; une nouvelle installation nécessite donc à nouveau `chloros-cli login`.
{% endhint %}

### 2. Vérifier l’état de votre licence

```bash
chloros-cli status
```

`chloros-cli status` fonctionne sur tous les niveaux (y compris la version gratuite) ; vous pouvez donc toujours voir pourquoi l’accès est disponible ou non.

### 3. Lancer les diagnostics système

```bash
chloros-cli selftest
```

Sept vérifications s’exécutent dans l’ordre, et la commande renvoie un code de sortie différent de zéro si l’une d’entre elles échoue :

| # | Vérification | Ce qu’elle vérifie |
| --- | --- | --- |
| 1 | **Version** | L'CLI indique sa version (`v1.3.0`). |
| 2 | **Port disponible** | Le port 5000 est libre, *ou* déjà occupé par un backend Chloros en bon état de fonctionnement (ce qui est considéré comme une réussite). |
| 3 | **Démarrage du backend** | Le binaire du backend se lance. |
| 4 | **Test de l'API (`/api/test`)** | Le backend répond `status: ok`. |
| 5 | **Informations système** | Affiche `GPU: <name>, CUDA: <bool>, PyTorch: <version>` à partir de `/api/system-info`. |
| 6 | **Modèles de débruitage** | Trouve des modèles `*.pth.enc` (sous Linux : `/usr/lib/chloros/models`). |
| 7 | **CUDA + Dénoyauteur**| La fonctionnalité « Texture Aware » est effectivement utilisable — nécessite CUDA**et** au moins un fichier de modèle. |

L'exécution se termine par `N/7 checks passed`, qui répertorie les éventuels échecs par nom.

### 4. Traitez votre premier ensemble de données

```bash
chloros-cli process ~/datasets/flight001
```

***

## Fichiers et répertoires

### Par utilisateur

Chloros conserve ses identifiants et sa configuration CLI dans un répertoire unique multiplateforme, **`~/.chloros/`** (sous Windows, `%USERPROFILE%\.chloros\`). Deux caches spécifiques à Linux suivent quant à eux les conventions XDG — ceux-ci respectent les chemins `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` lorsqu’ils sont définis.

| Chemin | Objectif |
| --- | --- |
| `~/.chloros/user_session.json` | Cache de session de connexion créé par `chloros-cli login` (conservé lors des mises à jour du paquet ; effacé lorsque le paquet est supprimé) |
| `~/.chloros/working_directory.txt` | Remplacement par défaut du dossier de projet (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Préférence de langue de l'interface CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Paramètre de langue partagé avec l'interface graphique Windows — une valeur `language` ici a priorité sur `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache d’une heure pour la vérification des mises à jour au démarrage sous Linux/Jetson |
| `~/.chloros/backend.log` | Journal du backend lorsque celui-ci a été lancé par la CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Paquets de calibrage LATTICE mis en cache par caméra, indexés par numéro de série et hachage du lot |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Remplacements facultatifs par l’utilisateur pour les profils de correction de la capacité DAQ |
| `~/.config/chloros/system_config.json` | Profil matériel mis en cache issu de l’adaptation dynamique des calculs (Dynamic Compute Adaptation) — le supprimer pour forcer une nouvelle détection du matériel |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Journaux du serveur backend, un fichier par lancement |
| `~/Documents/MAPIR/Chloros Projects/` | Dossier de projet par défaut lorsqu’aucune modification n’est définie |

### À l'échelle du système

| Chemin | Objectif |
| --- | --- |
| `/usr/bin/chloros-cli` | Script d'encapsulation — définit `LD_LIBRARY_PATH` pour les bibliothèques natives intégrées (et `TMPDIR` à `/mnt/ssd/tmp` lorsque ce dossier existe), puis exécute le binaire réel |
| `/usr/bin/chloros-backend` | Script d'encapsulation — identique pour le backend, qu’il marque comme une version de production installée |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | L’interface CLI et le backend compilés, installés sans décompression (nouveauté de la version 1.3.0) ; les programmes sont `cli/chloros-cli` et `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Scripts de lancement de deux lignes qui démarrent les programmes réels ci-dessus — conservés à ces emplacements car le SDK, la CLI et les anciens scripts les recherchent ici |
| `/usr/lib/chloros/arena_runtime/` | Environnement d’exécution Arena SDK requis par les caméras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modèles de débruitage chiffrés utilisés par le débayeur « Texture Aware » |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (ainsi que `lattice_sdk`, `lattice_cam`, `arena_api`, les chargeurs de projet) | Le SDK Python, installé pour le système `python3` et appartenant au paquet |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Wheel SDK SDK correspondant exactement à cette version, pour un virtualenv |
| `/usr/lib/chloros/exiftool` | Lanceur ExifTool intégré (lié par un lien symbolique à `/usr/local/bin/exiftool` uniquement si aucun exiftool système n’existe). À partir de la version 1.3.0, le script ExifTool est fourni avec sa propre bibliothèque Perl dans `/usr/lib/chloros/exiftool-dist/`, il fonctionne donc sans le paquet système `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Configuration du canal de mise à jour lue par `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Définit `net.ipv4.ip_unprivileged_port_start = 319` afin que le backend puisse se lier aux ports PTP sans droits root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Augmente la taille des tampons de réception des sockets du noyau (`net.core.rmem_max`, `net.core.rmem_default`) à 32 Mo pour les flux de caméras LATTICE, et les applique lors de l'installation (une valeur déjà définie à un niveau supérieur n’est pas modifiée). Avec les paramètres par défaut du noyau, les paquets des caméras sont rejetés par le noyau et un réseau de caméras perd des images et subit une baisse de la fréquence d’images ; `chloros-cli lattice network` signale les tampons lorsqu’ils sont faibles. Voir [Hôtes Linux — tampons de réception des sockets](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Dirige le chargeur dynamique vers `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Accorde à l’utilisateur connecté l’accès au pont série USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Service backend « toujours actif » à activer manuellement, qui exécute le backend sous l'identité de l'utilisateur que vous désignez (installé, **non activé**) — c'est celui-ci qu'il faut activer ; voir [PTP toujours actif](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Le même service exécutant le backend en tant que **root**(installé,**non activé**), conservé afin qu’une machine sur laquelle il a été activé continue de fonctionner |
| `/var/log/chloros/` | Le journal détaillé du service root, accessible au groupe `adm` (supprimé lors de la purge du paquet) |
| `/usr/share/applications/chloros-cli.desktop` | Entrée de menu de l'application « Chloros CLI » qui ouvre un terminal |

**Pare-feu (nouveau dans la version 1.3.0).** Si `ufw` ou `firewalld` est actif au moment de l’installation, le paquet ouvre les ports UDP `5002` (données DAQ-E), `5353` (découverte mDNS), `319` et `320` (PTP), et supprime ces règles lorsque le paquet est désinstallé. Si un pare-feu est installé mais inactif, le package n’ajoute rien et affiche la liste des ports à autoriser au cas où vous le mettriez en service ultérieurement. Il n’ouvre aucun port TCP : une mise à jour du micrologiciel DAQ-E, au cours de laquelle le capteur se reconnecte à cet ordinateur via TCP pour récupérer l’image, échoue avec le code d’erreur `… device never connected back on TCP <port> …` si un pare-feu actif bloque cette connexion.

## Emplacement de l’exécutable du backend

L’CLI et le SDK détectent automatiquement le backend :

| Composant | Chemin d’accès |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (un lanceur pour `/usr/lib/chloros/backend/chloros-backend`) |

Remplacez le chemin d’accès au backend à l’aide du drapeau CLI `--backend-exe` ou du paramètre de constructeur SDK `backend_exe`, et le port avec `--port` (valeur par défaut : `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` pointe vers les familles de commandes **`lattice`**,**`project`**et**`daq pool-*`** vers un backend distant. Les commandes principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) l’ignorent délibérément et ciblent toujours `http://127.0.0.1:<port>`.
{% endhint %}

***

## Caméras LATTICE et capteurs de lumière DAQ sous Linux

Les familles de commandes « live-hardware » fonctionnent toutes sous Linux (amd64 et Jetson) :

* **`chloros-cli lattice`** — détecte, connecte, configure et capture à partir des caméras LATTICE et des réseaux synchronisés. La commande `.deb` intègre le runtime Arena SDK dont elles ont besoin et l’enregistre auprès du chargeur dynamique.
* **`chloros-cli daq pool-*`** — permet de connecter des capteurs de lumière DAQ-U/M/E via le pool de backend, de diffuser des spectres calibrés, d’enregistrer des fichiers `.daq` et d’échanger des profils de capteur ; pour un DAQ-E, il permet également de modifier ses paramètres réseau, de le redémarrer et de mettre à jour son micrologiciel. `chloros-cli daq discover` recherche les capteurs que vous pouvez connecter, et `chloros-cli daq --help` répertorie les sous-commandes `pool-*`.
* **`chloros-cli project`** — exécute un projet enregistré (ses caméras, ses capteurs et ses paramètres de traitement) en mode sans interface graphique.
* **`chloros-cli time-sync`** — inspecter le « grandmaster » PTP sur lequel le backend Chloros s'exécute pour les caméras LATTICE et les capteurs DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` est requis par `pool-latest`, `pool-stream`, `pool-record` et `pool-set-cap` ; `pool-list` affiche les identifiants actuellement présents dans le pool.

{% hint style="info" %}
**`--eth-host` ignore la phase de découverte.** `pool-connect --eth` détecte un DAQ-E via mDNS, avec une solution de secours ARP fonctionnant à partir d'un cache ARP vide ; en transmettant l'adresse IP ou le nom d'hôte du capteur avec `--eth-host`, la connexion s'établit directement.
{% endhint %}

**Les autorisations série du DAQ-U** sont gérées par la règle udev installée (`uaccess` + groupe `dialout`). Si un capteur déjà branché reste inaccessible, rechargez les règles ou rebranchez-le :

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consultez la [référence CLI](../CLI.md) pour connaître l’ensemble complet des commandes.

### PTP toujours actif pour les hôtes sans affichage

Le paquet installe deux services systemd pour le backend, qui ne sont pas activés par défaut. Sur un Jetson sans écran ou un serveur devant maintenir la synchronisation temporelle PTP en fonctionnement continu pour les capteurs DAQ-E et les caméras LATTICE, ou lorsque des scripts SDK connectent des projets enregistrés (`connect_all()` ne démarre jamais de backend), activez le service **en tant qu’utilisateur** :

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` exécute le backend en tant que cet utilisateur : les captures qu’il écrit lui appartiennent, son journal détaillé se trouve dans son `~/.cache/chloros/logs`, et il utilise son `chloros-cli login` (`~/.chloros`), vous devez donc d’abord vous connecter une fois en tant que cet utilisateur. L’installation du paquet affiche cette commande avec votre nom d’utilisateur prérempli.

En l’absence de service, le backend (et avec lui le grand maître PTP) ne fonctionne que lorsqu’il a été lancé — par une commande `chloros-cli` ou un script SDK.

L’unité lie le backend aux paramètres d’environnement `127.0.0.1:5000` (paramètres d’environnement `CHLOROS_HOST` / `CHLOROS_PORT` au sein de l’unité ; à remplacer par `sudo systemctl edit chloros-backend@.service`) et le redémarre en cas d’échec après 5 secondes.

**Le service root.**`chloros-backend.service` exécute le backend en tant que**root**. Il reste installé afin qu’une machine sur laquelle il a été activé continue de fonctionner, mais tout ce qu’écrit un backend root appartient à root ; un utilisateur ne peut donc pas supprimer ses propres captures ; son journal se trouve dans `/var/log/chloros`, accessible au groupe `adm`. Pour passer à votre compte utilisateur :

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Le backend d’un service est partagé et n’est jamais arrêté par un client.** Les scripts `chloros-cli` et SDK l’utilisent tel quel : `process` s’exécute dessus, en chargeant son propre projet à la place du dernier. Là où la CLI, dans le cas contraire, redémarrerait le backend (`--restart`, ou un `process` dont l’importation n’a rien trouvé), elle nomme l’unité, affiche `sudo systemctl restart <unit>` et quitte `1`, car un backend arrêté de cette manière resterait à l’arrêt. Redémarrez-le vous-même avec `sudo systemctl restart chloros-backend@$USER.service`. Le backend d’un service ne s’arrête jamais de lui-même lorsqu’il est inactif, et `chloros-cli backend stop` le refuse, en indiquant la commande `systemctl stop`.**Comment PTP obtient ses ports.** PTP utilise les ports UDP 319 et 320, tous deux situés en dessous du seuil normal de 1 024 pour les ports privilégiés. La commande `postinst` du paquet écrit `/etc/sysctl.d/60-chloros-ptp.conf` avec `net.ipv4.ip_unprivileged_port_start = 319`, ce qui permet au backend de s’y lier tout en s’exécutant sous votre compte utilisateur. À partir de la version 1.3.0, le paquet ne définit aucune capacité de fichier (`setcap`) sur le backend, et `libcap2-bin` n’est plus une dépendance : sur le backend décompressé, une capacité de fichier l’empêcherait de démarrer, ne l’ajoutez donc pas vous-même.

***

## Exemples de scripts Bash

{% hint style="info" %}
**Codes de sortie adaptés aux scripts.**`chloros-cli process` renvoie `0` en cas de réussite et**une valeur non nulle en cas d’échec — y compris lors d’une exécution ayant demandé des produits d’imagerie mais n’en ayant généré aucun** (il affiche `Processing finished but wrote no image products.` et indique le nom du dossier du projet ainsi que les causes habituelles). Les exécutions réussies indiquent le nombre de produits d’image générés (`Image products written: N`, plus `(plus M clamp masks)` lorsque des masques ont été générés). Codes de sortie : `0` en cas de réussite, `1` en cas d’échec, `2` en cas d’erreur d’argument, `130` en cas d’interruption.
{% endhint %}

### Traiter plusieurs ensembles de données

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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Il existe exactement quatre valeurs `--format` valides, et elles contiennent des espaces — veillez à toujours les mettre entre guillemets :

| Valeur `--format` | Dossier de sortie |
| --- | --- |
| `TIFF (16-bit)` *(par défaut)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepte `standard` (par défaut) ou `texture-aware` (Chloros+).

### Traitement automatisé avec Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemple d'SDK Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Dépannage

### CLI introuvable après l'installation

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Le backend ne démarre pas après l'ajout de capacités de fichiers

N'exécutez pas `setcap` sur le backend. À partir de la version 1.3.0, le paquet ne définit aucune capacité de fichier, et les ports 319/320 de PTP proviennent du paramètre `sysctl` décrit dans la section [Comment PTP obtient ses ports](#always-on-ptp-for-headless-hosts). Une capacité de fichier place le backend décompressé en mode d’exécution sécurisée, dans lequel il ne peut pas trouver ses propres bibliothèques, ce qui l’empêche de démarrer. Si vous en avez ajouté une, supprimez-la :

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP ne démarre pas / ne parvient pas à se lier au port 319

Vérifiez que le seuil des ports non privilégiés a bien été abaissé, et réappliquez-le pour le démarrage en cours si ce n’est pas le cas :

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Vérifiez ensuite le grandmaster :

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Les commandes `lattice` échouent avec le message « Pilotes de caméra LATTICE introuvables »

Le backend n’a pas pu charger le runtime de la caméra LATTICE avec lequel il est installé. Toutes les commandes `lattice` s’exécutent sur le backend ; l’CLI elle-même ne contient aucun pilote de caméra. Réinstallez Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): la commande `.deb` installe le runtime à côté du backend, dans `/usr/lib/chloros/arena_runtime`, et l'enregistre auprès du chargeur dynamique. Pour vérifier cet enregistrement :

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Les journaux du backend relatifs à l'échec du lancement se trouvent dans `~/.cache/chloros/logs/`. Si le port 5000 appartient à un service backend (`systemctl status chloros-backend@$USER.service`), redémarrez le service plutôt que de tuer son processus.

### CUDA non détecté

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

Le fichier `chloros-cli selftest` signale la même chose sur une seule ligne : `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Bibliothèques partagées manquantes

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Démarrage lent sur les systèmes à carte SD

À partir de la version 1.3.0, le backend et l'CLI sont installés sous forme décompressée ; ils ne s'extractent donc plus dans un répertoire temporaire à chaque lancement. Les fichiers temporaires sont automatiquement placés dans `/mnt/ssd/tmp` si ce dossier existe ; sinon, vous pouvez les rediriger vers un système de fichiers rapide :

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Mise à jour d'Chloros sous Linux

La commande `update` est réservée à Linux/Jetson. Elle vérifie la version publiée dans le canal de mise à jour configuré dans `/etc/chloros/update.conf` et propose de télécharger et d’installer la version correspondante `.deb` :

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Sous Linux/Jetson, l’CLI effectue également une vérification de mise à jour non bloquante à chaque démarrage (résultat mis en cache pendant une heure dans `~/.chloros/update_cache.json`) et affiche `Update available: vX.Y.Z` lorsqu’une version plus récente existe. Vos paramètres, projets et identifiants de connexion sont conservés après une mise à jour.

La mise à niveau du paquet arrête tous les services backend pendant son installation ; ainsi, un script en cours d’exécution perd ses caméras et ses capteurs, puis redémarre chacun de ceux qui sont activés (un service lancé manuellement mais jamais activé reste arrêté, comme après un redémarrage).

## Désinstallation

```bash
sudo apt remove chloros
```

La désinstallation arrête et désactive tous les services backend, rétablit la limite par défaut des ports non privilégiés (1024), supprime le fichier de configuration du tampon de réception (les valeurs en cours de fonctionnement sont conservées jusqu’au redémarrage), le lien symbolique vers l’outil exiftool fourni, la configuration du chargeur Arena, les règles de pare-feu ajoutées par le paquet et le SDK Python qu’il a installé, et efface les identifiants mis en cache. Vos projets et vos fichiers de données `~/.chloros/` ne sont pas modifiés. `sudo apt purge chloros` supprime également `/var/log/chloros`.

***

## Étapes suivantes

* [Guide NVIDIA Jetson](nvidia-jetson-guide.md) — Optimisation et déploiement spécifiques à Jetson
* [CLI : ligne de commande](../CLI.md) — le guide de l’interface de ligne de commande
* [API : Python SDK](../api-python-sdk.md) — le guide du SDK
* [Référence CLI](../reference/cli-reference.md) et [Référence SDK](../reference/sdk-reference.md) — listes exhaustives des commandes et des API pour la version 1.3.0
* [Adaptation dynamique des ressources de calcul](../processing-architecture/dynamic-compute-adaptation.md) — comment Chloross s'adapte à votre matériel
