---
layout: default
title: TP 2
parent: Travaux pratiques
permalink: tp/tp2
---

# TP 2 — Migration de paradigme des bases de données  
**Des bases relationnelles (SQL) vers les bases orientées documents (NoSQL)**  
**Durée** : 3 semaines · **Équipe** : Individuel

---

## Contexte

Vous êtes **ingénieur·e de données** sur une plateforme e-commerce. Cette plateforme reçoit **des millions de requêtes par seconde** et les bases relationnelles actuelles ne suffisent plus en performance/scalabilité. L’entreprise décide donc de migrer vers une base **NoSQL (MongoDB)**.

Votre mission :  
1) **Exporter** les données de **PostgreSQL** au format **JSON** ;  
2) **Modéliser** et **importer** dans **MongoDB** (orienté document) ;  
3) **Comparer** et **traduire** vos analyses SQL en **pipelines d’agrégation** MongoDB.

---

## Objectifs d’apprentissage

- Comprendre la migration **PostgreSQL → MongoDB**  
- Savoir **exporter / importer** des **JSON**  
- Modéliser des collections (**imbriqué** vs **référencé**)  
- Créer et **justifier** des **index** (date, texte, géospatial)  
- Traduire des **requêtes SQL** en **pipelines MongoDB**  
- Mettre en œuvre des requêtes avancées : **géospatial**, **full-text**, **bucket**, **facet**

---

## Prérequis

Nous utilisons une **installation locale de MongoDB**.  
⚠️ **Mongo Atlas** peut être **limité** pour ce TP (volumes intermédiaires lors des transformations).

### Logiciels à installer

- **MongoDB 6.0+** — <https://www.mongodb.com/try/download/community>  
- **MongoDB Tools** (mongoimport / mongoexport) — <https://www.mongodb.com/try/download/database-tools>  
- **mongosh** & **MongoDB Compass** — Docs Compass : <https://docs.mongodb.com/compass/current/>  
- **Outils TP1** (psql / pgAdmin)

### Acronymes utiles

| Acronyme | Signification | Exemple / Description |
|---|---|---|
| CRUD | Create / Read / Update / Delete | Opérations de base |
| DDL | Data Definition Language | `CREATE`, `ALTER` |
| DML | Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE` |
| DQL | Data Query Language | `SELECT` |
| I/O | Input / Output | Lecture/écriture disque |

---

# Laboratoire

## 1) Base de données initiale (PostgreSQL)

Téléchargez **`dump_tp1_orig.sql`** (Moodle → *Ressources TP 2*), puis restaurez la base **avant** de commencer.

### Consignes
- Indiquez dans votre **README** la méthode utilisée (**ligne de commande** ou **pgAdmin**).
- La base **`tp1_ind500`** doit contenir **11 tables brutes** avant tout export JSON.

### Méthode rapide (CLI)

```bash
# Création de la base
psql -h localhost -U postgres -d postgres \
  -c "CREATE DATABASE tp1_ind500 ENCODING 'UTF8'"

# Restauration
pg_restore -h localhost -U postgres -d tp1_ind500 \
  --clean --no-owner --no-privileges \
  "....\dump_tp1_orig.sql"

# Vérification (liste des tables)
psql -h localhost -U postgres -d tp1_ind500 -c "\dt"
````

### Méthode graphique (pgAdmin)

1. Créez la base **`tp1_ind500`**.
2. Clic droit → **Restore…**

   * Format : *Custom or tar*
   * Fichier : `dump_tp1_orig.sql`
