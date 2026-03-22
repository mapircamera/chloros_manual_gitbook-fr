# Présentation de Linux

Chloros 1.1.0 apporte une prise en charge native de Linux pour les modèles **CLI**et**Python SDK**, permettant le traitement d&#x27;images multispectrales sans interface graphique sur les stations de travail, les serveurs et les périphériques de bord NVIDIA Jetson.

{% hint style="info" %}
**Pas d&#x27;interface graphique sur Linux.** L&#x27;interface graphique de bureau Chloros est disponible uniquement sur Windows. Les utilisateurs de Linux interagissent avec Chloros via [CLI](../CLI.md) et [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matrice de prise en charge des plateformes

| Fonctionnalité | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interface graphique de bureau** | Oui | N/A | Non | Non |
| **CLI** | Oui | Oui | Oui | Oui |
| **Python SDK** | Oui | Oui | Oui | Oui |
| **Accélération GPU (CUDA)** | Oui | Oui | Oui | Oui (JetPack 6) |
| **Débayérisation sensible aux textures** | Oui (Chloros+) | Oui (Chloros+) | Oui (Chloros+) | Oui (Chloros+) |
| **Adaptation dynamique des calculs** | Oui | Oui | Oui | Oui |***

## Architectures prises en charge

| Architecture | Description | Méthode d&#x27;installation |
| --- | --- | --- |
| **amd64 (x86_64)** | Processeurs de bureau/serveur standard (Intel, AMD) | Paquet `.deb` |
| **arm64 (aarch64)** | Processeurs basés sur ARM, principalement NVIDIA Jetson | Paquet `.deb` (JetPack 6) |

## Distributions Linux prises en charge

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — plateformes Jetson)***

## Ce dont bénéficient les utilisateurs de Linux

* **Chloros CLI** — Interface en ligne de commande complète pour le traitement par lots, l&#x27;automatisation et la création de scripts
* **Chloros Python SDK** — Interface programmatique Python (`pip install chloros-sdk`) pour l&#x27;intégration dans des pipelines de recherche et des outils personnalisés
* **Accélération GPU** — Traitement accéléré par CUDA sur les GPU NVIDIA (ordinateurs de bureau et Jetson)
* **Adaptation dynamique du calcul** — Détection automatique du matériel et optimisation de la stratégie de traitement
* **Toutes les fonctionnalités de traitement** — Même pipeline de traitement multispectral que Windows (étalonnage, correction de vignettage, indices de végétation, tous les formats d&#x27;exportation)
* **Fonctionnalités de Chloros+** — Traitement multithread, débayérisation sensible à la texture, indices personnalisés (avec la licence Chloros+)

## Ce dont les utilisateurs de Linux ne disposent pas

* **Interface graphique de bureau** — Pas d&#x27;interface graphique ; toutes les interactions se font via CLI ou Python SDK
* **Visionneuse d&#x27;images** — Pas de visionneuse d&#x27;images interactive, de vue en grille ni de marqueurs sur la carte
* **Gestion visuelle des projets** — Les projets sont gérés via les commandes CLI et les appels SDK***

## Premiers pas avec Linux

1. **Installer Chloros** — Voir [Installation de Linux](linux-installation.md) pour l&#x27;installation du package `.deb`
2. **Installez Python SDK** (facultatif) — `pip install chloros-sdk`
3. **Activez votre licence** — `chloros-cli login your@email.com 'password'`
4. **Traitez votre premier ensemble de données** — `chloros-cli process ~/datasets/flight001`

Pour les utilisateurs de NVIDIA Jetson, consultez le [Guide NVIDIA Jetson](nvidia-jetson-guide.md) dédié pour la configuration et l&#x27;optimisation spécifiques à la plateforme.

***

## Étapes suivantes

* [Installation de XPROTX](linux-installation.md) — Instructions d&#x27;installation détaillées pour amd64 et arm64
* [Guide NVIDIA Jetson](nvidia-jetson-guide.md) — Configuration spécifique à Jetson, gestion thermique et déploiement sur le terrain
* [CLI : Ligne de commande](../CLI.md) — Référence complète de CLI
* [API : Python SDK](../api-python-sdk.md) — Référence complète de SDK
* [Adaptation dynamique des calculs](../processing-architecture/dynamic-compute-adaptation.md) — Comment Chloros s&#x27;adapte à votre matériel
