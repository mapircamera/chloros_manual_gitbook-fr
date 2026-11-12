# Mise en réseau et synchronisation horaire du DAQ-E

> La configuration physique du réseau pour le capteur — câblage, PoE, attribution d'adresse IP et paramètres réseau propres à l'appareil — est décrite dans le **[manuel d'utilisation du DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Cette page traite des aspects liés à Chloros : connexion, synchronisation horaire et marche à suivre en cas d’échec de la détection.

Le DAQ-E est le membre « Ethernet » de la famille DAQ : alimenté par PoE, détecté via mDNS (service `_daq-e._tcp`) et accessible par un nom d’hôte dérivé de son identifiant de capteur — `daq-e-<6 hex>.local`, par exemple `daq-e-def330.local`. Cette page explique comment il transfère les données sur le réseau et comment il participe à la synchronisation temporelle PTP.

## Modes de transport

| Mode | Point d’extrémité | Utilisateurs | Remarques |
| --- | --- | --- | --- |
| **Multidiffusion** (par défaut) | UDP `239.10.10.10:5002` | N'importe quel appareil sur le même réseau local reçoit le même flux | Chaque datagramme est validé par CRC-16/CCITT |
| **Brut** | Port TCP `5000` | Un seul client (exclusif) | Compatibilité octet à octet avec DAQ-U |

Chloros utilise la multidiffusion par défaut, ce qui permet à l’interface graphique, à la CLI et au SDK de surveiller tous ensemble un même capteur.

## Configuration réseau requise

* **Même domaine de diffusion.** La machine exécutant Chloros doit se trouver sur le même segment de réseau de couche 2 que le capteur — la découverte mDNS ne traverse pas les routeurs.
* **Pare-feu.** Sous Windows, le programme d’installation ajoute la règle de pare-feu **Chloros Backend (UDP-In)**pour le backend, sur chaque profil réseau, qui couvre les données DAQ-E (UDP 5002), le mDNS (UDP 5353) et le PTP (UDP 319/320), ainsi qu’une règle**« Chloros Backend (TCP-In) »**, limitée au sous-réseau local, pour la connexion que le DAQ-E établit vers l’ordinateur lors d’une mise à jour du micrologiciel. Sous Linux, le fichier `.deb` ouvre les ports UDP dans `ufw` ou `firewalld` lorsque ce pare-feu est actif au moment de l’installation. Un pare-feu tiers, ou un pare-feu activé ultérieurement, nécessite que ces mêmes ports soient autorisés.
* **Alimentation PoE, pas de LED d’état.** Le DAQ-E ne dispose pas de LED propre — vérifiez l’alimentation via le voyant « Link/PoE » du commutateur ou du port de l’injecteur, et patientez quelques secondes après la mise sous tension pour qu’il démarre et se connecte au réseau.

## Connexion

**Interface graphique :** Onglet « Capteurs de lumière » → « Connecter un capteur » → Type d’appareil « DAQ-E (Ethernet) ». La détection ne s’effectue que tant que la boîte de dialogue de connexion est affichée à l’écran (recherche mDNS et balayage ARP sous Windows), et se répète toutes les 15 secondes ; le bouton « Actualiser » relance immédiatement la recherche. Les capteurs détectés apparaissent dans la liste déroulante ; le premier capteur détecté est sélectionné automatiquement.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend en cours d’exécution) :

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Un DAQ-E déjà présent dans le pool est reconnu aussi bien par son nom d’hôte que par son adresse IP : le reconnecter sous une autre adresse renvoie l’entrée existante du pool plutôt que d’ouvrir l’unité une deuxième fois.

### Hôtes à plusieurs cartes réseau et première connexion après le démarrage

