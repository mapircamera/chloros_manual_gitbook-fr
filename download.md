---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Téléchargement

Téléchargez la dernière version de Chloros pour commencer à utiliser le traitement d&#x27;images multispectrales.

### Configuration système requise

| Configuration requise          | Minimum                                              | Recommandée                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Système d&#x27;exploitation** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Processeur**        | Intel Core i5 ou équivalent                          | Intel Core i7 ou supérieur                              |
| **Mémoire (RAM)**     | 8 Go                                                  | 16 Go ou plus                                         |
| **Carte graphique**    | Compatible DirectX 11                                | GPU NVIDIA avec 4 Go+ de VRAM                            |
| **Stockage**          | 6 Go d&#x27;espace libre                                       | SSD avec 10 Go+ d&#x27;espace libre                            |
| **Affichage**          | 1920 x 1080                                            | 2560 x 1440 ou supérieur                                  |
| **Internet**         | Requis pour l&#x27;activation de la licence [facultative] Chloros+ | Requis pour l&#x27;activation de la licence [facultative] Chloros+ |

{% hint style="info" %}
**Accélération GPU** : les utilisateurs de Chloros+ équipés de GPU NVIDIA peuvent utiliser l&#x27;accélération CUDA pour un traitement nettement plus rapide. Les utilisateurs de Chloros+ bénéficient également d&#x27;un traitement multithread pour une vitesse maximale.
{% endhint %}

***

## Télécharger Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Télécharger Chloros ici</a>

### Dernière version stable

