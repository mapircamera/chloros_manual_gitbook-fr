---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Téléchargement

Téléchargez la dernière version d'Chloros pour vous lancer dans le traitement d'images multispectrales.

### Configuration requise

#### Windows


| Exigence          | Minimum                                              | Recommandée                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Système d'exploitation** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Processeur**        | Intel Core i5 ou équivalent                          | Intel Core i7 ou supérieur                              |
| **Mémoire (RAM)**     | 8 Go                                                  | 16 Go ou plus                                         |
| **Carte graphique**    | Compatible DirectX 11                                | Carte graphique NVIDIA avec au moins 4 Go de VRAM                            |
| **Espace disque**          | 6 Go d'espace libre                                       | SSD avec au moins 10 Go d'espace libre                            |
| **Affichage**          | 1920x1080                                            | 2560x1440 ou plus                                  |
| **Internet**         | Requis pour l'activation [facultative] de la licence Chloros+ | Requis pour l'activation [facultative] de la licence Chloros+ |

#### Linux amd64 (x86_64)

| Configuration requise       | Minimum                    | Recommandée               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processeur**     | x86_64 (Intel/AMD)        | Intel Core i7 ou supérieur   |
| **Mémoire (RAM)**  | 8 Go                        | 16 Go ou plus              |
| **Carte graphique** | Aucune (traitement par le processeur)      | Carte graphique NVIDIA avec 4 Go ou plus de VRAM |
| **Espace disque**       | 2 Go d’espace libre             | SSD avec 10 Go ou plus d’espace libre       |
| **Python**        | Python 3.7+ (pour le SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Configuration requise      | Minimum                      | Recommandée                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plateforme**     | NVIDIA Jetson avec JetPack 6 | Jetson Orin NX 16 Go ou AGX Orin |
| **Mémoire (RAM)** | 8 Go (partagée GPU/CPU)         | 16 Go ou plus partagée                    |
| **Stockage**      | 2 Go d'espace libre               | SSD NVMe avec 10 Go+ d'espace libre        |
| **Python**       | Python 3.7+ (pour le SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accélération GPU** : les utilisateurs de Chloros+ disposant de GPU NVIDIA peuvent utiliser l’accélération CUDA pour un traitement nettement plus rapide. Cela fonctionne avec les GPU de bureau NVIDIA sous Windows et Linux amd64, ainsi que sur NVIDIA Jetson avec JetPack 6. Les utilisateurs de Chloros+ bénéficient également d’un traitement multithread pour une vitesse maximale.
{% endhint %}

***

## Télécharger Chloros


### Dernière version stable : version 1.3.0 (11 novembre 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Télécharger Chloros pour Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Télécharger Chloros pour Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Télécharger Chloros pour Linux arm64 / Jetson (.deb)</a>


#### Programme d'installation Windows (interface graphique + CLI + backend)

* **Type de fichier** : .exe (programme d'installation Windows)

**Étapes d'installation :**

1. Téléchargez le fichier .exe ci-dessus
2. Double-cliquez sur le programme d'installation pour lancer l'installation
3. Suivez les instructions de l'assistant d'installation
4. Choisissez le répertoire d'installation (par défaut : `C:\Program Files\Chloros\`)
5. Sur la page **Sélectionner des tâches supplémentaires**, laissez les deux options**Paramètres système** cochées (recommandé — voir ci-dessous)
6. Terminez l’installation et lancez Chloros ou Chloros CLI
7. Connectez-vous avec votre [compte MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continuez avec la version gratuite)

**Options des paramètres système** (toutes deux cochées par défaut) :

| Option, telle qu'elle apparaît dans le programme d'installation | Fonction | Désactiver l'installation silencieuse |
| --- | --- | --- |
| **Activer les chemins d’accès longs sous Windows** (permet aux dossiers de projet d’être situés à plus de 260 caractères de profondeur ; recommandé) | Définit le paramètre `LongPathsEnabled` de Windows, applicable à l’ensemble du système, sur 1. Windows est livré avec ce paramètre désactivé, ce qui limite la longueur des chemins d’accès à 260 caractères ; les produits Chloros s’installent jusqu’à environ 130 caractères en dessous du dossier de projet. Ainsi, lorsque les chemins longs sont désactivés, un dossier de projet plus profond est refusé. Cette option prend effet pour les programmes lancés après l’installation. Elle reste activée après une désinstallation, car d’autres programmes peuvent en dépendre. | `/MERGETASKS="!longpaths"` |
| **Exclure le backend Chloros de l’analyse par Microsoft Defender** (capture vidéo nettement plus rapide ; recommandé) | Ajoute une exclusion de processus Microsoft Defender pour `chloros-backend.exe` dans le dossier d’installation, afin que la protection en temps réel n’analyse pas chaque fichier écrit par le backend, quel que soit l’emplacement où il est écrit. À des fréquences de capture élevées, l’analyse représente environ 30 % de la fréquence de capture (77,7 contre 52,7 captures par seconde lors d’une capture continue avec 9 caméras). Aucun autre programme n’est exclu, et l’exclusion est supprimée lors de la désinstallation. Lorsque Defender est absent ou verrouillé par une stratégie, cette étape échoue silencieusement et l’installation se poursuit. | `/MERGETASKS="!defenderexclusion"` |

Une installation silencieuse (`/SILENT` ou `/VERYSILENT`) applique ces deux options, sauf si vous les désactivez à l’aide de la valeur `/MERGETASKS` indiquée.

{% hint style="success" %}
Le programme d’installation ajoute le dossier CLI (`C:\Program Files\Chloros\cli\`) à votre système `PATH` une seule fois — une réinstallation ou une mise à niveau n’ajoute plus d’entrée en double et supprime les doublons ajoutés par les installations précédentes. Ouvrez un nouveau terminal après l’installation pour utiliser `chloros-cli`. Le programme d’installation ajoute également deux règles de pare-feu Windows pour le backend, sur chaque profil réseau : **Backend Chloros (UDP-In)**, pour les flux de caméra, PTP, mDNS et le flux de données DAQ-E, et**Backend Chloros (TCP-In)**, limité au sous-réseau local, pour une mise à jour du micrologiciel DAQ-E, au cours de laquelle le capteur se reconnecte à cet ordinateur pour collecter l’image. Les deux sont supprimées lors de la désinstallation.
{% endhint %}

#### Linux amd64 (paquet .deb — CLI + backend)

* **Type de fichier** : .deb (paquet Debian/Ubuntu)
* **Architecture** : x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (paquet .deb — CLI + backend)

* **Type de fichier** : .deb (JetPack 6)
* **Architecture** : aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Utilisez le nom de fichier que vous avez effectivement téléchargé, et conservez le `./` afin que `apt` installe ce fichier plutôt que de rechercher un paquet par son nom. Effectuez l’installation avec `apt install`, et non avec `dpkg -i` : apt installe également les dépendances du Python SDK, que le paquet recommande plutôt qu'il n'exige, tandis que `dpkg -i` les ignore. Depuis la version 1.3.0, le backend et l’CLI sont installés décompressés sous `/usr/lib/chloros/`, ce qui leur permet de démarrer sans avoir à se décompresser au préalable.

Le paquet Jetson utilise la version CUDA de PyTorch fournie par JetPack 6 ; ainsi, le traitement, y compris le débayeur « Texture Aware », s’exécute sur le GPU du Jetson. Il utilise les bibliothèques CUDA et cuDNN installées par JetPack 6 (`sudo apt install nvidia-jetpack` sur une image qui ne les contient pas).

Consultez la section [Installation sous Linux](linux/linux-installation.md) pour obtenir des instructions de configuration détaillées et le [Guide NVIDIA Jetson](linux/nvidia-jetson-guide.md) pour des conseils spécifiques au Jetson.

#### Python SDK (toutes plateformes)

Chaque programme d’installation intègre un « wheel » `chloros_sdk` correspondant, de sorte que la version du SDK correspond toujours à l’interface graphique, à la CLI ou au backend installé. Sous Windows, le programme d’installation l’installe automatiquement dans votre Python système. Sous Linux, le fichier `.deb` installe le SDK pour le système `python3` lui-même (dans `/usr/lib/python3/dist-packages`), de sorte que `import chloros_sdk` fonctionne immédiatement après l’installation, sans pip ni connexion réseau. Seul un virtualenv, qui n’a pas accès aux paquets du système, nécessite l’installation du fichier wheel fourni à l’intérieur :

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pour les hôtes utilisant uniquement pip (aucun paquet Chloros installé ; le SDK pilote alors un backend sur une autre machine), le SDK est également disponible sur PyPI :

```bash
pip install chloros-sdk
```

Consultez la page [API : Python SDK](api-python-sdk.md) et la [Référence du SDK](reference/sdk-reference.md) pour la documentation.

{% hint style="info" %}
**Utilisateurs Linux** : le paquet `.deb` installe l’interface en ligne de commande (CLI), le backend et le SDK Python. Il n’existe pas d’interface graphique pour Linux — toutes les interactions s’effectuent via l’interface en ligne de commande ou le SDK.
{% endhint %}

***

## Ressources supplémentaires

### SDK Python pour Chloros


Pour les développeurs et les workflows d’automatisation, le Chloros Python SDK pour Chloros est fourni avec chaque programme d’installation : le programme d’installation Windows l’installe dans votre Python, et le Linux `.deb` l’installe pour le système `python3`. Sur une machine ne disposant pas du paquet Chloros (celle qui pilote un backend sur une autre machine), installez-le depuis PyPI :

```bash
pip install chloros-sdk
```

**Documentation** : [API : Python SDK](api-python-sdk.md)

**Prérequis** : un backend Chloros à piloter (le programme d’installation Windows ou le paquet Linux `.deb` en installe un), et un identifiant Chloros+ sur un forfait payant

***

## Contenu de l’installation

### Programme d’installation Windows


* ✅ **Interface graphique Chloros** - Interface graphique complète
* ✅ **CLI Chloros** - Interface en ligne de commande (nécessite une licence Chloros+)
* ✅ **Backend Chloros** - Moteur de traitement
* ✅ **Python SDK** - installé dans votre environnement Python s'il est présent (nécessite une licence Chloros+)
* ✅ **Profils de caméra** - Modèles de caméra MAPIR préconfigurés

### Paquet .deb pour Linux


* ✅ **Chloros CLI** - Interface en ligne de commande (nécessite une licence Chloros+)
* ✅ **Chloros Backend** - Moteur de traitement
* ✅ **Python SDK** - Installé pour le système `python3` (nécessite une licence Chloros+)
* ✅ **Profils de caméra** - Modèles de caméra MAPIR préconfigurés
* ❌ Pas d’interface graphique — Linux est en mode « headless » (sans affichage), uniquement CLI/SDK


### Python SDK (PyPI, pour les machines sans paquet Chloros)

* ✅ **Chloros SDK** - Python API (nécessite une licence Chloros+)

***

## Passer à Chloros+

Débloquez des fonctionnalités avancées grâce à un abonnement Chloros+ :

* 🚀 **Traitement multithread** - Traitez les images en parallèle
* ⚡ **Accélération GPU (CUDA)** - Tirez parti de la puissance des GPU NVIDIA
* 💻 **Accès CLI** - Automatisez vos tâches à l’aide d’outils en ligne de commande
* 🐍 **SDK Python Python** - Accès programmatique à l’API

* 📱 **Plusieurs appareils** - Utilisation sur 2 à 10 appareils ou plus (selon le forfait)
* **🐻 Méthode avancée de débayérisation tenant compte des textures** - une débayérisation de haute qualité tenant compte des contours, combinée à un modèle de débruitage basé sur l'IA/ML qui élimine la quasi-totalité du bruit lié à la débayérisation.
* 🧮 **Formules personnalisées** - Créez des indices multispectraux personnalisés

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Consultez les formules et tarifs d’Chloros+</a></p>

***

## Aide à l'installation

### Dépannage

**L'installation échoue avec le message d'erreur suivant :**

* Assurez-vous de disposer des droits d'administrateur
* Désactivez temporairement votre logiciel antivirus
* Vérifiez que vous remplissez la configuration minimale requise

**L'application ne démarre pas (sous Windows) :**

* Vérifiez que Windows 10/11 (64 bits) est installé
* Mettez à jour les pilotes graphiques
* Consultez l’Observateur d’événements de Windows pour obtenir des détails sur l’erreur
* Contactez l’assistance en fournissant les journaux d’erreurs

**L’interface CLI ne démarre pas (Linux) :**

* Vérifiez que le paquet `.deb` est correctement installé : `dpkg -l | grep chloros`
* Vérifiez les droits d'accès : `sudo chmod +x /usr/bin/chloros-cli`
* Lancez les diagnostics : `chloros-cli selftest`
* Vérifiez s’il manque des bibliothèques : `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (depuis la version 1.3.0, `/usr/lib/chloros/chloros-backend` est un petit script de lancement ; le programme lui-même se trouve dans `/usr/lib/chloros/backend/`)

**Problèmes d'activation de la licence :**

* Assurez-vous que la connexion Internet est active
* Vérifiez vos identifiants sur [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Vérifiez que le pare-feu ne bloque pas Chloros

* Consultez [Chloros+ Connexion](chloros+-login.md) pour obtenir des instructions détaillées

### Obtenir de l'aide

Besoin d'aide pour l'installation ou la configuration ?

* 📧 **E-mail** : info@mapir.camera
* 🌐 **Site web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentation** : [Prise en main](./)
* ❓ **FAQ** : [Foire aux questions](faq.md)

***

## Mises à jour logicielles

Chloros vérifie la présence de mises à jour, vous informe lorsqu'une nouvelle version est disponible et vous redirige vers cette page de téléchargement — vous effectuez la mise à jour en exécutant le nouveau programme d'installation signé. Vos paramètres, vos projets et vos identifiants de connexion sont conservés après les mises à jour. Sous Linux et Jetson, la commande `chloros-cli update` vérifie si une version plus récente est disponible et propose de télécharger et d’installer la version correspondante `.deb` (cette commande est réservée à Linux).

***

## Journal des modifications**Version 1.3.0 (dernière version, 11 novembre 2026)**— consultez la section**Nouveautés de Chloros

1.3.0** sur la page [Prise en main](./) pour obtenir la liste complète.**Version 1.2.0**— voir**Nouveautés de Chloros 1.2.0** sur la page [Mise en route](./).

<details>

<summary>Version 1.0.5</summary>

**Date de sortie : 10 février 2026**

**Nouvelles fonctionnalités*** **Méthode de débayérisation « Texture Aware » \[Chloros+ uniquement] -** « Texture Aware » utilise un débayériseur de haute qualité sensible aux contours, associé à un modèle de débruitage basé sur l’IA/ML qui élimine la quasi-totalité du bruit de débayérisation.
* **Prise en charge des cibles d’étalonnage T4P*** **Traitement GPU plus rapide d’Chloros+, meilleure gestion de la mémoire**

**Corrections de bogues*** Interface utilisateur (GUI) entièrement nouvelle, qui devrait désormais fonctionner sur tous les ordinateurs Windows.

</details>

<details>

<summary>Version 1.0.4</summary>

**Date de sortie : 5 janvier 2026**

**Nouvelles fonctionnalités*** **Bouton de basculement image/métadonnées** : ajout d’un bouton dans l’explorateur de fichiers permettant d’afficher les métadonnées de l’image sélectionnée sous forme de tableau plutôt que dans la grille d’images
* **Curseur de zoom de la grille d’images** : nouveau curseur dans l’interface utilisateur pour ajuster la taille des vignettes (prend également en charge la combinaison CTRL + molette de souris)
* **Boutons d’exportation de la grille d’images** : boutons situés dans la rangée supérieure permettant de basculer les vignettes du format JPG vers des exportations traitées (Cibles, Réflectance, Indice, LUT)
* **Onglet « Carte »** : nouvelle carte 2D interactive affichant les marqueurs de localisation GPS des images
  * Prise en charge de Google Maps et des tuiles cartographiques ESRI (sélection automatique du meilleur service de tuiles en fonction de la disponibilité des niveaux de zoom)
  * Aperçu des vignettes au survol de la souris sur les marqueurs de la carte

**Corrections de bogues*** Amélioration de la prise en charge de l'installation d'Chloros sur des ordinateurs dont la langue n'est pas l'anglais

</details>

<details>

<summary>Version 1.0.3</summary>

**Date de sortie : 20 décembre 2025**

**Nouvelles fonctionnalités*** Lancement initial

**Améliorations*** Lancement initial

**Corrections de bogues*** Lancement initial

**Problèmes connus*** Lancement initial

</details>

***

## Contrat de licence**Logiciel propriétaire** - Copyright (c) 2026 MAPIR Inc.

Toute utilisation, distribution ou modification non autorisée est interdite.

**Version gratuite** : disponible pour un usage personnel et commercial avec des fonctionnalités limitées**Chloros+** : licence par abonnement pour des fonctionnalités avancées et des déploiements commerciaux
