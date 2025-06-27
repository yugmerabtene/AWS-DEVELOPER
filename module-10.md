# Module 10 – Infrastructure as Code avec AWS CloudFormation

## Objectifs pédagogiques

* Comprendre le concept d’infrastructure-as-code (IaC)
* Automatiser le déploiement de ressources AWS avec CloudFormation
* Savoir structurer des templates YAML/JSON robustes
* Gérer les mises à jour, rollbacks et dépendances entre ressources
* Comparer CloudFormation avec Terraform et SAM

---

## 1. Introduction à CloudFormation

AWS CloudFormation permet de décrire toute l’infrastructure d’un projet sous forme de fichiers texte déclaratifs (YAML ou JSON), appelés **templates**.

Une **stack** CloudFormation est l’ensemble des ressources déployées à partir d’un template (ex : VPC, S3, Lambda, DynamoDB…).

Avantages :

* Automatisation complète de l’infrastructure
* Répétabilité et versionnage (dev, staging, prod)
* Reproductibilité entre environnements
* Intégration avec IAM, CodePipeline, SAM, etc.

---

## 2. Structure d’un template CloudFormation

Un template se compose de plusieurs sections. En YAML :

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Déploiement d'une API avec Lambda

Parameters:
  EnvType:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-api-bucket

Outputs:
  BucketName:
    Value: !Ref MyBucket
    Export:
      Name: MyBucketName
```

---

## 3. Sections essentielles d’un template

### Parameters

* Permettent de rendre le template réutilisable.
* Exemple : type d’environnement, nom d’application, instance type.

### Mappings

* Définissent des couples clé/valeur statiques, comme des AMI par région.

### Conditions

* Permettent de conditionner la création de ressources selon des paramètres (ex. : créer un log group seulement en production).

### Resources

* Partie principale du template : décrit les services AWS à créer.
* Chaque ressource est identifiée par un nom logique (`MyLambdaFunction`) et un type (`AWS::Lambda::Function`).

### Outputs

* Donnent des informations utiles à la fin du déploiement (ex. : URL d’API, ARN de la Lambda).
* Peuvent être exportées pour être réutilisées dans d’autres stacks.

---

## 4. Mise à jour d’une stack et rollback

Lorsqu’on redéploie un template mis à jour :

* CloudFormation applique un **change set** : il compare l’existant avec le nouveau template.
* Si une erreur survient (mauvaise ressource, dépendance rompue, etc.) :

  * Le **rollback** est automatique, pour restaurer l’état précédent.
  * Les logs d’événements permettent d’identifier la source de l’échec.

Bonnes pratiques :

* Tester les changements dans un environnement isolé.
* Utiliser des noms logiques constants pour éviter les suppressions/recréations inutiles.
* Sauvegarder les templates dans un dépôt Git versionné.

---

## 5. Comparaison avec Terraform et SAM

| Critère                | CloudFormation                | Terraform                                | SAM (Serverless Application Model)         |
| ---------------------- | ----------------------------- | ---------------------------------------- | ------------------------------------------ |
| Fournisseur            | AWS uniquement                | Multi-cloud (AWS, Azure, GCP…)           | AWS uniquement                             |
| Langage                | YAML / JSON                   | HCL (HashiCorp Configuration Language)   | YAML (extension CloudFormation)            |
| Modèle                 | Déclaratif                    | Déclaratif                               | Déclaratif (simplifié pour Lambda)         |
| État                   | Géré par AWS                  | État local ou distant (backend)          | Géré par CloudFormation                    |
| Usage recommandé       | Infrastructures AWS complexes | Environnements hybrides, multi-cloud     | Applications serverless avec Lambda, API   |
| Courbe d’apprentissage | Modérée                       | Moyenne à élevée                         | Faible à modérée (usage simplifié)         |
| Intégration GitOps     | Native (avec CodePipeline)    | Excellente (avec GitHub Actions, ArgoCD) | Native (supporte sam deploy, sam local...) |

---

## 6. Bonnes pratiques CloudFormation

* Utiliser **des modules ou des nested stacks** pour diviser les gros templates.
* Ajouter des **tags aux ressources** pour le suivi et la facturation.
* Employer **des variables d’environnement** pour la portabilité.
* Valider les templates avec `aws cloudformation validate-template`.
* Simuler les changements avec les **Change Sets** avant application.
* Archiver les templates dans Git pour suivre l’évolution des ressources.
* Gérer les secrets en dehors du template (Parameter Store ou Secrets Manager).
