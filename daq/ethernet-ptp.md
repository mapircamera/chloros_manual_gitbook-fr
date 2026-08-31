# Mise en réseau et synchronisation horaire du DAQ-E

> La configuration physique du réseau pour le capteur — câblage, PoE, attribution d&#x27;adresse IP et paramètres réseau propres à l&#x27;appareil — est décrite dans le **[manuel d&#x27;utilisation du DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Cette page traite des aspects liés au Chloros : connexion, synchronisation horaire et marche à suivre en cas d’échec de la détection.

Le DAQ-E est le membre « Ethernet » de la famille DAQ : alimenté par PoE, détecté via mDNS (service `_daq-e._tcp`) et accessible par un nom d’hôte dérivé de son identifiant de capteur — `daq-e-<6 hex>.local`, par exemple `daq-e-def330.local`. Cette page explique comment il transfère les données sur le réseau et comment il participe à la synchronisation temporelle PTP.

## Modes de transport

| Mode | Point d’extrémité | Utilisateurs | Remarques |
| --- | --- | --- | --- |
| **Multidiffusion** (par défaut) | UDP `239.10.10.10:5002` | N&#x27;importe quel appareil sur le même réseau local reçoit le même flux | Chaque datagramme est validé par CRC-16/CCITT |
| **Brut** | Port TCP `5000` | Un seul client (exclusif) | Compatibilité octet à octet avec DAQ-U |

Chloros utilise la multidiffusion par défaut, ce qui permet à l&#x27;interface graphique, à CLI et à SDK de surveiller tous ensemble un même capteur.

## Configuration réseau requise

* **Même domaine de diffusion.** La machine exécutant Chloros doit se trouver sur le même segment de réseau de couche 2 que le capteur — la découverte mDNS ne traverse pas les routeurs.
* **Invite du pare-feu Windows : acceptez-la.** La première fois que Chloros lie les sockets multicast, Windows Defender demande l’autorisation une seule fois. L&#x27;autorisation couvre les données DAQ-E (UDP 5002), mDNS (UDP 5353) et PTP (UDP 319/320). Sur Linux, cette opération s&#x27;effectue en arrière-plan.
* **Alimentation PoE, pas de LED d&#x27;état.** Le DAQ-E ne dispose pas de LED propre — vérifiez l&#x27;alimentation via le voyant Link/PoE sur le commutateur ou le port de l&#x27;injecteur, et patientez quelques secondes après la mise sous tension pour qu&#x27;il démarre et rejoigne le réseau.

## Connexion

**Interface graphique :** Onglet « Capteurs de lumière » → « Connecter un capteur » → Type de périphérique « DAQ-E (Ethernet) ». La détection ne s’effectue que tant que la boîte de dialogue de connexion est affichée à l’écran (recherche mDNS et balayage ARP sur Windows), et se répète toutes les 15 secondes ; le bouton « Actualiser » relance immédiatement la recherche. Les capteurs détectés apparaissent dans la liste déroulante ; le premier capteur détecté est sélectionné automatiquement.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend en cours d’exécution) :

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hôtes à plusieurs cartes réseau et première connexion après le démarrage

Sur les hôtes disposant de plusieurs interfaces réseau actives, le **premier** `pool-connect --eth` après le démarrage peut s’avérer vide même lorsque le capteur est opérationnel — la recherche de découverte peut ne pas détecter l’interface sur laquelle réside le capteur tant que le cache ARP est encore « froid ». La solution fiable consiste à ignorer la détection et à passer l’adresse explicitement :

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepte le nom d’hôte mDNS ou l’adresse IP, cible toujours le bon capteur et constitue la forme recommandée pour les scripts et les installations sans interface graphique. Dans l’interface graphique, utilisez le bouton « Actualiser » de la boîte de dialogue de connexion et laissez le système effectuer un nouveau cycle de recherche.

## Paramètres du périphérique et micrologiciel

Le capteur lui-même contient les paramètres réseau — adresse IP statique ou DHCP + adressage local de liaison, nom du périphérique, diffusion automatique au démarrage, mot de passe OTA. Ces paramètres côté appareil ne sont pas accessibles sous forme de commandes dans le CLI fourni ; ils sont gérés via l’interface graphique Chloros où ils sont affichés, ou avec l’assistance MAPIR.

