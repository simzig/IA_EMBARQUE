# IA_EMBARQUEE
GITHUB du projet IA embarquée
Auteurs: TRAN Théo, WIEST Simon

# 1. Introduction et Contexte du Projet
###    1.1 Objectif Général
   
Ce projet a pour ambition de concevoir, optimiser et déployer un Réseau de Neurones Profonds (DNN) directement sur un microcontrôleur STM32L4R9. L'objectif final est de réaliser une tâche de maintenance prédictive en temps réel, capable d'analyser les données de capteurs industriels pour anticiper les défaillances machines avec une empreinte mémoire extrêmement faible.

Conformément aux exigences pédagogiques, ce dépôt GitHub sert à la fois de documentation technique pour le déploiement et de rapport étudiant détaillant la démarche de développement, depuis le prétraitement des données jusqu'à l'inférence sur cible.

### 1.2 Contexte Industriel : Vers l'Edge AI

Dans l'industrie 4.0, la maintenance prédictive est un levier crucial pour réduire les temps d'arrêt et optimiser les coûts. Traditionnellement, les données sont envoyées vers le cloud pour analyse, ce qui pose des problèmes de latence, de bande passante et de confidentialité.

Ce projet s'inscrit dans une démarche Edge AI (IA en périphérie) : traiter la donnée localement sur le matériel même qui la génère. Cela permet une réactivité immédiate face aux anomalies et une autonomie accrue du système.

### 1.3 Problématiques et Défis Techniques

Le développement de ce système a dû relever plusieurs défis majeurs identifiés lors de l'analyse du jeu de données AI4I 2020 :
- Déséquilibre des classes : Le dataset présente une forte majorité de cas "sans défaillance" par rapport aux types de pannes spécifiques (TWF, HDF, PWF, OSF, RNF).
- Qualité des données : Certaines instances présentaient des incohérences (pannes globales sans type spécifique ou classes trop peu représentées comme la RNF), nécessitant    un nettoyage approfondi.

### 1.4 Solution Proposée

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

Le jeu de données utilisé est le **AI4I 2020 Predictive Maintenance Dataset**, qui contient **10 000 instances** de données issues de capteurs industriels. Chaque instance représente l'état de fonctionnement d'une machine et est associée à un label indiquant si une panne s'est produite et, le cas échéant, le type de panne.

### Types de défaillances

| Code | Nom complet |
|------|-------------|
| `TWF` | Tool Wear Failure |
| `HDF` | Heat Dissipation Failure |
| `PWF` | Power Failure |
| `OSF` | Overstrain Failure |
| `RNF` | Random Failure |

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


# 4 Guide d'Exécution

### Étape 1 — Entraînement et Export du Modèle (Google Colab)

1. Ouvrir le notebook `TP_IA_EMBARQUEE.ipynb` dans Google Colab
2. Exécuter l'intégralité des cellules afin d'entraîner le modèle `model_unbalanced`
3. Le notebook génère automatiquement un fichier `modele_SMOTE.tflite`

> **Note technique — Compatibilité TensorFlow**
> L'export est réalisé au format `.tflite` plutôt que `.h5` afin de garantir la compatibilité avec STM32Cube.AI.
> Google Colab embarque TensorFlow 2.19 avec une version de Python récente, ce qui rend impossible une rétrogradation vers TensorFlow 2.15, version requise par STM32Cube.AI pour lire les fichiers `.h5`.

4. Télécharger les fichiers suivants en local :
   - `modele_SMOTE.tflite` — le modèle entraîné
   - `X_test.npy` / `Y_test.npy` — les données de test pour validation

---

### Étape 2 — Intégration du Modèle (STM32CubeMX / X-CUBE-AI)

1. Créer un nouveau projet dans STM32CubeMX et activer le package **X-CUBE-AI**
2. Dans l'interface IA, importer le fichier `my_mlp_model.tflite`
3. Cliquer sur **Analyze** pour vérifier les ressources nécessaires :

 <img width="600" height="330" alt="image" src="https://github.com/user-attachments/assets/457bc4c9-a4d1-4a07-9dda-759e3b560b53" />


| Ressource | Consommation |
|-----------|-------------|
| Flash     | ~20.2 KiB    |
| RAM       | ~2.3KiB     |

4. Cliquer sur **Generate Code** pour générer l'ossature du projet STM32CubeIDE

---

### Étape 3 — Programmation de l'Inférence en C (STM32CubeIDE)

Le fichier `app_x-cube-ai.c` a été adapté pour gérer la communication avec le PC :

| Paramètre | Détail |
|-----------|--------|
| **Entrée** | 20 octets — 5 variables `float32` : température air, température process, vitesse rotation, couple, usure outil |
| **Sortie** | 5 octets — 5 probabilités de classe renvoyées en `uint8_t` |

Une fonction `synchronize_UART()` a été implémentée pour assurer la synchronisation de la liaison série avec le PC.

1. **Compiler** le projet (deux builds successifs recommandés)
2. **Flasher** la carte STM32 via le bouton **Run / Debug**

---

### Étape 4 — Test en Temps Réel via UART

Une fois la carte flashée et connectée au PC :

1. Appuyer sur le bouton **RESET** de la carte STM32 pour initialiser le programme
2. Ouvrir le script `Communication_STM32_NN.py`
3. Modifier la variable de configuration du port série :
```python
PORT = "COMX"  # Remplacer X par le numéro de port de votre carte
```
4. Lancer le script Python — la communication UART est alors établie et l'inférence s'exécute en temps réel
