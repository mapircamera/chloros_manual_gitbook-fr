---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Téléchargement

Téléchargez la dernière version de Chloros pour vous lancer dans le traitement d&#x27;images multispectrales.

### Configuration requise

#### Windows

| Exigence          | Minimum                                              | Recommandé                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Système d&#x27;exploitation** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Processeur**        | Intel Core i5 ou équivalent                          | Intel Core i7 ou supérieur                              |
| **Mémoire (RAM)**     | 8 Go                                                  | 16 Go ou plus                                         |
| **Carte graphique**    | Compatible DirectX 11                                | GPU NVIDIA avec au moins 4 Go de VRAM                            |
| **Espace disque**          | 6 Go d&#x27;espace libre                                       | SSD avec au moins 10 Go d&#x27;espace libre                            |
| **Résolution d&#x27;écran**          | 1920 x 1080                                            | 2560 x 1440 ou plus                                  |
| **Internet**         | Requis pour l&#x27;activation de la licence [facultative] Chloros+ | Requis pour l&#x27;activation de la licence [facultative] Chloros+ |

#### Linux amd64 (x86_64)

| Configuration requise | Minimum                    | Recommandée               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processeur**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou supérieur   |
| **Mémoire (RAM)**  | 8 Go                        | 16 Go ou plus              |
| **Carte graphique** | Aucune (traitement par le processeur) | Carte graphique NVIDIA avec au moins 4 Go de VRAM |
| **Espace disque**       | 2 Go d&#x27;espace libre             | SSD avec au moins 10 Go d&#x27;espace libre       |
| **Python**        | Python 3.7+ (pour SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Configuration requise      | Minimum                      | Recommandée                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plateforme**     | NVIDIA Jetson avec JetPack 6 | Jetson Orin NX 16 Go ou AGX Orin |
| **Mémoire (RAM)** | 8 Go (partagée GPU/CPU)         | 16 Go+ partagée                    |
| **Stockage**      | 2 Go d’espace libre               | SSD NVMe avec 10 Go+ d’espace libre        |
| **Python**       | Python 3.7+ (pour SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accélération GPU** : les utilisateurs de Chloros+ équipés de GPU NVIDIA peuvent utiliser l’accélération CUDA pour un traitement nettement plus rapide. Cette fonctionnalité est disponible à la fois sur Windows (cartes graphiques de bureau) et sur Linux (cartes graphiques de bureau et NVIDIA Jetson). Les utilisateurs de Chloros+ bénéficient également d’un traitement multithread pour une vitesse maximale.
{% endhint %}

***

## Télécharger Chloros

### Dernière version stable : version 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Télécharger Chloros pour Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Télécharger Chloros pour Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Télécharger Chloros pour Linux arm64 / Jetson (.deb)</a>

#### Programme d&#x27;installation Windows (interface graphique + CLI + backend)

* **Type de fichier** : .exe (programme d&#x27;installation Windows)**Étapes d’installation :**

1. Téléchargez le fichier .exe ci-dessus
2. Double-cliquez sur le programme d’installation pour lancer l’installation
3. Suivez les instructions de l’assistant d’installation
4. Choisissez le répertoire d’installation (par défaut : `C:\Program Files\MAPIR\Chloros\`)
5. Terminez l&#x27;installation et lancez Chloros ou Chloros CLI
6. Connectez-vous avec votre [compte MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continuez avec la version gratuite)

{% hint style="success" %}
Le programme d&#x27;installation ajoute automatiquement `chloros-cli` au PATH de votre système pour permettre l&#x27;accès en ligne de commande.
{% endhint %}

#### Linux amd64 (paquet .deb — CLI + backend)

* **Type de fichier** : .deb (paquet Debian/Ubuntu)
* **Architecture** : x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (paquet .deb — CLI + backend)

* **Type de fichier** : .deb (JetPack 6)
* **Architecture** : aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Consultez la [page d&#x27;installation de Linux](linux/linux-installation.md) pour obtenir des instructions de configuration détaillées et le [guide NVIDIA Jetson](linux/nvidia-jetson-guide.md) pour des conseils spécifiques à Jetson.

#### Python SDK (Toutes les plateformes)

Chaque programme d&#x27;installation intègre une version correspondante de `chloros_sdk`, de sorte que la version SDK correspond toujours à l&#x27;interface graphique (GUI)/CLI/backend installé(e). Sur Windows, le programme d’installation l’installe automatiquement dans votre système Python ; sur Linux, le programme `.deb` place le fichier « wheel » dans `/usr/lib/chloros/sdk/` et affiche la commande d&#x27;installation :

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pour les hôtes utilisant uniquement pip (sans paquet Chloros installé), le fichier SDK est également disponible sur PyPI :

```bash
pip install chloros-sdk
```

Voir [API : Python SDK](api-python-sdk.md) et la [Référence SDK](reference/sdk-reference.md) pour la documentation.

{% hint style="info" %}
**Utilisateurs de Linux** : le paquet `.deb` installe CLI et le backend. Il n’existe pas d’interface graphique pour Linux — toutes les interactions s’effectuent via CLI ou SDK.
{% endhint %}

***

## Ressources supplémentaires

### Python SDK

Pour les développeurs et les workflows d’automatisation, installez Chloros, Python et SDK :

```bash
pip install chloros-sdk
```

**Documentation** : [API : Python SDK](api-python-sdk.md)**Configuration requise** : Chloros doit être installé (programme d&#x27;installation Windows ou package Linux `.deb`), connexion à la licence Chloros+ requise***

## Contenu du pack

### Programme d’installation Windows

* ✅ **Chloros GUI** - Interface graphique complète
* ✅ **Chloros CLI** - Interface en ligne de commande (nécessite une licence Chloros+)
* ✅ **Backend Chloros** - Moteur de traitement
* ✅ **Profils de caméra** - Modèles de caméra MAPIR préconfigurés

### Paquet .deb Linux

* ✅ **Chloros CLI** - Interface en ligne de commande (nécessite une licence Chloros+)
* ✅ **Backend Chloros** - Moteur de traitement
* ✅ **Profils de caméra** - Modèles de caméra MAPIR préconfigurés
* ❌ Pas d&#x27;interface graphique — Linux est une version sans interface graphique, uniquement compatible avec CLI/SDK

### Python SDK (pip, toutes plateformes)

* ✅ **Chloros SDK** - Python API (nécessite une licence Chloros+)***

## Passez à Chloros+

Accédez à des fonctionnalités avancées grâce à un abonnement Chloros+ :

* 🚀 **Traitement multithread** - Traitez les images en parallèle
* ⚡ **Accélération GPU (CUDA)** - Tirez parti de la puissance des GPU NVIDIA
* 💻 **Accès à CLI** – Automatisez vos tâches à l’aide d’outils en ligne de commande
* 🐍 **Python SDK** - Accès programmatique à API
* 📱 **Plusieurs appareils** - Utilisation sur 2 à 10 appareils ou plus (selon le forfait)
* **🐻 Méthode avancée de débayérisation sensible aux textures** - une débayérisation de haute qualité tenant compte des contours, combinée à un modèle de débruitage basé sur l&#x27;IA/ML qui élimine la quasi-totalité du bruit lié à la débayérisation.
* 🧮 **Formules personnalisées** - Créez des indices multispectraux personnalisés

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Consultez les formules et tarifs Chloros+</a></p>***

## Aide à l&#x27;installation

### Dépannage

**L&#x27;installation échoue avec le message d&#x27;erreur suivant :**

* Assurez-vous de disposer des droits d&#x27;administrateur
* Désactivez temporairement votre logiciel antivirus
* Vérifiez que vous remplissez la configuration minimale requise

**L’application ne démarre pas (Windows) :**

* Vérifiez que Windows 10/11 (64 bits) est installé
* Mettez à jour les pilotes graphiques
* Consultez l’Observateur d’événements Windows pour obtenir des détails sur l’erreur
* Contactez le support technique en joignant les journaux d’erreurs

**CLI ne démarre pas (Linux) :**

* Vérifiez que le package `.deb` est correctement installé : `dpkg -l | grep chloros`
* Vérifiez les droits d&#x27;accès : `sudo chmod +x /usr/bin/chloros-cli`
* Lancez les diagnostics : `chloros-cli selftest`
* Vérifiez s&#x27;il manque des bibliothèques : `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problèmes d&#x27;activation de la licence :**

* Assurez-vous que la connexion Internet est active
* Vérifiez vos identifiants sur [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Vérifiez que le pare-feu ne bloque pas Chloros
* Consultez [Chloros+ Connexion](chloros+-login.md) pour obtenir des instructions détaillées

### Obtenir de l&#x27;aide

Besoin d&#x27;aide pour l&#x27;installation ou la configuration ?

* 📧 **E-mail** : info@mapir.camera
* 🌐 **Site web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentation** : [Prise en main](./)
* ❓ **FAQ** : [Foire aux questions](faq.md)***

## Mises à jour logicielles

Chloros recherche les mises à jour, vous informe lorsqu&#x27;une nouvelle version est disponible et vous redirige vers cette page de téléchargement — vous effectuez la mise à jour en exécutant le nouveau programme d&#x27;installation signé. Vos paramètres et vos projets sont conservés après les mises à jour. Sur Linux et Jetson, `chloros-cli update` vérifie si une version plus récente est disponible et vous propose de télécharger et d’installer la version correspondante `.deb` (cette commande est réservée à la version Linux).

***

## Journal des modifications**Version 1.2.0 (dernière version)**— consultez la section**Nouveautés de Chloros 1.2.0** sur la page [Mise en route](./) pour obtenir la liste complète des fonctionnalités.

<details>

<summary>Version 1.0.5</summary>

**Date de sortie : 10 février 2026**

**Nouvelles fonctionnalités*** **Méthode de débayérisation « Texture Aware » \[Chloros+ uniquement] —** « Texture Aware » utilise une débayérisation de haute qualité tenant compte des contours, combinée à un modèle de débruitage basé sur l’IA/ML qui élimine la quasi-totalité du bruit de débayérisation.
* **Prise en charge des cibles d’étalonnage T4P*** **Traitement GPU Chloros+ plus rapide, meilleure gestion de la mémoire**

**Corrections de bogues*** Interface utilisateur (GUI) entièrement nouvelle, qui devrait désormais fonctionner sur tous les ordinateurs Windows.

</details>

<details>

<summary>Version 1.0.4</summary>

**Date de sortie : 5 janvier 2026**

**Nouvelles fonctionnalités*** **Bouton de basculement image/métadonnées** : ajout d’un bouton dans l’explorateur de fichiers permettant d’afficher les métadonnées de l’image sélectionnée sous forme de tableau plutôt que dans la grille d’images
* **Curseur de zoom de la grille d’images** : nouveau curseur dans l’interface utilisateur pour ajuster la taille des vignettes (prend également en charge CTRL + molette de souris)
* **Boutons d’exportation de la grille d’images** : boutons situés dans la rangée supérieure permettant de basculer les vignettes du format JPG vers des exportations traitées (Cibles, Réflectance, Index, LUT)
* **Onglet « Carte »** : nouvelle carte 2D interactive affichant les marqueurs de localisation GPS des images
  * Prise en charge de Google Maps et des tuiles cartographiques ESRI (sélection automatique du meilleur service de tuiles en fonction de la disponibilité des niveaux de zoom)
  * Aperçu des vignettes au survol de la souris sur les marqueurs de la carte

**Corrections de bogues*** Amélioration de la prise en charge de l’installation de Chloros sur des ordinateurs dont la langue n’est pas l’anglais

</details>

<details>

<summary>Version 1.0.3</summary>

**Date de sortie : 20 décembre 2025**

**Nouvelles fonctionnalités*** Lancement initial

**Améliorations*** Lancement initial

**Corrections de bogues*** Lancement initial

**Problèmes connus*** Lancement initial

</details>***

## Contrat de licence**Logiciel propriétaire** - Copyright (c) 2026 MAPIR Inc.

Toute utilisation, distribution ou modification non autorisée est interdite.

**Version gratuite** : Disponible pour un usage personnel et commercial avec des fonctionnalités limitées**Chloros+** : Licence par abonnement pour des fonctionnalités avancées et des déploiements commerciaux
