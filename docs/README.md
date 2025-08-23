# IND500 - Bases de données distribuées modernes

[![Documentation](https://img.shields.io/badge/docs-IND500-blue)](https://uquebec-ets.github.io/IND500/cours/plan_de_cours)
[![Site officiel ÉTS](https://img.shields.io/badge/ÉTS-site--officiel-red)](https://www.etsmtl.ca/etudes/cours/ind500)
[![pages-build-deployment](https://github.com/uquebec-ets/IND500/actions/workflows/pages/pages-build-deployment/badge.svg)](https://github.com/uquebec-ets/IND500/deployments/activity_log?environment=github-pages)

[![TP1](https://img.shields.io/badge/TP1-Introduction-green)](tp/tp1.md)  
[![TP2](https://img.shields.io/badge/TP2-NoSQL-orange)](tp/tp2.md)  
[![TP3](https://img.shields.io/badge/TP3-ETL-blue)](tp/tp3.md)  

## Description du cours
Le cours vise à expliquer les **technologies**, la **modélisation** et l’**utilisation** des bases de données NoSQL.  
Il couvre les principes des systèmes distribués et les compromis nécessaires entre **performance**, **fiabilité** et **scalabilité**.

## Objectifs d’apprentissage
À la fin du cours, vous serez en mesure de :
- Déterminer quand utiliser un type particulier de base de données.
- Identifier les exigences et les défis des bases de données en matière de performance et de fiabilité.
- Mettre en pratique les techniques d’**extraction, transformation et chargement (ETL)** pour de grandes quantités de données.
- Comprendre les enjeux de **sécurité** et de **performance** propres aux bases distribuées.

## Contenu
- Survol : bases relationnelles vs bases NoSQL.  
- Modèles NoSQL : 
  - Documents  
  - Clé-valeur  
  - Colonnes  
  - Séries chronologiques  
  - Graphes  
- Introduction à l’ETL à grande échelle.  
- Sélecteurs de requêtes, pipelines d’agrégation, requêtes hétérogènes.  
- Modélisation flexible et schémas dynamiques.  

## Liens utiles
- 📖 [Plan de cours](https://uquebec-ets.github.io/IND500/cours/plan_de_cours)  
- 🌐 [Site officiel ÉTS](https://www.etsmtl.ca/etudes/cours/ind500)  

## Développement
Le site est construit avec [Jekyll](https://jekyllrb.com/) et utilise le thème [Just-the-Docs](https://just-the-docs.github.io/just-the-docs/).

### Installation locale
```bash
# Installer les dépendances Ruby
bundle install

# Lancer le serveur local
bundle exec jekyll serve
````
