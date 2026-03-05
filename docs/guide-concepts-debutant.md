# 📖 Guide des concepts Fabric — Pour débutants

Ce guide explique **en langage simple** tous les concepts utilisés dans la démo. Il est conçu pour quelqu'un qui vient d'un univers data métier et qui n'a pas encore de bagage technique sur Azure et Fabric.

---

## Table des matières

1. [Les bases : Cloud et Azure](#1-les-bases--cloud-et-azure)
2. [Microsoft Fabric : la vue d'ensemble](#2-microsoft-fabric--la-vue-densemble)
3. [Le Tenant et Entra ID](#3-le-tenant-et-entra-id)
4. [Le Workspace](#4-le-workspace)
5. [La Capacité Fabric](#5-la-capacité-fabric)
6. [Le Lakehouse](#6-le-lakehouse)
7. [OneLake](#7-onelake)
8. [Les Tables Delta](#8-les-tables-delta)
9. [Le SQL Analytics Endpoint](#9-le-sql-analytics-endpoint)
10. [Les Notebooks](#10-les-notebooks)
11. [Spark : le moteur de calcul](#11-spark--le-moteur-de-calcul)
12. [Les Data Pipelines](#12-les-data-pipelines)
13. [Les APIs REST](#13-les-apis-rest)
14. [Le Modèle Sémantique](#14-le-modèle-sémantique)
15. [Semantic Link](#15-semantic-link)
16. [La Sécurité (RLS)](#16-la-sécurité-rls)
17. [Les Shortcuts](#17-les-shortcuts)
18. [Glossaire rapide](#18-glossaire-rapide)

---

## 1. Les bases : Cloud et Azure

### C'est quoi le "cloud" ?
Le cloud, ce sont des **ordinateurs très puissants** installés dans des centres de données Microsoft, partout dans le monde. Au lieu d'acheter et maintenir vos propres serveurs (on-premise), vous **louez** de la puissance de calcul et du stockage à Microsoft.

### C'est quoi Azure ?
**Azure** est la plateforme cloud de Microsoft. C'est un peu comme un supermarché de services informatiques : vous y trouvez du stockage, des bases de données, de l'intelligence artificielle, et bien sûr **Microsoft Fabric**.

### C'est quoi une "ressource Azure" ?
Une ressource, c'est **quelque chose que vous avez créé** dans Azure. Par exemple :
- Une capacité Fabric → c'est une ressource
- Une base de données → c'est une ressource
- Un compte de stockage → c'est une ressource

Chaque ressource a un coût, un emplacement (région), et des paramètres.

---

## 2. Microsoft Fabric : la vue d'ensemble

### En une phrase
Microsoft Fabric est une **plateforme de données tout-en-un** qui regroupe le stockage, le traitement, l'analyse et la visualisation des données.

### Pourquoi c'est différent ?
Avant Fabric, il fallait assembler plein de services séparés :
- Azure Data Lake pour le stockage
- Azure Databricks pour le traitement Spark
- Azure Data Factory pour l'orchestration
- Power BI pour les rapports
- ...chacun avec sa propre facturation, sa propre sécurité, sa propre interface

**Fabric regroupe tout** dans une seule interface, avec une seule sécurité et une seule facturation.

### Comparaison Cloudera → Fabric

| Aspect | Cloudera (ancien monde) | Fabric (nouveau monde) |
|--------|------------------------|----------------------|
| Stockage | HDFS (vos serveurs) | OneLake (cloud Microsoft) |
| Traitement | Spark sur vos machines | Spark managé |
| SQL | Hive / Impala | SQL analytics endpoint |
| Orchestration | Oozie / Airflow | Data Pipelines |
| Rapports | Outils externes | Power BI intégré |
| Administration | Ambari / Cloudera Manager | Portail Fabric |
| Infrastructure | Vous gérez les serveurs | Microsoft gère tout |

---

## 3. Le Tenant et Entra ID

### C'est quoi un "tenant" ?
Un **tenant** (locataire en français), c'est l'**espace de votre organisation** dans le cloud Microsoft. Quand votre entreprise souscrit à Microsoft 365 ou Azure, un tenant est créé. C'est comme le **nom de domaine de votre entreprise** dans le cloud.

Exemple : `MngEnvMCAP183094.onmicrosoft.com` est un tenant.

### C'est quoi Entra ID ?
**Entra ID** (anciennement Azure Active Directory / Azure AD) est le **système de gestion des identités**. C'est là que sont stockés :
- Les **utilisateurs** (qui a un compte)
- Les **groupes** (quelles équipes)
- Les **licences** (qui a droit à quoi)
- Les **permissions** (qui peut faire quoi)

### Analogie
| Monde réel | Cloud Microsoft |
|------------|----------------|
| Le bâtiment de votre entreprise | Le tenant |
| Le service des badges | Entra ID |
| Le badge d'un employé | Le compte utilisateur |
| L'étage autorisé | Les permissions |

---

## 4. Le Workspace

### C'est quoi ?
Un **workspace** (espace de travail) est un **conteneur** dans Fabric où vous rangez tous vos éléments de travail : données, notebooks, pipelines, rapports.

### Analogie
Pensez à un **dossier de projet partagé** :
- Le workspace = le dossier
- Les items (Lakehouse, Notebook...) = les fichiers dans le dossier
- Les rôles = les permissions sur le dossier

### Les rôles

| Rôle | En gros... |
|------|-----------|
| **Viewer** | Peut regarder, ne peut rien toucher |
| **Contributor** | Peut modifier le contenu existant |
| **Member** | Peut modifier et partager |
| **Admin** | Peut tout faire, y compris supprimer |

---

## 5. La Capacité Fabric

### C'est quoi ?
La **capacité** c'est le **moteur** qui fait tourner vos traitements. C'est un peu comme louer une voiture :
- Petite capacité (F2) = petite voiture (pour les tests)
- Grande capacité (F64) = gros 4x4 (pour la production)

### Comment ça marche ?
- Vous créez une capacité dans Azure (c'est une **ressource Azure**)
- Vous l'associez à un ou plusieurs workspaces
- Tous les utilisateurs du workspace bénéficient de cette puissance
- Vous payez en fonction de la taille choisie

### Important
- La capacité est attachée **au workspace**, pas aux utilisateurs
- Si vous mettez la capacité en pause → les workspaces ne fonctionnent plus
- Vous pouvez changer la taille à tout moment

---

## 6. Le Lakehouse

### C'est quoi ?
Le **Lakehouse** est l'endroit où vous **stockez vos données**. C'est la fusion de deux concepts :
- **Data Lake** (lac de données) = stockage brut de fichiers
- **Data Warehouse** (entrepôt de données) = données structurées en tables

### Les deux zones

```
Mon Lakehouse
├── Tables/     ← Données structurées, requêtables en SQL
│   ├── clients
│   └── commandes
└── Files/      ← Fichiers bruts (CSV, JSON, images...)
    ├── imports/
    └── exports/
```

### Analogie
| Zone | C'est comme... |
|------|---------------|
| **Tables/** | Un classeur Excel avec des tableaux bien rangés |
| **Files/** | Un disque dur avec des fichiers en vrac |

---

## 7. OneLake

### C'est quoi ?
**OneLake** est le **système de stockage** qui se cache derrière tous les Lakehouses. C'est un data lake unique pour tout votre tenant (toute votre organisation).

### Pourquoi c'est important ?
- **Un seul endroit** pour toutes les données de l'organisation
- Accessible par **tous les workspaces** (avec les bonnes permissions)
- Pas besoin de copier les données d'un endroit à l'autre (on utilise des **shortcuts**)

### Analogie
Si les Lakehouses sont des **tiroirs**, OneLake est la **grande armoire** qui contient tous les tiroirs.

---

## 8. Les Tables Delta

### C'est quoi Delta ?
**Delta Lake** est un **format de stockage** de données open-source créé par Databricks. Dans Fabric, toutes les tables sont au format Delta.

### Pourquoi c'est mieux qu'un fichier CSV ?

| CSV classique | Table Delta |
|---------------|------------|
| Pas de schéma (colonnes libres) | Schéma défini (colonnes typées) |
| Pas de transactions | Transactions ACID (pas de données corrompues) |
| Pas de versioning | Historique des modifications (time travel) |
| Lecture séquentielle | Lecture optimisée (predicate pushdown) |
| Pas de mise à jour partielle | UPDATE, DELETE, MERGE possibles |

### En pratique
Vous n'avez pas besoin de "penser Delta" au quotidien. Quand vous faites `df.write.saveAsTable("ma_table")`, la table est automatiquement créée en Delta.

---

## 9. Le SQL Analytics Endpoint

### C'est quoi ?
C'est une **couche SQL** automatiquement créée au-dessus de votre Lakehouse. Elle permet de **requêter vos tables avec du SQL** sans démarrer Spark.

### Pourquoi c'est utile ?
- **Instantané** : pas besoin d'attendre le démarrage de Spark (30s-1min)
- **SQL standard** : `SELECT * FROM ma_table WHERE ville = 'Paris'`
- **Parfait pour** : les analyses rapides, les rapports Power BI, les vérifications

### Limitation
C'est en **lecture seule**. Vous ne pouvez pas modifier les données via le SQL endpoint. Pour écrire, il faut passer par Spark (notebooks).

---

## 10. Les Notebooks

### C'est quoi ?
Un **notebook** est un document interactif qui mélange :
- Du **texte explicatif** (en Markdown)
- Du **code exécutable** (Python, SQL, Scala, R)
- Des **résultats** (tableaux, graphiques)

### Analogie
C'est comme un **document Word** dans lequel vous pouvez aussi **exécuter du code** et voir les résultats immédiatement.

### Les langages disponibles

| Langage | Quand l'utiliser |
|---------|-----------------|
| **PySpark (Python)** | Traitement de données, ETL, ML — le plus courant |
| **Spark SQL** | Requêtes SQL sur les tables |
| **Scala** | Performance critique (rare) |
| **R** | Statistiques avancées (rare) |

### Comment ça se passe ?
1. Vous écrivez du code dans une **cellule**
2. Vous cliquez sur ▶ (Play)
3. Le code s'exécute dans le cloud (pas sur votre PC !)
4. Le résultat s'affiche sous la cellule

---

## 11. Spark : le moteur de calcul

### C'est quoi Spark ?
**Apache Spark** est un **moteur de calcul distribué**. En gros, il découpe votre travail en petits morceaux et les exécute en parallèle sur plusieurs machines.

### Pourquoi c'est puissant ?
- **Un fichier de 10 Go** : Excel plante. Spark le traite en quelques secondes.
- **Une requête sur 1 milliard de lignes** : SQL classique rame. Spark distribue le calcul.

### Bonne nouvelle pour Quadient
Vous utilisez déjà Spark sur Cloudera ! Le code Spark est **le même** dans Fabric. La seule différence : vous n'avez plus à gérer les machines.

### Le `spark` dans le code
Quand vous voyez `spark.sql(...)` ou `spark.read.csv(...)` dans un notebook, `spark` est l'objet qui représente votre connexion au moteur Spark. Il est pré-configuré automatiquement dans les notebooks Fabric.

---

## 12. Les Data Pipelines

### C'est quoi ?
Un **Data Pipeline** est un enchaînement d'étapes automatisées pour traiter vos données. C'est l'équivalent d'Oozie ou Airflow.

### Exemple concret
```
Matin 6h00 (automatique) :
  Étape 1 : Copier les nouveaux fichiers depuis le FTP du client
  Étape 2 : Lancer le notebook de nettoyage des données
  Étape 3 : Mettre à jour les tables du Lakehouse
  Étape 4 : Rafraîchir le rapport Power BI
  Étape 5 : Envoyer un email de confirmation
```

### Comment ça se crée ?
C'est du **drag & drop** ! Vous glissez des blocs (activités) et vous les reliez avec des flèches. Pas besoin d'écrire du XML (comme Oozie) ou du Python (comme Airflow).

---

## 13. Les APIs REST

### C'est quoi une API ?
Une **API** (Application Programming Interface) c'est une **interface pour les machines**. Au lieu de cliquer dans une interface graphique, un programme envoie des commandes.

### C'est quoi REST ?
**REST** est un standard pour les APIs web. Ça utilise les mêmes protocoles que votre navigateur (HTTP).

### Analogie
| Action humaine | API REST |
|----------------|---------|
| Aller sur un site web | `GET https://api.fabric.microsoft.com/workspaces` |
| Remplir un formulaire et cliquer "Envoyer" | `POST https://api.fabric.microsoft.com/jobs` |
| Regarder le statut d'une commande | `GET https://api.fabric.microsoft.com/jobs/123/status` |

### Pourquoi c'est utile ?
- Automatiser des tâches répétitives
- Intégrer Fabric avec d'autres systèmes (Jenkins, Airflow, systèmes métier)
- Déclencher des traitements depuis un script

### Le Token d'authentification
Pour utiliser une API, il faut un **token** (jeton d'authentification). C'est un long texte crypté qui prouve votre identité. C'est comme scanner votre badge avant d'entrer dans un bâtiment sécurisé.

---

## 14. Le Modèle Sémantique

### C'est quoi ?
Un **modèle sémantique** (semantic model) est une **couche métier** au-dessus de vos données. Il transforme vos tables brutes en quelque chose de compréhensible pour le métier.

### Ce qu'il contient
- **Tables** : les données source
- **Relations** : comment les tables sont liées (ex: Commandes → Clients)
- **Mesures** : les calculs métier (ex: CA = SUM(montant), Marge = CA - Coûts)
- **Hiérarchies** : les niveaux d'analyse (ex: Année → Trimestre → Mois)

### Analogie
| Données brutes | Modèle sémantique |
|----------------|-------------------|
| Tas de LEGO en vrac | Notice + modèle assemblé |
| Chiffres incompréhensibles | KPIs métier clairs |
| `SUM(col_ht * (1 + col_tva_pct/100))` | "CA TTC" |

### En pratique
Quand vous créez un Lakehouse, un **modèle sémantique par défaut** est automatiquement créé. Il contient les tables de votre Lakehouse. Vous pouvez ensuite y ajouter des mesures et des relations.

---

## 15. Semantic Link

### C'est quoi ?
**Semantic Link** est une librairie Python qui fait le pont entre les **notebooks** (monde data engineering) et les **modèles sémantiques** (monde Power BI).

### En 3 mots
Lire les données Power BI depuis Python.

### Les 3 fonctions à retenir

```python
import sempy.fabric as fabric

# 1. Voir les modèles disponibles
fabric.list_datasets()

# 2. Charger une table dans un DataFrame
df = fabric.read_table("mon_modele", "ma_table")

# 3. Exécuter une requête DAX
result = fabric.evaluate_dax("mon_modele", "EVALUATE ma_requete")
```

---

## 16. La Sécurité (RLS)

### C'est quoi la RLS ?
**Row-Level Security** (sécurité au niveau de la ligne) = chaque utilisateur ne voit que **les lignes qui le concernent**.

### Exemple
La table `ventes` contient des données de 3 filiales :

| Sans RLS | Avec RLS (connecté en tant que "France") |
|----------|------------------------------------------|
| Voit les 3 filiales | Ne voit que les lignes "France" |

### Limitation actuelle dans Fabric
- ✅ La RLS fonctionne via le **SQL analytics endpoint**
- ✅ La RLS fonctionne via le **modèle sémantique** (Power BI)
- ❌ La RLS **ne fonctionne PAS** via Spark / OneLake shortcuts (erreur 403)

---

## 17. Les Shortcuts

### C'est quoi ?
Un **shortcut** (raccourci) est un **lien** vers des données stockées ailleurs, **sans les copier**.

### Analogie
C'est comme un **raccourci Windows** : quand vous double-cliquez dessus, ça ouvre le fichier original. Le fichier n'est pas dupliqué.

### Pourquoi c'est utile pour Quadient ?
Avec des filiales multiples, on veut :
- Stocker les données **une seule fois** (dans un Lakehouse source)
- Donner accès à chaque filiale **sans copier** les données
- Les shortcuts permettent ça

---

## 18. Glossaire rapide

| Terme | Définition simple |
|-------|-------------------|
| **Azure** | Le cloud de Microsoft |
| **Tenant** | L'espace de votre organisation dans le cloud |
| **Entra ID** | Le système de gestion des utilisateurs et permissions |
| **Fabric** | La plateforme de données tout-en-un de Microsoft |
| **Workspace** | Un dossier de projet partagé dans Fabric |
| **Capacité** | Le moteur (puissance de calcul) |
| **Lakehouse** | Stockage de données (tables + fichiers) |
| **OneLake** | Le data lake unique derrière tous les Lakehouses |
| **Delta** | Le format de stockage des tables (open-source) |
| **SQL Endpoint** | Couche SQL en lecture sur le Lakehouse |
| **Notebook** | Document interactif code + texte |
| **Spark** | Moteur de calcul distribué (même que sur Cloudera) |
| **Pipeline** | Orchestration automatisée de tâches |
| **API REST** | Interface pour contrôler Fabric par code |
| **Modèle sémantique** | Couche métier (tables + mesures + relations) |
| **Semantic Link** | Librairie Python pour lire les données Power BI |
| **RLS** | Sécurité au niveau de la ligne (qui voit quoi) |
| **Shortcut** | Lien vers des données sans copie |
| **DAX** | Langage de requête Power BI |
| **PySpark** | Python + Spark (le langage principal des notebooks) |
| **Token** | Jeton d'authentification pour les APIs |
| **Service Principal** | Un "robot" avec un compte pour automatiser |

---

*Ce guide fait partie du projet [fabric-demo-quadient](../README.md).*
