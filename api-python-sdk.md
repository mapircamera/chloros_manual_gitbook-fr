# API : Python SDK

{% hint style="info" %}
**Vous recherchez la documentation complète sur API ?** Cette page est un tutoriel pratique. Toutes les classes publiques, méthodes, signatures exactes et exemples copiables-collables se trouvent dans la [Référence SDK](reference/sdk-reference.md), optimisée pour les assistants IA.**Vous travaillez avec un assistant IA ?** Collez ce code URL dans le chat pour qu’il dispose de la version complète et à jour de Chloros 1.2.0 API :

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Chaque page de ce manuel est disponible au format Markdown brut sous la forme de son slug en minuscules + `.md`, et l&#x27;intégralité du manuel est indexée à l&#x27;adresse `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

Le **Chloros Python SDK** (`chloros-sdk` sur PyPI) gère toutes les fonctionnalités de l’application de bureau à partir de Python : traitement d’images par lots, contrôle en direct de la caméra LATTICE et du réseau de capteurs, sessions DAQ avec capteurs de lumière et automatisation des projets enregistrés. Il s&#x27;agit d&#x27;une couche légère superposée au même backend local que celui utilisé par l&#x27;interface graphique et CLI (HTTP sur `127.0.0.1:5000`) ; le comportement est donc identique sur les trois interfaces.

## Installation

L&#x27;installation se déroule en deux étapes : d&#x27;abord le paquet de bureau Chloros (qui fournit le backend de traitement et les moteurs d&#x27;exécution matériels), puis le paquet Python.

