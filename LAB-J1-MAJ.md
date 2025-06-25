* Objectifs pédagogiques
* Explications étape par étape
* **Architecture du projet claire**
* Scripts **Python et Bash commentés ligne par ligne**
* **Droits IAM nécessaires** à chaque lab
* Instructions **à la fois sur la console AWS et en terminal local**
* **Zéro icône**, 100% texte

---

# 🧱 STRUCTURE DU PROJET (Arborescence)

```
aws-labs/
│
├── lab-01-iam-cli/
│   (aucun code, configuration IAM et CLI)
│
├── lab-02-s3/
│   ├── fichier.txt
│   ├── s3_create_bucket.py
│   ├── s3_upload_file.py
│   ├── s3_list_files.py
│   ├── s3_download_file.py
│   └── s3_delete_file.py
│
├── lab-03-lambda-api/
│   ├── lambda_function.py
│   ├── function.zip
│   ├── deploy_lambda.sh
│   └── test_curl.sh
│
└── lab-04-dynamodb/
    ├── create_table.py
    ├── insert_item.py
    ├── read_item.py
    ├── update_item.py
    └── delete_item.py
```

---

# LAB 1 – CRÉATION UTILISATEUR IAM ET CONFIGURATION AWS CLI

## Objectifs

* Créer un utilisateur IAM avec accès sécurisé
* Gérer AWS depuis le terminal via AWS CLI

## Étapes

1. Console AWS → Rechercher “IAM”
2. Menu “Users” → “Create user”
3. Nom : `dev-user`
4. Cocher : `Programmatic access`

## Groupe IAM à créer : `AWSLabGroup`

### Politiques à attacher au groupe :

| Politique IAM            | Pourquoi                                  |
| ------------------------ | ----------------------------------------- |
| AmazonS3FullAccess       | Pour manipuler les buckets/objets (Lab 2) |
| AmazonDynamoDBFullAccess | Pour la base NoSQL DynamoDB (Lab 4)       |

### Association

* Créer le groupe, y attacher l’utilisateur `dev-user`
* Télécharger le fichier .csv (Access Key ID + Secret)

### Configuration AWS CLI

```bash
aws configure
```

Saisir les informations du fichier `.csv` :

* Access Key ID
* Secret Access Key
* Region : `eu-west-3`
* Format : `json`

### Vérification

```bash
aws sts get-caller-identity
```

---

# LAB 2 – MANIPULER S3 AVEC BOTO3 (SDK PYTHON)

## Objectif

Créer un bucket, y stocker un fichier, le lister, le télécharger et le supprimer.

## Préparation

Créer le fichier `fichier.txt` :

```bash
echo "Bonjour AWS S3" > fichier.txt
```

### Fichier : `s3_create_bucket.py`

```python
import boto3

# Crée un client S3
s3 = boto3.client('s3')

# Nom unique mondialement
bucket_name = 'mon-bucket-yug-demo'

# Création du bucket dans la région Paris
s3.create_bucket(
    Bucket=bucket_name,
    CreateBucketConfiguration={'LocationConstraint': 'eu-west-3'}
)

print("Bucket créé :", bucket_name)
```

### Fichier : `s3_upload_file.py`

```python
import boto3

s3 = boto3.client('s3')
bucket_name = 'mon-bucket-yug-demo'

# Upload d'un fichier local dans S3
s3.upload_file('fichier.txt', bucket_name, 'fichier.txt')

print("Fichier uploadé")
```

### Fichier : `s3_list_files.py`

```python
import boto3

s3 = boto3.client('s3')
bucket_name = 'mon-bucket-yug-demo'

# Lister les fichiers dans le bucket
response = s3.list_objects_v2(Bucket=bucket_name)

for obj in response.get('Contents', []):
    print("Objet :", obj['Key'])
```

### Fichier : `s3_download_file.py`

```python
import boto3

s3 = boto3.client('s3')
bucket_name = 'mon-bucket-yug-demo'

# Télécharger fichier depuis S3 vers local
s3.download_file(bucket_name, 'fichier.txt', 'copie.txt')

print("Téléchargement terminé")
```

### Fichier : `s3_delete_file.py`

```python
import boto3

s3 = boto3.client('s3')
bucket_name = 'mon-bucket-yug-demo'

# Suppression du fichier dans S3
s3.delete_object(Bucket=bucket_name, Key='fichier.txt')

print("Fichier supprimé")
```

---

# LAB 3 – LAMBDA + API GATEWAY

## Objectif

Créer une fonction Lambda et l’exposer via une API HTTP.

## Rôle IAM à créer : `LambdaFullLabRole`

### Politiques à attacher :

| Politique IAM               | Utilité                     |
| --------------------------- | --------------------------- |
| AWSLambdaBasicExecutionRole | Pour écrire dans CloudWatch |
| AmazonS3FullAccess          | Si Lambda accède à S3       |
| AmazonDynamoDBFullAccess    | Si Lambda accède à DynamoDB |

## Étapes

### Fichier : `lambda_function.py`

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Bonjour depuis Lambda !'
    }
```

### Compression

```bash
zip function.zip lambda_function.py
```

### Script : `deploy_lambda.sh`

```bash
aws lambda create-function \
  --function-name HelloLambda \
  --runtime python3.12 \
  --role arn:aws:iam::TON_ID:role/LambdaFullLabRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

Créer une API Gateway HTTP :

```bash
aws apigatewayv2 create-api \
  --name HelloAPI \
  --protocol-type HTTP \
  --target arn:aws:lambda:eu-west-3:TON_ID:function:HelloLambda
```

Ajouter la permission :

```bash
aws lambda add-permission \
  --function-name HelloLambda \
  --statement-id apigateway-permission \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn arn:aws:execute-api:eu-west-3:TON_ID:API_ID/*/*
```

### Test : `test_curl.sh`

```bash
curl https://<API_ID>.execute-api.eu-west-3.amazonaws.com/hello
```

---

# LAB 4 – MANIPULER DYNAMODB AVEC PYTHON

## Objectif

Créer une table NoSQL, y insérer, lire, modifier et supprimer un élément.

## Fichier : `create_table.py`

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
print("Table créée")
```

## Fichier : `insert_item.py`

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Produits')

table.put_item(Item={'id': '123', 'nom': 'Clavier', 'prix': 89.99})
print("Élément inséré")
```

## Fichier : `read_item.py`

```python
item = table.get_item(Key={'id': '123'})
print(item['Item'])
```

## Fichier : `update_item.py`

```python
table.update_item(
    Key={'id': '123'},
    UpdateExpression='SET prix = :val',
    ExpressionAttributeValues={':val': 79.99}
)
print("Prix mis à jour")
```

## Fichier : `delete_item.py`

```python
table.delete_item(Key={'id': '123'})
print("Élément supprimé")
```
