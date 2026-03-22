# Guide NVIDIA Jetson

Chloros sur NVIDIA Jetson permet le traitement d&#x27;images multispectrales en périphérie — sur le terrain, sur des drones et dans des installations distantes. Chloros détecte automatiquement votre modèle Jetson et optimise sa stratégie de traitement en fonction de votre matériel.

***

## Modèles Jetson pris en charge

| Modèle                | RAM            | Stratégie de traitement                                   | Utilisation recommandée                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 Go partagés | `GPU_PARALLEL` (4 workers)                            | Performances maximales, grands ensembles de données                      |
| **Jetson Orin NX**   | 8-16 Go partagés  | `GPU_PARALLEL` (3 workers, 16 Go) / `GPU_SINGLE` (8 Go) | Recommandation principale pour les déploiements aériens et sur le terrain |
| **Jetson Orin Nano** | 8 Go partagés     | `GPU_SINGLE` (1 worker)                               | Informatique en périphérie d&#x27;entrée de gamme                                 |
| **Jetson Nano**      | 4-8 Go partagés   | `GPU_SINGLE` (1 worker)                               | D&#x27;entrée de gamme, mémoire limitée                          |

{% hint style="info" %}
**Les anciens modèles Jetson** (TX2, TX1, Xavier NX) peuvent ne pas être pris en charge. Les performances varient en fonction de la mémoire GPU disponible et des capacités CUDA.
{% endhint %}

***

## Configuration requise

* **JetPack 6.x** (dernière version recommandée)
* **NVIDIA CUDA** (inclus avec JetPack)
* **Licence Chloros+** (requise pour accéder à CLI/SDK)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Pour plus de détails sur l&#x27;installation générale de Linux, consultez [Installation de Linux](linux-installation.md).

***

## Adaptation dynamique du calcul sur Jetson

Chloros détecte automatiquement votre modèle Jetson et sélectionne la stratégie de traitement optimale. **Aucun réglage manuel n&#x27;est nécessaire.**

### Fonctionnement

Au démarrage, Chloros analyse votre système :

1. **Détecte le modèle Jetson** via `/proc/device-tree/model`
2. **Lit la mémoire GPU/partagée disponible**

3.**Sélectionne une stratégie de traitement** (`GPU_PARALLEL`, `GPU_SINGLE` ou `CPU_PARALLEL`)
4. **Définit automatiquement le nombre de workers, le type de pipeline et l&#x27;allocation de mémoire**

### Comportement par modèle

| Modèle Jetson                | Stratégie       | Tâches | Pipeline                       | Concurrence |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 Go**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (efficace en mémoire) | Sérialisé  |
| **Jetson Orin Nano 8 Go**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Sérialisé  |
| **Jetson Orin NX 8 Go**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Sérialisé  |
| **Jetson Orin NX 16 Go**     | `GPU_PARALLEL` | 3       | `fused_gpu` (chemin GPU complet)    | Concurrent  |
| **Jetson AGX Orin 32-64 Go** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Concurrent  |

{% hint style="success" %}
**Jetson Orin NX 16 Go** est le choix idéal pour le déploiement en périphérie : il bénéficie de la stratégie `GPU_PARALLEL` avec 3 workers simultanés, offrant un véritable traitement GPU parallèle dans un format compact.
{% endhint %}

La principale différence entre les plateformes réside dans la **mémoire**. Un Jetson Nano doté de 8 Go de mémoire partagée doit traiter les images une par une à l&#x27;aide d&#x27;une approche par mosaïques économe en mémoire, tandis qu&#x27;un Orin NX doté de 16 Go peut traiter 3 images simultanément via le GPU à l&#x27;aide du pipeline fusionné à haut débit.

Pour consulter la référence complète sur l&#x27;adaptation du calcul, voir [Adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md).

***

## Gestion thermique

Les appareils Jetson ont une marge thermique limitée, en particulier dans les déploiements en espace clos ou aéroportés. Chloros inclut une surveillance thermique et une régulation automatiques :

