# LAB 1 – Création d’un utilisateur IAM et configuration AWS CLI

## Objectifs pédagogiques

* Créer un utilisateur IAM avec accès programmatique.
* Générer une paire de clés d'accès (Access Key ID + Secret Key).
* Installer et configurer AWS CLI localement.
* Vérifier la connexion via un appel sécurisé à AWS.

---

## Droits IAM nécessaires (pour l’admin)

| Ressource | Politique IAM requise |
| --------- | --------------------- |
| IAM       | IAMFullAccess         |
| Général   | AWSAccountUsageAccess |

---

## Étapes

### 1. Création de l’utilisateur IAM

1. AWS Console > IAM > Users > Create user
2. Nom de l'utilisateur : `dev-user`
3. Cocher : `Programmatic access`
4. Ajouter au groupe `AWSLabGroup` (ou créer ce groupe)

### 2. Groupe IAM – Politiques à attacher

| Politique IAM                 | Utilité                        |
| ----------------------------- | ------------------------------ |
| AmazonS3FullAccess            | Manipuler objets S3 (Lab 2)    |
| AmazonDynamoDBFullAccess      | Accéder à DynamoDB (Lab 4)     |
| AWSLambda\_FullAccess         | Déployer des fonctions (Lab 3) |
| AmazonAPIGatewayAdministrator | Gérer les API REST             |

### 3. Génération de la clé

1. À l’étape finale, télécharger le `.csv` contenant :

   * Access Key ID
   * Secret Access Key

### 4. Installer et configurer AWS CLI

```bash
aws configure
```

Saisir les valeurs du `.csv` :

* AWS Access Key ID : (copier depuis `.csv`)
* AWS Secret Access Key : (copier)
* Region : `eu-west-3`
* Output format : `json`

### 5. Vérification

```bash
aws sts get-caller-identity
```

---

## Livrable : `prenom_nom_lab-01.zip`

Contenu :

* screenshots/iam\_user\_config.png
* outputs/aws\_config\_test.txt

---

# LAB 2 – Manipuler un bucket S3 avec AWS SDK (Boto3)

## Objectifs pédagogiques

* Créer un bucket S3.
* Uploader, lister, télécharger et supprimer des fichiers depuis le code Python.

---

## Droits IAM nécessaires

| Ressource | Politique AWS      |
| --------- | ------------------ |
| S3        | AmazonS3FullAccess |

---

## Étapes

### 1. Créer un fichier texte à uploader

```bash
echo "Bonjour AWS S3" > fichier.txt
```

### 2. Script : `s3_create_bucket.py`

```python
import boto3

s3 = boto3.client('s3')
bucket_name = 'bucket-yug-demo'

s3.create_bucket(
    Bucket=bucket_name,
    CreateBucketConfiguration={'LocationConstraint': 'eu-west-3'}
)

print("Bucket créé :", bucket_name)
```

### 3. Script : `s3_upload_file.py`

```python
s3.upload_file('fichier.txt', bucket_name, 'fichier.txt')
```

### 4. Script : `s3_list_files.py`

```python
response = s3.list_objects_v2(Bucket=bucket_name)
for obj in response.get('Contents', []):
    print(obj['Key'])
```

### 5. Script : `s3_download_file.py`

```python
s3.download_file(bucket_name, 'fichier.txt', 'copie.txt')
```

### 6. Script : `s3_delete_file.py`

```python
s3.delete_object(Bucket=bucket_name, Key='fichier.txt')
```

---

## Structure du projet

```
lab-02-s3/
├── fichier.txt
├── s3_create_bucket.py
├── s3_upload_file.py
├── s3_list_files.py
├── s3_download_file.py
└── s3_delete_file.py
```

---

## Livrable : `prenom_nom_lab-02.zip`

---

# LAB 3 – Déployer et tester une fonction Lambda avec API Gateway

## Objectifs pédagogiques

* Écrire une Lambda simple.
* Déployer via AWS CLI.
* Exposer via API Gateway HTTP.
* Tester l’API avec `curl`.

---

## Droits IAM nécessaires

