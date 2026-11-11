---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Puis-je traiter avec Chloros des images provenant d&#x27;appareils photo d&#x27;autres marques que MAPIR ?</summary>

Non, Chloros ne prend en charge que le traitement des images provenant des caméras MAPIR, à savoir les gammes Survey3 et LATTICE. Veuillez consulter la liste des [modèles de caméras pris en charge](supported-cameras.md) pour plus d&#x27;informations. Nous proposons toutefois le traitement d&#x27;images provenant d&#x27;autres caméras sur MAPIR Cloud ; consultez la liste complète [ici](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Chloros prend-il en charge les caméras LATTICE ?</summary>

Oui. Chloros 1.3.0 prend en charge de bout en bout les modules de caméra LATTICE M3C et M3M : **contrôle en direct**— détection, connexion, prévisualisation et capture depuis l’onglet « Cameras » de l’interface graphique, `chloros-cli lattice`, ou via le SDK SDK, y compris les réseaux multicaméras synchronisés avec synchronisation temporelle PTP — et**traitement radiométrique complet** des captures (brut → débayérisé → radiance → réflectance → indice). Consultez la page [Caméras prises en charge](supported-cameras.md) et le [guide LATTICE](lattice/README.md).

</details>

<details>

<summary>Puis-je calibrer mes images en réflectance sans cible d’étalonnage ?</summary>

**Survey3

:** Non. Sans une image de la cible d’étalonnage capturée au moment de la prise de vue des images hors cible, vous ne pourrez pas établir de correspondance entre les valeurs des pixels de l’image et un pourcentage de réflectance connu. Si vous n’incluez pas non plus le journal d’un capteur de lumière MAPIR

, le spectre de la lumière ambiante ne sera pas mesuré et les résultats de réflectance ne seront pas précis.

**LATTICE :** Oui. La réflectance peut être rapportée à l’irradiance descendante mesurée par un capteur de lumière DAQ plutôt que par un panneau (ρ = π·L/E). Lorsqu’une cible dans le cadre ayant passé le contrôle qualité *est* présente, elle devient par défaut la référence absolue (`--reflectance-source auto`), sauf si elle est validée avec un indicateur, auquel cas la cible et les chaînes DAQ se font concurrence sur le score mesuré. Une exception : « La réflectance F988 est calibrée à l’aide d’un panneau de réflectance présent dans la scène : la bande se situant au-delà de la plage calibrée du capteur de lumière du DAQ, Chloros

utilise votre dernière capture du panneau et la conserve entre deux observations du panneau. » Voir [Cibles d’étalonnage](calibration-targets.md).

</details>

<details>

<summary>Ai-je besoin d’un capteur de lumière DAQ ?</summary>

Pas pour la radiance : les exportations de radiance LATTICE proviennent de l’étalonnage radiométrique d’usine de chaque caméra et ne nécessitent ni capteur DAQ ni cible. Pour la **réflectance**, vous avez besoin d’une référence pour la lumière ambiante — soit une mesure descendante d’un capteur de lumière DAQ, soit une cible d’étalonnage intégrée à l’image. Un capteur DAQ vous permet de produire une réflectance étalonnée**sans placer aucun panneau dans la scène**. Les fichiers `.daq` enregistrés sont automatiquement associés à vos images grâce à leur horodatage. Consultez la page [Cibles d’étalonnage](calibration-targets.md) et la [Référence CLI

](reference/cli-reference.md).

</details>

<details>

<summary>Puis-je utiliser Chloros avec un assistant IA (Claude, ChatGPT, etc.) ?</summary>

Oui — ce manuel ainsi que la CLI et le SDK sont conçus à cet effet :

* L’index complet du manuel est accessible à l’adresse `https://mapir.gitbook.io/chloros/llms.txt` afin que les assistants IA puissent découvrir chaque page.
* Le code Markdown brut de chaque page est disponible à l&#x27;URL de la page en minuscules, suivie de `.md` (par exemple `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Référence CLI](reference/cli-reference.md) et la [Référence SDK](reference/sdk-reference.md) sont rédigées pour être utilisées par les modèles de langage (LLM) : indicateurs exacts, valeurs par défaut, sémantique de sortie et commandes copiables-collables.

Consultez la section [Assistants IA](ai-assistants.md) pour savoir comment configurer votre assistant pour qu’il utilise Chloros.

</details>

<details>

<summary>Où sont enregistrés mes fichiers de sortie traités ?</summary>

