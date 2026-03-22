# CLI : Ligne de commande

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** offre un accès puissant en ligne de commande au moteur de traitement d&#x27;images Chloros, permettant l&#x27;automatisation, la création de scripts et le fonctionnement sans interface graphique pour vos workflows d&#x27;imagerie.

### Principales fonctionnalités

* 🚀 **Automatisation** - Traitement par lots via des scripts de plusieurs ensembles de données
* 🔗 **Intégration** - Intégration dans les workflows et pipelines existants
* 💻 **Fonctionnement sans interface graphique** - Exécution sans interface graphique
* 🌍 **Multilingue** - Prise en charge de 38 langues
* ⚡ **Traitement parallèle** - [Adaptation dynamique des ressources de calcul](processing-architecture/dynamic-compute-adaptation.md) optimise automatiquement en fonction de votre matériel

### Configuration requise

| Exigence          | Détails                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Système d&#x27;exploitation** | Windows 10/11 (64 bits), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licence**          | Chloros+ ([formule payante requise](https://cloud.mapir.camera/pricing)) |
| **Mémoire**           | 8 Go de RAM minimum (16 Go recommandés)                                  |
| **Internet**         | Requis pour l&#x27;activation de la licence                                     |
| **Espace disque**       | Varie en fonction de la taille du projet                                              |

{% hint style="warning" %}
**Conditions de licence** : CLI nécessite un abonnement payant Chloros+. Les formules Standard (gratuites) ne donnent pas accès à CLI. Rendez-vous sur [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) pour passer à un abonnement supérieur.
{% endhint %}

## Démarrage rapide

### Installation

#### Windows

Le CLI est automatiquement inclus dans le programme d&#x27;installation Chloros :

1. Téléchargez et exécutez **Chloros Installer.exe**

