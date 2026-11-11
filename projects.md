# Interface graphique : Projets

Chloros vous permet de créer des projets que vous pourrez rouvrir ultérieurement. Un projet est un simple dossier (situé dans votre dossier « Project ») contenant :

* `project.json` — paramètres du projet, liste des fichiers et préférences d’affichage
* `cameras.json` — caméras et réseaux connectés pendant que le projet était ouvert, avec leurs paramètres
* `sensors.json` — capteurs de lumière DAQ connectés pendant que le projet était ouvert, ainsi que les associations caméra↔capteur
* vos captures, vos enregistrements `.daq` et les dossiers de sortie traités

Il n’existe pas de format de fichier de projet propriétaire : le dossier et ses fichiers JSON constituent le projet, ce qui facilite également la copie, l’archivage et l’exécution des projets depuis la [CLI](CLI.md) ou le [Python SDK](api-python-sdk.md).

À moins que vous ne le modifiiez dans les [Paramètres du projet](project-settings/project-settings.md), le dossier du projet est `Documents\MAPIR\Chloros Projects` dans votre dossier Documents (`~/Documents/MAPIR/Chloros Projects` sous Linux).

{% hint style="warning" %}
**Windows : veillez à ce que les dossiers de projet soient courts, sauf si les chemins d’accès longs sont activés.**Windows est livré avec les chemins d’accès longs désactivés ; aucun chemin d’accès ne peut alors dépasser 260 caractères. Les produits traités se trouvent jusqu’à environ 130 caractères en dessous du dossier de projet ; ainsi, à partir de la version 1.3.0, Chloros refuse un dossier de projet trop profond — lors de sa création, de son ouverture ou du lancement du traitement — au lieu de s’exécuter sans rien écrire. Le message indique le dossier de projet le plus long possible. Utilisez un dossier de projet plus court, ou activez les chemins d’accès longs de Windows (option**Activer les chemins d’accès longs de Windows** du programme d’installation, cochée par défaut) et redémarrez Chloros. Voir [Lancement du traitement](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nouveau projet

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Sélectionnez « Nouveau projet » dans le menu principal et saisissez un nom unique pour votre projet.

Si vous avez enregistré des modèles de projet, une liste déroulante **Sélectionner un modèle** apparaît sous le champ de nom — en choisir un permet de démarrer le nouveau projet à partir des paramètres de ce modèle. Les modèles sont enregistrés à partir des [Paramètres du projet](project-settings/project-settings.md) : saisissez un nom dans le champ « Nom du modèle de projet à enregistrer », puis cliquez sur l’icône d’enregistrement.

## Ouvrir un projet

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>La liste « Ouvrir un projet » répertorie tous les projets de votre dossier de projets, avec l’option <strong>Ouvrir le dossier de projets</strong> en bas</p></figcaption></figure>

Sélectionnez « Ouvrir un projet » pour afficher la liste des projets existants dans le dossier de projets. S’il n’y a aucun projet, le menu latéral secondaire ne s’ouvrira pas. Vous pouvez voir certains projets créés via l’interface graphique (t1, t2, t3) répertoriés sur la photo ci-dessus. Les projets DATE\_TIME ont été créés par l’CLI à l’aide du schéma de nommage par défaut. Cliquez sur le nom d’un projet pour l’ouvrir.

Cliquez sur le bouton « Ouvrir le dossier du projet » pour ouvrir l’explorateur de fichiers de votre ordinateur à l’emplacement du projet. Vous pouvez modifier le chemin d’accès au projet dans les [Paramètres du projet](project-settings/project-settings.md).

Si l’un des fichiers image source du projet a été déplacé ou supprimé depuis sa dernière ouverture, Chloros affiche une boîte de dialogue répertoriant précisément les fichiers manquants au lieu d’ouvrir une grille vide.

## Dupliquer le projet

Cette option est disponible une fois le projet ouvert. Sélectionnez « Dupliquer le projet » pour copier le projet actuel sous un nouveau nom — Chloros suggère le prochain nom disponible (par exemple « MonProjet (2) ») — et la copie s’ouvre immédiatement.

## Ajouter des fichiers

Une fois le projet ouvert, sélectionnez « Ajouter des fichiers » dans le menu principal pour ajouter des fichiers image individuels au projet en cours. Cette fonctionnalité reprend celle du navigateur de fichiers, mais est accessible directement depuis le menu principal pour plus de commodité.

## Ajouter un dossier

Une fois le projet ouvert, sélectionnez « Ajouter un dossier » dans le menu principal pour ajouter des dossiers d’images au projet en cours. Vous pouvez sélectionner plusieurs dossiers en une seule fois. Les fichiers en double sont ignorés.

## Démarrer / Arrêter le traitement

Une fois les fichiers ajoutés à un projet, l’option « Démarrer le traitement » devient disponible dans le menu principal. Cela revient à cliquer sur le bouton Lecture/Démarrer situé dans l’en-tête supérieur. Pendant le traitement, l’option du menu devient « Arrêter le traitement » pour vous permettre d’interrompre le pipeline.

## Se connecter à une caméra / Se connecter à un capteur de lumière

La partie inférieure du menu principal comporte deux raccourcis matériels, disponibles qu’un projet soit ouvert ou non :

* **Se connecter à une caméra** — ouvre l’[onglet Caméras](lattice/) pour connecter une caméra ou un réseau de caméras LATTICE.
* **Se connecter à un capteur de lumière** — ouvre l’[onglet « Capteurs de lumière »](daq/) pour connecter un capteur de lumière DAQ.

La connexion de matériel alors qu’un projet est ouvert entraîne son enregistrement dans le projet (voir ci-dessous). En l’absence de projet, les connexions ne sont valables que pour la session en cours.

{% hint style="info" %}
Les éléments de menu **Ajouter des fichiers**,**Ajouter un dossier**et**Démarrer/Arrêter le traitement** ne sont visibles ou activés que lorsqu’un projet est ouvert et que des fichiers ont été ajoutés. Ils permettent d’accéder rapidement à des actions également disponibles via la barre latérale du navigateur de fichiers et les boutons de l’en-tête.
{% endhint %}

## Les projets mémorisent votre matériel

Nouveauté de la version 1.2.0 : un projet conserve en mémoire le matériel que vous connectez tant qu’il est ouvert. Les caméras et les matrices (avec leurs paramètres par caméra, leurs noms, leurs couleurs et leur disposition en grille) sont enregistrées dans `cameras.json`, et les capteurs de lumière (avec leurs noms, leurs couleurs et leurs associations de caméras) dans `sensors.json` — automatiquement, au fur et à mesure que vous travaillez.

Lorsque vous **rouvrez** un projet, Chloros n’intervient pas immédiatement sur le matériel. Chaque partie se reconnecte la première fois que vous accédez à l’onglet qui lui est dédié :

* L’ouverture de l’onglet **Caméras** reconnecte les caméras et les matrices enregistrées et réapplique leurs paramètres enregistrés.
* L’ouverture de l’onglet **Capteurs de lumière**reconnecte les capteurs DAQ enregistrés et réapplique leurs paramètres enregistrés, capuchon compris. Un capteur réglé sur**Aucun (capteur nu)** dans son menu Cap reste nu. Un projet enregistré par une version antérieure de Chloros a enregistré un capteur nu comme option par défaut ; il réapparaît donc avec la correction cosinus d’ensoleillement ; réglez-le à nouveau sur Aucun et enregistrez le projet.

De cette manière, l’ouverture d’un projet dans le seul but de parcourir ou d’exporter des images ne déclenche jamais la diffusion en continu des caméras. Si un périphérique enregistré est introuvable à l’ouverture de son onglet, une boîte de dialogue vous indique quels périphériques sont indisponibles afin que vous puissiez les reconnecter ou les supprimer.

## Enregistrements DAQ et fichiers .daq dans un projet

* Les enregistrements `.daq` effectués pendant que le projet est ouvert (à partir de l’onglet « Capteurs de lumière » ou pendant les captures) sont **automatiquement ajoutés au projet**.
* Les fichiers `.daq` importés, ainsi que tous les enregistrements du projet, sont répertoriés dans la section **Capteurs de lumière DAQ** des [Paramètres du projet](project-settings/project-settings.md), chacun avec son profil de correction de crête.
* Lors du traitement, les fichiers `.daq` du projet fournissent l’éclairement descendant pour les produits de réflectance — voir [Formats d’images de sortie](output-image-formats.md).

## Exécution d’un projet enregistré en mode sans interface graphique

Un projet enregistré peut être exécuté sans l’interface graphique :

* **CLI** : `chloros-cli project open / connect / capture / sensor / align / run` fonctionne à partir du chemin d’accès au dossier du projet — voir la [Référence CLI](reference/cli-reference.md).
* **SDK** : `chloros_sdk.open_project(path)` renvoie un identifiant de projet ; `connect_all()` met en ligne toutes les caméras, tous les réseaux de caméras et tous les capteurs de lumière enregistrés avec leurs paramètres enregistrés (les capteurs de lumière via un backend Chloros en cours d’exécution) — voir la [Référence SDK](reference/sdk-reference.md).
