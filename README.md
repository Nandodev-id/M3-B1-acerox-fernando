# M3-B1 — Acerox Métallurgie

Analyse d’un nouveau besoin métier et identification de sources de données hétérogènes pour enrichir un modèle existant de prédiction des défauts qualité.

## Objectif du projet

Acerox Métallurgie dispose déjà d’un modèle qui analyse les données des lignes de production afin d’identifier les risques de non-conformité.

L’entreprise souhaite enrichir ce modèle avec trois nouvelles sources :

* les mesures des capteurs IoT ;
* les ordres de production issus de l’ERP ;
* les événements contenus dans les logs machines.

La mission consiste à comprendre le besoin réel du client, explorer les sources, identifier les problèmes de qualité et les risques RGPD, puis proposer une cartographie des flux de données.

Ce brief porte sur l’analyse et la préparation. Il ne comprend pas le développement du pipeline d’ingestion ni la modification du modèle de machine learning.

## Besoin métier reformulé

La demande initiale de Sébastien Marchand était d’ajouter de nouvelles données au modèle existant.

Le besoin métier identifié est plus précis :

> Détecter plus tôt les dérives des lignes de production afin de limiter les pièces non conformes, éviter les arrêts brutaux et permettre aux équipes de maintenance d’intervenir de manière préventive.

Acerox souhaite surtout éviter les défauts non détectés. Le rappel du modèle constitue donc un indicateur prioritaire, tout en maintenant un niveau raisonnable de précision afin de ne pas générer trop d’interventions inutiles.

## Sources analysées

### Capteurs IoT

Fichier :

```text
data/capteurs_iot.csv
```

Caractéristiques principales :

* format CSV ;
* 51 000 lignes ;
* 7 colonnes ;
* 3 sites ;
* 8 capteurs ;
* données couvrant environ un mois.

Principales observations :

* 749 valeurs manquantes dans `vibration_mms`, soit 1,47 % ;
* 1 000 doublons complets ;
* 2 591 températures entre 150 et 160 °C concentrées sur Roubaix, ligne 3 ;
* vibration constante à 12 mm/s sur ces valeurs extrêmes ;
* fréquence annoncée d’une mesure par seconde non confirmée dans l’extrait ;
* intervalle médian réel hors doublons : 266 secondes ;
* intervalle moyen réel hors doublons : 401,41 secondes.

### ERP

Fichier :

```text
data/erp_export.json
```

Caractéristiques principales :

* format JSON ;
* 2 000 ordres de production ;
* 9 champs ;
* export quotidien.

Principales observations :

* 109 valeurs manquantes dans `ouvrier_id`, soit 5,45 % ;
* aucun doublon sur `ordre_id` ;
* 10 références produits ;
* 4 statuts de production ;
* présence de données personnelles indirectement identifiantes ;
* absence du site de Lyon alors qu’il apparaît dans les autres sources.

### Logs machines

Fichier :

```text
data/logs_machines.log
```

Caractéristiques principales :

* texte brut semi-structuré ;
* 30 000 événements ;
* environ 1,8 Mo ;
* fréquence événementielle.

Principales observations :

* 22 501 événements `INFO` ;
* 5 758 événements `WARN` ;
* 1 741 événements `ERROR` ;
* 4 615 événements `operator_login` ;
* aucune ligne malformée détectée avec la structure analysée.

## Fusion multi-sources et provenance

Deux exports supplémentaires ont été étudiés :

```text
data/capteurs_site_A.csv
data/capteurs_site_B.csv
```

La fusion naïve des deux fichiers a révélé trois problèmes :

1. les identifiants `M04` et `M05` existent dans les deux sites ;
2. les températures du site B sont exprimées en Fahrenheit malgré le nom de colonne `temp_c` ;
3. `debit_l_min` est absent du site B, tandis que `firmware` est absent du site A.

Les corrections réalisées sont les suivantes :

* ajout d’une colonne `source` avant la fusion ;
* création d’une clé composite comme `site_A::M04` ;
* conversion des températures du site B en Celsius ;
* conservation de la valeur et de l’unité originales ;
* conservation explicite des valeurs manquantes ;
* aucune valeur de débit inventée.

Le tableau final contient 47 lignes et conserve la provenance de chaque enregistrement.

## Risques RGPD identifiés

Le principal risque ne vient pas d’une colonne isolée, mais du croisement des sources.

La combinaison suivante peut permettre de reconstruire l’activité d’un salarié :

```text
ouvrier_id
+ operator_login
+ site
+ ligne de production
+ timestamp
```