| Ressource   | Politique AWS                 |
| ----------- | ----------------------------- |
| Lambda      | AWSLambda\_FullAccess         |
| API Gateway | AmazonAPIGatewayAdministrator |
| CloudWatch  | AWSLambdaBasicExecutionRole   |

---

## Étapes

### 1. Script : `lambda_function.py`

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Bonjour depuis Lambda via API Gateway'
    }
```

### 2. Packaging

```bash
zip function.zip lambda_function.py
```

### 3. Déploiement Lambda

```bash
aws lambda create-function \
  --function-name HelloLambda \
  --runtime python3.12 \
  --handler lambda_function.lambda_handler \
  --role arn:aws:iam::TON_ID:role/LambdaFullLabRole \
  --zip-file fileb://function.zip
```

### 4. Création API Gateway

```bash
aws apigatewayv2 create-api \
  --name HelloAPI \
  --protocol-type HTTP \
  --target arn:aws:lambda:eu-west-3:TON_ID:function:HelloLambda
```

### 5. Permission d’appel

```bash
aws lambda add-permission \
  --function-name HelloLambda \
  --statement-id permission1 \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn arn:aws:execute-api:eu-west-3:TON_ID:API_ID/*/*
```

### 6. Test de l'API

```bash
curl https://API_ID.execute-api.eu-west-3.amazonaws.com/hello
```

---

## Livrable : `prenom_nom_lab-03.zip`

---

# LAB 4 – Interagir avec DynamoDB via Python (Boto3)

## Objectifs pédagogiques

* Créer une table DynamoDB.
* Insérer, lire, modifier et supprimer des éléments via Python.

---

## Droits IAM nécessaires

| Ressource | Politique AWS            |
| --------- | ------------------------ |
| DynamoDB  | AmazonDynamoDBFullAccess |

---

## Étapes

### 1. Script : `create_table.py`

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

### 2. Scripts supplémentaires

* `insert_item.py`
* `read_item.py`
* `update_item.py`
* `delete_item.py`

---

## Structure du projet

```
lab-04-dynamodb/
├── create_table.py
├── insert_item.py
├── read_item.py
├── update_item.py
└── delete_item.py
```

---

## Livrable : `prenom_nom_lab-04.zip`

---

# PROJET FINAL – API Serverless Upload Produit

## Objectifs pédagogiques

* Créer une API REST complète avec Lambda, S3, DynamoDB.
* Automatiser l’enregistrement de produits.
* Gérer les fichiers en base64 via une requête HTTP POST.

---

## Énoncé

Créer une API REST `POST /upload` qui :

* Reçoit un nom de produit et un fichier encodé en base64.
* Sauvegarde le fichier dans S3.
* Enregistre une ligne dans DynamoDB avec :

  * id (UUID)
  * nom
  * url du fichier S3
* Retourne un JSON avec le message et l’URL du fichier.

---

## Droits IAM nécessaires

* AmazonS3FullAccess
* AmazonDynamoDBFullAccess
* AWSLambdaBasicExecutionRole
* AmazonAPIGatewayAdministrator

---

## Étapes

### 1. Lambda : `lambda_function.py`

```python
import json, boto3, uuid, base64

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
bucket = 'upload-produits-bucket'
table = dynamodb.Table('Produits')

def lambda_handler(event, context):
    body = json.loads(event['body'])
    file_data = base64.b64decode(body['fichier'])
    filename = str(uuid.uuid4()) + '.jpg'
    s3.put_object(Bucket=bucket, Key=filename, Body=file_data)
    s3_url = f"https://{bucket}.s3.amazonaws.com/{filename}"
    table.put_item(Item={'id': str(uuid.uuid4()), 'nom': body['nom'], 'url': s3_url})
    return {'statusCode': 200, 'body': json.dumps({'message': 'OK', 'url': s3_url})}
```

### 2. Test API avec `curl_post.sh`

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
│   ├── lambda_code_console.png
│   ├── s3_bucket_file.png
│   ├── dynamodb_record.png
│   └── postman_response.png
└── outputs.txt
```

---

## Livrable : `prenom_nom_projet_final_01.zip`