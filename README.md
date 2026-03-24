# IA_EMBARQUEE
GITHUB du projet IA embarquée

# 1. Introduction et Contexte du Projet
#    1.1 Objectif Général
   
Ce projet a pour ambition de concevoir, optimiser et déployer un Réseau de Neurones Profonds (DNN) directement sur un microcontrôleur STM32L4R9. L'objectif final est de réaliser une tâche de maintenance prédictive en temps réel, capable d'analyser les données de capteurs industriels pour anticiper les défaillances machines avec une empreinte mémoire extrêmement faible.

Conformément aux exigences pédagogiques, ce dépôt GitHub sert à la fois de documentation technique pour le déploiement et de rapport étudiant détaillant la démarche de développement, depuis le prétraitement des données jusqu'à l'inférence sur cible.

# 1.2 Contexte Industriel : Vers l'Edge AI

Dans l'industrie 4.0, la maintenance prédictive est un levier crucial pour réduire les temps d'arrêt et optimiser les coûts. Traditionnellement, les données sont envoyées vers le cloud pour analyse, ce qui pose des problèmes de latence, de bande passante et de confidentialité.

Ce projet s'inscrit dans une démarche Edge AI (IA en périphérie) : traiter la donnée localement sur le matériel même qui la génère. Cela permet une réactivité immédiate face aux anomalies et une autonomie accrue du système.

# 1.3 Problématiques et Défis Techniques

Le développement de ce système a dû relever plusieurs défis majeurs identifiés lors de l'analyse du jeu de données AI4I 2020 :
- Déséquilibre des classes : Le dataset présente une forte majorité de cas "sans défaillance" par rapport aux types de pannes spécifiques (TWF, HDF, PWF, OSF, RNF).
- Qualité des données : Certaines instances présentaient des incohérences (pannes globales sans type spécifique ou classes trop peu représentées comme la RNF), nécessitant    un nettoyage approfondi.

# 1.4 Solution Proposée

Pour répondre à ces contraintes, notre approche se divise en deux phases distinctes :
- Entraînement et Optimisation (PC/Cloud) : Utilisation de Python et TensorFlow/Keras pour le prétraitement (normalisation, gestion du déséquilibre via SMOTE et undersampling), la conception du modèle et l'entraînement.
- Déploiement Embarqué (STM32) : Conversion du modèle au format .tflite, intégration via STM32Cube.AI, et développement de l'application C pour l'acquisition capteur et l'inférence.


# 2 Architecture du Projet

Le projet contient les fichiers suivants :
```
projet_IA
├── TP_IA_EMBARQUEE.ipynb          # Notebook d'entraînement et d'export du modèle (Google Colab)
├── Communication_STM32_NN.py      # Script de communication UART PC ↔ STM32
├── app_x-cube-ai.c / .h           # Fichiers C générés par STM32Cube.AI + modifications UART
├── model_SMOTE.tflite            # Modèle TensorFlow Lite optimisé pour l'embarqué
├── X_test.npy / Y_test.npy          # Données de test pour validation PC et cible
├── ai4i2020.csv                   # Dataset brut
└── README.md                      # Documentation technique et rapport étudiant
```

<p align="center">
<img width="250" height="500" alt="mermaid-1774375698300" src="https://github.com/user-attachments/assets/83cdb92f-9955-4512-a66c-ca32c4a257fb" />
</p>


# 3 Dataset Utilisé

## 📊 Dataset utilisé

Le jeu de données utilisé est le **AI4I 2020 Predictive Maintenance Dataset**, qui contient **10 000 instances** de données issues de capteurs industriels. Chaque instance représente l'état de fonctionnement d'une machine et est associée à un label indiquant si une panne s'est produite et, le cas échéant, le type de panne.

### Types de défaillances

| Code | Nom complet | Description |
|------|-------------|-------------|
| `TWF` | Tool Wear Failure | Défaillance due à l'usure de l'outil |
| `HDF` | Heat Dissipation Failure | Défaillance liée à une mauvaise dissipation thermique |
| `PWF` | Power Failure | Défaillance due à un problème de puissance |
| `OSF` | Overstrain Failure | Défaillance causée par une surcharge mécanique |
| `RNF` | Random Failure | Défaillance aléatoire |

### Analyse du dataset

Le dataset présente un **fort déséquilibre de classes** :

- **9 661 cas** sans panne (≈ 96.6%)
- **339 cas** avec panne (≈ 3.4%), soit un ratio de 1:28

Parmi les pannes, la répartition est la suivante :

| Type | Occurrences |
|------|-------------|
| HDF  | 115 |
| OSF  | 98  |
| PWF  | 95  |
| TWF  | 46  |
| RNF  | 19  |

Par ailleurs, **9 machines** présentaient une défaillance globale sans qu'aucun type de panne spécifique ne soit renseigné (*No Specific Failure*).

### Features utilisées

Les 5 colonnes de capteurs suivantes ont été utilisées comme entrées du modèle :

| Feature | Description |
|---------|-------------|
| `Air temperature [K]` | Température de l'air |
| `Process temperature [K]` | Température du processus |
| `Rotational speed [rpm]` | Vitesse de rotation |
| `Torque [Nm]` | Couple mécanique |
| `Tool wear [min]` | Usure de l'outil |

Les colonnes `UDI`, `Product ID` et `Type` ont été exclues car elles n'ont pas de valeur prédictive.

### Rééquilibrage

Du fait du déséquilibre important (1:28), le modèle entraîné sans rééquilibrage prédisait quasi-exclusivement *"No Failure"* malgré une accuracy apparente de 98%. Un rééquilibrage par **SMOTE** a été appliqué uniquement sur le jeu d'entraînement afin d'éviter toute fuite de données (*data leakage*).
