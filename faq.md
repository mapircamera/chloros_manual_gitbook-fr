---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Puis-je traiter des images provenant de caméras qui ne sont pas de la marque MAPIR avec Chloros ?</summary>

Non, Chloros ne prend en charge que le traitement des images provenant d&#x27;appareils photo MAPIR. Veuillez consulter la liste des [modèles d&#x27;appareils photo pris en charge](supported-cameras.md) pour plus d&#x27;informations. Nous proposons le traitement d&#x27;autres appareils photo sur MAPIR Cloud, voir la liste complète [ici](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Puis-je calibrer mes images pour la réflectance sans cible de calibrage ?</summary>

Non. Sans une image de la cible d&#x27;étalonnage capturée au moment où les images sans cible sont prises, vous ne pourrez pas relier les valeurs de pixels de l&#x27;image à un pourcentage de réflectance connu. Si vous n&#x27;incluez pas non plus le journal d&#x27;un capteur de lumière MAPIR, le spectre de la lumière ambiante ne sera pas mesuré et les résultats de réflectance ne seront pas précis.

</details>

<details>

<summary>Puis-je modifier mes images avant de les traiter dans Chloros ?</summary>

Non. Chloros part du principe que les données d&#x27;entrée n&#x27;ont pas été modifiées. Ne modifiez pas les noms de fichiers.

</details>

<details>

<summary>Puis-je régler mes caméras MAPIR et Survey3 sur l&#x27;exposition automatique et traiter les images dans Chloros ?</summary>

Non. Les ensembles de données d&#x27;images Survey3 doivent avoir une exposition fixe/verrouillée, donc pas de vitesse d&#x27;obturation automatique ni d&#x27;ISO automatique. Toutes les images d&#x27;un même modèle de caméra doivent avoir une vitesse d&#x27;obturation et une sensibilité ISO (exposition) identiques.

</details>

<details>

<summary>Chloros peut-il traiter ou analyser des images orthomosaïques ?</summary>

Non. Seules les images individuelles provenant d&#x27;une caméra MAPIR sont prises en charge, et non les images assemblées telles qu&#x27;une carte orthomosaïque.

</details>

<details>

<summary>Comment puis-je accélérer l&#x27;étape de détection des cibles de Chloros ?</summary>

Dans le tableau du navigateur de fichiers, présélectionner les images cibles dans la colonne de droite indiquera à Chloros de rechercher les cibles d&#x27;étalonnage uniquement dans ces images, ce qui accélère considérablement le traitement.

</details>

<details>

<summary>Si je télécharge mes images sur <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> dois-je les traiter dans Chloros avant le téléchargement ?</summary>

Si vous prévoyez de télécharger vos images sur notre plateforme de traitement en ligne [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ne modifiez pas les images avant le téléchargement. Cloud effectuera tout le traitement nécessaire et bien plus encore.

</details>

<details>

<summary>MAPIR prendra-t-il un jour en charge la fonctionnalité X ? J&#x27;aimerais vraiment que MAPIR propose X.</summary>

Nous sommes toujours intéressés par vos commentaires sur nos produits. Si vous rencontrez un problème avec nos produits ou si vous avez une suggestion pour les améliorer, veuillez [NOUS CONTACTER](https://www.mapir.camera/community/contact) pour nous faire part de vos remarques. La plupart de nos activités de R&amp;D s&#x27;orientent en fonction des besoins les plus importants de nos clients.

</details>

<details>

<summary>Chloros est-il disponible pour Linux ?</summary>

Oui ! Chloros 1.1.0 prend en charge Linux amd64 (x86_64) et arm64 (NVIDIA Jetson JetPack 6) via les paquets `.deb`. Les versions CLI et Python SDK sont entièrement prises en charge sur Linux. Il n&#x27;y a pas d&#x27;interface graphique pour Linux — toute interaction se fait via [CLI](CLI.md) ou [Python SDK](api-python-sdk.md). Voir [Présentation de Linux](linux/linux-overview.md) pour plus de détails.

</details>

<details>

<summary>Puis-je exécuter Chloros sur NVIDIA Jetson ?</summary>

Oui ! Chloros 1.1.0 prend en charge les plateformes NVIDIA Jetson, notamment Jetson Nano, Orin Nano, Orin NX et AGX Orin fonctionnant sous JetPack 6. Chloros détecte automatiquement votre modèle Jetson et optimise sa stratégie de traitement. Consultez le [Guide NVIDIA Jetson](linux/nvidia-jetson-guide.md) pour obtenir des instructions de configuration et de déploiement.

</details>

<details>

<summary>Chloros s&#x27;optimise-t-il automatiquement pour mon matériel ?</summary>

Oui ! Chloros 1.1.0 inclut la [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) qui détecte automatiquement votre CPU, votre GPU, votre RAM et (sur Jetson) vos capteurs thermiques. Il sélectionne ensuite la stratégie de traitement optimale — de `GPU_PARALLEL` sur les systèmes à grande mémoire à `GPU_SINGLE` sur les appareils aux ressources limitées, en passant par `CPU_PARALLEL` sur les systèmes sans GPU NVIDIA. Aucune configuration manuelle n&#x27;est nécessaire.

</details>

<details>

<summary>Qu&#x27;est-ce que le pipeline de traitement à 4 threads ?</summary>

Chloros 1.1.0 utilise une architecture en pipeline à 4 threads pour les utilisateurs de Chloros+ : Le thread 1 (Détection) charge les images et détecte les cibles d&#x27;étalonnage, le thread 2 (Étalonnage) calcule l&#x27;étalonnage de réflectance, le thread 3 (Traitement) effectue le débayering accéléré par GPU et le calcul de l&#x27;indice, et le thread 4 (Exportation) écrit les fichiers de sortie. Plusieurs images peuvent être traitées simultanément dans différents threads pour un débit maximal. Voir [Pipeline de traitement](processing-architecture/processing-pipeline.md) pour plus de détails.

</details>

<details>

<summary>Comment puis-je exécuter des diagnostics sur mon installation Chloros ?</summary>

Utilisez la commande `selftest` pour exécuter 7 diagnostics système, notamment la vérification de la version, la disponibilité des ports, le démarrage du backend, la connectivité API, les informations système, les modèles de débruitage et la disponibilité de CUDA :

```bash
chloros-cli selftest
```

Cette commande est particulièrement utile sur les systèmes Linux/Jetson pour vérifier la configuration du GPU et de CUDA.

</details>
