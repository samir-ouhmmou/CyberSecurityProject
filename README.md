# 🛡️ Détection des Anomalies et des Comportements Suspects dans les Logs d'Authentification

\<p align="center">   \<img src="[https://img.shields.io/badge/Python-3.10%2B-blue?logo=python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)" />   \<img src="[https://img.shields.io/badge/Machine%20Learning-Anomaly%20Detection-orange](https://img.shields.io/badge/Machine%20Learning-Anomaly%20Detection-orange)" />   \<img src="[https://img.shields.io/badge/Dataset-LANL-red](https://img.shields.io/badge/Dataset-LANL-red)" />   \<img src="[https://img.shields.io/badge/Status-In%20Progress-yellow](https://img.shields.io/badge/Status-In%20Progress-yellow)" /> \</p>

## 📌 1. Présentation du projet

Ce projet a pour objectif de détecter automatiquement les comportements suspects et les activités potentiellement malveillantes à partir des logs d'authentification du dataset **LANL Comprehensive Cyber-Security Events**.

L'approche repose sur plusieurs algorithmes de Machine Learning et d'analyse séquentielle afin d'identifier les anomalies dans les comportements des utilisateurs et les connexions entre machines.

### 🎯 Objectifs

- Identifier les comportements inhabituels des utilisateurs.
- Détecter les connexions suspectes entre différentes machines.
- Identifier les activités correspondant à des attaques (*Red Team*).
- Comparer plusieurs approches de détection d'anomalies.
- Évaluer les performances des modèles avec des métriques adaptées.

## 🧰 2. Technologies et bibliothèques

| Technologie          | Utilisation                                          |
| -------------------- | ---------------------------------------------------- |
| Python               | Langage principal                                    |
| Jupyter Notebook     | Analyse et expérimentation                           |
| Pandas & NumPy       | Manipulation des données                             |
| Scikit-learn         | Algorithmes de Machine Learning                      |
| Matplotlib & Seaborn | Visualisation des données                            |
| NetworkX             | Analyse des relations entre utilisateurs et machines |

### Installation

```bash
pip install pandas numpy scikit-learn matplotlib seaborn networkx
```

## 📂 3. Dataset utilisé

**LANL Comprehensive Cyber-Security Events**

