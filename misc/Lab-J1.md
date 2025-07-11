# **LAB AWS – Introduction aux Services Cloud avec AWS CLI, SDK, Lambda, S3, DynamoDB, API Gateway**

## Organisation

* **Public cible** : développeurs / DevOps débutants ou intermédiaires
* **Prérequis** : un compte AWS avec droits admin, AWS CLI installée, Python ou Node.js installé selon les labs
* **Durée estimée** : 1 après-midi par lab, 1 à 2 après-midis pour le projet final

---

## **Lab 1 – Création d’un utilisateur IAM et configuration de l’AWS CLI**

### Objectifs pédagogiques

* Créer un utilisateur IAM avec des permissions restreintes et accès programmatique.
* Générer une paire de clés d’accès (Access Key ID + Secret Key).
* Configurer AWS CLI localement.
* Vérifier la connexion en appelant un service AWS via la CLI.

---

### Étapes détaillées

### 1. Connexion à la Console AWS

* Ouvre ton navigateur et connecte-toi à [https://console.aws.amazon.com](https://console.aws.amazon.com) avec un **compte administrateur**.

---

### 2. Création de l’utilisateur IAM

1. Accède au service **IAM**.
2. Dans le menu gauche, clique sur **Users (Utilisateurs)**.
3. Clique sur le bouton **\[Create user]**.
4. Renseigne les informations :

   * **User name** : `dev-user` (ou `yug`)
   * **Access type** : coche **Programmatic access**
5. Clique sur **Next: Permissions**.

---

### 3. Attribution des permissions

1. Sélectionne **Add user to group**.
2. Crée un groupe nommé `S3FullAccessGroup`.
3. Attache la politique **AmazonS3FullAccess** à ce groupe.
4. Clique sur **Create group**, puis **Next** jusqu'à **Create user**.

---

### 4. Création d'une Access Key (manuelle après création)

Si tu as déjà créé l’utilisateur sans générer de clé d’accès, voici la procédure à suivre :

1. Va dans **IAM > Users > yug** (ou `dev-user`).
2. Clique sur l'utilisateur pour ouvrir sa fiche.
3. Accède à l’onglet **Security credentials**.
4. Descends jusqu’à la section **Access keys**.
5. Clique sur **Create access key**.
6. À l’étape d’usage prévu, choisis **Command Line Interface (CLI)**.
7. Clique sur **Next**, puis sur **Create access key**.
8. Une fois créée, clique sur **Download .csv** pour la sauvegarder localement.

 **Ne perds pas le fichier `.csv`**, tu ne pourras **plus voir la Secret Key** après cette étape.

---

### 5. Installer AWS CLI (si ce n’est pas déjà fait)

Lien officiel :
[https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

Vérifie l'installation :

```bash
aws --version
```

---

### 6. Configurer AWS CLI

Dans un terminal (`cmd.exe`, PowerShell ou bash), exécute :

```bash
aws configure
```

Entre les valeurs depuis ton fichier `.csv` :

* **AWS Access Key ID** : (ex : `AKIAIOSFODNN7EXAMPLE`)
* **AWS Secret Access Key** : (ex : `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`)
* **Default region name** : `eu-west-3`
* **Default output format** : `json`

---

### 7. Tester la configuration

Commande :

```bash
aws sts get-caller-identity
```

Résultat attendu :

```json
{
  "UserId": "AIDXXXXXXXXXXXXXX",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/yug"
}
```

---

### En cas d’erreur :

> `InvalidClientTokenId` ou `The security token included in the request is invalid`

Cela signifie que :

* Les clés saisies sont incorrectes ou expirées
* Tu n’as pas encore généré de Access Key pour l’utilisateur IAM


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
