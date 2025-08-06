---
layout: default
title: TP 1
parent: Travaux pratiques
permalink: tp/tp1
---

# Énoncé TP 1
Ce fournisseur de produits de commerce électronique a récolté les commandes passées sur son magasin en ligne. Il contient environ 100 000 commandes enregistrées entre 2016 et 2018. Les données permettent d’analyser :
•	Le statut des commandes, leur montant (prix + frais de port), le paiement et la livraison,
•	La localisation des clients,
•	Les caractéristiques des produits,
•	Les avis clients (note, titre, commentaire) sur chaque produit.

Comme sur Amazon, après commande, le vendeur est notifié. Dès réception du produit (ou à la date de livraison estimée), le client reçoit une enquête de satisfaction par courriel. Une commande peut contenir plusieurs items, chacun approvisionné par un partenaire différent : Starks, Lannisters, Arryns, Greyjoys, Tullys, Tyrells, Baratheons of Storm’s End, Martells

Shéma de la base de données
<img width="371" height="288" alt="image" src="https://github.com/user-attachments/assets/4e5cd2c6-0e02-4206-93ae-21d40abac61e" />

Tables finales attendues
•	tp1_ind500_product_category_name_translation
•	tp1_ind500_sellers
•	tp1_ind500_customers
•	tp1_ind500_products
•	tp1_ind500_geolocation
•	tp1_ind500_leads_qualified
•	tp1_ind500_leads_closed
•	tp1_ind500_orders
•	tp1_ind500_order_items
•	tp1_ind500_order_payments
•	tp1_ind500_order_reviews

Travail à faire
1) Chargement initial (Étape 0 – x points)
    1.	Dossier data/
        •	Y déposer tous les CSV bruts (préfixés tp1_ind500_stage_…), chacun contenant la colonne exercise_noise.

2.	README
        •	Lister les fichiers du répertoire data/ et indiquer à quelle table de staging ils correspondent.
        •	Expliquer l’ordre d’exécution des scripts :
              1.	ddl_staging.sql (création des tables de staging)
              2.	load_staging.sql (import + nettoyage des staging)
              3.	ddl_finales.sql (création des tables finales)
              4.	load_finales.sql (transfert staging vers finales)
              5.	queries.sql (Q1–Q5)

3.	DDL staging (ddl_staging.sql)
        •	Créer les tables de staging, incluant la colonne exercise_noise
        •	Dans un seul fichier ddl_staging.sql.

4.	Import en staging (load_staging.sql)
        •	Écrire les 11 scripts COPY … FROM 'data/…csv' pour peupler les tables staging.

5.	Nettoyage minimal & vérification (load_staging.sql)
        •	Normalisation des dates (conversion en TIMESTAMP columns + suffix _ts etc.).
        •	Standardisation des chaînes (trim(), initcap(), unaccent()).
        •	Suppression des doublons
        •	Vérifier volumes attendus (SELECT COUNT(*) …).

2) Tables finales & chargement (Étape 1 – x points)
1.	Adaptation du schéma
        •	Adapter le schéma SQLite vers PostgreSQL.

2.	Création des tables finales (ddl_finales.sql)
        •	Exécuter le script DDL pour créer les 11 tables finales.
        •	Gérer toutes les conversions de type et contraintes.

3.	Transfert staging vers finales (load_finales.sql)
        •	Écrire les scripts sql pour charger chaque table finale à partir de sa table de staging (conversion de types, gestion des clés).

3) Requêtes guidées (Étape 2 – x points)

Vous êtes en deux équipes (A et B). Chaque équipe se voit confier cinq énoncés de requêtes à traduire en SQL, exécuter, valider et commenter. Il s’agit de complexités croissantes :
Équipe A
    1.	Q1-A : Nombre de commandes par état du client
        Pour chaque état (customer_state), comptez le nombre total de commandes.
    2.	Q2-A : Top 10 des commandes par montant facturé
        Calculez pour chaque order_id la somme price + freight_value, puis affichez les 10 commandes les plus chères.
    3.	Q3-A : Top 10 des vendeurs par chiffre d’affaires
        Pour chaque vendeur (seller_id,seller_city), comptez le nombre de commandes et le chiffre d’affaires total (price + freight_value), puis affichez         les 10 meilleurs vendeurs par chiffres d’affaires(CA).
    4.	Q4-A : Revenu cumulé mensuel par état du client
        Pour chaque mois et chaque état, calculez le revenu mensuel, puis ajoutez une colonne « running_revenue » qui cumule ce revenu mois après mois.
    5.	Q5-A : Taux de croissance mensuel du CA par catégorie de produit
        Pour chaque mois et chaque catégorie de produit (product_category_name), calculez le chiffre d’affaires, puis le pourcentage d’évolution par         rapport au mois précédent (fonction LAG()).

