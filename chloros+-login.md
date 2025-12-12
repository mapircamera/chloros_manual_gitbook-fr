# Chloros+ Login

## Chloros et Chloros (Navigateur) Login

Le menu latéral utilisateur <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vous permet de vous connecter à votre compte Chloros+ et de débloquer des fonctionnalités supplémentaires.

Lorsque vous êtes connecté, les détails de votre compte s'affichent :

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI Connexion

Connectez-vous avec vos Chloros+ informations d'identification pour permettre le traitement CLI.

**Syntaxe:**

```bash
chloros-cli login <email> <password>
```

**Exemple:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caractères spéciaux** : Utilisez des guillemets simples autour des mots de passe contenant des caractères tels que `$`, `!`, ou des espaces.
{% endhint %}

**Sortie:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Expiration du plan

L'expiration du plan dans l'interface graphique indique la date à laquelle votre licence deviendra invalide. Pour les abonnements mensuels récurrents, l'expiration a lieu à la fin du mois. Pour les abonnements annuels, l'expiration intervient un an après le début de l'abonnement. La vérification de la licence nécessite une connexion internet mensuelle, avec un délai de grâce de 30 jours.

### Limite d'appareils

Chaque plan Chloros+ offre un nombre différent d'appareils enregistrés. Chaque appareil auquel vous vous connectez avec un compte Chloros+ sera pris en compte dans le calcul de votre nombre d'appareils enregistrés. Vous pouvez renommer et supprimer un appareil sur la page de votre compte MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ Plan</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Devices Supported</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
