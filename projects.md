# Interface graphique : Projets

Chloros vous permet de créer des projets que vous pourrez rouvrir ultérieurement. Un projet est un simple dossier (situé dans votre dossier de projets) contenant :

* `project.json` — les paramètres du projet, la liste des fichiers et les préférences d’affichage
* `cameras.json` — les caméras et les réseaux connectés pendant que le projet était ouvert, avec leurs paramètres
* `sensors.json` — les capteurs de lumière DAQ connectés pendant que le projet était ouvert, ainsi que les associations caméra↔capteur
* vos captures, vos enregistrements `.daq` et les dossiers de sortie traités

Il n’existe pas de format de fichier de projet propriétaire : le dossier et ses fichiers JSON constituent le projet, ce qui facilite également la copie, l’archivage et le transfert des projets depuis le [CLI](CLI.md) ou [Python SDK](api-python-sdk.md).

## Nouveau projet

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Sélectionnez « Nouveau projet » dans le menu principal et saisissez un nom unique pour votre projet.

Si vous avez enregistré des modèles de projet, une liste déroulante **Sélectionner un modèle** apparaît sous le champ de nom — en choisir un permet de démarrer le nouveau projet à partir des paramètres de ce modèle. Les modèles sont enregistrés à partir des [Paramètres du projet](project-settings/project-settings.md) : saisissez un nom dans le champ « Nom du modèle de projet » et cliquez sur l&#x27;icône d&#x27;enregistrement.

## Ouvrir un projet

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>La liste « Ouvrir un projet » répertorie tous les projets présents dans votre dossier de projets, avec l’option « <strong>Ouvrir le dossier de projets »</strong> en bas</p></figcaption></figure>Sélectionnez « Ouvrir un projet » pour afficher la liste des projets existants dans le dossier de projets. S’il n’y a aucun projet, le menu latéral secondaire ne s’ouvrira pas. Vous pouvez voir certains projets créés via l’interface graphique (t1, t2, t3) répertoriés sur la photo ci-dessus. Les projets DATE\_TIME ont été créés par CLI à l’aide du schéma de nommage de projet par défaut. Cliquer sur le nom d’un projet quelconque l’ouvrira.

Cliquer sur le bouton « Ouvrir le dossier du projet » ouvre l’explorateur de fichiers de votre ordinateur à l’emplacement du projet. Vous pouvez modifier l’emplacement du projet dans les [Paramètres du projet](project-settings/project-settings.md).

Si l’un des fichiers image sources du projet a été déplacé ou supprimé depuis sa dernière ouverture, Chloros affiche une boîte de dialogue répertoriant précisément les fichiers manquants au lieu d’ouvrir une grille vide.

## Dupliquer un projet

Cette option est disponible une fois le projet ouvert. Sélectionnez « Dupliquer le projet » pour copier le projet actuel sous un nouveau nom — Chloros suggère le prochain nom disponible (par exemple « MonProjet (2) ») — et la copie s’ouvre immédiatement.

## Ajouter des fichiers

Une fois le projet ouvert, sélectionnez « Ajouter des fichiers » dans le menu principal pour ajouter des fichiers image individuels au projet en cours. Cette fonction reprend celle du navigateur de fichiers, mais est accessible directement depuis le menu principal pour plus de commodité.

## Ajouter un dossier

Une fois le projet ouvert, sélectionnez « Ajouter un dossier » dans le menu principal pour ajouter des dossiers d’images au projet en cours. Vous pouvez sélectionner plusieurs dossiers en une seule fois. Les fichiers en double sont ignorés.

## Démarrer / Arrêter le traitement

Une fois les fichiers ajoutés à un projet, l’option « Démarrer le traitement » devient disponible dans le menu principal. Cela revient à cliquer sur le bouton Lecture/Démarrer situé dans l’en-tête supérieur. Pendant le traitement, l’option du menu devient « Arrêter le traitement » pour vous permettre d’interrompre le pipeline.

## Se connecter à une caméra / Se connecter à un capteur de lumière

La partie inférieure du menu principal comporte deux raccourcis matériels, disponibles qu’un projet soit ouvert ou non :

* **Se connecter à une caméra** — ouvre l’[onglet Caméras](lattice/) pour connecter une caméra ou un réseau de caméras LATTICE.
* **Se connecter à un capteur de lumière** — ouvre l’[onglet Capteurs de lumière](daq/) pour connecter un capteur de lumière DAQ.

La connexion d’un périphérique alors qu’un projet est ouvert permet de l’enregistrer dans ce projet (voir ci-dessous). En l’absence de projet, les connexions ne sont valables que pour la session en cours.

{% hint style="info" %}
Les éléments de menu **Ajouter des fichiers**,**Ajouter un dossier**et**Démarrer/Arrêter le traitement** ne sont visibles ou activés que lorsqu’un projet est ouvert et que des fichiers ont été ajoutés. Ils permettent d’accéder rapidement à des actions également disponibles via la barre latérale du navigateur de fichiers et les boutons de l’en-tête.
{% endhint %}

## Les projets mémorisent votre matériel

Nouveauté de la version 1.2.0 : un projet conserve en mémoire le matériel que vous connectez tant qu’il est ouvert. Les caméras et les matrices (avec leurs paramètres par caméra, leurs noms, leurs couleurs et leur disposition en grille) sont enregistrées dans `cameras.json`, et les capteurs de lumière (avec leurs noms, leurs couleurs et leurs associations de caméras) dans `sensors.json` — automatiquement, au fur et à mesure que vous travaillez.

Lorsque vous **rouvrez** un projet, le fichier Chloros n’interagit pas immédiatement avec le matériel. Chaque partie se reconnecte la première fois que vous accédez à l’onglet qui lui est associé :

* L’ouverture de l’onglet **Caméras** reconnecte les caméras et les matrices enregistrées et réapplique leurs paramètres enregistrés.
* L&#x27;ouverture de l&#x27;onglet **Capteurs de lumière** reconnecte les capteurs DAQ enregistrés.

De cette manière, ouvrir un projet uniquement pour parcourir ou exporter des images ne met jamais les caméras en mode streaming. Si un périphérique enregistré est introuvable à l&#x27;ouverture de son onglet, une boîte de dialogue vous indique quels périphériques sont indisponibles afin que vous puissiez les reconnecter ou les supprimer.

## Enregistrements DAQ et fichiers .daq dans un projet

* Les enregistrements `.daq` réalisés alors que le projet est ouvert (à partir de l’onglet Capteurs de lumière ou pendant les captures) sont **automatiquement ajoutés au projet**.
* Les fichiers `.daq` importés, ainsi que tous les enregistrements du projet, sont répertoriés dans la section **Capteur de lumière DAQ** des [Paramètres du projet](project-settings/project-settings.md), chacun avec son profil de correction de crête.
* Lors du traitement, les fichiers `.daq` du projet fournissent l’éclairage descendant pour les produits de réflectance — voir [Formats d’images de sortie](output-image-formats.md).

## Exécution d’un projet enregistré en mode sans interface graphique

Un projet enregistré peut être exécuté sans l’interface graphique :

* **CLI** : `chloros-cli project open / connect / capture / sensor / align / run` fonctionne à partir du chemin d’accès au dossier du projet — voir la [Référence CLI](reference/cli-reference.md).
* **SDK** : `chloros_sdk.open_project(path)` renvoie un descripteur de projet ; `connect_all()` met en ligne toutes les caméras et tous les capteurs enregistrés avec leurs paramètres sauvegardés — voir la [Référence SDK](reference/sdk-reference.md).