Source officielle : [https://csr.lanl.gov/data/cyber1/](https://csr.lanl.gov/data/cyber1/)

Fichiers nécessaires :

- `auth.txt.gz` : événements d'authentification entre utilisateurs et machines.
- `redteam.txt.gz` : événements associés aux activités offensives simulées (*Red Team*).

> ⚠️ Le fichier `auth.txt.gz` est volumineux. Il est recommandé de travailler initialement sur un échantillon de données afin de limiter la consommation de mémoire.

## 🚀 4. Les étapes de réalisation du projet

### Étape 1 — 📥 Télécharger les datasets

- Accéder au site officiel du LANL.
- Télécharger `auth.txt.gz` et `redteam.txt.gz`.
- Conserver les fichiers compressés pour économiser l'espace disque.

### Étape 2 — ⚙️ Préparer l'environnement

- Installer Python.
- Installer Jupyter Notebook.
- Installer les bibliothèques nécessaires.

### Étape 3 — 📊 Charger un échantillon des données

Charger les premières lignes du dataset pour commencer l'expérimentation.

```python
import pandas as pd

auth = pd.read_csv(
    "auth.txt.gz",
    sep=",",
    header=None,
    nrows=5_000_000
)
```

L'objectif est de sélectionner une période pertinente contenant des événements Red Team, en vérifiant les dates et les timestamps avant l'analyse.

### Étape 4 — 🔍 Analyse exploratoire (EDA)

Explorer les caractéristiques principales des données :

- Nombre d'événements par heure.
- Taux de connexions réussies et échouées.
- Utilisateurs les plus actifs.
- Machines les plus sollicitées.
- Types d'authentification.
- Répartition temporelle des événements.

Créer des graphiques pour mieux comprendre les comportements habituels.

### Étape 5 — 🧹 Prétraitement des données

- Convertir les timestamps en heures et en jours.
- Supprimer les comptes machines (comptes se terminant par `$`), selon le périmètre retenu.
- Identifier les événements Red Team.
- Attribuer un label :
  - `0` : comportement normal.
  - `1` : activité associée à une attaque.

### Étape 6 — 🧪 Extraction des caractéristiques (Feature Engineering)

Regrouper les événements par utilisateur et par fenêtre temporelle (une heure).

Calculer les caractéristiques suivantes :

| Feature           | Description                              |
| ----------------- | ---------------------------------------- |
| `login_count`     | Nombre total de connexions               |
| `unique_machines` | Nombre de machines différentes           |
| `failed_count`    | Nombre de connexions échouées            |
| `failure_ratio`   | Ratio des connexions échouées            |
| `auth_types`      | Nombre de types d'authentification       |
| `off_hours_ratio` | Part des connexions hors heures normales |
| `new_machines`    | Nombre de nouvelles machines utilisées   |

Ces caractéristiques serviront d'entrée aux modèles de détection.

### Étape 7 — 🤖 Modèle 1 : Isolation Forest

Utiliser Isolation Forest pour identifier les comportements qui s'écartent des habitudes normales.

Étapes :

1. Normaliser les caractéristiques avec `StandardScaler`.
2. Entraîner le modèle `IsolationForest`.
3. Calculer les scores d'anomalie.
4. Identifier les utilisateurs ou fenêtres temporelles suspectes.

### Étape 8 — 🔗 Modèle 2 : Chaîne de Markov

Analyser les séquences de connexions des utilisateurs.

Exemple :

```text
User A : Machine 1 → Machine 2 → Machine 3
```

Étapes :

1. Construire les séquences de machines pour chaque utilisateur.

2. Compter les transitions entre machines.

3. Calculer les probabilités de transition :

   `P(j | i)`

4. Calculer un score d'anomalie basé sur la probabilité des transitions :

   `Score = -Σ log(P(j | i))`

Une transition inhabituelle ou peu probable peut indiquer un comportement suspect.

### Étape 9 — 🎯 Modèle 3 : K-Means (optionnel)

Appliquer K-Means pour regrouper les comportements similaires.

- Identifier les clusters de comportements.
- Calculer la distance entre chaque observation et le centre de son cluster.
- Considérer les observations très éloignées comme potentiellement suspectes.

### Étape 10 — 📈 Évaluation des modèles

Comparer les anomalies détectées avec les événements identifiés dans `redteam.txt`.

Métriques utilisées :

- **Precision** : proportion des alertes réellement positives.
- **Recall** : proportion des attaques détectées.
- **F1-score** : équilibre entre précision et rappel.
- **PR-AUC** : performance globale sur la courbe précision-rappel.

Présenter les résultats dans un tableau comparatif :

| Modèle           | Precision  | Recall     | F1-score   | PR-AUC     |
| ---------------- | ---------- | ---------- | ---------- | ---------- |
| Isolation Forest | À calculer | À calculer | À calculer | À calculer |
| Markov Chain     | À calculer | À calculer | À calculer | À calculer |
| K-Means          | À calculer | À calculer | À calculer | À calculer |

### Étape 11 — 📉 Visualisation des résultats

Créer des visualisations pour faciliter l'interprétation :

- Top 20 des utilisateurs les plus suspects.
- Timeline des anomalies détectées.
- Graphe des relations utilisateur → machines avec NetworkX.
- Comparaison des performances des modèles.

### Étape 12 — 📝 Analyse et conclusion

Interpréter les résultats obtenus :

- Quel modèle détecte le mieux les activités suspectes ?
- Quels sont les avantages et les limites de chaque approche ?
- Quel est l'impact de la taille de l'échantillon et de la fenêtre temporelle ?
- Quels sont les risques de faux positifs ?

Perspectives d'amélioration :

- Combiner les scores de plusieurs modèles.
- Tester des modèles séquentiels comme LSTM.
- Expérimenter avec différentes fenêtres temporelles.
- Améliorer la sélection des caractéristiques.

### Étape 13 — 📚 Rapport et présentation

Préparer les livrables finaux :

- Rapport scientifique rédigé en LaTeX.
- Schémas représentant l'architecture et le fonctionnement des modèles.
- Graphiques et tableaux des résultats expérimentaux.
- Présentation PowerPoint pour la soutenance.

## 🏗️ 5. Architecture générale du projet

```text
              LANL DATASET
                   |
          +--------+--------+
          |                 |
       auth.txt.gz      redteam.txt.gz
          |                 |
          v                 |
     Data Preprocessing     |
          |                 |
          v                 |
     Feature Engineering    |
          |                 |
          v                 |
     Anomaly Detection      |
          |                 |
     +----+------+----------+
     |           |          |
     v           v          v
 Isolation     Markov     K-Means
  Forest       Chain     (Optional)
     |           |          |
     +-----------+----------+
                 |
                 v
          Model Evaluation
                 |
                 v
          Visualization
                 |
                 v
          Final Results
```

## 📁 6. Structure prévisionnelle du projet

```text
Cybersecurity-Anomaly-Detection/
│
├── dataset/
│   ├── auth.txt.gz
│   └── redteam.txt.gz
│
├── notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_exploratory_analysis.ipynb
│   ├── 03_preprocessing.ipynb
│   ├── 04_feature_engineering.ipynb
│   ├── 05_isolation_forest.ipynb
│   ├── 06_markov_chain.ipynb
│   ├── 07_kmeans.ipynb
│   └── 08_model_evaluation.ipynb
│
├── results/
│   ├── figures/
│   └── metrics/
│
├── reports/
│   └── final_report.tex
│
├── requirements.txt
└── README.md
```

## ⚠️ 7. Points importants à considérer

- Éviter de charger l'intégralité du dataset en mémoire dès le début.
- Vérifier la correspondance temporelle entre les événements d'authentification et les événements Red Team.
- Éviter les fuites de données (*Data Leakage*) pendant l'entraînement et l'évaluation.
- Tenir compte du déséquilibre entre les événements normaux et malveillants.
- Choisir les seuils de détection en fonction des résultats expérimentaux.

## 👨‍💻 8. Résultat attendu

À la fin du projet, nous souhaitons obtenir un système capable de :

- Analyser les logs d'authentification.
- Apprendre les comportements habituels des utilisateurs.
- Détecter les comportements anormaux.
- Identifier les activités potentiellement malveillantes.
- Comparer plusieurs modèles de détection et présenter leurs performances.

---

\<p align="center">
&#x20; \<b>🛡️ Cybersecurity | Machine Learning | Anomaly Detection\</b>
&#x20; \<br/>
&#x20; \<i>Detecting suspicious behavior through intelligent analysis of authentication logs.\</i>
\</p>
