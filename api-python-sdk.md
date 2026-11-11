# API : SDK pour Python

{% hint style="info" %}
**Vous recherchez l'API complète ?** Cette page est un tutoriel pratique. Toutes les classes publiques, méthodes, signatures exactes et exemples copiables-collables se trouvent dans la [Référence SDK](reference/sdk-reference.md), optimisée pour les assistants IA.**Vous travaillez avec un assistant IA ?** Collez cette URL dans le chat pour qu’il dispose de l’API complète et à jour de Chloros 1.3.0 :

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Chaque page de ce manuel est disponible au format Markdown brut sous la forme de son slug en minuscules + `.md`, et l’intégralité du manuel est indexée à l’adresse `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

Le **Chloros Python SDK** (`chloros-sdk` sur PyPI) permet de contrôler toutes les fonctionnalités de l’application de bureau depuis Python : traitement d’images par lots, contrôle en temps réel des caméras LATTICE et des réseaux de caméras, sessions d’acquisition de données (DAQ) avec capteurs de lumière, et automatisation des projets enregistrés. Il s'agit d'une couche mince superposée au même backend local que celui utilisé par l'interface graphique et la CLI (HTTP sur `127.0.0.1:5000`), de sorte que le comportement est identique sur les trois interfaces.

## Installation

Le SDK est fourni avec le package de bureau Chloros, qui fournit également le backend de traitement et les environnements d’exécution matériels, et dont la version correspond toujours à celle de ce dernier.

**Étape 1 — Installez Chloros.** Windows : exécutez le programme d’installation de la version de bureau (chemin par défaut : `C:\Program Files\Chloros\`) à partir de la page [Téléchargement](download.md). Linux : installez le paquet `.deb` avec `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Installation sous Linux](linux/linux-installation.md)).**Étape 2 — Vérification du SDK** (Python 3.7+). Il n’y a généralement rien à installer :

* **Windows :** le programme d’installation installe le fichier « wheel » du SDK fourni dans votre Python système (si Python n’est pas installé, cette étape est ignorée).
* **Linux :** le paquet `.deb` installe le SDK correspondant au système `python3`, dans `/usr/lib/python3/dist-packages`. Ses prérequis, `python3-numpy` et `python3-requests`, sont recommandés par le paquet : `apt install` les inclut, contrairement à `dpkg -i`.
* **Un virtualenv** ne voit pas les paquets du système ; installez-y le fichier wheel fourni : `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` sous Linux, ou le fichier `chloros_sdk-*.whl` situé dans le dossier `sdk` du dossier d'installation Windows (`C:\Program Files\Chloros\sdk\` par défaut).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Sur une machine ne disposant pas du paquet Chloros, dont le SDK pilote un backend sur une autre machine (voir [Ce qu’apporte une installation via pip uniquement](#what-a-pip-only-install-gets)), installez-le depuis PyPI : `pip install chloros-sdk`. PyPI est mis à jour à chaque nouvelle version, il correspond donc à la dernière version stable.

**Étape 3 — Connectez-vous une fois par machine :**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Les identifiants sont mis en cache dans `~/.chloros/` (sur les deux plateformes). Sous Windows, vous pouvez également vous connecter via l’onglet « Utilisateur » <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> de l’application de bureau. Le SDK nécessite un abonnement payant Chloros+ — voir [Conditions de licence](#license-requirement) ci-dessous.

| Exigence | Détails |
| --- | --- |
| **Chloros installé** | Windows : programme d'installation de bureau ; Linux : paquet `.deb` (fournit le binaire du backend et le SDK) |
| **Python** | 3.7 ou version ultérieure (développé/testé sur la version 3.10) |
| **Système d’exploitation** | Windows 10/11 64 bits, Ubuntu 22.04 LTS ou version plus récente, ou NVIDIA Jetson (JetPack 6) |
| **Licence** | Compte Chloros+ actif, n'importe quel niveau payant (Copper ou supérieur) |

## Le gain en 60 secondes

Un seul appel permet de créer un projet, d'importer un dossier, de configurer le traitement et d'exécuter le pipeline — en démarrant automatiquement le backend s'il n'est pas déjà en cours d'exécution :

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Sous Linux, utilisez les chemins Linux : `/home/user/drone_images/flight001`. Le SDK fonctionne de manière identique sur les deux plateformes.)