3. Lancez la restauration : le message **“Process returned exit code 0”** confirme la réussite.
   **Guide** : [https://www.pgadmin.org/docs/pgadmin4/latest/backup\_and\_restore.html#restore](https://www.pgadmin.org/docs/pgadmin4/latest/backup_and_restore.html#restore)

> ℹ️ Tout le monde part du **même dump**. La normalisation et les requêtes avancées se feront **dans MongoDB**.

---

## 2) Schéma source et export JSON

Toutes les tables PostgreSQL doivent être **exportées en JSON** vers `data/`.

### 2.1 Étapes

1. Connexion :

   ```bash
   psql -U postgres -d tp1_ind500 -h localhost
   ```

2. Encodage :

   ```sql
   \encoding UTF8
   ```

3. Export **une table** (ex. générique) :

   ```sql
   \copy (
     SELECT row_to_json(t)
     FROM public.<table> AS t
   ) TO 'C:/Data/<table>.json';
   ```

   📖 `row_to_json` : [https://www.postgresql.org/docs/current/functions-json.html](https://www.postgresql.org/docs/current/functions-json.html)

4. Répétez pour **chaque table** du schéma.

### 2.2 Règle de correspondance (à vérifier)

* **1 ligne PostgreSQL** → **1 document JSON**
* Exemple : `orders` a **98 000** lignes → `orders.json` = **98 000** objets.

---

## 3) Import dans MongoDB

### 3.1 Sélection de la base

```javascript
use tp2_ind500
```

> La base est créée **automatiquement** à la première écriture (pas de `CREATE DATABASE`).

### 3.2 (Optionnel) Pré-création pour Compass

```javascript
db.createCollection("orders");
```

### 3.3 Import JSON (un fichier par collection)

```bash
mongoimport \
  --uri "mongodb://localhost:27017" \
  --db tp2_ind500 \
  --collection <collection> \
  --file "C:/Data/<table>.json" \
  --drop
```

* Remplacez `<collection>` et `<table>` (souvent mêmes noms).
* `--drop` évite les doublons si vous rejouez.

### 3.4 Vérification

```javascript
db.<collection>.countDocuments()
```

Doit **égaler** le nombre de lignes exportées depuis PostgreSQL.

**Livrable** : script d’import (`import.sh` / `import.ps1`) **ou** commandes listées dans `README.md`.

---

## 4) Modélisation MongoDB

### 4.1 Rappels : imbriqué vs référencé

**Imbriqué (embedded)** : sous-documents dans le même document.

* **Quand** : relations **1:1** ou **1\:n** **faible volume**.
* **+** : une seule lecture ; logique simple.
* **–** : documents lourds si volume augmente ; duplication possible.

**Référencé** : on garde l’**identifiant** vers une autre collection.

* **Quand** : **n\:n**, gros volumes, accès séparé.
* **+** : évite la duplication ; gestion indépendante.
* **–** : requêtes plus complexes ; **plusieurs lectures** ; dépend des **index**.

### 4.2 Documentation du modèle (`model.md` + rapport)

Pour **chaque collection** (ex. `orders`, `customers`, etc.) :

1. **Champs racine** & **sous-docs imbriqués**
2. **Références** (ids vers autres collections)
3. **Justification métier** (2–3 phrases : pourquoi imbriquer / référencer)
4. **Schéma visuel** (draw\.io / Lucidchart)

> `model.md` est la **source de vérité** pour l’implémentation (5).

### 4.3 Import → OK, passons à la matérialisation (implémentée en 5)

---

## 5) Matérialisation du modèle (obligatoire)

Créer **`00-build-modeled.js`** (exécuté dans **mongosh**) pour construire vos collections **modélisées** (`tp2_*`) à partir des **collections brutes**.

* Respectez **strictement** vos choix de `model.md`.
* Pipelines **lisibles et commentés** ; utilisez `$lookup`, `$set`, `$unset`, `$project`, **\$merge** (idempotent).

### Exigences

* **Index de jointure** créés **avant** d’exécuter (voir §7).
* **1 doc racine** → **1 doc modélisé** (mêmes cardinalités) ; toute différence doit être **expliquée**.
* **Rejouable** : relancer le script **ne duplique pas**.

### Livrables (5.2)

* `00-build-modeled.js`
* **Captures Compass** : 1 doc **brut** vs 1 doc **modélisé** (par collection)
* **Tableau des collections modélisées** (dans `results.md`) :

  * `Collection modélisée | Sources | Count attendu | Count obtenu | Commentaire`

---

## 6) Normalisation des données

**But** : uniformiser les textes pour de meilleures agrégations / recherches.

### Champs ciblés (min.)

* `customer_city`, `seller_city` → villes normalisées
* `review_comment_message` → texte nettoyé

**Règles** : créer de **nouveaux** champs `*_norm` (**ne pas écraser** l’original) :

* minuscules ;
* retrait **accents** ;
* retrait **symboles** inutiles ;
* trim / espaces normalisés.

**Où** :

* Avant 5 (sur **brut**) et propagation ; **ou**
* Après 5 (directement sur **`tp2_*`**).
  Expliquez votre choix dans le rapport.

### Livrables (6.3)

* `01-normalize.js` (commenté)
* Captures **avant / après** (3 docs **par champ**)
* Note dans `README.md` : où + pourquoi + règles + cas particuliers

---

## 7) Création et justification des index

**Objectifs** :

* **Date d’achat** → tri/filtre rapide
* **Texte** → recherche plein-texte sur commentaires
* **Géospatial** → proximité (2dsphere)

### À faire

* Script **`02-create-indexes.js`** (commenté, chemins adaptés)
* Exécuter dans **mongosh**
* Vérifier dans **Compass → Indexes** (captures)

**Livrables (7.2)**

* `02-create-indexes.js`
* Captures :

  * 2 index sur `tp2_orders` (date, texte)
  * 1 index sur `tp2_sellers_geo` (2dsphere)
* `README.md` : 1 ligne **par index** → **but + test + plan observé**

**Exemples de tests (Explain)**

* Tri date :

  ```javascript
  db.tp2_orders.find().sort({ order_purchase_timestamp: 1 }).explain("executionStats")
  ```
* Plein-texte :

  ```javascript
  db.tp2_orders.find({ $text: { $search: "retard" } }).explain("executionStats")
  ```
* Géo (2dsphere) :

  ```javascript
  db.tp2_sellers_geo.find({
    "geo.location": {
      $near: {
        $geometry: { type: "Point", coordinates: [ -73.56, 45.5 ] }, // [lng, lat]
        $maxDistance: 10000
      }
    }
  }).explain("executionStats")
  ```

> Rappels : **1 seul index texte par collection** ; coordonnées `[longitude, latitude]` ; créez les index **avant** de tester (éviter **COLLSCAN**).

---

## 8) Traduction SQL → pipelines MongoDB (travail individuel)

* **Groupe A** : Q1-A → Q5-A
* **Groupe B** : Q1-B → Q5-B

> Le reste du TP est commun.

**Règle d’or** : partez des collections **modélisées** (`tp2_*`). Dérogation → **justification** (1–2 phrases) requise.

### À réaliser (8.1)

* Fichier **`03-queries.js`**

  * Pour **chaque question** : collez l’**énoncé SQL** (en **commentaire**)
  * Écrivez le **pipeline** équivalent
  * **Commentez** chaque stage (`$match`, `$unwind`, `$group`, …)

* **Exécution + capture** : mongosh ou Compass → 1 **capture** (résultat final) **par question**

* **`results.md`** : pour chaque question

  * ID (ex. Q1-A)
  * **1–2 phrases** d’**interprétation** (ex. « 215 commandes expédiées en retard »)

---

## 9) Requêtes avancées (obligatoires, travail individuel)

Travaillez sur vos **collections modélisées** (`tp2_*`) et codez **4 pipelines** (fichier **`04-advanced-queries.js`**), chacun précédé d’un **bref commentaire** :

1. **\$near** : clients à moins de **10 km** d’un point
2. **\$text** : commentaires contenant **« défectueux »** ou **« retard »**
3. **\$bucket** : tranches de **délai de conversion** (0-7, 8-30, 31-90, >90 jours) sur `tp2_leads`
4. **\$facet** : en **1 requête**, **top 5 produits** ET **top 5 vendeurs** par CA

**Livrables (9.2)**

* `04-advanced-queries.js` (4 pipelines, commentés)
* 4 **captures** (PNG/JPG) montrant :

  * `$near` : distances
  * `$text` : documents trouvés
  * `$bucket` : décompte par tranche
  * `$facet` : deux **tops 5**
* `results.md` : **1 phrase** par pipeline (ex. « 20 clients dans 10 km »)

---

## 10) Environnement d’exécution

**Option A — Local** : installez MongoDB 6.0+, démarrez le service, test :

```bash
mongosh "mongodb://localhost:27017" --eval "db.runCommand({ ping: 1 })"
```

**Option B — Conteneur (recommandé)**

```bash
# Podman
podman run -d --name mongo \
  -p 27017:27017 -v mongodata:/data/db \
  --restart unless-stopped mongo:6.0

# Docker
docker run -d --name mongo \
  -p 27017:27017 -v mongodata:/data/db \
  --restart unless-stopped mongo:6.0
```

---

## 11) Livrable

### 11.1 Archive à remettre

* Nom : **`TP_MongoDB_<votre_nom>.zip`**

### 11.2 Arborescence attendue

```
TP_MongoDB_<votre_nom>/
├─ data/                      # JSON exportés depuis PostgreSQL
│  ├─ orders.json
│  └─ ...
├─ import.sh | import.ps1     # mongoimport rejouable (--drop)
├─ 00-build-modeled.js        # matérialisation tp2_* ($lookup/$set/$unset/$merge)
├─ 01-normalize.js            # champs *_norm (villes, commentaires)
├─ 02-create-indexes.js       # index (date, texte, 2dsphere)
├─ 03-queries.js              # 5 pipelines (votre groupe), SQL en tête + commentaires
├─ 04-advanced-queries.js     # 4 pipelines : $near, $text, $bucket, $facet
├─ captures/                  # preuves (noms explicites)
│  └─ ...
├─ README.md                  # exécution pas-à-pas, ordre des scripts, vérifs
├─ model.md                   # schéma, embedded vs référencé, justifs, diagramme
└─ rapport_final.pdf          # synthèse, limites, pistes d'amélioration
```

### 11.3 Critères d’acceptation (résumé)

* **data/** : JSON **=** lignes SQL (reportez les comptes dans `results.md`)
* **00-build-modeled.js** : idempotent, pas de doublons, cardinalités respectées
* **01-normalize.js** : champs `_norm`, démonstration **avant/après**
* **02-create-indexes.js** : index OK, **Explain** sans **COLLSCAN** évitable
* **03-queries.js** : 5 pipelines (groupe), **SQL en tête**, **stages commentés**
* **04-advanced-queries.js** : 4 pipelines + **captures**
* **README.md** : environnement, commandes, **ordre des scripts**, vérifs
* **rapport\_final.pdf** : clarté, justification, limites, pistes d’amélioration

### 11.4 Check-list (ultime)

* [ ] Export JSON : comptes **OK**
* [ ] Import MongoDB : comptes **OK**
* [ ] Modélisation : **tp2\_**\* générées, **idempotent**
* [ ] Normalisation : champs `_norm` + **captures**
* [ ] Index : **date**, **texte**, **2dsphere** + **Explain**
* [ ] SQL→Mongo : 5 pipelines **commentés** + **captures**
* [ ] Avancées : 4 pipelines + **captures**
* [ ] README + `results.md` complets
* [ ] Rapport final livré

---

## Annexes

### Références

**PostgreSQL / pgAdmin**

* row\_to\_json : [https://www.postgresql.org/docs/current/functions-json.html](https://www.postgresql.org/docs/current/functions-json.html)
* Restauration pgAdmin : [https://www.pgadmin.org/docs/pgadmin4/latest/backup\_and\_restore.html#restore](https://www.pgadmin.org/docs/pgadmin4/latest/backup_and_restore.html#restore)

**MongoDB — Téléchargements & Outils**

* Community Server : [https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
* Database Tools : [https://www.mongodb.com/try/download/database-tools](https://www.mongodb.com/try/download/database-tools)
* Compass (docs) : [https://docs.mongodb.com/compass/current/](https://docs.mongodb.com/compass/current/)

**MongoDB — Documentation**

* Data model design : [https://docs.mongodb.com/manual/core/data-model-design/](https://docs.mongodb.com/manual/core/data-model-design/)
* Aggregation pipeline : [https://docs.mongodb.com/manual/core/aggregation-pipeline/](https://docs.mongodb.com/manual/core/aggregation-pipeline/)
* Aggregation operators : [https://docs.mongodb.com/manual/reference/operator/aggregation/](https://docs.mongodb.com/manual/reference/operator/aggregation/)
* `$lookup` : [https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/)
* `mongoimport` : [https://docs.mongodb.com/manual/reference/program/mongoimport/](https://docs.mongodb.com/manual/reference/program/mongoimport/)
* Indexes : [https://docs.mongodb.com/manual/indexes/](https://docs.mongodb.com/manual/indexes/)
* Geospatial : [https://docs.mongodb.com/manual/geospatial-queries/](https://docs.mongodb.com/manual/geospatial-queries/)
* Text search : [https://docs.mongodb.com/manual/text-search/](https://docs.mongodb.com/manual/text-search/)
* `$bucket` : [https://docs.mongodb.com/manual/reference/operator/aggregation/bucket/](https://docs.mongodb.com/manual/reference/operator/aggregation/bucket/)
* `$facet` : [https://docs.mongodb.com/manual/reference/operator/aggregation/facet/](https://docs.mongodb.com/manual/reference/operator/aggregation/facet/)

---

**Bon TP !**
Pour toute question, consultez la documentation MongoDB ou le forum du cours.
