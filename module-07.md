# Module 7 – Intégration et Déploiement Continu (CI/CD)

## Objectifs pédagogiques

* Comprendre les principes du CI/CD dans le cloud
* Mettre en œuvre une pipeline automatisée sur AWS
* Utiliser les services AWS CodeCommit, CodeBuild, CodeDeploy, CodePipeline
* Déployer automatiquement des fonctions Lambda ou des conteneurs
* Manipuler AWS SAM pour gérer une architecture serverless de bout en bout

---

## 1. Les services AWS pour le CI/CD

### AWS CodeCommit

Service de gestion de code source compatible Git.
C’est un dépôt Git privé, sécurisé, totalement managé par AWS.
Fonctionnalités principales :

* Chiffrement des données avec KMS
* Intégration IAM pour les autorisations
* Pas besoin d’héberger soi-même un dépôt

### AWS CodeBuild

Service de build automatisé. Il permet de compiler du code, exécuter des tests, produire des artefacts prêts pour le déploiement.
Il fonctionne à partir d’un fichier `buildspec.yml` qui définit :

* Les phases : install, pre\_build, build, post\_build
* Les variables d’environnement
* Les fichiers de sortie à archiver

### AWS CodeDeploy

Service de déploiement automatique vers des instances EC2, fonctions Lambda ou conteneurs ECS.
Points clés :

* Support des déploiements canaris, blue/green, all-at-once
* Décrit les étapes de déploiement via un fichier `appspec.yml`
* Possibilité de rollback automatique en cas d’échec

### AWS CodePipeline

Service d’orchestration d’un pipeline CI/CD complet.
Chaque étape du pipeline peut être connectée à CodeCommit, GitHub, CodeBuild, CodeDeploy ou même des approbations manuelles.
Avantages :

* Orchestration native avec IAM
* Déclenchement automatique à chaque commit
* Historique des déploiements traçable et auditable

---

## 2. Architecture typique d’un pipeline CI/CD

Les étapes classiques sont :

* **Source** : dépôt Git (CodeCommit, GitHub)
* **Build** : compilation et tests automatisés (CodeBuild)
* **Test** : vérifications supplémentaires (tests fonctionnels, sécurité)
* **Approval** : validation manuelle ou automatique
* **Deploy** : déploiement vers Lambda, ECS, EC2

Chaque étape peut être surveillée via CloudWatch pour les logs et les métriques.

---

## 3. Déploiement serverless avec AWS SAM

### Présentation

AWS SAM (Serverless Application Model) est un framework qui étend AWS CloudFormation pour simplifier le développement, le test et le déploiement d’applications serverless.
Il fonctionne avec un fichier `template.yaml` plus simple que les templates CloudFormation classiques.

### Composants typiques dans un projet SAM :

* Fonctions Lambda
* API Gateway (HTTP, REST)
* Tables DynamoDB
* Permissions IAM
* Triggers (S3, SQS, EventBridge...)

### Fonctionnalités :

* Déploiement rapide avec `sam deploy`
* Simulation locale avec `sam local invoke` ou `sam local start-api`
* Packaging automatique dans un bucket S3
* Intégration avec CI/CD (CodePipeline, GitHub Actions...)

### Exemple de fichier `template.yaml` :

```yaml
Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./src/
      Handler: app.lambda_handler
      Runtime: python3.12
      Events:
        HelloApi:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

---

## 4. Bonnes pratiques CI/CD sur AWS

* Utiliser des rôles IAM distincts avec des permissions minimales pour chaque service.
* Ne jamais stocker de secrets dans le code : utiliser AWS Secrets Manager ou Parameter Store.
* Activer la journalisation CloudTrail pour auditer chaque étape.
* Segmenter les environnements (dev, staging, prod) dans des comptes ou VPC séparés.
* Ajouter des étapes de validation humaine avant la mise en production.
* Activer les notifications via SNS pour suivre les statuts de pipeline.

---

## 5. Déploiement automatisé de Lambda ou conteneurs

### Exemple : déploiement de Lambda

* Le code est versionné dans CodeCommit.
* CodeBuild compile et génère le paquet ZIP.
* CodeDeploy met à jour la fonction Lambda en conservant les versions.
* CodePipeline orchestre le tout, de façon événementielle ou planifiée.

### Exemple : déploiement de conteneurs

* Le conteneur est construit dans CodeBuild, tagué et poussé dans ECR.
* CodeDeploy met à jour les tâches ECS.
* CodePipeline surveille les mises à jour et déclenche les étapes.

---

## 6. Stratégies de déploiement avancées

### Déploiement Canary

* Déploie le nouveau code à 10 % des utilisateurs
* Analyse des métriques (erreurs, latence)
* Si OK, déploiement complet
* Sinon, rollback automatique

### Déploiement Blue/Green

* Deux environnements (Blue : actif / Green : en test)
* Bascule du trafic contrôlée via Route 53 ou Application Load Balancer
* Rollback immédiat en cas de souci

Avantages :

* Continuité de service
* Réduction des risques
* Monitoring précis des comportements utilisateur post-déploiement