Objectif Étape 3 : Cette requête doit montrer des lenteurs. Vous l’optimiserez ensuite.

Équipe B
    1.	Q1-B : Nombre de clients uniques par état
        Pour chaque état (customer_state), comptez le nombre de customer_unique_id distincts.
    2.	Q2-B : Nombre moyen de lignes de commande par commande
        Calculez pour chaque commande son nombre de lignes (order_items), puis déterminez la moyenne de ces compteurs.
    3.	Q3 -B: Top 5 des catégories produit par nombre de commandes
        Pour chaque catégorie de produit (product_category_name), comptez le nombre de commandes distinctes, puis affichez les 5 premières catégories.
    4.	Q4-B : Répartition du chiffre d’affaires(CA) par vendeur
        Calculez d’abord le chiffre d’affaires de chaque vendeur, puis :
            •	La part (%) de chaque vendeur sur le CA total,
            •	et le pourcentage cumulé de CA.
    5.	Q5-B : Médiane du délai de livraison par état
        Pour chaque customer_state, calculez la différence en heures entre order_delivered_customer_date et order_delivered_carrier_date, puis calculez la         médiane de ces durées (PERCENTILE_CONT(0.5)).

Objectif Étape 3 : Cette requête doit montrer des lenteurs. Vous l’optimiserez ensuite.

4) Optimisation de la requête lente (Étape 3 – x points)
   
Pour la question 5(Q5) de votre équipe :
    1.	Exécutez votre Q5 brute sous :
        EXPLAIN (ANALYZE, BUFFERS)  <votre requête Q5 brute>;
    2.	Analysez le plan (nœuds coûteux : Seq Scan, Sort sur disque, WindowAgg…).
    3.	Proposez et implémentez au moins un correctif :
        •	Création d’index (Ex. CREATE INDEX … ON tp1_ind500_orders(order_delivered_carrier_date), ou sur (product_category_name,           order_purchase_timestamp)),
        •	Réécriture de la requête (éliminer CTE redondants, remplacer WINDOW par JOINs, etc.),
        •	Partitionnement.
    4.	Ré-exécutez EXPLAIN (ANALYZE, BUFFERS) et comparez les performances avant/après (temps, buffers, lectures disques).

Livrables finaux (à rendre pour chaque équipe)

1. README
   •	Organisation du dépôt (arborescence)
   •	Ordre d’exécution des scripts
   •	Bref mode d’emploi pour reproduire le chargement, le nettoyage, l’exécution des requêtes et l’optimisation
2. Schéma visuel
   •	Export PNG ou PDF du modèle relationnel final (tables finales + clés)
3. Script de création de la base
   •	Fichier create_database.sql ou dump SQL complet (.sql ou backup) permettant de recréer la base tp1_ind500 (schemas, extensions, etc.).
4. Scripts SQL
   •	DDL staging & tables finales
   •	Import des CSV bruts & nettoyage (uniformisation des dates, standardisation des chaînes, suppression des doublons)
   •	Transfert staging vers tables finales
   •	Requêtes Q1–Q5 - Un seul fichier queries.sql listant les 5 énoncés (Q1–Q5) avec pour chaque requête un espace réservé pour vos commentaires /       captures de résultats
   •	Correctif d’optimisation pour Q5 - Script de création d’index minimal (obligatoire) et script de partitionnement (pour 15 points bonus)
5. Rapport d’optimisation (pour Q5)
   •	Capture du plan EXPLAIN (ANALYZE, BUFFERS) avant correctif
   •	Identification et commentaire des nœuds coûteux (seq scan, sort sur disque, WindowAgg…)
   •	Description du correctif retenu (index ou réécriture) et script correspondant
   •	Capture du plan après correctif
   •	Comparaison chiffrée des temps (avant vs après) et courts commentaires sur les gains

NB
   •	La création d’au moins un index pertinent pour Q5 est obligatoire.
   •	Le partitionnement donnes des points bonus

Bonne chance à toutes et à tous !
