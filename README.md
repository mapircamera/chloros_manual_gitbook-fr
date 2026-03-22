---
metaLinks: {}
---

# Mise en route

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros est une application logicielle de [MAPIR](https://www.mapir.camera) permettant de traiter des images et d&#x27;autres données de capteurs.

***{% hint style="success" %}**Nouveautés de Chloros 1.1.0** : prise en charge native de Linux (amd64 et arm64), edge computing NVIDIA Jetson, adaptation dynamique des calculs, pipeline de traitement à 4 threads, nouvelles commandes et options CLI. Voir [Téléchargement](download.md) pour le journal des modifications complet.
{% endhint %}

Chloros est disponible en 3 modes d&#x27;application :

## Chloros : Application GUI de bureau

Fenêtre autonome distincte avec toutes les fonctionnalités. _Windows uniquement._

## [Chloros CLI : Interface en ligne de commande](CLI.md)

Traitement par lots en ligne de commande. Idéal pour l&#x27;automatisation, la création de scripts et le fonctionnement sans interface graphique. Disponible sur **Windows, Linux amd64 et Linux arm64 (NVIDIA Jetson)**. _L&#x27;interface CLI nécessite une licence Chloros+ pour y accéder._

## [Chloros API : Python SDK](api-python-sdk.md)

Interface programmatique Python pour l&#x27;automatisation et les flux de travail personnalisés. Idéale pour les pipelines de recherche, l&#x27;intégration avec des applications Python existantes et la création d&#x27;outils personnalisés. Disponible sur **toutes les plateformes** via `pip install chloros-sdk`. _L&#x27;API nécessite une licence Chloros+ pour y accéder._***

## Plateformes prises en charge

| Plateforme | Interface graphique | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Oui | Oui | Oui |
| **Linux amd64 (x86_64)** | Non | Oui | Oui |
| **Linux arm64 (NVIDIA Jetson)** | Non | Oui | Oui |

Pour les instructions d&#x27;installation de Linux, consultez la section [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Bien que Chloros soit gratuit pour la plupart des tâches, vous pourriez avoir besoin de fonctionnalités supplémentaires. C&#x27;est là qu&#x27;une licence payante pour Chloros+ peut vous être utile. Avec une licence Chloros+, vous pouvez débloquer de nouvelles fonctionnalités telles que :

* **Traitement multithread** : accélérez considérablement le traitement des images pour les projets de grande envergure en traitant simultanément les images via le pipeline.
* **Accélération GPU (CUDA)** : tirez parti des options de mémoire GPU plus performantes d&#x27;aujourd&#x27;hui pour accélérer encore davantage le pipeline de traitement d&#x27;images. Nous recommandons 4 Go ou plus de VRAM pour obtenir les meilleurs résultats.
* **Chloros+**[**CLI**](CLI.md)**Accès** : exécutez Chloros+ depuis la ligne de commande pour automatiser et intégrer le logiciel à votre propre application.
* **Chloros+**[**API**](api-python-sdk.md)**Accès :** exécutez Chloros+ depuis Python pour un contrôle programmatique, permettant une intégration transparente avec vos pipelines de recherche, vos workflows d&#x27;analyse de données et vos applications personnalisées.
* **Utilisation sur plusieurs appareils** : chaque licence Chloros+ permet d&#x27;enregistrer au moins 2 appareils. Utilisez votre compte MAPIR Cloud pour gérer les appareils enregistrés. Ajoutez la prise en charge d&#x27;appareils supplémentaires en mettant à niveau votre licence Chloros+.
* **Méthode avancée de débayérisation sensible à la texture :** un débayériseur de haute qualité sensible aux contours, combiné à un modèle de débruitage IA/ML qui élimine la quasi-totalité du bruit de débayérisation. 
* **Formules d&#x27;indices multispectraux personnalisées :** saisissez des indices multispectraux personnalisés dans les calculateurs raster Chloros, tant pour le traitement que pour le bac à sable de visualisation d&#x27;images.
* **Linux et Edge Computing :** exécutez Chloros sur les plateformes Linux x86_64 et ARM64, y compris NVIDIA Jetson, pour le traitement sur le terrain et en périphérie. Voir [Présentation de Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Tarifs et inscription à Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
