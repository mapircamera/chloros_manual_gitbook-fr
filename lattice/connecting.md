# Connexion des caméras

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>L'onglet « Caméras » avant toute connexion</p></figcaption></figure>

Chloros détecte automatiquement les caméras LATTICE sur le réseau — depuis l’onglet « Caméras » de l’interface graphique, depuis `chloros-cli lattice` ou depuis le SDK Python Python. La chaîne de caractères indiquant le modèle de la caméra détermine toutes les étapes en aval : Chloros détermine le profil du capteur, la configuration des bandes et l’étalonnage d’usine à partir des fichiers `DeviceUserID` et `DeviceSerialNumber` de la caméra. Il n’y a donc **rien à configurer pour chaque caméra**.

Avant la connexion, assurez-vous que le réseau hôte est correctement configuré : adressage local de liaison, trames jumbo et, pour les réseaux de caméras, paramètres du tampon de réception de la carte réseau. Il s’agit de la configuration matérielle, décrite dans le manuel LATTICE : [**Configuration réseau**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Connexion depuis l’interface graphique

Ouvrez l’onglet **Caméras** dans la barre latérale d’Chloros (les onglets matériels apparaissent une fois que le backend a fini de démarrer), ou utilisez le menu principal → **Se connecter à une caméra**. Ces deux options ouvrent la boîte de dialogue**Connecter une ou plusieurs caméras**.

### La boîte de dialogue **Connecter une ou plusieurs caméras**Dès son ouverture (« Analyse du réseau... »), la boîte de dialogue analyse le réseau et répertorie toutes les caméras détectées. Chaque ligne affiche le**modèle**de la caméra (par exemple `LATT-M3M-L41-F550`), son**numéro de série**et son**adresse IP**.

* **Cliquez sur une ligne pour la sélectionner**(mise en surbrillance en vert). Vous pouvez sélectionner**plusieurs caméras** et les connecter en une seule fois — Chloros les connecte les unes après les autres.
* Les lignes portant la mention **« Connectée »** sont déjà connectées et ne peuvent pas être resélectionnées.
* Les lignes portant la mention **« Dans un réseau »** appartiennent à un réseau de caméras actuellement connecté. Déconnectez d’abord le groupe pour utiliser cette caméra de manière autonome.
* **Connecter** — connecte la ou les caméras sélectionnées ; le bouton affiche un nombre, par exemple « Connecter (3) », lorsque plusieurs caméras sont sélectionnées.
* **Re-scanner** — relance la recherche.
* **Fermer** — ferme la boîte de dialogue.
* Si la recherche se termine sans résultat, la boîte de dialogue affiche **« Aucune caméra trouvée sur le réseau »** — voir [Dépannage](connecting.md#troubleshooting) ci-dessous.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>La boîte de dialogue « Connecter une ou plusieurs caméras » — illustrée ici en l’absence de caméras sur le réseau</p></figcaption></figure>

### Première connexion : téléchargement du pack d’étalonnage

La **première fois** qu’une caméra donnée est connectée à un ordinateur, Chloros récupère le pack d’étalonnage d’usine de la caméra (\~3,8 Mo) directement depuis la caméra via GigE. Pendant cette opération, la boîte de dialogue affiche un panneau vert **« Téléchargement des données d’étalonnage depuis la caméra »**avec une barre de progression par numéro de série — comptez environ**70 secondes** par caméra. S’il y a plus de six caméras, les lignes de progression défilent à l’intérieur du panneau. Le pack étant mis en cache sur l’hôte, les connexions ultérieures de la même caméra ignorent complètement le téléchargement (et le panneau ne s’affiche jamais).

### Analyser le système

Le bouton **Analyser le système** de la boîte de dialogue interroge l’hôte et le réseau (le libellé indique « Analyse en cours… » pendant l’exécution) et génère un rapport de diagnostic :

* **Hôte** — Cœurs de processeur et mémoire vive ; nom et mémoire du GPU, ou « GPU : aucun détecté ».
* **Interfaces réseau** — nom de chaque carte réseau, débit de liaison, MTU (avec la mention « jumbo » lorsqu’il est actif), état (actif/inactif) et indication si elle est connectée à un bus USB.
* **Caméras**— numéro de série, modèle, adresse IP et**carte réseau sur laquelle chaque caméra est connectée**.
* **Performances** — nombre d’images par seconde (fps) actuel par rapport au nombre idéal pour chaque caméra, en fonction du format de pixels, avec une ligne verte indiquant « Potentiel : amélioration de N× possible » lorsque la valeur idéale dépasse la valeur actuelle.
* **Avertissements et recommandations numérotées** — ou « Le système semble fonctionner correctement pour le nombre actuel de caméras. » lorsqu’il n’y a rien à corriger.

Lancez cet outil dès que la détection ou la diffusion en continu présente un comportement inattendu — il identifie la plupart des problèmes liés aux cartes réseau (MTU incorrect, caméra sur la mauvaise interface, limites de l’adaptateur USB) sans quitter la boîte de dialogue.

### Connexion d’un ensemble

Pour connecter deux caméras ou plus en tant qu’**ensemble synchronisé**, utilisez plutôt l’assistant de connexion d’ensemble (**Connecter un ensemble de caméras**) : il vous guide à travers la sélection maître/esclave (préremplie par une détection du câble de synchronisation — toute caméra câblée peut être maître, et la caméra RGB `-FRGB` est présélectionnée lorsqu’elle est câblée), le choix du mode d’affichage (mosaïque séparée ou combinée), ainsi qu’une scène de configuration du réseau avec une projection en temps réel du nombre d’images par seconde (fps) et de la bande passante disponible avant de valider. L’assistant et les workflows de réseau de caméras sont décrits dans la section consacrée aux réseaux de caméras multiples de ce manuel ; l’équivalent en CLI est le « Workflow de première connexion des caméras LATTICE » dans la [Référence CLI](../reference/cli-reference.md).

## Connexion depuis la CLI et le SDK
L’accès à la

CLI et au SDK nécessite un forfait Chloros+ payant et une connexion ; cette condition est appliquée côté serveur (`401 AUTH_REQUIRED` en l’absence de connexion, `403 PLAN_UPGRADE_REQUIRED` sur le forfait gratuit).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Signatures complètes, options et workflows de capture : [Référence CLI](../reference/cli-reference.md) § `chloros-cli lattice`, [Référence SDK](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Comment l’étalonnage est-il effectué lors de la connexion ?

Chaque caméra LATTICE intègre son pack d’étalonnage d’usine **en local**, et Chloros vérifie également le cloud d’MAPIR lorsque la caméra se connecte :

| Situation   | Ce qu’utilise Chloros                                                                                                                                                                                                           |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **En ligne**| Le**dernier étalonnage publié pour ce numéro de série** — la copie du cloud prévaut sur celle intégrée à la caméra. Une caméra qui a été réétalonnée ou mise à jour par MAPIR se met donc automatiquement à jour ; aucune intervention de l’utilisateur n’est nécessaire. |
| **Hors ligne**| Le**pack intégré à l’appareil photo**, tel quel. Les flux de travail entièrement hors ligne continuent de fonctionner ; ils ne prennent simplement pas en compte les calibrations plus récentes tant que l’appareil photo n’est pas connecté une fois (ou réinitialisé aux paramètres d’usine).                                                  |

Au moment de la capture, les coefficients effectivement appliqués sont **enregistrés de manière définitive dans les métadonnées XMP de chaque image**. Une mise à jour ultérieure de l’étalonnage ne modifie jamais en silence les images que vous avez déjà capturées — le retraitement d’une ancienne capture utilise les coefficients enregistrés dans ses métadonnées XMP, et non les plus récents à ce jour.

## Dépannage

* **«Aucune caméra détectée sur le réseau »**— vérifiez la configuration locale de la liaison dans [Configuration réseau](https://mapir.gitbook.io/lattice-camera/setup/network-setup) : carte réseau hôte statique `169.254.x.x/16`, caméras sur la même liaison, pas de DHCP ni de passerelle attendus. Utilisez ensuite**Analyser le système**dans la boîte de dialogue de connexion pour vérifier sur quelle carte réseau chaque caméra est (ou n’est pas) visible. Effectuez un**nouveau balayage** après tout changement de câblage ou de carte réseau.
* **Une installation qui fonctionnait auparavant refuse de se connecter** (panneaux de matrice bloqués avec `FRAMES WILL DROP` / `Reduce ROI to enable`) — une mise à jour du pilote de carte réseau a réinitialisé en arrière-plan les paramètres du circuit de réception. Réappliquez-les ou exécutez la commande `chloros-cli lattice network --fix` depuis un terminal avec des privilèges administrateur ; voir [Configuration réseau](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Une caméra affiche « In Array »** — elle fait partie d’une session de réseau connectée. Déconnectez le réseau pour utiliser la caméra en mode autonome.
