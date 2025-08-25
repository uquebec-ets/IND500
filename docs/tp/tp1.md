---
layout: default
title: TP 1
parent: Travaux pratiques
permalink: tp/tp1
---

# Rapport final — TP1  
**Cours : IND500 - Bases de données distribuées modernes**  
**Titre : Mise en place d’une base de données SQL**  
**Durée : 3 semaines**  
**Équipe : Individuel**

---

## 1) Objectifs du laboratoire
- Mettre en place une base relationnelle à partir de données brutes (~100 000 commandes, 2016–2018).
- Concevoir un pipeline **staging → transformation → modèle final**.
- Appliquer **intégrité référentielle**, **contraintes**, **typage** appropriés.
- Exécuter des **requêtes analytiques** et **optimiser** leur performance.

---

## 2) Données et modèle (résumé)
- **Commandes** : `orders`, `order_items`, `order_payments`, `order_reviews`.
- **Produits** : `products`, `product_category_name_translation`.
- **Acteurs & localisation** : `customers` (distinction `customer_id` / `customer_unique_id`), `sellers`, `geolocation`.
- **Acquisition vendeurs** : `leads_qualified`, `leads_closed` (prospects avec `seller_id` temporaire, `sr_id`, `sdr_id`).

> Rappel calcul total commande : somme des `price` + `freight_value` de **toutes** les lignes `order_items` rattachées à la commande.

---

## 3) Choix de conception (types & contraintes)

### 3.1 Types SQL retenus
- **INT** : identifiants, compteurs.  
- **NUMERIC(p,s)** : montants (`price`, `freight_value`) — *p.ex.* `NUMERIC(12,2)`.  
- **TIMESTAMP** : événements (`order_purchase_timestamp`, `order_approved_at`, etc.).  
- **VARCHAR(n)** : codes courts (état, ville, code postal).  
- **TEXT** : libellés/ commentaires (titres d’avis, catégories).  
- **BOOLEAN** : drapeaux logiques si requis.  
- **DOUBLE PRECISION** : mesures continues si nécessaire.

### 3.2 Contraintes d’intégrité
- **PRIMARY KEY** sur chaque table.  
- **FOREIGN KEY** pour lier commandes ↔ items ↔ clients ↔ vendeurs ↔ produits.  
- **NOT NULL** sur colonnes critiques.  
- **UNIQUE** (ex. `customer_unique_id` si utilisé comme identifiant réel).  

> Les contraintes sont **définies au CREATE TABLE** (ordre de création cohérent). Pas d’index additionnels au-delà des PK/FK auto-générées à ce stade.

---

## 4) Pipeline ETL

### 4.1 Staging (brut)
- Tables **sans contraintes**, **toutes colonnes en TEXT**.
- Chargement via `COPY` depuis les CSV.

### 4.2 Transformations en staging
- **Dates/temps** : ajout de colonnes `TIMESTAMP` et conversion depuis `TEXT`.
- **Chaînes** : `trim`, minuscules, suppression accents, caractères non alphanumériques (sauf apostrophes / tirets), réduction espaces multiples.
- Contrôles de cohérence (formats, valeurs nulles anormales, doublons apparents).

### 4.3 Passage au modèle final
- **INSERT INTO … SELECT …** depuis le staging :  
  - conversions explicites (**CAST**),  
  - normalisations (**NULLIF**, **COALESCE**),  
  - **DISTINCT** si nécessaire,  
  - **JOIN** garantissant l’existence des référents (FK).  
- **Interdiction d’utiliser `ON CONFLICT`**.

---

## 5) Requêtes guidées (à inclure dans `queries.sql`)

### 5.1 Variation A (si attribuée)
- **Q1-A** — Comptage par état (`customer_state`) des **commandes**.  
- **Q2-A** — **Top 10** des commandes par **montant total** (prix + frais).  
- **Q3-A** — **Top 10 vendeurs** par **chiffre d’affaires** (prix + frais).  
- **Q4-A** — **Dépenses mensuelles par état** + **cumul** (mois = `YYYY-MM` basé sur `order_purchase_timestamp`).  
- **Q5-A** — **Analyse mensuelle du CA par vendeur** (6 derniers mois) :  
  `order_month`, `seller_city`, `num_orders`, `total_sales`, `median_order_value` (`PERCENTILE_CONT(0.5)`), `cumulative_customers` (corrélation).

