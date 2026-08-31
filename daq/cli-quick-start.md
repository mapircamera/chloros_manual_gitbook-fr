# CLI Guide de démarrage rapide (pool-*)

Le `chloros-cli` fourni pilote les capteurs DAQ via la famille de commandes **`daq pool-*`** — des clients légers HTTP qui pilotent le capteur via le pool de capteurs persistants du backend Chloros. Le backend gère le transport ; ainsi, l’interface graphique, le script CLI et le script SDK partagent tous un même descripteur actif au lieu de se disputer le port. Tout ce dont un client a besoin est accessible via `pool-*` : se connecter, diffuser en continu, enregistrer des fichiers `.daq` calibrés et changer de profil de condensateur.

`pool-*` est également la **seule** interface DAQ dans les versions publiées. `chloros-cli daq --help` répertorie les sous-commandes `pool-*`, et l’appel d’une sous-commande DAQ à accès direct au matériel sur une version distribuée se termine par une erreur explicite indiquant le paquet manquant et vous renvoyant vers `pool-*` — rien ne se passe en silence. (Les commandes d&#x27;acquisition de données directement sur le matériel ne s&#x27;exécutent qu&#x27;à partir d&#x27;un checkout du code source MAPIR ; `pip install chloros-sdk` ne les fournit pas non plus.)

***

## Prérequis

* **Le backend Chloros doit être en cours d’exécution** — les commandes `pool-*` sont des clients HTTP, et non des pilotes matériels. Sur Windows, lancez l’application de bureau Chloros (elle démarre le backend). Sur un Linux/Jetson sans interface graphique, activez le service : `sudo systemctl enable --now chloros-backend.service`.
* **Connexion à Chloros+ (niveau payant)** : exécutez d’abord `chloros-cli login`. La validation s’effectue côté serveur : sans connexion, les commandes échouent avec le code d’erreur `401 AUTH_REQUIRED` ; sur la formule gratuite (Iron), elles échouent avec le code d’erreur `403 PLAN_UPGRADE_REQUIRED`.
* Les commandes ciblent `http://127.0.0.1:5000` par défaut ; la famille `daq pool-*` prend en compte la variable d’environnement `CHLOROS_BACKEND_URL` si votre backend s’exécute ailleurs.

***

## Une session de cinq minutes

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — ouvrir un capteur dans le pool

| Variante | Signification |
| --- | --- |
| `daq pool-connect` | Détection intelligente : rechercher n&#x27;importe quel DAQ sur cette machine. |
| `daq pool-connect --port PORT` | DAQ-U sur un port série spécifique (par ex. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, adresse MAC détectée automatiquement. |
| `daq pool-connect --mac MAC` | DAQ-M sur une adresse MAC BLE connue (implique `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E avec un nom d’hôte ou une adresse IP connue — **la méthode fiable**. |
| `daq pool-connect --eth` | DAQ-E avec découverte automatique (mDNS, avec repli sur ARP). Voir la mise en garde ci-dessous. |

Indicateurs de réglage, tous facultatifs :

| Indicateur | Signification |
| --- | --- |
| `--integration-time MS` / `-t MS` | Temps d&#x27;intégration manuel en millisecondes. |
| `--frame-avg N` / `-f N` | Nombre moyen de trames par spectre rapporté. |
| `--no-ae` | Désactiver l&#x27;exposition automatique (l&#x27;AE est activée par défaut). |
| `--no-stream` | Se connecter sans démarrer le flux (reprendre plus tard avec `pool-stream --start`). |
| `--cap-id CAP` | Profil de correction Cap ; la valeur par défaut du backend est `sunshine_cosine`. Voir [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Remarque concernant la détection automatique de `--eth`.** Sur un hôte multi-homed (disposant de plusieurs interfaces réseau actives), le *premier* `pool-connect --eth` après le démarrage peut s&#x27;avérer vide même si le capteur est en bon état — la recherche de découverte peut ne pas détecter l&#x27;interface du capteur tant que le cache ARP n&#x27;est pas encore alimenté. Si `--eth` ne trouve rien, réessayez, ou ignorez complètement la détection en utilisant `--eth-host <ip-or-hostname>`, qui constitue la méthode fiable sur les machines multi-homed. Le nom d’hôte du DAQ-E est `daq-e-<id>.local` (par exemple `daq-e-def330.local`) ; son adresse IP simple fonctionne également.
{% endhint %}

## `pool-list` — afficher les périphériques connectés

Affiche tous les capteurs du pool de backend, y compris le `sensor_id` requis par toutes les autres commandes :

| Modèle | Format `sensor_id` | Exemple |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octets séparés par des tirets | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — lecture des trames de spectre

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Renvoie la trame la plus récente, ou les trames `--recent N` les plus récentes ; `--json` génère une sortie lisible par machine pour les scripts. Les trames correspondent à l’irradiance spectrale (W/m²/nm) calibrée radiométriquement sur une grille de 135 points, comprise entre 340 et 1 010 nm, avec le profil de capuchon du capteur déjà appliqué. Pour obtenir des valeurs d’irradiance quantitatives, faites la moyenne d’au moins 15 secondes d’images — il s’agit d’une caractéristique de l’instrument, et non d’un défaut.

## `pool-stream` — mettre en pause ou reprendre la diffusion en continu

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — enregistrer un fichier `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Indicateur | Valeur par défaut | Signification |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Durée d’enregistrement en secondes ; `0` signifie « s&#x27;exécuter jusqu&#x27;à ce que vous lanciez `--stop` ». |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Répertoire de sortie, déterminé **sur la machine exécutant le backend**. |
| `--device-name NAME` | — | Étiquette stockée avec l&#x27;enregistrement. |
| `--stop` | — | Arrête un enregistrement en cours. |

{% hint style="info" %}
L&#x27;enregistrement s&#x27;effectue dans le backend, le fichier `.daq` se retrouve donc sur le système de fichiers de la **machine du backend** — par défaut dans `~/Documents/DAQ Live View/` à cet emplacement, et pas nécessairement là où vous avez exécuté CLI. Les noms de fichiers comprennent l’identifiant du capteur et un horodatage.
{% endhint %}

## `pool-set-cap` — Déclarer le capuchon installé

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

L’identifiant du capuchon sélectionne le profil de correction mesuré en usine appliqué à chaque spectre, et il **doit correspondre au capuchon physiquement monté sur le capteur** — ni le capteur ni le logiciel ne peuvent détecter le capuchon par eux-mêmes, et cette sélection est enregistrée dans chaque fichier `.daq`. La valeur par défaut partout est `sunshine_cosine` (chaque DAQ est livré avec le capuchon de correction cosinus « Sunshine » installé, offrant une atténuation d’environ 12× par conception — un changement de capuchon non déclaré entraîne une correction erronée des spectres d’un facteur proche de celui-ci).

| `--cap-id` | Disponible sur |
| --- | --- |
| `sunshine_cosine` (par défaut) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | DAQ-U uniquement |
| `none` | DAQ-E uniquement — voir remarque |

Un identifiant de capuchon ne figurant pas dans le jeu défini pour le capteur est refusé lors de la connexion et génère une erreur explicite. `none` (DAQ-E) signifie que le capuchon a été physiquement retiré — il applique toujours un profil géométrique d&#x27;usine pour le diffuseur en verre encastré du DAQ-E ; il ne s&#x27;agit donc pas d&#x27;une opération sans effet, et un DAQ-E « nu » correspond à une configuration de laboratoire, et non à une configuration de terrain prise en charge. (Un DAQ-U nu est véritablement « nu » et ne nécessite aucun profil de correction ; le DAQ-M s’utilise avec son capuchon Sunshine.)

## `pool-disconnect` — libération des capteurs

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Résumé des commandes

| Commande | Objectif |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Ouvrir un capteur dans le pool backend. |
| `daq pool-list` | Afficher tous les capteurs du pool avec leur `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Les N dernières trames de spectre calibrées. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reprendre / mettre en pause le streaming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Démarrer / arrêter un enregistrement `.daq` (côté backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Changer le profil de correction de crête pendant l&#x27;exécution. |
| `daq pool-disconnect --sensor-id ID [--all]` | Libérer un capteur ou tous les capteurs. |

***

## Dépannage d&#x27;une première connexion DAQ-E

1. Le DAQ-E ne dispose pas de LED d&#x27;état — vérifiez qu&#x27;il est alimenté via le voyant PoE/liaison sur le commutateur ou le port de l&#x27;injecteur, et attendez quelques secondes après la mise sous tension pour qu&#x27;il démarre et se connecte au réseau.
2. La machine backend doit se trouver sur le **même domaine de diffusion** que le capteur — le protocole mDNS ne traverse pas les routeurs.
3. Sur Windows, acceptez l&#x27;invite du pare-feu Defender lors du premier démarrage (mDNS UDP 5353, données DAQ-E UDP 5002, PTP UDP 319/320).
4. Toujours rien de la part de `--eth` ? Utilisez `--eth-host` avec le nom d’hôte de l’appareil (`daq-e-<id>.local`) ou son adresse IP — c’est la méthode la plus fiable, en particulier sur les hôtes multi-homés.

***{% hint style="info" %}**Astuce pour les assistants IA.** Chaque page de ce manuel est fournie au format Markdown brut — ajoutez `.md` à la balise (slug) en minuscules d&#x27;une page, URL (cette page : `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`) ; l’index lisible par machine est alors `https://mapir.gitbook.io/chloros/llms.txt`. Pour consulter la documentation complète sur les options de `chloros-cli daq` et de toutes les autres familles de commandes, consultez la [Référence CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`) ; le chemin d’accès de Python est `chloros_sdk.connect_daq_sensor()` dans la [Référence SDK](../reference/sdk-reference.md).
{% endhint %}
