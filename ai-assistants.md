# Utilisation d’Chloros avec des assistants IA

Ce manuel s'adresse à deux publics : les humains et les assistants IA avec lesquels les humains travaillent de plus en plus souvent. Chaque page présente des valeurs exactes, des paramètres par défaut et des commandes copiables-collables afin qu'un assistant (Claude, ChatGPT, Copilot, un agent de codage, etc.) puisse créer une automatisation Chloros fonctionnelle dès le premier essai.

Version de Chloros : **

1.3.0**. Plateformes CLI/SDK : Windows 10/11 x64 et Linux (x86_64 / Jetson aarch64).

## Ce qu’il faut fournir à votre assistant

| Ressource | URL | À quoi cela sert |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Index lisible par machine de toutes les pages de ce manuel. |
| **Référence CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | L'ensemble complet des commandes `chloros-cli` : chaque commande, indicateur, valeur par défaut, code de sortie et règle relative au dossier de sortie. Rédigé pour être utilisé par un LLM. |
| **Référence SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | L'API Python complète de `chloros_sdk` : classes, signatures, exceptions et exemples concrets. Rédigée pour être utilisée par les LLM. |
| **N'importe quelle page au format Markdown brut** | ajoutez `.md` à l'URL de la page | par exemple, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` renvoie la page au format Markdown brut — idéal pour la coller dans une fenêtre de contexte ou la récupérer depuis un agent. |

Liens dans le manuel : [Référence CLI](reference/cli-reference.md) · [Référence SDK](reference/sdk-reference.md).

{% hint style="info" %}
Ces deux pages de référence sont autonomes : un assistant qui en a lu une n'a pas besoin du reste du manuel pour écrire un script correct.
{% endhint %}

## Laissez votre assistant exécuter Chloros directement : `chloros-mcp`

Tout ce qui précède demande à votre assistant d’*écrire* un script ou une commande. Avec `chloros-mcp`, un assistant qui utilise le [Model Context Protocol](https://modelcontextprotocol.io) (MCP), tel que Claude Desktop, Claude Code ou Cursor, pilote Chloros lui-même. Vous formulez votre demande en langage naturel et il fait appel aux outils du serveur :

* **Traitement des images :** créer un projet, importer un dossier de captures, choisir les paramètres, traiter les images et générer un rapport sur les résultats de l’exécution.
* **Pilotage du matériel :** connecter des caméras LATTICE, des réseaux de capteurs et des capteurs de lumière DAQ, prévisualiser, régler l’exposition et les couleurs, capturer, enregistrer et mettre à jour le micrologiciel.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Les outils du serveur couvrent les fonctionnalités du [Python SDK](api-python-sdk.md). Ils s’installent avec le SDK ; il n’y a donc généralement rien d’autre à installer : sous Windows, le programme d’installation de Chloros place `chloros-mcp.exe` dans le dossier `Scripts` de votre Python, et sous Linux, `.deb` place `chloros-mcp` dans `/usr/bin`.

**Claude Code :**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop :**ajoutez ceci à `claude_desktop_config.json` (**Paramètres > Développeur > Modifier la configuration**permet d’y accéder), puis redémarrez Claude Desktop.**Cursor** utilise la même entrée dans `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Si le client indique que le serveur n’a pas pu démarrer, `chloros-mcp` ne figure probablement pas dans le fichier `PATH` qu’il détecte : indiquez plutôt le chemin d’accès complet (`where chloros-mcp` sous Windows, `which chloros-mcp` sous Linux).

Chloros doit être installé sur le même ordinateur et vous devez y être connecté, soit via l'application, soit via `chloros-cli login`, avec un abonnement Chloros+ payant (Copper ou supérieur). Le serveur ne peut pas se connecter à votre place.

