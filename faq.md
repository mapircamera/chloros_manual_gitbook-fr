---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Puis-je traiter avec Chloros des images provenant de caméras qui ne sont pas de la marque MAPIR ?</summary>

Non, Chloros ne prend en charge que le traitement des images provenant des caméras MAPIR, à savoir les gammes Survey3 et LATTICE. Veuillez consulter la liste des [modèles de caméras pris en charge](supported-cameras.md) pour plus d’informations. Nous proposons toutefois le traitement d’autres caméras sur MAPIR Cloud ; consultez la liste complète [ici](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Chloros prend-il en charge les caméras LATTICE ?</summary>

Oui. Chloros 1.2.0 prend en charge de bout en bout les modules de caméra LATTICE M3C et M3M : **contrôle en direct**— détection, connexion, prévisualisation et capture depuis l’onglet « Caméras » de l’interface graphique, `chloros-cli lattice` ou Python SDK, y compris les réseaux multicaméras synchronisés avec synchronisation temporelle PTP — ainsi que le**traitement radiométrique complet** des captures (brut → débayérisé → radiance → réflectance → indice). Voir [Caméras prises en charge](supported-cameras.md) et le [guide LATTICE](lattice/README.md).

</details>

<details>

<summary>Puis-je calibrer mes images en réflectance sans cible d&#x27;étalonnage ?</summary>

**Survey3 :** Non. Sans une image de la cible d’étalonnage capturée au moment où les images hors cible sont enregistrées, vous ne pourrez pas établir de correspondance entre les valeurs des pixels de l’image et un pourcentage de réflectance connu. Si vous n’incluez pas non plus le journal d’un capteur de lumière MAPIR, le spectre de la lumière ambiante ne sera pas mesuré et les résultats de réflectance ne seront pas précis.**LATTICE :** Oui. La réflectance peut être rapportée à l’irradiance descendante mesurée par un capteur de lumière DAQ plutôt que par un panneau (ρ = π·L/E). Lorsqu’une cible intégrée à l’image et ayant passé le contrôle qualité *est* présente, elle devient par défaut la référence absolue (`--reflectance-source auto`). Une exception : « La réflectance F988 est étalonnée à l’aide d’un panneau de réflectance intégré à la scène : la bande se situant au-delà de la plage d’étalonnage du capteur de lumière DAQ, Chloros utilise votre dernière capture du panneau et la conserve entre les observations du panneau. » Voir [Cibles d’étalonnage](calibration-targets.md).

</details>

<details>

<summary>Ai-je besoin d’un capteur de lumière DAQ ?</summary>

Pas pour la radiance : les exportations de radiance LATTICE proviennent de l’étalonnage radiométrique d’usine de chaque caméra et ne nécessitent ni capteur DAQ ni cible. Pour la **réflectance**, vous avez besoin d’une référence pour la lumière ambiante — soit une mesure de la lumière descendante effectuée par un capteur de lumière DAQ, soit une cible d’étalonnage intégrée à l’image. Un capteur DAQ vous permet de produire une réflectance étalonnée**sans placer aucun panneau dans la scène**. Les fichiers `.daq` enregistrés sont automatiquement associés à vos images grâce à leur horodatage. Consultez la page [Cibles d&#x27;étalonnage](calibration-targets.md) et la [Référence CLI](reference/cli-reference.md).

</details>

<details>

<summary>Puis-je utiliser Chloros avec un assistant IA (Claude, ChatGPT, etc.) ?</summary>

Oui — ce manuel et les fichiers CLI/SDK sont conçus à cet effet :

* L&#x27;index complet du manuel est disponible à l&#x27;adresse `https://mapir.gitbook.io/chloros/llms.txt` afin que les assistants IA puissent découvrir chaque page.
* Le code Markdown brut de chaque page est disponible à l&#x27;adresse de la page correspondante en minuscules, suivie de l&#x27;extension `.md` (par exemple `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Référence CLI](reference/cli-reference.md) et [la référence SDK](reference/sdk-reference.md) sont rédigées pour être utilisées par les LLM : indicateurs exacts, valeurs par défaut, sémantique de sortie et commandes copiables-collables.

Consultez la rubrique [Assistants IA](ai-assistants.md) pour savoir comment configurer votre assistant sur Chloros.

</details>

<details>

<summary>Où sont enregistrés mes fichiers de sortie traités ?</summary>

Les fichiers de sortie sont enregistrés dans le dossier du projet, regroupés par caméra puis par format de fichier :

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **dossier-caméra** — `LATT-<sensor>-<lens>-F<filter>` pour LATTICE, `<model>_<filter>` (par exemple `Survey3N_RGN`) pour Survey3
* **dossier-format** — `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32`
* **dossiers de produits** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (toujours sous `tiff32`), `<INDEX>_Index_Images/`**Les fichiers exportés conservent le nom du fichier source — c’est le dossier qui identifie le produit, et non un suffixe de nom de fichier.**Avec CLI, le dossier du projet est créé à côté du dossier d&#x27;entrée, sauf si vous passez `-o`. Notez qu’une exécution de `chloros-cli process` qui a demandé des produits mais n’en a écrit aucun affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie différent de zéro**, ce qui permet aux scripts de la détecter. Consultez la section [Formats d&#x27;images de sortie](output-image-formats.md) et la [Référence CLI](reference/cli-reference.md).

</details>

<details>

<summary>Puis-je modifier mes images avant leur traitement dans Chloros ?</summary>

Non. Chloros part du principe que les données d&#x27;entrée n&#x27;ont pas été modifiées. Ne modifiez pas les noms de fichiers.

</details>

<details>

<summary>Puis-je régler mes caméras MAPIR et Survey3 sur l’exposition automatique et traiter les images dans Chloros ?</summary>

Non. Les ensembles de données d’images Survey3 doivent avoir une exposition fixe/verrouillée ; il ne faut donc pas utiliser de vitesse d’obturation automatique ni de sensibilité ISO automatique. Toutes les images provenant d’un même modèle de caméra doivent avoir une vitesse d’obturation et une sensibilité ISO (exposition) identiques.

Les caméras LATTICE ne sont pas soumises à cette restriction : le logiciel Chloros gère leur exposition en temps réel (Smart AE), et chaque capture enregistre l’exposition et le gain réellement utilisés, ce dont tient compte le pipeline radiométrique.

</details>

<details>

<summary>Chloros peut-il traiter ou analyser des images orthomosaïques ?</summary>

Non. Seules les images individuelles prises par une caméra MAPIR sont prises en charge, et non les images assemblées telles qu’une carte orthomosaïque.

</details>

<details>

<summary>Comment puis-je accélérer l&#x27;étape de détection des cibles dans Chloros ?</summary>

Dans le tableau du navigateur de fichiers, présélectionner les images cibles dans la colonne de droite indiquera à Chloros de ne rechercher les cibles d’étalonnage que dans ces images, ce qui accélérera considérablement le traitement.

</details>

<details>

<summary>Si je prévois de télécharger mes images sur <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> dois-je les traiter dans Chloros avant le téléchargement ?</summary>

Si vous prévoyez de télécharger vos images sur notre plateforme de traitement en ligne [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ne modifiez pas les images avant le téléchargement. Cloud effectuera tous les traitements nécessaires, et bien plus encore.

</details>

<details>

<summary>MAPIR prendra-t-il un jour en charge la fonctionnalité X ? J&#x27;aimerais vraiment que MAPIR propose la fonctionnalité X.</summary>

Nous sommes toujours intéressés par vos retours sur nos produits. Si vous rencontrez un problème avec nos produits ou si vous avez une suggestion pour les améliorer, n’hésitez pas à [NOUS CONTACTER](https://www.mapir.camera/community/contact) pour nous faire part de vos remarques. La majeure partie de notre R&amp;D s’appuie sur l’écoute des besoins les plus importants de nos clients.

</details>

<details>

<summary>Chloros est-il disponible pour Linux ?</summary>

Oui ! Chloros 1.2.0 prend en charge Linux amd64 (x86_64) et arm64 (NVIDIA Jetson JetPack 6) via les paquets `.deb`. Les modèles CLI et Python SDK sont entièrement pris en charge sur Linux, y compris le contrôle en temps réel des caméras LATTICE et des capteurs DAQ. Il n’existe pas d’interface graphique pour Linux — toutes les interactions s&#x27;effectuent via [CLI](CLI.md) ou [Python SDK](api-python-sdk.md). Consultez la [Linux Présentation](linux/linux-overview.md) pour plus de détails.

</details>

<details>

<summary>Puis-je exécuter Chloros sur NVIDIA Jetson ?</summary>

Oui ! Chloros prend en charge les plateformes NVIDIA Jetson, notamment Jetson Nano, Orin Nano, Orin NX et AGX Orin fonctionnant sous JetPack 6. Chloros détecte automatiquement votre modèle Jetson et optimise sa stratégie de traitement. Consultez le [Guide NVIDIA Jetson](linux/nvidia-jetson-guide.md) pour obtenir des instructions de configuration et de déploiement.

</details>

<details>

<summary>Chloros s&#x27;optimise-t-il automatiquement pour mon matériel ?</summary>

Oui ! Chloros intègre la [fonctionnalité Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) qui détecte automatiquement votre CPU, votre GPU, votre mémoire vive et (sur les Jetson) vos capteurs thermiques. Il sélectionne ensuite la stratégie de traitement optimale : de `GPU_PARALLEL` sur les systèmes dotés d’une mémoire importante à `GPU_SINGLE` sur les appareils aux ressources limitées, en passant par `CPU_PARALLEL` sur les systèmes sans GPU NVIDIA. Aucune configuration manuelle n’est nécessaire.

</details>

<details>

<summary>Qu&#x27;est-ce que le pipeline de traitement à 4 threads ?</summary>

Chloros utilise une architecture en pipeline à 4 threads pour les utilisateurs de Chloros+ : Le thread 1 (détection) charge les images et détecte les cibles d’étalonnage, le thread 2 (étalonnage) calcule l’étalonnage de réflectance, le thread 3 (traitement) effectue le débayering accéléré par GPU et le calcul de l’indice, et le thread 4 (exportation) écrit les fichiers de sortie. Plusieurs images peuvent être traitées simultanément dans différents threads pour un débit maximal. Voir [Pipeline de traitement](processing-architecture/processing-pipeline.md) pour plus de détails.

</details>

<details>

<summary>Comment effectuer des diagnostics sur mon installation Chloros ?</summary>

Utilisez la commande `selftest` pour lancer un test de fonctionnement en 7 étapes : version, disponibilité des ports, démarrage du backend, connectivité API (`/api/test`), informations système (`/api/system-info` — GPU/CUDA/PyTorch), présence du modèle de débruitage et état de préparation de CUDA et du débruitage :

```bash
chloros-cli selftest
```

Ceci est particulièrement utile sur les systèmes Linux/Jetson pour vérifier la configuration du GPU et de CUDA.

</details>
