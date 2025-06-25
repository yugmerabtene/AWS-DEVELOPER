## Module 4 – API et développement d’applications serverless

---

### 1. **Introduction à l’architecture serverless**

#### 1.1 Qu’est-ce que le serverless ?

* Modèle d’exécution où vous ne gérez **ni serveurs ni conteneurs**.
* Le cloud provider (AWS) alloue dynamiquement les ressources et facture à l’usage.
* Exemple typique : API Gateway + Lambda + DynamoDB.

#### 1.2 Pourquoi utiliser une architecture serverless ?

* Scalabilité automatique.
* Réduction des coûts d’exploitation.
* Faible complexité opérationnelle (infrastructure invisible).
* Adapté aux applications événementielles (API REST, webhook, IoT, traitements asynchrones).

---

### 2. **API Gateway : en profondeur**

#### 2.1 Rôle d’API Gateway

* Passerelle d’entrée des appels HTTP/HTTPS.
* Gère :

  * Routage
  * Sécurité
  * Transformation des requêtes/réponses
  * Authentification
  * Contrôle de débit
  * Monitoring et métriques

#### 2.2 Types d’API : REST vs HTTP vs WebSocket

| Fonctionnalité           | REST API          | HTTP API                                     | WebSocket API |
| ------------------------ | ----------------- | -------------------------------------------- | ------------- |
| Mapping Template         | Oui               | Non (sauf avec VTL sur integration response) | Non           |
| Authorizer (Lambda)      | Oui               | Oui                                          | Oui           |
| JWT Native               | Non               | Oui                                          | Non           |
| Intégration AWS Services | Oui               | Oui (limité)                                 | Non           |
| Cache                    | Oui               | Non                                          | Non           |
| Utilisation recommandée  | Complexes, legacy | Modernes, microservices                      | Temps réel    |

---

### 3. **Intégration API Gateway – Lambda**

#### 3.1 Fonctionnement général

1. Le client envoie une requête HTTP.
2. API Gateway la reçoit, applique les règles (auth, throttling...).
3. API Gateway appelle la fonction Lambda (proxy ou non-proxy).
4. Lambda traite la logique métier.
5. La réponse est renvoyée via API Gateway.

#### 3.2 Lambda Proxy vs Non-Proxy

**Lambda Proxy Integration** :

* `event` contient toute la requête HTTP (headers, path, body, query...).
* Retour : `{"statusCode": 200, "body": "contenu", "headers": {...}}`
* Plus simple à mettre en œuvre, très utilisé dans les projets modernes.

**Lambda Non-Proxy** :

* Mapping VTL en entrée/sortie.
* Permet de transformer des messages, appeler d’autres services AWS sans Lambda.
* Plus flexible mais plus complexe.

#### 3.3 Exemple : Proxy

```python
def handler(event, context):
    name = event["queryStringParameters"].get("name", "inconnu")
    return {
        "statusCode": 200,
        "headers": { "Content-Type": "application/json" },
        "body": json.dumps({ "message": f"Bonjour, {name}" })
    }
```

---

### 4. **Gestion des erreurs**

#### 4.1 Côté Lambda

* Retourner une réponse structurée avec code HTTP explicite (`statusCode`).
* Utiliser `try-except` pour capturer les erreurs.
* Attention : une exception non capturée retourne un code 502 sans explication.

#### 4.2 Côté API Gateway

* Mapping des erreurs : on peut transformer une erreur Lambda en message utilisateur clair.
* Exemple : une erreur `InvalidParameterException` → code 400 + message “Paramètre invalide”.

#### 4.3 Meilleures pratiques

* Centraliser la gestion des erreurs dans une fonction middleware.
* Loguer toutes les erreurs dans CloudWatch Logs.
* Prévoir des réponses personnalisées par type d’erreur.

---

### 5. **Throttling (limitation de débit)**

#### 5.1 Pourquoi ?

* Éviter la saturation du backend.
* Protéger contre les attaques par déni de service léger.
* Garantir une QoS minimale pour tous les utilisateurs.

#### 5.2 Niveaux de throttling

* Par stage : ex. 100 req/s avec burst de 200.
* Par usage plan (liée à API Key) : quotas, limites par seconde, minute ou jour.

#### 5.3 Réponse d’erreur

* Si la limite est dépassée : code **429 Too Many Requests**.

---

### 6. **CORS : Cross-Origin Resource Sharing**

#### 6.1 Contexte

* Si une page web appelle une API hébergée sur un autre domaine, le navigateur bloque l’appel sauf si le serveur accepte les CORS.

#### 6.2 Configuration dans API Gateway

* Autoriser les méthodes : `Access-Control-Allow-Methods`
* Autoriser les origines : `Access-Control-Allow-Origin`
* Autoriser les headers : `Access-Control-Allow-Headers`
* OPTIONS → doit renvoyer les bons headers pour le preflight.

#### 6.3 Exemple

```json
{
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type"
}
```

---

### 7. **Sécurisation des APIs**

#### 7.1 IAM (Identity and Access Management)

* Basé sur des rôles et des politiques.
* Réservé aux clients internes (ex : applications AWS).
* Exemple : EC2 ou Lambda peut appeler l’API via SDK avec ses permissions IAM.

#### 7.2 Lambda Authorizer

* Authentifie chaque requête via un token (Bearer, OAuth, JWT, session...).
* La fonction retourne une policy IAM : `Allow` ou `Deny`.
* Peut ajouter des `context` pour personnaliser la réponse.
* Puissant mais nécessite un temps d'exécution (→ possible surcharge).

#### 7.3 API Keys

* Clé générée côté API Gateway.
* Transmise dans l’en-tête `x-api-key`.
* Liée à un Usage Plan (throttling + quotas).
* Ne remplace pas l’authentification. Utile pour tracking, limitation.

#### 7.4 JWT Authorizer (HTTP API uniquement)

* Authentification native via JWT.
* Pas besoin de Lambda Authorizer.
* Déclare un issuer (`iss`) et une audience (`aud`).
* Plus rapide, moins coûteux.

---

### 8. **Observabilité : logs et métriques**

* **CloudWatch Logs** : journalisation automatique des appels Lambda, des erreurs API Gateway.
* **CloudWatch Metrics** :

  * Nombre d’appels par route.
  * Erreurs par code HTTP.
  * Latence des appels.
  * Taux de throttling.
* **X-Ray (optionnel)** : traçabilité des appels distribués (API → Lambda → DynamoDB...).

---

### Cas d’usage final : "API Produits"

* `/produits` : liste des produits (GET)
* `/produits/{id}` : détails d’un produit (GET)
* `/produits` : créer un produit (POST avec authorizer)
* `/produits/{id}` : suppression (DELETE avec authorizer + API Key)
