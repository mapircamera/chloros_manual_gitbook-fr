# Profils des capuchons et plage d&#x27;étalonnage

> Les capuchons eux-mêmes — quels capuchons sont fournis avec quels capteurs, comment ils se montent et leur comportement optique — sont décrits dans le **[manuel d’utilisation du DAQ](https://mapir.gitbook.io/daq)**. Cette page traite de la *déclaration* du capuchon installé à Chloros
, ce qui permet d’assurer l’exactitude de la correction.

L&#x27;étalonnage radiométrique d&#x27;usine de chaque capteur de lumière DAQ décrit le capteur *nu*. Le capuchon physique monté sur le diffuseur modifie la lumière captée par le capteur ; c&#x27;est pourquoi Chloros
applique un **profil de correction du capuchon** mesuré en usine en plus de l&#x27;ensemble d&#x27;étalonnage. La déclaration du capuchon approprié fait partie intégrante de l’obtention de données étalonnées — cette page présente les capuchons disponibles pour chaque modèle, la manière de les déclarer et la plage spectrale réellement étalonnée du capteur.

## Disponibilité des capuchons par modèle

| Profil de capuchon (`cap_id`) | Capuchon physique | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Capuchon de correction cosinus « Sunshine » (**par défaut sur tous les modèles**) | Oui | Oui | Oui |
| `fov_15` / `fov_45` / `fov_90` | Cônes de limitation du champ de vision (15° / 45° / 90°) | Oui | — | Oui |
| `fov_30` / `fov_60` | Cônes de limitation du champ de vision (30° / 60°) | Oui | — | — |
| `none` | Sans capuchon | — | — | Oui |

Remarques spécifiques au modèle :

* **Le DAQ-M dispose d’un seul profil de capuchon : `sunshine_cosine`.** « Bare-plus-Sunshine-cap » correspond à sa définition produit, et un DAQ-M « nu » ne nécessite aucun profil géométrique.
* **Un DAQ-U « nu » est véritablement « nu »** — il ne nécessite absolument aucun profil géométrique, c&#x27;est pourquoi aucun profil `none` n&#x27;existe pour lui.
* **Le profil `none` sur un DAQ-E n’est PAS une opération fictive.** Le diffuseur encastré et recouvert de verre du DAQ-E possède sa propre correction géométrique réelle ; ainsi, l’absence de capuchon constitue en soi un profil mesuré sur ce modèle.
* Un **DAQ-E nu ne peut mesurer la lumière directe du soleil à aucune élévation** — le capuchon Sunshine constitue la configuration de terrain. Ne prévoyez pas de travaux en extérieur avec un DAQ-E nu.

Dans les paramètres par capteur de l’interface graphique (icône en forme d’engrenage dans l’onglet « Capteurs de lumière »), le menu déroulant **Cap** propose également l’option « Aucun (capteur nu) » sur les modèles DAQ-U et DAQ-M — sur ces deux modèles, « nu » signifie simplement qu’aucune correction liée au capuchon n’est appliquée, conformément aux remarques ci-dessus. Ne sélectionnez cette option que lorsque le capuchon est physiquement retiré.

## Déclaration du capuchon — et pourquoi c’est important

**Le code `cap_id` déclaré doit correspondre au capuchon physiquement installé sur le capteur.** Ni le capteur ni le logiciel ne peuvent détecter le capuchon installé. Cette déclaration a deux conséquences :

1. La **correction en temps réel** appliquée à chaque spectre.
2. La **mention du capuchon inscrite dans chaque enregistrement `.daq`**, sur laquelle s’appuie le traitement de réflectance en aval.

Le capuchon Sunshine atténue le signal d’environ **12 fois par conception** ; par conséquent, un enregistrement avec un capuchon incorrect déclaré entraîne une erreur d’échelle des spectres d’environ ce facteur. Déclarez immédiatement tout changement de capuchon.

### Configuration du capuchon

Interface graphique : onglet « Capteurs de lumière » → icône en forme d’engrenage sur la ligne du capteur → menu déroulant **Capuchon**. La valeur par défaut pour tous les modèles est `sunshine_cosine`, une valeur par défaut (les modèles DAQ-U, DAQ-M et DAQ-E sont livrés sans cap, et un enregistrement indique que le cap a été supposé jusqu’à ce que vous en choisissiez un), et la sélection est conservée pour le projet.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(le backend doit être en cours d’exécution) :

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

La CLI
accepte syntaxiquement la liste complète `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`) ; chaque profil est validé par rapport au modèle du capteur lors de la connexion ; ainsi, un identifiant de capteur indisponible (par exemple, un identifiant « E » uniquement sur un DAQ-U) génère une erreur claire plutôt qu’une correction erronée. La valeur par défaut du backend lorsqu’aucun paramètre n’est fourni est `sunshine_cosine`.

