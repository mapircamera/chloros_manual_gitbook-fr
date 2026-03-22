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
| **Carte graphique**    | Compatible DirectX 11                                | GPU NVIDIA avec 4 Go ou plus de VRAM                            |
| **Stockage**          | 6 Go d&#x27;espace libre                                       | SSD avec 10 Go ou plus d&#x27;espace libre                            |
| **Affichage**          | 1920x1080                                            | 2560x1440 ou plus                                  |
| **Internet**         | Requis pour l&#x27;activation de la licence [facultative] Chloros+ | Requis pour l&#x27;activation de la licence [facultative] Chloros+ |

#### Linux amd64 (x86\_64)

| Configuration requise | Minimum                    | Recommandée               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processeur**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou supérieur   |
| **Mémoire (RAM)**  | 8 Go                        | 16 Go ou plus              |
| **Carte graphique** | Aucune (traitement CPU)      | GPU NVIDIA avec 4 Go+ de VRAM |
| **Stockage**       | 2 Go d&#x27;espace libre             | SSD avec 10 Go+ d&#x27;espace libre       |
| **Python**        | Python 3.7+ (pour SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Exigence      | Minimum                      | Recommandé                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plateforme**     | NVIDIA Jetson avec JetPack 6 | Jetson Orin NX 16 Go ou AGX Orin |
| **Mémoire (RAM)** | 8 Go (partagée GPU/CPU)         | 16 Go+ partagée                    |
| **Stockage**      | 2 Go d&#x27;espace libre               | SSD NVMe avec 10 Go+ d&#x27;espace libre        |
| **Python**       | Python 3.7+ (pour SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accélération GPU** : les utilisateurs de Chloros+ équipés de GPU NVIDIA peuvent utiliser l&#x27;accélération CUDA pour un traitement nettement plus rapide. Cela fonctionne à la fois sur Windows (GPU de bureau) et Linux (GPU de bureau et NVIDIA Jetson). Les utilisateurs de Chloros+ bénéficient également d&#x27;un traitement multithread pour une vitesse maximale.
{% endhint %}

***

## Télécharger Chloros

### Dernière version stable (23 mars 2026) : Version 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Télécharger Chloros pour Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Télécharger Chloros pour Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Télécharger Chloros pour Linux arm64 / Jetson (.deb)</a>

#### Programme d&#x27;installation Windows (interface graphique + CLI + backend)

* **Type de fichier** : .exe (programme d&#x27;installation Windows)**Étapes d&#x27;installation :**

1. Téléchargez le fichier .exe ci-dessus
2. Double-cliquez sur le programme d&#x27;installation pour lancer l&#x27;installation
3. Suivez les instructions de l&#x27;assistant d&#x27;installation
4. Choisissez le répertoire d&#x27;installation (par défaut : `C:\Program Files\[USER]\Chloros\`)
5. Terminez l&#x27;installation et lancez Chloros ou Chloros CLI
6. Connectez-vous avec votre [compte MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continuez avec la version gratuite)

{% hint style="success" %}
Le programme d&#x27;installation ajoute automatiquement `chloros-cli` au PATH de votre système pour permettre l&#x27;accès en ligne de commande.
{% endhint %}

#### Linux amd64 (paquet .deb — CLI + backend)

* **Type de fichier** : .deb (paquet Debian/Ubuntu)
* **Architecture** : x86_64 (amd64)

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

Consultez [Installation de Linux](linux/linux-installation.md) pour obtenir des instructions de configuration détaillées et le [Guide NVIDIA Jetson](linux/nvidia-jetson-guide.md) pour des conseils spécifiques à Jetson.

#### Python SDK (Toutes les plateformes)

```bash
pip install chloros-sdk
```

Consultez [API : Python SDK](api-python-sdk.md) pour la documentation.

{% hint style="info" %}
**Utilisateurs de Linux** : le paquet `.deb` installe CLI et le backend. Python et SDK sont installés séparément via pip. Il n&#x27;y a pas d&#x27;interface graphique pour Linux — toutes les interactions se font via CLI ou SDK.
{% endhint %}

***

## Ressources supplémentaires

### Python SDK

Pour les développeurs et les workflows d&#x27;automatisation, installez Chloros Python SDK :

```bash
pip install chloros-sdk
```

**Documentation** : [API : Python SDK](api-python-sdk.md)**Configuration requise** : Chloros doit être installé (programme d&#x27;installation Windows ou package Linux `.deb`), connexion à la licence Chloros+ requise***

## Contenu

### Programme d&#x27;installation Windows

* ✅ **Chloros GUI** - Interface graphique complète
* ✅ **Chloros CLI** - Interface en ligne de commande (nécessite une licence Chloros+)
* ✅ **Chloros Backend** - Moteur de traitement
* ✅ **Profils de caméra** - Modèles de caméra MAPIR préconfigurés

### Paquet .deb Linux

* ✅ **Chloros CLI** - Interface en ligne de commande (nécessite une licence Chloros+)
* ✅ **Backend Chloros** - Moteur de traitement
* ✅ **Profils de caméra** - Modèles de caméra MAPIR préconfigurés
* ❌ Pas d&#x27;interface graphique — Linux est uniquement disponible en mode sans interface graphique (headless) CLI/SDK

### Python SDK (pip, toutes plateformes)

* ✅ **Chloros SDK** - Python API (nécessite une licence Chloros+)***

## Passer à Chloros+

Débloquez des fonctionnalités avancées avec un abonnement Chloros+ :

* 🚀 **Traitement multithread** - Traitez les images en parallèle
* ⚡ **Accélération GPU (CUDA)** - Tirez parti de la puissance des GPU NVIDIA
* 💻 **Accès CLI** - Automatisez avec des outils en ligne de commande
* 🐍 **Python SDK** - Accès programmatique à API
* 📱 **Plusieurs appareils** - Utilisation sur 2 à 10 appareils ou plus (selon le forfait)
* **🐻 Méthode avancée de débayérisation sensible aux textures** - un débayériseur de haute qualité sensible aux contours, combiné à un modèle de débruitage IA/ML qui élimine la quasi-totalité du bruit de débayérisation.
* 🧮 **Formules personnalisées** - Création d&#x27;indices multispectraux personnalisés

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Voir les forfaits et tarifs Chloros+</a></p>***

## Aide à l&#x27;installation

### Dépannage

**L&#x27;installation échoue avec le message d&#x27;erreur suivant :**

* Assurez-vous de disposer des droits d&#x27;administrateur
* Désactivez temporairement votre logiciel antivirus
* Vérifiez que vous répondez à la configuration minimale requise

**L&#x27;application ne démarre pas (Windows) :**

* Vérifiez que Windows 10/11 (64 bits) est installé
* Mettez à jour les pilotes graphiques
* Consultez l&#x27;Observateur d&#x27;événements Windows pour obtenir des détails sur l&#x27;erreur
* Contactez le support technique en joignant les journaux d&#x27;erreurs

**CLI ne démarre pas (Linux) :**

* Vérifiez que le package `.deb` est correctement installé : `dpkg -l | grep chloros`
* Vérifiez les autorisations : `sudo chmod +x /usr/bin/chloros-cli`
* Exécutez les diagnostics : `chloros-cli selftest`
* Vérifiez s&#x27;il manque des bibliothèques : `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problèmes d&#x27;activation de licence :**

* Assurez-vous que la connexion Internet est active
* Vérifiez les identifiants sur [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Vérifiez que le pare-feu ne bloque pas Chloros
* Consultez [Chloros+ Connexion](chloros+-login.md) pour obtenir des instructions détaillées

### Obtenir de l&#x27;aide

Besoin d&#x27;aide pour l&#x27;installation ou la configuration ?

* 📧 **E-mail** : info@mapir.camera
* 🌐 **Site web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentation** : [Guide de démarrage](./)
* ❓ **FAQ** : [Foire aux questions](faq.md)***

## Journal des modifications

<details>

<summary>Version 1.1.0 (Dernière version)</summary>

**Date de sortie : mars 2026**

**Nouvelles fonctionnalités*** **Prise en charge de Linux** — CLI et SDK natifs pour Linux amd64 (x86\_64) et arm64 (NVIDIA Jetson JetPack 6). Installation via les paquets `.deb`.
* **Prise en charge de NVIDIA Jetson** — Traitement optimisé pour les périphériques de bord Jetson Nano, Orin Nano, Orin NX et AGX Orin.
* **Adaptation dynamique du calcul** — Détection automatique du matériel et optimisation de la stratégie de traitement. Chloros s&#x27;adapte à votre matériel, du Jetson Nano à une station de travail multi-GPU.
* **Pipeline de traitement à 4 threads** — Threads simultanés de détection, d&#x27;étalonnage, de traitement et d&#x27;exportation avec allocation dynamique de la mémoire GPU.
* **Nouvelles commandes CLI** — `selftest` (diagnostics système) et `update` (gestion des mises à jour Linux).
* **Nouveaux indicateurs de processus CLI** — `--debayer` (standard/sensible aux textures), `--indices` (spécifier les indices), `--target` (rechercher d&#x27;abord dans le sous-dossier cible pour une détection plus rapide).
* **Nouveaux éléments du menu de l&#x27;interface graphique** — Ajouter des fichiers, Ajouter un dossier et Démarrer/Arrêter le traitement sont désormais accessibles depuis le menu déroulant principal.**Améliorations**

* Détection automatique du backend multiplateforme (chemins Windows et Linux)
* Amélioration de SDK et `get_status()` avec suivi de la progression par thread
* Nouvelles exceptions SDK : `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Gestion thermique et régulation adaptative pour NVIDIA Jetson
* Gestion automatique de la mémoire avec repli OOM vers le traitement GPU en mosaïque

</details>

<details>

<summary>Version 1.0.5</summary>

**Date de sortie : 10 février 2026**

**Nouvelles fonctionnalités*** **Méthode de débayérisation sensible à la texture \[Chloros+ uniquement] -** La méthode sensible à la texture utilise un débayériseur de haute qualité sensible aux contours, combiné à un modèle de débruitage IA/ML qui élimine la quasi-totalité du bruit de débayérisation.
* **Prise en charge des cibles d&#x27;étalonnage T4P*** **Traitement GPU Chloros+ plus rapide, meilleure gestion de la mémoire**

**Corrections de bogues*** Interface utilisateur (GUI) entièrement nouvelle, devrait désormais fonctionner sur tous les ordinateurs Windows.

</details>

<details>

<summary>Version 1.0.4</summary>

**Date de sortie : 5 janvier 2026**

**Nouvelles fonctionnalités*** **Bouton de basculement image/métadonnées** : ajout d&#x27;un bouton dans l&#x27;explorateur de fichiers pour afficher les métadonnées de l&#x27;image sélectionnée sous forme de tableau plutôt que dans la grille d&#x27;images
* **Curseur de zoom de la grille d&#x27;images** : nouveau curseur dans l&#x27;interface utilisateur pour ajuster la taille des vignettes (prend également en charge CTRL + molette de la souris)
* **Boutons d&#x27;exportation de la grille d&#x27;images** : boutons dans la rangée supérieure pour basculer les vignettes du format JPG vers des exportations traitées (Cibles, Réflectance, Indice, LUT)
* **Onglet Carte** : nouvelle carte 2D interactive affichant les marqueurs de localisation GPS des images
  * Prise en charge des tuiles cartographiques Google Maps et ESRI (sélection automatique du meilleur service de tuiles en fonction de la disponibilité au niveau de zoom)
  * Aperçu des vignettes au survol de la souris sur les marqueurs de la carte

**Corrections de bogues*** Amélioration de la prise en charge de l&#x27;installation de Chloros sur les ordinateurs non anglophones

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
