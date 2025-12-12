# Chloros Traduction manuelle avec DeepL API

Ce répertoire contient des scripts permettant de traduire automatiquement le manuel Chloros en plus de 30 langues à l'aide de DeepL API, qui fournit une traduction automatique de qualité professionnelle.

## 🚀 Quick Start

### Conditions préalables
- Python 3.8 ou supérieur
- DeepL API clé (déjà configurée dans le script)
- Dépôts Git clonés pour chaque langue cible

### Installation

1. Installer les dépendances Python :
```bash
pip install -r requirements.txt
```

### Utilisation

#### Option 1 : Traduire toutes les langues (Windows)
```bash
translate-deepl.bat
```

#### Option 2 : Traduire une langue spécifique (Windows)
```bash
translate-deepl.bat es     # Spanish
translate-deepl.bat fr     # French
translate-deepl.bat de     # German
```

#### Option 3 : utiliser directement Python

**Traduire toutes les langues:**
```bash
python translate_with_deepl.py .
```

**Traduire une langue spécifique:**
```bash
python translate_with_deepl.py . --lang es
```

**Traduire plusieurs langues spécifiques:**
```bash
python translate_with_deepl.py . --langs es fr de it
```

**Vérifier l'utilisation de DeepL API:**
```bash
python translate_with_deepl.py . --usage
```

**Liste de toutes les langues supportées:**
```bash
python translate_with_deepl.py . --list
```

## 🌍 Langues supportées

DeepL API prend en charge les langues suivantes de notre liste :

| Code | Langue | DeepL Code |
|------|----------|------------|
| ES | espagnol | ES | pt | portugais (Portugal)
| pt | portugais (Portugal) | PT-PT | pt-BR | portugais (Portugal) | PT-PT | pt-BR | portugais (Portugal)
pt-BR | Portugais (Brésil) | PT-BR | pt-BR | Portugais (Brésil) | PT-BR | pt-BR | Portugais (Brésil) | PT-PT
| pt-BR | portugais (Brésil) | PT-BR | pt-BR | portugais (Brésil) | PT-BR
| français | FR | de | allemand | DE | it | italien | IT
| it | Italien | IT | Ja | Japonais | JA
| JA | Japonais | JA | Japonais | JA | Japonais
| ko | Coréen | KO |
| ZH* - zh-CN - chinois simplifié - ZH* - zh-TW - chinois traditionnel - ZH* - ZH* - ZH* - ZH* - ZH
| zh-TW | chinois (traditionnel) | ZH* | zh-CN | chinois (simplifié) | ZH
zh-CN | Chinois (simplifié) | ZH | zh-TW | Chinois (traditionnel) | ZH* | ru | Russe | RU
| nl | néerlandais | NL | ar | arabe | AR | AR
| ar | Arabe | AR | pl | Polonais | PL
| pl | Polonais | PL | tr | Turc | TR | TR
| tr | turc | TR | id | indonésien | ID
| id | indonésien | ID | bg | bulgare | BG
bg | Bulgare | BG | bg | Bulgare | BG | bg | Bulgare | BG | bg | Bulgare
bg | Bulgare | BG | cs | Tchèque | CS
| DA | danois | DA | bg | bulgare | BG | cs | tchèque | CS | cs | danois | DA
| EL | Grec | EL | EL | EL | EL | EL | EL | EL | EL | EL | EL | EL | EL | EL
| et | Estonien | ET
| fi | finnois | FI
| HU | HU | HU | HU | HU | HU | HU | HU | HU | HU | HU | HU | HU | HU | HU | HU
lv | letton | LV | lt | lituanien | LT | lt
lt | Lituanien | LT | lv | Letton | LV
| ro | Roumain | RO | sk | Slovaque | SK | SK
| sk | slovaque | SK | lt | lituanien | LT | ro | roumain | RO
sl | slovène | SL | sv | suédois | SV
| sv | Suédois | SV
ukrainien | UK | ukrainien | UK | ukrainien | UK
nb | norvégien | NB | anglais | français | anglais

\* Note : DeepL ne prend en charge que le chinois simplifié (ZH). Le chinois traditionnel utilise le même code mais peut nécessiter un post-traitement.

### ⚠️ Langues NON prises en charge par DeepL

Les langues suivantes nécessitent des méthodes de traduction alternatives :
- **hi** (hindi)
- **vi** (vietnamien)
- **th** (thaïlandais)

Pour ces langues, vous devrez :
1. Utiliser Google Translate API (qualité moindre mais couverture plus large)
2. Utiliser des services de traduction manuelle
3. Ne pas les utiliser pour l'instant et ajouter des avis "Coming Soon" (bientôt disponible)

## 🛡️ Caractéristiques de protection

Le script de traduction protège automatiquement :

✅ **Noms de produits** : Chloros, MAPIR
✅ **Termes techniques** : RGB, NDVI, NDRE, toutes les formules d'indexation
✅ **Code Blocks** : blocs ` ```code``` ` et blocs inline `code`
✅ **Liens** : `[text](url)` et `![alt](image.png)`
✅ **Balises HTML** : `<div>`, `<figure>`, etc.  
✅ **Chemins d'accès aux fichiers** : `supported-languages.md`, etc.  
✅ **GitBook Syntaxe** : `{% hint %}`, `{% embed %}`, etc.  
✅ **URLs & Emails** : Toutes les adresses web et les adresses électroniques

