---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Puis-je traiter des images provenant d'appareils photo qui ne sont pas de la marque MAPIR avec Chloros ?</summary>

Non, Chloros ne prend en charge que le traitement des images d'appareils photo MAPIR. Veuillez consulter la liste des [supported camera models](supported-cameras.md) pour plus d'informations. Nous proposons le traitement d'autres caméras sur MAPIR Cloud, voir la liste complète [here](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Puis-je calibrer mes images pour la réflectance sans cible de calibrage ?</summary>

Non. Sans une image de la cible de calibration capturée à peu près au moment où les images sans cible sont capturées, vous ne serez pas en mesure de relier les valeurs des pixels de l'image à un pourcentage de réflectance connu. Si vous n'incluez pas non plus le journal d'un capteur de lumière MAPIR, le spectre de la lumière ambiante ne sera pas mesuré et les résultats de réflectance ne seront pas précis.

</details>

<details>

<summary>Puis-je éditer mes images avant leur traitement dans Chloros ?</summary>

Non. Chloros suppose que les données d'entrée n'ont pas été modifiées. Ne changez pas les noms de fichiers.

</details>

<details>

<summary>Puis-je régler mes appareils photo MAPIR Survey3 sur l'exposition automatique et traiter les images dans Chloros ?</summary>

Non. Les ensembles de données d'images Survey3 doivent avoir une exposition fixe/verrouillée, donc pas de vitesse d'obturation automatique ou d'ISO automatique. Toutes les images d'un même modèle d'appareil photo doivent avoir une vitesse d'obturation et une sensibilité ISO (exposition) identiques.

</details>

<details>

<summary>Est-ce que Chloros peut traiter ou analyser des images orthomosaïques ?</summary>

Non. Seules les images individuelles de la caméra MAPIR sont prises en charge, et non les images assemblées comme une carte orthomosaïque.

</details>

<details>

<summary>Comment puis-je accélérer l'étape de détection de la cible de Chloros ?</summary>

Dans le tableau du navigateur de fichiers, la présélection des images cibles dans la colonne de droite indiquera à Chloros de ne rechercher que les cibles d'étalonnage dans ces images, ce qui accélérera considérablement le traitement.

</details>

<details>

<summary>Si je veux télécharger mes images sur <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>, dois-je les traiter dans Chloros avant de les télécharger ?</summary>

Si vous prévoyez de télécharger sur notre plateforme de traitement en ligne [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ne modifiez pas les images avant de les télécharger. Le nuage effectuera tous les mêmes traitements et plus encore.

</details>

<details>

<summary>Est-ce que MAPIR supportera un jour la fonction X ? J'aimerais vraiment que MAPIR propose X.</summary>

Nous sommes toujours intéressés par les commentaires sur nos produits. Si vous rencontrez un problème avec nos produits ou si vous avez une suggestion sur la manière dont nous pouvons les améliorer, veuillez [CONTACT US](https://www.mapir.camera/community/contact) nous faire part de vos réflexions. La plupart de nos activités de recherche et développement sont guidées par l'écoute des besoins les plus importants de nos clients.

</details>