### 5.2 Variation B (si attribuée)
- **Q1-B** — **Clients réels** par état via `customer_unique_id`.  
- **Q2-B** — **Moyenne** de lignes par **commande**.  
- **Q3-B** — **Top 5 catégories** (`product_category_name`) par **nombre de commandes distinctes**.  
- **Q4-B** — **Répartition des ventes** par vendeur + **%** et **% cumulés** (prix + frais).  
- **Q5-B** — **Médiane valeur de commande** + **cumul clients distincts** (6 derniers mois).

---

## 6) Optimisation (focalisée sur Q5)

### 6.1 Profilage de la version brute
- Exécuter :  
  ```sql
  EXPLAIN (ANALYZE, BUFFERS)
  -- requête Q5 brute
  ;
````

* Repérer nœuds coûteux : **Seq Scan** volumineux, **Sort** avec spill, **WindowAgg**, **HashAggregate**, **CTE Scan** matérialisés, **Nested Loop** explosifs.

### 6.2 Correctifs proposés et implémentés

* **Indexation ciblée** des colonnes filtrées/triées/jointes :

  * `orders(order_purchase_timestamp)` ou colonne matérialisée `order_month` (+ index),
  * `order_items(order_id)`, `order_items(seller_id)`,
  * éventuellement index fonctionnel sur `date_trunc('month', order_purchase_timestamp)`.
* **Réécriture** pour **réduire le volume** :

  * éliminer/fusionner CTE redondants,
  * **pré-agréger** `order_items` (par `order_id`, et `seller_id` si requis),
  * remplacer corrélations par **fenêtres** ou **JOIN** sur agrégats par `mois/ville`,
  * **matérialiser** `order_month` pour éviter `date_trunc` répétés.

### 6.3 Mesures avant / après

> Renseigner tous les champs ci-dessous à partir des sorties `EXPLAIN (ANALYZE, BUFFERS)`.

| Critère                     | Avant (brute)                                            | Après (optimisée)             |
| --------------------------- | -------------------------------------------------------- | ----------------------------- |
| Temps total (ms)            | \_\_\_\_\_\_                                             | \_\_\_\_\_\_                  |
| Buffers — shared hit / read | \_\_ / \_\_                                              | \_\_ / \_\_                   |
| Buffers — temp read / write | \_\_ / \_\_                                              | \_\_ / \_\_                   |
| Nœud le plus coûteux        | \_\_\_\_\_\_                                             | \_\_\_\_\_\_ (réduit/éliminé) |
| Changements appliqués       | \[index] \[réécriture] \[order\_month matérialisée] \[…] |                               |
| Impact observé              | ex. **-65%** temps, **0 spill**                          |                               |

---

## 7) Résultats et discussion

* **Synthèse** des requêtes Q1–Q5 (résultats clés, vérifications).
* **Gains** de performance observés après optimisation.
* **Limites** (qualité des données, choix de types, coûts restants).
* **Pistes** futures (index additionnels, partitionnement, compression, parallel query).

---

## 8) Fichiers remis (Moodle)

* `create_staging.sql`
* `load_staging.sql`
* `transform_staging.sql`
* `create_finales.sql`
* `load_finales.sql`
* `queries.sql` *(Q1–Q5)*
* **Correctif d’optimisation Q5** — `optimize_q5.sql` + **captures/mesures avant/après**
* **Rapport final** *(ce document)*

---

## Annexe — Sommaire (style PDF)

* **Laboratoire 1 :** Mise en place d’une base de données SQL
* **Modèle de données :**

  * Tables de commandes, Données produites, Clients/Vendeurs/Géolocalisation, Pipeline d’acquisition vendeurs
* **Travail à faire :**

  * 1. Initialisation
  * 2. Staging & Normalisation

    * 2.1 Création des tables de staging
    * 2.2 Import CSV
    * 2.3 Nettoyage & standardisation
  * 3. Modèle final & chargement

    * 3.1 Tables finales
    * 3.2 Transfert vers finales
  * 4. Requêtes guidées

    * Variation A : Q1–Q5 (+ optimisation)
    * Variation B : Q1–Q5 (+ optimisation)
  * 5. Optimisation des requêtes
  * **Gabarit comparaison** (avant/après)
  * **Remise** (scripts + rapport)
