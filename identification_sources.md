# Note d'identification des sources — Acerox Métallurgie

> Document remis à **Sébastien Marchand**, chef de projet industrialisation chez Acerox.
> Public : décideur métier non technique.
> **Auteur :** Fernando — **Date :** 21 juillet 2026

## 1. Contexte

Acerox Métallurgie est une entreprise industrielle d’environ 800 salariés répartis sur trois sites, dont deux sites de production. L’entreprise dispose déjà d’un modèle qui analyse les mesures issues des lignes de production afin d’évaluer la conformité des pièces fabriquées. Acerox souhaite aujourd’hui enrichir ce modèle avec de nouvelles sources de données : les capteurs IoT, l’ERP et les logs machines. La mission FastIA consiste à identifier les sources pertinentes, évaluer leur qualité et leurs risques, puis proposer un flux de données cohérent vers le modèle existant.

## 2. Demande métier reformulée

**Ce que Sébastien a demandé :**

Sébastien souhaite enrichir le modèle existant avec de nouvelles sources de données afin d’améliorer la détection des défauts qualité.

**Ce que je comprends qu’il cherche vraiment :**

Le besoin métier est de passer d’une logique principalement réactive à une logique plus préventive. Acerox souhaite détecter suffisamment tôt les dérives de fonctionnement des lignes afin de limiter les pièces non conformes, éviter les arrêts brutaux de production et permettre aux équipes de maintenance d’intervenir avant qu’un défaut important ne se produise.

La priorité semble être de ne pas laisser passer de pièces défectueuses. Le modèle devra donc détecter davantage de vraies anomalies, même si cela entraîne quelques interventions de maintenance inutiles, à condition que leur nombre reste raisonnable.

## 3. Inventaire des sources

| Source                                         | Format                    |                                               Volume | Fréquence                                                                                                                                                                                                                           | Qualité observée                                                                                                                                                                                                                                                                                                                                                                       | Risques RGPD                                                                                                                                                                                  | Pertinence métier                                                                                                                                                                                   |
| ---------------------------------------------- | ------------------------- | ---------------------------------------------------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `capteurs_iot.csv`                             | CSV structuré             |            51 000 lignes, 7 colonnes, environ 3,6 Mo | Une mesure par seconde annoncée. Après retrait des doublons temporels, l’intervalle médian réel est de 266 secondes et l’intervalle moyen de 401,41 secondes. Seulement 0,26 % des intervalles sont exactement égaux à une seconde. | 749 valeurs manquantes dans `vibration_mms`, soit 1,47 %, et 1 000 doublons complets. Les 2 591 températures entre 150 et 160 °C sont toutes concentrées sur le site de Roubaix, ligne 3, capteur `SROU-L3-T01`. Sur ces lignes, la vibration vaut toujours 12 mm/s, avec un écart-type nul. Le fichier semble échantillonné, filtré ou incomplet par rapport à la fréquence annoncée. | Pas d’identité directe. Toutefois, le croisement du site, de la ligne, du capteur et de l’horodatage avec l’ERP, les logs ou un planning peut permettre d’associer une activité à un salarié. | **Priorité haute.** Cette source permet d’observer les dérives de température, vibration et débit pouvant précéder une non-conformité.                                                              |
| `erp_export.json`                              | JSON structuré            | 2 000 ordres de production, 9 champs, environ 544 Ko | Export quotidien sur environ un mois                                                                                                                                                                                                | 109 valeurs manquantes dans `ouvrier_id`, soit 5,45 %. Aucun doublon sur `ordre_id`. Les dates sont stockées comme du texte. L’ERP couvre uniquement Roubaix et Saint-Étienne, contrairement aux capteurs et aux logs qui contiennent également Lyon.                                                                                                                                  | `ouvrier_id` est une donnée personnelle indirectement identifiante. Croisé avec les horaires, les lignes et les logs, il peut permettre de reconstruire l’activité d’un salarié.              | **Priorité haute.** L’ERP apporte le contexte de fabrication : produit, ordre de production, site, ligne, statut, quantité et période.                                                              |
| `logs_machines.log`                            | Texte brut semi-structuré |                    30 000 événements, environ 1,8 Mo | Événementielle, au fil de l’activité des machines                                                                                                                                                                                   | Les 30 000 lignes respectent la structure analysée. Aucune ligne malformée n’a été détectée. Le fichier contient 22 501 événements `INFO`, 5 758 `WARN` et 1 741 `ERROR`. Une documentation officielle des événements reste nécessaire.                                                                                                                                                | Les 4 615 événements `operator_login`, croisés avec `ouvrier_id`, le site, la ligne et l’heure, peuvent permettre de suivre ou réidentifier un salarié.                                       | **Priorité moyenne à haute.** Les logs peuvent expliquer les événements précédant un défaut : dérive de température, dépassement de vibration, maintenance, arrêt d’urgence ou changement d’équipe. |
| `capteurs_site_A.csv` et `capteurs_site_B.csv` | Deux fichiers CSV         | 25 lignes pour le site A et 22 lignes pour le site B | Extraits ponctuels                                                                                                                                                                                                                  | Les deux fichiers ont des schémas différents. `debit_l_min` existe uniquement dans le site A et `firmware` uniquement dans le site B. Les identifiants `M04` et `M05` existent dans les deux sites. Les températures du site B étaient exprimées en Fahrenheit malgré le nom `temp_c`.                                                                                                 | La colonne `source` doit être conservée afin de garantir la provenance, la traçabilité et la possibilité de filtrer les données par site.                                                     | **Source pédagogique de fusion.** Elle montre les contrôles nécessaires avant de réunir des exports provenant de plusieurs sites.                                                                   |

