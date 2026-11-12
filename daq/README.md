# Capteurs de lumière DAQ

> **Vous cherchez des informations sur le matériel ?**Les capteurs eux-mêmes — modèles, montage, capuchons, ports, alimentation et l'application SCANNER — sont décrits dans le**[manuel d'utilisation DAQ](https://mapir.gitbook.io/daq)**. Ce chapitre traite de leur utilisation depuis Chloros. Les capteurs de lumière**DAQ** d’

MAPIR mesurent la lumière ambiante sous forme de spectres étalonnés radiométriquement. Dans Chloros, ils remplissent deux rôles :

* **Un instrument spectral autonome** — graphiques spectraux en temps réel, données colorimétriques et enregistrements `.daq`, le tout accessible depuis l’ [onglet « Capteurs de lumière »](gui.md), de la [CLI](cli-quick-start.md) ou du SDK Python Python.
* **Une source d’irradiance descendante pour la réflectance** — lors du traitement, Chloros interpole vos mesures `.daq` en fonction de l’horodatage d’exposition de chaque prise de vue et utilise la lumière descendante mesurée pour convertir la radiance de la caméra en réflectance (`--reflectance-source daq`) ; aucun panneau de calibrage sur le terrain n’est nécessaire pour les bandes calibrées.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Trois modèles, un seul format de données

| Modèle | Transport | Découverte |
| --- | --- | --- |
| **DAQ-U** | USB (série) | balayage du port série |
| **DAQ-M** | Bluetooth Low Energy | balayage BLE par nom |
| **DAQ-E** | Ethernet (IPv4, alimentation PoE) | mDNS `_daq-e._tcp` (nom d’hôte `daq-e-<id>.local`) |

**DAQ-E-S (nouveau dans la version 1.3.0).** Un DAQ-E équipé d’un diffuseur fixe et d’un capteur d’inclinaison. Chloros le répertorie sous le nom `DAQ-E-S` et le fait fonctionner exactement comme un DAQ-E, à l’exception qu’il n’effectue aucune correction de cap et que chaque mesure inclut l’inclinaison, le roulis et le tangage de l’appareil. Voir [DAQ-E-S : diffuseur fixe et inclinaison](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Ces trois modèles utilisent le même protocole de communication et fournissent des données identiques :

* Un **spectre de 135 points compris entre 340 et 1 010 nm par pas de 5 nm**, ainsi que les valeurs tristimulus CIE XYZ, dans chaque trame.
* **Irradiance spectrale calibrée radiométriquement en W/m²/nm** — le jeu de calibrage d’usine de chaque appareil (ainsi que son profil de correction de capuchon actif) est appliqué avant que les données ne vous parviennent.
* Le même **format d’enregistrement `.daq`** (un fichier SQLite). Le traitement en aval est identique, quel que soit le mode de transport ayant généré le fichier.

Les piles de transport (série USB, BLE, mDNS/zeroconf) sont intégrées au backend d’Chloros — aucune installation n’est nécessaire pour communiquer avec l’un des trois modèles depuis l’interface graphique ou via les commandes `pool-*` de l’CLI.

***

## Plage étalonnée : 340–1010 nm déclarée, ~374–974 nm étalonnée

Le capteur déclare la grille complète de 340 à 1010 nm, mais le gain radiométrique traçable selon le NIST s’étend approximativement de **374 à 974 nm**. Chloros refuse la division par réflectance absolue pour toute bande de caméra dont moins de la moitié du poids spectral se situe à l’intérieur de cette plage étalonnée ; la bande ignorée est signalée avec le motif d’ignorance `dls-uncalibrated-band-<nm>`.

Parmi les références de filtres LATTICE disponibles à la vente, seule la **F988** est concernée :

La réflectance de la F988 est étalonnée à l’aide d’un panneau de réflectance intégré à la scène : la bande se situant au-delà de la plage étalonnée du capteur de lumière DAQ, Chloros utilise votre dernière capture du panneau et la conserve entre deux relevés du panneau.

Si une capture F988 est traitée alors que seules les données DAQ sont disponibles, Chloros rejette la réflectance basée sur les données DAQ pour cette bande avec le motif d’exclusion `dls-uncalibrated-band-988` — le [workflow du panneau de réflectance](../calibration-targets.md) est la procédure prise en charge pour le F988.

***

## Identifiants des capteurs

Chaque DAQ communique un identifiant de capteur stable. Son format varie selon le modèle :

| Modèle | Format de l’identifiant | Exemple |
| --- | --- | --- |
| DAQ-U | 5 octets séparés par des tirets | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octets séparés par des tirets | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

L'identifiant du capteur est :

* gravé dans chaque fichier `.daq` qu'il enregistre,
* la clé utilisée par Chloros pour récupérer le pack d'étalonnage d'usine de cet appareil,
* la valeur que vous transmettez à `--sensor-id` dans les commandes CLI `pool-*`, et
* pour le DAQ-E, également son nom d’hôte mDNS (`daq-e-def330.local`) — la valeur que `--eth-host` accepte.

***

## Étalonnage d’usine et cloud

Chaque unité DAQ est étalonnée individuellement en usine à l’aide d’une chaîne radiométrique traçable selon les normes du NIST, et Chloros charge le fichier d’étalonnage de chaque unité en fonction de l’identifiant de son capteur. Le rapport d’étalonnage propre à chaque unité (PDF) peut être téléchargé à partir des paramètres du capteur dans l’[onglet Capteurs de lumière](gui.md).

{% hint style="warning" %}
**Les modèles DAQ-U et DAQ-M nécessitent un accès au cloud pour l’étalonnage.**Aucun des deux modèles ne stocke de données en local : leurs fichiers d’étalonnage d’usine sont hébergés dans le cloud d’MAPIR et sont récupérés à l’aide de l’identifiant du capteur (puis mis en cache localement). Chloros a besoin d’une connexion Internet pour fournir des données calibrées en W/m²/nm provenant d’un DAQ-U ou d’un DAQ-M.**Le DAQ-E fait exception** : il intègre son étalonnage directement sur l’appareil.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Où sont enregistrés les enregistrements

| Surface | Destination `.daq` par défaut |
| --- | --- |
| Interface graphique — onglet « Capteurs de lumière » | `<project folder>/light_sensor/` (les enregistrements terminés sont automatiquement ajoutés au projet) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` sur la machine exécutant le backend (un chemin relatif `-o` est résolu par rapport au dossier dans lequel vous avez exécuté la commande lorsque le backend se trouve sur le même ordinateur) |

Chaque nom de fichier `.daq` comprend l’identifiant du capteur et un horodatage.

***

## Dans ce chapitre

* [**L'onglet DAQ dans Chloros**](gui.md) — présentation complète de l'interface graphique : connexion de chaque modèle, paramètres par capteur, graphiques spectraux, données colorimétriques en temps réel, réflectance à double capteur et enregistrement.
* [**Guide de démarrage rapide de la CLI (pool-\*)**](cli-quick-start.md) — pilotage des capteurs DAQ à partir de `chloros-cli daq pool-*`, le chemin d’accès en ligne de commande pris en charge.
* [**Profils de capteurs et plage étalonnée**](caps-and-range.md) — quels capteurs existent pour chaque modèle, comment les déclarer et la plage spectrale étalonnée en détail.
* [**Enregistrement et format .daq**](recording.md) — le format SQLite `.daq` et les workflows d’enregistrement.
* [**Mise en réseau DAQ-E et synchronisation temporelle**](ethernet-ptp.md) — modes de transport DAQ-E et synchronisation temporelle PTP.
* [**Workflows de réflectance**](reflectance.md) — utilisation des données DAQ descendantes pour calculer la réflectance.
* Pour une documentation complète au niveau des indicateurs, consultez la [Référence CLI](../reference/cli-reference.md) (section `chloros-cli daq`) et la [Référence SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), toutes deux rédigées pour être directement exploitables par des assistants IA.
