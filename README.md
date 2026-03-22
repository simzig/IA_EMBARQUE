# IA_EMBARQUE
GITHUB du projet IA embarqué

# 1. Introduction et Contexte du Projet
#    1.1 Objectif Général
   
Ce projet a pour ambition de concevoir, optimiser et déployer un Réseau de Neurones Profonds (DNN) directement sur un microcontrôleur STM32L4R9. L'objectif final est de réaliser une tâche de maintenance prédictive en temps réel, capable d'analyser les données de capteurs industriels pour anticiper les défaillances machines avec une empreinte mémoire extrêmement faible.

Conformément aux exigences pédagogiques, ce dépôt GitLab sert à la fois de documentation technique pour le déploiement et de rapport étudiant détaillant la démarche de développement, depuis le prétraitement des données jusqu'à l'inférence sur cible.

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
- Déploiement Embarqué (STM32) : Conversion du modèle au format h5, intégration via STM32Cube.AI, et développement de l'application C pour l'acquisition capteur et l'inférence.
