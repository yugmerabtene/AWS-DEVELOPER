# **Module 3 – Compute : AWS Lambda et autres services**

**Durée estimée** : 3h (théorie)

---

## **Objectifs pédagogiques**

À l’issue de ce module, les participants seront capables de :

* Comprendre les fondamentaux d’AWS Lambda et les modèles serverless.
* Déployer, sécuriser et déclencher des fonctions Lambda.
* Identifier le rôle des layers, variables d’environnement et IAM dans Lambda.
* Différencier EC2, ECS, Fargate et Lambda selon les cas d’usage cloud-native.

---

## **1. AWS Lambda – Exécution de fonctions serverless**

### 1.1 Présentation

* AWS Lambda est un **service serverless** qui exécute du code **sans provisionner de serveur**.
* Prise en charge de plusieurs langages : **Python, Node.js, Java, Go, .NET, Ruby**.
* **Exécution événementielle** : la fonction est déclenchée automatiquement par un événement (HTTP, S3, SNS, etc.).
* Tarification basée sur :

  * Le **nombre d’exécutions**
  * La **durée d’exécution** (en ms)
  * La **quantité de mémoire** allouée

### 1.2 Modèle d’exécution

* Architecture totalement **stateless**
* Une fonction = un **handler** (`lambda_handler(event, context)` en Python)
* Le code peut être :

  * Déployé depuis un ZIP (via AWS CLI, Console, SDK, ou CI/CD)
  * Déployé depuis un **container Docker** (via ECR)
  * Géré via **AWS SAM** (Serverless Application Model) ou **CloudFormation**

### 1.3 Permissions et IAM

* Chaque fonction Lambda **assume un rôle IAM** pour interagir avec d’autres services AWS.
* Exemple : lecture d’un bucket S3 → rôle avec action `s3:GetObject`
* Le rôle est défini lors de la création ou modifié ensuite.

Exemple de politique attachée au rôle Lambda :

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

### 1.4 Déclencheurs (Triggers)

Lambda peut être appelé automatiquement par des services comme :

| Service déclencheur | Type d’événement            |
| ------------------- | --------------------------- |
| API Gateway         | Requête HTTP (GET/POST/...) |
| S3                  | Ajout/suppression d’objet   |
| DynamoDB Streams    | Modification de données     |
| CloudWatch Events   | Programmation (cron)        |
| SNS, SQS            | Messagerie/pub-sub          |
| EventBridge         | Routage d’événements        |

### 1.5 Variables d’environnement

* Permettent de configurer une fonction sans modifier le code.
* Exemples : `DB_HOST`, `DEBUG`, `REGION`
* Sécurisables avec KMS pour stockage chiffré.

### 1.6 Lambda Layers

* Permettent d’extraire des bibliothèques ou dépendances communes dans une **couche partagée**.
* Réduction de duplication entre fonctions.
* Exemple : inclure `pandas`, `requests`, `boto3` dans un layer réutilisable.
* Limite : 5 layers max par fonction.

---

## **2. EC2, ECS, Fargate – Comparatif des services Compute**

### 2.1 EC2 (Elastic Compute Cloud)

* Service d’**infrastructure classique** pour déployer une machine virtuelle.
* Nécessite :

  * Choix du type d’instance
  * Gestion du système d’exploitation
  * Patchs, firewall, clés SSH, configuration réseau
* Cas d’usage :

  * Applications existantes à migrer
  * Besoins spécifiques en OS ou pilotes
  * Applications nécessitant un long runtime (ex : serveurs web persistants)

### 2.2 ECS (Elastic Container Service)

* Orchestrateur de **conteneurs Docker** propre à AWS.
* Permet de définir des **tâches (tasks)** contenant une ou plusieurs images.
* Deux modes de déploiement :

  * **ECS on EC2** : conteneurs tournent sur des instances EC2 gérées par l’utilisateur.
  * **ECS on Fargate** : AWS gère automatiquement l’infrastructure.

### 2.3 Fargate

* Mode **serverless pour conteneurs**.
* Plus besoin de gérer d’instances EC2.
* L’utilisateur définit :

  * L’image à exécuter
  * La mémoire et CPU
  * Les réseaux, variables d’env, IAM
* AWS gère le scaling, le runtime, et la sécurité.
* Cas d’usage :

  * Microservices
  * Tâches de traitement par lots
  * Conteneurs isolés, déclenchés ponctuellement

---

## **3. Comparaison Lambda / EC2 / ECS / Fargate**

| Critère                  | EC2                    | ECS                  | Fargate               | Lambda                 |
| ------------------------ | ---------------------- | -------------------- | --------------------- | ---------------------- |
| Modèle                   | VM                     | Orchestration Docker | Conteneurs serverless | Fonctions serverless   |
| Gestion serveur          | Oui                    | Oui (sur EC2)        | Non                   | Non                    |
| Runtime persistant       | Oui                    | Oui                  | Oui                   | Non (max 15 min)       |
| Scalable automatiquement | Non (Auto Scaling req) | Oui                  | Oui                   | Oui                    |
| Cas d’usage              | Web serveur, legacy    | Microservices Docker | Traitement conteneur  | Backend événementiel   |
| Coût                     | Fixe (selon instance)  | Variable             | À l’usage             | À l’usage (ms + invoc) |

