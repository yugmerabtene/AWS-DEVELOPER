# **Module 1 – Introduction à AWS pour les développeurs**

## **Objectifs pédagogiques**

À l’issue de ce module, les participants seront capables de :

* Comprendre l’architecture physique et logique d’AWS (régions, AZs, edge locations).
* Identifier les impacts de la localisation des ressources sur la latence, la conformité et la disponibilité.
* Expliquer le modèle de facturation AWS et anticiper les coûts à l’aide des outils budgétaires.
* Gérer les identités et les permissions avec IAM (utilisateurs, rôles, groupes, politiques JSON).
* Interagir avec AWS via la CLI et les SDK (Python avec Boto3, JavaScript avec AWS SDK).

---

## **1. Architecture globale AWS**

### 1.1 Régions AWS

* Une **région** AWS est une zone géographique indépendante comprenant plusieurs zones de disponibilité (AZ).
* Chaque région est **isolée** des autres pour des raisons de résilience, conformité et latence.
* Exemples :

  * `us-east-1` : Virginie du Nord (région historique, très utilisée)
  * `eu-west-1` : Irlande
  * `eu-west-3` : Paris
* La **région doit être choisie** en fonction :

  * de la proximité des utilisateurs finaux (latence),
  * de la réglementation locale (ex. RGPD),
  * des services disponibles (certains services ne sont pas partout).

### 1.2 Zones de disponibilité (Availability Zones – AZ)

* Une **AZ** est un ou plusieurs datacenters situés dans une même région, mais isolés électriquement et physiquement.
* Objectif : garantir **la continuité de service** en cas de panne d’un datacenter.
* Chaque région contient au moins 2 AZ, souvent 3 voire plus.
* Exemple pour Paris :

  * `eu-west-3a`, `eu-west-3b`, `eu-west-3c`
* Un service multi-AZ permet une **réplication automatique et une haute disponibilité** (ex. RDS Multi-AZ, Load Balancer, Auto Scaling).

### 1.3 Edge Locations

* Utilisées pour le réseau CDN **Amazon CloudFront** et les services de latence faible.
* Présentes dans des villes du monde entier.
* Permettent :

  * le **caching de contenu statique**,
  * la **réduction du temps de réponse** via la proximité,
  * l’**authentification rapide** grâce à AWS Global Accelerator ou Lambda\@Edge.

---

## **2. Modèle de facturation et de contrôle des coûts**

### 2.1 Modèle de tarification AWS

* **Pay-as-you-go** : paiement à l’usage réel.
* Chaque service a sa propre unité de facturation :

  * EC2 : par seconde, selon le type d’instance
  * Lambda : par milliseconde d’exécution et nombre d’appels
  * S3 : par Go stocké et par type d’opération (PUT, GET…)
  * Data transfer sortant : facturé au Go
* **Types de tarification** :

  * À la demande (On-demand)
  * Réservée (Reserved Instances, Savings Plans)
  * Spot Instances (calcul à très bas coût mais interrompable)

### 2.2 Free Tier (offre gratuite)

* Valable pendant 12 mois pour un nouveau compte.
* Services inclus :

  * 750h d’EC2 t2.micro par mois
  * 1 million d’invocations Lambda
  * 5 Go de S3
  * 25 Go de DynamoDB
* Certains services sont **gratuits à vie** avec limites (AWS Lambda, CloudWatch Logs…).

### 2.3 Outils de gestion des coûts

* **AWS Cost Explorer** :

  * Affiche l’historique des dépenses par service, région, tag…
  * Permet l’analyse des tendances de coûts.

* **AWS Budgets** :

  * Définition de seuils d’alerte.
  * Notification par e-mail ou webhook (SNS).
  * Peut être couplé à CloudWatch ou Lambda pour automatiser des réponses.

* **Tagging** :

  * Balises (clé/valeur) sur toutes les ressources AWS : `"Projet:Formation"`, `"Client:NomEntreprise"`
  * Permet de filtrer, regrouper, tracer et attribuer les coûts par projet.

---

## **3. Gestion des identités et des accès (IAM)**

### 3.1 Concepts clés

* **IAM (Identity and Access Management)** est un service global (non régional) pour la gestion des **droits et des identités**.
* **Par défaut, tout est interdit** dans AWS : l’accès aux ressources doit être explicitement accordé.
* 4 entités principales :

  * **Utilisateur IAM** : humain ou processus avec login/pwd ou clés d’API.
  * **Groupe IAM** : collection logique d’utilisateurs avec mêmes permissions.
  * **Rôle IAM** : identité temporaire assumée par un service (EC2, Lambda...) ou utilisateur externe.
  * **Politique IAM** : document JSON qui définit ce qu’un utilisateur/groupe/rôle peut faire.

### 3.2 Structure d’une politique IAM

Exemple basique :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::mon-bucket"
    }
  ]
}
```

Explication :

* `"Effect"` : `Allow` ou `Deny`
* `"Action"` : ex : `s3:PutObject`, `ec2:StartInstances`
* `"Resource"` : l’ARN (Amazon Resource Name) de la ressource cible
* `"Condition"` : (optionnelle) pour restreindre selon IP, heure, tag, etc.

### 3.3 Types de politiques

* **AWS Managed Policies** : politiques prêtes à l’emploi maintenues par AWS.
* **Customer Managed Policies** : créées et gérées par l’administrateur.
* **Inline Policies** : définies directement dans un utilisateur/groupe/rôle (non réutilisables).

### 3.4 Évaluation des permissions

* IAM applique toujours la règle suivante :

  * Si `Deny` explicite → accès refusé, même si un `Allow` existe.
  * Si `Allow` existe, et qu’aucun `Deny` ne le contredit → accès autorisé.
  * Si rien n’est spécifié → accès refusé par défaut.

---

## **4. Outils pour les développeurs : CLI et SDKs**

### 4.1 AWS CLI (Command Line Interface)

* Permet d’automatiser et tester toutes les actions disponibles dans AWS.

* **Installation** : via pip, `brew`, apt, ou installeur binaire.

* **Configuration** :

  ```bash
  aws configure
  ```

  * Access Key ID
  * Secret Access Key
  * Default region name
  * Output format (json, text, table)

* **Fichiers créés** :

  * `~/.aws/credentials`
  * `~/.aws/config`

* **Commandes courantes** :

  ```bash
  aws s3 ls
  aws ec2 describe-instances
  aws lambda list-functions
  aws dynamodb list-tables
  ```

* **Profils multiples** :

  * `--profile dev` permet de basculer entre comptes

### 4.2 SDK AWS pour développeurs

#### a. Boto3 – SDK Python

* **Installation** :

  ```bash
  pip install boto3
  ```
* **Connexion à un service** :

  ```python
  import boto3
  s3 = boto3.client('s3')
  print(s3.list_buckets())
  ```
* **Exemples courants** :

  * Uploader un fichier dans S3
  * Lire un item DynamoDB
  * Déclencher une Lambda

#### b. AWS SDK JavaScript (v2 ou v3)

* **Installation (v2)** :

  ```bash
  npm install aws-sdk
  ```

* **Connexion et appel API** :

  ```javascript
  const AWS = require('aws-sdk');
  const s3 = new AWS.S3();
  s3.listBuckets((err, data) => {
    console.log(data.Buckets);
  });
  ```

* **Utilisations pratiques** :

  * Upload depuis un navigateur web
  * Génération d’URL pré-signées pour téléchargement sécurisé
  * Appels vers Lambda depuis une API frontend

---
