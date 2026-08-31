# Langues prises en charge

Chloros offre une prise en charge complète de l&#x27;interface dans **38 langues à travers le monde**, ce qui le rend accessible aux utilisateurs du monde entier. Vous pouvez changer de langue instantanément aussi bien dans l&#x27;interface graphique de bureau que dans CLI.

Chloros prend en charge les langues suivantes :

| # | Langue | Nom dans la langue d&#x27;origine | Code CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Anglais | English | `en` |
| 2 | 🇪🇸 Espagnol | Español | `es` |
| 3 | 🇵🇹 Portugais | Português | `pt` |
| 4 | 🇫🇷 Français | Français | `fr` |
| 5 | 🇩🇪 Allemand | Deutsch | `de` |
| 6 | 🇮🇹 Italien | Italiano | `it` |
| 7 | 🇯🇵 Japonais | 日本語 | `ja` |
| 8 | 🇰🇷 Coréen | 한국어 | `ko` |
| 9 | 🇨🇳 Chinois (simplifié) | 简体中文 | `zh` |
| 10 | 🇹🇼 Chinois (traditionnel) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Russe | Русский | `ru` |
| 12 | 🇳🇱 Néerlandais | Nederlands | `nl` |
| 13 | 🇸🇦 Arabe | العربية | `ar` |
| 14 | 🇵🇱 Polonais | Polski | `pl` |
| 15 | 🇹🇷 Turc | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonésien | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamien | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thaï | ไทย | `th` |
| 20 | 🇸🇪 Suédois | Svenska | `sv` |
| 21 | 🇩🇰 Danois | Dansk | `da` |
| 22 | 🇳🇴 Norvégien | Norsk | `no` |
| 23 | 🇫🇮 Finnois | Suomi | `fi` |
| 24 | 🇬🇷 Grec | Ελληνικά | `el` |
| 25 | 🇨🇿 Tchèque | Čeština | `cs` |
| 26 | 🇭🇺 Hongrois | Magyar | `hu` |
| 27 | 🇷🇴 Roumain | Română | `ro` |
| 28 | 🇺🇦 Ukrainien | Українська | `uk` |
| 29 | 🇧🇷 Portugais brésilien | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Cantonais | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malais | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovaque | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgare | Български | `bg` |
| 34 | 🇭🇷 Croate | Hrvatski | `hr` |
| 35 | 🇱🇹 Lituanien | Lietuvių | `lt` |
| 36 | 🇱🇻 Letton | Latviešu | `lv` |
| 37 | 🇪🇪 Estonien | Eesti | `et` |
| 38 | 🇸🇮 Slovène | Slovenščina | `sl` |

## Comment changer de langue

### Dans Chloros Desktop

1. Ouvrez les paramètres de l&#x27;application
2. Accédez au menu de sélection de la langue
3. Choisissez votre langue préférée dans la liste
4. L&#x27;interface s&#x27;actualisera instantanément

### Dans Chloros CLI

Utilisez la commande `language` pour afficher ou modifier la langue de l&#x27;interface CLI :

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Pour plus de détails, consultez la [documentation CLI](CLI.md).

## Couverture

Les 38 langues sont entièrement prises en charge dans :

* **Chloros Desktop** - Traduction complète de l’interface graphique
* **Chloros CLI** - Interface en ligne de commande et messages de sortie

Python, SDK, API et sa [documentation de référence](reference/sdk-reference.md) sont fournis en anglais.

La prise en charge multilingue garantit aux utilisateurs du monde entier de pouvoir travailler efficacement dans leur langue maternelle, sans aucune barrière.
