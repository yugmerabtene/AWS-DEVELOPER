##  Module 6 – Configuration et sécurité dans les applications serverless AWS

---

###  Objectifs pédagogiques

À la fin de ce module, les participants sauront :

* Gérer la configuration d'une application sans la coder en dur.
* Faire la différence entre paramètres de configuration et secrets.
* Utiliser AWS Systems Manager Parameter Store et AWS Secrets Manager.
* Mettre en œuvre le chiffrement avec AWS KMS.
* Appliquer les bonnes pratiques de **sécurité des credentials** dans des applications Lambda et cloud-native.

---

## 1. Introduction

### 1.1 Pourquoi gérer la configuration et les secrets proprement ?

* Éviter de **hardcoder** une configuration dans le code (ex : endpoints, mots de passe, tokens).
* Centraliser et versionner les paramètres.
* Respecter les exigences **de sécurité, traçabilité, et conformité** (ISO/IEC 27001, RGPD, PCI-DSS…).
* Faciliter les changements sans redéploiement.

---

## 2. Parameter Store vs Secrets Manager

### 2.1 Parameter Store (AWS Systems Manager)

#### Fonctionnalités

* Stockage de **paires clé-valeur** (configuration simple ou complexe).
* Gestion des versions.
* Chiffrement possible avec **KMS**.
* Possibilité d’organiser les paramètres en **hiérarchie (`/app/stage/cle`)**.
* Lecture via SDK AWS, CLI ou Lambda (`boto3`, `aws-sdk`...).

#### Deux types :

| Type                   | Description                                        |
| ---------------------- | -------------------------------------------------- |
| **Standard Parameter** | Gratuit, limité à 4 Ko, jusqu’à 10 000 paramètres  |
| **Advanced Parameter** | Payant, versioning étendu, notifications, policies |

#### Cas d’usage

* Endpoints API, chemins de fichiers, configuration par environnement (`/prod/db-url`, `/dev/api-url`).

#### Ex :

```bash
aws ssm put-parameter \
 --name "/app/prod/url" \
 --value "https://api.example.com" \
 --type "String"
```

---

### 2.2 Secrets Manager

#### Fonctionnalités

* Spécialisé dans la gestion des **informations sensibles** :

  * Mots de passe
  * Clés API
  * Tokens d’accès
  * Paires utilisateur/mot de passe

* **Chiffrement automatique avec AWS KMS**.

* Rotation automatique des secrets (ex : pour RDS, Redshift, custom via Lambda).

* Intégration native avec Lambda, RDS, etc.

* Suivi des accès via **CloudTrail**.

#### Cas d’usage

* Stocker et sécuriser un mot de passe MySQL pour une Lambda.
* Garder une clé Stripe API.
* Rotation automatique des credentials de base de données.

#### Exemple :

```bash
aws secretsmanager create-secret \
 --name prod/stripe-api-key \
 --secret-string '{"key":"sk_test_123456"}'
```

---

### 2.3 Comparatif

| Critère              | Parameter Store    | Secrets Manager               |
| -------------------- | ------------------ | ----------------------------- |
| Données sensibles    | Optionnel          | Oui (par défaut)              |
| Rotation automatique | Non                | Oui                           |
| Hiérarchie des noms  | Oui                | Non                           |
| Prix                 | Gratuit (standard) | Payant (\~0.40\$/mois/secret) |
| Limite taille        | 4 Ko               | 64 Ko                         |

---

## 3. AWS KMS – Key Management Service

### 3.1 Définition

* Service AWS de gestion des clés de chiffrement.
* Intégré à S3, EBS, RDS, Lambda, Secrets Manager, etc.
* Permet de **chiffrer/déchiffrer** des données côté serveur ou côté client.

### 3.2 Concepts

| Terme               | Description                                            |
| ------------------- | ------------------------------------------------------ |
| KMS Key (CMK)       | Clé maître gérée par AWS ou le client                  |
| Envelope Encryption | AWS chiffre une clé de données (data key) avec une CMK |
| Grants              | Permissions temporaires sur une CMK                    |

### 3.3 Utilisation avec Lambda

* Secrets Manager ou Parameter Store utilisent **KMS pour chiffrer** leurs contenus.
* Lambda a besoin de la permission suivante :

```json
{
  "Effect": "Allow",
  "Action": "kms:Decrypt",
  "Resource": "arn:aws:kms:...:key/my-key"
}
```

### 3.4 Exemple d'appel avec Boto3 (Python)

```python
import boto3
ssm = boto3.client('ssm')
param = ssm.get_parameter(
    Name='/prod/db-password',
    WithDecryption=True
)
print(param['Parameter']['Value'])
```

---

## 4. Bonnes pratiques de gestion des credentials

### 4.1 À faire

* Toujours **chiffrer** les secrets (KMS ou nativement avec Secrets Manager).
* **Ne jamais** stocker des secrets dans le code (même chiffrés).
* Utiliser des **rôles IAM avec le principe du moindre privilège** (Least Privilege).
* Utiliser des **rôles IAM attribués aux Lambdas** et non des clés IAM statiques.
* Activer les **logs CloudTrail** sur l’accès aux secrets.
* Segmenter les paramètres par environnement (`/app/dev/param`, `/app/prod/param`).
* Utiliser des variables d’environnement Lambda uniquement pour des infos non sensibles.

### 4.2 À éviter

* Utiliser `env` pour stocker des mots de passe sensibles.
* Partager une CMK entre trop de services.
* Laisser les clés IAM dans un repo Git.
* Mettre les credentials dans un fichier `.env` en clair.

---

## 5. Cas d’usage combiné

**Architecture sécurisée Lambda + DB MySQL**

* Secrets Manager stocke le mot de passe DB.
* Lambda a un rôle IAM avec :

  * `secretsmanager:GetSecretValue`
  * `kms:Decrypt`
* Le secret est injecté dynamiquement dans la Lambda à l’exécution.
* Aucun secret dans le code.
