# **LAB 01 – Création d’un utilisateur IAM et configuration AWS CLI**

## Objectifs pédagogiques

* Créer un utilisateur IAM sécurisé
* Générer une paire de clés pour AWS CLI
* Tester la connexion à AWS en ligne de commande

## Architecture (schéma logique ASCII)

```
+-----------------+       +-----------------------+
| Console AWS IAM | ---> | Utilisateur 'dev-user'|
+-----------------+       +-----------------------+
                                  |
                                  v
                        +--------------------+
                        | AWS CLI configurée |
                        +--------------------+
```

## Droits IAM nécessaires (admin)

| Ressource | Politique IAM |
| --------- | ------------- |
| IAM       | IAMFullAccess |

## Étapes détaillées

### 1. Créer l’utilisateur IAM

* AWS Console → IAM → Users → Create user
* Nom : `dev-user`
* Cocher "Programmatic access"

### 2. Créer un groupe IAM : `AWSLabGroup`

* Politiques à attacher :

| Politique                     | Rôle           |
| ----------------------------- | -------------- |
| AmazonS3FullAccess            | Lab 2          |
| AmazonDynamoDBFullAccess      | Lab 4          |
| AWSLambda\_FullAccess         | Lab 3 + Projet |
| AmazonAPIGatewayAdministrator | Lab 3 + Projet |

* Associer l’utilisateur `dev-user` à ce groupe

### 3. Générer une Access Key

* Après création → télécharger le fichier `.csv`
* ⚠️ Obligatoire pour configurer la CLI

### 4. Installer AWS CLI

