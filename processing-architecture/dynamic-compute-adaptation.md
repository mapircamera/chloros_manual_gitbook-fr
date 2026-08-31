# Adaptation dynamique des ressources de calcul

Chloros 1.2.0 utilise la détection matérielle et la sélection automatique de la stratégie de traitement. Le moteur de traitement s&#x27;adapte à votre matériel — du Jetson Orin Nano à une station de travail multi-GPU — sans aucune configuration manuelle.

***

## Comment ça marche

Au démarrage, Chloros effectue un profilage de votre système :

1. **Détecte le système d’exploitation** — Windows ou Linux
2. **Identifie les cœurs de processeur et la mémoire vive totale**

3.**Détecte la présence d’un GPU** — Capacité NVIDIA CUDA, VRAM, modèle
4. **Identifie le modèle Jetson** (le cas échéant) — via `/proc/device-tree/model`
5. **Vérifie les capteurs thermiques** (Jetson) — pour un traitement tenant compte de la température
6. **Sélectionne la stratégie de calcul** — en fonction de l’ensemble du matériel détecté
7. **Configure automatiquement le nombre de workers, le type de pipeline et l’allocation de mémoire**

Le profil détecté est mis en cache pour la session, en mémoire et sur disque, ce qui permet d’accélérer les exécutions suivantes :

