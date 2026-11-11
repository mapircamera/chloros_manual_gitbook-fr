# Adaptation dynamique des ressources de calcul :

Chloros 1.3.0 utilise la détection matérielle et la sélection automatique de la stratégie de traitement. Le moteur de traitement s’adapte à votre matériel — du Jetson Orin Nano à une station de travail multi-GPU — sans aucune configuration manuelle.

***

## Comment ça marche

Au démarrage, Chloros effectue un profilage de votre système :

1. **Détecte le système d’exploitation** — Windows ou Linux
2. **Identifie les cœurs de processeur et la mémoire vive totale**

3.**Détecte la présence d’un GPU** — compatibilité NVIDIA CUDA, mémoire vidéo (VRAM), modèle
4. **Identifie le modèle Jetson** (le cas échéant) — via `/proc/device-tree/model`
5. **Vérifie les capteurs thermiques** (Jetson) — pour un traitement tenant compte de la température
6. **Sélectionne la stratégie de calcul** — en fonction de l’ensemble du matériel détecté
7. **Configure automatiquement le nombre de workers, le type de pipeline et l’allocation de mémoire**

Le profil détecté est mis en cache pour la session, en mémoire et sur disque, ce qui permet d’accélérer le démarrage des exécutions ultérieures :

| Plateforme | Profil mis en cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (privilégie `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Supprimez ce fichier pour forcer une nouvelle détection — cela s’avère utile après l’ajout d’un GPU ou de mémoire vive supplémentaire. Chloros effectue également une nouvelle détection automatiquement lorsque le cache a été écrit par une ancienne version incompatible.

***

## Stratégies de calcul

Chloros sélectionne l’une des trois stratégies de calcul en fonction de votre matériel :

| Stratégie | Sélectionnée lorsque | Workers | Exécuteur | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA signalant**12 Go ou plus de VRAM**(sur mémoire unifiée Jetson, nécessite également au moins 12 Go de RAM partagée au total) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimum 2 —**plafonné à 2 sur Jetson** | Processus de travail (contextes CUDA distincts) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA avec**2 à 12 Go de VRAM**| 3 (chevauchement des E/S ; accès au GPU sérialisé par un sémaphore).**1 (séquentiel) sur les Jetson disposant de moins de 12 Go de RAM** | Processus de travail (contextes CUDA distincts) ; séquentiel en cours de traitement sur les Jetson à faible capacité de RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Pas de GPU CUDA, ou moins de 2 Go de VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Exemples concrets de la formule de calcul du nombre de workers `GPU_PARALLEL` : 12 Go de VRAM → 2 workers, 16 Go → 3 workers, 20 Go et plus → 4 workers. Chaque worker dispose d’un budget de 3,5 Go, soit le besoin mesuré pour une tâche « Texture Aware », dans la limite de 90 % de la VRAM restante après une réserve de 0,5 Go.

{% hint style="info" %}
**Les stratégies GPU nécessitent une version de PyTorch capable d’utiliser CUDA.** Une machine dont la version de PyTorch ne dispose d’aucun périphérique CUDA utilisable se voit attribuer la valeur `CPU_PARALLEL`, quel que soit son GPU. Le package Jetson 1.3.0 utilise la version CUDA de PyTorch fournie par JetPack 6 ; un Jetson est donc éligible dès lors que les bibliothèques CUDA et cuDNN de JetPack sont installées. Un Jetson disposant de moins de 12 Go de RAM traite toujours une image à la fois, en cours de traitement (enregistré sous le code `GPU_SINGLE`).
{% endhint %}

Les stratégies GPU exécutent un pool de processus de travail, chacun étant un processus système distinct doté de son propre contexte CUDA, qu’Chloros lance en relançant son propre programme backend installé (et non via `multiprocessing` de Python, dont le démarrage des processus de travail ne fonctionne pas au sein de l’application packagée). La stratégie CPU utilise un pool de threads (`ThreadPoolExecutor` de Python). Chloros n’utilise aucun framework distribué tiers (tel que Ray).

### Types de pipelines

* **`fused_gpu`** — Parcours de traitement entièrement sur GPU. Les opérations de débayérisation, de correction et d’indexation s’exécutent sur le GPU en un seul passage fusionné. Débit maximal, nécessite le plus de VRAM.
* **`tiled_gpu`** — Parcours GPU optimisé pour la mémoire. Traite les images par tuiles afin de s’adapter à la mémoire GPU limitée. Débit inférieur, mais fonctionne sur les appareils à mémoire restreinte.
* **`cpu_fallback`** — Traitement exclusivement sur CPU utilisant le parallélisme multithread. Utilisé lorsqu’aucun GPU NVIDIA n’est disponible, et comme solution de secours en dernier recours lorsque les deux chemins GPU échouent.

La chaîne de repli en exécution est toujours `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Remplacement manuel de la stratégie

Définissez la variable d’environnement `CHLOROS_STRATEGY` pour forcer une stratégie spécifique — une solution de secours pour les experts lorsque la détection automatique choisit une option inadaptée à votre situation (par exemple, pour garder le GPU libre pour d’autres tâches) :

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

La correspondance de la variable ne tient pas compte de la casse ; tout ce qui ne correspond pas à l’un des trois noms est ignoré et la détection automatique se poursuit normalement. Même en cas de remplacement, Chloros continue de choisir le nombre de workers à votre place :

| Remplacement | Nombre de workers utilisé |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Une substitution `GPU_SINGLE` ou `GPU_PARALLEL` sur une machine ne disposant d’aucun périphérique CUDA utilisable est refusée, comme l’indique la ligne `[STRATEGY]` dans le journal, et l'exécution utilise `CPU_PARALLEL`.

Il est préférable de définir ce paramètre par commande plutôt que de manière permanente, afin que les exécutions normales continuent à s'adapter automatiquement.

***

## Comportement spécifique à la plateforme

| Plateforme | Stratégie | Workers | Pipeline | Remarques |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 Go** | `GPU_SINGLE` | 1 | `tiled_gpu` (séquentiel) | Mode économe en mémoire, une image à la fois |
| **Jetson Orin NX 8 Go** | `GPU_SINGLE` | 1 | `tiled_gpu` (séquentiel) | La mémoire vive partagée inférieure à 12 Go impose un traitement séquentiel |
| **Jetson Orin NX 16 Go** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurrent) | Appareil de périphérie recommandé — Limité à 2 workers sur Jetson |
| **Jetson AGX Orin 32-64 Go** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurrent) | Performances maximales en périphérie (également limité à 2 workers par Jetson) |
| **Ordinateur de bureau avec GPU de 8 Go** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 workers se partagent les E/S tandis qu’un sémaphore sérialise l’accès au GPU |
| **Ordinateur de bureau avec GPU de 12 Go ou plus** | `GPU_PARALLEL` | 2-4 | `fused_gpu` (simultanés) | Performances optimales sur ordinateur de bureau : 12 Go → 2 travailleurs, 16 Go → 3, 20 Go et plus → 4 |
| **Système CPU uniquement** | `CPU_PARALLEL` | cœurs physiques − 1 (min. 2) | `cpu_fallback` | Aucun GPU requis, utilise un pool de threads |

