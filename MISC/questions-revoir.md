### 1. Layer Lambda (AWS Lambda Layers)

**Définition :**
Un *Layer* dans AWS Lambda est une **bibliothèque partagée** ou un **ensemble de fichiers** (code, dépendances, binaires) que tu peux **réutiliser dans plusieurs fonctions Lambda**. Cela évite de dupliquer des dépendances communes dans chaque fonction.

**À quoi ça sert :**

* Réduire la taille de ton code source Lambda.
* Séparer les dépendances du code métier.
* Centraliser les bibliothèques partagées entre fonctions.
* Gérer les versions plus facilement.

**Exemple d’usage :**
Tu as plusieurs fonctions Lambda Python qui utilisent `pandas`. Au lieu d’intégrer `pandas` dans chaque archive, tu crées un Layer :

```bash
mkdir python
pip install pandas -t python/
zip -r pandas_layer.zip python/
```

Puis tu publies le Layer :

```bash
aws lambda publish-layer-version \
  --layer-name pandas-layer \
  --zip-file fileb://pandas_layer.zip \
  --compatible-runtimes python3.12
```

Et tu l’ajoutes à ta fonction Lambda :

```bash
aws lambda update-function-configuration \
  --function-name MaFonction \
  --layers arn:aws:lambda:eu-west-3:xxxxxxxxxxxx:layer:pandas-layer:1
```

---

### 2. IAM (Identity and Access Management)

**Définition :**
IAM permet de gérer **les utilisateurs, rôles, groupes et permissions** dans AWS. C’est le système de contrôle d’accès global.

**Composants principaux :**

* Utilisateur IAM : Identité persistante avec mot de passe et/ou clé d’accès.
* Groupe : Ensemble d’utilisateurs partageant les mêmes permissions.
* Rôle IAM : Identité temporaire utilisée par des services comme Lambda, EC2, etc.
* Policy (politique) : Document JSON qui définit les permissions autorisées ou interdites.

**Exemple :**
Tu veux que ta Lambda puisse lire un fichier dans S3.

Tu crées une policy :

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::mon-bucket/*"
  }]
}
```

Tu attaches cette policy à un rôle IAM, et ce rôle est attribué à la fonction Lambda.

**IAM répond à la question :**
*Qui a le droit de faire quoi, sur quelle ressource, et dans quelles conditions ?*

---

### 3. ECS et Fargate

**Définition :**
Amazon ECS (Elastic Container Service) est une solution de **gestion de conteneurs Docker**. Elle permet de déployer, exécuter et scaler des applications conteneurisées.

Il existe deux modes de fonctionnement :

* ECS avec EC2 : tu gères toi-même les instances EC2 qui hébergent les conteneurs.
* ECS avec Fargate : **mode serverless**, AWS gère l’infrastructure automatiquement.

**Exemple :**
Tu développes une API en Node.js. Tu la conteneurises avec Docker, tu pushes l’image sur ECR, puis tu crées une tâche ECS Fargate qui :

* Utilise l’image Docker
* Expose un port
* Est lancée sans avoir à gérer de serveur.

**Avantages de Fargate :**

* Pas de gestion de machines.
* Déploiement rapide.
* Facturation à l’usage.

---

### 4. Versioning S3

**Définition :**
Le *versioning* dans S3 permet de **conserver plusieurs versions d’un même objet**. Ainsi, chaque modification ou suppression d’un fichier est enregistrée comme une nouvelle version.

**Fonctionnement :**

* Chaque objet a une version unique.
* Supprimer un fichier ne le détruit pas, mais crée un "delete marker".
* Tu peux restaurer n’importe quelle version passée.

**Activation :**

```bash
aws s3api put-bucket-versioning \
  --bucket mon-bucket \
  --versioning-configuration Status=Enabled
```

**Exemple :**
Tu uploades deux fois `rapport.pdf` → S3 stocke deux versions.
Tu peux demander une version spécifique avec l’option `--version-id`.

**Utilité :**

* Restauration de fichiers supprimés par erreur.
* Archivage.
* Audits et conformité.

---

### 5. Débit en lecture/écriture de DynamoDB

**Principe :**
DynamoDB fonctionne avec des unités de capacité :

* **RCU (Read Capacity Unit)** : capacité de lecture.
* **WCU (Write Capacity Unit)** : capacité d’écriture.

**Règles :**

* 1 RCU = 1 lecture fortement cohérente par seconde d’un objet de 4 Ko.
* 1 RCU = 2 lectures faiblement cohérentes par seconde (car moins de ressources).
* 1 WCU = 1 écriture par seconde d’un objet de 1 Ko.

**Exemple :**
Tu veux permettre 100 lectures/s d’objets de 4 Ko en lecture forte → il faut 100 RCU.
Tu veux écrire 50 objets de 1 Ko par seconde → il faut 50 WCU.

**Modes de capacité :**

* Provisionné : tu choisis toi-même RCU/WCU.
* On-Demand : AWS ajuste automatiquement.

**Important :**

* Les index secondaires (LSI/GSI) consomment aussi du débit.
* Si tu dépasses la capacité, tu peux avoir du throttling (ralentissement).

---

### 6. Combinaison permettant de générer une Pre-signed URL

**Définition :**
Une pre-signed URL est un lien temporaire permettant d’accéder à un objet S3 **sans authentification**, mais de manière **sécurisée**.

**Conditions nécessaires :**

1. Un bucket S3 contenant un objet.
2. Un utilisateur ou rôle IAM avec la permission `s3:GetObject`.
3. Utiliser un SDK AWS (ex : Boto3, AWS CLI).
4. Fournir une durée d’expiration (en secondes).

**Exemple en ligne de commande :**

```bash
aws s3 presign s3://mon-bucket/fichier.txt --expires-in 600
```

**Exemple en Python (Boto3) :**

```python
import boto3

s3 = boto3.client('s3')
url = s3.generate_presigned_url('get_object',
    Params={'Bucket': 'mon-bucket', 'Key': 'fichier.txt'},
    ExpiresIn=600)
print(url)
```

Ce lien est valable pendant 10 minutes.
La personne qui le possède peut télécharger le fichier, même sans compte AWS.

**Cas d’usage :**

* Téléchargement de fichiers privés.
* Envoi temporaire de documents (CV, contrat...).
* Intégration avec des clients qui ne doivent pas avoir accès au backend.