[https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

### 5. Configurer CLI

```bash
aws configure
```

Saisir :

* Access Key ID
* Secret Key
* Default region : `eu-west-3`
* Output format : `json`

### 6. Vérification

```bash
aws sts get-caller-identity
```

---

## Arborescence du lab

```
lab-01-iam-cli/
└── (aucun code, configuration uniquement)
```

## Livrable

**prenom\_nom\_lab-01.zip**

---

# **LAB 02 – Manipuler un bucket S3 avec Python (Boto3)**

## Objectifs pédagogiques

* Créer un bucket S3
* Uploader, lister, télécharger, supprimer un fichier avec Boto3

## Architecture

```
+------------+     +-------------------+     +-------------+
| fichier.txt| --> | Bucket S3 AWS     | --> | Actions CLI |
+------------+     +-------------------+     +-------------+
                             ^ Python + Boto3
```

## Droits IAM nécessaires

| Ressource | Politique IAM      |
| --------- | ------------------ |
| S3        | AmazonS3FullAccess |

## Étapes

1. Préparer l’environnement :

```bash
pip install boto3
echo "Bonjour AWS S3" > fichier.txt
```

2. Créer un bucket – `s3_create_bucket.py`

```python
import boto3

s3 = boto3.client('s3')
bucket_name = 'mon-bucket-yug-demo'

s3.create_bucket(Bucket=bucket_name,
    CreateBucketConfiguration={'LocationConstraint': 'eu-west-3'})
```

3. Uploader un fichier – `s3_upload_file.py`

```python
s3.upload_file('fichier.txt', bucket_name, 'fichier.txt')
```

4. Lister fichiers – `s3_list_files.py`

```python
response = s3.list_objects_v2(Bucket=bucket_name)
for obj in response.get('Contents', []):
    print("Objet :", obj['Key'])
```

5. Télécharger – `s3_download_file.py`

```python
s3.download_file(bucket_name, 'fichier.txt', 'copie.txt')
```

6. Supprimer – `s3_delete_file.py`

```python
s3.delete_object(Bucket=bucket_name, Key='fichier.txt')
```

---

## Arborescence

```
lab-02-s3/
├── fichier.txt
├── s3_create_bucket.py
├── s3_upload_file.py
├── s3_list_files.py
├── s3_download_file.py
└── s3_delete_file.py
```

## Livrable

**prenom\_nom\_lab-02.zip**

---

# **LAB 03 – Déployer une Lambda exposée via API Gateway**

## Objectifs pédagogiques

* Déployer une Lambda simple
* L’exposer via API Gateway HTTP
* Tester via `curl` ou Postman

## Architecture

```
+-----------+      +-------------+       +--------------+
| POSTMAN / | -->  | API Gateway | -->   | Lambda Hello |
|   curl    |      +-------------+       +--------------+
```

## Droits IAM nécessaires

| Ressource   | Politique IAM                 |
| ----------- | ----------------------------- |
| Lambda      | AWSLambda\_FullAccess         |
| API Gateway | AmazonAPIGatewayAdministrator |
| Logs        | AWSLambdaBasicExecutionRole   |

## Étapes

1. Écrire le code – `lambda_function.py`

```python
def lambda_handler(event, context):
    return {'statusCode': 200, 'body': 'Bonjour depuis Lambda'}
```

2. Zipper

```bash
zip function.zip lambda_function.py
```

3. Déployer Lambda – `deploy_lambda.sh`

```bash
aws lambda create-function \
  --function-name HelloLambda \
  --runtime python3.12 \
  --handler lambda_function.lambda_handler \
  --role arn:aws:iam::TON_ID:role/LambdaFullLabRole \
  --zip-file fileb://function.zip
```

4. Créer l’API HTTP

```bash
aws apigatewayv2 create-api \
  --name HelloAPI \
  --protocol-type HTTP \
  --target arn:aws:lambda:eu-west-3:TON_ID:function:HelloLambda
```

5. Ajouter permission :

```bash
aws lambda add-permission \
  --function-name HelloLambda \
  --statement-id permission1 \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn arn:aws:execute-api:eu-west-3:TON_ID:API_ID/*/*
```

6. Test – `test_curl.sh`

```bash
curl https://API_ID.execute-api.eu-west-3.amazonaws.com/hello
```

---

## Arborescence

```
lab-03-lambda-api/
├── lambda_function.py
├── function.zip
├── deploy_lambda.sh
└── test_curl.sh
```

## Livrable

**prenom\_nom\_lab-03.zip**

---

# **LAB 04 – Manipuler DynamoDB avec Boto3**

## Objectifs pédagogiques

* Créer une table NoSQL
* Ajouter, lire, modifier, supprimer un enregistrement

## Architecture

```
+--------+       +--------------------+
| Python | <-->  | Table DynamoDB     |
+--------+       +--------------------+
```

## Droits IAM nécessaires

| Ressource | Politique IAM            |
| --------- | ------------------------ |
| DynamoDB  | AmazonDynamoDBFullAccess |

## Étapes

1. Créer la table – `create_table.py`

```python
dynamodb = boto3.resource('dynamodb')
table = dynamodb.create_table(
    TableName='Produits',
    KeySchema=[{'AttributeName': 'id', 'KeyType': 'HASH'}],
    AttributeDefinitions=[{'AttributeName': 'id', 'AttributeType': 'S'}],
    ProvisionedThroughput={'ReadCapacityUnits': 5, 'WriteCapacityUnits': 5}
)
table.wait_until_exists()
```

2. Ajouter un élément – `insert_item.py`

```python
table.put_item(Item={'id': '123', 'nom': 'Clavier', 'prix': 89.99})
```

3. Lire – `read_item.py`

```python
item = table.get_item(Key={'id': '123'})
print(item['Item'])
```

4. Mettre à jour – `update_item.py`

```python
table.update_item(
    Key={'id': '123'},
    UpdateExpression='SET prix = :val',
    ExpressionAttributeValues={':val': 79.99}
)
```

5. Supprimer – `delete_item.py`

```python
table.delete_item(Key={'id': '123'})
```

---

## Arborescence

```
lab-04-dynamodb/
├── create_table.py
├── insert_item.py
├── read_item.py
├── update_item.py
└── delete_item.py
```

## Livrable

**prenom\_nom\_lab-04.zip**

---

# **PROJET FINAL – API Serverless Upload Produit**

## Objectifs pédagogiques

* Intégrer Lambda + API Gateway + DynamoDB + S3
* Créer une API POST /upload
* Gérer les fichiers et métadonnées produits

## Architecture

```
POSTMAN/curl
    |
    v
API Gateway (HTTP)
    |
    v
Lambda UploadProduit
    |               \
    v                v
S3 (fichier)     DynamoDB (id, nom, url)
```

## Droits IAM nécessaires

* AmazonS3FullAccess
* AmazonDynamoDBFullAccess
* AWSLambdaBasicExecutionRole
* AmazonAPIGatewayAdministrator

---

## Étapes

1. Créer S3

```bash
aws s3 mb s3://upload-produits-lab --region eu-west-3
```

2. Créer DynamoDB

```bash
aws dynamodb create-table \
  --table-name Produits \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

3. Code Lambda – `lambda_function.py`

```python
import json, boto3, uuid, base64

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Produits')
bucket = 'upload-produits-lab'

def lambda_handler(event, context):
    body = json.loads(event['body'])
    name = body['nom']
    file_data = base64.b64decode(body['fichier'])

    key = str(uuid.uuid4()) + '.jpg'
    s3.put_object(Bucket=bucket, Key=key, Body=file_data)
    url = f"https://{bucket}.s3.amazonaws.com/{key}"

    table.put_item(Item={'id': str(uuid.uuid4()), 'nom': name, 'url': url})

    return {'statusCode': 200, 'body': json.dumps({'message': 'Upload réussi', 'url': url})}
```

4. Créer Lambda

```bash
zip function.zip lambda_function.py
aws lambda create-function ...
```

5. Ajouter permissions + API Gateway

6. Tester – `curl_post.sh`

```bash
curl -X POST https://API_ID.execute-api.eu-west-3.amazonaws.com/upload \
  -H "Content-Type: application/json" \
  -d '{"nom": "Casque", "fichier": "BASE64"}'
```

---

## Arborescence

```
projet-final-upload/
├── code/
│   └── lambda_function.py
├── scripts/
│   └── curl_post.sh
├── screenshots/
│   ├── lambda_code.png
│   ├── s3_bucket_file.png
│   ├── dynamodb_record.png
│   └── postman_response.png
└── outputs.txt
```

## Livrable

**prenom\_nom\_projet\_final.zip**
