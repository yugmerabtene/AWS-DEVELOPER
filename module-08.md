# Module 8 – Monitoring, Débogage et Traçabilité

## Objectifs pédagogiques

* Mettre en place la supervision des applications AWS
* Comprendre les métriques et logs CloudWatch
* Utiliser X-Ray pour le traçage distribué
* Assurer l’audit et la traçabilité des actions via CloudTrail

---

## 1. Amazon CloudWatch : Logs, Metrics et Dashboards

### Logs CloudWatch

* Collecte les logs générés par les services AWS comme Lambda, ECS, EC2, etc.
* Chaque fonction Lambda ou container peut envoyer ses logs directement dans un groupe de logs.
* Permet d’analyser les erreurs, les exceptions, les événements système.

### Metrics CloudWatch

* Indicateurs chiffrés en temps réel (CPUUtilization, MemoryUsage, Duration, Invocations, Errors, etc.).
* Les métriques peuvent être natives (exposées par AWS) ou personnalisées (définies par l’utilisateur via les SDK ou API).

### Dashboards CloudWatch

* Interface graphique permettant d’agréger et visualiser les métriques sélectionnées.
* Supporte l’utilisation de graphiques personnalisés pour surveiller plusieurs ressources dans un seul écran.
* Peut être partagé entre équipes (accès IAM) pour une vue centralisée.

### Alarmes CloudWatch

* Définissent des seuils sur des métriques (ex. : CPU > 80 % pendant 5 minutes).
* Permettent d’envoyer des notifications SNS, d’exécuter des fonctions Lambda, ou même d’arrêter une instance.

---

## 2. Alarmes sur seuils personnalisés

### Objectif

* Détecter des anomalies en production
* Réagir automatiquement (notification, scalabilité, redéploiement)

### Exemple avec Amazon RDS

* Surveillance de métriques telles que :

  * CPUUtilization
  * ReadLatency / WriteLatency
  * FreeableMemory

### Déclenchement d’actions

* Si un seuil est dépassé, déclenchement de :

  * Notification par SNS
  * Exécution d’une fonction Lambda pour corriger automatiquement
  * Ouverture d’un ticket via EventBridge vers un outil de gestion des incidents

---

## 3. AWS X-Ray : Tracing distribué

### Objectif

* Visualiser et diagnostiquer les performances d’une application distribuée
* Identifier les goulots d’étranglement, erreurs ou appels lents entre microservices

### Fonctionnement

* Chaque requête qui traverse plusieurs services (ex : API Gateway → Lambda → DynamoDB) est traçable.
* Chaque composant ajoute une trace à la requête (segment).
* X-Ray centralise tous les segments en une seule vue.

### Architecture type

* Une requête est envoyée via Route 53 vers une API hébergée sur Fargate ou Lambda.
* Un conteneur "sidecar" X-Ray capte les appels entre microservices.
* Ces informations sont envoyées à AWS X-Ray pour analyse.
* Les services comme DynamoDB ou Step Functions peuvent être instrumentés également.

### Avantages

* Vue complète du parcours d'une requête
* Aide à identifier les appels lents
* Aucune gestion manuelle des serveurs nécessaire

---

## 4. AWS CloudTrail : Audit des actions et conformité

### Objectif

* Assurer la traçabilité des actions administratives et programmatiques sur AWS
* Répondre aux exigences de conformité (ISO 27001, RGPD...)

### Fonctionnement

* Enregistre chaque appel API effectué via la console AWS, CLI ou SDK.
* Les événements sont stockés dans un bucket S3.
* Ces logs sont structurés en JSON avec : nom du service, utilisateur, heure, IP, paramètres, etc.

### Exploitation

* Requêtage via Athena ou Amazon OpenSearch
* Analyse temps réel possible via EventBridge ou intégration avec un SIEM
* Peut être utilisé pour reconstituer la chronologie d'une attaque ou d’un incident

### Cas d’usage

* Audit RGPD : qui a accédé à quel bucket S3 ?
* Sécurité : détection d’usage anormal du compte root
* Investigation forensic : reconstruction post-incident