2. Suivez les étapes de l&#x27;assistant d&#x27;installation
3. CLI est installé dans : `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Le programme d&#x27;installation ajoute automatiquement `chloros-cli` au PATH de votre système. Redémarrez votre terminal après l&#x27;installation.
{% endhint %}

#### Linux

Installez le paquet `.deb` correspondant à votre architecture :

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Pour une configuration détaillée de Linux, consultez [Installation de Linux](linux/linux-installation.md).

### Première configuration

Avant d&#x27;utiliser CLI, activez votre licence Chloros+ :

**Windows :**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux :**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Utilisation de base

Traiter un dossier avec les paramètres par défaut :

**Windows :**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux :**

```bash
chloros-cli process ~/images/dataset001
```

***

## Référence des commandes

### Syntaxe générale

```
chloros-cli [global-options] <command> [command-options]
```

***

## Commandes

### `process` - Traiter les images

Traite les images d&#x27;un dossier avec étalonnage.

**Syntaxe :**

```bash
chloros-cli process <input-folder> [options]
```

**Exemples :**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Options de la commande de traitement

| Option                | Type    | Par défaut        | Description                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Chemin    | _Obligatoire_     | Dossier contenant les images multispectrales RAW/JPG                                         |
| `-o, --output`        | Chemin    | Identique à l&#x27;entrée  | Dossier de sortie pour les images traitées                                                     |
| `-n, --project-name`  | Chaîne  | Généré automatiquement | Nom de projet personnalisé                                                                    |
| `--vignette`          | Indicateur    | Activé        | Activer la correction du vignettage                                                             |
| `--no-vignette`       | Indicateur    | -              | Désactiver la correction du vignettage                                                            |
| `--reflectance`       | Indicateur    | Activé        | Activer l&#x27;étalonnage de la réflectance                                                         |
| `--no-reflectance`    | Indicateur    | -              | Désactiver l&#x27;étalonnage de la réflectance                                                        |
| `--ppk`               | Indicateur    | Désactivé       | Appliquer les corrections PPK à partir des données du capteur de lumière .daq                                      |
| `--format`            | Choix  | TIFF (16 bits)  | Format de sortie : `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Entier | Auto           | Taille minimale de la cible en pixels pour la détection du panneau d&#x27;étalonnage                          |
| `--target-clustering` | Entier | Auto           | Seuil de regroupement des cibles (0-100)                                                    |
| `--debayer`           | Choix  | `standard`     | Méthode de débayérisation : `standard` ou `texture-aware` (Chloros+ uniquement)                          |
| `--target`, `--targets` | Indicateur  | Désactivé       | Rechercher uniquement les cibles d&#x27;étalonnage dans un sous-dossier « target » ou « targets » (accélère le traitement) |
| `--indices`           | Liste    | Aucun           | Indices de végétation à calculer (par ex., `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Chaîne  | Aucun           | Verrouiller l&#x27;exposition pour le modèle de caméra (broche 1)                                                 |
| `--exposure-pin-2`    | Chaîne  | Aucun           | Verrouiller l&#x27;exposition pour le modèle de caméra (broche 2)                                                 |
| `--recal-interval`    | Entier | Auto           | Intervalle de recalibrage en secondes                                                      |
| `--timezone-offset`   | Entier | 0              | Décalage horaire en heures                                                               |

***

### `login` - Authentification du compte

Connectez-vous avec vos identifiants Chloros+ pour activer le traitement CLI.

**Syntaxe :**

```bash
chloros-cli login <email> <password>
```

**Exemple :**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caractères spéciaux** : Utilisez des guillemets simples pour encadrer les mots de passe contenant des caractères tels que `$`, `!` ou des espaces.
{% endhint %}

**Résultat :**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Effacer les identifiants

Effacez les identifiants enregistrés et déconnectez-vous de votre compte.

**Syntaxe :**

```bash
chloros-cli logout
```

**Exemple :**

```bash
chloros-cli logout
```

**Résultat :**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Utilisateurs de SDK** : Python SDK fournit également une méthode programmatique `logout()` permettant d&#x27;effacer les identifiants dans les scripts Python. Consultez la [documentation Python SDK](api-python-sdk.md#logout) pour plus de détails.
{% endhint %}

***

### `status` - Vérifier l&#x27;état de la licence

Affiche l&#x27;état actuel de la licence et de l&#x27;authentification.

**Syntaxe :**

```bash
chloros-cli status
```

**Exemple :**

```bash
chloros-cli status
```

**Résultat :**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Vérifier la progression de l&#x27;exportation

Surveiller la progression de l&#x27;exportation du thread 4 pendant ou après le traitement.

**Syntaxe :**

```bash
chloros-cli export-status
```

**Exemple :**

```bash
chloros-cli export-status
```

**Cas d&#x27;utilisation :** Appelez cette commande pendant l&#x27;exécution du traitement pour vérifier la progression de l&#x27;exportation.***

### `language` - Gérer la langue de l&#x27;interface

Affichez ou modifiez la langue de l&#x27;interface CLI.

**Syntaxe :**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Exemples :**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Langues prises en charge (38 au total)

| Code    | Langue              | Nom d&#x27;origine      |
| ------- | --------------------- | ---------------- |
| `en`    | Anglais               | English          |
| `es`    | Espagnol               | Español          |
| `pt`    | Portugais            | Português        |
| `fr`    | Français                | Français         |
| `de`    | Allemand                | Deutsch          |
| `it`    | Italien               | Italiano         |
| `ja`    | Japonais              | 日本語              |
| `ko`    | Coréen                | 한국어              |
| `zh`    | Chinois (simplifié)  | 简体中文             |
| `zh-TW` | Chinois (traditionnel) | 繁體中文             |
| `ru`    | Russe               | Русский          |
| `nl`    | Néerlandais                | Nederlands       |
| `ar`    | Arabe                | العربية          |
| `pl`    | Polonais                | Polski           |
| `tr`    | Turc               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonésien            | Bahasa Indonesia |
| `vi`    | Vietnamien            | Tiếng Việt       |
| `th`    | Thaï                  | ไทย              |
| `sv`    | Suédois               | Svenska          |
| `da`    | Danois                | Dansk            |
| `no`    | Norvégien             | Norsk            |
| `fi`    | Finnois               | Suomi            |
| `el`    | Grec                 | Ελληνικά         |
| `cs`    | Tchèque                 | Čeština          |
| `hu`    | Hongrois             | Magyar           |
| `ro`    | Roumain              | Română           |
| `uk`    | Ukrainien             | Українська       |
| `pt-BR` | Portugais brésilien  | Português Brasileiro |
| `zh-HK` | Cantonais             | 粵語             |
| `ms`    | Malais                | Bahasa Melayu    |
| `sk`    | Slovaque                | Slovenčina       |
| `bg`    | Bulgare             | Български        |
| `hr`    | Croate              | Hrvatski         |
| `lt`    | Lituanien            | Lietuvių         |
| `lv`    | Letton               | Latviešu         |
| `et`    | Estonien              | Eesti            |
| `sl`    | Slovène             | Slovenščina      |

{% hint style="success" %}
**Persistance automatique** : votre préférence linguistique est enregistrée dans `~/.chloros/cli_language.json` et est conservée d&#x27;une session à l&#x27;autre.
{% endhint %}

***

### `set-project-folder` - Définir le dossier de projet par défaut

Modifie l&#x27;emplacement du dossier de projet par défaut (partagé avec l&#x27;interface graphique dans Windows).

**Syntaxe :**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exemples :**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Afficher le dossier de projet

Affiche l&#x27;emplacement actuel du dossier de projet par défaut.

**Syntaxe :**

```bash
chloros-cli get-project-folder
```

**Exemple :**

```bash
chloros-cli get-project-folder
```

**Résultat :**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Réinitialiser aux paramètres par défaut

Réinitialise le dossier de projet à son emplacement par défaut.

**Syntaxe :**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Exécuter les diagnostics système

Exécute 7 vérifications de diagnostic pour vérifier la configuration de votre système.

**Syntaxe :**

```bash
chloros-cli selftest
```

**Diagnostics effectués :**

1. Vérification de la version
2. Disponibilité du port (5000)
3. Démarrage du backend
4. Test de connectivité API
5. Informations système et détection du GPU
6. Vérification des modèles de débruitage
7. Vérification de la disponibilité de CUDA

{% hint style="info" %}
**Utile pour le dépannage** : exécutez `selftest` après l&#x27;installation pour vérifier que votre système est correctement configuré, en particulier sur Linux/Jetson où la configuration du GPU et de CUDA peut nécessiter une vérification.
{% endhint %}

***

### `update` - Recherche de mises à jour (Linux uniquement)

Recherche et installation des mises à jour CLI sur les systèmes Linux.

**Syntaxe :**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Option    | Description                        |
| --------- | ---------------------------------- |
| `--check` | Rechercher uniquement les mises à jour, ne pas les installer |

{% hint style="info" %}
Cette commande est disponible uniquement sur Linux. Sur Windows, les mises à jour sont fournies via le programme d&#x27;installation.
{% endhint %}

***

## Options globales

Ces options s&#x27;appliquent à toutes les commandes :

| Option            | Type    | Par défaut       | Description                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Chemin    | Détecté automatiquement | Chemin vers l&#x27;exécutable du backend                       |
| `--port`          | Entier | 5000          | Numéro de port du backend API                          |
| `--restart`       | Indicateur    | -             | Forcer le redémarrage du backend (tue les processus existants) |
| `--version`       | Indicateur    | -             | Afficher les informations de version et quitter                |
| `--help`          | Indicateur    | -             | Afficher les informations d&#x27;aide et quitter                   |

{% hint style="info" %}
**Détection automatique du backend** : le chemin `--backend-exe` est détecté automatiquement selon la plateforme :
* **Windows** : `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)** : `/usr/lib/chloros/chloros-backend`
* **Linux (manuel)** : `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Exemple avec options globales :**

**Windows :**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux :**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Guide des paramètres de traitement

### Traitement parallèle et adaptation dynamique du calcul

Chloros 1.1.0 inclut l&#x27;[adaptation dynamique du calcul](processing-architecture/dynamic-compute-adaptation.md) — le moteur de traitement **détecte automatiquement votre matériel** et sélectionne la stratégie optimale :

| Plateforme | Stratégie | Workers | Pipeline | Remarques |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 Go** | `GPU_SINGLE` | 1 | `tiled_gpu` | Économique en mémoire, sérialisé |
| **Jetson Orin NX 16 Go** | `GPU_PARALLEL` | 3 | `fused_gpu` | Traitement GPU simultané |
| **Ordinateur de bureau avec GPU 8 Go** | `GPU_SINGLE` | 3 | `tiled_gpu` | Bonnes performances de bureau |
| **Ordinateur de bureau avec GPU de 12 Go ou plus** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Performances optimales sur ordinateur de bureau |
| **Système avec processeur uniquement** | `CPU_PARALLEL` | cœurs - 1 | `cpu_fallback` | Aucun GPU requis |

{% hint style="success" %}
**Aucune configuration manuelle requise !** Chloros détecte automatiquement votre CPU, votre GPU, votre RAM et (sur Jetson) vos capteurs thermiques, puis configure automatiquement le pipeline de traitement optimal.
{% endhint %}

### Méthodes de débayage

| Méthode | Indicateur CLI | Qualité | Vitesse | Licence |
| --- | --- | --- | --- | --- |
| **Standard (Rapide, qualité moyenne)** | `--debayer standard` | Bonne | Rapide | Gratuit / Chloros+ |
| **Sensible à la texture (lent, qualité optimale)** | `--debayer texture-aware` | Optimale | Lent | Chloros+ uniquement |

La méthode de débayérisation par défaut est **Standard**. La méthode**Texture Aware** utilise un modèle de débruitage IA/ML pour un résultat de la plus haute qualité, mais nécessite une licence Chloros+ et un GPU NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Correction du vignettage

**Fonction :** Corrige la perte de luminosité sur les bords de l&#x27;image (coins plus sombres fréquents dans les images prises par une caméra).

* **Activé par défaut** - La plupart des utilisateurs devraient laisser cette option activée
* Utilisez `--no-vignette` pour la désactiver

{% hint style="success" %}
**Recommandation** : Activez toujours la correction de vignettage pour garantir une luminosité uniforme sur l&#x27;ensemble du cadre.
{% endhint %}

### Calibrage de la réflectance

Convertit les valeurs brutes du capteur en pourcentages de réflectance normalisés à l&#x27;aide de panneaux de calibrage.

* **Activé par défaut** - Indispensable pour l&#x27;analyse de la végétation
* Nécessite la présence de panneaux de calibrage dans les images
* Utilisez `--no-reflectance` pour désactiver

{% hint style="info" %}
**Conditions requises** : assurez-vous que les panneaux d&#x27;étalonnage sont correctement exposés et visibles dans vos images pour une conversion précise de la réflectance.
{% endhint %}

### Corrections PPK

**Fonction :** Applique des corrections cinématiques post-traitement à l&#x27;aide des données de journal DAQ-A-SD pour améliorer la précision du GPS.

* **Désactivé par défaut**
* Utilisez `--ppk` pour l&#x27;activer
* Nécessite des fichiers .daq dans le dossier du projet provenant du capteur de lumière DAQ-A-SD MAPIR.

### Formats de sortie

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Profondeur de bits</th><th width="116.5999755859375">Taille du fichier</th><th>Idéal pour</th></tr></thead><tbody><tr><td><strong>TIFF (16 bits)</strong> ⭐</td><td>Entier 16 bits</td><td>Grand</td><td>Analyse SIG, photogrammétrie (recommandé)</td></tr><tr><td><strong>TIFF (32 bits, pourcentage)</strong></td><td>Nombre à virgule flottante 32 bits</td><td>Très grand</td><td>Analyse scientifique, recherche</td></tr><tr><td><strong>PNG (8 bits)</strong></td><td>Entier 8 bits</td><td>Moyen</td><td>Inspection visuelle, partage sur le Web</td></tr><tr><td><strong>JPG (8 bits)</strong></td><td>Entier 8 bits</td><td>Petit</td><td>Aperçu rapide, sortie compressée</td></tr></tbody></table>***

## Automatisation et scripts

### Traitement par lots PowerShell (Windows)

Traitez automatiquement plusieurs dossiers de jeux de données sur Windows :

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Script par lots Windows (Windows)

Boucle simple pour le traitement par lots sur Windows :

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Traitement par lots Bash (Linux)

Traiter plusieurs dossiers de jeux de données sur Linux :

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Script d&#x27;automatisation Python (multiplateforme)

Automatisation avancée avec gestion des erreurs (fonctionne sur Windows et Linux) :

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Flux de traitement

### Flux de travail standard

1. **Entrée** : dossier contenant des paires d&#x27;images RAW/JPG
2. **Détection** : CLI recherche automatiquement les fichiers image pris en charge
3. **Traitement** : le mode parallèle s&#x27;adapte au nombre de cœurs de votre processeur (Chloros+)
4. **Sortie** : Crée des sous-dossiers par modèle d&#x27;appareil photo contenant les images traitées

### Exemple de structure de sortie

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Estimations du temps de traitement

Temps de traitement types pour 100 images (12 MP chacune) :

| Plateforme | Mode | Temps estimé | Remarques |
| --- | --- | --- | --- |
| **Ordinateur de bureau avec GPU 12 Go+** | `GPU_PARALLEL` | 5-10 min | Option la plus rapide |
| **Ordinateur de bureau avec GPU 8 Go** | `GPU_SINGLE` | 10-15 min | Bonnes performances |
| **Jetson Orin NX 16 Go** | `GPU_PARALLEL` | 15-25 min | Calcul en périphérie |
| **Jetson Nano 8 Go** | `GPU_SINGLE` | 30-60 min | Mémoire limitée |
| **CPU uniquement** | `CPU_PARALLEL` | 20-40 min | Pas de GPU requis |

{% hint style="info" %}
**Conseil de performance** : le temps de traitement varie en fonction du nombre d&#x27;images, de la résolution, de la méthode de débayérisation et du matériel. La débayérisation « Texture Aware » prend nettement plus de temps que la méthode standard. Voir [Adaptation dynamique du calcul](processing-architecture/dynamic-compute-adaptation.md) pour plus de détails.
{% endhint %}

***

## Dépannage

### CLI introuvable

**Erreur Windows :**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Solutions :**

1. Vérifiez l&#x27;emplacement d&#x27;installation :

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utilisez le chemin d&#x27;accès complet s&#x27;il ne figure pas dans PATH :

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ajoutez-le manuellement au PATH :
   * Ouvrez Propriétés du système → Variables d&#x27;environnement
   * Modifiez la variable PATH
   * Ajoutez : `C:\Program Files\Chloros\resources\cli`
   * Redémarrez le terminal

**Erreur Linux :**

```
chloros-cli: command not found
```

**Linux Solutions :**

1. Vérifiez l&#x27;installation :

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Rechargez votre shell :

```bash
source ~/.bashrc
```

3. Vérifiez les autorisations :

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Échec du démarrage du backend**Erreur :**

```

