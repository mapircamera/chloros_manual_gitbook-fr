# API : Python SDK

Le **Chloros Python SDK** fournit un accès programmatique au moteur de traitement d'images Chloros, permettant l'automatisation, les flux de travail personnalisés et l'intégration transparente avec vos applications et pipelines de recherche Python.

### Caractéristiques principales

* 🐍 **Natif Python** - Propre, Pythonique API pour le traitement d'images
* 🔧 **Accès complet API** - Contrôle total sur le traitement Chloros
* 🚀 **Automation** - Création de flux de traitement par lots personnalisés
* 🔗 **Intégration** - Intégrer Chloros dans les applications existantes Python
* 📊 **Research-Ready** - Parfait pour les pipelines d'analyse scientifique
* ⚡ **Traitement parallèle** - S'adapte aux cœurs de votre processeur (Chloros+)

### Exigences

| Exigences | Détails |
| -------------------- | ------------------------------------------------------------------- |
**Chloros Desktop** | Doit être installé localement **Chloros Desktop** | Doit être installé localement ****
| **Licence** | Chloros+ ([paid plan required](https://cloud.mapir.camera/pricing)) | **Système d'exploitation** | Windows+ ([paid plan required](https://cloud.mapir.camera/pricing))
| **Système d'exploitation** | Windows 10/11 (64-bit) | **Chloros+ ([paid plan required](https://cloud.mapir.camera/pricing))
| Système d'exploitation** Windows 10/11 (64-bit) | **Python** | Python 3.7 ou plus
| Mémoire**** - 8 Go de RAM minimum (16 Go recommandés)
| **Internet** | Nécessaire pour l'activation de la licence

{% hint style="warning" %}
**License Requirement**: The Python SDK requires a paid Chloros+ subscription for API access. Standard (free) plans do not have API/SDK access. Visit [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) to upgrade.
{% endhint %}

## Démarrage rapide

### Installation

Installer via pip :

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**First-Time Setup**: Before using the SDK, activate your Chloros+ license by opening Chloros, Chloros (Browser) or Chloros CLI and logging in with your credentials. This only needs to be done once.
{% endhint %}

### Utilisation de base

Traiter un dossier en quelques lignes :

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

## Guide d'installation

### Conditions préalables

Avant d'installer le SDK, assurez-vous d'avoir :

1. **Chloros Desktop** installé ([download](download.md))
2. **Python 3.7+** installé ([python.org](https://www.python.org))
3. **Licence Chloros+ active** ([upgrade](https://cloud.mapir.camera/pricing))

### Installer via pip

**Installation standard:**

```bash
pip install chloros-sdk
```

**Avec prise en charge du suivi de la progression:**

```bash
pip install chloros-sdk[progress]
```

**Installation de développement:**

```bash
pip install chloros-sdk[dev]
```

### Vérifier l'installation

Vérifiez que le SDK est correctement installé :

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Première installation

### Activation de la licence

Le SDK utilise la même licence que le Chloros, le Chloros (Navigateur) et le Chloros CLI. Activer une fois via l'interface graphique ou CLI :

1. Ouvrez **Chloros ou Chloros (navigateur)** et connectez-vous à l'onglet Utilisateur <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">. Ou, ouvrez le **CLI**.
2. Saisissez vos Chloros+ informations d'identification et connectez-vous
3. La licence est mise en cache localement (persiste lors des redémarrages)

{% hint style="success" %}
**One-Time Setup**: After logging in via the GUI or CLI, the SDK automatically uses the cached license. No additional authentication needed!
{% endhint %}

### Test de connexion

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

## API Référence

### Classe ChlorosLocal

Classe principale pour le traitement des images locales Chloros.

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

**Paramètres:**

| Paramètres : - Paramètre : - Type : - Valeur par défaut : - Description : - Paramètre : - Type : - Valeur par défaut : - Description
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url` | str | `"http://localhost:5000"` | URL du backend local Chloros - Chloros - `auto_start_backend` | bool
| `auto_start_backend` | bool | `True` | Démarrer automatiquement le backend si nécessaire |
| `backend_exe` | str | `None` (auto-détection) | Chemin d'accès à l'exécutable du backend | `timeout` | bool
| PLH:000058] | int | `30` | Délai d'attente de la demande en secondes
| PLH:000060] | int | `60` | Délai d'attente pour le démarrage du backend (secondes) |

**Exemples:**

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

**Paramètres:**

| Paramètre | Type | Obligatoire | Description
| -------------- | ---- | -------- | -------------------------------------------------------- |
| PLH:000063] | str | Oui | Nom du projet
| Non | Modèle de caméra (par exemple, "Survey3N\_RGN", "Survey3W\_OCN") |

**Returns:** `dict` - Réponse de création de projet

**Exemple:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importer des images à partir d'un dossier.

**Paramètres:**

| Paramètre | Type | Obligatoire | Description
| ------------- | -------- | -------- | ---------------------------------- |
| PLH:000067] | str/Path | Yes | Chemin d'accès au dossier contenant les images |
| PLH:000068] | bool | Non | Recherche dans les sous-dossiers (par défaut : False) |

**Returns:** `dict` - Résultats de l'importation avec le nombre de fichiers

**Exemple:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configurer les paramètres de traitement.

**Paramètres:**

| Paramètres : - Paramètres : - Type : - Défaut : - Description : - Paramètres : - Type : - Défaut : - Description
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer` | str | "Haute qualité (plus rapide)" | Méthode Debayer
| `vignette_correction` | bool | `True` | Activer la correction de vignette |
| `reflectance_calibration` | bool | `True` | Activer la calibration de la réflectance |
| `indices` | liste | `None` | Indices de végétation à calculer | `export_format` | liste
| `export_format` | str | "TIFF (16-bit)"         | Format de sortie
| `ppk` | bool | `False` | Activer les corrections PPK | `custom_settings` | bool
| PLH:000081] | dict | `None` | Paramètres personnalisés avancés |

**Formats d'exportation:**

* `"TIFF (16-bit)"` - Recommandé pour les SIG/photogrammétrie
* `"TIFF (32-bit, Percent)"` - Analyse scientifique
* `"PNG (8-bit)"` - Inspection visuelle
* `"JPG (8-bit)"` - Sortie compressée

**Indices disponibles

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2, et plus encore.

**Exemple:**

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

**Paramètres:**

| Paramètre | Type | Valeur par défaut | Description
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode` | str | `"parallel"` | Mode de traitement : "parallèle" ou "série" | `"parallel"` | Mode de traitement : "parallèle" ou "série" |
| `wait` | bool | `True` | Attendre la fin du traitement |
| `progress_callback` | callable | `None` | Fonction de rappel de progression (progress, msg) |
| PLH:000094] | float | `2.0` | Intervalle d'interrogation pour la progression (secondes) |

**Returns:** `dict` - Traitement des résultats

{% hint style="warning" %}
**Parallel Mode**: Requires Chloros+ license. Automatically scales to your CPU cores (up to 16 workers).
{% endhint %}

**Exemple:**

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

**Returns:** `dict` - Configuration actuelle du projet

**Exemple:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Obtenir des informations sur le statut du backend.

**Returns:** `dict` - Statut du backend

**Exemple:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Arrêter le backend (s'il a été démarré par SDK).

**Exemple:**

```python
chloros.shutdown_backend()
```

***

### Fonctions pratiques

#### `process_folder(folder_path, **options)`

Fonction de commodité en une ligne pour traiter un dossier.

**Paramètres:**

| Paramètre | Type | Valeur par défaut | Description
| ------------------------- | -------- | --------------- | ------------------------------ |
pLH:000103] | str/Path | Requis | Chemin d'accès au dossier contenant les images | PLH:000103] | str/Path | Requis | Chemin d'accès au dossier contenant les images
| PLH:000104] | str | Auto-généré | Nom de projet
| PLH:000105] | str | `None` | modèle de caméra |
| `indices` | liste | `["NDVI"]` | Indices à calculer | `vignette_correction` | liste | `vignette_correction` | liste
| PLH:000109] | bool | `True` | Activer la correction de vignette | `reflectance_calibration` | bool
| `reflectance_calibration` | bool | `True` | Activer la calibration de la réflectance |
| `export_format` | str | "TIFF (16-bit)" | Format de sortie |
| `mode` | str | `"parallel"` | Mode de traitement | `progress_callback` | Mode de traitement
| `progress_callback` | callable | `None` | Callback de progression |

**Returns:** `dict` - Résultats du traitement

**Exemple:**

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

Le SDK prend en charge les gestionnaires de contexte pour un nettoyage automatique :

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

### Exemple 1 : Traitement de base

Traiter un dossier avec les paramètres par défaut :

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Exemple 2 : Flux de travail personnalisé

Contrôle total sur le pipeline de traitement :

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

### Exemple 3 : Traitement par lots de plusieurs dossiers

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

### Exemple 4 : Intégration du pipeline de recherche

Intégrer Chloros à l'analyse des données :

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

### Exemple 5 : Suivi personnalisé des progrès

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

### Exemple 7 : Outil de ligne de commande

Créez un outil CLI personnalisé avec le SDK :

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

**Usage:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Gestion des exceptions

Le SDK fournit des classes d'exceptions spécifiques pour différents types d'erreurs :

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

### Exemples d'exceptions

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

Utiliser un emplacement ou une configuration de backend personnalisé :

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Traitement non bloquant

Commencer le traitement et poursuivre les autres tâches :

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

Pour les grands ensembles de données, traiter par lots :

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

**Issue:** SDK ne parvient pas à démarrer le backend

**Solutions:**

1. Vérifiez que Chloros Desktop est installé :

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Vérifier Windows Le pare-feu n'est pas bloqué
3. Essayez le chemin d'accès manuel au backend :

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licence non détectée

**Issue:** SDK avertit de l'absence de licence

**Solutions:**

1. Ouvrez Chloros, Chloros (Navigateur) ou Chloros CLI et connectez-vous.
2. Vérifiez que la licence est mise en cache :

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Contacter le support : info@mapir.camera

***

### Erreurs d'importation

**Issue:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Solutions:**

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

### Délai de traitement

**Issue:** Le traitement est interrompu

**Solutions:**

1. Augmenter le délai d'attente :

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Traiter des lots plus petits
3. Vérifier l'espace disque disponible
4. Contrôler les ressources du système

***

### Port déjà utilisé

**Issue:** Port backend 5000 occupé

**Solutions:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ou trouver et fermer le processus conflictuel :

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Conseils de performance

### Optimiser la vitesse de traitement

1. **Utiliser le mode parallèle** (nécessite Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Réduire la résolution de sortie** (si acceptable)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Désactiver les indices inutiles**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Traiter sur le SSD** (pas sur le HDD)

***

### Optimisation de la mémoire

Pour les grands ensembles de données :

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Traitement de fond

Libère Python pour d'autres tâches :

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Exemples d'intégration

### Intégration de Django

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

### Q : Le SDK nécessite-t-il une connexion internet ?

**A:** Uniquement pour l'activation initiale de la licence. Après s'être connecté via Chloros, Chloros (navigateur) ou Chloros CLI, la licence est mise en cache localement et fonctionne hors ligne pendant 30 jours.

***

### Q : Puis-je utiliser SDK sur un serveur sans interface graphique ?

**A:** Oui ! Exigences :

* Windows Serveur 2016 ou ultérieur
* Chloros installé (une seule fois)
* Licence activée sur n'importe quelle machine (licence en cache copiée sur le serveur)

***

### Q : Quelle est la différence entre Desktop, CLI et SDK ?

| Les différences entre les deux systèmes sont les suivantes : CLI, SDK, Python, SDK et SDK
| --------------- | ----------- | ---------------- | ----------- |
| | Interface** | Pointer-cliquer | Commande | Python API |
| Le meilleur moyen d'y parvenir est de travailler visuellement, d'écrire des scripts et de s'intégrer dans le système
| **Automatisation** | Limitée | Bonne | Excellente
| **Flexibilité** | De base | Bonne | Maximale
| | Chloros+ | Chloros+ | Chloros+ | | Chloros - Chloros+ - Chloros+ |

***

### Q : Puis-je distribuer des applications créées avec la SDK ?

**A:** Le code SDK peut être intégré dans vos applications, mais.. :

* Les utilisateurs finaux doivent avoir installé Chloros
* Les utilisateurs finaux ont besoin de licences actives Chloros+
* La distribution commerciale nécessite une licence OEM

Contactez info@mapir.camera pour les demandes OEM.

***

### Q : Comment mettre à jour le SDK ?

```bash
pip install --upgrade chloros-sdk
```

***

### Q : Où sont sauvegardées les images traitées ?

Par défaut, dans le chemin du projet :

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Q : Puis-je traiter des images à partir de scripts Python exécutés selon un calendrier ?

**A:** Oui ! Utilisez le planificateur de tâches Windows avec les scripts Python :

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Planifier via le planificateur de tâches une exécution quotidienne.

***

### Q : Le SDK prend-il en charge les fonctions async/await ?

**A:** La version actuelle est synchrone. Pour un comportement asynchrone, utilisez `wait=False` ou exécutez dans un thread séparé :

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Obtenir de l'aide

### Documentation

* **API Référence** : Cette page

### Canaux de support

* **Courriel** : info@mapir.camera
* **Site web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Tarification** : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Exemple de code

Tous les exemples énumérés ici ont été testés et sont prêts pour la production. Copiez-les et adaptez-les à votre cas d'utilisation.

***

## Licence

**Proprietary Software** - Copyright (c) 2025 MAPIR Inc.

SDK nécessite un abonnement Chloros+ actif. L'utilisation, la distribution ou la modification non autorisées sont interdites.
