# Connexion à Chloros+

## Connexion via l&#x27;interface graphique

Le menu latéral de l&#x27;<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">e utilisateur vous permet de vous connecter à votre compte Chloros+ et de débloquer des fonctionnalités supplémentaires.

**Vous n’avez besoin de vous connecter qu’une seule fois par machine.** L’interface graphique, CLI et Python SDK partagent la même session mise en cache : la connexion via l’interface graphique de bureau active également les applications CLI et SDK sur cette machine (et inversement via `chloros-cli login`).

Une fois connecté, les détails de votre compte s&#x27;affichent :

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Niveaux de forfait

| Forfait | `plan_id` | Type |
| --- | --- | --- |
| Iron | `0` | Gratuit |
| Copper | `1` | Payant (Chloros+) |
| Bronze | `2` | Payant (Chloros+) |
| Argent | `3` | Payant (Chloros+) |
| Or | `4` | Payant (Chloros+) |

Consultez la page [Formules et tarifs](https://cloud.mapir.camera/pricing) pour connaître les avantages de chaque niveau payant.

### L&#x27;accès à CLI / SDK nécessite un niveau d&#x27;abonnement payant

L&#x27;accès à CLI, Python et SDK nécessite **n&#x27;importe quel niveau payant Chloros+ (Copper ou supérieur)**. Cette règle est appliquée**côté serveur** : chaque requête CLI/SDK doit comporter à la fois une session active et un forfait payant :

| Statut HTTP | `error_code` | Signification | Solution |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Non connecté sur cette machine | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Connecté, mais le niveau de forfait est trop bas (niveau Iron gratuit) | Passez à n’importe quel forfait Chloros+ payant |

`chloros-cli status` reste accessible dans le niveau gratuit ; vous pouvez donc toujours consulter votre forfait actuel et la raison pour laquelle l’accès vous est refusé.

### Limites de matériel connecté par forfait

Chaque forfait limite le nombre de caméras LATTICE et de capteurs de lumière DAQ pouvant être connectés en direct simultanément :

| Formule | Caméras LATTICE | Capteurs de lumière DAQ |
| --- | --- | --- |
| Iron (gratuit / non connecté) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Connexion à CLI

Connectez-vous avec vos identifiants Chloros+ pour activer le traitement CLI. Sur Linux (sans interface graphique), c’est le seul moyen d’activer votre licence.

**Syntaxe :**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utilisateurs de SDK** : La fonction Python SDK fournit également une méthode programmatique `logout()` permettant d&#x27;effacer les identifiants mis en cache. Pour plus de détails, consultez la [Référence SDK](reference/sdk-reference.md).
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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Stockage des identifiants

Les identifiants et la configuration mis en cache sont stockés dans le dossier `.chloros` de votre répertoire personnel sur **toutes les plateformes** :

| Plateforme | Chemin d&#x27;accès au cache des identifiants |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Expiration du forfait et délai de grâce hors ligne

La date d’expiration du forfait affichée dans l’interface graphique indique quand votre licence cessera d’être valide. Pour les abonnements mensuels récurrents, l’expiration intervient à la fin du mois ; pour les abonnements annuels, elle intervient un an après le début de l’abonnement.

Chloros valide votre licence en ligne, mais l&#x27;utilisation hors ligne est prise en charge pendant un délai de grâce :

* Les validations réussies auprès du serveur sont mises en cache pendant **5 minutes** ; ainsi, dans le cadre d&#x27;une utilisation normale, les appels de licence sont très peu nombreux.
* Un cache de licence signé et lié à la machine couvre des périodes hors ligne plus longues : **30 jours pour les forfaits mensuels**, et**jusqu’à la date d’expiration de votre abonnement (365 jours au maximum) pour les forfaits annuels**.
* À l’expiration de la période de grâce, le forfait bascule vers le niveau gratuit « Iron » jusqu’à ce que l’ordinateur puisse se connecter au serveur de licences ; l’accès reprend dès la prochaine vérification réussie.

### Limite d’appareils

Chaque forfait Chloros+ offre un nombre différent d’appareils enregistrés. Chaque appareil sur lequel vous vous connectez avec un compte Chloros+ est comptabilisé dans votre nombre d&#x27;appareils enregistrés. Vous pouvez renommer et supprimer un appareil sur la page de votre compte MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Formule Chloros+</th><th align="center">CUIVRE</th><th align="center">BRONZE</th><th align="center">ARGENT</th><th align="center">OR</th></tr></thead><tbody><tr><td align="right">Appareils pris en charge</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Le nombre exact d&#x27;appareils autorisés pour votre compte est indiqué sur la page de votre compte MAPIR Cloud. La déconnexion d&#x27;un appareil libère systématiquement son emplacement, et un appareil déjà enregistré peut toujours se reconnecter, même lorsque le compte a atteint sa limite d&#x27;appareils.
