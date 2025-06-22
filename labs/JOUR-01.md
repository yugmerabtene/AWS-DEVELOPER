## **Lab 1 – Création d’un utilisateur IAM et configuration AWS CLI**

### **Objectif**

Créer un utilisateur IAM dédié au développement, l’ajouter à un groupe avec des permissions de base, et configurer la CLI AWS en local.

### **Énoncé**

1. Connectez-vous à la console AWS avec un compte administrateur.
2. Créez un utilisateur IAM nommé `dev-user`.
3. Créez un groupe `Developers` avec la politique gérée `PowerUserAccess`.
4. Ajoutez `dev-user` au groupe.
5. Générez une paire de clés d’accès (Access Key + Secret Access Key).
6. Sur votre terminal, installez AWS CLI s’il n’est pas présent.
7. Configurez la CLI pour `dev-user`.

### **Résolution**

```bash
# Installer AWS CLI (si besoin)
pip install awscli

# Configurer le profil dev-user
aws configure --profile dev-user
```

Saisir :

* AWS Access Key ID : \[collée depuis la console]
* AWS Secret Access Key : \[collée depuis la console]
* Default region name : eu-west-3 (par exemple)
* Default output format : json

### **Validation**

```bash
aws s3 ls --profile dev-user
```

---

## **Lab 2 – Manipuler un bucket S3 avec AWS SDK**

### **Objectif**

Créer un bucket S3, y uploader un fichier, lister les fichiers, puis le télécharger, le tout avec un SDK (Python ou JS).

### **Énoncé**

1. Créez un bucket nommé `formation-dev-[votre-nom]` en région `eu-west-3`.
2. Utilisez le SDK AWS en Python (`boto3`) pour :

   * Uploader un fichier texte
   * Lister les objets du bucket
   * Télécharger ce fichier localement

### **Résolution (Python / Boto3)**

```python
import boto3

s3 = boto3.client('s3')

bucket_name = 'formation-dev-yug'
file_name = 'demo.txt'
key = 'documents/demo.txt'

# Upload
s3.upload_file(Filename=file_name, Bucket=bucket_name, Key=key)

# List
response = s3.list_objects_v2(Bucket=bucket_name)
for obj in response.get('Contents', []):
    print(obj['Key'])

# Download
s3.download_file(Bucket=bucket_name, Key=key, Filename='downloaded_demo.txt')
```

---

## **Lab 3 – Déployer et tester une fonction Lambda avec API Gateway**

### **Objectif**

Créer une fonction Lambda simple, l’exposer via une API REST (API Gateway), et tester un appel HTTP.

### **Énoncé**

1. Créez un rôle IAM nommé `lambda-basic-exec-role` avec la politique `AWSLambdaBasicExecutionRole`.
2. Écrivez une fonction Lambda en Python qui retourne `"Hello, AWS!"`.
3. Déployez-la depuis la console ou AWS CLI.
4. Connectez cette fonction à une API Gateway REST avec un endpoint `/hello`.
5. Testez un appel `GET` depuis votre navigateur ou `curl`.

### **Résolution (extrait en Python 3.9)**

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello, AWS!'
    }
```

### **Étapes via Console**

* Créer Lambda → Runtime Python 3.9
* Ajouter trigger : API Gateway → HTTP API
* Sauvegarder → Copiez l’URL fournie

### **Test**

```bash
curl https://<api-id>.execute-api.eu-west-3.amazonaws.com/hello
```

---

## **Lab 4 – Interagir avec une table DynamoDB via SDK**

### **Objectif**

Créer une table DynamoDB, ajouter un élément, lire un élément, mettre à jour et supprimer via SDK Python (Boto3).

### **Énoncé**

1. Créez une table nommée `Students` avec :

   * Clé de partition : `student_id` (string)
2. Utilisez Boto3 pour faire les opérations suivantes :

   * Ajouter un étudiant
   * Lire ses infos
   * Mettre à jour son nom
   * Supprimer l’étudiant

### **Résolution (Python / Boto3)**

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Students')

# Insert
table.put_item(Item={
    'student_id': '001',
    'name': 'Alice',
    'email': 'alice@example.com'
})

# Read
response = table.get_item(Key={'student_id': '001'})
print(response.get('Item'))

# Update
table.update_item(
    Key={'student_id': '001'},
    UpdateExpression='SET #n = :val1',
    ExpressionAttributeNames={'#n': 'name'},
    ExpressionAttributeValues={':val1': 'Alice Smith'}
)

# Delete
table.delete_item(Key={'student_id': '001'})
```
