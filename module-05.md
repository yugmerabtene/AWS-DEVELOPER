##  Module 5 – Intégration asynchrone et gestion des événements

---

###  Objectifs pédagogiques

À la fin de ce module, les participants seront capables de :

* Comprendre les différences entre l’intégration synchrone et asynchrone.
* Mettre en œuvre des architectures découplées avec **SQS**, **SNS** et **EventBridge**.
* Relier ces services à Lambda pour des traitements automatisés.
* Concevoir des schémas d’**orchestration d’événements** (event-driven architecture).
* Appliquer des patterns de fanout, filtres, et DLQ (Dead Letter Queue).

---

## 1. Introduction : pourquoi l’asynchrone ?

### 1.1 Intégration synchrone

* Requêtes enchaînées (ex : API Gateway → Lambda → base de données).
* Faible latence, retour immédiat.
* **Problème** : couplage fort, pas de tolérance aux pannes entre services.

### 1.2 Intégration asynchrone

* Déclenchements basés sur des **événements ou messages**.
* Les producteurs et les consommateurs sont **découplés**.
* Permet de mettre en file des traitements, rejouer des événements, lisser les pics de charge.

---

## 2. Amazon SQS – Simple Queue Service

### 2.1 Concepts clés

| Terme                   | Description                    |
| ----------------------- | ------------------------------ |
| Message                 | Donnée envoyée dans la file    |
| Producteur              | Service qui envoie un message  |
| Consommateur            | Service qui lit un message     |
| DLQ (Dead Letter Queue) | File pour messages non traités |

### 2.2 Types de files

**Standard Queue**

* Messages livrés **au moins une fois**, parfois en double.
* Pas d’ordre garanti.
* Très scalable et rapide.

**FIFO Queue (First In First Out)**

* Messages livrés **exactement une fois**.
* Ordre strictement conservé.
* Moins de débit (\~300 messages/sec par message group ID).

### 2.3 Architecture type

* Application ou Lambda publie un message dans SQS.
* Lambda ou EC2 lit le message.
* Message supprimé après traitement réussi.

### 2.4 Dead Letter Queue (DLQ)

* En cas d’erreur répétée, les messages sont déplacés dans une autre file (DLQ).
* Permet l’analyse des erreurs, le retraitement manuel, ou les alertes CloudWatch.

### 2.5 Paramètres importants

* **Visibility Timeout** : temps de verrouillage d’un message pendant le traitement.
* **Message Retention** : durée de conservation (jusqu’à 14 jours).
* **Delay Seconds** : délai avant qu’un message soit visible.
* **Batch Size** : nombre de messages traités par lot (pour Lambda ou EC2).

---

## 3. Amazon SNS – Simple Notification Service

### 3.1 Concepts clés

| Terme         | Description                |
| ------------- | -------------------------- |
| Sujet (Topic) | Canal logique de diffusion |
| Publisher     | Source de l’événement      |
| Subscriber    | Cible(s) du message        |

### 3.2 Fonctionnement

* Un message est publié sur un **topic**.
* Il est **diffusé (fanout)** à tous les abonnés :

  * Lambda
  * SQS
  * Email/SMS
  * HTTP/S endpoints

### 3.3 Exemple de fanout avec SNS

```
                       ┌──────────────┐
                       │  Publisher   │
                       └──────┬───────┘
                              ▼
                        ┌────────┐
                        │  SNS   │
                        └───┬────┘
            ┌──────────────┼─────────────┐
            ▼              ▼             ▼
         Lambda 1       Queue SQS      Email
```

### 3.4 Filtres de messages SNS

* Utilisation d'attributs de messages pour faire du **routing conditionnel**.
* Exemple :

  * Attribut `eventType = "invoice"` → Lambda A
  * Attribut `eventType = "user"` → Lambda B

### 3.5 Avantages

* Communication **1→N**, rapide et asynchrone.
* Faible coût.
* Bonne intégration avec Lambda, SQS et d’autres services.

---

## 4. Amazon EventBridge

### 4.1 Présentation

* Service de bus d’événements, successeur logique de CloudWatch Events.
* Permet de gérer des **flux d’événements personnalisés** ou natifs (AWS, SaaS, custom).

### 4.2 Sources d’événements

* Services AWS (S3, EC2, Lambda, etc.).
* Services SaaS (Zendesk, Shopify...).
* Événements personnalisés (application interne).

### 4.3 Cible des règles EventBridge

* Lambda
* Step Functions
* EC2, SQS, SNS, Kinesis, etc.

### 4.4 Filtrage d’événements

* Très fin : via la structure JSON de l’événement (avec wildcard, égalité, etc.).
* Exemple de règle :

```json
{
  "source": ["app.paiement"],
  "detail-type": ["facture"],
  "detail": {
    "montant": [{ "numeric": [">", 5000] }]
  }
}
```

### 4.5 Avantages

* Couplage très faible entre services.
* Routing basé sur contenu structuré.
* Observabilité native.
* Permet des architectures **Event-Driven modernes**.

---

## 5. Comparaison SQS / SNS / EventBridge

| Critère         | SQS              | SNS             | EventBridge                   |
| --------------- | ---------------- | --------------- | ----------------------------- |
| Type            | File             | Pub/Sub         | Bus d’événements              |
| Fanout          | Non              | Oui             | Oui                           |
| Ordonnancement  | FIFO (optionnel) | Non             | Non                           |
| Routing         | Non              | Filtres simples | Filtres avancés               |
| Source multiple | Non              | Non             | Oui (multi services + custom) |
| Persistance     | Oui              | Non             | Oui (log possible)            |

---

## 6. Bonnes pratiques

* Utiliser **SQS + Lambda** pour le traitement fiable en différé.
* Utiliser **SNS** pour le fanout simple et les notifications multi-consommateurs.
* Utiliser **EventBridge** pour la coordination d’événements entre services, ou entre environnements SaaS et AWS.
* Toujours associer une **DLQ à Lambda/SQS** pour éviter la perte silencieuse de messages.
* Éviter les appels directs entre Lambda : passer par des messages.
* Suivre les métriques CloudWatch (Age, NumberOfMessages, Invocations, Errors...).