Backend failed to start within 30 seconds
```

**Solutions :**

1. Vérifiez si le backend est déjà en cours d&#x27;exécution (fermez-le d&#x27;abord)
2. Vérifiez que le pare-feu ne bloque pas le trafic (Windows) ou vérifiez la disponibilité du port (Linux : `lsof -i :5000`)
3. Essayez un autre port :

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Forcez le redémarrage du backend :

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Sur Linux, vérifiez que l&#x27;exécutable du backend existe :

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problèmes de licence / d&#x27;authentification**Erreur :**

```

Chloros+ license required for CLI access
```

**Solutions :**

1. Vérifiez que vous disposez d&#x27;un abonnement Chloros+ actif
2. Connectez-vous avec vos identifiants :

```bash
chloros-cli login user@example.com 'password'
```

3. Vérifiez l&#x27;état de la licence :

```bash
chloros-cli status
```

4. Contactez le support : info@mapir.camera

***

### Aucune image trouvée**Erreur :**

```

No images found in the specified folder
```

**Solutions :**

1. Vérifiez que le dossier contient des formats pris en charge (.RAW, .TIF, .JPG)
2. Vérifiez que le chemin d&#x27;accès au dossier est correct (utilisez des guillemets pour les chemins contenant des espaces)
3. Assurez-vous de disposer des droits de lecture pour le dossier
4. Vérifiez que les extensions de fichiers sont correctes

***

### Le traitement se bloque ou s&#x27;arrête**Solutions :**

1. Vérifiez l&#x27;espace disque disponible (assurez-vous qu&#x27;il y en a suffisamment pour la sortie)
2. Fermez les autres applications pour libérer de la mémoire
3. Réduisez le nombre d&#x27;images (traitez par lots)

***

### Port déjà utilisé**Erreur :**

```

