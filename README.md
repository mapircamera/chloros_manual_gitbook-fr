---
metaLinks: {}
---

# Mise en route

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

est une application logicielle de [MAPIR

](https://www.mapir.camera) permettant de traiter des images multispectrales, de contrôler en temps réel le matériel d&#x27;MAPIR

, et d&#x27;enregistrer les données des capteurs.Chloros

1.2.0 prend en charge l&#x27;ensemble de la gamme de produitsMAPIR

:

* **CamérasSurvey3** — traitent les captures RAW+JPG pour obtenir des cartes calibrées de réflectance et d’indices de végétation. Voir [Caméras prises en charge](supported-cameras.md).
* **Caméras LATTICE** — connectez les modules de caméras multispectrales GigE en temps réel, individuellement ou sous forme de réseaux multicaméras synchronisés : prévisualisez, capturez et traitez les données pour obtenir des produits de radiance et de réflectance calibrés. Consultez la [section LATTICE](lattice/README.md).
* **Capteurs de lumière DAQ** — Capteurs spectraux DAQ-U (USB), DAQ-M (Bluetooth) et DAQ-E (Ethernet) : spectres calibrés en temps réel, enregistrements `.daq` et éclairement descendant pour le traitement de la réflectance. Consultez la [section DAQ](daq/README.md).

{% hint style="success" %}
**Nouveautés de la version 1.2.0 d’Chloros** : contrôle en temps réel des caméras et des réseaux LATTICE, intégration des capteurs de lumière DAQ, modes de capture et enregistreurs, pipeline complet de traitement radiométrique LATTICE, automatisation des projets depuisCLI

/SDK

, et bien plus encore. Consultez la liste des nouveautés ci-dessous et [téléchargez](download.md) le journal des modifications.
{% endhint %}

{% hint style="info" %}
**Vous utilisezChloros

avec un assistant IA ?** Ce manuel est conçu pour cela. Indiquez à votre assistant :

* `https://mapir.gitbook.io/chloros/llms.txt` — index lisible par machine de toutes les pages.
* N&#x27;importe quelle page au format Markdown brut — ajoutez `.md` à sonURL

(par exemple, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [RéférenceCLI

](reference/cli-reference.md) et la [RéférenceSDK

](reference/sdk-reference.md) — des pages de référence complètes, contenant des valeurs exactes, rédigées pour être utilisées par les LLM.

Exemple de prompt : *« Lisez https://mapir.gitbook.io/chloros/reference/cli-reference.md,, puis rédigez un script qui se connecte et traite le dossier ~/flights/flight_001 pour le convertir en fichiers GeoTIFF de réflectance et d’NDVI

. »*

Guide complet : [Utilisation d’Chloros

avec des assistants IA](ai-assistants.md).
{% endhint %}

***

## Nouveautés de la version 1.2.0 d’Chloros



* **Contrôle en direct des caméras — nouvel onglet « Caméras ».** Connectez des caméras LATTICE une par une ou sous forme de réseaux multicaméras synchronisés (synchronisation temporelle PTP, capture déclenchée par le matériel), avec superposition d’aperçus en temps réel, histogrammes par bande, exposition automatique intelligente, calculateur d’indice en temps réel et mises à jour du micrologiciel des caméras directement depuis l’application.
* **Capteurs de lumière — nouvel onglet « Capteurs de lumière ».** Connectez des capteurs DAQ-U (USB), DAQ-M (Bluetooth) et DAQ-E (Ethernet) ; affichez des spectres calibrés en temps réel (W/m²/nm), enregistrez des fichiers `.daq` dans votre projet, sélectionnez des profils de correction de capacité et mettez à jour le micrologiciel du DAQ-E via le réseau.
* **Modes de capture et enregistreurs.** Capture unique / continue / par intervalle, plus un mode « Capture la plus rapide » en format brut uniquement ; sélection par projet des caméras et des types d’exportation générés par « Capture tout » ; enregistreurs matriciels pour des vidéos indexées de qualité surveillance et des rafales brutes de qualité analyse avec création de vidéos hors ligne.
* **Pipeline de traitement LATTICE.** Importez des dossiers de capture LATTICE et décomposez chaque image brute en produits débayérisés, de prévisualisation, de radiance en float32 (W/m²/sr/nm) et de réflectance, avec des boutons de basculement par produit. La réflectance peut provenir d’une cible d’étalonnage intégrée à l’image ou d’un flux descendant DAQ ; l’alignement de la matrice est appliqué aux exportations ; l’étalonnage d’usine manquant est téléchargé automatiquement en fonction du numéro de série de la caméra.
* **Les projets mémorisent la configuration matérielle.** Les caméras et capteurs de lumière connectés sont enregistrés avec le projet (`cameras.json` / `sensors.json`) et se reconnectent avec leurs paramètres enregistrés lorsque vous rouvrez le projet. Voir [Interface graphique : Projets](projects.md).
* **Améliorations de la visionneuse d’images.** Affichage des pixels/indices du curseur avec mise à l’échelle correcte de la réflectance par fichier, histogrammes par couche, curseur de regroupement GSD, modes de grille « Par déclenchement » / « Par caméra », vues des produits LATTICE et exportations vers le disque de la zone de test Index/LUT.
* **Fonctions «CLI

» et «SDK

» (Traitement par lot), considérablement étendues.** Nouvelles familles de commandes `lattice`, `daq pool-*`, `project` et `time-sync` ; nouvelles options `process` (`--input-level`, commutateurs par produit, `--reflectance-source`, indicateurs d’alignement de tableaux) ;SDK

des descripteurs « smart-connect » (`connect_camera` / `connect_array` / `connect_daq_sensor`) qui lancent automatiquement le backend ; automatisation `open_project()` ; la roueSDK

est intégrée aux programmes d’installation et publiée sur PyPI sous le nom `chloros-sdk`.
* **Sémantique d’échec transparente.** Une exécution de `chloros-cli process` qui a demandé des produits mais n’en a créé aucun échoue désormais de manière évidente et se termine avec un code de sortie différent de zéro ; les exécutions réussies indiquent le nombre de produits d’image créés.
* **Nouvelle structure de sortie.** Les produits sont placés dans des dossiers `<project>/<camera>/<format>/<Product>_Images/` et conservent le nom de fichier source — c’est le dossier, et non un suffixe de nom de fichier, qui identifie le produit. Voir [Formats d’images de sortie](output-image-formats.md).
* **Davantage de sources d’entrée, de plans et de langues.** Prise en charge des sources d’entrée `.dng` ; les 38 langues d’interface sont désormais entièrement disponibles ; limites matérielles par plan avec une utilisation gratuite (sans connexion) pouvant aller jusqu’à 4 caméras et 2 capteurs de lumière.
* **Fiabilité.** La fonction « Arrêter le traitement » se termine proprement avec un résumé précis de l’exécution, les projets multi-caméras exportent toutes les caméras, et les mises à jour du programme d’installation ne vous déconnectent plus.***

Chloros

est disponible sous 3 formes d’application :

##Chloros

: application GUI de bureau

Fenêtre autonome distincte offrant toutes les fonctionnalités, y compris les onglets « Caméras en direct » et « Capteurs de lumière ». _Windows uniquement._

## [Chloros

CLI

: interface en ligne de commande](CLI.md)

Traitement par lots en ligne de commande et commandes en temps réel `lattice`, `daq pool-*`, `project` et `time-sync`. Idéal pour l’automatisation, la création de scripts et le fonctionnement sans interface graphique. Disponible sur **Windows

,Linux

amd64 etLinux

arm64 (NVIDIA Jetson)**. _L’accès à la CLI nécessite un abonnement payant de niveau «Chloros

+ »._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Interface d’Python

programmatique pour l’automatisation et les flux de travail personnalisés : traitement complet du pipeline, sessions en direct avec caméra/réseau de caméras, sessions avec capteurs DAQ et automatisation des projets enregistrés. Installée avec le package « desktop/CLI

» et également publiée sous la référence `pip install chloros-sdk`. _L&#x27;accès à l&#x27;API nécessite un abonnement payant de niveau «Chloros

+ »._

***

## Plateformes prises en charge

| Plateforme | Interface graphique |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Oui | Oui | Oui |
| **Linux

amd64 (x86_64)** | Non | Oui | Oui |
| **Linux

arm64 (NVIDIA Jetson)** | Non | Oui | Oui |

Pour obtenir des instructions d’installation d’Linux

, consultez la section [Linux

et Edge Computing](linux/linux-overview.md).

***

## Pour commencer en trois étapes

1. **Installation** — téléchargez et exécutez le programme d’installation correspondant à votre plateforme. Consultez la page [Téléchargement](download.md).
2. **Connectez-vous (facultatif pour l’interface graphique)** — l’interface graphique traite gratuitement les images sans compte. Une [Chloros

+ connexion](chloros+-login.md) débloque le traitement parallèle, l’accélération GPU, des limites de périphériques plus élevées et l’accès àCLI

/SDK

.
3. **Créez votre premier projet** — ouvrezChloros

, créez un [Nouveau projet](projects.md), [ajoutez vos images](processing-images-gui/adding-files-to-a-project.md) et [lancez le traitement](processing-images-gui/starting-the-processing.md). Pour piloter du matériel en temps réel, ouvrez l’onglet « Caméras » ou « Capteurs de lumière » — voir [GUI : Navigation](navigation.md).

***

##Chloros

+

Bien qu’Chloros

soit gratuit pour la plupart des tâches, vous pourriez avoir besoin de fonctionnalités supplémentaires. C’est là qu’une licence payante pourChloros

+ peut vous être utile. Avec une licenceChloros

+, vous pouvez débloquer de nouvelles fonctionnalités telles que :

* **Traitement multithread** : accélérez considérablement le traitement des images pour les projets de grande envergure en traitant simultanément les images tout au long du pipeline.
* **Accélération GPU (CUDA)** : tirez parti des capacités de mémoire GPU plus élevées disponibles aujourd’hui pour accélérer encore davantage le pipeline de traitement d’images. Nous recommandons au moins 4 Go de VRAM pour obtenir les meilleurs résultats.
* **Accès àChloros

+**[**CLI**](CLI.md) : exécutezChloros

+ depuis la ligne de commande pour automatiser et intégrer le logiciel à vos propres applications. Disponible sur tous les abonnements payants ; appliqué côté serveur.
* **Chloros

+**[**API**](api-python-sdk.md) **Accès :** exécutezChloros

+ depuisPython

pour un contrôle programmatique, permettant une intégration transparente avec vos pipelines de recherche, vos workflows d’analyse de données et vos applications personnalisées. Disponible sur tous les forfaits payants ; appliqué côté serveur.
* **Limites matérielles plus élevées** : connectez davantage de caméras et de capteurs de lumière simultanément. Sans connexion, l’interface graphique permet de connecter jusqu’à 4 caméras et 2 capteurs de lumière DAQ ; les formules payantes augmentent ces deux limites :

| Formule | Caméras | Capteurs de lumière DAQ |
| --- | --- | --- |
| Iron (gratuit, sans connexion) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Utilisation de plusieurs appareils** : chaque licence «Chloros

+ » permet d’enregistrer au moins 2 appareils. Utilisez votre compte CloudMAPIR

pour gérer les appareils enregistrés. Ajoutez la prise en charge d’appareils supplémentaires en mettant à niveau votre licence «Chloros

+ ».
* **Méthode avancée de débayérisation sensible à la texture :** une débayérisation de haute qualité sensible aux contours, combinée à un modèle de débruitage basé sur l’IA/ML qui élimine la quasi-totalité du bruit de débayérisation.
* **Formules d’indices multispectraux personnalisées :** saisissez des indices multispectraux personnalisés dans les calculateurs de raster deChloros

, tant pour le traitement que pour l’environnement de test de visualisation d’images.
* **«Linux

» et « Edge Computing » :** exécutez «Chloros

» sur les plateformes x86_64 et ARM64 deLinux

, y compris NVIDIA Jetson, pour le traitement sur le terrain et en périphérie. Consultez la [présentation deLinux

](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Tarifs et inscription</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