| Plateforme | Profil mis en cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (privilégie `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Supprimez ce fichier pour forcer une nouvelle détection — utile après l’ajout d’un GPU ou de mémoire vive supplémentaire. Chloros effectue également une nouvelle détection automatiquement lorsque le cache a été écrit par une ancienne version incompatible.

***

## Stratégies de calcul

Chloros sélectionne l’une des trois stratégies de calcul en fonction de votre matériel :

| Stratégie | Sélectionnée lorsque | Workers | Exécuteur | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA déclarant**au moins 12 Go de VRAM**(sur la mémoire unifiée Jetson, nécessite également au moins 12 Go de RAM partagée au total) | `min(4, VRAM ÷ 4GB)`, minimum 2 —**plafonné à 2 sur Jetson** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA avec**2 à 12 Go de VRAM**| 3 (chevauchement d’E/S ; accès au GPU sérialisé par un sémaphore).**1 (séquentiel) sur les Jetson disposant de moins de 12 Go de RAM** | `ProcessPoolExecutor` (spawn) ; séquentiel en cours de traitement sur les Jetson à faible mémoire | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Pas de GPU CUDA, ou moins de 2 Go de VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Exemples concrets de la formule de calcul du nombre de workers `GPU_PARALLEL` : 12 Go de VRAM → 3 workers, 16 Go et plus → 4 workers, tout modèle Jetson → 2 workers.

Le parallélisme est implémenté à l’aide de la stratégie standard `concurrent.futures` de Python : les stratégies GPU utilisent un `ProcessPoolExecutor` avec la méthode de démarrage **spawn** (chaque worker est un processus distinct disposant de son propre contexte CUDA — `fork` copierait un état CUDA déjà initialisé et corromprait les processus enfants), tandis que la stratégie CPU utilise une classe `ThreadPoolExecutor`. Chloros n’utilise aucun framework distribué tiers (tel que Ray).

### Types de pipelines

* **`fused_gpu`** — Parcours de traitement entièrement sur GPU. Les opérations de débayérisation, de correction et d’indexation s’exécutent sur le GPU en un seul passage fusionné. Débit maximal, nécessite le plus de VRAM.
* **`tiled_gpu`** — Parcours GPU économe en mémoire. Traite les images par tuiles afin de s’adapter à la mémoire GPU limitée. Débit inférieur, mais fonctionne sur les appareils à mémoire restreinte.
* **`cpu_fallback`** — Traitement exclusivement sur CPU utilisant le parallélisme multithread. Utilisé lorsqu’aucun GPU NVIDIA n’est disponible, et comme solution de secours en dernier recours lorsque les deux chemins de traitement GPU échouent.

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

Il est préférable de définir ce paramètre par commande plutôt que de manière permanente, afin que les exécutions normales continuent de s’adapter automatiquement.

***

## Comportement spécifique à la plateforme

| Plateforme | Stratégie | Tâches | Pipeline | Remarques |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 Go** | `GPU_SINGLE` | 1 | `tiled_gpu` (séquentiel) | Mode économe en mémoire, une image à la fois |
| **Jetson Orin NX 8 Go** | `GPU_SINGLE` | 1 | `tiled_gpu` (séquentiel) | La mémoire RAM partagée inférieure à 12 Go impose un traitement séquentiel |
| **Jetson Orin NX 16 Go** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurrent) | Appareil de périphérie recommandé — Jetson limité à 2 workers |
| **Jetson AGX Orin 32-64 Go** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurrent) | Performances maximales en périphérie (également limité à 2 workers sur Jetson) |
| **Ordinateur de bureau avec GPU de 8 Go** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | Les 3 workers se partagent les E/S tandis qu’un sémaphore sérialise l’accès au GPU |
| **Ordinateur de bureau avec GPU de 12 Go ou plus** | `GPU_PARALLEL` | 3-4 | `fused_gpu` (simultanés) | Performances optimales sur ordinateur de bureau : 12 Go → 3 travailleurs, 16 Go et plus → 4 |
| **Système CPU uniquement** | `CPU_PARALLEL` | cœurs physiques − 1 (min. 2) | `cpu_fallback` | Aucun GPU requis, utilise un pool de threads |

{% hint style="info" %}
**Mémoire unifiée Jetson** : les appareils Jetson partagent la mémoire du GPU et du CPU. Un Jetson Orin NX de 16 Go affiche environ 15,3 Go de VRAM, mais il s’agit de la même mémoire RAM physique que celle utilisée par le système d’exploitation et les processus du CPU. C’est pourquoi les Jetson de 16 Go et plus sont éligibles à `GPU_PARALLEL` au même titre qu’un GPU de bureau de 12 Go et plus, tout en étant limités à 2 workers : le GPU, les processus workers et leurs contextes CUDA par worker puisent tous dans le même pool partagé.
{% endhint %}

### Budget GPU en fonction de la VRAM (cartes graphiques discrètes)

Sur les hôtes x86_64 équipés d’une carte graphique NVIDIA discrète, la VRAM détectée détermine également la part de ressources que le traitement de la carte peut revendiquer et la taille maximale des lots :

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

* **Thread 1** (Détection) — Chargement de l&#x27;image, analyse des données EXIF, détection de la cible
* **Thread 2** (étalonnage) — Calcul de l’étalonnage de la réflectance
* **Thread 3** (traitement) — Débayérisation par GPU, correction du vignettage, calcul de l’indice
* **Thread 4** (exportation) — Écriture de fichiers, intégration des métadonnées

Les threads 1, 2 et 4 sollicitent peu le GPU ; le thread 3 est celui qui le sollicite le plus. À mesure que les threads des étapes précédentes du pipeline se terminent, leur budget GPU est **redistribué aux threads actifs restants**, de sorte que le thread 3 dispose progressivement de plus de mémoire au fur et à mesure de l’avancement de l’exécution.

### Étapes d’allocation

| Étape | Threads actifs | Répartition de la mémoire GPU |
| --- | --- | --- |
| **Début** | 1, 2, 3, 4 | Répartie entre tous les threads, la majeure partie allant au thread 3 |
| **Début-milieu** | 2, 3, 4 | La part du thread 1 est redistribuée |
| **Milieu-fin** | 3, 4 | Les parts des threads 1 et 2 sont attribuées aux threads 3 et 4 |
| **Fin** | 3 ou 4 | Le dernier thread actif reçoit son allocation maximale |

Deux règles régissent ces chiffres :

* Un thread qui est le **seul** actif se voit attribuer l&#x27;allocation maximale définie dans son profil.
* Lorsque plusieurs tâches GPU *lourdes* sont actives, l’allocation de base de chaque tâche lourde est répartie entre elles (sans jamais descendre en dessous du minimum configuré).

La valeur effectivement utilisée lors de l’exécution est la **plus faible** entre l’allocation du profil de la plateforme et la recommandation en temps réel du moniteur de mémoire GPU ; ainsi, une carte très sollicitée l’emporte toujours sur un profil optimiste.***

## Traitement sensible aux textures

Le débayeur sensible aux textures (**Chloros+ uniquement** — `--debayer texture-aware`) exécute un modèle de débruitage IA/ML qui nécessite environ 1,75 Go de VRAM en FP16 par copie ; il utilise donc bien plus de mémoire GPU que la méthode standard :

* Les systèmes dotés de **moins de 7 Go de VRAM**traitent le mode « Texture Aware » dans une**boucle synchrone, une image à la fois** — plusieurs copies du modèle ne peuvent pas y tenir, et un pool de workers ne ferait qu’ajouter des conflits d’accès
* Les systèmes dotés de **7 Go ou plus de VRAM** peuvent traiter Texture Aware en parallèle, mais avec un nombre de workers réduit par rapport à la méthode Standard
* Sur **Jetson**, Texture Aware est toujours affecté à un seul worker, et sur les modèles à faible consommation (Nano, Orin Nano), il applique également automatiquement une limite de fréquence du GPU — voir le [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Gestion thermique (Jetson)

Les appareils Jetson sont soumis à des contraintes thermiques, en particulier dans les déploiements en milieu fermé ou aéroporté. Chloros surveille les capteurs de température intégrés au Jetson et adapte automatiquement la taille des lots :

| Température | Réaction |
| --- | --- |
| **&lt; 70 °C** | Fonctionnement normal — pleine vitesse |
| **70 °C** (Avertissement) | La taille des lots diminue progressivement (100 % → 50 % entre 70 °C et 80 °C) |
| **80 °C** (Critique) | Limitation agressive (50 % → 0 % entre 80 °C et 90 °C) |
| **90 °C** (Arrêt) | Arrêt complet du traitement par le GPU |

Sur les ordinateurs de bureau dotés d’un refroidissement adéquat, la limitation thermique est rarement déclenchée.

***

## Gestion de la pression mémoire

Chloros surveille en permanence la mémoire du GPU pendant le traitement et réagit à trois niveaux.

**Taille des lots.** Un lot commence à 8 images multipliées par le coefficient de la plateforme indiqué dans les tableaux ci-dessus. Chloros vérifie ensuite la VRAM libre, en réserve 20 % pour la surcharge propre à PyTorch, et estime à environ 100 Mo de mémoire GPU par image de 12 MP — le lot correspond à la plus petite des deux valeurs : la limite dérivée de la mémoire ou la base de la plateforme. Il ne descend jamais en dessous de 1.**Réduction préventive.**Au-delà de**85 % d’utilisation de la VRAM**, la taille des lots est réduite avant toute défaillance.**Réduction de l’allocation par thread.** À mesure que l’utilisation en temps réel augmente, le budget GPU de chaque thread est réduit : ×0,75 au-delà de 80 % d’utilisation, ×0,5 au-delà de 90 %. Les seuils de surveillance sont de 70 % (prudent), 85 % (limite de fonctionnement normale) et 95 % (risque de manque de mémoire).**Recul et récupération en cas de manque de mémoire.** Si un événement de manque de mémoire se produit malgré tout :

* la taille du lot est **divisée par deux**, puis à nouveau divisée par deux à chaque « OOM » consécutif — chaque lot suivant réussi réduit cette pénalité d’un cran
* les allocations des threads actifs sont réduites à 70 % de leur valeur actuelle et l’allocateur passe à sa stratégie prudente, s’assouplissant à nouveau après une série d’allocations réussies
* en cas de forte pression, le pipeline passe de `fused_gpu` à `tiled_gpu`, puis à `cpu_fallback` en dernier recours

**Mémoire vive de l’hôte (Jetson).** Avant le traitement, CLI estime la mémoire maximale de l’hôte à partir du nombre d’images et du mode de débayérisation, et émet un avertissement si la RAM et l’espace d’échange sur disque risquent d’être insuffisants, en affichant les commandes exactes pour ajouter de l’espace d’échange — voir le [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Surveillance de l’adaptation de calcul

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

La stratégie et le nombre de workers sont choisis au sein du backend au début de chaque exécution — il n’y a pas de bannière CLI qui les annonce. Lorsqu’un comportement inattendu se produit (retour au chemin GPU, OOM, débruiteur qui ne se charge pas), cela apparaît dans le journal du backend de cette session :

| Plateforme | Emplacement du journal |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (un fichier par lancement) |
| **Linux, CLI-backend démarré** | ainsi que `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Progression en temps réel

Pendant une exécution, le fichier CLI affiche la progression en temps réel par thread (détection, analyse, traitement, exportation) transmise via des événements Server-Sent Events — ce qui permet de déterminer concrètement si le thread 3 constitue le goulot d&#x27;étranglement. Voir [Pipeline de traitement](processing-pipeline.md).

***

## Étapes suivantes

* [Pipeline de traitement](processing-pipeline.md) — Comprendre l’architecture du pipeline à 4 threads
* [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Déploiement et optimisation spécifiques à Jetson
* [CLI : Ligne de commande](../CLI.md) — Le guide CLI
* [Référence CLI](../reference/cli-reference.md) — Liste exhaustive des commandes pour la version 1.2.0