Vous traitez un dossier de captures LATTICE ? Utilisez le wrapper adapté à LATTICE — il applique les paramètres par défaut appropriés (pas de détection de la cible du panneau, débayérisation standard) :

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — contrôle total du pipeline

Pour toute tâche nécessitant plus qu’une simple ligne de commande, utilisez `ChlorosLocal`. Il utilise un backend déjà en cours d’exécution (celui de l’application de bureau) ou lance celui qui est installé lors de la première utilisation (`auto_start_backend=True` ; sous Windows, à partir du dossier `backend` du dossier d’installation de Chloros ; `C:\Program Files\Chloros\backend\` par défaut), crée et configure des projets, surveille leur progression et renvoie un résumé à l’issue de l’exécution.

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

{% hint style="info" %}
Conservez la valeur par défaut `http://127.0.0.1:5000` plutôt que de la remplacer par `localhost` — sous Windows, `localhost` est d’abord résolu en `::1` et coûte environ 2 secondes par requête sur le backend IPv4 uniquement.
{% endhint %}

Utilisez-le comme gestionnaire de contexte. Quitter le bloc met fin à l’utilisation du backend par cette instance, mais pas au backend lui-même : celui-ci continue de fonctionner pour le script suivant et s’arrête automatiquement après 30 minutes d’inactivité. `shutdown_backend()` arrête cette instance une fois qu’elle a été lancée.

```python
import chloros_sdk

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

`configure()` accepte les mots-clés suivants : `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` et `custom_settings`. Les valeurs principales :

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Les paramètres spécifiques à LATTICE (`input_level`, `radiometric_output`, la famille `array_alignment*`) sont documentés avec leurs tableaux de valeurs complets dans la [Référence SDK](reference/sdk-reference.md#supported-values).

### Suivi de la progression

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Lecture du résumé post-exécution — et détection des exécutions vides

Une fois l’exécution terminée, `process()` joint le résumé de traitement du backend sous la forme de `result["summary"]`. Chaque entrée de `summary["hints"]` est une phrase complète expliquant tout élément notable — par exemple, pourquoi une exécution n’a produit aucun résultat — et chaque indication est également réémise sous forme de code Python `UserWarning`, de sorte que les exécutions vides sont auto-diagnostiquées même si vous n’inspectez jamais le dictionnaire :

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` ne se déclenche pas lorsqu’une exécution ne produit aucune image.** C’est le seul point sur lequel le SDK et la CLI diffèrent délibérément : `chloros-cli process` considère que « des produits ont été demandés, mais aucun n’a été écrit » comme un échec et se termine avec un code de sortie non nul, tandis que l’SDK se termine normalement et signale cette condition via `summary` / des indications. Si votre pipeline doit s’arrêter en cas d’exécution vide, vérifiez-le vous-même — inspectez `summary` (ou comptez les fichiers dans le dossier du projet) plutôt que de vous fier à une exception.
{% endhint %}

## Smart Connect — matériel en temps réel

Trois assistants ouvrent des sessions persistantes dans le pool de matériel du backend — le même pool que celui utilisé par l’CLI, les autres scripts et les onglets « caméra » de l’interface graphique —, ce qui permet aux scripts SDK de coexister avec eux sans se disputer les ports série ou la bande passante réseau. Tous trois démarrent automatiquement un backend local si aucun n’est en cours d’exécution. Le panneau DAQ de l’interface graphique ne fait pas partie du pool : fermez un capteur de lumière dans l’interface graphique avant de l’ouvrir à partir d’un script.

### Caméra LATTICE unique — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Les chemins relatifs sont calculés par rapport à l’emplacement d’exécution de votre script (modifié dans la version 1.3.0).** Avec un backend local, un chemin relatif `output_dir` (y compris le chemin par défaut `"output"`) — et les autres chemins ouverts par le backend (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — sont résolus par rapport au répertoire de travail de votre script, de sorte que `cam.capture("output/")` écrit dans `<current folder>/output/`. Dans la version 1.2.0, un backend installé les résolvait dans son propre dossier d’installation et générait une erreur « Accès refusé ». Avec un backend distant, les chemins d’accès sont envoyés tels qu’ils ont été saisis.
{% endhint %}

### Tableau synchronisé — `connect_array`

`connect_array` est le point d’entrée recommandé pour les installations multi-caméras. Il exécute le même flux de préparation intelligente que l’interface graphique : analyse du réseau, sélection automatique du niveau de synchronisation, synchronisation temporelle PTP, sélection du format de pixel par caméra, initialisation de l’exposition automatique (AE) et armement du déclencheur GPIO. La **première caméra en série est la caméra maître** (c'est elle qui émet l'impulsion de déclenchement matériel) ; les autres sont des caméras esclaves.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Ajoutez `smart=True` à n’importe quelle capture en tableau pour attendre que l’exposition automatique se stabilise sur toutes les caméras avant le déclenchement.

Nouveauté de la version 1.3.0 : une fonction `ArraySession` vous fournit également directement des images synchronisées, sans écriture de fichiers :

* `arr.grab()` renvoie une image par caméra, toutes issues d’un seul déclenchement, sous forme de tableaux numpy (`{serial: array}`) ; `.group_id` correspond au déclenchement partagé et `.errors` désigne toute caméra non prise en compte.
* `arr.frame_stream(fps=5.0)` fournit un tel ensemble par tick, pour un traitement en temps réel dans votre propre pipeline.
* `arr.continuous("output/")` exécute la capture continue propre au backend (le mode « Continu » de l’interface graphique) et renvoie un `ContinuousHandle` avec `status()`, `wait()` et `stop()` — utilisez-le à la place d’une boucle de capture lorsque vous avez besoin que chaque déclenchement soit enregistré à un taux élevé.

Pour les modes de capture (Unique / Continu / Intervalle / Le plus rapide), les enregistreurs, la capture en rafale vers vidéo et l’alignement de tableaux, consultez la [Référence SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Capteur de lumière DAQ — `connect_daq_sensor`

Sans argument, `connect_daq_sensor()` détecte automatiquement le protocole de transport (ordre de priorité : Ethernet → BLE → USB) :

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Chaque trame contient la valeur de 135 points `spectrum` (W/m²/nm une fois calibrée), un indicateur `is_saturated` et les valeurs CIE `x`, `y`, `z`. Pour désigner un capteur ou un mode de transport spécifique, transmettez une indication explicite :

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Notez que les profils de correction de crête (`cap_id`) ne constituent **pas** un paramètre de l’SDK — sélectionnez-les plutôt via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Projets enregistrés — `open_project`

Un projet Chlorosenregistré conserve le matériel qui y est connecté (`cameras.json` + `sensors.json` ainsi que `project.json`), et `chloros_sdk.open_project(path)` peut reconnecter simultanément ses caméras, ses matrices et ses capteurs de lumière, puis déclencher des captures par nom de périphérique. Les capteurs de lumière se reconnectent via un backend Chloros en cours d’exécution, avec leurs paramètres et leur capacité enregistrés. Voir [Automatisation des projets](reference/sdk-reference.md#project-automation--chlorosproject) dans la documentation de référence.

## Ce qu’offre une installation « pip-only »

Vérifiez les indicateurs de disponibilité au niveau des modules avant d’utiliser des surfaces matérielles :

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Sur un hôte disposant **uniquement** de `pip install chloros-sdk` et ne comportant pas de package de bureau Chloros :

* `ChlorosLocal`, `process_folder` et `process_lattice_capture` ne fonctionnent **pas** — ils nécessitent le binaire backend fourni dans le programme d’installation du bureau.
* Les utilitaires Smart-Connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) sont de purs clients HTTP, ils fonctionnent donc avec un backend situé sur une autre machine — mais les backends fournis ne se connectent qu’en boucle locale ; vous devez donc rediriger le port vous-même (par exemple `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) et passer `backend_url="http://127.0.0.1:5000"` avec `auto_start_backend=False`. Voir [Mode backend distant](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Les classes LATTICE à accès direct au matériel (`LatticeCamera`, `CameraPool`, …) s’importent, mais nécessitent le runtime Arena SDK issu du package de bureau — sans celui-ci, `CAMERA_AVAILABLE` devient `False`.
* Les capteurs DAQ sont pilotés uniquement via un backend (tunnelisé) — `connect_daq_sensor()`, ou le `proj.sensors` d’un projet ; le backend ouvre le capteur sur sa propre machine.

## Assistants IA — `chloros-mcp`

Le SDK comprend `chloros-mcp`, un serveur MCP qui permet à Claude Desktop, Claude Code, Cursor et d’autres clients MCP de piloter Chloros : traiter des images et faire fonctionner les caméras LATTICE, les réseaux de capteurs et les capteurs de lumière DAQ. Ses outils couvrent toutes les fonctionnalités de ce SDK. Il s’installe avec le SDK ; pour Claude Code, la configuration se résume donc à une seule ligne :

```bash
claude mcp add chloros -- chloros-mcp
```

Autres clients : [Utilisation de Chloros avec les assistants IA](ai-assistants.md). Options et tous les outils : [Serveur MCP (assistants IA)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Conditions de licence

L’accès au SDK nécessite un compte Chloros+ actif sur n’importe quel niveau payant — **Copper ou supérieur**(Copper / Bronze / Silver / Gold) ; le niveau gratuit « Iron » ne donne pas accès au SDK ni à la CLI. La validation s'effectue**côté serveur** : chaque requête SDK doit être associée à la fois à une session active et à un forfait payant, sinon le backend renvoie les codes d'erreur `403` / `PLAN_UPGRADE_REQUIRED` (générées sous la forme `ChlorosLicenseError` par `ChlorosLocal`, et sous la forme `ChlorosConnectError` par les fonctions d’aide `connect_*`). Un appelant déconnecté obtient `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — relancer `chloros-cli login` résout le premier cas, mais pas le second.

L’utilisation hors ligne fonctionne pendant la période de grâce du forfait : le niveau d’accès est lu à partir du cache de validation du serveur (5 minutes) ou du cache de licences signées et liées à la machine (30 jours pour les forfaits mensuels ; jusqu’à l’expiration de l’abonnement pour les forfaits annuels). Une fois le délai de grâce écoulé, le forfait passe en version gratuite et l’accès au SDK est interrompu jusqu’à ce que la machine se connecte au serveur au moins une fois. `chloros-cli status` reste accessible dans le niveau gratuit, ce qui permet de toujours voir la raison du problème. Voir [Connexion à Chloros+](chloros+-login.md).

## Exceptions

Interceptez la classe de base pour gérer « tout problème survenu avec Chloros » :

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

Toutes les exceptions de pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) dérivent de `ChlorosError`. Une précision : `ChlorosConnectError` — n'est déclenché que par `connect_camera` / `connect_array` / `connect_daq_sensor` — provient de l'exception `Exception`, **et non** de `ChlorosError` ; par conséquent, `except ChlorosError` ne la détectera pas. La hiérarchie complète se trouve dans la [Référence SDK](reference/sdk-reference.md#exceptions).

## Voir aussi

* [Référence SDK](reference/sdk-reference.md) — l'ensemble complet de l'API, optimisé pour les assistants IA.
* [Référence CLI](reference/cli-reference.md) — chaque sous-commande CLI correspond à un appel SDK.
* [Utilisation d’Chloros avec les assistants IA](ai-assistants.md) — recettes de configuration et d’invite de commande `chloros-mcp`.
* [Téléchargement](download.md) — programmes d'installation pour Windows et Linux.