Port 5000 is already in use
```

**Solutions :**

**Windows :**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux :**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## FAQ

### Q : Ai-je besoin d&#x27;une licence pour CLI ?

**R :**Oui ! CLI nécessite une**licence Chloros+** payante.

* ❌ Formule Standard (gratuite) : CLI désactivé
* ✅ Formules Chloros+ (payantes) : CLI entièrement activé

Abonnez-vous sur : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q : Puis-je utiliser CLI sur un serveur sans interface graphique ?**R :** Oui ! CLI fonctionne entièrement en mode headless. C&#x27;est le principal cas d&#x27;utilisation de Linux.**Serveur Windows :**
* Serveur Windows 2016 ou version ultérieure
* Visual C++ Redistributable installé

**Serveur Linux :**
* Ubuntu 20.04+ / Debian 11+ (amd64) ou JetPack 6 (arm64)
* Installation via le paquet `.deb`

**Les deux plateformes :**
* 8 Go de RAM minimum (16 Go recommandés)
* Activation unique de la licence : `chloros-cli login user@example.com 'password'`

***

### Q : Où sont enregistrées les images traitées ?**R :**Par défaut, les images traitées sont enregistrées dans le**même dossier que celui d&#x27;entrée**, dans des sous-dossiers correspondant au modèle d&#x27;appareil photo (par exemple, `Survey3N_RGN/`).

Utilisez l&#x27;option `-o` pour spécifier un dossier de sortie différent :

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Q : Puis-je traiter plusieurs dossiers à la fois ?**R :** Pas directement en une seule commande, mais vous pouvez utiliser des scripts pour traiter les dossiers de manière séquentielle. Consultez la section [Automatisation et scripts](CLI.md#automation--scripting).***

### Q : Comment enregistrer la sortie de CLI dans un fichier journal ?**PowerShell :**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch :**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash :**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Q : Que se passe-t-il si j&#x27;appuie sur Ctrl+C pendant le traitement ?**R :** CLI va :

1. Arrêter le traitement en douceur
2. Fermer le backend
3. Quitter avec le code 130

Des images partiellement traitées peuvent rester dans le dossier de sortie.

***

### Q : Puis-je automatiser le traitement de CLI ?**R :** Absolument ! CLI est conçu pour l&#x27;automatisation. Consultez la section [Automatisation et scripts](CLI.md#automation--scripting) pour PowerShell (Windows), Batch (Windows), Bash (Linux) et Python (multiplateforme).***

### Q : Comment vérifier la version de CLI ?**R :**

```bash
chloros-cli --version
```

**Résultat :**

```

Chloros CLI 1.1.0
```

***

## Obtenir de l&#x27;aide

### Aide en ligne de commande

Affichez les informations d&#x27;aide directement dans CLI :

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canaux d&#x27;assistance

* **E-mail** : info@mapir.camera
* **Site web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Tarifs** : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Exemples complets

### Exemple 1 : Traitement de base

Traitement avec les paramètres par défaut (vignette, réflectance) :

**Windows :**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux :**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Exemple 2 : Résultats scientifiques de haute qualité

32 bits en virgule flottante TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux :**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Exemple 3 : Traitement rapide des aperçus

PNG 8 bits sans étalonnage pour un examen rapide :

**Windows :**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux :**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Exemple 4 : Traitement corrigé par PPK

Appliquer les corrections PPK avec la réflectance :

**Windows :**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux :**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Exemple 5 : Emplacement de sortie personnalisé

Traiter vers un emplacement différent avec un format spécifique :

**Windows :**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux :**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Exemple 6 : Workflow d&#x27;authentification

Flux d&#x27;authentification complet (identique sur toutes les plateformes) :

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exemple 7 : Utilisation multilingue

Changement de la langue de l&#x27;interface (identique sur toutes les plateformes) :

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
