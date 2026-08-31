# Présentation de Linux

Chloros 1.2.0 offre une prise en charge native de Linux pour **CLI**et**Python SDK** — traitement d’images multispectrales sans affichage, ainsi que le contrôle en temps réel des caméras LATTICE et des capteurs de lumière DAQ — sur les stations de travail, les serveurs et les périphériques de bord NVIDIA Jetson.

{% hint style="info" %}
**Pas d’interface graphique de bureau sur Linux.**L’interface graphique de bureau Chloros est réservée à Windows. Les utilisateurs de Linux interagissent avec Chloros via [CLI](../CLI.md) et [Python SDK](../api-python-sdk.md). `.deb` ajoute bien une**Chloros CLI** à votre menu d’application — il ouvre simplement un émulateur de terminal exécutant `chloros-cli`.
{% endhint %}

***

## Matrice de prise en charge des plateformes

| Fonctionnalité | Windows (interface graphique) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interface graphique de bureau** | Oui | N/A | Non | Non |
| **CLI** (`chloros-cli`) | Oui | Oui | Oui | Oui |
| **Python SDK** (`chloros-sdk`) | Oui | Oui | Oui | Oui |
| **Pipeline de traitement d&#x27;images** | Oui | Oui | Oui | Oui |
| **Contrôle des caméras LATTICE (en direct)** | Oui (onglet « Caméras ») | Oui (`chloros-cli lattice`, SDK) | Oui | Oui |
| **Capteurs de lumière DAQ (en temps réel)** | Oui (onglet « Capteurs de lumière ») | Oui (`chloros-cli daq pool-*`, SDK) | Oui | Oui |
| **Synchronisation temporelle PTP (l&#x27;hôte est le grand maître)** | Oui | Oui (`chloros-cli time-sync`) | Oui | Oui |
| **Accélération GPU (CUDA)** | Oui | Oui | Oui | Oui (JetPack 6) |
| **Débayérisation tenant compte des textures** | Oui (Chloros+) | Oui (Chloros+) | Oui (Chloros+) | Oui (Chloros+) |
| **Adaptation dynamique des calculs** | Oui | Oui | Oui | Oui |
| **Backend en tant que service système** (`chloros-backend.service`) | Non | Non | Oui (activation requise) | Oui (activation requise) |
| **Programme de mise à jour sur place** (`chloros-cli update`) | Non (exécuter le programme d&#x27;installation) | Non (exécuter le programme d&#x27;installation) | Oui | Oui |***

## Architectures prises en charge

| Architecture | Description | Paquet |
| --- | --- | --- |
| **amd64 (x86_64)** | Processeurs standard pour ordinateurs de bureau/serveurs (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Processeurs ARM — Gamme NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (version JetPack 6) |

## Distributions Linux prises en charge

* **Ubuntu 22.04 LTS ou version plus récente** (amd64)
* **Debian 12 ou version plus récente** (amd64)
* **NVIDIA JetPack 6** (arm64 — plateformes Jetson Orin)***

## Ce dont bénéficient les utilisateurs de Linux

* **Chloros CLI** — l’interface de ligne de commande complète pour le traitement par lots, l’automatisation et la création de scripts
* **Chloros Python SDK** — une interface Python programmatique pour les pipelines de recherche et les outils personnalisés (installable depuis PyPI, et également fournie dans le package `.deb` sous forme de fichier wheel compatible avec la version)
* **Contrôle des caméras LATTICE** — détection, connexion, configuration et acquisition d&#x27;images à partir de caméras LATTICE et de réseaux multicaméras synchronisés via `chloros-cli lattice` et SDK ; le `.deb` intègre le runtime Arena SDK requis par les caméras
* **Contrôle des capteurs de lumière DAQ** — connectez des capteurs DAQ-U/M/E, diffusez des spectres étalonnés et enregistrez des fichiers `.daq` via `chloros-cli daq pool-*` et SDK
* **Synchronisation temporelle PTP** — le backend Chloros exécute le « grandmaster » PTP auquel les caméras LATTICE et les capteurs DAQ-E sont asservis ; inspectez-le avec `chloros-cli time-sync`, et assurez son fonctionnement en mode sans interface graphique à l&#x27;aide de l&#x27;unité systemd `chloros-backend.service` (voir [Installation de Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatisation des projets** — exécutez les projets enregistrés en mode sans interface graphique à l’aide de `chloros-cli project` et de la fonction `open_project` de SDK
* **Accélération GPU** — traitement accéléré par CUDA sur les GPU NVIDIA (ordinateurs de bureau et Jetson)
* **Adaptation dynamique du calcul** — détection automatique du matériel et sélection de la stratégie de traitement, avec la commande `CHLOROS_STRATEGY` comme solution de secours pour les experts
* **Toutes les fonctionnalités de traitement** — même pipeline que Windows : étalonnage, correction du vignettage, indices de végétation et tous les formats d’exportation
* **Fonctionnalités de Chloros+** — traitement multithread (en pipeline), débayérisation « Texture Aware » et indices personnalisés, avec un abonnement payant à Chloros+

## Ce dont les utilisateurs de Linux ne bénéficient pas

* **Interface graphique de bureau** — pas d’interface graphique ; toutes les interactions se font via CLI ou Python SDK
* **Visionneuse d&#x27;images** — pas de visionneuse d&#x27;images interactive, d&#x27;affichage en grille ni de marqueurs sur carte
* **Gestion visuelle des projets** — les projets sont créés et pilotés via des commandes CLI et des appels SDK (le matériel lui-même — caméras, capteurs, capture — reste entièrement contrôlable depuis le terminal)***

## Conditions de licence

L&#x27;accès à CLI et SDK nécessite un **niveau Chloros+ payant — Copper ou supérieur**(Copper, Bronze, Silver, Gold). Le niveau gratuit**Iron** ne donne pas accès aux fonctions CLI/SDK. Cette restriction est appliquée par le backend, et pas uniquement par l’API CLI :

| Situation | Réponse du backend |
| --- | --- |
| Non connecté | `401` avec `error_code: AUTH_REQUIRED` |
| Connecté sur le niveau Iron gratuit | `403` avec `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` fonctionne sur tous les niveaux — c’est la seule route exemptée du contrôle d’accès —, de sorte que la raison d’un refus est toujours visible.

***

## Premiers pas avec Linux

1. **Installez Chloros** — consultez [Installation de Linux](linux-installation.md) pour l’installation de `.deb`
2. **Vérification** — `chloros-cli --version` affiche `Chloros CLI 1.2.0` ; `chloros-cli selftest` exécute le diagnostic en 7 étapes
3. **Installez Python et SDK** (facultatif) — `pip install chloros-sdk`
4. **Connectez-vous** — `chloros-cli login your@email.com 'your-password'` (une fois par machine, puis à nouveau après chaque mise à jour du package)
5. **Traitez votre premier ensemble de données** — `chloros-cli process ~/datasets/flight001`

Pour NVIDIA Jetson, consultez le [Guide NVIDIA Jetson](nvidia-jetson-guide.md) dédié pour connaître la configuration spécifique à la plateforme, le comportement thermique et le déploiement sur le terrain.

***

## Étapes suivantes

* [Installation de Linux](linux-installation.md) — installation détaillée, emplacements des fichiers et dépannage pour amd64 et arm64
* [Guide NVIDIA Jetson](nvidia-jetson-guide.md) — configuration spécifique à Jetson, comportement en matière de mémoire et de gestion thermique, déploiement sur le terrain
* [CLI : Ligne de commande](../CLI.md) — le guide CLI
* [API : Python SDK](../api-python-sdk.md) — le guide SDK
* [Référence CLI](../reference/cli-reference.md) et [Référence SDK](../reference/sdk-reference.md) — listes exhaustives des commandes/API pour la version 1.2.0
* [Adaptation dynamique des ressources de calcul](../processing-architecture/dynamic-compute-adaptation.md) — comment Chloros s&#x27;adapte à votre matériel

{% hint style="info" %}
**Lecture de ce manuel par programmation.** Chaque page est également disponible au format Markdown brut à l’adresse URL plus `.md` (par exemple `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), et un index de l’ensemble du manuel est publié à l’adresse [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
