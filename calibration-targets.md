---
description: Lab-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Cibles d&#x27;étalonnage

MAPIR propose différentes cibles d&#x27;étalonnage pour couvrir toute une gamme d&#x27;applications. Le T4-R50 compact présenté ci-dessous contient 4 panneaux dont la réflectance lumineuse a été mesurée entre 250 et 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Les cibles de référence diffuses T4 présentent les courbes de réflectance suivantes, [téléchargement des données ici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157) :

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Réflectance :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Réflectance :: 400-1000 nm</p></figcaption></figure>

En observant le graphique de réflectance, vous pouvez voir que les valeurs correspondent à la longueur d&#x27;onde (axe x) par rapport au pourcentage de réflectance (axe y). Lorsque nous capturons une image de la cible d&#x27;étalonnage, nous créons ensuite une relation entre la valeur des pixels et le pourcentage de réflectance, dans le spectre auquel chacune des bandes du capteur de la caméra est sensible.

Cela signifie que pour chaque image que vous capturez avec nos caméras, vous pouvez utiliser une photo de nos cibles de réflectance, telles que la [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou la [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), afin de calibrer les images en fonction de la réflectance. Une fois calibré, chaque pixel de l&#x27;image correspond à un pourcentage de réflectance.

Si vous exportez les images calibrées dans Chloros au format JPG ou TIFF classique, le pourcentage de réflectance est calculé en divisant la valeur du pixel par la profondeur de bits du format d&#x27;image. Ainsi, pour le format JPG, divisez par 255, et pour le format TIFF, divisez par 65 535. Vous pouvez également choisir le format PERCENT dans Chloros, auquel cas chaque pixel aura une valeur comprise entre 0,0 et 1,0 (0 % à 100 % de réflectance). N&#x27;oubliez pas que certaines applications d&#x27;image ne prennent pas en charge les images en pourcentage (virgule flottante) et que celles-ci occupent beaucoup d&#x27;espace de stockage.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