**Chloros Programme d&#x27;installation pour Windows*** **Version** : 1.0.5
* **Date de sortie** : 10 février 2026
* **Taille du fichier (téléchargement)** : 1,6 Go
* **Taille du fichier (installé)** : 5,7 Go
* **Type de fichier** : .exe (programme d&#x27;installation Windows)

#### **Étapes d&#x27;installation :**

1. Téléchargez le fichier `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Double-cliquez sur le programme d&#x27;installation pour lancer l&#x27;installation
3. Suivez les instructions de l&#x27;assistant d&#x27;installation
4. Choisissez le répertoire d&#x27;installation (par défaut : `C:\Program Files\[USER]\Chloros\`)
5. Terminez l&#x27;installation et lancez Chloros ou Chloros CLI
6. Connectez-vous avec votre [compte MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continuez avec la version gratuite).

{% hint style="success" %}
Le programme d&#x27;installation ajoute automatiquement `chloros-cli` au chemin d&#x27;accès PATH de votre système pour permettre l&#x27;accès en ligne de commande.
{% endhint %}

***

## Ressources supplémentaires

### Python SDK

Pour les développeurs et les workflows d&#x27;automatisation, installez Chloros Python SDK :

```bash
pip install chloros-sdk
```

**Documentation** : [API : Python SDK](api-python-sdk.md)**Configuration requise** : Chloros Desktop doit être installé, connexion à la licence Chloros+ requise.***

## Contenu

L&#x27;installation de Chloros comprend :

* ✅ **Chloros** - Interface graphique complète (GUI)
* ✅ **Chloros CLI** - Interface en ligne de commande (nécessite une licence Chloros+)
* ✅ **Chloros SDK** - Python API (nécessite une licence Chloros+)
* ✅ **Profils de caméra** - Modèles de caméra MAPIR préconfigurés***

## Mise à niveau vers Chloros+

Débloquez des fonctionnalités avancées avec un abonnement Chloros+ :

* 🚀 **Traitement multithread** - Traitez les images en parallèle
* ⚡ **Accélération GPU (CUDA)** - Tirez parti de la puissance des GPU NVIDIA
* 💻 **Accès CLI** - Automatisez avec des outils en ligne de commande
* 🐍 **Python SDK** - Accès programmatique à API
* 📱 **Plusieurs appareils** - Utilisation sur 2 à 10 appareils ou plus (selon le forfait)
* **🐻 Méthode avancée de débayérisation sensible à la texture** - une débayérisation de haute qualité sensible aux contours, combinée à un modèle de débruitage IA/ML qui élimine la quasi-totalité du bruit de débayérisation. 
* 🧮 **Formules personnalisées** - Créez des indices multispectraux personnalisés

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Voir les forfaits et tarifs Chloros+</a></p>***

## Aide à l&#x27;installation

### Dépannage

**L&#x27;installation échoue avec le message d&#x27;erreur suivant :**

* Assurez-vous que vous disposez des droits d&#x27;administrateur
* Désactivez temporairement votre logiciel antivirus
* Vérifiez que vous disposez de la configuration minimale requise

**L&#x27;application ne démarre pas :**

* Vérifiez que Windows 10/11 (64 bits) est installé
* Mettez à jour les pilotes graphiques
* Vérifiez l&#x27;observateur d&#x27;événements Windows pour obtenir des détails sur l&#x27;erreur
* Contactez l&#x27;assistance avec les journaux d&#x27;erreurs

**Problèmes d&#x27;activation de la licence :**

* Assurez-vous que votre connexion Internet est active.
* Vérifiez vos identifiants sur [https://cloud.mapir.camera](https://cloud.mapir.camera).
* Vérifiez que le pare-feu ne bloque pas Chloros.
* Consultez [Chloros+ Connexion](chloros+-login.md) pour obtenir des instructions détaillées

### Obtenir de l&#x27;aide

Besoin d&#x27;aide pour l&#x27;installation ou la configuration ?

* 📧 **E-mail** : info@mapir.camera
* 🌐 **Site web** : [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentation** : [Pour commencer](./)
* ❓ **FAQ** : [Foire aux questions](faq.md)***

## Journal des modifications

<details>

<summary>Version 1.0.5</summary>

#### **Date de sortie** : 10 février 2026**Nouvelles fonctionnalités*** **Méthode de débayérisation sensible à la texture \[Chloros+ uniquement] -** La fonction « Texture Aware » utilise une débayérisation sensible aux contours de haute qualité combinée à un modèle de débruitage IA/ML qui élimine la quasi-totalité du bruit de débayérisation.
* **Prise en charge des cibles d&#x27;étalonnage T4P*** **Traitement GPU Chloros+ plus rapide, meilleure gestion de la mémoire**

**Corrections de bogues*** Interface utilisateur (GUI) entièrement nouvelle, devrait désormais fonctionner sur tous les ordinateurs Windows.

</details>

<details>

<summary>Version 1.0.4</summary>

#### **Date de sortie** : 5 janvier 2026**Nouvelles fonctionnalités*** **Bouton Image/Métadonnées** : ajout d&#x27;un bouton dans le navigateur de fichiers pour afficher les métadonnées de l&#x27;image sélectionnée dans un tableau plutôt que dans la grille d&#x27;images
* **Curseur de zoom de la grille d&#x27;images** : nouveau curseur dans l&#x27;interface utilisateur pour ajuster la taille des vignettes (prend également en charge CTRL + molette de la souris)
* **Boutons d&#x27;exportation de la grille d&#x27;images** : boutons dans la rangée supérieure pour passer des vignettes JPG aux exportations traitées (cibles, réflectance, index, LUT)
* **Onglet Carte** : nouvelle carte 2D interactive affichant les marqueurs de localisation GPS des images.
  * Prend en charge les tuiles cartographiques Google Maps et ESRI (sélectionne automatiquement le meilleur service de tuiles en fonction de la disponibilité du niveau de zoom).
  * Aperçu des vignettes au survol de la souris sur les marqueurs cartographiques.

**Corrections de bogues*** Amélioration de la prise en charge de l&#x27;installation de Chloros sur les ordinateurs non anglophones.

</details>

<details>

<summary>Version 1.0.3</summary>

#### **Date de sortie** : 20 décembre 2025**Nouvelles fonctionnalités*** Lancement initial

**Améliorations*** Lancement initial

**Corrections de bogues*** Lancement initial

**Problèmes connus*** Lancement initial

</details>***

## Contrat de licence**Logiciel propriétaire** - Copyright (c) 2026 MAPIR Inc.

Toute utilisation, distribution ou modification non autorisée est interdite.

**Version gratuite** : disponible pour un usage personnel et commercial avec des fonctionnalités limitées.**Chloros+** : licence par abonnement pour des fonctionnalités avancées et des déploiements commerciaux.
