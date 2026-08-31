# Profils des capuchons et plage calibrée

> Les capuchons eux-mêmes — quels capuchons sont fournis avec quels capteurs, comment ils se montent et leur comportement optique — sont décrits dans le **[manuel d&#x27;utilisation du DAQ](https://mapir.gitbook.io/daq)**. Cette page traite de la *déclaration* du capuchon installé à Chloros, ce qui permet d’assurer l’exactitude de la correction.

L&#x27;étalonnage radiométrique d&#x27;usine de chaque capteur de lumière DAQ décrit le capteur *nu*. Le capuchon physique monté sur le diffuseur modifie la lumière captée par le capteur ; c&#x27;est pourquoi Chloros applique un **profil de correction du capuchon** mesuré en usine en plus du jeu de données d&#x27;étalonnage. La déclaration du capuchon approprié fait partie intégrante de l’obtention de données étalonnées — cette page présente les capuchons disponibles pour chaque modèle, la manière de les déclarer et la plage spectrale réellement étalonnée du capteur.

## Disponibilité des capuchons par modèle

| Profil de capuchon (`cap_id`) | Capuchon physique | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Capuchon de correction cosinus « Sunshine » (**par défaut sur tous les modèles**) | Oui | Oui | Oui |
| `fov_15` / `fov_45` / `fov_90` | Cônes de restriction du champ de vision (15° / 45° / 90°) | Oui | — | Oui |
| `fov_30` / `fov_60` | Cônes de limitation du champ de vision (30° / 60°) | Oui | — | — |
| `none` | Sans capuchon | — | — | Oui |

Remarques spécifiques au modèle :

* **Le DAQ-M dispose d’un seul profil de capuchon : `sunshine_cosine`.** « Bare-plus-Sunshine-cap » correspond à sa définition produit, et un DAQ-M « nu » ne nécessite aucun profil géométrique.
* **Un DAQ-U « nu » est véritablement « nu »** — il ne nécessite aucun profil géométrique, c&#x27;est pourquoi aucun profil `none` n&#x27;existe pour celui-ci.
* **Le profil `none` sur un DAQ-E n’est PAS une opération fictive.** Le diffuseur encastré et recouvert de verre du DAQ-E possède sa propre correction géométrique réelle ; ainsi, « sans capuchon » constitue en soi un profil mesuré sur ce modèle.
* Un **DAQ-E « nu » ne peut mesurer la lumière solaire directe à aucune élévation** — le capuchon Sunshine constitue la configuration de terrain. Ne prévoyez pas de travaux en extérieur avec un DAQ-E « nu ».

Dans les paramètres par capteur de l’interface graphique (icône en forme d’engrenage dans l’onglet « Capteurs de lumière »), le menu déroulant **Cap** propose également l’option « Aucun (capteur nu) » sur les modèles DAQ-U et DAQ-M — sur ces deux modèles, « nu » signifie simplement qu’aucune correction liée au capuchon n’est appliquée, conformément aux remarques ci-dessus. Ne sélectionnez cette option que lorsque le capuchon est physiquement retiré.

## Déclaration du capuchon — et pourquoi c’est important

**Le code `cap_id` déclaré doit correspondre au capuchon physiquement installé sur le capteur.** Ni le capteur ni le logiciel ne peuvent détecter le capuchon installé. Cette déclaration a deux conséquences :

1. La **correction en temps réel** appliquée à chaque spectre.
2. La **mention du capuchon inscrite dans chaque enregistrement `.daq`**, sur laquelle s’appuie le traitement de réflectance en aval.

Le capuchon Sunshine atténue le signal d’environ **12 fois par conception** ; par conséquent, un enregistrement avec un capuchon incorrectement déclaré entraîne une erreur d’échelle des spectres d’environ ce facteur. Déclarez immédiatement tout changement de capuchon.

### Réglage du capuchon

Interface graphique : onglet « Capteurs de lumière » → icône en forme d’engrenage sur la ligne du capteur → menu déroulant **Capuchon**. La valeur par défaut pour tous les modèles est `sunshine_cosine` (tous les capteurs DAQ sont livrés avec le correcteur cosinus installé), et cette sélection est conservée pour le projet.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (le backend doit être en cours d’exécution) :

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Le modèle CLI accepte syntaxiquement la liste complète `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`) ; chaque profil est validé par rapport au modèle du capteur lors de la connexion ; ainsi, un identifiant de capteur indisponible (par exemple un identifiant « E-only » sur un DAQ-U) génère une erreur claire plutôt qu’une correction erronée. La valeur par défaut du backend lorsqu’aucun paramètre n’est fourni est `sunshine_cosine`.

Python SDK Remarque : `cap_id` n’est **pas** un bouton SDK — `connect_daq_sensor()` / `DAQSensorSession` n’exposent aucun paramètre de capacité. Sélectionnez la limite à l’aide des commandes CLI ci-dessus ou du menu déroulant de l’interface graphique ; consultez la [Référence SDK](../reference/sdk-reference.md).

Avancé : les profils sont fournis avec l’installation Chloros à l’emplacement `daq/cap_profiles/<u|m|e>/<cap_id>.json` et peuvent être remplacés par l’utilisateur à l’emplacement `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Indépendamment des plafonds, les capteurs qui n’ont jamais été recalibrés bénéficient automatiquement d’un léger ajustement du décalage d’obscurité dérivé de la flotte — aucune intervention de l’utilisateur n’est nécessaire.

## Performances du plafond d’ensoleillement (configuration en extérieur)

Chiffres sur lesquels vous pouvez baser vos procédures :

| Propriété | Valeur |
| --- | --- |
| Champ de vision | Hémisphérique à 180° |
| Erreur de réponse cosinus | ≤ ±4 % jusqu’à 60° d’incidence ; ≤ ±4,5 % jusqu’à 70° |
| Limite en cas de soleil bas | Non recommandé en dessous d’une élévation solaire d’environ 15° |
| Atténuation | ~12× (conforme à la conception) |
| Répétabilité du remontage de la casquette | ≈ 1,5 % |
| Irradiance quantitative | Moyenne de **≥ 15 s** de relevés (caractéristique de l’instrument, ce n’est pas un défaut) |

Pour toute valeur d’irradiance quantitative — y compris les références de réflectance —, utilisez une moyenne d’au moins 15 secondes de relevés plutôt qu’une seule image.

## Plage spectrale étalonnée

| Propriété | Valeur |
| --- | --- |
| Échantillonnage spectral | 340–1 010 nm par pas de 5 nm (135 points) |
| Plage étalonnée radiométriquement | **~374–974 nm** (imposée par le logiciel) |

Le capteur fournit la grille complète de 340 à 1 010 nm, mais le gain radiométrique traçable selon le NIST s&#x27;étend de ~374 à 974 nm. Chloros **refuse la division par réflectance absolue** pour toute bande de caméra dont moins de la moitié du poids spectral se situe dans cette plage, et signale alors le motif de saut `dls-uncalibrated-band-<nm>` plutôt que de produire un résultat non étalonné. Parmi les références de caméras disponibles à la vente, seul le filtre F988 se situe en dehors de cette plage ; il utilise à la place le flux de travail par panneau de réflectance — voir [Flux de travail par réflectance](reflectance.md).

Pour les modèles de capteurs, les transports et les identifiants de capteurs, consultez la [présentation du DAQ](README.md). Pour savoir comment le « cap stamp » est utilisé lors du traitement, consultez [Enregistrement et format .daq](recording.md).
