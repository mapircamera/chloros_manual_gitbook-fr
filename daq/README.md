# Capteurs de lumière DAQ

> **Vous cherchez des informations sur le matériel ?**Les capteurs eux-mêmes (modèles, fixation, capuchons, ports, alimentation et application SCANNER) sont décrits dans le**[manuel d&#x27;utilisation DAQ](https://mapir.gitbook.io/daq)**. Ce chapitre traite de leur utilisation à partir de la version Chloros.

Les capteurs de lumière **DAQ** de l&#x27;MAPIR mesurent la lumière ambiante sous forme de spectres étalonnés radiométriquement. Dans l&#x27;Chloros, ils jouent deux rôles :

* **Un instrument spectral autonome** — graphiques spectraux en temps réel, données colorimétriques et enregistrements `.daq`, le tout accessible depuis l’[onglet « Capteurs de lumière »](gui.md), le [CLI](cli-quick-start.md) ou le Python SDK.
* **Une source d’irradiance descendante pour la réflectance** — lors du traitement, Chloros interpole vos mesures `.daq` à chaque horodatage d’exposition de la capturehorodatage d’exposition de chaque capture et utilise la lumière descendante mesurée pour convertir la radiance de la caméra en réflectance (`--reflectance-source daq`) ; aucun panneau de scène n’est nécessaire pour les bandes calibrées.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Trois modèles, un seul format de données

| Modèle | Transport | Découverte |
| --- | --- | --- |
| **DAQ-U** | USB (série) | balayage du port série |
| **DAQ-M** | Bluetooth Low Energy | balayage BLE par nom |
| **DAQ-E** | Ethernet (IPv4, alimentation PoE) | mDNS `_daq-e._tcp` (nom d&#x27;hôte `daq-e-<id>.local`) |

Les trois modèles utilisent le même protocole de communication et fournissent des données identiques :

* Un **spectre de 135 points compris entre 340 et 1 010 nm par pas de 5 nm**, ainsi que les valeurs tristimulus CIE XYZ, dans chaque trame.
* **Une irradiance spectrale étalonnée radiométriquement en W/m²/nm** — le jeu d’étalonnage d’usine de chaque appareil (ainsi que son profil de correction de capuchon actif) est appliqué avant que les données ne vous parviennent.
* Le même **format d’enregistrement `.daq`** (un fichier SQLite). Le traitement en aval est identique, quel que soit le protocole de transport ayant généré le fichier.

Les piles de transport (série USB, BLE, mDNS/zeroconf) sont intégrées au backend Chloros — il n’y a rien à installer pour communiquer avec l’un des trois modèles depuis l’interface graphique ou via les commandes `pool-*` de CLI.

***

## Plage étalonnée : 340–1010 nm signalée, ~374–974 nm étalonnée

Le capteur signale la grille complète de 340 à 1010 nm, mais le gain radiométrique traçable selon les normes NIST s’étend approximativement de **374 à 974 nm**. Chloros refuse la division par réflectance absolue pour toute bande de caméra dont moins de la moitié de la pondération spectrale se situe dans cette plage étalonnée ; la bande ignorée est signalée avec le motif d’ignorance `dls-uncalibrated-band-<nm>`.

Parmi les références de filtres LATTICE disponibles à la vente, seule la **F988** est concernée :

La réflectance du F988 est calibrée à l’aide d’un panneau de réflectance intégré à la scène : la bande se situant au-delà de la plage calibrée du capteur de lumière DAQ, Chloros applique votre dernière capture du panneau et la conserve entre deux observations du panneau.

Si une capture F988 est traitée alors que seules les données DAQ sont disponibles, Chloros rejette la réflectance basée sur le DAQ pour cette bande avec le motif d’exclusion `dls-uncalibrated-band-988` — le [workflow du panneau de réflectance](../calibration-targets.md) est la procédure prise en charge pour le F988.

***

## Identifiants de capteurs

Chaque DAQ renvoie un identifiant de capteur stable. Son format varie selon le modèle :

| Modèle | Format de l’identifiant | Exemple |
| --- | --- | --- |
| DAQ-U | 5 octets séparés par des tirets | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octets séparés par des tirets | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

L&#x27;identifiant du capteur est :

* gravé dans chaque fichier `.daq` qu&#x27;il enregistre,
* la clé que Chloros utilise pour récupérer le pack d&#x27;étalonnage d&#x27;usine de cet appareil,
* la valeur que vous transmettez à `--sensor-id` dans les commandes CLI `pool-*`, et
* pour le DAQ-E, également son nom d’hôte mDNS (`daq-e-def330.local`) — la valeur acceptée par `--eth-host`.

***

## Étalonnage d’usine et cloud

Chaque unité DAQ est étalonnée individuellement en usine à l’aide d’une chaîne radiométrique traçable selon les normes du NIST, et Chloros charge le fichier d’étalonnage de chaque unité en fonction de l’identifiant de son capteur. Le rapport d’étalonnage propre à chaque unité (PDF) peut être téléchargé à partir des paramètres du capteur dans l’[onglet Capteurs de lumière](gui.md).

{% hint style="warning" %}
**Les modèles DAQ-U et DAQ-M nécessitent un accès au cloud pour l’étalonnage.**Aucun des deux modèles ne stocke de données en local : leurs ensembles d’étalonnage d’usine sont hébergés dans le cloud de MAPIR et sont récupérés à l’aide de l’identifiant du capteur (puis mis en cache localement). Chloros a besoin d’une connexion Internet pour fournir des données calibrées en W/m²/nm provenant d’un DAQ-U ou d’un DAQ-M.**Le DAQ-E fait exception** : il intègre son étalonnage directement sur l’appareil.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Où sont enregistrées les données

| Surface | Destination par défaut pour `.daq` |
| --- | --- |
| Interface graphique — Onglet « Capteurs de lumière » | `<project folder>/light_sensor/` (les enregistrements terminés sont automatiquement ajoutés au projet) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` sur la machine exécutant le backend |

Chaque nom de fichier `.daq` comprend l&#x27;identifiant du capteur et un horodatage.

***

## Dans ce chapitre

* [**L&#x27;onglet DAQ dans Chloros**](gui.md) — présentation complète de l’interface graphique : connexion de chaque modèle, paramètres par capteur, graphiques de spectre, données colorimétriques en temps réel, réflectance à double capteur et enregistrement.
* [**Guide de démarrage rapide de CLI (pool-\*)**](cli-quick-start.md) — pilotage des capteurs DAQ à partir de `chloros-cli daq pool-*`, le chemin d’accès en ligne de commande pris en charge.
* [**Profils de seuils et plage calibrée**](caps-and-range.md) — quels seuils existent pour chaque modèle, comment les déclarer, et la plage spectrale calibrée en détail.
* [**Enregistrement et format .daq**](recording.md) — le format SQLite `.daq` et les workflows d&#x27;enregistrement.
* [**Mise en réseau DAQ-E et synchronisation temporelle**](ethernet-ptp.md) — modes de transport DAQ-E et synchronisation temporelle PTP.
* [**Workflows de réflectance**](reflectance.md) — utilisation des données DAQ descendantes pour calculer la réflectance.
* Pour une documentation complète au niveau des indicateurs, consultez la [référence CLI](../reference/cli-reference.md) (section `chloros-cli daq`) et la [référence SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), toutes deux rédigées pour être directement exploitables par des assistants IA.
