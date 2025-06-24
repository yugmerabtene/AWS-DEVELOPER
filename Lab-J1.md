# **LAB AWS – Introduction aux Services Cloud avec AWS CLI, SDK, Lambda, S3, DynamoDB, API Gateway**

---

## Organisation

* **Public cible** : développeurs / DevOps débutants ou intermédiaires
* **Prérequis** : un compte AWS avec accès administrateur, AWS CLI installée, Python ou Node.js installé selon les besoins
* **Durée estimée** : 1 après-midi par lab, 1 à 2 après-midis pour le projet final

---

## **Lab 1 – Création d’un utilisateur IAM et configuration de l’AWS CLI**

### Objectifs pédagogiques

* Comprendre la création d'un utilisateur IAM avec des permissions spécifiques.
* Générer et sécuriser des credentials d’accès programmatique.
* Configurer AWS CLI localement.
* Tester les appels à AWS via la CLI.

---

### Étapes détaillées

#### 1. Connexion à la console AWS

* Connectez-vous à [https://console.aws.amazon.com](https://console.aws.amazon.com) avec un **utilisateur disposant de droits administrateur** (ex. `root` ou `admin` IAM).

---

#### 2. Création de l’utilisateur IAM

1. Accédez au service **IAM**.
2. Dans le menu de gauche, cliquez sur **Utilisateurs**.
3. Cliquez sur **Ajouter un utilisateur**.
4. Paramètres de l’utilisateur :

   * **Nom de l’utilisateur** : `dev-user` (ou `yug` si personnalisé)
   * **Type d’accès** : cochez **Accès programmatique**
5. Cliquez sur **Suivant**.

---

#### 3. Attribution des permissions

1. Choisissez **Ajouter l’utilisateur à un groupe**.
2. Créez un groupe appelé `S3FullAccessGroup`.
3. Attachez la politique **AmazonS3FullAccess** à ce groupe.
4. Validez.

---

#### 4. Création de la clé d’accès

1. Sur la page de confirmation, cliquez sur **Afficher les clés d’accès**.
2. Téléchargez le fichier `.csv` contenant :

   * `Access Key ID`
   * `Secret Access Key`

**Important** : Ce fichier ne pourra **plus être récupéré** une fois quitté.
Garde-le en lieu sûr pour la configuration CLI.

---

#### 5. Installer AWS CLI (si non installé)

Lien officiel :
[https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Vérifier l'installation :

```bash
aws --version
```

---

#### 6. Configuration d’AWS CLI avec les credentials

Dans le terminal ou `cmd.exe` :

```bash
aws configure
```

Saisir les valeurs du fichier `.csv` :

* **AWS Access Key ID** : (ex : `AKIAIOSFODNN7EXAMPLE`)
* **AWS Secret Access Key** : (ex : `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`)
* **Default region name** : `eu-west-3`
* **Default output format** : `json`

---

#### 7. Tester la configuration avec une commande AWS

```bash
aws sts get-caller-identity
```

Sortie attendue :

```json
{
  "UserId": "AIDXXXXXXXXXXXXXX",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/dev-user"
}
```

Si tu obtiens une erreur comme :

```
An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation: The security token included in the request is invalid.
```

> Cela signifie que les **clés d’accès saisies sont incorrectes ou expirées** :
> Recommence depuis l’étape 4 pour générer une nouvelle paire.

---

### Vérifications complémentaires

* Vérifie que le fichier suivant existe :
  `%USERPROFILE%\.aws\credentials`

* Si besoin, supprime manuellement les anciennes entrées pour réinitialiser :

  ```bash
  del %USERPROFILE%\.aws\credentials
  del %USERPROFILE%\.aws\config
  ```

---

### Résultat attendu

* Utilisateur IAM actif avec accès programmatique
* AWS CLI opérationnelle avec `aws sts get-caller-identity` fonctionnel


---

## **Lab 2 – Manipuler un bucket S3 avec AWS SDK**

### Objectifs :

* Créer un bucket S3.
* Uploader et télécharger des fichiers via SDK.
* Lister et supprimer des objets.

### Étapes en Python (Boto3) :

1. **Préparation de l’environnement** :

   * Python installé
   * Installer Boto3 :

     ```bash
     pip install boto3
     ```

2. **Créer un bucket S3** :

   ```python
   import boto3

   s3 = boto3.client('s3')
   bucket_name = 'mon-bucket-lab-s3-demo'

   s3.create_bucket(Bucket=bucket_name, CreateBucketConfiguration={
       'LocationConstraint': 'eu-west-3'
   })
   ```

3. **Uploader un fichier** :

   ```python
   s3.upload_file('fichier.txt', bucket_name, 'fichier.txt')
   ```

4. **Lister les objets** :

   ```python
   response = s3.list_objects_v2(Bucket=bucket_name)
   for obj in response.get('Contents', []):
       print(obj['Key'])
   ```

5. **Télécharger un fichier** :

   ```python
   s3.download_file(bucket_name, 'fichier.txt', 'local.txt')
   ```

6. **Supprimer un fichier** :

   ```python
   s3.delete_object(Bucket=bucket_name, Key='fichier.txt')
   ```

---

## **Lab 3 – Déployer et tester une fonction Lambda avec API Gateway**

### Objectifs :

* Créer une fonction Lambda.
* Exposer via API Gateway.
* Tester l’appel HTTP.

### Étapes :

1. **Code de base Lambda (Python)** :

   ```python
   def lambda_handler(event, context):
       return {
           'statusCode': 200,
           'body': 'Bonjour depuis Lambda !'
       }
   ```

2. **Créer un rôle IAM pour Lambda** avec :

   * `AWSLambdaBasicExecutionRole`

3. **Déployer la fonction Lambda** :

   * Zipper le code :

     ```bash
     zip function.zip lambda_function.py
     ```
   * Créer la fonction avec AWS CLI ou via Console.

4. **Créer une API Gateway HTTP** :

   * Ajouter route `GET /hello`.
   * Intégrer la fonction Lambda.
   * Déployer.

5. **Tester avec curl ou Postman** :

   ```bash
   curl https://<API-ID>.execute-api.eu-west-3.amazonaws.com/hello
   ```

---

## **Lab 4 – Interagir avec une table DynamoDB via SDK**

### Objectifs :

* Créer une table NoSQL.
* Ajouter, lire, modifier et supprimer des données via SDK.

### Étapes en Python (Boto3) :

1. **Créer la table** :

   ```python
   import boto3

   dynamodb = boto3.resource('dynamodb')

   table = dynamodb.create_table(
       TableName='Produits',
       KeySchema=[{'AttributeName': 'id', 'KeyType': 'HASH'}],
       AttributeDefinitions=[{'AttributeName': 'id', 'AttributeType': 'S'}],
       ProvisionedThroughput={'ReadCapacityUnits': 5, 'WriteCapacityUnits': 5}
   )
   table.wait_until_exists()
   ```

2. **Ajouter un élément** :

   ```python
   table.put_item(Item={'id': '123', 'nom': 'Clavier', 'prix': 89.99})
   ```

3. **Lire un élément** :

   ```python
   item = table.get_item(Key={'id': '123'})
   print(item['Item'])
   ```

4. **Mettre à jour un champ** :

   ```python
   table.update_item(
       Key={'id': '123'},
       UpdateExpression='SET prix = :val',
       ExpressionAttributeValues={':val': 79.99}
   )
   ```

5. **Supprimer un élément** :

   ```python
   table.delete_item(Key={'id': '123'})
   ```

---

# **Projet Final – API Serverless complète avec Lambda, S3, DynamoDB, API Gateway**

## Objectif :

Déployer une API REST `POST /upload` qui :

* Reçoit un fichier encodé en base64 et un nom de produit.
* Sauvegarde le fichier dans S3.
* Enregistre le produit dans DynamoDB.
* Retourne l’URL du fichier S3.

---

## Étapes du projet

### Étape 1 – Préparation

1. Créer un bucket S3 :

   ```bash
   aws s3 mb s3://upload-produits-lab --region eu-west-3
   ```

2. Créer une table DynamoDB :

   ```bash
   aws dynamodb create-table \
     --table-name Produits \
     --attribute-definitions AttributeName=id,AttributeType=S \
     --key-schema AttributeName=id,KeyType=HASH \
     --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
   ```

3. Créer un rôle IAM avec :

   * `AmazonS3FullAccess`
   * `AmazonDynamoDBFullAccess`
   * `AWSLambdaBasicExecutionRole`

---

### Étape 2 – Code de la fonction Lambda

```python
import json
import boto3
import uuid
import base64

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Produits')
bucket = 'upload-produits-lab'

def lambda_handler(event, context):
    body = json.loads(event['body'])
    product_name = body['nom']
    file_data = base64.b64decode(body['fichier'])  # base64

    file_name = str(uuid.uuid4()) + '.jpg'
    s3.put_object(Bucket=bucket, Key=file_name, Body=file_data)

    s3_url = f"https://{bucket}.s3.amazonaws.com/{file_name}"
    table.put_item(Item={'id': str(uuid.uuid4()), 'nom': product_name, 'url': s3_url})

    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Upload réussi', 'url': s3_url})
    }
```

---

### Étape 3 – Déploiement

1. Zipper le fichier :

   ```bash
   zip function.zip lambda_function.py
   ```

2. Créer la fonction Lambda :

   ```bash
   aws lambda create-function \
     --function-name UploadProduit \
     --runtime python3.12 \
     --handler lambda_function.lambda_handler \
     --role arn:aws:iam::<ACCOUNT_ID>:role/<ROLE_NAME> \
     --zip-file fileb://function.zip
   ```

3. Ajouter les permissions API Gateway :

   ```bash
   aws lambda add-permission \
     --function-name UploadProduit \
     --statement-id apigateway-permission \
     --action lambda:InvokeFunction \
     --principal apigateway.amazonaws.com \
     --source-arn "arn:aws:execute-api:eu-west-3:<ACCOUNT_ID>:<API_ID>/*/*"
   ```

---

### Étape 4 – Création de l’API Gateway

1. Créer l’API :

   ```bash
   aws apigatewayv2 create-api \
     --name 'APIUploadProduit' \
     --protocol-type HTTP \
     --target arn:aws:lambda:eu-west-3:<ACCOUNT_ID>:function:UploadProduit
   ```

2. Noter l’URL fournie pour la tester.

---

### Étape 5 – Test

Utiliser curl ou Postman pour envoyer une requête POST :

```json
{
  "nom": "Casque Bluetooth",
  "fichier": "BASE64_DU_FICHIER"
}
```

Résultat attendu :

```json
{
  "message": "Upload réussi",
  "url": "https://upload-produits-lab.s3.amazonaws.com/<fichier>.jpg"
}
```