| Température         | Action                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Fonctionnement normal — vitesse de traitement maximale          |
| **70 °C** (Avertissement)  | Réduction automatique de la taille des lots                   |
| **80 °C** (Critique) | Limitation agressive — concurrence réduite         |
| **90 °C** (Arrêt) | Arrêt complet du traitement GPU — refroidissement requis |

{% hint style="warning" %}
**Assurez une ventilation et un dissipateur thermique adéquats** pour un traitement continu, en particulier dans les boîtiers de terrain fermés ou les systèmes embarqués. La limitation thermique réduira le débit de traitement afin de protéger le matériel.
{% endhint %}

***

## Gestion de la mémoire

Les appareils Jetson utilisent une **mémoire unifiée** : le GPU et le CPU partagent la même RAM physique. Cela signifie que la VRAM indiquée (par exemple, 15,3 Go sur l&#x27;Orin NX 16 Go) n&#x27;est pas de la mémoire dédiée au GPU ; elle est partagée avec le système d&#x27;exploitation et d&#x27;autres processus.

### Recommandations concernant l&#x27;espace d&#x27;échange

Pour les grands ensembles de données ou le traitement de débayérisation avec prise en compte des textures, Chloros peut recommander la création d&#x27;un espace d&#x27;échange :

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Estimations de mémoire par image :**

* Débayérisation standard : environ 10 Mo par image
* Débayérisation Texture Aware : environ 15 Mo par image

Chloros calcule automatiquement la mémoire requise en fonction de la taille de votre ensemble de données et vous avertit si un espace d&#x27;échange est recommandé.

### Solution de secours en cas de mémoire insuffisante (OOM)

Si une situation de mémoire insuffisante est détectée pendant le traitement :

1. Chloros réduit automatiquement le nombre de workers GPU
2. Passe du pipeline `fused_gpu` au pipeline `tiled_gpu` (plus économe en mémoire)
3. Poursuit le traitement à un débit réduit plutôt que de planter

***

## Déploiement sur le terrain

### Considérations relatives à l&#x27;alimentation

| Modèle Jetson     | Consommation électrique typique | Remarques                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C ou prise cylindrique    |
| Jetson Orin Nano | 7-15 W              | Connecteur cylindrique CC          |
| Jetson Orin NX   | 10-25 W             | Connecteur cylindrique CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD ou connecteur cylindrique |

Planifiez votre budget énergétique pour un traitement continu — la consommation électrique maximale se produit pendant le Thread 3 (Traitement), qui sollicite fortement le GPU.

### Recommandations en matière de stockage

* **SSD NVMe** fortement recommandé pour les déploiements arm64
* Les cartes SD sont trop lentes pour le traitement — utilisez-les uniquement comme support de démarrage
* Prévoyez 2 à 3 fois la taille de vos données d&#x27;image brutes pour les données de sortie traitées

### Fonctionnement sans écran via SSH

Chloros CLI est idéal pour les déploiements Jetson sans interface graphique :

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

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

### Traitement Jetson de base

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK sur Jetson

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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Systèmes Jetson recommandés pour une utilisation sur le terrain

Pour les déploiements sur le terrain et en vol, envisagez ces options de cartes porteuses Jetson Orin NX 16 Go :

* **En vol/drone** : systèmes résistants aux vibrations (norme MIL-STD), légers (moins de 300 g), à refroidissement passif
* **Environnement de terrain difficile** : boîtiers étanches IP67/IP69K avec connectivité caméra GigE PoE
* **Configuration minimale/économique** : kits de développement avec boîtiers supplémentaires

Contactez [le support MAPIR](https://www.mapir.camera/community/contact) pour obtenir des recommandations matérielles spécifiques à votre scénario de déploiement.

***

## Étapes suivantes

* [Installation Linux](linux-installation.md) — Détails généraux sur l&#x27;installation de Linux
* [Adaptation dynamique des ressources de calcul](../processing-architecture/dynamic-compute-adaptation.md) — Référence complète sur les stratégies de calcul
* [Pipeline de traitement](../processing-architecture/processing-pipeline.md) — Comprendre le pipeline à 4 threads
* [CLI : Ligne de commande](../CLI.md) — Référence complète sur CLI
* [API : Python SDK](../api-python-sdk.md) — Référence complète de SDK