Python
SDK
: nommez le cap au moment de la connexion avec `connect_daq_sensor(cap_id="sunshine_cosine")`, ou remplacez-le en cours d’exécution avec `DAQSensorSession.set_cap("fov_45")` ; consultez la [Référence SDK
](../reference/sdk-reference.md#daq-sensor-sessions).

Avancé : les profils sont fournis avec l’installation d’Chloros
à l’adresse `daq/cap_profiles/<u|m|e>/<cap_id>.json` et peuvent être remplacés par l’utilisateur à l’adresse `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Indépendamment des limites, les capteurs qui n’ont jamais été recalibrés bénéficient automatiquement d’un léger ajustement du décalage de fond dérivé de la flotte — aucune intervention de l’utilisateur n’est nécessaire.

## DAQ-E-S : diffuseur fixe et inclinaison

Le **DAQ-E-S** est un DAQ-E présentant deux différences importantes pour Chloros
. Tout le reste (détection, connexion, PTP, enregistrement, paramètres de l’appareil) fonctionne exactement comme pour un DAQ-E.

* **Son diffuseur est fixe, il ne nécessite donc aucune correction de cap.** Le diffuseur était déjà installé sur l’appareil lors de la mesure de son étalonnage en usine ; cet étalonnage indique donc déjà l’irradiance descendante réelle. Chloros
n’y applique aucun profil de limite, quelle que soit la demande : une limite qui lui est attribuée est ignorée avec un message d’erreur, ses enregistrements indiquent que la limite provient du modèle, et l’avertissement de limite supposée ne se déclenche jamais pour cet appareil. Dans Paramètres du projet → Capteur de lumière DAQ, sa ligne n’affiche aucun plafond, et la dérogation au plafond à l’échelle du projet ne s’applique pas à lui. N’appliquez pas manuellement de plafond à ses données : la correction « Sunshine » appliquée par-dessus donnerait une valeur environ 11 fois trop élevée.
* **Il intègre un accéléromètre à 3 axes.** La réponse cosinus du diffuseur n’est valable que lorsqu’il est à l’horizontale ; ainsi, un capteur incliné renvoie une irradiance erronée sans autre symptôme. Chaque lecture DAQ-E-S comporte donc ses données d’inclinaison, de roulis et de tangage, et un fichier `.daq` enregistré à partir de l’un d’entre eux les stocke à côté de chaque spectre (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), ce qui permet de distinguer un nuage d’un support déstabilisé.

Pour connaître tous les champs d’attitude et la manière dont un défaut dans le flux d’attitude est signalé, consultez [DAQ-E-S dans la référence CLI
](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Performances du capuchon anti-soleil (configuration en extérieur)

Chiffres sur lesquels vous pouvez baser vos procédures :

| Propriété | Valeur |
| --- | --- |
| Champ de vision | Hémisphérique à 180° |
| Erreur de réponse cosinus | ≤ ±4 % jusqu’à 60° d’incidence ; ≤ ±4,5 % jusqu’à 70° |
| Limite en cas de soleil bas | Non recommandé en dessous d’une élévation solaire d’environ 15° |
| Atténuation | Environ 12× (conformément à la conception) |
| Répétabilité du remontage du capuchon | ≈ 1,5 % |
| Irradiance quantitative | Moyenne de **≥ 15 s** de mesures (caractéristique de l’instrument, ce n’est pas un défaut) |

Pour toute valeur d’irradiance quantitative — y compris les références de réflectance —, utilisez une moyenne d’au moins 15 secondes de mesures plutôt qu’une seule image.

## Plage spectrale étalonnée

| Propriété | Valeur |
| --- | --- |
| Échantillonnage spectral | 340–1 010 nm par pas de 5 nm (135 points) |
| Plage étalonnée radiométriquement | **~374–974 nm** (imposée par le logiciel) |

Le capteur fournit la grille complète de 340 à 1 010 nm, mais le gain radiométrique traçable selon le NIST s&#x27;étend de ~374 à 974 nm. Chloros
**refuse la division par réflectance absolue** pour toute bande de caméra dont moins de la moitié de la pondération spectrale se situe dans cette plage, et signale alors le motif de saut `dls-uncalibrated-band-<nm>` plutôt que de produire un résultat non étalonné. Parmi les références de caméras disponibles à la vente, seul le filtre F988 se situe en dehors de cette plage ; il utilise à la place le flux de travail par panneau de réflectance — voir [Flux de travail par réflectance](reflectance.md).

Pour les modèles de capteurs, les transports et les identifiants de capteurs, consultez la [présentation du DAQ](README.md). Pour savoir comment le « cap stamp » est utilisé pendant le traitement, consultez [Enregistrement et format .daq](recording.md).
