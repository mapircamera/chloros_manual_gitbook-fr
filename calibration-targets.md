---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cibles d'étalonnage

MAPIR propose diverses cibles d'étalonnage pour couvrir un large éventail d'applications. La cible compacte T4-R50 présentée ci-dessous contient 4 panneaux dont la réflectance lumineuse a été mesurée entre 250 et 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Les cibles de référence diffuses T4 ont les courbes de réflectance suivantes, [data download here](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157) :

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 400-1000nm</p></figcaption></figure>

En regardant le graphique de réflectance, vous pouvez voir que les valeurs sont la longueur d'onde (axe x) en fonction du pourcentage de réflectance (axe y). Lorsque nous capturons une image de la cible d'étalonnage, nous créons une relation entre la valeur du pixel et le pourcentage de réflectance, dans le spectre auquel chaque bande du capteur de la caméra est sensible.

Cela signifie qu'avec chaque image que vous capturez avec nos caméras, vous pouvez utiliser une photo de nos cibles de réflectance, telles que [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) pour calibrer les images en fonction de la réflectance. Une fois calibré, chaque pixel de l'image est égal à un pourcentage de réflectance.

Si vous produisez les images calibrées dans Chloros en tant que JPG typique ou TIFF, le pourcentage de réflectance est calculé en divisant la valeur du pixel par la profondeur de bits du format d'image. Ainsi, pour JPG, divisez par 255, et pour TIFF, divisez par 65 535. Vous pouvez également choisir le format de sortie PERCENT dans Chloros, et chaque pixel aura alors une valeur en pourcentage comprise entre 0,0 et 1,0 (0 % et 100 % de réflectance). N'oubliez pas que certaines applications d'image ne peuvent pas accepter les images en pourcentage (virgule flottante) et que leur taille de stockage est importante.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