**Étape 1 — Installez Chloros.** Windows : exécutez le programme d’installation pour ordinateur de bureau (chemin par défaut `C:\Program Files\MAPIR\Chloros\`) depuis la page [Téléchargement](download.md). Linux : installez le paquet `.deb` ([Installation de Linux](linux/linux-installation.md)).**Étape 2 — Installez SDK** (Python 3.7+) :

```bash
pip install chloros-sdk
```

Vous n’aurez peut-être même pas besoin de pip : chaque programme d’installation fournit un fichier wheel SDK correspondant. Le programme d’installation Windows l’installe automatiquement dans votre système Python ; le programme d’installation Linux `.deb` le place dans `/usr/lib/chloros/sdk/` et affiche la commande exacte `pip install --user`. PyPI est mis à jour lors des versions de publication, de sorte que `pip install chloros-sdk` correspond à la dernière version stable.

**Étape 3 — Se connecter une fois par machine :**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Les identifiants sont mis en cache dans `~/.chloros/` (sur les deux plateformes). Sur Windows, vous pouvez également vous connecter via l’onglet « Utilisateur » <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> de l’application de bureau. SDK nécessite un abonnement payant Chloros+ — voir [Conditions de licence](#license-requirement) ci-dessous.

| Condition | Détails |
| --- | --- |
| **Chloros installé** | Windows : programme d&#x27;installation pour ordinateur de bureau ; Linux : paquet `.deb` (fournit le binaire du backend) |
| **Python** | 3.7 ou version ultérieure (développé/testé sur la version 3.10) |
| **Système d&#x27;exploitation** | Windows 10/11 64 bits, Ubuntu 22.04 LTS ou plus récent, ou NVIDIA Jetson (JetPack 6) |
| **Licence** | Compte Chloros+ actif, n&#x27;importe quel niveau payant (Copper ou supérieur) |

## Le gain de 60 secondes

Un seul appel permet de créer un projet, d’importer un dossier, de configurer le traitement et d’exécuter le pipeline — en démarrant automatiquement le backend s’il n’est pas déjà en cours d’exécution :

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Sur Linux, utilisez les chemins d’accès Linux : `/home/user/drone_images/flight001`. La commande SDK fonctionne de manière identique sur les deux plateformes.)

Vous traitez un dossier de capture LATTICE ? Utilisez le wrapper compatible avec LATTICE — il applique les paramètres par défaut appropriés (pas de détection de cible de panneau, débayérisation standard) :

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — contrôle complet du pipeline

Pour toute tâche dépassant une simple ligne de commande, utilisez `ChlorosLocal`. Il lance le backend lors de la première utilisation (`auto_start_backend=True`), crée et configure les projets, surveille la progression et renvoie un résumé à la fin de l’exécution.

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
Conservez la valeur par défaut `http://127.0.0.1:5000` plutôt que de la remplacer par `localhost` — sur Windows, `localhost` est d’abord résolu en `::1` et prend environ 2 secondes par requête sur le backend IPv4 uniquement.
{% endhint %}

Utilisez-le comme gestionnaire de contexte pour garantir le nettoyage :

```python
import chloros_sdk

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

`configure()` accepte les mots-clés suivants : `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` et `custom_settings`. Les valeurs principales :

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Les potentiomètres spécifiques à LATTICE (`input_level`, `radiometric_output`, la famille `array_alignment*`) sont documentés avec leurs tableaux de valeurs complets dans la [Référence SDK](reference/sdk-reference.md#supported-values).

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

Une fois l&#x27;exécution terminée, `process()` joint le résumé de traitement du backend sous le nom `result["summary"]`. Chaque entrée de `summary["hints"]` est une phrase complète expliquant tout élément notable — par exemple, pourquoi une exécution n’a produit aucun résultat — et chaque indication est également réémise sous la forme d’un Python `UserWarning`, de sorte que les exécutions vides s’autodiagnostiquent même si vous n’inspectez jamais le dictionnaire :

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` ne se déclenche pas lorsqu’une exécution ne produit aucune image.** C’est le seul point sur lequel SDK et CLI diffèrent délibérément : `chloros-cli process` considère que « des produits ont été demandés, mais aucun n’a été écrit » comme un échec et se termine avec un code de sortie non nul, tandis que SDK se termine normalement et signale cette situation via `summary` / des messages d’aide. Si votre pipeline doit s’arrêter en cas d’exécution vide, vérifiez-le vous-même — inspectez `summary` (ou comptez les fichiers dans le dossier du projet) plutôt que de vous fier à une exception.
{% endhint %}

## Smart Connect — matériel en direct

Trois assistants ouvrent des sessions persistantes dans le pool de matériel du backend — le même pool que celui utilisé par l&#x27;interface graphique —, ce qui permet aux scripts SDK de coexister avec l&#x27;application de bureau sans se disputer les ports série ni la bande passante réseau. Tous trois lancent automatiquement un backend local si aucun n&#x27;est en cours d&#x27;exécution.

### Caméra LATTICE unique — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Matrice synchronisée — `connect_array`

`connect_array` est le point d’entrée recommandé pour les configurations à plusieurs caméras. Il exécute le même flux de préparation intelligente que l’interface graphique : analyse du réseau, sélection automatique du niveau de synchronisation, synchronisation temporelle PTP, sélection du format de pixel par caméra, initialisation de l’exposition automatique (AE) et armement du déclencheur GPIO. La **première caméra en série est la caméra maître** (c&#x27;est elle qui émet l&#x27;impulsion de déclenchement matériel) ; les autres sont des esclaves.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Ajoutez `smart=True` à n&#x27;importe quelle capture en matrice pour attendre que l&#x27;exposition automatique se stabilise sur toutes les caméras avant le déclenchement. Pour les modes de capture (Unique / Continu / Intervalle / Le plus rapide), les enregistreurs, la capture en rafale vers vidéo et l’alignement du réseau, consultez la [Référence SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Capteur de lumière DAQ — `connect_daq_sensor`

Sans argument, `connect_daq_sensor()` détecte automatiquement le protocole de transport (par ordre de priorité : Ethernet → BLE → USB) :

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Chaque trame contient la valeur de 135 points `spectrum` (W/m²/nm une fois étalonné), un indicateur `is_saturated` et les valeurs CIE `x`, `y`, `z`. Pour désigner un capteur ou un mode de transport spécifique — ce qui constitue le choix le plus fiable sur les hôtes dotés de plusieurs interfaces réseau, où la détection automatique via Ethernet peut ne pas repérer un DAQ-E en bon état de fonctionnement dès la première tentative —, il suffit de passer un indicateur explicite :

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Notez que les profils de correction de crête (`cap_id`) ne sont **pas** un paramètre SDK — sélectionnez-les plutôt via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Projets enregistrés — `open_project`

Un projet Chloros enregistré conserve le matériel qui y est connecté (`cameras.json` + `sensors.json` ainsi que `project.json`), et `chloros_sdk.open_project(path)` peut tout reconnecter en une seule fois et effectuer des captures de pilotes par nom de périphérique. Voir [Automatisation des projets](reference/sdk-reference.md#project-automation--chlorosproject) dans la documentation de référence.

## Ce que procure une installation « pip-only »

Vérifiez les indicateurs de disponibilité au niveau du module avant d’utiliser les surfaces matérielles :

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Sur un hôte disposant **uniquement** de `pip install chloros-sdk` et ne disposant pas du package de bureau Chloros :

* `ChlorosLocal`, `process_folder` et `process_lattice_capture` **ne**fonctionnent**pas** — ils nécessitent le binaire backend fourni dans le programme d’installation du bureau.
* Les assistants Smart-Connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) sont de purs clients HTTP ; ils fonctionnent donc avec un backend situé sur une autre machine — mais les backends fournis ne se connectent qu’en boucle locale ; vous devez donc rediriger le port vous-même (par exemple `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) et associer `backend_url="http://127.0.0.1:5000"` à `auto_start_backend=False`. Voir [Mode backend distant](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Les classes LATTICE en accès direct au matériel (`LatticeCamera`, `CameraPool`, …) s’importent, mais nécessitent le runtime Arena SDK issu du paquet de bureau — sans celui-ci, `CAMERA_AVAILABLE` correspond à `False`.
* `daq_sdk` (les classes DAQ directes) est fourni avec l’installation « desktop », et non avec le paquet PyPI, donc `DAQ_AVAILABLE` correspond à `False` sur un hôte utilisant uniquement pip — pilotez plutôt les capteurs DAQ via `connect_daq_sensor()` vers un backend (en tunnel).

## Conditions de licence

L&#x27;accès à SDK nécessite un compte Chloros+ actif sur n&#x27;importe quel niveau payant — **Copper ou supérieur**(Copper / Bronze / Silver / Gold) ; le niveau gratuit Iron ne donne pas accès à SDK/CLI. La validation s’effectue**côté serveur** : chaque requête SDK doit comporter à la fois une session active et un forfait payant, sinon le backend renvoie `403` / `PLAN_UPGRADE_REQUIRED` (généré sous la forme `ChlorosLicenseError` par `ChlorosLocal`, et sous la forme `ChlorosConnectError` par les aides `connect_*`). Un appelant déconnecté obtient `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — relancer `chloros-cli login` résout le premier cas, mais pas le second.

L&#x27;utilisation hors ligne fonctionne pendant la période de grâce du forfait : le niveau d’accès est lu à partir du cache de validation du serveur (5 minutes) ou du cache de licence signée et liée à la machine (30 jours pour les forfaits mensuels ; jusqu’à l’expiration de l’abonnement pour les forfaits annuels). Une fois la période de grâce écoulée, le forfait passe en mode gratuit et l’accès via SDK est interrompu jusqu’à ce que l’ordinateur se connecte au serveur au moins une fois. La commande `chloros-cli status` reste accessible dans le niveau gratuit, ce qui permet de toujours voir la raison de ce problème. Voir [Chloros+ Connexion](chloros+-login.md).

## Exceptions

Interceptez la classe de base pour gérer « tout ce qui a mal tourné » :

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

Toutes les exceptions de pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) découlent de `ChlorosError`. Une précision : `ChlorosConnectError` — n&#x27;est déclenché que par `connect_camera` / `connect_array` / `connect_daq_sensor` — provient du simple `Exception`, **et non** de `ChlorosError` ; par conséquent, `except ChlorosError` ne le détectera pas. La hiérarchie complète se trouve dans la [Référence SDK](reference/sdk-reference.md#exceptions).

## Voir aussi

* [Référence SDK](reference/sdk-reference.md) — la surface complète API, optimisée pour les assistants IA.
* [Référence CLI](reference/cli-reference.md) — chaque sous-commande CLI correspond à un appel SDK.
* [Téléchargement](download.md) — programmes d&#x27;installation pour Windows et Linux.
