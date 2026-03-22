# Installation de Linux

Chloros est distribué pour Linux sous forme de paquets `.deb` qui installent CLI et le backend. Python et SDK s&#x27;installent séparément via pip.

***

## Linux amd64 (x86_64)

### Configuration requise

| Exigence | Minimum | Recommandé |
| --- | --- | --- |
| **Distribution** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Processeur** | x86_64 (Intel/AMD) | Intel Core i7 ou supérieur |
| **Mémoire (RAM)** | 8 Go | 16 Go ou plus |
| **Carte graphique** | Aucune (traitement par le processeur) | Carte graphique NVIDIA avec 4 Go ou plus de VRAM |
| **Stockage** | 2 Go d&#x27;espace libre | SSD avec 10 Go ou plus d&#x27;espace libre |
| **Python** | Python 3.7+ (pour SDK) | Python 3.10+ |

### Installation

Téléchargez le paquet `.deb` et installez-le :

```bash
sudo dpkg -i chloros-amd64.deb
```

Vérifiez l&#x27;installation :

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Configuration requise

| Exigence | Minimum | Recommandé |
| --- | --- | --- |
| **Plateforme** | NVIDIA Jetson avec JetPack 6 | Jetson Orin NX 16 Go ou AGX Orin |
| **JetPack** | JetPack 6.x | Dernière version de JetPack 6 |
| **Mémoire (RAM)** | 8 Go (partagée GPU/CPU) | 16 Go+ partagée |
| **Stockage** | 2 Go d&#x27;espace libre | SSD NVMe avec 10 Go+ d&#x27;espace libre |
| **Python** | Python 3.7+ (pour SDK) | Python 3.10+ |

### Installation

Téléchargez le package JetPack 6 `.deb` et installez-le :

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Vérifiez l&#x27;installation :

```bash
chloros-cli --version
```

Pour une configuration détaillée du Jetson, y compris la gestion thermique et le déploiement sur le terrain, consultez le [Guide NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Installation de Python et SDK (toutes les versions Linux)

Python et SDK s&#x27;installent séparément via pip et fonctionnent à la fois sur amd64 et arm64 :

```bash
pip install chloros-sdk
```

Pour inclure la prise en charge facultative de la diffusion en continu de la progression :

```bash
pip install chloros-sdk[progress]
```

Vérifiez le SDK :

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Le paquet `.deb` installe le backend Chloros CLI. Le paquet Python SDK est un paquet pip distinct qui communique avec le backend via un HTTP API local.
{% endhint %}

***

## Répertoires de configuration

Chloros sur Linux suit la [spécification du répertoire de base XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) :

| Objectif | Chemin Linux | Équivalent Windows |
| --- | --- | --- |
| **Configuration** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Données / Projets** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Identifiants** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Emplacements des exécutables du backend

Le paquet `.deb` installe le backend à un emplacement standard. Les paquets CLI et SDK détectent automatiquement le chemin d&#x27;accès au backend :

| Méthode d&#x27;installation | Chemin d&#x27;accès au backend |
| --- | --- |
| Paquet `.deb` | `/usr/lib/chloros/chloros-backend` |
| Manuelle / personnalisée | `/opt/mapir/chloros/backend/chloros-backend` |

Vous pouvez remplacer le chemin d&#x27;accès au backend à l&#x27;aide du drapeau `--backend-exe` CLI ou du paramètre de constructeur `backend_exe` SDK.

***

## Première configuration

### 1. Activez votre licence

Une licence Chloros+ est requise pour accéder à CLI et SDK :

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Vérifiez l&#x27;état de votre licence

```bash
chloros-cli status
```

### 3. Traitez votre premier ensemble de données

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Exécutez les diagnostics système

Vérifiez que votre système est correctement configuré :

```bash
chloros-cli selftest
```

Cela lance 7 vérifications de diagnostic, notamment la version, le démarrage du backend, la connectivité API et la disponibilité de CUDA/GPU.

***

## Exemples de scripts Bash

### Traiter plusieurs ensembles de données

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Traiter avec des paramètres personnalisés

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Traitement automatisé avec Cron

Ajoutez à votre crontab (`crontab -e`) pour traiter automatiquement les nouveaux ensembles de données :

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemple Python SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Dépannage

### CLI introuvable après l&#x27;installation

Si `chloros-cli` est introuvable après l&#x27;installation du paquet `.deb` :

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Autorisation refusée

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Échec du démarrage du backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA non détecté

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Bibliothèques partagées manquantes

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Mise à jour de Chloros sur Linux

Utilisez la commande de mise à jour intégrée pour rechercher et installer les mises à jour :

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Étapes suivantes

* [Guide NVIDIA Jetson](nvidia-jetson-guide.md) — Optimisation et déploiement spécifiques à Jetson
* [CLI : Ligne de commande](../CLI.md) — Référence complète des commandes CLI
* [API : Python SDK](../api-python-sdk.md) — Référence complète de SDK
* [Adaptation dynamique du calcul](../processing-architecture/dynamic-compute-adaptation.md) — Comment Chloros s&#x27;adapte à votre matériel
