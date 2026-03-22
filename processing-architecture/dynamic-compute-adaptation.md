# Adaptation dynamique du calcul

Chloros 1.1.0 introduit la détection intelligente du matériel et la sélection automatique de la stratégie de traitement. Le moteur de traitement s&#x27;adapte à votre matériel — du Jetson Nano à une station de travail multi-GPU — sans aucune configuration manuelle.

***

## Comment ça marche

Au démarrage de Chloros, le système effectue automatiquement un profilage de votre système :

1. **Détecte le système d&#x27;exploitation** — Windows ou Linux
2. **Identifie les cœurs de processeur et la mémoire vive totale**

3.**Détecte la présence d’un GPU** — capacité NVIDIA CUDA, VRAM, modèle
4. **Identifie le modèle Jetson** (le cas échéant) — via `/proc/device-tree/model`
5. **Vérifie les capteurs thermiques** (Jetson) — pour un traitement tenant compte de la température
6. **Sélectionne la stratégie de calcul optimale** — en fonction de l&#x27;ensemble du matériel détecté
7. **Configure automatiquement le nombre de workers, le type de pipeline et l&#x27;allocation de mémoire**Le résultat est mis en cache afin que les exécutions suivantes démarrent plus rapidement. Si le matériel change (par exemple, si un GPU est ajouté), Chloros effectue un nouveau profilage au prochain lancement.***

## Stratégies de calcul

Chloros sélectionne l&#x27;une des trois stratégies de calcul en fonction de votre matériel :

| Stratégie | GPU requis | Tâches | Pipeline | Idéal pour |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Oui (12 Go+ de VRAM ou 16 Go+ partagés) | 3-4 | `fused_gpu` | GPU de bureau avec 12 Go+, Jetson Orin NX 16 Go, AGX Orin |
| **`GPU_SINGLE`** | Oui (&lt; 12 Go de VRAM) | 1-3 | `tiled_gpu` | GPU d&#x27;entrée de gamme, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Non | cœurs - 1 | `cpu_fallback` | Systèmes sans GPU NVIDIA |

### Types de pipeline

* **`fused_gpu`** — Chemin de traitement GPU complet. Toutes les opérations de débayérisation, de correction et d&#x27;indexation s&#x27;exécutent sur le GPU en un seul passage fusionné. Débit maximal, mais nécessite davantage de VRAM.
* **`tiled_gpu`** — Chemin GPU économe en mémoire. Traite les images par tuiles pour s&#x27;adapter à la mémoire GPU limitée. Débit réduit, mais fonctionne sur les appareils à mémoire limitée.
* **`cpu_fallback`** — Traitement sur CPU uniquement utilisant le parallélisme multithread. Utilisé lorsqu&#x27;aucun GPU NVIDIA n&#x27;est disponible.***

## Comportement spécifique à la plateforme

| Plateforme | Stratégie | Workers | Pipeline | Remarques |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 Go** | `GPU_SINGLE` | 1 | `tiled_gpu` (sérialisé) | Mode économe en mémoire, traite une image à la fois |
| **Jetson Orin NX 16 Go** | `GPU_PARALLEL` | 3 | `fused_gpu` (concurrent) | Appareil périphérique recommandé — traitement GPU parallèle réel |
| **Jetson AGX Orin 64 Go** | `GPU_PARALLEL` | 4 | `fused_gpu` (concurrent) | Performances de périphérie maximales |
| **Ordinateur de bureau avec GPU 8 Go** | `GPU_SINGLE` | 3 | `tiled_gpu` | Bonnes performances de bureau avec des tuiles économes en mémoire |
| **Ordinateur de bureau avec GPU de 12 Go ou plus** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Performances optimales sur ordinateur de bureau |
| **Système CPU uniquement** | `CPU_PARALLEL` | cœurs - 1 | `cpu_fallback` | Aucun GPU requis, utilise ThreadPool |

{% hint style="info" %}
**Mémoire unifiée Jetson** : les appareils Jetson partagent la mémoire du GPU et du CPU. Un Jetson Orin NX 16 Go affiche environ 15,3 Go de VRAM, mais il s&#x27;agit de la même mémoire RAM physique utilisée par le système d&#x27;exploitation et les processus du CPU. Chloros en tient compte lors de la définition des seuils d&#x27;allocation de mémoire.
{% endhint %}

***

