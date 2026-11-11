# Connexion à Chloros+

## Connexion via l&#x27;interface graphique

Le menu latéral de l&#x27;utilisateur <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vous permet de vous connecter à votre compte Chloros+ et de débloquer des fonctionnalités supplémentaires.

**Vous ne devez vous connecter qu&#x27;une seule fois par machine.** L&#x27;interface graphique, la CLI et le SDK Python partagent la même session mise en cache : la connexion via l&#x27;interface graphique de bureau active également la CLI et le SDK sur cette machine (et inversement via `chloros-cli login`).

Une fois connecté, les détails de votre compte s&#x27;affichent :

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Niveaux de forfait

| Forfait | `plan_id` | Type |
| --- | --- | --- |
| Iron | `0` | Gratuit |
| Copper | `1` | Payant (Chloros

+) |
| Bronze | `2` | Payant (Chloros

+) |
| Silver | `3` | Payant (Chloros

+) |
| Or | `4` | Payant (Chloros

+) |

Consultez la page [Formules et tarifs](https://cloud.mapir.camera/pricing) pour connaître le contenu de chaque formule payante.

### L&#x27;accès à la CLI

et au SDK

nécessite une formule payante

L&#x27;accès à la CLI

et au SDK

Python

en SDK

nécessite **n&#x27;importe quel niveau Chloros

+ payant (Copper ou supérieur)**. Cette règle est appliquée**côté serveur** : chaque requête CLI

/SDK

doit comporter à la fois une session active et un forfait payant :

| Statut HTTP

| `error_code` | Signification | Solution |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Non connecté sur cette machine | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Connecté, mais le niveau de forfait est trop bas (niveau Iron gratuit) | Passez à n&#x27;importe quel forfait payant Chloros

+ |

`chloros-cli status` reste accessible dans le cadre de la formule gratuite ; vous pouvez donc toujours consulter votre formule actuelle et la raison pour laquelle l&#x27;accès vous est refusé.

### Limites de matériel connecté par formule

Chaque formule limite le nombre de caméras LATTICE et de capteurs de lumière DAQ pouvant être connectés en direct simultanément :

| Formule | Caméras LATTICE | Capteurs de lumière DAQ |
| --- | --- | --- |
| Iron (gratuit / non connecté) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Connexion à l’CLI



Connectez-vous avec vos identifiants Chloros

+ pour activer le traitement via l’CLI

. Sous Linux

(sans interface graphique), c’est le seul moyen d’activer votre licence.

**Syntaxe :**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utilisateurs du SDK** : le Python

SDK fournit également une méthode programmatique `logout()` permettant d’effacer les identifiants mis en cache. Consultez la [référence du SDK

](reference/sdk-reference.md) pour plus de détails.
{% endhint %}

**Exemple :**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caractères spéciaux** : placez les mots de passe contenant des caractères tels que `$`, `!` ou des espaces entre des guillemets simples.
{% endhint %}

**Résultat :**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Stockage des identifiants

Les identifiants et la configuration mis en cache sont stockés dans le dossier `.chloros` de votre répertoire personnel sur **toutes les plateformes** :

| Plateforme | Chemin d’accès au cache des identifiants |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

L&#x27;installation d&#x27;une version plus récente par-dessus une version existante conserve vos identifiants de connexion, sous Windows

(le programme d&#x27;installation) et sous Linux

(le dossier `.deb`, y compris `chloros-cli update`). La désinstallation de Chloros

— ou la suppression du paquet Linux

— efface les identifiants mis en cache ; vous devrez donc vous reconnecter après une nouvelle installation.

### Expiration de l&#x27;abonnement et délai de grâce hors ligne

La date d&#x27;expiration de l&#x27;abonnement indiquée dans l&#x27;interface graphique indique quand votre licence deviendra invalide. Pour les abonnements mensuels récurrents, l’expiration a lieu à la fin du mois ; pour les abonnements annuels, elle intervient un an après le début de l’abonnement.

Chloros

valide votre licence en ligne, mais le fonctionnement hors ligne est pris en charge pendant une période de grâce :

* Les validations réussies auprès du serveur sont mises en cache pendant **5 minutes**, ce qui signifie qu’une utilisation normale ne nécessite que très peu d’appels de licence.
* Un cache de licence signé et lié à la machine couvre des périodes hors ligne plus longues : **30 jours pour les forfaits mensuels**, et**jusqu’à la date d’expiration de votre abonnement (365 jours au maximum) pour les forfaits annuels**.
* À l’expiration de la période de grâce, le forfait bascule vers le niveau gratuit « Iron » jusqu’à ce que l’appareil puisse se connecter au serveur de licences ; l’accès reprend dès la prochaine vérification réussie.

### Limite d’appareils

Chaque forfait Chloros

+ offre un nombre différent d’appareils enregistrés. Chaque appareil sur lequel vous vous connectez avec un compte Chloros

+ est comptabilisé dans votre nombre d’appareils enregistrés. Vous pouvez renommer et supprimer un appareil depuis la page de votre compte MAPIR

Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Formule Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Appareils pris en charge</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Le nombre exact d&#x27;appareils autorisés pour votre compte est indiqué sur la page de votre compte MAPIR Cloud. La déconnexion d&#x27;un appareil libère systématiquement son emplacement, et un appareil déjà enregistré peut toujours se reconnecter, même lorsque le compte a atteint sa limite d&#x27;appareils.
