# Notes d'entretien — Sébastien Marchand (Acerox)

> Notes prises au fil de l'eau pendant l'entretien fictif de 9h30 à 10h00.
> Distinction explicite entre **dit** — ce que Sébastien a formulé — et **interprété** — notre lecture du besoin.

**Date** : 21 juillet 2026 — **Durée** : 30 min — **Présents** : Sébastien Marchand, chef de projet industrialisation Acerox, équipe FastIA dont Fernando

---

## 1. Besoin métier

> Quelle décision Sébastien veut-il améliorer ? Quel KPI métier ?

**Dit :**

* Acerox dispose déjà d’un modèle qui analyse les données issues des capteurs présents sur les lignes de production.
* Le modèle fournit une information sur la conformité des pièces produites.
* L’entreprise souhaite enrichir ce modèle avec de nouvelles sources de données.
* L’objectif est de limiter le nombre de pièces non conformes.
* Acerox souhaite passer d’un fonctionnement réactif, avec parfois un arrêt brutal de la production, à une approche plus préventive.
* Le modèle doit prévenir suffisamment tôt un risque de non-conformité pour permettre une intervention de maintenance.
* L’entreprise préfère déclencher quelques maintenances inutiles plutôt que laisser passer une pièce défectueuse, dans une limite raisonnable.
* Plusieurs sites industriels sont concernés.

**Interprété :**

Le besoin métier n’est pas simplement d’ajouter des données au modèle. Acerox souhaite anticiper les dérives des lignes de production afin de réduire les défauts qualité et d’éviter les arrêts imprévus.

La priorité semble être de réduire les **faux négatifs**, c’est-à-dire les pièces réellement défectueuses que le modèle ne détecte pas. Le **rappel — recall —** paraît donc prioritaire, tout en conservant une précision suffisante pour éviter un nombre excessif d’interventions inutiles.

---

## 2. Sources et formats

> Qu’a-t-il à disposition ? Où ? Sous quel format ?

**Dit :**

Trois nouvelles sources sont disponibles :

* données de capteurs IoT au format CSV ;
* données issues de l’ERP au format JSON ;
* logs des machines au format texte brut.

Les données IoT contiennent des mesures produites sur les lignes de production.

Les données ERP apportent des informations liées aux ordres de production.

Les logs conservent la trace des événements générés par les machines.

**Interprété :**

Les trois sources peuvent être complémentaires :

* les capteurs décrivent l’état physique des machines ;
* l’ERP apporte le contexte de production ;
* les logs expliquent les événements ou incidents survenus sur les lignes.

Le principal enjeu sera de trouver des identifiants ou des règles temporelles permettant de relier correctement ces sources au modèle existant et aux non-conformités observées.

---

## 3. Volumétrie et fréquence

> Combien de données ? À quelle cadence arrivent-elles ?

**Dit :**

* Les capteurs IoT produisent environ une mesure par seconde.
* Les données ERP sont fournies sous la forme d’un export quotidien.
* Les logs machines sont produits au fil des événements.
* Environ un mois de données est disponible.
* Acerox possède trois sites au total.
* Deux sites réalisent de la production.
* Les machines utilisées sur les deux sites de production sont similaires.
* Environ 2 000 ordres de production sont générés sur la période évoquée.

**Interprété :**

La source IoT sera probablement la plus volumineuse et nécessitera une gestion précise des horodatages, de la fréquence et des doublons.

L’ERP sera moins volumineux mais indispensable pour replacer les mesures dans leur contexte de production.

Les logs seront événementiels et semi-structurés. Leur fréquence dépendra de l’activité des machines et des incidents.

La comparaison entre les deux sites de production peut être utile, mais il faudra vérifier que les unités, les identifiants et les schémas de données sont réellement homogènes.

---

## 4. Contraintes — RGPD, sécurité, propriété

> Quelles obligations légales ? Qui possède les données ? Quels accès sécurisés ?

**Dit :**

