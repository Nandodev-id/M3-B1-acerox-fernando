# Schéma des flux de données — Acerox Métallurgie

> Ce schéma représente les sources identifiées, leur passage par une future couche d’ingestion et de normalisation, leur stockage dans une base pivot, puis leur consommation par le modèle existant de prédiction des défauts qualité.

## Schéma

```mermaid
flowchart LR
    IOT["📡 Capteurs IoT<br/>capteurs_iot.csv<br/>CSV structuré<br/>51 000 mesures<br/>Fréquence annoncée : 1 mesure/s<br/>Fréquence médiane observée : 266 s"]

    ERP["📋 ERP de production<br/>erp_export.json<br/>JSON structuré<br/>2 000 ordres<br/>Export quotidien"]

    LOGS["📝 Logs machines<br/>logs_machines.log<br/>Texte semi-structuré<br/>30 000 événements<br/>Fréquence événementielle"]

    INGEST["🔄 Ingestion multi-sources<br/>À concevoir en M3-B2<br/><br/>Contrôles :<br/>provenance, version, fraîcheur<br/>unités, doublons, schémas"]

    PRIVACY["🔐 Contrôles RGPD<br/>Pseudonymisation<br/>Minimisation des données<br/>Accès et conservation"]

    BDD[("🗄️ BDD pivot<br/>SQLite<br/><br/>Données normalisées<br/>et traçables")]

    MODEL["🧠 Modèle existant Acerox<br/>Prédiction des défauts qualité"]

    MAINT["🛠️ Équipe maintenance<br/>Alertes et aide à la décision"]

    IOT -->|"mesures machine<br/>température, vibration, débit"| INGEST
    ERP -->|"contexte de production<br/>produit, ordre, ligne, quantité"| INGEST
    LOGS -->|"événements machine<br/>alertes, arrêts, maintenance"| INGEST

    INGEST -->|"contrôle des données personnelles"| PRIVACY
    PRIVACY -->|"normalisation + déduplication<br/>harmonisation des unités<br/>conservation de la provenance"| BDD

    BDD -->|"données enrichies"| MODEL
    MODEL -->|"score ou alerte de non-conformité"| MAINT

    classDef source fill:#e1f5ff,stroke:#0277bd,stroke-width:1px
    classDef tofix fill:#fff4e1,stroke:#c97a00,stroke-width:1px,stroke-dasharray:5 5
    classDef security fill:#fce4ec,stroke:#ad1457,stroke-width:1px
    classDef storage fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px
    classDef consumer fill:#ede7f6,stroke:#5e35b1,stroke-width:1px

    class IOT,ERP,LOGS source
    class INGEST tofix
    class PRIVACY security
    class BDD storage
    class MODEL,MAINT consumer
```

## Légende

* **Producteurs :** les capteurs IoT produisent les mesures physiques, l’ERP fournit le contexte des ordres de production et les machines génèrent les logs événementiels.
* **Traitement intermédiaire :** la future ingestion devra conserver la provenance, vérifier la fraîcheur, harmoniser les unités, gérer les doublons et documenter les versions.
* **Protection des données :** les identifiants de salariés doivent être supprimés ou pseudonymisés lorsqu’ils ne sont pas nécessaires.
* **Stockage :** la BDD pivot regroupera des données normalisées, traçables et prêtes à être utilisées.
* **Consommateur final :** le modèle Acerox produit une alerte ou un score destiné à l’équipe de maintenance.

## Contraintes critiques

* La fréquence annoncée des capteurs — une mesure par seconde — n’est pas confirmée dans l’extrait : l’intervalle médian observé est de 266 secondes.
* Les températures extrêmes du capteur `SROU-L3-T01` doivent être confirmées avant ingestion.
* Les identifiants machines ne sont pas uniques entre les sites : la provenance doit être intégrée dans la clé.
* Les températures du site B doivent être converties de Fahrenheit vers Celsius.
* `debit_l_min` n’est pas disponible dans l’export du site B.
* Le croisement de `ouvrier_id`, `operator_login`, du site, de la ligne et de l’heure peut permettre de réidentifier un salarié.

## Décisions associées

* **Sources retenues en priorité :**

  * les capteurs IoT, car ils décrivent directement l’état physique des machines ;
  * l’ERP, car il apporte le contexte des ordres de fabrication ;
  * les logs machines, après documentation et structuration de leurs événements.

* **Sources écartées :**

  * aucune des trois sources principales n’est définitivement écartée ;
  * les logs sont placés en seconde priorité, car leur intégration est plus complexe et présente davantage de risques de suivi individuel.

* **Source bonus — rapports techniques `.md` :**

  * non traitée dans cette analyse ;
  * elle pourra être évaluée ultérieurement pour un cas d’usage documentaire ou RAG, mais elle n’est pas nécessaire au premier enrichissement du modèle tabulaire existant.

* **Traçabilité :**

  * chaque donnée devra conserver au minimum sa source, sa date d’extraction, sa fraîcheur, sa version et son unité d’origine.

---

*Schéma produit par Fernando, le 21 juillet 2026, dans le cadre du brief M3-B1 Acerox Métallurgie.*