{% hint style="info" %}
Configuration client pour chaque application, options, chaque outil et comportement d’une exécution : [Serveur MCP (assistants IA)](reference/sdk-reference.md#mcp-server-ai-assistants) dans la référence SDK.
{% endhint %}

## Exemples de commandes

Copiez, complétez le champ `<placeholders>`, puis collez-le dans votre assistant.

### 1. Traiter un dossier de vol en NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Surveiller par lots un répertoire de captures

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Connecter un réseau LATTICE et effectuer une capture

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Enregistrer les spectres d'un capteur de lumière DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Les scripts DAQ depuis la ligne de commande passent toujours par la famille `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Les autres sous-commandes `daq` que votre assistant pourrait inventer ne sont pas disponibles dans les versions livrées et provoquent une erreur.
{% endhint %}

## Pourquoi les scripts écrits par l’IA fonctionnent bien avec Chloros

Chacun de ces comportements est réel et a été vérifié dans Chloros 1.3.0 — ils éliminent les modes d’échec classiques de l’automatisation écrite par des machines :

* **Pas de configuration fastidieuse.**Les aides « smart-connect » du SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) et les points d’entrée de traitement (`ChlorosLocal`, `process_folder`)**lancent automatiquement le backend local**. Un script généré n’a pas besoin que l’interface graphique soit ouverte ni qu’un serveur soit démarré manuellement — il suffit que le package pour bureau ou CLI soit installé.
* **L’ensemble du pipeline se résume à un seul appel.** `chloros_sdk.process_folder("path", indices=["NDVI"])` exécute de bout en bout les étapes suivantes : importation → étalonnage → réflectance → exportation de l’indice. Moins de surface d’intervention, moins de points de défaillance potentiels pour un script généré.
* **Les exécutions sans résultat font l’objet d’un autodiagnostic.** Après `process()`, le résumé de l’exécution est joint au résultat, et chaque indication relative au traitement (par exemple, *pourquoi* une exécution n’a produit aucun résultat) est également réémise sous forme de script Python `UserWarning` — ainsi, même un script qui n’inspecte jamais le dictionnaire de résultats affiche le diagnostic.
* **L’CLI signale clairement les échecs.**Une exécution `chloros-cli process` qui a demandé des produits mais n’en a écrit aucun affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie non nul**, ce qui permet aux scripts shell et à l’infrastructure de continuité (CI) de la détecter par une simple vérification du code de sortie. Les exécutions réussies renvoient `Image products written: N`, ainsi que `(plus M clamp masks)` lorsque des masques ont été écrits. De même, les commandes de capture nomment chaque fichier qu’elles n’ont pas pu écrire et se terminent avec le code `1`.
* **Les chemins relatifs correspondent à ce qu’attend le script.** Avec un backend local, un chemin de sortie relatif (`-o out/` dans l’CLI, `output_dir="out/"` dans le SDK) est résolu par rapport au dossier à partir duquel la commande ou le script s’exécute.

Une asymétrie qu’un assistant doit connaître : la commande `process()` de l’SDK ne déclenche délibérément **pas** d’exception lors d’une exécution sans résultat — elle signale l’erreur via le résumé et les conseils à la place. Si un pipeline Python doit s’arrêter lors d’une exécution vide, vérifiez le résumé (c’est ce que fait la recette 2).

## Mises en garde

* **Connexion Chloros+ requise.**L’CLI et le SDK nécessitent un niveau d’abonnement**payant** à Chloros+, contrôlé côté serveur : les requêtes échouent avec l’erreur `401 AUTH_REQUIRED` si vous n’êtes pas connecté et avec l’erreur `403 PLAN_UPGRADE_REQUIRED` sur le niveau gratuit. Exécutez `chloros-cli login` une fois par machine avant d’exécuter les scripts générés. Voir [Connexion à Chloros+](chloros+-login.md).
* **Les commandes de capture pilotent du matériel physique.** Les commandes `lattice` / `daq` / `project`, les objets de session du SDK et les outils de `chloros-mcp` permettent de se connecter à des caméras et capteurs physiques, de diffuser des flux et de les déclencher. Vérifiez un script généré avant sa première exécution, puis exécutez-le en présence du matériel.
* **Effectuez des contrôles ponctuels des résultats.** Vérifiez les dossiers de sortie et quelques valeurs de pixels avant de publier les résultats. En particulier, les fichiers TIFF de réflectance sont mis à l’échelle par source — consultez la balise XMP `Chloros:PixelScale` (les fichiers LATTICE la contiennent : 32768 = réflectance 1,0 ; ce n’est pas le cas des fichiers Survey3, qui utilisent 65535 = 1,0) au lieu de supposer un diviseur. Les deux pages de référence documentent cela sous la rubrique « Lecture des pixels de réflectance ».
* **Petits pièges qui peuvent perturber le code généré :**`pool-record` écrit sur le système de fichiers de l’**hôte backend** (par défaut `~/Documents/DAQ Live View/`) ; sur les machines disposant de plusieurs interfaces réseau, privilégiez `daq pool-connect --eth-host <ip-or-hostname>` plutôt que la détection automatique ; utilisez `http://127.0.0.1:5000` (jamais `localhost`) partout où apparaît une URL de backend ; un script parcourant l'arborescence de sortie d'un projet doit ignorer les dossiers `clamp_masks/` (leurs fichiers `*_clampmask.tif` sont des masques, pas des images — voir [Formats d'images de sortie](output-image-formats.md#clamp-masks)) ; et sous Windows, lorsque les chemins d'accès longs sont désactivés, veillez à ce que les noms de dossiers de projet restent courts — un dossier de projet dont le nom dépasse environ 130 caractères est rejeté avant le début du traitement.
