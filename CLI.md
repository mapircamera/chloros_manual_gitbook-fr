# CLI : Ligne de commande

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>Le **Chloros CLI** fournit un accès puissant par ligne de commande au moteur de traitement d&#x27;images Chloros, permettant l&#x27;automatisation, la création de scripts et le fonctionnement sans affichage pour vos flux de travail d&#x27;imagerie.

### Principales fonctionnalités

* 🚀 **Automatisation** - Traitement par lots de plusieurs ensembles de données à l&#x27;aide de scripts
* 🔗 **Intégration** - Intégration dans les flux de travail et les pipelines existants
* 💻 **Fonctionnement sans interface graphique** - Exécution sans interface graphique
* 🌍 **Multilingue** - Prise en charge de 38 langues
* ⚡ **Traitement parallèle** - S&#x27;adapte dynamiquement à votre CPU (jusqu&#x27;à 16 travailleurs parallèles)

### Configuration requise

| Configuration requise          | Détails                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Système d&#x27;exploitation** | Windows 10/11 (64 bits)                                              |
| **Licence**          | Chloros+ ([forfait payant requis](https://cloud.mapir.camera/pricing)) |
| **Mémoire**           | 8 Go de RAM minimum (16 Go recommandés)                                  |
| **Internet**         | Requis pour l&#x27;activation de la licence                                     |
| **Espace disque**       | Varie en fonction de la taille du projet                                              |

{% hint style=&quot;warning&quot; %}
**Exigence de licence** : CLI nécessite un abonnement payant à Chloros+. Les forfaits standard (gratuits) ne donnent pas accès à CLI. Rendez-vous sur [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) pour passer à un forfait supérieur.
{% endhint %}

## Démarrage rapide

### Installation

Le CLI est automatiquement inclus dans le programme d&#x27;installation Chloros :

1. Téléchargez et exécutez **Chloros Installer.exe**
2. Suivez les instructions de l&#x27;assistant d&#x27;installation
3. CLI installé dans : `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Le programme d&#x27;installation ajoute automatiquement `chloros-cli` au chemin d&#x27;accès PATH de votre système. Redémarrez votre terminal après l&#x27;installation.
{% endhint %}

### Première configuration

Avant d&#x27;utiliser CLI, activez votre licence Chloros+ :

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Utilisation de base

Traitez un dossier avec les paramètres par défaut :

```powershell
chloros-cli process "C:\Images\Dataset001"
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

Traite les images d&#x27;un dossier avec calibrage.

**Syntaxe :**

```bash
chloros-cli process <input-folder> [options]
```

**Exemple :**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Options de commande de traitement

| Option                | Type    | Par défaut        | Description                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Chemin    | _Obligatoire_     | Dossier contenant les images multispectrales RAW/JPG                                         |
| `-o, --output`        | Chemin    | Identique à l&#x27;entrée  | Dossier de sortie pour les images traitées                                                     |
| `-n, --project-name`  | Chaîne  | Généré automatiquement | Nom du projet personnalisé                                                                    |
| `--vignette`          | Indicateur    | Activé        | Activer la correction du vignettage                                                             |
| `--no-vignette`       | Indicateur    | -              | Désactiver la correction du vignettage                                                            |
| `--reflectance`       | Indicateur    | Activé        | Activer l&#x27;étalonnage de la réflectance                                                         |
| `--no-reflectance`    | Indicateur    | -              | Désactiver l&#x27;étalonnage de la réflectance                                                        |
| `--ppk`               | Indicateur    | Désactivé       | Appliquer les corrections PPK à partir des données du capteur de lumière .daq                                      |
| `--format`            | Choix  | TIFF (16 bits)  | Format de sortie : `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Entier | Auto           | Taille minimale cible en pixels pour la détection du panneau d&#x27;étalonnage                          |
| `--target-clustering` | Entier | Auto           | Seuil de regroupement des cibles (0-100)                                                    |
| `--exposure-pin-1`    | Chaîne  | Aucun           | Verrouillage de l&#x27;exposition pour le modèle de caméra (broche 1)                                                 |
| `--exposure-pin-2`    | Chaîne  | Aucun           | Verrouillage de l&#x27;exposition pour le modèle de caméra (broche 2)                                                 |
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

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caractères spéciaux** : utilisez des guillemets simples autour des mots de passe contenant des caractères tels que `$`, `!` ou des espaces.
{% endhint %}

**Résultat :**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Effacer les identifiants

Effacez les identifiants enregistrés et déconnectez-vous de votre compte.

**Syntaxe :**

```bash
chloros-cli logout
```

**Exemple :**

```powershell
chloros-cli logout
```

**Sortie :**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Vérifier l&#x27;état de la licence

Affiche l&#x27;état actuel de la licence et de l&#x27;authentification.

**Syntaxe :**

```bash
chloros-cli status
```

**Exemple :**

```powershell
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

Surveille la progression de l&#x27;exportation du thread 4 pendant ou après le traitement.

**Syntaxe :**

```bash
chloros-cli export-status
```

**Exemple :**

```powershell
chloros-cli export-status
```

**Cas d&#x27;utilisation :** Appelez cette commande pendant le traitement pour vérifier la progression de l&#x27;exportation.

***

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

#### Langues prises en charge (38 au total)

| Code    | Langue              | Nom natif      |
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
| `nl`    | Néerlandais                 | Nederlands       |
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
| `ms`    | Malais                 | Bahasa Melayu    |
| `sk`    | Slovaque                | Slovenčina       |
| `bg`    | Bulgare             | Български        |
| `hr`    | Croate              | Hrvatski         |
| `lt`    | Lituanien            | Lietuvių         |
| `lv`    | Letton               | Latviešu         |
| `et`    | Estonien              | Eesti            |
| `sl`    | Slovène             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Persistance automatique** : votre préférence linguistique est enregistrée dans `~/.chloros/cli_language.json` et persiste tout au long des sessions.
{% endhint %}

***

### `set-project-folder` - Définir le dossier de projet par défaut

Modifiez l&#x27;emplacement du dossier de projet par défaut (partagé avec l&#x27;interface graphique).

**Syntaxe :**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exemple :**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Afficher le dossier du projet

Affiche l&#x27;emplacement actuel du dossier de projet par défaut.

**Syntaxe :**

```bash
chloros-cli get-project-folder
```

**Exemple :**

```powershell
chloros-cli get-project-folder
```

**Sortie :**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Réinitialiser les paramètres par défaut

Réinitialise le dossier du projet à son emplacement par défaut.

**Syntaxe :**

```bash
chloros-cli reset-project-folder
```

***

## Options globales

Ces options s&#x27;appliquent à toutes les commandes :

| Option          | Type    | Par défaut       | Description                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Chemin    | Détecté automatiquement | Chemin vers l&#x27;exécutable backend                       |
| `--port`        | Entier | 5000          | Numéro de port backend API                          |
| `--restart`     | Indicateur    | -             | Forcer le redémarrage du backend (tue les processus existants) |
| `--version`     | Indicateur    | -             | Afficher les informations de version et quitter                |
| `--help`        | Indicateur    | -             | Afficher les informations d&#x27;aide et quitter                   |

**Exemple avec les options globales :**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guide des paramètres de traitement

### Traitement parallèle

Chloros+ CLI **adapte automatiquement** le traitement parallèle aux capacités de votre ordinateur :

**Fonctionnement :**

* Détecte les cœurs de votre processeur et votre mémoire vive
* Alloue les tâches : **2× cœurs de processeur** (utilise l&#x27;hyperthreading)
* **Maximum : 16 tâches parallèles** (pour la stabilité)

**Niveaux du système :**

| Type de système   | Processeur        | Mémoire vive      | Tâches  | Performances     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Haut de gamme**  | 16+ cœurs  | 32+ Go   | Jusqu&#x27;à 16 | Vitesse maximale   |
| **Milieu de gamme** | 8-15 cœurs | 16-31 Go | 8-16     | Excellente vitesse |
| **Bas de gamme**   | 4-7 cœurs  | 8-15 Go  | 4-8      | Bonne vitesse      |

{% hint style=&quot;success&quot; %}
**Optimisation automatique** : le CLI détecte automatiquement les spécifications de votre système et configure un traitement parallèle optimal. Aucune configuration manuelle n&#x27;est nécessaire !
{% endhint %}

### Méthodes de débayérisation

Le CLI utilise **Haute qualité (plus rapide)** comme algorithme de débayérisation par défaut et recommandé :

| Méthode                      | Qualité | Vitesse | Description                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Haute qualité (plus rapide)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algorithme sensible aux contours (par défaut, recommandé) |

### Correction du vignettage

**Fonction :** corrige la perte de lumière aux contours de l&#x27;image (coins plus sombres courants dans les images prises avec un appareil photo).

* **Activé par défaut** - La plupart des utilisateurs doivent laisser cette option activée
* Utilisez `--no-vignette` pour la désactiver

{% hint style=&quot;success&quot; %}
**Recommandation** : activez toujours la correction du vignettage pour garantir une luminosité uniforme sur l&#x27;ensemble du cadre.
{% endhint %}

### Calibrage de la réflectance

Convertit les valeurs brutes du capteur en pourcentages de réflectance normalisés à l&#x27;aide de panneaux de calibrage.

* **Activé par défaut** - Indispensable pour l&#x27;analyse de la végétation.
* Nécessite des panneaux cibles d&#x27;étalonnage dans les images.
* Utilisez `--no-reflectance` pour désactiver.

{% hint style=&quot;info&quot; %}
**Exigences** : assurez-vous que les panneaux d&#x27;étalonnage sont correctement exposés et visibles dans vos images pour une conversion précise de la réflectance.
{% endhint %}

### Corrections PPK

**Fonction :** applique des corrections cinématiques post-traitées à l&#x27;aide des données de journal DAQ-A-SD pour améliorer la précision du GPS.

* **Désactivé par défaut**
* Utilisez `--ppk` pour l&#x27;activer
* Nécessite des fichiers .daq dans le dossier du projet à partir du capteur de lumière DAQ-A-SD MAPIR.

### Formats de sortie

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Profondeur de bits</th><th width="116.5999755859375">Taille du fichier</th><th>Idéal pour</th></tr></thead><tbody><tr><td><strong>TIFF (16 bits)</strong> ⭐</td><td>Entier 16 bits</td><td>Grande</td><td>Analyse SIG, photogrammétrie (recommandé)</td></tr><tr><td><strong>TIFF (32 bits, pourcentage)</strong></td><td>Flottant 32 bits</td><td>Très grand</td><td>Analyse scientifique, recherche</td></tr><tr><td><strong>PNG (8 bits)</strong></td><td>Entier 8 bits</td><td>Moyen</td><td>Inspection visuelle, partage Web</td></tr><tr><td><strong>JPG (8 bits)</strong></td><td>Entier 8 bits</td><td>Petit</td><td>Aperçu rapide, sortie compressée</td></tr></tbody></table>***

## Automatisation et script

### Traitement par lots PowerShell

Traitez automatiquement plusieurs dossiers de jeux de données :

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

### Script par lots Windows

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

### Script d&#x27;automatisation Python

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

## Flux de travail de traitement

### Flux de travail standard

1. **Entrée** : dossier contenant des paires d&#x27;images RAW/JPG
2. **Détection** : CLI recherche automatiquement les fichiers image pris en charge
3. **Traitement** : le mode parallèle s&#x27;adapte à vos cœurs de processeur (Chloros+)
4. **Sortie** : crée des sous-dossiers par modèle d&#x27;appareil photo avec les images traitées

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

Temps de traitement type pour 100 images (12 MP chacune) :

| Mode              | Temps      | Matériel                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Mode parallèle** | 5-10 min  | i7/Ryzen 7, 16 Go de RAM, SSD (jusqu&#x27;à 16 travailleurs) |
| **Mode parallèle** | 10-15 min | i5/Ryzen 5, 8 Go de RAM, disque dur (jusqu&#x27;à 8 travailleurs)   |

{% hint style=&quot;info&quot; %}
**Conseil de performance** : le temps de traitement varie en fonction du nombre d&#x27;images, de la résolution et des spécifications de l&#x27;ordinateur.
{% endhint %}

***

## Dépannage

### CLI introuvable

**Erreur :**

```
'chloros-cli' is not recognized as an internal or external command
```

**Solutions :**

1. Vérifiez l&#x27;emplacement d&#x27;installation :

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utilisez le chemin complet s&#x27;il ne se trouve pas dans PATH :

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ajoutez-le manuellement à PATH :
   * Ouvrez Propriétés système → Variables d&#x27;environnement.
   * Modifiez la variable PATH.
   * Ajoutez : `C:\Program Files\Chloros\resources\cli`
   * Redémarrez le terminal.

***

### Échec du démarrage du backend.

**Erreur :**

```
Backend failed to start within 30 seconds
```

**Solutions :**

1. Vérifiez si le backend est déjà en cours d&#x27;exécution (fermez-le d&#x27;abord).
2. Vérifiez que le pare-feu Windows ne bloque pas.
3. Essayez un autre port :

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Forcez le redémarrage du backend :

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problèmes de licence / d&#x27;authentification

**Erreur :**

```
Chloros+ license required for CLI access
```

**Solutions :**

1. Vérifiez que vous disposez d&#x27;un abonnement Chloros+ actif.
2. Connectez-vous avec vos identifiants :

```powershell
chloros-cli login user@example.com 'password'
```

3. Vérifiez l&#x27;état de la licence :

```powershell
chloros-cli status
```

4. Contactez l&#x27;assistance : info@mapir.camera

***

### Aucune image trouvée

**Erreur :**

```
No images found in the specified folder
```

**Solutions :**

1. Vérifiez que le dossier contient des formats pris en charge (.RAW, .TIF, .JPG).
2. Vérifiez que le chemin d&#x27;accès au dossier est correct (utilisez des guillemets pour les chemins contenant des espaces).
3. Assurez-vous que vous disposez des droits d&#x27;accès en lecture pour le dossier.
4. Vérifiez que les extensions de fichiers sont correctes.

***

### Le traitement se bloque ou se fige

**Solutions :**

1. Vérifiez l&#x27;espace disque disponible (assurez-vous qu&#x27;il est suffisant pour la sortie).
2. Fermez les autres applications pour libérer de la mémoire.
3. Réduisez le nombre d&#x27;images (traitez-les par lots).

***

### Port déjà utilisé

**Erreur :**

```
Port 5000 is already in use
```

**Solution :**

Spécifiez un autre port :

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## FAQ

### Q : Ai-je besoin d&#x27;une licence pour le CLI ?

**R :** Oui ! Le CLI nécessite une licence payante **Chloros+**.

* ❌ Formule Standard (gratuite) : CLI désactivé
* ✅ Formules Chloros+ (payantes) : CLI entièrement activé

Abonnez-vous à : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q : Puis-je utiliser CLI sur un serveur sans interface graphique ?

**R :** Oui ! CLI fonctionne entièrement sans interface graphique. Configuration requise :

* Windows Server 2016 ou version ultérieure
* Visual C++ Redistributable installé
* RAM suffisante (8 Go minimum, 16 Go recommandés)
* Activation unique de la licence GUI sur n&#x27;importe quelle machine

***

### Q : Où sont enregistrées les images traitées ?

**R :** Par défaut, les images traitées sont enregistrées dans le **même dossier que les images d&#x27;entrée**, dans des sous-dossiers correspondant au modèle d&#x27;appareil photo (par exemple, `Survey3N_RGN/`).

Utilisez l&#x27;option `-o` pour spécifier un autre dossier de sortie :

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Q : Puis-je traiter plusieurs dossiers à la fois ?

**R :** Pas directement en une seule commande, mais vous pouvez utiliser des scripts pour traiter les dossiers de manière séquentielle. Consultez la section [Automatisation et scripts](CLI.md#automation--scripting).

***

### Q : Comment enregistrer la sortie CLI dans un fichier journal ?

**PowerShell :**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch :**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Q : Que se passe-t-il si j&#x27;appuie sur Ctrl+C pendant le traitement ?

**R :** CLI va :

1. Arrêter le traitement en douceur
2. Fermer le backend
3. Quitter avec le code 130

Les images partiellement traitées peuvent rester dans le dossier de sortie.

***

### Q : Puis-je automatiser le traitement CLI ?

**R :** Absolument ! Le CLI est conçu pour l&#x27;automatisation. Consultez [Automation &amp; Scripting](CLI.md#automation--scripting) pour des exemples PowerShell, Batch et Python.

***

### Q : Comment vérifier la version CLI ?

**R :**

```powershell
chloros-cli --version
```

**Résultat :**

```
Chloros CLI 1.0.2
```

***

## Obtenir de l&#x27;aide

### Aide en ligne de commande

Affichez les informations d&#x27;aide directement dans CLI :

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canaux d&#x27;assistance

* **E-mail** : info@mapir.camera
* **Site Web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Tarifs** : [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Exemples complets

### Exemple 1 : traitement de base

Traitement avec les paramètres par défaut (vignette, réflectance) :

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Exemple 2 : résultat scientifique de haute qualité

32 bits flottant TIFF :

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Exemple 3 : traitement rapide de l&#x27;aperçu

8 bits PNG sans étalonnage pour un examen rapide :

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Exemple 4 : traitement corrigé PPK

Appliquer les corrections PPK avec réflectance :

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Exemple 5 : emplacement de sortie personnalisé

Traiter vers un autre lecteur avec un format spécifique :

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Exemple 6 : flux de travail d&#x27;authentification

Flux d&#x27;authentification complet :

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

### Exemple 7 : utilisation multilingue

Modifier la langue de l&#x27;interface :

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
