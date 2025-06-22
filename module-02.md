# **Module 2 – Stockage et bases de données**

## **Objectifs pédagogiques**

À l’issue de ce module, les participants seront capables de :

* Expliquer le fonctionnement de S3, ses options de gestion, de sécurité et d’accès.
* Utiliser des fonctionnalités avancées comme le versioning et les URLs pré-signées.
* Comprendre le modèle NoSQL de DynamoDB, ses contraintes de partitionnement et de débit.
* Comparer les bases relationnelles (RDS) et NoSQL (DynamoDB) selon les besoins applicatifs cloud-native.

---

## **1. Amazon S3 – Stockage d’objets**

### 1.1 Présentation générale

* **Amazon S3 (Simple Storage Service)** est un service de stockage d’objets (et non de fichiers ou de blocs).
* Très utilisé pour stocker des :

  * fichiers statiques (images, vidéos, documents),
  * backups,
  * logs d’application,
  * assets web (SPA, fichiers JS/CSS).

### 1.2 Concepts clés

* **Bucket** : conteneur logique globalement unique pour stocker des objets.
* **Objet** : un fichier stocké dans un bucket, identifié par une clé (`key`).
* **Clé (key)** : chemin logique dans le bucket (`/images/logo.png`).
* **Région** : le bucket est associé à une région lors de sa création.
* **Taille max** : 5 To par objet.

### 1.3 Fonctionnalités importantes

* **Versioning** :

  * Active l’historique des versions pour chaque objet.
  * Permet de restaurer des versions supprimées ou modifiées.
  * Doit être activé lors de la création du bucket ou après.
* **Lifecycle Policies** :

  * Déplacement automatique d’objets vers d’autres classes de stockage (S3 IA, Glacier).
  * Suppression automatique après une durée.
* **Storage Classes** :

  * Standard, Intelligent-Tiering, IA (Infrequent Access), Glacier, Glacier Deep Archive.

### 1.4 Sécurité et contrôle d’accès

* **Contrôle par IAM** :

  * Permissions S3 définies via politiques IAM (`s3:PutObject`, `s3:GetObject`, etc.).
* **Contrôle par politique de bucket (bucket policy)** :

  * Permissions attachées directement au bucket.
  * Format JSON.
  * Utilisé pour : rendre un bucket public, donner accès à un autre compte, etc.
* **Encryption** :

  * SSE-S3 : clé gérée par AWS.
  * SSE-KMS : clé gérée par AWS KMS.
  * SSE-C : clé gérée par le client.

### 1.5 Presigned URLs

* Génèrent une URL temporaire et sécurisée pour :

  * Télécharger (`GET`) ou uploader (`PUT`) un objet.
  * Accorder un accès **contrôlé dans le temps** sans exposer les permissions IAM.
* Utilisable via **Boto3** ou le **SDK JS**.
* Exemple (Python) :

  ```python
  url = s3.generate_presigned_url('get_object', 
      Params={'Bucket': 'my-bucket', 'Key': 'doc.pdf'}, 
      ExpiresIn=3600)
  ```

---

## **2. Amazon DynamoDB – Base de données NoSQL gérée**

### 2.1 Présentation

* Base de données **NoSQL serverless** totalement gérée.
* Optimisée pour :

  * **hautes performances** (latence <10ms),
  * **scalabilité automatique**,
  * **applications à forte volumétrie** : IoT, jeux, e-commerce, streaming.

### 2.2 Architecture et modèle de données

* **Table** : contient des éléments (items), chacun avec un schéma libre.
* **Clé primaire obligatoire** :

  * Soit **clé de partition** seule (hash key),
  * Soit **clé de partition + clé de tri** (range key).
* **Pas de jointures**, mais possibilité d’indexation secondaire :

  * **LSI** (Local Secondary Index) : même clé de partition, clé de tri différente.
  * **GSI** (Global Secondary Index) : clés totalement différentes.

### 2.3 Capacité et débit

* **Mode provisionné** :

  * On définit des unités de lecture/écriture (RCU/WCU).
* **Mode à la demande (on-demand)** :

  * Facturation à l’usage, sans configuration de capacité.
* **Throughput élevé**, mais :

  * Risque de **hot partitions** si mauvaise distribution de clé de partition.
  * Nécessité de concevoir des modèles d’accès efficaces.

### 2.4 Sécurité et accès

* Accès via **IAM** (politiques en JSON).
* Contrôle fin sur les actions :

  * Lecture (`dynamodb:GetItem`)
  * Écriture (`dynamodb:PutItem`, `dynamodb:UpdateItem`)
* Possibilité de faire de l’**accès conditionnel** selon :

  * Tag de l’utilisateur
  * Heure, IP source, ou contenu de l’item

---

## **3. Comparaison : DynamoDB vs RDS**

| Critère      | Amazon RDS (relationnel)        | Amazon DynamoDB (NoSQL)                      |
| ------------ | ------------------------------- | -------------------------------------------- |
| Modèle       | Relationnel (SQL)               | NoSQL (clé/valeur, document)                 |
| Schéma       | Défini et structuré             | Flexible, schéma libre                       |
| Transactions | Oui (ACID)                      | Oui (limitées, à clé unique ou groupées)     |
| Échelle      | Verticale (via type d’instance) | Horizontale automatique                      |
| Performance  | Bonne, dépend des indexes       | Latence très faible, scalable                |
| Maintenance  | DBMS à gérer (patchs, tuning)   | Complètement géré (serverless)               |
| Use cases    | ERP, CRM, reporting             | IoT, analytics temps réel, e-commerce rapide |

### Choix guidé :

* **Choisir RDS** si :

  * Vous avez un modèle relationnel structuré (MySQL, PostgreSQL, etc.)
  * Besoin de transactions complexes, reporting, relations entre tables.
* **Choisir DynamoDB** si :

  * Lecture/écriture à haute fréquence
  * Schéma évolutif ou inconnu
  * Performance ultra-réactive sans administration
