# Utilisation de Chloros avec des assistants IA

Ce manuel s’adresse à deux publics : les humains et les assistants IA avec lesquels les humains travaillent de plus en plus. Chaque page présente des valeurs exactes, des paramètres par défaut et des commandes copiables-collables afin qu’un assistant (Claude, ChatGPT, Copilot, un agent de programmation, etc.) puisse créer une automatisation Chloros fonctionnelle dès le premier essai.

Version de Chloros : **

1.2.0**. Plates-formes CLI/SDK : Windows 10/11 x64 et Linux (x86_64 / Jetson aarch64).

## Ce qu&#x27;il faut remettre à votre assistant

| Ressource | URL | À quoi ça sert |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Index lisible par machine de toutes les pages de ce manuel. |
| **CLI Référence** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | L&#x27;ensemble complet des commandes `chloros-cli` : chaque commande, indicateur, valeur par défaut, code de sortie et règle relative au dossier de sortie. Rédigé à l&#x27;intention des modèles de langage de grande envergure (LLM). |
| **Référence SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | L&#x27;ensemble complet des classes, signatures, exceptions et exemples pratiques de `chloros_sdk` Python API. Rédigé à l&#x27;intention des étudiants en LLM. |
| **N&#x27;importe quelle page au format Markdown brut** | ajoutez `.md` à la page URL | par ex. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` renvoie la page au format Markdown brut — idéal pour la coller dans une fenêtre contextuelle ou la récupérer depuis un agent. |

Liens dans le manuel : [CLI Référence](reference/cli-reference.md) · [Référence SDK](reference/sdk-reference.md).

{% hint style="info" %}
Ces deux pages de référence sont autonomes : un assistant qui en a lu une n&#x27;a pas besoin du reste du manuel pour écrire un script correct.
{% endhint %}

## Exemples de commandes

Copiez, complétez le fichier `<placeholders>`, puis collez-le dans votre assistant.

### 1. Traiter un dossier de vol dans NDVI

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

### 4. Enregistrer les spectres des capteurs de lumière DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Les scripts DAQ à partir de la ligne de commande passent toujours par la famille `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Les autres sous-commandes `daq` que votre assistant pourrait inventer ne sont pas disponibles dans les versions livrées et génèrent une erreur.
{% endhint %}

## Pourquoi les scripts écrits par l’IA fonctionnent bien avec Chloros

Chacun de ces comportements est réel et vérifié dans la version 1.2.0 de Chloros — ils éliminent les modes d’échec classiques de l’automatisation écrite par des machines :

* **Pas de « danse de configuration ».**Les aides de connexion intelligente de SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) et les points d’entrée de traitement (`ChlorosLocal`, `process_folder`)**lancent automatiquement le backend local**. Un script généré n&#x27;a pas besoin que l&#x27;interface graphique soit ouverte ni qu&#x27;un serveur soit démarré manuellement — il suffit que le paquet desktop/CLI soit installé.
* **L’ensemble du pipeline se résume à un seul appel.** `chloros_sdk.process_folder("path", indices=["NDVI"])` exécute de bout en bout les étapes suivantes : importation → étalonnage → réflectance → exportation de l’indice. Moins de surface d’intervention, moins de points de défaillance potentiels pour un script généré.
* **Les exécutions sans résultat font l’objet d’un autodiagnostic.** Après `process()`, le résumé de l’exécution est joint au résultat, et chaque indication relative au traitement (par exemple, *pourquoi* une exécution n’a produit aucun résultat) est également réémise sous la forme d’un Python `UserWarning` — ainsi, même un script qui n’inspecte jamais le dictionnaire de résultats affiche le diagnostic.
* **Le code CLI échoue de manière flagrante.**Une exécution `chloros-cli process` qui a demandé des produits mais n’en a écrit aucun affiche `Processing finished but wrote no image products.` et**se termine avec un code de sortie non nul**, ce qui permet aux scripts shell et à l’infrastructure de continuité (CI) de la détecter par une simple vérification du code de sortie. Les exécutions réussies renvoient le code `Image products written: N`.

Une particularité qu&#x27;un assistant doit connaître : le `process()` du SDK **ne**déclenche délibérément**aucune** exception lors d&#x27;une exécution sans produit — il le signale plutôt via le résumé et les conseils. Si un pipeline Python doit s’arrêter en cas d’exécution vide, vérifiez le résumé (c’est ce que fait la recette 2).

## Mises en garde

* **Connexion Chloros+ requise.**Les CLI et SDK nécessitent un niveau**payant**Chloros+**payant**, appliquée côté serveur : les requêtes échouent avec le code `401 AUTH_REQUIRED` si vous n’êtes pas connecté et avec le code `403 PLAN_UPGRADE_REQUIRED` sur le niveau gratuit. Exécutez `chloros-cli login` une fois par machine avant d&#x27;exécuter les scripts générés. Voir [Chloros+ Connexion](chloros+-login.md).
* **Les commandes de capture pilotent du matériel physique.** Les commandes `lattice` / `daq` / `project` et les objets de session SDK permettent de se connecter, de diffuser en continu et de déclencher des caméras et des capteurs physiques. Vérifiez le script généré avant sa première exécution, puis exécutez-le en présence du matériel.
* **Effectuez des contrôles ponctuels des résultats.** Vérifiez les dossiers de produits et quelques valeurs de pixels avant de publier les résultats. En particulier, les fichiers TIFF de réflectance sont mis à l’échelle par source — consultez la balise XMP `Chloros:PixelScale` (LATTICE : 32768 = réflectance 1,0 ; Survey3 : 65535) au lieu de supposer un diviseur. Les deux pages de référence documentent cela sous la rubrique « Lecture des pixels de réflectance ».
* **Petits pièges qui peuvent causer des problèmes au code généré :**`pool-record` écrit sur le système de fichiers de l’**hôte backend** (par défaut `~/Documents/DAQ Live View/`) ; sur les machines disposant de plusieurs interfaces réseau, préférez `daq pool-connect --eth-host <ip-or-hostname>` à la détection automatique ; et utilisez `http://127.0.0.1:5000` (jamais `localhost`) partout où apparaît un backend URL.
