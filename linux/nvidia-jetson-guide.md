# Guide NVIDIA Jetson

Chloros sur NVIDIA Jetson permet le traitement d’images multispectrales en périphérie — sur le terrain, sur des drones et dans des installations éloignées. Chloros 1.2.0 détecte votre modèle Jetson au démarrage et optimise sa stratégie de traitement en fonction du matériel détecté. **Aucun réglage manuel n&#x27;est nécessaire.**

***

## Modèles Jetson pris en charge

| Modèle                | Mémoire vive            | Stratégie de traitement                                     | Utilisation recommandée                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 Go partagés | `GPU_PARALLEL` (2 workers)                              | Performances maximales, grands ensembles de données                      |
| **Jetson Orin NX**   | 8 à 16 Go partagés  | `GPU_PARALLEL` (2 workers, 16 Go) / `GPU_SINGLE` (8 Go)   | Recommandation principale pour les déploiements aéroportés et sur le terrain |
| **Jetson Orin Nano** | 8 Go partagés     | `GPU_SINGLE` (1 nœud de traitement, séquentiel)                     | Calcul en périphérie d’entrée de gamme                                 |

{% hint style="info" %}
Le package arm64 Linux nécessite **JetPack 6**, disponible sur la gamme Jetson Orin. Les anciens modèles (Nano, TX2, Xavier NX) ne peuvent pas exécuter JetPack 6 et ne sont pas pris en charge par le package actuel.
{% endhint %}

***

## Configuration requise

* **JetPack 6.x** (dernière version recommandée)
* **NVIDIA CUDA** (inclus avec JetPack)
* **Formule Chloros+ payante** — niveau Copper ou supérieur (requis pour tout accès à CLI/SDK ; vérifié côté serveur)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Pour obtenir des informations générales sur l’installation de Linux, l’emplacement des fichiers et le dépannage, consultez la page [Installation de Linux](linux-installation.md).

{% hint style="info" %}
**Placez le répertoire d’extraction sur un support de stockage rapide.** Les binaires compilés se décompressent automatiquement dans un répertoire temporaire à chaque lancement — ce qui est extrêmement lent à partir d’une carte SD. Chloros utilise automatiquement `/mnt/ssd/tmp` lorsqu’il existe ; sinon, configurez `TMPDIR` pour qu’il pointe vers un chemin d’accès sur votre NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptation dynamique des ressources de calcul sur Jetson

### Fonctionnement

Au démarrage, Chloros analyse votre système :

1. **Détecte le modèle Jetson** via `/proc/device-tree/model`
2. **Lit la mémoire partagée disponible du GPU/CPU** (Jetson utilise une mémoire unifiée)
3. **Sélectionne une stratégie de traitement** (`GPU_PARALLEL`, `GPU_SINGLE` ou `CPU_PARALLEL`)
4. **Définit automatiquement le nombre de workers, le type de pipeline et l’allocation de mémoire**Le choix dépend de la**quantité totale de RAM partagée**, et non du nom du modèle :

* **Moins de 12 Go de RAM totale**(tous les Jetson de 8 Go) : `GPU_SINGLE` avec**1 worker — traitement séquentiel délibéré**. La mémoire est trop limitée pour des workers simultanés ; les images sont donc traitées une par une. Sur les Jetson dotés de**8 Go ou moins**, le thread 3 ignore complètement le pool de workers et exécute son travail image par image en cours de traitement.
* **12 Go ou plus**(Orin NX 16 Go, AGX Orin) : la mémoire unifiée est compatible avec `GPU_PARALLEL`, mais le nombre de workers est**plafonné à 2 sur les Jetson** — le GPU, la mémoire vive des processus de travail et leurs contextes CUDA par processus de travail puisent tous dans le même pool partagé ; ainsi, un nombre plus élevé de processus de travail augmente le risque d’erreurs de mémoire insuffisante.

