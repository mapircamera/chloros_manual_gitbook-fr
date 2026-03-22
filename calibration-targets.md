---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cibles d&#x27;étalonnage

MAPIR propose diverses cibles d&#x27;étalonnage destinées à une large gamme d&#x27;applications. Le modèle compact T4-R50 présenté ci-dessous comprend 4 panneaux dont la réflectance lumineuse a été mesurée dans la gamme de 250 à 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Les cibles de référence diffuses T4 présentent les courbes de réflectance suivantes, [téléchargement des données ici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157) :

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Réflectance T4 :: 250-2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Réflectance T4 :: 400-1 000 nm</p></figcaption></figure>Les cibles de référence diffuses T4P présentent les courbes de réflectance suivantes, [téléchargement des données ici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157) :

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Réflectance T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Réflectance T4P :: 400-1000 nm</p></figcaption></figure>En observant le graphique de réflectance, vous pouvez voir que les valeurs représentent la longueur d&#x27;onde (axe des x) en fonction du pourcentage de réflectance (axe des y). Lorsque nous capturons une image de la cible d&#x27;étalonnage, nous établissons alors une relation entre la valeur des pixels et le pourcentage de réflectance, dans le spectre auquel chacune des bandes du capteur de la caméra est sensible.

Cela signifie que pour chaque image que vous capturez avec nos caméras, vous pouvez utiliser une photo de nos cibles de réflectance, telles que la [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou la [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), pour calibrer les images en fonction de la réflectance. Une fois calibrée, chaque pixel de l&#x27;image correspond à un pourcentage de réflectance.

Si vous exportez les images calibrées au format JPG standard (Chloros) ou au format TIFF, le pourcentage de réflectance est calculé en divisant la valeur du pixel par la profondeur de bits du format d&#x27;image. Ainsi, pour le format JPG, divisez par 255, et pour le format TIFF, divisez par 65 535. Vous pouvez également choisir le format de sortie PERCENT dans Chloros ; chaque pixel aura alors une valeur comprise entre 0,0 et 1,0 (réflectance de 0 % à 100 %). Gardez simplement à l&#x27;esprit que certaines applications d&#x27;image ne prennent pas en charge les images en pourcentage (à virgule flottante) et que celles-ci occupent beaucoup d&#x27;espace de stockage.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