## Allocation dynamique de la mémoire GPU

Chloros utilise un [pipeline de traitement à 4 threads](processing-pipeline.md) :

* **Thread 1** (Détection) — Chargement de l&#x27;image, analyse EXIF, détection de la cible
* **Thread 2** (Calibrage) — Calcul du calibrage de réflectance
* **Thread 3** (Traitement) — Débayérisation GPU, correction du vignettage, calcul de l&#x27;indice
* **Thread 4** (Exportation) — Écriture de fichiers, intégration des métadonnées

À mesure que les threads précédents du pipeline terminent leur travail (par exemple, lorsque toutes les images ont été détectées), leur allocation de mémoire GPU est libérée et **redistribuée aux threads actifs restants**. Cela signifie que le thread 3 (l&#x27;étape la plus gourmande en ressources GPU) dispose progressivement de plus de mémoire à mesure que le pipeline avance, ce qui améliore le débit pour les tâches les plus gourmandes en calcul.

### Étapes d&#x27;allocation

| Étape | Threads actifs | Répartition de la mémoire GPU |
| --- | --- | --- |
| **Début** | 1, 2, 3, 4 | Répartie entre tous les threads |
| **Début-milieu** | 2, 3, 4 | Mémoire du thread 1 redistribuée |
| **Milieu-fin** | 3, 4 | La mémoire des threads 1 et 2 est allouée aux threads 3 et 4 |
| **Fin** | 3 ou 4 | Mémoire maximale pour le thread restant |***

## Traitement Texture Aware

La méthode de débayérisation Texture Aware (Chloros+ uniquement) utilise nettement plus de mémoire GPU que la méthode Standard en raison du modèle de débruitage IA/ML :

* Les systèmes disposant de **moins de 7 Go de VRAM** sont contraints d&#x27;utiliser une boucle de traitement synchrone pour le mode Texture Aware (une image à la fois)
* Les systèmes dotés de **7 Go ou plus de VRAM** peuvent traiter le mode « Texture Aware » en parallèle, mais avec un nombre de workers réduit par rapport au mode Standard***

## Gestion thermique (Jetson)

Les appareils Jetson sont soumis à des contraintes thermiques, en particulier dans les déploiements en espace clos ou aéroportés. Chloros surveille les températures du GPU et du CPU et ajuste automatiquement le traitement :

| Température | Réaction |
| --- | --- |
| **&lt; 70 °C** | Fonctionnement normal — pleine vitesse |
| **70 °C** (Avertissement) | Réduire la taille des lots |
| **80 °C** (Critique) | Limitation agressive — réduire la concurrence et le nombre de workers |
| **90 °C** (Arrêt) | Arrêt complet du traitement GPU |

La surveillance de la température utilise `tegrastats` sur les plateformes Jetson. Sur les systèmes de bureau dotés d&#x27;un refroidissement adéquat, la limitation thermique est rarement déclenchée.

***

## Gestion de la pression mémoire

Chloros surveille la pression sur la mémoire système pendant le traitement :

* **Seuil de mémoire** : une utilisation de 85 % déclenche un comportement prudent
* **Réduction OOM** : si un événement de mémoire insuffisante se produit, l&#x27;allocation est réduite de 25 % (multiplicateur de 0,75)
* **Retour au pipeline** : En cas de forte pression sur la mémoire, le pipeline bascule automatiquement de `fused_gpu` vers `tiled_gpu`
* **Recommandations relatives à l&#x27;espace d&#x27;échange** : sur Jetson, Chloros vous avertit si l&#x27;espace d&#x27;échange est insuffisant pour la taille de votre ensemble de données***

## Surveillance de l&#x27;adaptation du calcul

### Sortie d&#x27;état de CLI

Au démarrage du traitement, CLI affiche le profil matériel détecté :

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnostics système

Exécutez `chloros-cli selftest` pour afficher un profil matériel complet et vérifier les capacités de calcul :

```bash
chloros-cli selftest
```

Cela permet de vérifier la disponibilité de CUDA, la mémoire GPU, les modèles de débruitage et la connectivité du backend.

***

## Étapes suivantes

* [Pipeline de traitement](processing-pipeline.md) — Comprendre l&#x27;architecture du pipeline à 4 threads
* [Guide NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Déploiement et optimisation spécifiques à Jetson
* [CLI : Ligne de commande](../CLI.md) — Référence complète de CLI