Les fichiers de sortie sont enregistrés dans le dossier du projet, regroupés par caméra puis par format de fichier :

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **dossier-caméra** — `LATT-<sensor>-<lens>-F<filter>` pour LATTICE, `<model>_<filter>` (par exemple `Survey3N_RGN`) pour Survey3
* **dossier-format** — `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32`
* **dossiers de produits** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (toujours sous `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — LATTICE multispectral uniquement : masques par pixel à côté des dossiers de produits, voir ci-dessous**Les fichiers exportés conservent le nom du fichier source — c’est le dossier qui identifie le produit, et non un suffixe de nom de fichier.**Avec l’CLI, le dossier de projet est créé dans votre dossier de projet par défaut (`Documents/MAPIR/Chloros Projects`, sauf si vous l’avez modifié), sauf si vous passez `-o` ; un chemin relatif `-o` est interprété par rapport au dossier à partir duquel vous exécutez la commande. Notez qu’une exécution `chloros-cli process` qui a demandé des produits mais n’en a écrit aucun affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie différent de zéro**, ce qui permet aux scripts de la détecter. Voir [Formats d’images de sortie](output-image-formats.md) et la [Référence de l’interface CLI](reference/cli-reference.md).

</details>

<details>

<summary>Que sont les dossiers « clamp_masks » et les fichiers *_clampmask.tif ?</summary>

Nouveauté de la version 1.3.0. Les produits de radiance et de réflectance d’une caméra multispectrale LATTICE M3C subissent un démixage de bandes 3×3, et lorsqu’une valeur négative est générée dans une bande, Chlorose écrête cette valeur à zéro. Un produit comportant au moins un pixel écrêté reçoit un masque `uint8` à bande unique qui indique quels pixels et quelles bandes ont été écrêtés (bit 0 = canal R du produit, bit 1 = G, bit 2 = B). Les masques sont enregistrés dans un dossier `clamp_masks/` **à côté** des dossiers de produits — par exemple `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — et la balise XMP du produit `Chloros:UnmixClampMask` désigne son masque (ou indique `none`).

Il ne s’agit pas d’images de la scène : l’importation propre à Chloros les ignore, et comme elles ne se trouvent jamais à l’intérieur d’un dossier de produit, vous pouvez transmettre un dossier de produit à Pix4D ou Metashape tel quel. Voir [Masques de clamp](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Pourquoi Chloros affiche-t-il le message « Le dossier du projet est trop profond pour cet ordinateur » ?</summary>

Cela concerne uniquement Windows. Windows est livré avec les chemins d’accès longs désactivés, et aucun chemin d’accès ne peut alors dépasser 260 caractères. Les produits Chloros se trouvent jusqu’à environ 130 caractères en dessous du dossier de projet ; Chloros refuse donc un dossier de projet plus profond avant le traitement — plutôt que de s’exécuter sans rien écrire, comme cela pouvait arriver avant la version 1.3.0. Le message indique le dossier de projet le plus long autorisé.

Pour résoudre ce problème, procédez de l’une des deux manières suivantes :

* Utilisez un dossier de projet plus court (le **répertoire de travail** dans les paramètres du projet, ou `-o` / `chloros-cli set-project-folder` pour l’CLI).
* Activez les chemins d’accès longs sous Windows et redémarrez Chloros. L’option **Activer les chemins d’accès longs sous Windows** du programme d’installation de Chloros (cochée par défaut) permet de le faire ; relancez le programme d’installation si vous l’avez décochée.

Voir [Lancement du traitement](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Puis-je modifier mes images avant leur traitement dans Chloros ?</summary>

Non. Chloros part du principe que les données d’entrée n’ont pas été modifiées. Ne modifiez pas les noms de fichiers.

</details>

<details>

<summary>Puis-je régler mes caméras MAPIR Survey3 sur l&#x27;exposition automatique et traiter les images dans Chloros ?</summary>

Non. Les jeux d’images Survey3 doivent avoir une exposition fixe/verrouillée ; il ne faut donc pas utiliser de vitesse d’obturation automatique ni de sensibilité ISO automatique. Toutes les images prises avec le même modèle de caméra doivent avoir une vitesse d’obturation et une sensibilité ISO (exposition) identiques.

Les caméras LATTICE ne sont pas soumises à cette restriction : Chloros contrôle leur exposition en temps réel (Smart AE), et chaque capture enregistre l’exposition et le gain réellement utilisés, ce dont tient compte le pipeline radiométrique.

</details>

<details>

<summary>Chloros peut-il traiter ou analyser des images orthomosaïques ?</summary>

Non. Seules les images individuelles prises par les caméras MAPIR sont prises en charge, et non les images assemblées telles qu’une carte orthomosaïque.

</details>

<details>

<summary>Comment puis-je accélérer l’étape de détection des cibles dans Chloros ?</summary>

Dans le tableau du navigateur de fichiers, présélectionner les images cibles dans la colonne de droite indiquera à Chloros de ne rechercher les cibles d’étalonnage que dans ces images, ce qui accélère considérablement le traitement.

</details>

<details>

<summary>Si je prévois de télécharger mes images sur <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> dois-je les traiter dans Chloros avant le téléchargement ?</summary>

Si vous prévoyez de les télécharger sur notre plateforme de traitement en ligne [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ne modifiez pas les images avant le téléchargement. Le Cloud effectuera tout le traitement nécessaire, et bien plus encore.

</details>

<details>

<summary>MAPIR prendra-t-il un jour en charge la fonctionnalité X ? J&#x27;aimerais vraiment que MAPIR propose X.</summary>

Nous sommes toujours intéressés par vos retours sur nos produits. Si vous rencontrez un problème avec nos produits ou si vous avez une suggestion pour les améliorer, n’hésitez pas à [NOUS CONTACTER](https://www.mapir.camera/community/contact) pour nous faire part de vos remarques. La majeure partie de notre R&amp;D s&#x27;appuie sur l&#x27;écoute des besoins les plus importants de nos clients.

</details>

<details>

<summary>Chloros est-il disponible sous Linux ?</summary>

Oui ! Chloros 1.3.0 prend en charge Linux amd64 (x86_64) et arm64 (NVIDIA Jetson JetPack 6) via les paquets `.deb`. L’interface en ligne de commande (CLI) et le SDK Python sont entièrement pris en charge sous Linux, y compris le contrôle en temps réel des caméras LATTICE et des capteurs DAQ. Il n’existe pas d’interface graphique (GUI) pour Linux : toutes les interactions s’effectuent via l’[interface en ligne de commande](CLI.md) ou le [SDK Python](api-python-sdk.md). Consultez la [Présentation de Linux](linux/linux-overview.md) pour plus de détails.

</details>

<details>

<summary>Puis-je exécuter Chloros sur NVIDIA Jetson ?</summary>

Oui ! Chloros prend en charge la **gamme Orin** de NVIDIA Jetson — Orin Nano, Orin NX et AGX Orin — fonctionnant sous JetPack 6. Les modèles plus anciens (Jetson Nano, TX2, Xavier NX) ne peuvent pas exécuter JetPack 6 et ne sont pas pris en charge par le package actuel. Chloros détecte automatiquement votre modèle de Jetson et optimise sa stratégie de traitement. Le package Jetson utilise la version CUDA de PyTorch fournie par JetPack 6, ce qui permet d’exécuter le traitement sur le GPU du Jetson. Consultez le [Guide NVIDIA Jetson](linux/nvidia-jetson-guide.md) pour obtenir des instructions de configuration et de déploiement.

</details>

<details>

<summary>Chloros s&#x27;adapte-t-il automatiquement à mon matériel ?</summary>

Oui ! Chloros intègre la [fonctionnalité « Dynamic Compute Adaptation »](processing-architecture/dynamic-compute-adaptation.md) qui détecte automatiquement votre CPU, votre GPU, votre mémoire vive et (sur Jetson) vos capteurs thermiques. Il sélectionne ensuite la stratégie de traitement optimale — allant de `GPU_PARALLEL` sur les GPU à grande capacité de mémoire à `GPU_SINGLE` sur les plus petits, en passant par `CPU_PARALLEL` sur les systèmes dépourvus d’un GPU CUDA utilisable par PyTorch. Aucune configuration manuelle n’est nécessaire.

</details>

<details>

<summary>Qu’est-ce que le pipeline de traitement à 4 threads ?</summary>

Chloros
utilise une architecture en pipeline à 4 threads pour les utilisateurs de Chloros
+ : Le thread 1 (Détection) charge les images et détecte les cibles d’étalonnage, le thread 2 (Étalonnage) calcule l’étalonnage de réflectance, le thread 3 (Traitement) effectue le débayérage accéléré par GPU et le calcul de l’indice, et le thread 4 (Exportation) écrit les fichiers de sortie. Plusieurs images peuvent être traitées simultanément dans différents threads pour un débit maximal. Voir [Pipeline de traitement](processing-architecture/processing-pipeline.md) pour plus de détails.

</details>

<details>

<summary>Comment effectuer des diagnostics sur mon installation d’Chloros ?</summary>

Utilisez la commande `selftest` pour lancer un test de fonctionnement en 7 étapes : version, disponibilité des ports, démarrage du backend, connectivité de l’API (`/api/test`), informations système (`/api/system-info` — GPU/CUDA/PyTorch), présence du modèle de débruitage et état de préparation de CUDA et du débruitage :

```bash
chloros-cli selftest
```

Ceci est particulièrement utile sur les systèmes Linux pour vérifier la configuration du GPU et de CUDA. Sur un Jetson, `CUDA: False` signifie généralement que les bibliothèques CUDA et cuDNN de JetPack 6 sont manquantes : installez-les à l’aide de la commande `sudo apt install nvidia-jetpack`.

</details>
