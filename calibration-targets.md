---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cibles d&#x27;étalonnage

MAPIR propose diverses cibles d&#x27;étalonnage destinées à une large gamme d&#x27;applications. Le modèle compact T4-R50 présenté ci-dessous comprend 4 panneaux dont la réflectance lumineuse a été mesurée dans la plage de 250 à 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Les cibles de référence diffuses T4 présentent les courbes de réflectance suivantes, [téléchargement des données ici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157) :

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Réflectance T4 :: 250-2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Réflectance T4 :: 400-1 000 nm</p></figcaption></figure>Les cibles de référence diffuses T4P présentent les courbes de réflectance suivantes, [téléchargement des données ici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157) :

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Réflectance T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Réflectance T4P :: 400-1 000 nm</p></figcaption></figure>En observant le graphique de réflectance, vous pouvez constater que les valeurs représentent la longueur d’onde (axe des x) en fonction du pourcentage de réflectance (axe des y). Lorsque nous capturons une image de la cible d’étalonnage, nous établissons alors une relation entre la valeur des pixels et le pourcentage de réflectance, au sein du spectre auquel chacune des bandes du capteur de la caméra est sensible.

Cela signifie que pour chaque image que vous capturez avec nos caméras, vous pouvez utiliser une photo de nos cibles de réflectance, telles que la [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), pour étalonner les images en termes de réflectance. Une fois l&#x27;étalonnage effectué, chaque pixel de l&#x27;image correspond à un pourcentage de réflectance.

Pour les **sorties Survey3** , si vous exportez les images calibrées au format Chloros (JPG classique) ou TIFF, le pourcentage de réflectance est calculé en divisant la valeur du pixel par la profondeur de bits du format d’image. Ainsi, pour le format JPG, divisez par 255, et pour le format TIFF, divisez par 65 535. Vous pouvez également choisir le format de sortie « PERCENT » dans Chloros ; chaque pixel prendra alors une valeur comprise entre 0,0 et 1,0 (réflectance de 0 % à 100 %). Gardez simplement à l’esprit que certaines applications d’imagerie ne prennent pas en charge les images en pourcentage (à virgule flottante), et que celles-ci occupent beaucoup d’espace de stockage.

{% hint style="info" %}
**La réflectance LATTICE utilise une échelle de pixels différente.** La réflectance LATTICE est stockée avec DN 32 768 = 100 % de réflectance (et non 65 535), et chaque fichier comporte une balise XMP `Chloros:PixelScale` indiquant son échelle. Lisez cette balise et divisez par cette valeur plutôt que de supposer une constante — voir [Formats d’images de sortie](output-image-formats.md).
{% endhint %}

## Cibles d’étalonnage avec les caméras LATTICE

Avec les caméras LATTICE, une cible d’étalonnage est **facultative** pour la réflectance : Chloros peut à la place référencer la réflectance par rapport à l’irradiance descendante mesurée par un capteur de lumière DAQ (ρ = π·L/E). La référence est choisie via le paramètre « source de réflectance » (Paramètres du projet dans l’interface graphique ; `--reflectance-source` dans le fichier CLI ; `reflectance_source` dans le fichier SDK) :

| Valeur | Comportement |
| --- | --- |
| `auto` *(par défaut)* | Une cible dans le cadre ayant passé le contrôle qualité (QA) constitue la **référence absolue** ; en l’absence de cible ou en cas d’échec du contrôle qualité, Chloros utilise par défaut la division descendante du DAQ. |
| `target` | Cible stricte uniquement — aucune substitution DAQ. |
| `daq` | DAQ faisant autorité — la mesure descendante sert toujours de référence. |

Comportement supplémentaire des cibles pour LATTICE :

* **Géométries des cibles** — Les panneaux marqués ArUco, les panneaux à zone d’intérêt (ROI) fixe et les cibles en bande sont tous pris en charge ; la géométrie provient de la configuration des cibles du projet.
* **Données de cibles mesurées par unité** — `--target-reflectance-dir DIR` pointe vers un répertoire de balayages de réflectance de cibles mesurés par unité (`<serial>.csv`, recherchés à l&#x27;aide du numéro de série ou du code QR de l&#x27;unité cible). En cas d’échec, Chloros utilise par défaut les spectres nominaux T3/T4P.
* **Ancrage temporel** — une cible détectée permet de calibrer les images environnantes et est conservée entre deux observations de la cible.

La sémantique complète des indicateurs et des exemples se trouve dans la [Référence CLI](reference/cli-reference.md) (voir « Commutateurs d’exportation par produit »).

### F988

« La réflectance F988 est calibrée à l’aide d’un panneau de réflectance intégré à la scène : la bande se situant au-delà de la plage calibrée du capteur de lumière DAQ, Chloros applique votre dernière capture de panneau et la conserve entre les observations du panneau. »

Si F988 est exécuté avec un étalonnage basé uniquement sur le DAQ, Chloros rejette la réflectance basée sur le DAQ pour cette bande et en indique la raison (motif de rejet `dls-uncalibrated-band-988`) ; le flux de travail avec le panneau est la méthode prise en charge.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