Recommandations :

* vérifier si l’identifiant individuel est réellement nécessaire ;
* supprimer ou pseudonymiser `ouvrier_id` avant ingestion lorsque possible ;
* limiter les droits d’accès ;
* définir une durée de conservation ;
* documenter la finalité du traitement ;
* valider le traitement avec le DPO Acerox.

## Livrables

| Fichier                                   | Description                                              |
| ----------------------------------------- | -------------------------------------------------------- |
| `notes_entretien.md`                      | Notes structurées de l’entretien avec Sébastien Marchand |
| `notebooks/M3-B1_template.ipynb`          | Exploration des trois sources principales                |
| `notebooks/M3-B1_fusion_provenance.ipynb` | Fusion des exports avec conservation de la provenance    |
| `identification_sources.md`               | Note principale d’identification et recommandations      |
| `flux_donnees.md`                         | Schéma Mermaid des flux de données                       |
| `README.md`                               | Présentation et guide de reproduction                    |

## Structure du repo

```text
M3-B1-acerox-fernando/
├── data/
│   ├── capteurs_iot.csv
│   ├── erp_export.json
│   ├── logs_machines.log
│   ├── capteurs_site_A.csv
│   └── capteurs_site_B.csv
├── notebooks/
│   ├── M3-B1_template.ipynb
│   └── M3-B1_fusion_provenance.ipynb
├── ressources/
├── identification_sources.md
├── flux_donnees.md
├── notes_entretien.md
├── requirements.txt
├── .gitignore
└── README.md
```

Les fichiers principaux de données sont exclus de Git par le `.gitignore`.

## Installation

### 1. Cloner le projet

```bash
git clone https://github.com/Nandodev-id/M3-B1-acerox-fernando.git
cd M3-B1-acerox-fernando
```

### 2. Créer l’environnement virtuel

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Sous Windows :

```bash
.venv\Scripts\activate
```

### 3. Installer les dépendances

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Exécution des notebooks

Lancer Jupyter :

```bash
jupyter notebook
```

Ouvrir ensuite :

```text
notebooks/M3-B1_template.ipynb
```

puis :

```text
notebooks/M3-B1_fusion_provenance.ipynb
```

Pour vérifier un notebook :

```text
Kernel → Restart Kernel and Run All Cells
```

Les deux notebooks doivent s’exécuter entièrement sans erreur.

## Principales recommandations

* intégrer les données IoT en priorité, après clarification de leur fréquence réelle et des valeurs extrêmes ;
* intégrer l’ERP pour apporter le contexte de fabrication ;
* intégrer les logs dans un second temps, après documentation de leurs événements ;
* conserver systématiquement la provenance, la version, la fraîcheur et l’unité d’origine ;
* utiliser une clé composite pour les machines des différents sites ;
* ne pas inventer les données absentes ;
* valider les traitements de données personnelles avec le DPO.

## Questions encore ouvertes

* Le fichier IoT est-il complet ou échantillonné ?
* Les températures extrêmes sur Roubaix, ligne 3, correspondent-elles à une situation réelle ou à une saturation du capteur ?
* Quelle clé relie les mesures IoT, les ordres ERP, les logs et les résultats qualité ?
* Pourquoi Lyon apparaît-il dans les capteurs et les logs mais pas dans l’ERP ?
* Le site B mesure-t-il le débit ?
* Quelles sont les performances actuelles du modèle ?
* L’objectif métier est-il une réduction de 20 % des non-conformités ou un taux final de 20 % ?
* L’identité de l’ouvrier est-elle nécessaire pour la prédiction ?

## Limites

* aucune modification du modèle existant ;
* aucun pipeline de production développé ;
* aucune migration de base de données ;
* aucune analyse statistique complète ;
* aucune analyse juridique RGPD formelle ;
* aucune validation métier des événements machines ;
* aucune confirmation de la fréquence réelle du flux IoT complet.

## Restitution

Les points principaux à présenter en trois minutes sont :

* la fréquence IoT annoncée ne correspond pas à la fréquence observée ;
* les températures extrêmes sont concentrées sur une seule ligne et un seul capteur ;
* la fusion de deux exports comparables révèle des différences de clés, d’unités et de schéma ;
* le croisement ERP, logs et horaires peut permettre de réidentifier un salarié ;
* la provenance doit être conservée à chaque étape du flux.

---

Projet réalisé par **Fernando** dans le cadre du brief **M3-B1 — Acerox Métallurgie**, le 21 juillet 2026.