# 🏭 Fabric Demo — Migration Cloudera → Microsoft Fabric

## Contexte

Ce projet contient une **démo complète** de Microsoft Fabric à destination du client **Quadient**, dans le cadre d'une migration depuis **Cloudera** (Hadoop/Spark on-prem) vers Microsoft Fabric (plateforme data cloud unifiée).

## Objectifs de la démo

1. **Présenter le Workspace Fabric** — comprendre l'environnement de travail et ses composants
2. **Découvrir le Lakehouse & OneLake** — le stockage unifié qui remplace HDFS + Hive
3. **Déclencher des workflows via APIs** — automatiser les traitements (remplacement d'Oozie/Airflow)
4. **Semantic Link** — interroger les modèles sémantiques depuis un notebook Python

## Structure du projet

```
fabric-demo-quadient/
├── README.md                          ← Ce fichier
├── .gitignore
├── docs/
│   └── guide-concepts-debutant.md     ← Guide des concepts Fabric pour débutants
└── notebooks/
    ├── 01-decouverte-workspace.ipynb   ← Notebook 1 : Découverte du workspace Fabric
    ├── 02-lakehouse-onelake.ipynb      ← Notebook 2 : Lakehouse, tables Delta, OneLake
    ├── 03-pipelines-apis.ipynb         ← Notebook 3 : Data Pipelines & APIs REST Fabric
    └── 04-semantic-link.ipynb          ← Notebook 4 : Semantic Link (bonus)
```

## Correspondance Cloudera → Fabric

| Composant Cloudera | Équivalent Fabric | Notebook |
|--------------------|-------------------|----------|
| Cluster Hadoop | Workspace Fabric | 01 |
| HDFS | OneLake | 02 |
| Hive Metastore | Lakehouse (Tables Delta) | 02 |
| Spark Submit / Zeppelin | Notebook Fabric (PySpark) | 01, 02 |
| Oozie / Airflow | Data Pipeline | 03 |
| API Oozie | API REST Fabric | 03 |
| Ranger / Sentry | RLS + Workspace Roles | 02 |
| N/A | Semantic Link | 04 |

## Pré-requis

- Un **tenant Microsoft Fabric** avec une capacité active (F2 minimum pour les tests)
- Un compte utilisateur avec le rôle **Member** ou **Admin** sur un workspace
- Un navigateur web (Edge ou Chrome recommandé)
- Pour le notebook 04 : la librairie `semantic-link` (pré-installée dans Fabric)

## Comment utiliser cette démo

1. **Lire le guide des concepts** : commencez par [docs/guide-concepts-debutant.md](docs/guide-concepts-debutant.md) si vous êtes nouveau sur Fabric
2. **Importer les notebooks dans Fabric** : dans votre workspace, cliquez sur `+ Nouveau` → `Importer un notebook` → sélectionnez les fichiers `.ipynb`
3. **Exécuter les notebooks dans l'ordre** : 01 → 02 → 03 → 04
4. **Chaque notebook contient des explications** en markdown + du code exécutable

## Déroulé recommandé pour la démo (30 min)

| Temps | Sujet | Support |
|-------|-------|---------|
| 0-2 min | Introduction, contexte migration | Oral |
| 2-10 min | Workspace Fabric (parallèle Cloudera) | Notebook 01 + live |
| 10-17 min | Lakehouse & OneLake en détail | Notebook 02 + live |
| 17-25 min | Data Pipelines & APIs REST | Notebook 03 + live |
| 25-28 min | Semantic Link (bonus) | Notebook 04 |
| 28-30 min | Q&A | — |

## Auteur

Demo préparée par l'équipe Microsoft — Solution Engineering
