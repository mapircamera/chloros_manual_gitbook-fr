___PROTÉGÉ_0001___

# Cibles d'étalonnage

MAPIR propose diverses cibles d'étalonnage pour couvrir un large éventail d'applications. La cible compacte T4-R50 présentée ci-dessous contient 4 panneaux dont la réflectance lumineuse a été mesurée entre 250 et 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>§

Les cibles de référence diffuses T4 ont les courbes de réflectance suivantes, [data download here](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157)X :

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Réflectance T4 : : 250-2500nm</p></figcaption></figure>

ZXZXZX000015ZXZZZXZZXX000016ZXZZZZZZZXZX000017ZXZZZZZZZX000018ZXZZZZZXMAPIR Réflexion T4 : : 400-1000nm</p></figcaption></figure>

En regardant le graphique de réflectance, vous pouvez voir que les valeurs sont la longueur d'onde (axe x) en fonction du pourcentage de réflectance (axe y). Lorsque nous capturons une image de la cible d'étalonnage, nous créons une relation entre la valeur du pixel et le pourcentage de réflectance, dans le spectre auquel chaque bande du capteur de la caméra est sensible.

Cela signifie qu'avec chaque image que vous capturez avec nos caméras, vous pouvez utiliser une photo de nos cibles de réflectance, telles que ZXZXZX000046ZXXZXX ou ZXZXZX000047ZXXZXX pour calibrer les images en fonction de la réflectance. Une fois calibré, chaque pixel de l'image est égal à un pourcentage de réflectance.

Si vous produisez les images calibrées dans §§§42§§ sous forme de JPG ou §§§46§§, le pourcentage de réflectance est calculé en divisant la valeur du pixel par la profondeur de bits du format de l'image. Ainsi, pour JPG, divisez par 255, et pour TIFFX, divisez par 65 535. Vous pouvez également choisir le format de sortie PERCENT dans ZXXZXZ000049ZXZXZXX, et chaque pixel aura alors une valeur en pourcentage comprise entre 0,0 et 1,0 (0 % et 100 % de réflectance). Gardez à l'esprit que certaines applications d'image ne peuvent pas accepter les images en pourcentage (virgule flottante) et qu'elles ont une taille de stockage importante.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
