# Chloros Traduction manuelle avec DeepL API

Ce répertoire contient des scripts permettant de traduire automatiquement le manuel Chloros dans plus de 30 langues à l&#x27;aide de DeepL API, qui fournit une traduction automatique de qualité professionnelle.

## 🚀 Démarrage rapide

### Conditions préalables
- Python 3.8 ou supérieur
- Clé DeepL API (déjà configurée dans le script)
- Référentiels Git clonés pour chaque langue cible

### Installation

1. Installez les dépendances Python :
```bash
pip install -r requirements.txt
```

### Utilisation

#### Option 1 : traduire toutes les langues (Windows)
```bash
translate-deepl.bat
```

#### Option 2 : traduire une langue spécifique (Windows)
```bash
translate-deepl.bat es     # Spanish
translate-deepl.bat fr     # French
translate-deepl.bat de     # German
```

#### Option 3 : utiliser directement Python

**Traduire toutes les langues :**
```bash
python translate_with_deepl.py .
```

**Traduire une langue spécifique :**
```bash
python translate_with_deepl.py . --lang es
```

**Traduire plusieurs langues spécifiques :**
```bash
python translate_with_deepl.py . --langs es fr de it
```

**Vérifier l&#x27;utilisation de DeepL API :**
```bash
python translate_with_deepl.py . --usage
```

**Lister toutes les langues prises en charge :**
```bash
python translate_with_deepl.py . --list
```

## 🌍 Langues prises en charge

DeepL API prend en charge les langues suivantes de notre liste :

| Code | Langue | Code DeepL |
|------|----------|------------|
| es | Espagnol | ES |
| pt | Portugais (Portugal) | PT-PT |
| pt-BR | Portugais (Brésil) | PT-BR |
| fr | Français | FR |
| de | Allemand | DE |
| it | Italien | IT |
| ja | Japonais | JA |
| ko | Coréen | KO |
| zh-CN | Chinois (simplifié) | ZH |
| zh-TW | Chinois (traditionnel) | ZH* |
| ru | Russe | RU |
| nl | Néerlandais | NL |
| ar | Arabe | AR |
| pl | Polonais | PL |
| tr | Turc | TR |
| id | Indonésien | ID |
| bg | Bulgare | BG |
| cs | Tchèque | CS |
| da | Danois | DA |
| el | Grec | EL |
| et | Estonien | ET |
| fi | Finnois | FI |
| hu | Hongrois | HU |
| lv | Letton | LV |
| lt | Lituanien | LT |
| ro | Roumain | RO |
| sk | Slovaque | SK |
| sl | Slovène | SL |
| sv | Suédois | SV |
| uk | Ukrainien | UK |
| nb | Norvégien | NB |

\* Remarque : DeepL ne prend en charge que le chinois simplifié (ZH). Le chinois traditionnel utilise le même code, mais peut nécessiter un post-traitement.

### ⚠️ Langues NON prises en charge par DeepL

Les langues suivantes nécessitent d&#x27;autres méthodes de traduction :
- **hi** (hindi)
- **vi** (vietnamien)  
- **th** (thaï)

Pour ces langues, vous devrez :
1. Utiliser Google Translate API (qualité inférieure mais couverture plus large)
2. Utiliser des services de traduction manuelle
3. Les ignorer pour l&#x27;instant et ajouter des mentions « Bientôt disponible »

## 🛡️ Fonctions de protection

Le script de traduction protège automatiquement :

✅ **Noms de produits** : Chloros, MAPIR  
✅ **Termes techniques** : RGB, NDVI, NDRE, toutes les formules d&#x27;indice  
✅ **Blocs de code** : blocs ` ```code``` ` et `code` en ligne  
✅ **Liens** : `[text](url)` et `![alt](image.png)`  
✅ **Balises HTML** : `<div>`, `<figure>`, etc.  
✅ **Chemins d&#x27;accès aux fichiers** : `supported-languages.md`, etc.  
✅ **Syntaxe GitBook** : `{% hint %}`, `{% embed %}`, etc.  
✅ **URL et e-mails** : toutes les adresses Web et adresses e-mail  

## 📊 Utilisation et limites de API

### Offre gratuite DeepL
- **Limite** : 500 000 caractères/mois
- **Coût** : gratuit