**Les mises à jour du micrologiciel sont intégrées à l’interface graphique.**Lorsqu’un DAQ-E connecté utilise un micrologiciel plus ancien que l’image fournie avec votre version Chloros, sa ligne de capteur affiche une bulle orange**Mise à jour disponible**, et la fenêtre contextuelle des paramètres propose un<version>

bouton</version> « Mettre à jour vers<version>

». La mise à jour s’effectue via le réseau en environ 30 secondes ; le capteur redémarre et se reconnecte automatiquement, et une interruption du transfert laisse le micrologiciel actuel intact.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Synchronisation temporelle PTP

Le micrologiciel v1.2.0+ du DAQ-E participe au protocole IEEE 1588 PTPv2 en tant qu’horloge ordinaire (esclave uniquement). **Le backend de l’hôte Chloros fait office de « grand maître » PTP** — chaque DAQ-E et chaque caméra LATTICE du réseau local lui sont asservis dans le domaine 0, ce qui permet de maintenir tous les horodatages des périphériques avec une tolérance d’environ 1 ms. C’est cette horloge partagée qui permet de faire correspondre les horodatages des mesures du DAQ avec les expositions des caméras (voir [Enregistrement et format .daq](recording.md)).

Vérifiez la synchronisation à partir du fichier CLI :

| Commande | Affiche |
| --- | --- |
| `chloros-cli time-sync status` | État du maître principal de l’hôte, priorités BMCA, identité de l’horloge |
| `chloros-cli time-sync peers` | Tous les esclaves détectés (capteurs DAQ-E + caméras LATTICE) |
| `chloros-cli time-sync cameras` | État de santé PTP par caméra (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Redémarrer le processus « grandmaster » |

Dans l’interface graphique, la fenêtre modale des paramètres DAQ-E affiche une ligne **PTP Sync** en temps réel indiquant l’état PTP actuel du capteur.

Détails pour les consommateurs nécessitant un alignement strict :

* Chaque datagramme transmis comporte un champ d’indicateurs ; **le bit 2 est activé sur les trames dont l’horodatage est synchronisé PTP**. Les pipelines nécessitant un alignement strict entre la caméra et le DAQ doivent se baser sur ce bit.
* Avant une capture synchronisée, vérifiez que le capteur apparaît dans `chloros-cli time-sync peers`. (Les outils matériels directs internes de MAPIR peuvent également déclencher l’enregistrement lors du verrouillage PTP à l’aide d’un indicateur `--wait-ptp` qui attend jusqu’à 15 s que le capteur atteigne l’état SLAVE ; cet outil ne fait pas partie de la version livrée de CLI.)
* Tant que le PTP est activement en mode esclave, le capteur refuse les injections manuelles d’horloge (« PTP fournit l’horloge »). C’est voulu — faites confiance au PTP.

## Remarques sur Linux

* **PTP nécessite `libcap2-bin` lors de l’installation.** Le script post-installation `.deb` accorde les droits à `cap_net_bind_service=+ep` sur `/usr/lib/chloros/chloros-backend` afin qu’il puisse lier les ports PTP 319/320 sans être root. Si `libcap2-bin` est absent, cette étape est ignorée et PTP ne parviendra pas à démarrer. Solution :

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi sans interface graphique :** lors de la première installation, l&#x27;unité systemd `chloros-backend.service` est générée mais n&#x27;est pas activée. Pour un PTP toujours actif (et la disponibilité du DAQ) sans interface graphique :

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Sans cela, le PTP ne fonctionne que lorsque l’interface graphique Chloros est ouverte.

## Dépannage : « Aucun périphérique DAQ-E détecté »

| Vérification | Détail |
| --- | --- |
| Alimentation | Aucune LED allumée sur le capteur — vérifiez les voyants PoE et de liaison du commutateur/port d&#x27;injection ; attendez quelques secondes après la mise sous tension |
| Domaine de diffusion | L&#x27;hôte et le capteur se trouvent sur le même segment L2 ; le mDNS ne route pas |
| Pare-feu Windows | Acceptez l’invite de Defender lors du premier lancement (UDP 5002, 5353, 319/320) |
| Hôte à cartes réseau multiples | La première détection après le démarrage peut ne pas repérer le capteur — connectez-vous via `--eth-host <ip-or-hostname>` |
| Nouvelle recherche via l’interface graphique | La détection ne s’exécute que tant que la boîte de dialogue de connexion est ouverte ; utilisez le bouton « Actualiser » |</version>