Sur les hôtes disposant de plusieurs interfaces réseau actives, la **première** connexion `pool-connect --eth` après le démarrage peut s’avérer infructueuse même si le capteur est opérationnel — la recherche de découverte peut ne pas détecter l’interface sur laquelle réside le capteur tant que le cache ARP n’est pas encore chargé. La solution fiable consiste à ignorer la découverte et à passer l’adresse explicitement :

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepte le nom d’hôte mDNS ou l’adresse IP, cible toujours le bon capteur et constitue la méthode recommandée pour les scripts et les installations sans interface graphique. Dans l’interface graphique, utilisez le bouton « Actualiser » de la boîte de dialogue de connexion et laissez le système effectuer un nouveau cycle de recherche.

## Paramètres de l’appareil et micrologiciel

Le capteur conserve ses propres paramètres réseau : DHCP (réglage d’usine, avec repli sur une adresse locale de liaison 169.254.x.x en l’absence de réponse d’un serveur DHCP) ou une adresse statique. Il conserve également l’option indiquant s’il commence la diffusion dès la mise sous tension. Modifiez ces paramètres dans l’interface graphique (icône en forme d’engrenage des paramètres du capteur, **Réseau**:**Modifier…**,**Diffusion à la mise sous tension**,**Redémarrer le capteur** ; voir [L’onglet DAQ](gui.md)), ou depuis l’CLI et le SDK avec `pool-set-network`, `pool-set-auto-stream` et `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()` ; voir la [Référence CLI](../reference/cli-reference.md#daq-e-device-settings)). Le capteur redémarre pour appliquer une modification du réseau, et Chloros le reconnecte. Une adresse statique doit figurer sur le réseau de l’ordinateur exécutant Chloros : Chloros refuse toute adresse qu’il n’a pas pu atteindre après le redémarrage. L’ID du capteur et son nom d’hôte `daq-e-<id>.local` ne changent jamais. Nouveauté de la version 1.3.0.

**Les mises à jour du micrologiciel sont intégrées à l’interface graphique.** Lorsqu’un DAQ-E connecté utilise une version du micrologiciel antérieure à celle fournie avec votre version d’Chloros, la ligne correspondant à son capteur affiche une pastille orange **Mise à jour disponible**, et la fenêtre modale des paramètres (icône en forme d’engrenage) propose un <version>bouton</version>

« Mettre à jour vers <version>». La mise à jour s’effectue via le réseau en environ 30 secondes ; le capteur redémarre et se reconnecte automatiquement, et une interruption du transfert laisse le micrologiciel actuel intact. Depuis la CLI ou le SDK, et sous Linux (qui ne dispose pas d’interface graphique), exécutez `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Synchronisation temporelle PTP

Le micrologiciel v1.2.0+ du DAQ-E participe au protocole IEEE 1588 PTPv2 en tant qu’horloge ordinaire (esclave uniquement). **Le backend de l’hôte Chloros est le grand maître PTP** — chaque DAQ-E et chaque caméra LATTICE sur le réseau local lui sont asservis dans le domaine 0, ce qui permet de maintenir tous les horodatages des périphériques dans une tolérance d’environ 1 ms. C’est cette horloge partagée qui permet de faire correspondre les horodatages des lectures DAQ aux expositions des caméras (voir [Enregistrement et format .daq](recording.md)).

Vérifiez la synchronisation depuis l’CLI :

| Commande | Affiche |
| --- | --- |
| `chloros-cli time-sync status` | État du maître de l’horloge hôte, priorités BMCA, identité de l’horloge, et `health` : `OK`, ou `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` avec un `health_detail` indiquant pourquoi ses paquets n’atteignent pas le réseau |
| `chloros-cli time-sync peers` | Tous les esclaves détectés (capteurs DAQ-E + caméras LATTICE) |
| `chloros-cli time-sync cameras` | État de santé PTP par caméra (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Redémarrer le processus « grandmaster » |

Dans l’interface graphique, la fenêtre modale des paramètres DAQ-E affiche une ligne **PTP Sync** en temps réel indiquant l’état PTP actuel du capteur.**Un DAQ-E dont le PTP est désactivé reste désactivé.** L'appareil conserve son paramètre d'activation/désactivation du PTP dans sa propre mémoire ; ainsi, un appareil dont le PTP a été désactivé reste désactivé après une reconnexion ou un redémarrage, et ses horodatages ne sont pas synchronisés via PTP. `chloros-cli daq pool-list` signale un tel appareil (`!! PTP is switched OFF on this unit …`) et affiche la commande permettant de le corriger :

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros ne réactive jamais un appareil de lui-même, car la désactivation peut être intentionnelle.

Détails pour les utilisateurs exigeant un alignement strict :

* Chaque datagramme transmis comporte un champ « flags » ; **le bit 2 est activé sur les trames dont l’horodatage est synchronisé PTP**. Les pipelines nécessitant un alignement strict entre caméra et DAQ doivent se baser sur ce bit.
* Avant une capture synchronisée, vérifiez que le capteur apparaît dans `chloros-cli time-sync peers` et que `chloros-cli daq pool-list` n’indique pas que son PTP est désactivé. (Les outils matériels directs internes de MAPIR peuvent également déclencher l’enregistrement lors du verrouillage PTP grâce à un indicateur `--wait-ptp` qui attend jusqu’à 15 s que le capteur atteigne l’état SLAVE ; ces outils ne font pas partie de l’interface CLI fournie.)
* Tant que le PTP est activement en mode esclave, le capteur refuse les injections manuelles d’horloge (« PTP fournit l’horloge »). C’est voulu — faites confiance au PTP.

## Remarques concernant Linux

* **Ports PTP sans droits root.** Le PTP utilise les ports UDP 319/320. Le fichier `.deb` écrit dans `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), de sorte que le backend s’y lie en s’exécutant sous votre compte utilisateur. À partir de la version 1.3.0, le paquet ne définit aucune capacité de fichier (`setcap`) sur le backend et n’a pas besoin de `libcap2-bin` ; n’ajoutez pas de capacité vous-même, car le backend décompressé ne démarrera alors pas. Si PTP ne parvient pas à se lier au port 319, consultez la section [Installation sous Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson sans écran / hôte Linux :** le paquet installe les services du backend mais n’en active aucun. Pour un PTP toujours actif (et la disponibilité du DAQ) sans que personne n’exécute de commande, activez le service en tant qu’utilisateur :

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Sans cela, PTP ne fonctionne que si le backend a été lancé par une commande `chloros-cli` ou un script SDK. Consultez la [Installation sous Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) pour en savoir plus sur l’utilisateur root `chloros-backend.service` et sur la manière de passer d’un utilisateur à l’autre.

## Dépannage : « Aucun périphérique DAQ-E détecté »

| Vérification | Détails |
| --- | --- |
| Alimentation | Aucune LED allumée sur le capteur — vérifiez les voyants PoE et de liaison du commutateur/port de l’injecteur ; attendez quelques secondes après la mise sous tension |
| Domaine de diffusion | L’hôte et le capteur se trouvent sur le même segment L2 ; le mDNS ne s’achemine pas |
| Pare-feu | Windows : la règle **Chloros Backend (UDP-In)** du programme d’installation autorise le flux DAQ-E (UDP 5002), le mDNS (5353) et le PTP (319/320) ; un pare-feu tiers doit être configuré de la même manière. Linux : le fichier `.deb` ouvre ces ports dans `ufw` / `firewalld` uniquement si le pare-feu était actif au moment de l’installation — autorisez-les vous-même si le pare-feu a été activé ultérieurement |
| Hôte multi-carte réseau | La découverte parcourt le mDNS avec une solution de secours ARP ; `--eth-host <ip-or-hostname>` ignore la découverte et se connecte directement |
| Nouvelle analyse via l’interface graphique | La découverte ne s’exécute que tant que la boîte de dialogue de connexion est ouverte ; utilisez « Actualiser » |