### Offre DeepL Pro
- **Limite** : en fonction de votre forfait
- **Coût** : environ 5,49 $/mois pour 1 million de caractères

### Estimation de l&#x27;utilisation pour Chloros Manuel
Chaque traduction linguistique utilise environ **50 000 à 100 000 caractères**, selon la taille du manuel.

- Traduction complète (30 langues) : environ 2 à 3 millions de caractères
- Avec le niveau gratuit : possibilité de traduire environ 5 à 10 langues par mois
- Avec le niveau Pro : possibilité de traduire toutes les langues en une seule fois

**Vérifiez votre utilisation actuelle :**
```bash
python translate_with_deepl.py . --usage
```

## 🔧 Comment ça marche

1. **Recherche de fichiers** : recherche tous les fichiers `.md` dans le référentiel de langues
2. **Protection du contenu** : remplace le code, les liens et les termes protégés par des espaces réservés
3. **Découpage** : divise les fichiers volumineux en morceaux compatibles avec API
4. **Traduction** : envoie à DeepL API en conservant la mise en forme
5. **Restauration** : restaure tous les éléments protégés
6. **Écriture** : enregistre le contenu traduit dans le fichier

## 📁 Structure du répertoire

Le script attend la structure suivante :

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

Le script ignore automatiquement :
- `TRANSLATION-PROJECT-README.md`
- `TRANSLATION-PROJECT-QUICKSTART.md`
- `MANUAL-REPO-CREATION-INSTRUCTIONS.md`
- `language-repos-list.md`
- Tous les fichiers dans les répertoires `.git`
- Tous les fichiers dans les répertoires de langues imbriqués

## ⚙️ Configuration avancée

### Modification du point de terminaison API

Pour les comptes DeepL Pro, mettez à jour la ligne 21 dans `translate_with_deepl.py` :
```python
DEEPL_API_URL = "https://api.deepl.com/v2/translate"  # Pro endpoint
```

### Ajout de termes protégés supplémentaires

Modifiez la liste `PROTECTED_TERMS` dans `translate_with_deepl.py` (ligne 68) :
```python
PROTECTED_TERMS = [
    'Chloros',
    'MAPIR',
    # Add more here...
]
```

### Ajuster la limitation du débit

Modifiez les temps de veille dans le script :
- Ligne 399 : entre les blocs (actuellement 0,5 seconde)
- Ligne 459 : entre les dépôts (actuellement 2 secondes)

## 🐛 Dépannage

### « Quota DeepL API dépassé »
- Vous avez atteint votre limite mensuelle.
- Attendez le mois prochain ou passez à la version Pro.
- Vérifiez votre utilisation : `python translate_with_deepl.py . --usage`

### « Référentiel introuvable »
- Assurez-vous que les référentiels de langue sont clonés dans le répertoire actuel
- Vérifiez le nom du référentiel : il doit être `chloros_manual_gitbook-{lang_code}`

### « Aucun fichier Markdown trouvé »
- Vérifiez que le référentiel de langue contient des fichiers `.md`
- Vérifiez que les fichiers ne sont pas ignorés par les filtres

### Problèmes de qualité de traduction
- Certains termes sont-ils mal traduits ? Ajoutez-les à `PROTECTED_TERMS`.
- Les blocs de code sont-ils traduits ? Vérifiez que ``` markers are properly closed
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
# Traduire en espagnol
python translate_with_deepl.py . --lang es

# Accédez au dépôt espagnol
cd chloros_manual_gitbook-es

# Vérifiez les modifications
git diff

# Validez et envoyez
git add -A
git commit -m « Traduction professionnelle vers l&#x27;espagnol à l&#x27;aide de DeepL API »
git push origin main

# GitBook se synchronisera automatiquement !
```

## 📞 Assistance

Si vous rencontrez des problèmes :
1. Consultez la section Dépannage ci-dessus.
2. Consultez la documentation DeepL API : https://www.deepl.com/docs-api.
3. Contactez : info@mapir.camera.

## 📜 Licence

Ce script de traduction fait partie du projet de manuel Chloros.
© MAPIR Camera. Tous droits réservés.

---

**Prêt à traduire ?** Lancez `translate-deepl.bat` et regardez la magie opérer ! ✨

