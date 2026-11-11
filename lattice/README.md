# Caméras LATTICE

LATTICE est le système de caméras multispectrales modulaires d’MAPIR destiné à l’imagerie agricole et scientifique. Chaque caméra LATTICE est équipée du capteur à obturateur global Sony IMX265 (**3,1 MP, pixels de 3,45 µm**) et se connecte via Ethernet en tant que périphérique**GigE Vision**.

Chloros 1.3.0 permet de contrôler les caméras LATTICE en temps réel — détection, prévisualisation en direct, capture et réseaux multicaméras synchronisés — depuis trois interfaces :

| Interface    | Où                                                          | Plateformes                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Interface graphique        | Onglet **Caméras** dans la barre latérale de Chloros                         | Windows 10/11 x64                                        |
| CLI        | Famille de commandes `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Vous cherchez du matériel ?**Les modules de caméra, les objectifs, les filtres et les bandes, les châssis et les supports de fixation, les câbles, le câblage PoE et de déclenchement sont décrits dans le [**manuel d’utilisation LATTICE**](https://mapir.gitbook.io/lattice-camera). Ce chapitre traite du pilotage des caméras depuis Chloros.

Les captures LATTICE sont des fichiers standard `.tif`/`.tiff`, et Chloros les traite toujours à partir de la capture brute. Consultez la [Référence CLI](../reference/cli-reference.md) et la [Référence SDK](../reference/sdk-reference.md) pour connaître l’ensemble des commandes et de l’interface API.

## Deux configurations de capteurs

| Configuration | Capteur       | Filtre                                | Ce que fournit une caméra                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Couleur Bayer | filtre passe-bande triple                |**Trois bandes calibrées à partir d’une seule exposition**                 |
| **M3M**| Monochrome   | filtre d'interférence à bande étroite unique |**Une bande calibrée** ; combinez plusieurs caméras M3M pour obtenir des indices |

Comme une caméra M3M est monochrome derrière un filtre unique, chaque bande bénéficie de sa propre exposition. Une caméra M3C couvre ses trois bandes avec une seule exposition du capteur.

## Chaînes de modèle et nommage

Chaque caméra stocke son identité dans GenICam sous la forme d’une chaîne de modèle `DeviceUserID` :

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros l'affiche avec le préfixe `LATT-` (par exemple `LATT-M3M-L87-F450`). La même chaîne `LATT-…` est inscrite dans la balise EXIF `Model` de chaque exportation et sert de nom au dossier de sortie de l’appareil photo dans les projets traités.

| Composant | Valeurs                                                   | Signification                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Capteur    | `M3C` / `M3M`                                            | Couleur Bayer / monochrome                                                                          |
| Objectif      | `L41` / `L87`                                            | Ce nombre correspond au **champ de vision horizontal en degrés** : L41 = étroit (41°), L87 = large (87°)    |
| Filtre    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ou `F<nm>` (M3M) | Voir [Filtres et bandes spectrales](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

La chaîne de modèle détermine tout ce qui suit : Chloros détermine le profil du capteur, la disposition des bandes et l'étalonnage d'usine à partir de `DeviceUserID` + `DeviceSerialNumber`. Il n’y a rien à configurer au niveau de chaque caméra — voir [Connexion des caméras](connecting.md).

## Filtres et bandes

Les centres de bande, les largeurs à mi-FWHM et le catalogue complet des 23 références M3M font partie des spécifications produit ; ils figurent donc dans le manuel du matériel : [**Filtres et bandes spectrales**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ce qui importe du point de vue logiciel : le code du filtre dans la chaîne de modèle détermine les produits que Chloros peut générer. Les caméras à filtre RGB (`FRGB`) ne fournissent que des produits débayérisés et de prévisualisation — la radiance et la réflectance par bande n’ont pas de sens pour un capteur à large bande ; Chloros les ignore donc et le précise. Tous les autres filtres fournissent la chaîne complète radiance → réflectance → indice.

## L’étalonnage radiométrique en bref

Chaque caméra LATTICE est étalonnée individuellement en usine selon une chaîne traçable selon le NIST et est livrée avec un certificat propre à chaque caméra. Le contenu de cet étalonnage, la manière dont il est mesuré et la précision que vous pouvez indiquer figurent dans le manuel du matériel : [**Étalonnage radiométrique en usine**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Du point de vue logiciel, ce qui importe, c’est qu’Chloros détermine le bon étalonnage lorsqu’une caméra se connecte et fige les coefficients appliqués dans chaque exportation — voir [Connexion des caméras](connecting.md).

## Dans ce chapitre

* [Connexion des caméras](connecting.md) — détection automatique, boîte de dialogue de connexion de l’interface graphique, équivalents CLI/SDK, et comment l’étalonnage d’usine est déterminé (via le pack intégré à la caméra ou via le cloud) lorsqu’une caméra se connecte.
* [Paramètres des caméras](camera-settings.md) — toutes les commandes de l’onglet « Caméras » : la liste des caméras, le volet des paramètres par caméra et le volet des paramètres de matrice (y compris l’éditeur de ROI et l’alignement).
* [Paramètres et modes de capture](capture.md) — Capture de toutes les caméras, capture unique / continue / à intervalles réguliers, types d’exportation et structure du dossier des captures.
* [Réseaux multicaméras](arrays.md) — l’assistant de connexion du réseau, la sonde du câble de synchronisation, la bande passante, le protocole PTP et la vue en direct du réseau.
* [Caméras mono et indices de végétation](mono-indices.md) — Caméras M3M et flux de travail alignement → empilement → indice.

L'interface de commande complète est disponible dans la [Référence CLI](../reference/cli-reference.md) et la [Référence SDK](../reference/sdk-reference.md).
