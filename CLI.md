# CLI : Ligne de commande

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

Le **Chloros CLI** offre un accès puissant en ligne de commande au moteur de traitement d'images Chloros, permettant l'automatisation, l'écriture de scripts et le fonctionnement sans tête pour vos flux de travail d'imagerie.

### Caractéristiques principales

* 🚀 **Automation** - Traitement par lots de plusieurs ensembles de données par le biais de scripts
* 🔗 **Intégration** - Intégration dans les flux de travail et les pipelines existants
* 💻 **Headless Operation** - Exécution sans interface graphique
* 🌍 **Multi-langue** - Prise en charge de 38 langues
* ⚡ **Traitement parallèle** - S'adapte dynamiquement à votre processeur (jusqu'à 16 travailleurs parallèles)

### Exigences

| Exigences | Détails |
| -------------------- | ------------------------------------------------------------------- |
| Système d'exploitation** Windows 10/11 (64-bit)
| **Licence** | Chloros+ ([plan payant requis](https://cloud.mapir.camera/pricing)) |
| Mémoire**** 8 Go de RAM minimum (16 Go recommandés)
**Internet** | Nécessaire pour l'activation de la licence
| Espace disque*** : varie en fonction de la taille du projet

{% hint style="warning" %}
**License Requirement**: The CLI requires a paid Chloros+ subscription. Standard (free) plans do not have CLI access. Visit [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) to upgrade.
{% endhint %}

## Démarrage rapide

### Installation

Le CLI est automatiquement inclus dans l'installateur Chloros :

1. Téléchargez et exécutez **Chloros Installer.exe**
2. Complétez l'assistant d'installation
3. CLI installé sur : `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
The installer automatically adds `chloros-cli` to your system PATH. Restart your terminal after installation.
{% endhint %}

### Première installation

Avant d'utiliser le CLI, activez votre licence Chloros+ :

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Utilisation de base

Traite un dossier avec les paramètres par défaut :

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Référence de la commande

### Syntaxe générale

```
chloros-cli [global-options] <command> [command-options]
```

***

## Commandes

### `process` - Traiter les images

Traite les images d'un dossier avec l'étalonnage.

**Syntaxe:**

```bash
chloros-cli process <input-folder> [options]
```

**Exemple:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Options de commande du processus

| Option | Type | Défaut | Description
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
`<input-folder>` | Chemin d'accès | _Required_ | Dossier contenant les images multispectrales RAW/JPG | `<input-folder>` | Chemin d'accès | Identique à l'entrée
| `-o, --output` | Chemin d'accès | Identique à l'entrée | Dossier de sortie pour les images traitées |
| `-n, --project-name` | Chaîne de caractères | Auto-générée | Nom de projet personnalisé
| `--vignette` | Drapeau | Activé | Activer la correction de la vignette | `--vignette` | Drapeau | Activé | Activer la correction de la vignette
`--no-vignette` | Drapeau | - | Désactiver la correction de vignette |
`--reflectance` | Drapeau | Activé | Activer l'étalonnage de la réflectance |
`--no-reflectance` | Drapeau | - | Désactiver l'étalonnage de la réflectance |
| Drapeau | Désactivé | Appliquer les corrections PPK à partir des données du capteur de lumière .daq | Drapeau | Désactivé | Désactivé
| `--format` | Choix | TIFF (16 bits) | Format de sortie : `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| Entier | Auto | Taille minimale de la cible en pixels pour la détection du panneau d'étalonnage |
`--target-clustering` | Entier | Auto | Seuil de regroupement des cibles (0-100) |
| Chaîne | Aucune | Verrouillage de l'exposition pour le modèle de caméra (broche 1) |
| | Chaîne | Aucune | Verrouillage de l'exposition pour le modèle de caméra (broche 2) | `--exposure-pin-2` | Chaîne | Aucune | Verrouillage de l'exposition pour le modèle de caméra (broche 2) |
| | Entier | Auto | Intervalle de recalibrage en secondes | `--recal-interval` | Entier | Auto | Intervalle de recalibrage en secondes
| `--timezone-offset` | Entier | 0 | Décalage du fuseau horaire en heures |

***

### `login` - Authentification du compte

Connectez-vous avec vos Chloros+ informations d'identification pour permettre le traitement CLI.

**Syntaxe:**

```bash
chloros-cli login <email> <password>
```

**Exemple:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Special Characters**: Use single quotes around passwords containing characters like `$`, `!`, or spaces.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

***

### `logout` - Effacer les informations d'identification

Effacez les informations d'identification enregistrées et déconnectez-vous de votre compte.

**Syntaxe:**

```bash
chloros-cli logout
```

**Exemple:**

```powershell
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Vérifier l'état de la licence

Affiche l'état actuel de la licence et de l'authentification.

**Syntaxe:**

```bash
chloros-cli status
```

**Exemple:**

```powershell
chloros-cli status
```

**Output:**

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

### `export-status` - Vérifier la progression de l'exportation

Contrôler la progression de l'exportation du fil 4 pendant ou après le traitement.

**Syntaxe:**

```bash
chloros-cli export-status
```

**Exemple:**

```powershell
chloros-cli export-status
```

**Use Case:** Appeler cette commande en cours de traitement pour vérifier la progression de l'exportation.

***

### `language` - Gérer la langue de l'interface

Affichez ou modifiez la langue de l'interface CLI.

**Syntaxe:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Exemples:**

```powershell
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Langues supportées (38 au total)

| Code | Langue | Nom natif
| ------- | --------------------- | ---------------- |
| `en` | Anglais | English | English |
| `es` | Espagnol | Español |
| `pt` | Portugais | Português |
| `fr` | Français | French | Français |
| `de` | Allemand | Deutsch |
| `it` | Italian | Italiano |
| | | Japonais | 日本語 | | Japonais | 日本語 | | Japonais | 日本語 | Japonais
| `ko` | Coréen | 한국어 |
| `zh` | Chinois (simplifié) | 简体中文 |
| 繁體中文 | Chinese (Traditional) | 繁體中文 |
| | Russe | Русский
`nl` | Dutch | Nederlands |
| العربية | arabe | العربية | العربية | العربية | arabe
| `pl` | Polish | Polski |
| `tr` | Turc | Türkçe |
| Hindi | हिंदी | Hindi | हिंदी | Hindi | हिी | Hindi
| `id` | Indonésien | Bahasa Indonesia |
| Tiếng Việt | Viêt Nam | Tiếng Việt | Vietnamien
| `th` | Thai | ไทย |
| `sv` | suédois | Svenska |
| `da` | Danois | Dansk |
| `no` | Norvégien | Norsk |
`fi` | Finlandaise | Suomi |
| Grecque | Ελληνικά |
`cs` | Czech | Čeština |
| `hu` | Hongrois | Magyar |
`ro` | Roumain | Română |
| | Ukrainien | Українська |
| `pt-BR` | Brazilian Portuguese | Português Brasileiro |
| | Cantonese | 粵語 |
| `ms` | Malais | Bahasa Melayu |
| `sk` | Slovak | Slovenčina |
| Български | Български | Български | Български
`hr` | Croate | Hrvatski |
`lt` | Lithuanien | Lietuvių |
| `lv` | Letton | Latviešu |
| | Estonien | Eesti |
| `sl` | Slovène | Slovenščina |

{% hint style="success" %}
**Automatic Persistence**: Your language preference is saved to `~/.chloros/cli_language.json` and persists across all sessions.
{% endhint %}

***

### `set-project-folder` - Définir le dossier de projet par défaut

Modifier l'emplacement du dossier de projet par défaut (partagé avec l'interface graphique).

**Syntaxe:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exemple:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Afficher le dossier du projet

Affiche l'emplacement actuel du dossier de projet par défaut.

**Syntaxe:**

```bash
chloros-cli get-project-folder
```

**Exemple:**

```powershell
chloros-cli get-project-folder
```

**Output:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Réinitialisation par défaut

Réinitialise le dossier du projet à l'emplacement par défaut.

**Syntaxe:**

```bash
chloros-cli reset-project-folder
```

***

## Options globales

Ces options s'appliquent à toutes les commandes :

| Option | Type | Valeur par défaut | Description |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Chemin d'accès | Auto-détecté | Chemin d'accès à l'exécutable du backend | `--port` | Integer | 5000 | Chemin d'accès au backend
`--port` | Entier | 5000 | Numéro de port du backend API API | Numéro de port du backend API | Numéro de port du backend
`--restart` | Drapeau | - | Forcer le redémarrage du backend (tue les processus existants) |
`--version` | Drapeau | - | Afficher les informations sur la version et quitter |
`--help` | Drapeau | - | Afficher les informations d'aide et quitter |

**Exemple d'options globales:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guide des paramètres de traitement

### Traitement parallèle

Chloros+ CLI **Échelonne automatiquement** le traitement parallèle en fonction des capacités de votre ordinateur :

**Comment ça marche:**

* Détecte les cœurs de votre processeur et la RAM
* Alloue des travailleurs : **2× cœurs de CPU** (utilise l'hyperthreading)
* **Maximum : 16 travailleurs parallèles** (pour la stabilité)

**Tiers du système:**

| Type de système - CPU - RAM - Travailleurs - Performances - Niveau de performance - Niveau de performance - Niveau de performance
| ------------- | ---------- | -------- | -------- | --------------- |
**Haut de gamme** | 16+ cœurs | 32+ Go | Jusqu'à 16 | Vitesse maximale
**Moyen de gamme** | 8-15 cœurs | 16-31 Go | 8-16 | Excellente vitesse
**Bas de gamme** | 4-7 cœurs | 8-15 Go | 4-8 | Bonne vitesse |

{% hint style="success" %}
**Automatic Optimization**: The CLI automatically detects your system specs and configures optimal parallel processing. No manual configuration needed!
{% endhint %}

### Méthodes Debayer

Le CLI utilise **Haute qualité (plus rapide)** comme algorithme de débayerisation par défaut et recommandé :

| La méthode de débayerisation est la suivante : - Méthode | Qualité | Vitesse | Description |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Haute qualité (plus rapide)** ⭐ | ⭐⭐⭐⭐ | ⚡⚡⚡⚡ | Algorithme tenant compte des contours (par défaut, recommandé) |

### Correction de la vignette

**Ce qu'il fait:** Corrige la chute de lumière sur les bords de l'image (coins plus sombres communs dans l'imagerie de l'appareil photo).

* **Activée par défaut** - La plupart des utilisateurs devraient la laisser activée
* Utilisez `--no-vignette` pour la désactiver

{% hint style="success" %}
**Recommendation**: Always enable vignette correction to ensure uniform brightness across the frame.
{% endhint %}

### Calibration de la réflectance

Convertit les valeurs brutes des capteurs en pourcentages de réflectance normalisés à l'aide de panneaux d'étalonnage.

* **Activé par défaut** - Essentiel pour l'analyse de la végétation
* Nécessite des panneaux cibles d'étalonnage dans les images
* Utilisez `--no-reflectance` pour le désactiver

{% hint style="info" %}
**Requirements**: Ensure calibration panels are properly exposed and visible in your images for accurate reflectance conversion.
{% endhint %}

### Corrections PPK

**Ce qu'il fait:** Applique des corrections cinématiques post-traitées à l'aide des données d'enregistrement DAQ-A-SD pour améliorer la précision du GPS.

* **Désactivé par défaut
* Utilisez `--ppk` pour l'activer
* Nécessite des fichiers .daq dans le dossier du projet à partir du capteur de lumière DAQ-A-SD.

### Formats de sortie

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bit Depth</th><th width="116.5999755859375">File Size</th><th>Best For</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bit integer</td><td>Large</td><td>GIS analysis, photogrammetry (recommended)</td></tr><tr><td><strong>TIFF (32-bit, Percent)</strong></td><td>32-bit float</td><td>Very Large</td><td>Scientific analysis, research</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit integer</td><td>Medium</td><td>Visual inspection, web sharing</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bit integer</td><td>Small</td><td>Quick preview, compressed output</td></tr></tbody></table>

***

## Automatisation et scripts

### PowerShell Batch Processing

Traiter automatiquement plusieurs dossiers d'ensembles de données :

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

### Windows Script par lots

Boucle simple pour le traitement par lots :

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

### Python Script d'automatisation

Automatisation avancée avec gestion des erreurs :

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

## Traitement du flux de travail

### Workflow standard

1. **Entrée** : Dossier contenant des paires d'images RAW/JPG
2. **Découverte** : CLI recherche automatiquement les fichiers d'images pris en charge
3. **Traitement** : Le mode parallèle s'adapte aux cœurs de votre CPU (Chloros+)
4. **Sortie** : Crée des sous-dossiers de modèles de caméras avec les images traitées

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

### Estimation du temps de traitement

Temps de traitement typiques pour 100 images (12MP chacune) :

| Mode | Temps | Matériel
| ----------------- | --------- | -------------------------------------------- |
| i7/Ryzen 7, 16 Go de RAM, SSD (jusqu'à 16 travailleurs) | Mode parallèle** 5-10 min
| Mode parallèle : i5/Ryzen 5, 8 Go de RAM, disque dur (jusqu'à 8 travailleurs)

{% hint style="info" %}
**Performance Tip**: Processing time varies based on image count, resolution, and computer specs.
{% endhint %}

***

## Dépannage

### CLI Non trouvé

**Error:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Solutions:**

1. Vérifier l'emplacement de l'installation :

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utiliser le chemin complet s'il n'est pas dans PATH :

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ajouter manuellement au PATH :
   * Ouvrir Propriétés du système → Variables d'environnement
   * Modifier la variable PATH
   * Ajouter : `C:\Program Files\Chloros\resources\cli`
   * Redémarrer le terminal

***

### Échec du démarrage du backend

**Error:**

```
Backend failed to start within 30 seconds
```

**Solutions:**

1. Vérifier si le backend fonctionne déjà (le fermer d'abord)
2. Vérifier que Windows Le pare-feu n'est pas bloqué
3. Essayer un autre port :

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Forcer le redémarrage du backend :

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problèmes de licence / d'authentification

**Error:**

```
Chloros+ license required for CLI access
```

**Solutions:**

1. Vérifiez que vous avez un abonnement actif Chloros+
2. Connectez-vous avec vos identifiants :

```powershell
chloros-cli login user@example.com 'password'
```

3. Vérifier le statut de la licence :

```powershell
chloros-cli status
```

4. Contacter le support : info@mapir.camera

***

### Aucune image trouvée

**Error:**

```
No images found in the specified folder
```

**Solutions:**

1. Vérifier que le dossier contient les formats supportés (.RAW, .TIF, .JPG)
2. Vérifier que le chemin d'accès au dossier est correct (utiliser des guillemets pour les chemins d'accès comportant des espaces)
3. Vérifiez que vous disposez des autorisations de lecture pour le dossier
4. Vérifier que les extensions de fichiers sont correctes

***

### Le traitement s'arrête ou se bloque

**Solutions:**

1. Vérifier l'espace disque disponible (s'assurer qu'il est suffisant pour la sortie)
2. Fermer les autres applications pour libérer de la mémoire
3. Réduire le nombre d'images (traitement par lots)

***

### Port déjà utilisé

**Error:**

```
Port 5000 is already in use
```

**Solution:**

Spécifiez un port différent :

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## FAQ

### Q : Ai-je besoin d'une licence pour le CLI ?

**A:** Oui ! La CLI nécessite une licence payante **Chloros+.

* ❌ Plan standard (gratuit) : CLI désactivé
* ✅ Chloros+ (payant) : CLI entièrement activé

S'abonner à : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q : Puis-je utiliser le CLI sur un serveur sans GUI ?

**A:** Oui ! Le CLI fonctionne complètement sans interface graphique. Exigences :

* Windows Server 2016 ou version ultérieure
* Visual C++ Redistributable installé
* Mémoire vive suffisante (8 Go minimum, 16 Go recommandés)
* Activation unique de la licence d'interface graphique sur n'importe quelle machine

***

### Q : Où sont sauvegardées les images traitées ?

**A:** Par défaut, les images traitées sont enregistrées dans le **même dossier que l'entrée** dans les sous-dossiers du modèle d'appareil photo (par exemple, `Survey3N_RGN/`).

Utilisez l'option `-o` pour spécifier un dossier de sortie différent :

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Q : Puis-je traiter plusieurs dossiers à la fois ?

**A:** Pas directement en une seule commande, mais vous pouvez utiliser des scripts pour traiter les dossiers de manière séquentielle. Voir la section [Automation & Scripting](CLI.md#automation--scripting).

***

### Q : Comment enregistrer la sortie CLI dans un fichier journal ?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Q : Que se passe-t-il si j'appuie sur Ctrl+C pendant le traitement ?

**A:** Le CLI sera :

1. Arrêtera le traitement avec élégance
2. Arrêter le backend
3. Quitter avec le code 130

Des images partiellement traitées peuvent rester dans le dossier de sortie.

***

### Q : Puis-je automatiser le traitement CLI ?

**A:** Absolument ! Le CLI est conçu pour être automatisé. Voir [Automation & Scripting](CLI.md#automation--scripting) pour des exemples PowerShell, Batch, et Python.

***

### Q : Comment puis-je vérifier la version CLI ?

**A:**

```powershell
chloros-cli --version
```

**Sortie:**

```
Chloros CLI 1.0.2
```

***

## Obtenir de l'aide

### Aide en ligne de commande

Affichez des informations d'aide directement dans la CLI :

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canaux d'assistance

* **Email** : info@mapir.camera
* **Site web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Tarification** : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Exemples complets

### Exemple 1 : Traitement de base

Traiter avec les paramètres par défaut (vignette, réflectance) :

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Exemple 2 : Résultats scientifiques de haute qualité

flotteur 32 bits TIFF :

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Exemple 3 : Traitement rapide de l'aperçu

8-bit PNG sans calibration pour un examen rapide :

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Exemple 4 : Traitement corrigé par PPK

Appliquer les corrections PPK avec la réflectance :

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Exemple 5 : Emplacement de sortie personnalisé

Traiter vers un lecteur différent avec un format spécifique :

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Exemple 6 : flux de travail d'authentification

Flux d'authentification complet :

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exemple 7 : Utilisation multilingue

Changer la langue de l'interface :

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
