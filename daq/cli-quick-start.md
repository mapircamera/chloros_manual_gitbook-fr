# Guide de démarrage rapide de l'CLI (pool-*)

Le pilote `chloros-cli` fourni gère les capteurs DAQ via la famille de commandes **`daq pool-*`** — des clients HTTP « légers » qui pilotent le capteur via le pool de capteurs persistants du backend Chloros. Le backend gère le transport ; ainsi, l’interface graphique, la CLI et les scripts SDK partagent tous un même descripteur actif au lieu de se disputer le port. Tout ce dont un client a besoin est accessible via `pool-*` : se connecter, diffuser en continu, enregistrer des fichiers `.daq` calibrés et échanger des profils de capteurs.

`pool-*` est l’interface DAQ prise en charge dans les versions publiées. `chloros-cli daq --help` répertorie les sous-commandes `pool-*`, et `daq discover` / `daq list` exécutent le même balayage que `pool-discover`. Lorsqu’une sous-commande DAQ accédant directement au matériel n’est pas disponible, le programme se termine par une erreur explicite indiquant le paquet manquant et vous renvoyant vers `pool-*` — aucune erreur ne se produit en silence. (`pip install chloros-sdk` ne fournit pas non plus de commandes matérielles directes.)

***

## Prérequis

* **Le backend Chloros doit être en cours d’exécution** — les commandes `pool-*` sont des clients HTTP, et non des pilotes matériels. Sous Windows, lancez l’application de bureau Chloros (elle démarre le backend). Sur un système Linux/Jetson sans interface graphique, activez le service en tant qu’utilisateur : `sudo systemctl enable --now chloros-backend@$USER.service` (voir [Installation sous Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Connexion à Chloros+ (formule payante)** : exécutez d’abord la commande `chloros-cli login`. La validation s'effectue côté serveur : sans connexion, les commandes échouent avec `401 AUTH_REQUIRED` ; sur le forfait gratuit (Iron), elles échouent avec `403 PLAN_UPGRADE_REQUIRED`.
* Les commandes ciblent `http://127.0.0.1:5000` par défaut ; la famille `daq pool-*` prend en compte la variable d’environnement `CHLOROS_BACKEND_URL` si votre backend s’exécute ailleurs.

***

## Une session de cinq minutes

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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
| `daq pool-connect` | Détection intelligente : rechercher n'importe quel DAQ sur cette machine. |
| `daq pool-connect --port PORT` | DAQ-U sur un port série spécifique (par ex. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, adresse MAC détectée automatiquement. |
| `daq pool-connect --mac MAC` | DAQ-M sur une adresse MAC BLE connue (implique `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E avec un nom d’hôte ou une adresse IP connue — **la méthode fiable**. |
| `daq pool-connect --eth` | DAQ-E avec découverte automatique (mDNS, avec repli sur ARP). Voir la mise en garde ci-dessous. |

Indicateurs de réglage, tous facultatifs :

| Indicateur | Signification |
| --- | --- |
| `--integration-time MS` / `-t MS` | Temps d'intégration manuel en millisecondes. |
| `--frame-avg N` / `-f N` | Nombre moyen de trames par spectre rapporté. |
| `--no-ae` | Désactiver l'exposition automatique (AE est activée par défaut). |
| `--no-stream` | Se connecter sans démarrer le flux (reprendre plus tard avec `pool-stream --start`). |
| `--cap-id CAP` | Profil de correction Cap ; la valeur par défaut du backend est `sunshine_cosine`. Voir [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

La connexion d’un DAQ-E déjà présent dans le pool sous une autre de ses adresses — son nom d’hôte `daq-e-<id>.local` ou son adresse IP — renvoie l’entrée existante du pool au lieu d’ouvrir l’unité une deuxième fois.

{% hint style="warning" %}
**Remarque concernant la détection automatique `--eth`.** Sur un hôte multi-homed (disposant de plusieurs interfaces réseau actives), le *premier* `pool-connect --eth` après le démarrage peut ne rien renvoyer même si le capteur est en bon état — la recherche de découverte peut ne pas détecter l’interface du capteur tant que le cache ARP est vide. Si `--eth` ne trouve rien, réessayez, ou ignorez complètement la détection en utilisant `--eth-host <ip-or-hostname>`, qui constitue la méthode fiable sur les machines multi-homed. Le nom d’hôte du DAQ-E est `daq-e-<id>.local` (par exemple `daq-e-def330.local`) ; son adresse IP brute fonctionne également.
{% endhint %}

## `pool-discover` — analyse sans connexion

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Recherche les capteurs auxquels vous pourriez vous connecter et affiche l’adresse à utiliser pour la connexion, sans établir de connexion. `--only` prend un sous-ensemble séparé par des virgules de `usb,ble,eth` ; `--timeout SEC` définit la durée de balayage par transport (par défaut 5 s, comprise entre 1 et 20 s). Les capteurs déjà ouverts dans le pool ne sont pas répertoriés (un DAQ-M connecté cesse de s’annoncer) — utilisez `pool-list` pour ceux-ci. Dans une version livrée, `chloros-cli daq discover` et `daq list` exécutent ce même balayage.

## `pool-list` — afficher les appareils connectés

Affiche tous les capteurs du pool backend, y compris le `sensor_id` requis par toutes les autres commandes :

| Modèle | Forme `sensor_id` | Exemple |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octets séparés par des tirets | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Un DAQ-E dont le PTP a été désactivé (le réglage est enregistré sur l'appareil) est signalé dans `pool-list` avec `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` et la commande qui le réactive — voir [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — lecture des trames de spectre

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Renvoie la trame la plus récente, ou les trames les plus récentes `--recent N` ; `--json` génère une sortie lisible par machine pour la création de scripts. Les trames correspondent à l’irradiance spectrale (W/m²/nm) calibrée radiométriquement sur une grille de 135 points, comprise entre 340 et 1 010 nm, avec le profil de capuchon du capteur déjà appliqué. Pour obtenir des valeurs d’irradiance quantitatives, effectuez une moyenne sur au moins 15 secondes d’images — il s’agit d’une caractéristique de l’instrument, et non d’un défaut.

## `pool-stream` — mettre en pause ou reprendre la transmission en continu

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
| `--duration SEC` / `-d SEC` | `0` | Durée d’enregistrement en secondes ; `0` signifie « s'exécuter jusqu'à ce que vous lanciez `--stop` ». |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Répertoire de sortie. Avec le backend sur cet ordinateur, un chemin relatif est résolu par rapport au dossier dans lequel vous avez exécuté la commande ; avec un backend distant, le chemin est résolu **sur la machine exécutant le backend**. |
| `--device-name NAME` | — | Étiquette enregistrée avec l'enregistrement. |
| `--stop` | — | Arrêter un enregistrement en cours. |
| `--confirm-after SEC` | `20` | Après le démarrage, attendre jusqu’à ce délai pour que la première ligne soit écrite, et émettre un avertissement sonore (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) si aucune n’arrive. `0` désactive cette vérification. |
| `--heartbeat SEC` | `30` | Avec `--duration`, affiche le nombre de lignes toutes les SEC secondes et signale un avertissement s’il cesse d’augmenter. `0` la désactive. |

{% hint style="info" %}
L'enregistrement s'effectue en arrière-plan, le fichier `.daq` se retrouve donc sur le système de fichiers de la **machine backend** — par défaut dans `~/Documents/DAQ Live View/` à cet emplacement, et pas nécessairement là où vous avez exécuté la CLI. Les noms de fichiers comprennent l’identifiant du capteur et un horodatage.
{% endhint %}

## `pool-set-cap` — Déclarer le capuchon installé

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

L’identifiant du capuchon sélectionne le profil de correction mesuré en usine appliqué à chaque spectre, et il **doit correspondre au capuchon physiquement monté sur le capteur** — ni le capteur ni le logiciel ne peuvent détecter le capuchon par eux-mêmes, et cette sélection est enregistrée dans chaque fichier `.daq`. La valeur par défaut partout est `sunshine_cosine` (chaque DAQ est livré avec le capuchon de correction cosinus Sunshine installé, offrant une atténuation d’environ 12× par conception — un changement de capuchon non déclaré entraîne une correction erronée des spectres d’un facteur proche de celui-ci).

| `--cap-id` | Disponible sur |
| --- | --- |
| `sunshine_cosine` (par défaut) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | DAQ-U uniquement |
| `none` | DAQ-E uniquement — voir la remarque |

Un identifiant de capuchon ne faisant pas partie de l'ensemble prévu pour le capteur est refusé lors de la connexion et génère une erreur claire. `none` (DAQ-E) signifie que le capuchon est physiquement retiré — il applique toujours un profil géométrique d'usine pour le diffuseur en verre encastré du DAQ-E ; il ne s'agit donc pas d'une opération sans effet, et un DAQ-E « nu » correspond à une configuration de laboratoire, et non à une configuration de terrain prise en charge. (Un DAQ-U nu est véritablement nu et ne nécessite aucun profil de correction ; le DAQ-M s’utilise avec son capuchon Sunshine.)

## `pool-disconnect` — libération des capteurs

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` libère un capteur quelle que soit la personne qui l’a ouvert ; il libère donc également un capteur laissé dans le pool par un script qui a planté (à condition qu’un enregistrement en cours sur ce capteur soit d’abord terminé). Un capteur qui ne se trouve pas dans le pool n’a rien à libérer : la commande affiche `<sensor-id>: already released (not in the pool)` et se termine par `0`. Une libération refusée ou non confirmée par le backend se termine par `1`.

## `pool-enable-ptp` — réactiver le PTP d’un DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Un DAQ-E conserve son paramètre d’activation/désactivation du PTP dans sa propre mémoire ; ainsi, un appareil dont le PTP a été désactivé reste désactivé — et ses horodatages ne sont pas synchronisés via PTP — jusqu’à ce qu’il soit réactivé. `pool-list` signale un tel appareil et affiche cette commande pour celui-ci. Chloros ne réactive jamais un appareil de lui-même, car la désactivation peut être intentionnelle. Nouveauté de la version 1.3.0.

## Paramètres des appareils DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` affiche le micrologiciel du capteur (et indique si cette version de Chloros en propose un plus récent), son adresse et la manière dont il l’a obtenue, les paramètres réseau enregistrés lorsqu’ils diffèrent de ceux actuellement utilisés, la diffusion en continu à la mise sous tension et le PTP. `pool-set-network` redémarre le capteur pour appliquer la modification et le reconnecte avec les paramètres qu’il avait auparavant ; `--no-reboot` se contente de les enregistrer. Une adresse statique doit exister sur le réseau de cet ordinateur, sinon Chloros la refuse ; `--force` passe outre cette restriction, uniquement pour un ordinateur qui sera réadressé ou acheminé vers cette adresse. Les redémarrages sont refusés pendant que le capteur enregistre. `pool-update-firmware` correspond à la commande **Update to \<version\>**</version\> de l’interface graphique <version\>pour la ligne de commande, et permet de mettre à jour un DAQ-E sous Linux qui ne dispose pas d’interface graphique. Nouveauté de la version 1.3.0.***

## Résumé des commandes

| Commande | Objectif |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Ouvrir un capteur dans le pool du backend. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Rechercher des capteurs sans se connecter (retrouve l'adresse MAC BLE d'un DAQ-M). |
| `daq pool-list` | Affiche tous les capteurs du pool avec leur `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Les N dernières trames de spectre calibrées. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reprendre / mettre en pause la diffusion en continu. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Démarrer / arrêter un enregistrement `.daq` (côté backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Changer le profil de correction de crête en cours d’exécution. |
| `daq pool-disconnect --sensor-id ID [--all]` | Libérer un capteur, ou tous les capteurs. |
| `daq pool-enable-ptp --sensor-id ID` | Réactiver le PTP pour un DAQ-E dont le PTP est désactivé. |
| `daq pool-info --sensor-id ID [--json]` | Le micrologiciel, l'adresse, les paramètres réseau enregistrés, la diffusion à la mise sous tension et le PTP d'un DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP ou adresse statique ; redémarre et reconnecte le capteur. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Diffusion à la mise sous tension. |
| `daq pool-reboot --sensor-id ID` | Redémarre un DAQ-E et le reconnecte. |
| `daq pool-update-firmware --sensor-id ID` | Installe le micrologiciel DAQ-E de cette version sur un capteur plus ancien. |

***

## Dépannage d’une première connexion DAQ-E

1. Le DAQ-E ne dispose pas de LED d’état — vérifiez qu’il est sous tension via le voyant PoE/liaison sur le commutateur ou le port de l’injecteur, et patientez quelques secondes après la mise sous tension pour qu’il démarre et se connecte au réseau.
2. La machine backend doit se trouver sur le **même domaine de diffusion** que le capteur — le protocole mDNS ne traverse pas les routeurs.
3. Un pare-feu doit autoriser le passage du protocole mDNS (UDP 5353), des données DAQ-E (UDP 5002) et du protocole PTP (UDP 319/320). Le programme d’installation Windows ajoute une règle à cet effet, et le programme d’installation Linux `.deb` les ouvre dans un `ufw` / `firewalld` qui était actif au moment de l’installation ; un pare-feu tiers, ou un pare-feu activé ultérieurement, doit les autoriser.
4. Toujours rien de la part de `--eth` ? Utilisez `--eth-host` avec le nom d'hôte (`daq-e-<id>.local`) ou l'adresse IP de l'appareil, ce qui permet d'éviter la phase de découverte.

***
{% hint style="info" %}
**Astuce pour les assistants IA.** Chaque page de ce manuel est fournie au format Markdown brut — ajoutez `.md` à la partie en minuscules de l'URL de la page (cette page : `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`) ; l’index lisible par machine est `https://mapir.gitbook.io/chloros/llms.txt`. Pour consulter la documentation complète au niveau des options de `chloros-cli daq` et de toutes les autres familles de commandes, consultez la [Référence CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`) ; le chemin d'accès Python est `chloros_sdk.connect_daq_sensor()` dans la [Référence SDK](../reference/sdk-reference.md).
{% endhint %}
</version\>
