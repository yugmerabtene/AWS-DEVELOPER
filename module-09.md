# Module 9 – Résilience et Bonnes Pratiques Applicatives

## Objectifs pédagogiques

* Renforcer la tolérance aux pannes des applications cloud
* Comprendre les mécanismes de retry, timeout, et files de secours
* Mettre en place des déploiements progressifs et sûrs (Canary, Blue/Green)
* Appliquer les meilleures pratiques pour gérer les erreurs en production

---

## 1. Retry et Exponential Backoff

### Problématique

En environnement cloud, certaines erreurs sont transitoires (ex. : dépassement de quota, latence réseau, appel d’API externe échoué). Une simple nouvelle tentative peut suffire.

### Retry automatique

AWS (ex : Lambda, SQS, SDK clients) propose des politiques de retry automatiques.

### Exponential Backoff

* Les tentatives sont espacées selon un intervalle croissant (1s, 2s, 4s, 8s…).
* Cela réduit la congestion en cas de panne temporaire.
* Un facteur aléatoire ("jitter") est souvent ajouté pour éviter que toutes les fonctions ne se relancent en même temps.

### Stratégie complète

* Premier échec → retry immédiat
* Si échec persiste → attente avec backoff exponentiel
* Après un nombre maximal de tentatives → envoi vers une file de secours (DLQ)

---

## 2. Timeout, DLQ et gestion des erreurs

### Timeout

* Chaque fonction Lambda possède un `timeout` configurable (jusqu’à 15 minutes).
* Si le traitement dépasse cette durée, l’exécution est interrompue automatiquement.

### DLQ – Dead Letter Queue

* Une file de secours (souvent SQS) utilisée pour stocker les messages non traités après plusieurs tentatives.
* Permet l’analyse manuelle ou le traitement différé.

### Destinations asynchrones

* `on-success` : déclenchée si la Lambda réussit.
* `on-failure` : déclenchée si tous les essais échouent.
* Cela permet par exemple d’archiver un message échoué dans S3, ou d’alerter une équipe via SNS.

### Exemple de flux avec SNS et Lambda

1. SNS publie un message.
2. La Lambda abonnée traite le message.
3. En cas d’erreur ou timeout :

   * Retry automatique
   * Puis message envoyé dans une DLQ SNS
4. Possibilité de configurer une Lambda secondaire pour récupérer les messages échoués

---

## 3. Déploiements Canary et Blue/Green

### Objectif

Déployer du nouveau code **sans interruption de service**, tout en **réduisant les risques** grâce à des tests progressifs.

---

### Déploiement Canary

#### Fonctionnement :

* Un petit pourcentage du trafic est redirigé vers la nouvelle version (par exemple 10 %).
* Si aucune anomalie n’est détectée, la montée en charge est progressive.
* Si une erreur est détectée, rollback immédiat vers la version précédente.

#### Cas d’usage :

* Déploiement Lambda avec CodeDeploy
* Monitoring via CloudWatch ou X-Ray pour vérifier la stabilité

---

### Déploiement Blue/Green

#### Fonctionnement :

* Deux environnements indépendants :

  * **Blue** : version actuelle, stable, en production
  * **Green** : nouvelle version, testée en parallèle
* Le trafic est basculé progressivement de Blue vers Green via :

  * Application Load Balancer
  * DNS pondéré avec Route 53

#### Étapes :

1. Déploiement du code dans l’environnement Green
2. Tests manuels ou automatisés
3. Si tout est stable, bascule de trafic vers Green
4. L’environnement Blue reste disponible en secours (rollback immédiat possible)

#### Avantages :

* Zéro interruption
* Détection précoce des anomalies
* Sécurité renforcée lors de mises à jour majeures
* Rollback rapide et propre
