# API : Python SDK

Le **Chloros Python SDK** fournit un accès programmatique au moteur de traitement d&#x27;images Chloros, permettant l&#x27;automatisation, la personnalisation des flux de travail et une intégration transparente avec vos applications Python et vos pipelines de recherche.

### Principales fonctionnalités

* 🐍 **Python natif** - API propre et pythonesque pour le traitement d&#x27;images
* 🔧 **Accès complet à API** - Contrôle total du traitement Chloros
* 🚀 **Automatisation** - Créez des flux de travail de traitement par lots personnalisés
* 🔗 **Intégration** - Intégrez Chloros dans les applications Python existantes
* 📊 **Prêt pour la recherche** - Parfait pour les pipelines d&#x27;analyse scientifique
* ⚡ **Traitement parallèle** - S&#x27;adapte à vos cœurs de processeur (Chloros+)

### Exigences

| Exigence          | Détails                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Doit être installé localement                                           |
| **Licence**          | Chloros+ ([forfait payant requis](https://cloud.mapir.camera/pricing)) |
| **Système d&#x27;exploitation** | Windows 10/11 (64 bits)                                              |
| **Python**           | Python 3.7 ou supérieur                                                |
| **Mémoire**           | 8 Go de RAM minimum (16 Go recommandés)                                  |
| **Internet**         | Requis pour l&#x27;activation de la licence                                     |

{% hint style=&quot;warning&quot; %}
**Exigences en matière de licence** : Python SDK nécessite un abonnement payant à Chloros+ pour accéder à API. Les forfaits standard (gratuits) ne donnent pas accès à API/SDK. Rendez-vous sur [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) pour passer à un forfait supérieur.
{% endhint %}

## Démarrage rapide

### Installation

Installez via pip :

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Première configuration** : avant d&#x27;utiliser SDK, activez votre licence Chloros+ en ouvrant Chloros, Chloros (navigateur) ou Chloros CLI et en vous connectant avec vos identifiants. Cette opération ne doit être effectuée qu&#x27;une seule fois.
{% endhint %}

### Utilisation de base

Traitez un dossier en quelques lignes :

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Contrôle total

Pour les flux de travail avancés :

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Guide d&#x27;installation

### Conditions préalables

Avant d&#x27;installer SDK, assurez-vous que vous disposez des éléments suivants :

1. **Chloros Desktop** installé ([télécharger](download.md))
2. **Python 3.7+** installé ([python.org](https://www.python.org))
3. **Licence Chloros+ active** ([mise à niveau](https://cloud.mapir.camera/pricing))

### Installation via pip

**Installation standard :**

```bash
pip install chloros-sdk
```

**Avec prise en charge du suivi de progression :**

```bash
pip install chloros-sdk[progress]
```

**Installation de développement :**

```bash
pip install chloros-sdk[dev]
```

### Vérification de l&#x27;installation

Vérifiez que SDK est correctement installé :

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Première configuration

### Activation de la licence

Le SDK utilise la même licence que les Chloros, Chloros (navigateur) et Chloros CLI. Activez-la une fois via l&#x27;interface graphique ou CLI :

1. Ouvrez **Chloros ou Chloros (navigateur)** et connectez-vous à l&#x27;onglet Utilisateur <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Ou ouvrez **CLI**.
2. Entrez vos identifiants Chloros+ et connectez-vous
3. La licence est mise en cache localement (elle persiste après les redémarrages)

{% hint style=&quot;success&quot; %}
**Configuration unique** : après vous être connecté via l&#x27;interface graphique ou CLI, SDK utilise automatiquement la licence mise en cache. Aucune authentification supplémentaire n&#x27;est nécessaire !
{% endhint %}

### Tester la connexion

Vérifiez que SDK peut se connecter à Chloros :

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Référence API

### Classe ChlorosLocal

Classe principale pour le traitement d&#x27;images Chloros local.

#### Constructeur

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Paramètres :**

| Paramètre                 | Type | Par défaut                   | Description                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL du backend local Chloros          |
| `auto_start_backend`      | bool | `True`                    | Démarrer automatiquement le backend si nécessaire |
| `backend_exe`             | str  | `None` (détection automatique)      | Chemin d&#x27;accès à l&#x27;exécutable du backend            |
| `timeout`                 | int  | `30`                      | Délai d&#x27;attente de la requête en secondes            |
| `backend_startup_timeout` | int  | `60`                      | Délai d&#x27;attente pour le démarrage du backend (secondes) |

**Exemples :**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Méthodes

#### `create_project(project_name, camera=None)`

Créer un nouveau projet Chloros.

**Paramètres :**

| Paramètre      | Type | Obligatoire | Description                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Oui      | Nom du projet                                     |
| `camera`       | str  | Non       | Modèle de caméra (par exemple, « Survey3N\_RGN », « Survey3W\_OCN ») |

**Retourne :** `dict` - Réponse de création de projet

**Exemple :**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importe des images à partir d&#x27;un dossier.

**Paramètres :**

| Paramètre     | Type     | Obligatoire | Description                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Oui      | Chemin d&#x27;accès au dossier contenant les images         |
| `recursive`   | bool     | Non       | Rechercher dans les sous-dossiers (par défaut : False) |

**Renvoie :** `dict` - Résultats de l&#x27;importation avec nombre de fichiers

**Exemple :**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configure les paramètres de traitement.

**Paramètres :**

| Paramètre                 | Type | Par défaut                 | Description                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | « Haute qualité (plus rapide) » | Méthode de débayérisation                  |
| `vignette_correction`     | bool | `True`                  | Activer la correction du vignettage      |
| `reflectance_calibration` | bool | `True`                  | Activer l&#x27;étalonnage de la réflectance  |
| `indices`                 | liste | `None`                  | Indices de végétation à calculer |
| `export_format`           | str  | « TIFF (16 bits) »         | Format de sortie                   |
| `ppk`                     | bool | `False`                 | Activer les corrections PPK          |
| `custom_settings`         | dict | `None`                  | Paramètres personnalisés avancés        |

**Formats d&#x27;exportation :**

* `"TIFF (16-bit)"` - Recommandé pour le SIG/la photogrammétrie
* `"TIFF (32-bit, Percent)"` - Analyse scientifique
* `"PNG (8-bit)"` - Inspection visuelle
* `"JPG (8-bit)"` - Sortie compressée

**Indices disponibles :**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2, et plus encore.

**Exemple :**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Traiter les images du projet.

**Paramètres :**

| Paramètre           | Type     | Par défaut      | Description                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Mode de traitement : « parallel » ou « serial »   |
| `wait`              | bool     | `True`       | Attendre la fin                       |
| `progress_callback` | callable | `None`       | Fonction de rappel de progression (progression, msg) |
| `poll_interval`     | float    | `2.0`        | Intervalle d&#x27;interrogation pour la progression (secondes)   |

**Renvoie :** `dict` - Résultats du traitement

{% hint style=&quot;warning&quot; %}
**Mode parallèle** : nécessite une licence Chloros+. S&#x27;adapte automatiquement à vos cœurs de processeur (jusqu&#x27;à 16 processeurs).
{% endhint %}

**Exemple :**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Obtenir la configuration actuelle du projet.

**Renvoie :** `dict` - Configuration actuelle du projet

**Exemple :**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Récupère les informations d&#x27;état du backend.

**Renvoie :** `dict` - État du backend

**Exemple :**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Arrête le backend (s&#x27;il a été démarré par SDK).

**Exemple :**

```python
chloros.shutdown_backend()
```

***

### Fonctions pratiques

#### `process_folder(folder_path, **options)`

Fonction pratique d&#x27;une ligne pour traiter un dossier.

**Paramètres :**

| Paramètre                 | Type     | Par défaut         | Description                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Chemin | Obligatoire        | Chemin vers le dossier contenant les images     |
| `project_name`            | str      | Généré automatiquement  | Nom du projet                   |
| `camera`                  | str      | `None`          | Modèle de caméra                |
| `indices`                 | liste     | `["NDVI"]`      | Indices à calculer           |
| `vignette_correction`     | bool     | `True`          | Activer la correction du vignettage     |
| `reflectance_calibration` | bool     | `True`          | Activer l&#x27;étalonnage de la réflectance |
| `export_format`           | str      | « TIFF (16 bits) » | Format de sortie                  |
| `mode`                    | str      | `"parallel"`    | Mode de traitement                |
| `progress_callback`       | callable | `None`          | Rappel de progression              |

**Renvoie :** `dict` - Résultats du traitement

**Exemple :**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Prise en charge du gestionnaire de contexte

Le SDK prend en charge les gestionnaires de contexte pour le nettoyage automatique :

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Exemples complets

### Exemple 1 : traitement de base

Traiter un dossier avec les paramètres par défaut :

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Exemple 2 : flux de travail personnalisé

Contrôle total du pipeline de traitement :

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Exemple 3 : traitement par lots de plusieurs dossiers

Traiter plusieurs ensembles de données de vol :

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Exemple 4 : intégration du pipeline de recherche

Intégrer Chloros à l&#x27;analyse des données :

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Exemple 5 : Suivi personnalisé de la progression

Suivi avancé de la progression avec journalisation :

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Exemple 6 : Gestion des erreurs

Gestion robuste des erreurs pour une utilisation en production :

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Exemple 7 : Outil en ligne de commande

Créez un outil CLI personnalisé avec SDK :

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Utilisation :**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Gestion des exceptions

SDK fournit des classes d&#x27;exception spécifiques pour différents types d&#x27;erreurs :

### Hiérarchie des exceptions

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Exemples d&#x27;exceptions

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Sujets avancés

### Configuration personnalisée du backend

Utilisez un emplacement ou une configuration de backend personnalisé :

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Traitement non bloquant

Lancez le traitement et poursuivez d&#x27;autres tâches :

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Gestion de la mémoire

Pour les ensembles de données volumineux, effectuez le traitement par lots :

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Dépannage

### Le backend ne démarre pas

**Problème :** SDK ne parvient pas à démarrer le backend.

**Solutions :**

1. Vérifiez que Chloros Desktop est installé :

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Vérifiez que le pare-feu Windows ne bloque pas le programme.
3. Essayez le chemin d&#x27;accès manuel au backend :

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licence non détectée

**Problème :** SDK affiche un message d&#x27;avertissement indiquant que la licence est manquante.

**Solutions :**

1. Ouvrez Chloros, Chloros (navigateur) ou Chloros CLI et connectez-vous.
2. Vérifiez que la licence est mise en cache :

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Contactez l&#x27;assistance : info@mapir.camera

***

### Erreurs d&#x27;importation

**Problème :** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Solutions :**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Délai d&#x27;attente de traitement

**Problème :** Le traitement expire

**Solutions :**

1. Augmentez le délai d&#x27;expiration :

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Traitez des lots plus petits
3. Vérifiez l&#x27;espace disque disponible
4. Surveillez les ressources système

***

### Port déjà utilisé

**Problème :** Le port backend 5000 est occupé

**Solutions :**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ou recherchez et fermez le processus en conflit :

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Conseils de performance

### Optimiser la vitesse de traitement

1. **Utilisez le mode parallèle** (nécessite Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Réduisez la résolution de sortie** (si cela est acceptable)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Désactivez les index inutiles**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Traitez sur SSD** (et non sur HDD)

***

### Optimisation de la mémoire

Pour les grands ensembles de données :

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Traitement en arrière-plan

Libérez Python pour d&#x27;autres tâches :

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Exemples d&#x27;intégration

### Intégration Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## FAQ

### Q : Le SDK nécessite-t-il une connexion Internet ?

**R :** Uniquement pour l&#x27;activation initiale de la licence. Après vous être connecté via Chloros, Chloros (navigateur) ou Chloros CLI, la licence est mise en cache localement et fonctionne hors ligne pendant 30 jours.

***

### Q : Puis-je utiliser SDK sur un serveur sans interface graphique ?

**R :** Oui ! Configuration requise :

* Windows Server 2016 ou version ultérieure
* Chloros installé (une seule fois)
* Licence activée sur n&#x27;importe quelle machine (licence mise en cache copiée sur le serveur)

***

### Q : Quelle est la différence entre Desktop, CLI et SDK ?

| Fonctionnalité         | Interface graphique Desktop | Ligne de commande CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interface**   | Point-clic | Commande          | Python API  |
| **Idéal pour**    | Travail visuel | Scripting        | Intégration |
| **Automatisation**  | Limitée     | Bonne             | Excellente   |
| **Flexibilité** | Basique       | Bonne             | Maximale     |
| **Licence**     | Chloros+    | Chloros+         | Chloros+    |

***

### Q : Puis-je distribuer des applications créées avec SDK ?

**R :** Le code SDK peut être intégré à vos applications, mais :

* Les utilisateurs finaux doivent avoir installé Chloros
* Les utilisateurs finaux doivent disposer de licences Chloros+ actives
* La distribution commerciale nécessite une licence OEM.

Contactez info@mapir.camera pour toute question relative aux licences OEM.

***

### Q : Comment mettre à jour SDK ?

```bash
pip install --upgrade chloros-sdk
```

***

### Q : Où sont enregistrées les images traitées ?

Par défaut, dans le chemin d&#x27;accès au projet :

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Q : Puis-je traiter des images à partir de scripts Python s&#x27;exécutant selon un calendrier défini ?

**R :** Oui ! Utilisez le planificateur de tâches Windows avec les scripts Python :

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Planifiez l&#x27;exécution quotidienne via le planificateur de tâches.

***

### Q : SDK prend-il en charge async/await ?

**R :** La version actuelle est synchrone. Pour un comportement asynchrone, utilisez `wait=False` ou exécutez dans un thread séparé :

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Obtenir de l&#x27;aide

### Documentation

* **Référence API** : cette page

### Canaux d&#x27;assistance

* **E-mail** : info@mapir.camera
* **Site Web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Tarifs** : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Exemples de code

Tous les exemples répertoriés ici ont été testés et sont prêts à être utilisés en production. Copiez-les et adaptez-les à votre cas d&#x27;utilisation.

***

## Licence

**Logiciel propriétaire** - Copyright (c) 2025 MAPIR Inc.

SDK nécessite un abonnement Chloros+ actif. Toute utilisation, distribution ou modification non autorisée est interdite.
