## 1. Vue d’ensemble de l’infrastructure AWS

### 1.1 Régions et zones de disponibilité

* **Région AWS** : entité géographique autonome, comme `eu-west-3` (Paris), `us-east-1` (Virginie).
* **Zone de disponibilité (AZ)** : un ou plusieurs datacenters redondants dans une région.
* Une région peut avoir plusieurs AZ (ex. `eu-west-3a`, `eu-west-3b`, etc.).
* Permet la **haute disponibilité**, la **tolérance aux pannes** et la **réplication inter-AZ**.

### 1.2 Emplacements périphériques (Edge Locations)

* Utilisés pour les services à faible latence comme **Amazon CloudFront** (CDN).
* Situés partout dans le monde.
* Pas adaptés à l’hébergement direct d’applications, mais utiles pour la diffusion de contenu.

---

## 2. Facturation et gestion des coûts

### 2.1 Modèle de tarification AWS

* **Pay-as-you-go** : paiement à la seconde, au Go ou à l’appel d’API.
* **Facturation par service** : EC2 (temps d’exécution), Lambda (durée et nombre d’appels), S3 (volume stocké + requêtes).
* **Free Tier** : offre gratuite limitée (ex. 1 million d’invocations Lambda/mois).

### 2.2 Outils de gestion des coûts

* **AWS Cost Explorer** : visualisation des coûts par service ou projet.
* **AWS Budgets** : création d’alertes pour éviter les dépassements de budget.
* **Tagging des ressources** : classification par projet, client ou environnement avec des balises (`"Projet:Formation"`, `"Environnement:Dev"`).

---

## 3. Gestion des identités et des accès (IAM)

### 3.1 Concepts clés d’IAM

* **Utilisateur IAM** : identité pour un humain ou un système, avec identifiants.
* **Groupe IAM** : collection d’utilisateurs partageant les mêmes permissions.
* **Rôle IAM** : identité assumable temporairement par un service ou un compte externe (ex : Lambda, EC2).

### 3.2 Politiques IAM

* Fichiers JSON qui définissent ce qu’un **utilisateur/groupe/rôle peut faire**.
* Structure d’une politique :

  ```json
  {
    "Effect": "Allow",
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::mon-bucket/*"
  }
  ```
* Types :

  * **AWS Managed Policies** : prêtes à l’emploi (ex : `AmazonS3ReadOnlyAccess`)
  * **Customer Managed Policies** : personnalisées, pour des besoins spécifiques
* **Évaluation des politiques** :

  * Les règles `Deny` sont prioritaires sur les `Allow`
  * Principe du **moindre privilège** : accorder uniquement ce qui est nécessaire
* Outils : **IAM Policy Simulator** pour tester des politiques

---

## 4. Outils pour développeurs : CLI et SDKs

### 4.1 AWS CLI (Command Line Interface)

* **Installation** : via `pip`, gestionnaire de paquets Linux ou Windows Installer.
* **Configuration** :

  ```bash
  aws configure
  ```

  Saisie :

  * Clé d’accès
  * Secret
  * Région par défaut (`eu-west-3`)
  * Format de sortie (`json`, `table`, `text`)
* **Fichiers de configuration** :

  * `~/.aws/credentials` : stocke les clés d’accès
  * `~/.aws/config` : stocke les préférences régionales et de format
* **Exemples de commandes** :

  ```bash
  aws s3 ls
  aws lambda list-functions
  aws dynamodb list-tables
  ```

### 4.2 AWS SDK – Programmation dans le cloud

#### a. SDK Python (Boto3)

* **Installation** :

  ```bash
  pip install boto3
  ```
* **Connexion à un service AWS** :

  ```python
  import boto3
  s3 = boto3.client('s3')
  response = s3.list_buckets()
  print(response)
  ```
* **Utilisations courantes** :

  * Envoyer un fichier dans S3
  * Lire une ligne dans DynamoDB
  * Appeler une fonction Lambda

#### b. SDK JavaScript (Node.js)

* **Installation** :

  ```bash
  npm install aws-sdk
  ```
* **Connexion à un service** :

  ```javascript
  const AWS = require('aws-sdk');
  const s3 = new AWS.S3();
  s3.listBuckets((err, data) => {
    if (err) console.log(err);
    else console.log(data.Buckets);
  });
  ```
* **Utilisations courantes** :

  * Générer une URL pré-signée pour téléchargement sécurisé
  * Interagir avec une table DynamoDB
  * Déclencher une fonction Lambda depuis une app web