{% hint style="info" %}
**Mémoire unifiée Jetson** : les appareils Jetson partagent la mémoire du GPU et du CPU. Un Jetson Orin NX de 16 Go affiche environ 15,3 Go de VRAM, mais il s’agit de la même mémoire RAM physique que celle utilisée par le système d’exploitation et les processus du CPU. C’est pourquoi les Jetson de 16 Go et plus sont éligibles à `GPU_PARALLEL` au même titre qu’un GPU de bureau de 12 Go et plus, tout en étant limités à 2 workers — le GPU, les processus workers et leurs contextes CUDA par worker puisent tous dans le même pool partagé.
{% endhint %}

### Budget GPU en fonction de la VRAM (cartes graphiques discrètes)

Sur les hôtes x86_64 équipés d’une carte graphique NVIDIA discrète, la VRAM détectée détermine également la quantité de ressources que le traitement de la carte peut revendiquer et la taille maximale des lots :

| VRAM détectée | Plafond du budget GPU | Multiplicateur de taille de lot |
| --- | --- | --- |
| **8 Go et plus** | 90 % | ×2,0 |
| **6 à 8 Go** | 85 % | ×1,75 |
| **3,5 à 6 Go** | 80 % | ×1,5 |
| **2 à 3,5 Go** | 75 % | ×1,25 |
| **Moins de 2 Go** | 70 % | ×1,0 |