* Certaines données peuvent être personnelles ou sensibles.
* La conformité du traitement doit être vérifiée avec le DPO.
* Les données peuvent permettre d’identifier des employés ou de suivre leur activité.
* L’infrastructure actuelle semble suffisamment solide pour permettre le déploiement d’un modèle enrichi.
* Une pièce non conforme peut représenter un coût financier très important pour Acerox.

**Interprété :**

Les identifiants d’ouvriers, les connexions opérateurs, les horaires, les sites et les lignes de production peuvent permettre de reconstruire l’activité d’un salarié lorsqu’ils sont croisés.

Même lorsque les sources ne contiennent pas directement un nom, la combinaison des identifiants techniques et des horodatages peut permettre une réidentification indirecte.

Il faudra :

* vérifier si les identifiants individuels sont réellement nécessaires au modèle ;
* pseudonymiser ou supprimer les informations personnelles inutiles ;
* limiter les droits d’accès ;
* définir une durée de conservation ;
* faire valider la finalité du traitement par le DPO.

Le coût exact d’une pièce non conforme n’a pas été confirmé avec suffisamment de précision pendant l’entretien.

---

## 5. Critères de succès

> Comment Sébastien saura-t-il qu’on a réussi ?

**Dit :**

* Le nouveau modèle devra détecter davantage de vraies anomalies.
* Les anomalies détectées devront réellement correspondre à des anomalies.
* Acerox ne veut pas laisser passer des pièces défectueuses.
* Quelques maintenances inutiles peuvent être acceptées, mais leur nombre doit rester raisonnable.
* Un objectif lié à une amélioration de 20 % des non-conformités sur la ligne la plus problématique a été évoqué sur une période de trois mois.
* Les utilisateurs finaux seront principalement les équipes de maintenance.
* Le temps d’inférence et la capacité de l’infrastructure doivent être compatibles avec l’usage opérationnel.

**Interprété :**

Les indicateurs techniques principaux seront probablement :

* le rappel, pour limiter les défauts non détectés ;
* la précision, pour limiter les fausses alertes ;
* le taux de faux négatifs ;
* le taux de faux positifs ;
* le temps d’inférence.

Les indicateurs métier pourraient être :

* la réduction du nombre de pièces non conformes ;
* la réduction des arrêts brutaux ;
* la réduction du coût de non-qualité ;
* l’augmentation du nombre d’interventions préventives pertinentes.

L’objectif de 20 % reste ambigu : il faut confirmer s’il s’agit de réduire les non-conformités de 20 % ou d’atteindre un taux final de 20 %.

---

## Questions restées sans réponse

* Quelles sont les performances exactes du modèle actuel : rappel, précision, faux positifs et faux négatifs ?
* Quel est le défaut principal que le modèle actuel détecte mal ?
* Quelle est la cible exacte associée aux 20 % de non-conformité ?
* Quel est le coût exact d’une pièce non conforme ?
* Quel temps d’inférence maximal est acceptable ?
* Quelle est la taille actuelle du modèle ?
* Quelle infrastructure héberge le modèle et quelles en sont les limites ?
* Quels identifiants permettent de relier les données IoT, l’ERP, les logs et les résultats qualité ?
* Les unités des capteurs sont-elles identiques sur tous les sites ?
* Pourquoi le site de Lyon apparaît-il dans certaines sources mais pas dans l’ERP ?
* Les identifiants d’ouvriers sont-ils réellement nécessaires à la prédiction ?
* Qui possède chaque source et qui autorise son utilisation ?
* Quelle durée de conservation est prévue pour les données détaillées ?
* Existe-t-il une documentation des événements présents dans les logs machines ?

## Mes impressions à chaud

La demande initiale d’Acerox consiste à enrichir un modèle existant, mais le besoin réel est surtout de détecter plus tôt les dérives pouvant provoquer une non-conformité. Les trois sources semblent pertinentes, mais leur croisement crée des difficultés de qualité, de traçabilité et de protection des données personnelles. Avant de promettre une amélioration du modèle, il faudra vérifier les clés de rapprochement, les unités, la fiabilité des données et les performances actuelles servant de référence.

---

*Notes d'entretien produites par Fernando, le 21 juillet 2026.*