### Fusion des exports des sites A et B

Les exports des sites A et B ont été réunis avec une colonne `source` afin de conserver la provenance de chaque ligne.

Cette fusion a révélé trois écarts :

1. des identifiants de machines identiques entre les deux sites ;
2. une différence d’unité de température ;
3. une mesure de débit absente du site B.

Une clé composite, comme `site_A::M04`, a été créée pour distinguer les machines portant le même identifiant local. Les températures du site B ont été converties de Fahrenheit vers Celsius, tout en conservant la valeur et l’unité originales. Les valeurs de débit absentes n’ont pas été inventées.

La colonne `source` est conservée car elle permet de tracer l’origine de chaque ligne, de résoudre les conflits et de vérifier les transformations appliquées.

## 4. Recommandations

* **Intégrer les données IoT en priorité**, car elles sont directement liées à l’état des machines et peuvent aider à détecter des dérives avant la production d’une pièce non conforme. Leur fréquence réelle et les valeurs extrêmes doivent cependant être clarifiées avant utilisation.

* **Intégrer l’ERP en parallèle**, car il apporte le contexte indispensable pour relier les mesures à un produit, une ligne et un ordre de production. Une clé de rapprochement fiable avec les capteurs, les logs et les résultats qualité doit être définie.

* **Intégrer les logs dans un second temps**, après documentation de leur structure et de leurs événements. Ils sont utiles pour expliquer les anomalies, mais plus complexes à structurer et plus risqués du point de vue du suivi individuel.

* **Conserver systématiquement la provenance des données** avec une colonne `source`, une date d’extraction, une version du fichier et, lorsque cela est nécessaire, l’unité originale.

* **Limiter l’usage des identifiants de salariés**. `ouvrier_id` et les informations de connexion doivent être supprimés ou pseudonymisés si l’identité individuelle n’est pas nécessaire au modèle. La finalité du traitement, les droits d’accès et la durée de conservation doivent être validés avec le DPO.

## 5. Points à clarifier avec Sébastien

1. Le fichier IoT est-il un export complet du flux seconde par seconde ou un échantillon filtré ?
2. Les températures comprises entre 150 et 160 °C sur Roubaix, ligne 3, ainsi que la vibration constante à 12 mm/s, correspondent-elles à un état machine réel ou à une saturation du capteur ?
3. Quelle clé permet de relier précisément les capteurs, les ordres ERP, les logs machines et les résultats de conformité ?
4. Pourquoi le site de Lyon apparaît-il dans les capteurs et les logs, mais pas dans l’ERP ?
5. Le site B ne mesure-t-il pas le débit, ou la colonne `debit_l_min` a-t-elle simplement été oubliée dans l’export ?
6. Quelles sont les performances actuelles du modèle, notamment son rappel, sa précision et son taux de faux négatifs ?
7. L’objectif annoncé correspond-il à une réduction de 20 % des non-conformités ou à un taux final de 20 % ?
8. L’identité de l’ouvrier est-elle réellement nécessaire pour prédire les défauts qualité ?

## 6. Limites de cette note

* L’analyse réalisée est une identification initiale des sources, et non une étude statistique complète.
* Aucun modèle de machine learning n’a été entraîné ou modifié.
* Aucun pipeline d’ingestion en production n’a été développé.
* La relation exacte entre les données et les non-conformités n’a pas encore été testée.
* La fréquence réelle du flux IoT complet n’a pas été confirmée.
* Les événements des logs n’ont pas été validés avec une documentation métier officielle.
* Aucune analyse d’impact relative à la protection des données n’a été réalisée. Les risques identifiés doivent être transmis au DPO Acerox.
* Les règles de propriété, d’accès et de conservation des différentes sources restent à préciser.

---

*Note produite par Fernando, le 21 juillet 2026, dans le cadre du brief M3-B1 Acerox Métallurgie.*