Les GPU discrets ne réservent que 0,5 Go au système, car ils ne partagent pas la mémoire vive (RAM) du système. Les profils Jetson réservent beaucoup plus de mémoire et ont un plafond plus bas — voir le [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Allocation dynamique de la mémoire GPU

Chloros utilise un [pipeline de traitement à 4 threads](processing-pipeline.md) :

* **Thread 1** (Détection) — Chargement de l’image, analyse des données EXIF, détection de la cible
* **Thread 2** (Étalonnage) — Calcul de l’étalonnage de la réflectance
* **Thread 3** (Traitement) — Débayérisation GPU, correction du vignettage, calcul de l’indice
* **Thread 4** (Exportation) — Écriture de fichiers, intégration des métadonnées

Les threads 1, 2 et 4 sont peu gourmands en ressources GPU ; le thread 3 est celui qui en sollicite le plus. À mesure que les threads en amont du pipeline se terminent, leur budget GPU est **redistribué aux threads actifs restants**, de sorte que le thread 3 dispose progressivement de plus de mémoire au fur et à mesure que l'exécution avance.

### Étapes d'allocation

| Étape | Threads actifs | Répartition de la mémoire GPU |
| --- | --- | --- |
| **Début** | 1, 2, 3, 4 | Répartie entre tous les threads, la majeure partie allant au thread 3 |
| **Début-milieu** | 2, 3, 4 | La part du thread 1 est redistribuée |
| **Mi-fin** | 3, 4 | Les parts des threads 1 et 2 sont attribuées aux threads 3 et 4 |
| **Fin** | 3 ou 4 | Le dernier thread actif reçoit son allocation maximale |

Deux règles régissent ces chiffres :

* Un thread qui est le **seul** actif se voit attribuer l’allocation maximale de son profil.
* Lorsque plusieurs tâches GPU *lourdes* sont actives, l’allocation de base de chaque tâche lourde est répartie entre elles (sans jamais descendre en dessous du minimum configuré).

La valeur effectivement utilisée lors de l’exécution est la **plus faible** entre l’allocation du profil de la plateforme et la recommandation en temps réel du moniteur de mémoire GPU ; ainsi, une carte très sollicitée l’emporte toujours sur un profil optimiste.***

## Traitement tenant compte des textures

Le débayeur « Texture Aware » (**Chloros+ uniquement** — `--debayer texture-aware`) exécute un modèle de débruitage IA/ML qui nécessite environ 1,75 Go de VRAM en FP16 par copie ; il utilise donc bien plus de mémoire GPU que la méthode standard :

* Les systèmes disposant de **moins de 7 Go de VRAM**traitent le mode « Texture Aware » dans une**boucle synchrone, une image à la fois** — plusieurs copies du modèle ne peuvent pas tenir, et un pool de workers ne ferait qu’ajouter des conflits d’accès
* Les systèmes disposant de **7 Go ou plus de VRAM** peuvent traiter Texture Aware de manière concurrente, bien qu’avec un nombre de workers réduit par rapport à la méthode Standard
* Sur **Jetson**, le mode « Texture Aware » est toujours affecté à un seul worker
* En l’**absence de GPU CUDA utilisable**, l’exécution utilise le débayériseur « Standard », et le backend enregistre une ligne `[DEBAYER] … needs a CUDA GPU …` expliquant pourquoi
* Sur les Jetson à faible consommation (Nano, Orin Nano), l’CLI applique une limite de fréquence du GPU lorsque vous demandez l’utilisation de Texture Aware — voir le [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Gestion thermique (Jetson)

Les appareils Jetson sont soumis à des contraintes thermiques, en particulier dans les déploiements en espace clos ou aéroportés. Chloros surveille les capteurs de température intégrés au Jetson et adapte automatiquement la taille des lots :

| Température | Réaction |
| --- | --- |
| **< 70 °C** | Fonctionnement normal — pleine vitesse |
| **70 °C** (Avertissement) | La taille des lots diminue progressivement (100 % → 50 % entre 70 °C et 80 °C) |
| **80 °C** (Critique) | Limitation agressive (50 % → 0 % entre 80 °C et 90 °C) |
| **90 °C** (Arrêt) | Arrêt complet du traitement par le GPU |

Sur les ordinateurs de bureau dotés d’un refroidissement adéquat, la limitation thermique est rarement déclenchée.

***

## Gestion de la pression mémoire

Chloros surveille en permanence la mémoire du GPU pendant le traitement et réagit à trois niveaux.

**Dimensionnement des lots.** Un lot commence à 8 images multipliées par le coefficient de la plateforme indiqué dans les tableaux ci-dessus. Chloros vérifie ensuite la VRAM disponible, en réserve 20 % pour la surcharge propre à PyTorch, et estime à environ 100 Mo de mémoire GPU par image de 12 MP — la taille du lot correspond à la plus petite des deux valeurs : la limite dérivée de la mémoire ou la valeur de base de la plateforme. Elle ne descend jamais en dessous de 1.

**Réduction préventive.**Au-delà de**85 % d’utilisation de la VRAM**, la taille des lots est réduite avant toute défaillance.**Réduction de l’allocation par thread.** À mesure que l’utilisation en temps réel augmente, le budget GPU de chaque thread est réduit : ×0,75 au-delà de 80 % d’utilisation, ×0,5 au-delà de 90 %. Les seuils de surveillance sont fixés à 70 % (prudent), 85 % (limite de fonctionnement normale) et 95 % (risque de manque de mémoire).**Recul et récupération en cas de manque de mémoire.** Si un événement de manque de mémoire se produit malgré tout :

* la taille du lot est **divisée par deux**, puis à nouveau divisée par deux à chaque « OOM » consécutif — chaque lot suivant réussi réduit cette pénalité d’un cran
* les allocations des threads actifs sont réduites à 70 % de leur valeur actuelle et l’allocateur passe à sa stratégie prudente, s’assouplissant à nouveau après une série d’allocations réussies
* en cas de forte pression, le pipeline passe de `fused_gpu` à `tiled_gpu`, puis à `cpu_fallback` en dernier recours

**Mémoire vive de l’hôte (Jetson).** Avant le traitement, l’CLI estime la mémoire maximale de l’hôte en fonction du nombre d’images et du mode de débayérisation, et émet un avertissement si la RAM et l’espace d’échange sur disque risquent d’être insuffisants, en affichant les commandes exactes pour ajouter de l’espace d’échange — voir le [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Surveillance de l’adaptation du calcul

### Diagnostics système

`chloros-cli selftest` est le moyen le plus rapide de vérifier ce que perçoit la couche de calcul :

```bash
chloros-cli selftest
```

Ses 7 vérifications portent sur la version, la disponibilité des ports, le démarrage du backend, `/api/test`, les informations système, la présence du modèle de débruitage et la disponibilité de CUDA et du débruitage. La vérification n° 5 affiche directement la ligne relative au matériel :

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

La vérification n° 7 affiche `CUDA: <bool>, Denoiser: <bool>` — ces deux conditions doivent être remplies pour que Texture Aware soit utilisable.

### Journaux du backend

La stratégie et le nombre de workers sont choisis au sein du backend au début de chaque exécution — aucune bannière de l’CLI ne les indique. Lorsqu’un comportement inattendu se produit (retour à un chemin GPU de secours, un OOM, un débruiteur qui ne se charge pas), c’est dans le journal du backend de cette session que cela apparaît :

| Plateforme | Emplacement du journal |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (un fichier par lancement) |
| **Linux, backend lancé via la CLI** | également `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Progression en temps réel

Pendant une exécution, l’CLI affiche en temps réel la progression par thread (Détection, Analyse, Traitement, Exportation) transmise via Server-Sent Events — ce qui permet de déterminer concrètement si le thread 3 constitue le goulot d’étranglement. Voir [Pipeline de traitement](processing-pipeline.md).

***

## Étapes suivantes

* [Pipeline de traitement](processing-pipeline.md) — Comprendre l’architecture du pipeline à 4 threads
* [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Déploiement et optimisation spécifiques à Jetson
* [CLI : ligne de commande](../CLI.md) — Le guide de l’interface de ligne de commande
* [Référence CLI](../reference/cli-reference.md) — Liste exhaustive des commandes pour la version 1.3.0
