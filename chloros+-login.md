# Connexion à Chloros+

## Connexion à Chloros et Chloros (navigateur)

Le <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vous permet de vous connecter à votre compte Chloros+ et de débloquer des fonctionnalités supplémentaires.

Une fois connecté, les détails de votre compte s&#x27;afficheront :

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## Connexion à CLI

Connectez-vous avec vos identifiants Chloros+ pour activer le traitement CLI. Sur Linux (sans interface graphique), c&#x27;est le seul moyen d&#x27;activer votre licence.

**Syntaxe :**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utilisateurs de SDK** : Python SDK fournit également une méthode programmatique `logout()` pour effacer les identifiants mis en cache. Consultez la [documentation](api-python-sdk.md#logout) pour plus de détails.
{% endhint %}

**Exemple :**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caractères spéciaux** : placez les mots de passe contenant des caractères tels que `$`, `!` ou des espaces entre guillemets simples.
{% endhint %}

**Résultat :**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Stockage des identifiants

Les identifiants mis en cache sont stockés dans un emplacement spécifique à la plateforme :

| Plateforme | Chemin d&#x27;accès au cache des identifiants |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Expiration du forfait

La date d&#x27;expiration du forfait indiquée dans l&#x27;interface graphique indique quand votre licence deviendra invalide. Pour les abonnements mensuels récurrents, l&#x27;expiration a lieu à la fin du mois. Pour les abonnements annuels, elle a lieu un an après le début de l&#x27;abonnement. La vérification de la licence nécessite une connexion Internet mensuelle, avec un délai de grâce de 30 jours.

### Limite d&#x27;appareils

Chaque forfait Chloros+ offre un nombre différent d&#x27;appareils enregistrés. Chaque appareil sur lequel vous vous connectez avec un compte Chloros+ sera comptabilisé dans votre nombre d&#x27;appareils enregistrés. Vous pouvez renommer et supprimer un appareil sur la page de votre compte MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Forfait Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">ARGENT</th><th align="center">OR</th></tr></thead><tbody><tr><td align="right">Appareils pris en charge</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
