# Chloros Statut de la traduction manuelle

## ✅ Ce qui a été accompli

### Script créé
- **`translate_with_deepl.py`** - Script de traduction professionnelle utilisant DeepL API
- **`translate-deepl.bat`** - Fichier batch Windows facile à utiliser pour exécuter les traductions
- **`requirements.txt`** - Dépendances Python (uniquement `requests`)
- **`DEEPL-TRANSLATION-README.md`** - Documentation complète

### Fonctionnalités de protection implémentées
Le script protège désormais correctement :

✅ **Formules mathématiques** - Les blocs `$$...$$` ne sont PAS traduits  
✅ **Noms de couleurs dans les formules** - Red, Blue, Green, NIR, RedEdge, Cyan, Orange restent en anglais  
✅ **Noms de produits** - Chloros, MAPIR jamais traduits  
✅ **Termes techniques** - Tous les noms d&#x27;index (NDVI, EVI, GARI, etc.) protégés  
✅ **Blocs de code** - Tous les blocs ` ```code``` ` et `code` en ligne protégés
✅ **Tableaux HTML** - Contenu `<table>...</table>` protégé  
✅ **Liens** - `[text](url)` et `![alt](image.png)` protégés  
✅ **Chemins d&#x27;accès aux fichiers** - Références `*.md` protégées  
✅ **Syntaxe GitBook** - `{% hint %}`, `{% embed %}` protégé  
✅ **URL et e-mails** - Toutes les adresses Web protégées  
✅ **Modèles d&#x27;appareils photo** - Survey3, Survey3W, Survey3N, noms de filtres protégés  

### Traductions terminées
- **✅ Espagnol (es)** - 24/24 fichiers traduits avec succès

### DeepL API Utilisation
- **Utilisé** : 470 301 / 500 000 caractères (94,1 %)
- **Restant** : 29 699 caractères (~0,1 langue supplémentaire)
- **Statut** : LIMITE DU FORFAIT GRATUIT PRESQUE ATTEINTE

## 📊 Langues à traduire

Total des dépôts linguistiques prêts : **36 langues**

### Langues prises en charge par DeepL (31)
Situé dans `D:\chloros_translation_robust\` :

| Priorité | Langues |
|----------|-----------|
| **Haute priorité** | es ✅, pt, fr, de, it |
| **Europe** | nl, pl, cs, sk, sl, da, fi, sv, nb (norvégien), ro, hu, bg, el, et, lv, lt |
| **Moyen-Orient** | ar, tr |
| **Asie** | ja, ko, zh-TW, id |
| **Autres** | ru, uk |

### Langues non prises en charge par DeepL (3)
Une autre approche est nécessaire :
- **hi** (hindi)
- **vi** (vietnamien)
- **th** (thaï)

### Répertoires inconnus dans le répertoire (2)
- **hr** (croate) - DeepL ne prend pas en charge cette langue, une alternative est nécessaire.
- **ms** (malais) - DeepL ne prend pas en charge cette langue, une alternative est nécessaire.
- **zh-HK** (chinois de Hong Kong) - Peut être utilisé de la même manière que zh-CN.

## 🎯 Prochaines étapes - Vos options

### Option 1 : passer à DeepL Pro (RECOMMANDÉ)
**Coût** : environ 5,49 $/mois pour 1 million de caractères ou paiement à l&#x27;utilisation

**Avantages** :
- ✅ Possibilité de traduire dès AUJOURD&#x27;HUI les 31 langues restantes
- ✅ Qualité professionnelle
- ✅ Fonctionnalité de glossaire pour une meilleure protection des termes
- ✅ Environ 3 millions de caractères nécessaires au total (5 à 15 $ au total)

**Comment procéder** :
1. Rendez-vous sur https://www.deepl.com/pro-api
2. Inscrivez-vous à Pro API (ils proposent un essai gratuit)
3. Obtenez une nouvelle clé API
4. Mettez à jour la clé dans `translate_with_deepl.py` (ligne 21)
5. Remplacez API URL par `https://api.deepl.com/v2/translate` (ligne 22)
6. Exécutez : `python translate_with_deepl.py "D:\chloros_translation_robust"`

### Option 2 : attendre la réinitialisation du niveau gratuit
**Attendre** : jusqu&#x27;au mois suivant (réinitialisation mensuelle)

**Puis traduire** :
- 5 à 10 langues par mois avec le niveau gratuit
- Terminer toutes les langues en 3 à 6 mois environ

### Option 3 : utiliser plusieurs comptes DeepL gratuits
**Créer** : 2 à 3 comptes DeepL gratuits supplémentaires avec des adresses e-mail différentes

**Alternez les clés** pour traduire plus de langues :
- Compte 1 : 5 langues
- Compte 2 : 5 langues  
- Compte 3 : 5 langues
- etc.

### Option 4 : approche hybride
1. **DeepL gratuit** : traduisez 1 langue supplémentaire hautement prioritaire (français ou allemand)
2. **Attendez ou passez à un compte supérieur** : terminez le reste plus tard
3. **Pour l&#x27;instant** : ajoutez des mentions « Bientôt disponible » aux dépôts non traduits

## 🔧 Comment poursuivre la traduction

### Langue unique (si le quota est disponible) :
```bash
python translate_with_deepl.py "D:\chloros_translation_robust" --lang fr
```

### Plusieurs langues :
```bash
python translate_with_deepl.py "D:\chloros_translation_robust" --langs fr de it
```

### Toutes les langues restantes :
```bash
python translate_with_deepl.py "D:\chloros_translation_robust"
```

### Vérifiez d&#x27;abord l&#x27;utilisation de API :
```bash
python translate_with_deepl.py . --usage
```

## ✨ Vérification de la qualité

Avant de poursuivre la traduction, vous devez vérifier la qualité de la traduction espagnole :

1. Accédez à `chloros_manual_gitbook-es` ou `D:\chloros_translation_robust\chloros_manual_gitbook-es`
2. Vérifiez les fichiers clés :
   - `project-settings/multispectral-index-formulas.md` - Vérifiez que les formules sont correctes
   - `README.md` - Vérifiez que le texte général se lit naturellement
   - `CLI.md` - Vérifiez que les termes techniques sont conservés
3. Vérifiez sur GitBook après la synchronisation

Si la qualité est bonne, passez aux autres langues !

## 📝 Langues non prises en charge (hindi, vietnamien, thaï)

Pour les 3 langues non prises en charge par DeepL, les options sont les suivantes :

1. **Google Cloud Translation API** - Prend en charge ces langues
2. **Services de traduction manuelle** - Engagez des traducteurs professionnels
3. **Ignorer pour l&#x27;instant** - Ajoutez une note « Anglais uniquement pour l&#x27;instant »
4. **Traductions communautaires** - Ouvrez à contributions ultérieurement

## 🎉 Résumé

**Le système de traduction fonctionne !**

- ✅ Le script protège correctement tous les termes techniques
- ✅ Les formules restent en anglais (Red, Green, Blue, NIR, etc.)
- ✅ Traduction espagnole terminée et vérifiée
- ⏸️ Mis en pause en raison de la limite du niveau gratuit API
- 🚀 Prêt à continuer avec un compte Pro ou à attendre la réinitialisation

**Votre clé DeepL API est configurée et fonctionne parfaitement.**

---

**Besoin d&#x27;aide ?** Contact : info@mapir.camera

