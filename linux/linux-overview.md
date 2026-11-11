# Présentation de Linux

Chloros 1.3.0 offre une prise en charge native sous Linux de la **CLI**et du**Python SDK** — traitement d’images multispectrales sans affichage, ainsi que le contrôle en temps réel des caméras LATTICE et des capteurs de lumière DAQ — sur les postes de travail, les serveurs et les appareils périphériques NVIDIA Jetson sous Linux.

{% hint style="info" %}
**Pas d’interface graphique de bureau sous Linux.**L’interface graphique de bureau Chloros est réservée à Windows. Les utilisateurs Linux interagissent avec Chloros via la [CLI](../CLI.md) et le [Python SDK](../api-python-sdk.md). Le `.deb` ajoute bien une entrée**Chloros CLI** à votre menu d’applications — il ouvre simplement un émulateur de terminal exécutant `chloros-cli`.
{% endhint %}

***

## Tableau de prise en charge des plateformes

| Fonctionnalité | Windows (interface graphique) | Windows (CLI/SDK) | Linux amd64 (CLI/ --- | --- | --- | --- |
| **Interface graphique de bureau** | Oui | N/A | Non | Non |
| **Interface CLI** (`chloros-cli`) | Oui | Oui | Oui | Oui |
| **SDK Python** (`chloros-sdk`) | Oui | Oui | Oui | Oui |
| **Pipeline de traitement d'images** | Oui | Oui | Oui | Oui |
| **Contrôle des caméras LATTICE (en direct)** | Oui (onglet « Caméras ») | Oui (`chloros-cli lattice`, SDK) | Oui | Oui |
| **Capteurs de lumière DAQ (en direct)** | Oui (onglet « Capteurs de lumière ») | Oui (`chloros-cli daq pool-*`, SDK) | Oui | Oui |
| **Synchronisation temporelle PTP (l'hôte est le « grand maître »)** | Oui | Oui (`chloros-cli time-sync`) | Oui | Oui |
| **Accélération GPU (CUDA)** | Oui | Oui | Oui | Oui (JetPack 6) |
| **Débayérisation sensible aux textures** (nécessite un GPU CUDA) | Oui (Chloros+) | Oui (Chloros+) | Oui (Chloros+) | Oui (Chloros+) |
| **Adaptation dynamique des calculs** | Oui | Oui | Oui | Oui |
| **Backend en tant que service système** (`chloros-backend@<user>.service`) | Non | Non | Oui (activation optionnelle) | Oui (activation optionnelle) |
| **Mise à jour sur place** (`chloros-cli update`) | Non (exécuter le programme d'installation) | Non (exécuter le programme d'installation) | Oui | Oui |***

## Architectures prises en charge

| Architecture | Description | Paquet |
| --- | --- | --- |
| **amd64 (x86_64)** | Processeurs standard pour ordinateurs de bureau/serveurs (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Processeurs ARM — Famille NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Distributions Linux prises en charge

* **Ubuntu 22.04 LTS ou version plus récente** (amd64)
* **Debian 12 ou version plus récente** (amd64)
* **NVIDIA JetPack 6** (arm64 — plateformes Jetson Orin)***

## Ce dont bénéficient les utilisateurs Linux

* **Chloros CLI** — l’interface de ligne de commande complète pour le traitement par lots, l’automatisation et la création de scripts
* **Chloros Python SDK** — interface Python programmatique pour les pipelines de recherche et les outils personnalisés, installée par le paquet `.deb` pour le système `python3` (un « wheel » de version correspondante pour les environnements virtuels est disponible dans `/usr/lib/chloros/sdk/` ; PyPI est destiné aux machines ne disposant pas du paquet Chloros)
* **Contrôle des caméras LATTICE** — détection, connexion, configuration et acquisition à partir de caméras LATTICE et de réseaux multicaméras synchronisés via `chloros-cli lattice` et le SDK ; le paquet `.deb` intègre le runtime Arena SDK requis par les caméras
* **Contrôle des capteurs de lumière DAQ** — connectez des capteurs DAQ-U/M/E, diffusez des spectres étalonnés et enregistrez des fichiers `.daq` via `chloros-cli daq pool-*` et l’SDK
* **Synchronisation temporelle PTP** — le backend Chloros exécute le « grandmaster » PTP auquel les caméras LATTICE et les capteurs DAQ-E sont asservis ; vérifiez-le avec `chloros-cli time-sync` et assurez son fonctionnement en mode headless grâce à l’unité systemd `chloros-backend@<user>.service` (voir [Installation sous Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatisation des projets** — exécutez les projets enregistrés en mode sans interface graphique avec `chloros-cli project` et le SDK `open_project`
* **Accélération GPU** — Traitement accéléré par CUDA sur les GPU NVIDIA (GPU de bureau sur amd64 et GPU du Jetson avec JetPack 6)
* **Adaptation dynamique du calcul** — détection automatique du matériel et sélection de la stratégie de traitement, avec la fonction de remplacement `CHLOROS_STRATEGY` comme solution de secours pour les experts
* **Toutes les fonctionnalités de traitement** — le même pipeline que sous Windows : étalonnage, correction de la vignettage, indices de végétation et tous les formats d’exportation
* **Fonctionnalités Chloros+** — traitement multithread (en pipeline), débayérisation « Texture Aware » (sur un GPU CUDA) et indices personnalisés, avec un abonnement payant à Chloros+

## Ce dont les utilisateurs d’Linux ne disposent pas

* **Interface graphique de bureau** — pas d’interface graphique ; toutes les interactions s’effectuent via la CLI ou le Python SDK
* **Visionneuse d’images** — pas de visionneuse d’images interactive, de vue en grille ni de marqueurs sur la carte
* **Gestion visuelle des projets** — les projets sont créés et pilotés via des commandes CLI et des appels SDK (le matériel lui-même — caméras, capteurs, capture — reste entièrement contrôlable depuis le terminal)***

## Conditions de licence

L’accès à la CLI et au SDK nécessite un **forfait Chloros+ payant — Copper ou supérieur**(Copper, Bronze, Silver, Gold). Le forfait gratuit**Iron** ne donne pas accès à la CLI ni au SDK. Cette restriction est appliquée par le backend, et pas seulement par la CLI :

| Situation | Réponse du backend |
| --- | --- |
| Non connecté | `401` avec `error_code: AUTH_REQUIRED` |
| Connecté au niveau gratuit Iron | `403` avec `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` fonctionne sur tous les niveaux — c’est la seule route exemptée du contrôle —, de sorte que la raison d’un refus est toujours visible.

***

## Mise en route sous Linux

1. **Installez Chloros** — consultez [Installation sous Linux](linux-installation.md) pour l'installation de `.deb`
2. **Vérification** — `chloros-cli --version` affiche `Chloros CLI 1.3.0` ; `chloros-cli selftest` exécute le diagnostic en 7 étapes
3. **Vérifiez le SDK Python** — l’installation de `.deb` le fournit : `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (dans un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Connectez-vous** — `chloros-cli login your@email.com 'your-password'` (une fois par machine ; les mises à jour du paquet conservent les identifiants de connexion, la suppression du paquet les efface)
5. **Traitez votre premier ensemble de données** — `chloros-cli process ~/datasets/flight001`

Pour NVIDIA Jetson, consultez le [Guide NVIDIA Jetson](nvidia-jetson-guide.md) dédié pour la configuration spécifique à la plateforme, le comportement thermique et le déploiement sur le terrain.

***

## Étapes suivantes

* [Installation sous Linux](linux-installation.md) — installation détaillée, emplacements des fichiers et dépannage pour amd64 et arm64
* [Guide NVIDIA Jetson](nvidia-jetson-guide.md) — configuration spécifique à Jetson, comportement en matière de mémoire et de gestion thermique, déploiement sur le terrain
* [CLI : ligne de commande](../CLI.md) — le guide de l'interface de ligne de commande
* [API : Python SDK](../api-python-sdk.md) — le guide du SDK
* [Référence CLI](../reference/cli-reference.md) et [Référence SDK](../reference/sdk-reference.md) — listes exhaustives des commandes et des API pour la version 1.3.0
* [Adaptation dynamique des ressources de calcul](../processing-architecture/dynamic-compute-adaptation.md) — comment Chloross s'adapte à votre matériel

{% hint style="info" %}
**Lecture de ce manuel par programmation.** Chaque page est également disponible au format Markdown brut à son propre URL, à laquelle s'ajoute `.md` (par exemple `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), et un index de l'ensemble du manuel est publié à l'adresse [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
