# Caméras LATTICE

LATTICE est le système de caméras multispectrales modulaires de MAPIR destiné à l&#x27;imagerie agricole et scientifique. Chaque caméra LATTICE est équipée du capteur à obturateur global Sony IMX265 (**3,1 MP, pixels de 3,45 µm**) et se connecte via Ethernet en tant que périphérique**GigE Vision**.

Chloros 1.2.0 contrôle les caméras LATTICE en temps réel — détection, prévisualisation en direct, capture et réseaux synchronisés de plusieurs caméras — depuis trois interfaces :

| Interface    | Où                                                          | Plateformes                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Interface graphique        | Onglet **Caméras** dans la barre latérale de Chloros                         | Windows 10/11 x64                                        |
| CLI        | Famille de commandes `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |

> **Vous recherchez le matériel ?**Les modules de caméra, les objectifs, les filtres et les bandes, les cadres et les supports de fixation, les câbles, le câblage PoE et de déclenchement sont décrits dans le [**manuel d&#x27;utilisation LATTICE**](https://mapir.gitbook.io/lattice-camera). Ce chapitre traite du pilotage des caméras à partir de Chloros.

Les captures LATTICE sont des fichiers standard `.tif`/`.tiff`, et Chloros les traite toujours à partir de la capture brute. Consultez la [Référence CLI](../reference/cli-reference.md) et la [Référence SDK](../reference/sdk-reference.md) pour connaître la liste complète des commandes et l’interface API.

## Deux configurations de capteurs

| Configuration | Capteur       | Filtre                                | Ce que fournit une caméra                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Couleur Bayer | filtre passe-bande triple                |**Trois bandes étalonnées à partir d’une seule exposition**                 |
| **M3M**| Monochrome   | filtre d’interférence à bande étroite unique |**Une bande étalonnée** ; combiner plusieurs caméras M3M pour obtenir des indices |

Comme une caméra M3M est monochrome derrière un filtre unique, chaque bande bénéficie de sa propre exposition. Une caméra M3C couvre l’ensemble de ses trois bandes en une seule exposition du capteur.

## Chaînes de modèle et nommage

Chaque caméra stocke son identité dans GenICam `DeviceUserID` sous la forme d’une chaîne de modèle :

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros l&#x27;affiche avec le préfixe `LATT-` (par exemple `LATT-M3M-L87-F450`). Cette même chaîne `LATT-…` est inscrite dans la balise EXIF `Model` de chaque exportation et sert de nom au dossier de sortie de la caméra dans les projets traités.

| Composant | Valeurs                                                   | Signification                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Capteur    | `M3C` / `M3M`                                            | Couleur Bayer / monochrome                                                                          |
| Objectif      | `L41` / `L87`                                            | Le nombre correspond au **champ de vision horizontal en degrés** : L41 = étroit (41°), L87 = large (87°)    |
| Filtre    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ou `F<nm>` (M3M) | Voir [Filtres et bandes spectrales](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

La chaîne de modèle détermine tous les paramètres en aval : Chloros détermine le profil du capteur, la disposition des bandes et l&#x27;étalonnage d&#x27;usine à partir de `DeviceUserID` + `DeviceSerialNumber`. Il n&#x27;y a rien à configurer au niveau de chaque caméra — voir [Connexion des caméras](connecting.md).

## Filtres et bandes

Les centres de bande, les limites FWHM et l’intégralité du catalogue M3M (23 références) relèvent des spécifications produit ; ils figurent donc dans le manuel du matériel : [**Filtres et bandes spectrales**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ce qui importe du point de vue logiciel : le code de filtre dans la chaîne de modèle détermine les produits que Chloros peut générer. Les caméras à filtre RGB (`FRGB`) émettent uniquement des produits débayérisés et de prévisualisation — la radiance et la réflectance par bande n’ont pas de sens pour un capteur à large bande, c’est pourquoi Chloros les ignore et l’indique. Tous les autres filtres fournissent la chaîne complète radiance → réflectance → indice.

## L’étalonnage radiométrique en bref

Chaque caméra LATTICE est étalonnée individuellement en usine selon une chaîne traçable NIST et est livrée avec un certificat spécifique à chaque caméra. Le contenu de cet étalonnage, la méthode de mesure et la précision que vous pouvez indiquer figurent dans le manuel du matériel : [**Étalonnage radiométrique en usine**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Du côté logiciel, ce qui importe, c’est que Chloros détermine le bon étalonnage lorsqu’une caméra se connecte et fixe les coefficients appliqués dans chaque exportation — voir [Connexion des caméras](connecting.md).

## Dans ce chapitre

* [Connexion des caméras](connecting.md) — détection automatique, boîte de dialogue de connexion de l&#x27;interface graphique, équivalents CLI/SDK, et comment l’étalonnage d’usine est géré (pack intégré à la caméra ou cloud) lors de la connexion d’une caméra.

D&#x27;autres sujets liés à LATTICE — paramètres de caméra et contrôle en direct, modes de capture, réseaux multicaméras, ainsi que le traitement mono (M3M) et les index — sont traités dans des sections distinctes de ce manuel, et la liste complète des commandes se trouve dans la [Référence CLI](../reference/cli-reference.md) et [Référence SDK](../reference/sdk-reference.md).