## 📊 API Utilisation et limites

### DeepL Niveau gratuit
- **Limite** : 500 000 caractères/mois
- **Coût** : Gratuit

### DeepL Pro Tier
- **Limite** : Basé sur votre plan
- **Coût** : ~5.49$/mois pour 1M de caractères

### Utilisation estimée pour le manuel <!--PLHDR000081
Chaque traduction linguistique utilise environ **50 000 à 100 000 caractères** en fonction de la taille du manuel.

- Traduction complète (30 langues) : ~2-3 millions de caractères
- Avec le niveau gratuit : Peut traduire ~5-10 langues/mois
- Avec la version Pro : Possibilité de traduire toutes les langues à la fois

**Vérifiez votre utilisation actuelle:**
```bash
python translate_with_deepl.py . --usage
```

## 🔧 How It Works

1. **Découverte de fichiers** : Trouve tous les fichiers `.md` dans le répertoire de langues
2. **Protection du contenu** : Remplace le code, les liens et les termes protégés par des caractères de remplacement
3. **Découpage** : Divise les fichiers volumineux en morceaux compatibles API
4. **Traduction** : Envoi à DeepL API avec préservation du formatage
5. **Restauration** : Restaure tous les éléments protégés
6. **Writing** : Enregistre le contenu traduit dans le fichier

## 📁 Structure du répertoire

Le script attend cette structure :

```
current_directory/
├── chloros_manual_gitbook-es/          # Spanish repo
│   ├── README.md
│   ├── SUMMARY.md
│   ├── *.md files...
├── chloros_manual_gitbook-fr/          # French repo
├── chloros_manual_gitbook-de/          # German repo
├── ...
└── translate_with_deepl.py             # This script
```

## 🔍 Fichiers ignorés

Le script saute automatiquement :
- `TRANSLATION-PROJECT-README.md`
- `TRANSLATION-PROJECT-QUICKSTART.md`
- `MANUAL-REPO-CREATION-INSTRUCTIONS.md`
- `language-repos-list.md`
- Tous les fichiers dans les répertoires `.git`
- Tous les fichiers dans les dépôts de langues imbriqués

## ⚙️ Configuration avancée

### Changement du point de terminaison API

Pour les comptes DeepL Pro, mettez à jour la ligne 21 dans `translate_with_deepl.py` :
```python
DEEPL_API_URL = "https://api.deepl.com/v2/translate"  # Pro endpoint
```

### Ajout de termes protégés

Modifiez la liste `PROTECTED_TERMS` dans `translate_with_deepl.py` (ligne 68) :
```python
PROTECTED_TERMS = [
    'Chloros',
    'MAPIR',
    # Add more here...
]
```

### Ajustement de la limitation du taux

Modifier les temps de sommeil dans le script :
- Ligne 399 : entre les morceaux (actuellement 0,5 seconde)
- Ligne 459 : Entre les dépôts (actuellement 2 secondes)

## 🐛 Résolution des problèmes

### "DeepL API quota dépassé"
- Vous avez atteint votre limite mensuelle
- Attendez le mois prochain ou passez à la version Pro
- Vérifiez l'utilisation : `python translate_with_deepl.py . --usage`

### "Dépôt non trouvé"
- S'assurer que les dépôts de langues sont clonés dans le répertoire actuel
- Vérifier le nom du dépôt : doit être `chloros_manual_gitbook-{lang_code}`

### "Aucun fichier markdown trouvé"
- Vérifier que le dépôt de langue contient des fichiers `.md`
- Vérifier que les fichiers ne sont pas ignorés par les filtres

### Problèmes de qualité de la traduction
- Certains termes ont été mal traduits ? Ajoutez-les à `PROTECTED_TERMS`
- Blocs de code traduits ? Vérifier que ``` markers are properly closed
- Links broken? Verify markdown link syntax is correct

## 📝 After Translation

After running translations, you should:

1. **Review the output**: Check a few files for quality
2. **Commit changes**: Git commit the translated files
3. **Push to repos**: Push to each language repo
4. **GitBook sync**: Let GitBook automatically pull the changes
5. **QA check**: Review on GitBook to ensure formatting is correct

### Example Workflow:
```bash
# Traduire l'espagnol
python translate_with_deepl.py . --lang es

# Aller dans le repo espagnol
cd chloros_manual_gitbook-es

# Réviser les changements
git diff

# Commencer et pousser
git add -A
git commit -m "Traduction professionnelle vers l'espagnol en utilisant DeepL API"
git push origin main

# GitBook se synchronisera automatiquement !
```

## 📞 Support

Si vous rencontrez des problèmes :
1. Consultez la section de dépannage ci-dessus
2. Consultez la documentation de DeepL API : https://www.deepl.com/docs-api
3. Contact : info@mapir.camera

## 📜 Licence

Ce script de traduction fait partie du projet de manuel Chloros.
© MAPIR Caméra. Tous droits réservés.

---

**Lancez `translate-deepl.bat` et regardez la magie opérer ! ✨

