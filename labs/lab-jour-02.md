**Lab 5 – Créer une API REST avec API Gateway et Lambda**

Objectifs :

* Créer une fonction Lambda en Python ou Node.js.
* Créer une API REST via API Gateway connectée à la Lambda.
* Configurer les méthodes HTTP (GET ou POST).
* Gérer les CORS, les messages d’erreurs personnalisés et les stages.

Étapes :

1. Créer une Lambda `HelloLambda` qui retourne un message JSON.
2. Déployer la fonction via AWS Console ou AWS CLI.
3. Créer une API REST dans API Gateway.
4. Configurer la méthode GET de l’API pour invoquer la Lambda.
5. Activer CORS pour cette méthode.
6. Déployer un stage (ex : `prod`).
7. Tester l’API avec curl ou Postman.

Livrable attendu : `prenom_nom_lab-05.zip`
Contenu du livrable :

* Code source de la Lambda (`lambda_function.py`)
* Capture écran ou fichier texte contenant la commande curl et la réponse
* Optionnel : export de l’API Gateway au format JSON

---

**Lab 6 – Configurer un flux SQS vers Lambda, et SNS vers Lambda**

Objectifs :

* Créer une file SQS (standard).
* Créer un sujet SNS.
* Configurer une Lambda qui reçoit les événements des deux services.
* Vérifier l’exécution via CloudWatch Logs.

Étapes :

1. Créer une Lambda `EventHandlerLambda`.
2. Créer une file SQS et un sujet SNS.
3. Configurer l’event source mapping entre SQS et la Lambda.
4. Créer une subscription SNS vers la Lambda.
5. Publier un message dans SNS et un autre dans SQS.
6. Vérifier la réception des messages dans les logs CloudWatch.

Livrable attendu : `prenom_nom_lab-06.zip`
Contenu du livrable :

* Code source Lambda (`handler.py`)
* Script de test d’envoi de messages
* Captures des logs dans CloudWatch montrant les messages reçus

---

**Lab 7 – Utiliser Parameter Store pour injecter de la configuration dans une Lambda**

Objectifs :

* Créer un paramètre dans Systems Manager Parameter Store.
* Lire ce paramètre dans une fonction Lambda.
* Exposer la valeur via une API Gateway.

Étapes :

1. Créer un paramètre `/app/config/message` en utilisant AWS Console ou CLI.
   Exemple CLI :
   `aws ssm put-parameter --name "/app/config/message" --value "Bienvenue" --type "String"`
2. Créer une Lambda qui utilise Boto3 pour lire la valeur.
3. Retourner la valeur dans une réponse HTTP.
4. Connecter cette Lambda à une API Gateway pour exposer l'information.
5. Tester avec curl ou Postman.

Livrable attendu : `prenom_nom_lab-07.zip`
Contenu du livrable :

* Code Lambda (`lambda_ssm.py`)
* Capture de l’écran du paramètre dans SSM
* Capture de la réponse de l’API Gateway

---

**Lab 8 – Déclarer une stack CloudFormation pour créer une table DynamoDB et une Lambda**

Objectifs :

* Rédiger une stack CloudFormation en YAML.
* Créer une table DynamoDB.
* Créer une Lambda qui écrit dans cette table.
* Déployer et tester le fonctionnement.

Étapes :

1. Écrire un fichier `stack.yaml` contenant :

   * Une ressource `AWS::DynamoDB::Table`
   * Une ressource `AWS::Lambda::Function`
   * Une politique IAM minimale
2. Déployer la stack avec la commande suivante :
   `aws cloudformation deploy --template-file stack.yaml --stack-name test-lambda-dynamo --capabilities CAPABILITY_IAM`
3. Tester la Lambda manuellement via la console ou CLI.

Livrable attendu : `prenom_nom_lab-08.zip`
Contenu du livrable :

* Fichier `stack.yaml`
* Code Lambda (`lambda_dynamo.py`)
* Capture de la table créée et du test de la Lambda

---

**Projet-02 – Application de gestion de messages Serverless avec API Gateway, Lambda, SQS, SNS, DynamoDB et Parameter Store**

Objectif :
Concevoir une API de gestion de messages totalement serverless, intégrant tous les composants des Labs 5 à 8, avec automatisation par CloudFormation.

Fonctionnalités attendues :

* Un endpoint API Gateway (POST) qui reçoit un message.
* La Lambda de réception envoie le message dans une file SQS.
* Une Lambda connectée à SQS traite le message, l’enregistre dans DynamoDB, et publie une notification SNS.
* Une autre Lambda (ou email) est abonnée au sujet SNS.
* La configuration du message de bienvenue est stockée dans Parameter Store et lue par une Lambda.

Étapes :

1. Déployer toute l’architecture avec un ou plusieurs fichiers CloudFormation.
2. Tester un enchaînement complet de requête POST → enregistrement → notification.
3. Ajouter des logs CloudWatch.
4. (Optionnel) Générer un diagramme d’architecture (avec PlantUML ou simple schéma image).

Livrable attendu : `prenom_nom_projet-01.zip`
Contenu du livrable :

* Code des Lambdas
* Fichier(s) `template.yaml` CloudFormation
* Captures ou fichiers texte montrant les tests réalisés
* Schéma d’architecture