Vous pouvez remplacer le choix automatique à l’aide de la variable d’environnement `CHLOROS_STRATEGY` — voir [Adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportement par modèle

| Modèle Jetson                | Stratégie       | Tâches | Exécution                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 Go**    | `GPU_SINGLE`   | 1       | Boucle séquentielle en cours de traitement (`tiled_gpu` en cas de pression mémoire) |
| **Jetson Orin NX 8 Go**      | `GPU_SINGLE`   | 1       | Boucle séquentielle au sein du processus                     |
| **Jetson Orin NX 16 Go**     | `GPU_PARALLEL` | 2       | Processus de travail concurrents, chemin `fused_gpu`  |
| **Jetson AGX Orin 32-64 Go** | `GPU_PARALLEL` | 2       | Processus de travail concurrents, chemin `fused_gpu`  |

La principale différence entre ces plateformes réside dans la **mémoire**. Un Jetson de 8 Go doit traiter les images une par une en utilisant une approche par mosaïques économe en mémoire lorsque la charge est élevée, tandis qu’un Orin de 16 Go ou plus peut traiter simultanément deux images via le GPU en utilisant le pipeline fusionné à haut débit.

### Budget GPU par modèle

Chaque modèle Jetson est également associé à un profil matériel qui limite la part de mémoire que le traitement du pool partagé peut revendiquer et adapte la taille des lots :

| Modèle | Plafond du budget GPU | Multiplicateur de taille de lot | Réservé au système/à l’affichage |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 Go |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 Go |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 Go |

La mémoire RAM détectée ajuste le profil : un Jetson signalant **16 Go ou plus** voit son multiplicateur de lot augmenté de ×1,2. La taille de lot de base avant application des multiplicateurs est de 8 images.

Pour consulter la référence complète sur l’adaptation de calcul, voir [Adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md).

***

## Limitation de fréquence du GPU pour Texture Aware sur Nano et Orin Nano

Le débayeur Texture Aware exécute une inférence de réseau neuronal sur le GPU, ce qui peut déclencher des **avertissements de surintensité**sur les modèles Jetson à faible consommation (classe 10-15 W) lorsque le GPU fonctionne à sa vitesse d’horloge maximale. Avant le traitement par Texture Aware sur un**Jetson Nano ou Orin Nano**, Chloros vérifie la fréquence maximale du GPU et la limite à**510 MHz** (510000000) si elle est actuellement supérieure :

* Si la commande CLI peut écrire dans le nœud sysfs de la fréquence du GPU, la limitation est **appliquée automatiquement** et un message de confirmation s’affiche.
* Dans le cas contraire (accès root requis), le script CLI affiche la commande exacte `sudo` permettant d’appliquer manuellement la limite, attend un instant pour vous laisser le temps de la lire, puis continue — le traitement se poursuit mais peut afficher des avertissements de surintensité.

Pour appliquer vous-même la limite avant le traitement :

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Les modèles plus puissants (Orin NX 25 W, AGX Orin 60 W) fonctionnent à la vitesse maximale du GPU ; aucune limite n’est appliquée. Le débayeur standard ne déclenche jamais la limite, quel que soit le modèle.

{% hint style="info" %}
**Le mode « Texture Aware » sur Jetson traite toujours une image à la fois.** Chaque worker aurait besoin de son propre contexte CUDA (~1 Go) ainsi que de sa propre copie du modèle de débayérisation, ce que la mémoire unifiée ne peut pas supporter — c’est pourquoi, sur Jetson, le chemin « Texture Aware » est verrouillé sur un seul worker avec un accès au GPU sérialisé. Attendez-vous à ce que « Texture Aware » soit nettement plus lent que « Standard » sur n’importe quel Jetson.
{% endhint %}

***

## Gestion thermique

Les appareils Jetson disposent d&#x27;une marge thermique limitée, en particulier dans les déploiements en espace clos ou aériens. Chloros surveille la température du SoC et limite automatiquement la taille des lots :

| Température         | Action                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Fonctionnement normal — vitesse de traitement maximale          |
| **70 °C** (Avertissement) | La taille des lots diminue progressivement (100 % → 50 % entre 70 °C et 80 °C) |
| **80 °C** (Critique) | Réduction drastique (50 % → 0 % entre 80 °C et 90 °C) |
| **90 °C** (Arrêt) | Arrêt complet du traitement GPU — refroidissement requis |

{% hint style="warning" %}
**Veillez à assurer une ventilation et un dissipateur thermique adéquats** pour un traitement continu, en particulier dans des enceintes de terrain fermées ou des systèmes aéroportés. La limitation thermique réduira le débit de traitement afin de protéger le matériel.
{% endhint %}

***

## Gestion de la mémoire

Les appareils Jetson utilisent une **mémoire unifiée** : le GPU et le CPU partagent la même mémoire vive physique. La VRAM indiquée (par exemple, environ 15,3 Go sur un Orin NX 16 Go) n’est pas de la mémoire dédiée au GPU ; il s’agit de la même mémoire vive que celle utilisée par le système d’exploitation et tous les autres processus.

### Avertissement et recommandations concernant l’espace d’échange

Avant le traitement sur Jetson, le script CLI compte les images RAW présentes dans votre dossier d’entrée (`.tif`, `.tiff`, `.raw`, `.dng` — les aperçus JPG ne sont pas comptabilisés), estime la mémoire maximale requise pour l&#x27;exécution et **émet un avertissement avant le démarrage** si la RAM et l&#x27;espace d&#x27;échange risquent d&#x27;être insuffisants. L’avertissement porte le titre `LOW MEMORY WARNING - Jetson Detected`, affiche le nombre d’images, la RAM, l’espace d’échange actuel et la consommation maximale estimée, puis indique les commandes exactes `fallocate` / `chmod` / `mkswap` / `swapon` adaptées à votre projet (jamais inférieures à 8 Go). Il marque une pause de quelques secondes afin que le message ne soit pas perdu dans le défilement, puis le traitement se poursuit.**Estimations de mémoire utilisées par l’avertissement :**

| Mode de débayérisation | Base | Par image |
| --- | --- | --- |
| Standard | ~1,5 Go | ~10 Mo |
| Texture Aware | ~2,5 Go (modèle + runtime Python) | ~15 Mo |

L&#x27;avertissement se déclenche lorsque le pic estimé dépasse la RAM + l&#x27;espace d&#x27;échange, moins une marge de sécurité de 1 Go, et il ne prend en compte que l&#x27;espace d&#x27;échange **soutenu par des fichiers** — une configuration utilisant uniquement zram sera tout de même signalée.

Pour ajouter manuellement de l’espace d’échange (exemple : 8 Go) :



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### Gestion des situations OOM (Out of Memory)

Pendant le traitement, Chloros surveille la mémoire du GPU et réduit progressivement ses performances au lieu de planter :

1. Lorsque l’utilisation de la mémoire du GPU dépasse **85 %**, la taille des lots est réduite de manière préventive
2. Si un événement de manque de mémoire survient malgré tout, la taille du lot est **divisée par deux**, puis divisée à nouveau par deux à chaque manque de mémoire consécutif ; chaque lot traité avec succès par la suite réduit cette pénalité d’un cran
3. Sous une pression soutenue, le pipeline bascule de `fused_gpu` vers le chemin `tiled_gpu`, plus économe en mémoire, puis vers le traitement par le CPU en dernier recours

***

## Déploiement sur site

### Considérations relatives à la consommation électrique

| Modèle Jetson     | Consommation électrique typique | Remarques                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | Prise cylindrique CC          |
| Jetson Orin NX   | 10-25 W             | Prise cylindrique CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD ou prise cylindrique |

Prévoyez votre budget énergétique pour un traitement en continu — la consommation électrique maximale se produit pendant le Thread 3 (Traitement), très gourmand en ressources GPU.

### Recommandations en matière de stockage

* **SSD NVMe** fortement recommandé pour les déploiements arm64
* Les cartes SD sont trop lentes pour le traitement — utilisez-les uniquement comme support de démarrage
* Prévoyez 2 à 3 fois la taille brute de vos données d’image pour les données de sortie traitées

### Fonctionnement sans écran via SSH

Chloros et CLI sont idéaux pour les déploiements Jetson sans affichage :

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend toujours actif pour la synchronisation temporelle LATTICE / DAQ-E

Si votre Jetson contrôle des caméras LATTICE ou des capteurs de lumière DAQ-E en mode sans interface graphique, activez le service systemd du backend afin que le grand maître PTP fonctionne en continu (le service est installé mais n’est pas activé par défaut) :

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Consultez la section [Installation de Linux](linux-installation.md#always-on-ptp-for-headless-hosts) pour plus de détails, notamment sur la manière dont le paquet permet de lier les ports PTP 319/320 sans droits root.

### Traitement automatisé avec systemd

Créez un service systemd pour le traitement automatisé :

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` renvoie un code de sortie différent de zéro lorsqu’une exécution ayant demandé des produits n’écrit aucune image ; le statut d’échec de systemd est donc pertinent pour la surveillance.

Associez-le à une minuterie systemd pour un traitement planifié :

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Exemples de workflows

### Traitement de base sur Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Python et SDK sur Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Traitement par lots de plusieurs vols

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Systèmes Jetson recommandés pour une utilisation sur le terrain

Pour les déploiements sur le terrain et en vol, envisagez ces options de cartes porteuses Jetson Orin NX 16 Go :

* **Aérien/drone** : systèmes résistants aux vibrations (norme MIL-STD), légers (moins de 300 g), à refroidissement passif
* **Terrain, usage intensif** : boîtiers étanches IP67/IP69K avec connectivité caméra GigE PoE
* **Configuration minimale/économique** : kits de développement avec boîtiers optionnels

Contactez [le support MAPIR](https://www.mapir.camera/community/contact) pour obtenir des recommandations matérielles spécifiques à votre scénario de déploiement.

***

## Étapes suivantes

* [Installation Linux](linux-installation.md) — Détails généraux sur l’installation de Linux
* [Adaptation dynamique des ressources de calcul](../processing-architecture/dynamic-compute-adaptation.md) — Référence complète sur les stratégies de calcul
* [Pipeline de traitement](../processing-architecture/processing-pipeline.md) — Comprendre le pipeline à 4 threads
* [CLI : Ligne de commande](../CLI.md) — Le guide CLI
* [API : Python SDK](../api-python-sdk.md) — Le guide SDK
* [Référence CLI](../reference/cli-reference.md) et [Référence SDK](../reference/sdk-reference.md) — Liste exhaustive des commandes/API pour la version 1.2.0
