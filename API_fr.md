# Documentation API GeRoom

## Vue d'ensemble

Tous les points de terminaison de l'API nécessitent une authentification à l'aide de l'en-tête `x-api-key` et des informations de compte. Une interface de gestion d'API doit être fournie pour :
- Créer des clés API
- Supprimer des clés API
- Nombre maximum de clés API : 30
- Expiration : 120 jours

## Authentification

Tous les points de terminaison nécessitent l'en-tête `x-api-key` pour l'authentification :

```
x-api-key: <your-api-key>
```

## URL de base

```
Host: https://open.ge-room.com/
```

---

## Points de terminaison API

### 1. Obtenir les informations du compte

Récupérer les crédits du compte et le nombre total d'éléments.

**Point de terminaison :** `GET /account-info`

**Paramètres de requête :**
- `account` (chaîne, requis) : Nom du compte

**Exemple de requête :**
```http
GET /account-info?account=example-account-name HTTP/1.1
Host: xxxxx
x-api-key: <your-api-key>
```

**Codes de statut de réponse :**
- `200 OK` - Succès
- `401 Unauthorized` - Non authentifié (clé API invalide ou manquante)
- `400 Bad Request` - Requête invalide (champ compte manquant, format invalide)
- `404 Not Found` - Compte non trouvé
- `500 Internal Server Error` - Erreur serveur

**Réponse de succès (200 OK) :**
```json
{
    "status": "success",
    "data": {
        "credits": 1000,
        "total_items": 42
    }
}
```

**Réponse non authentifiée (401 Unauthorized) :**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Réponse d'erreur (400/404/500) :**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 2. Obtenir tous les éléments

Récupérer une liste de tous les éléments sous un compte (informations de base uniquement).

**Point de terminaison :** `GET /items`

**Paramètres de requête :**
- `account` (chaîne, requis) : Nom du compte

**Exemple de requête :**
```http
GET /items?account=example-account-name HTTP/1.1
Host: xxxxx
x-api-key: <your-api-key>
```

**Réponse de succès (200 OK) :**
```json
{
    "status": "success",
    "data": [
        {
            "id": "item-id-1",
            "name": "item name 1",
            "dimensions": {
                "unit": "cm", 
                "length": 100,
                "width": 100,
                "height": 50
            },
            "img_link": "https://example.com/image-link"
        },
        {
            "id": "item-id-2",
            "name": "item name 2",
            "dimensions": {
                "unit": "cm", 
                "length": 140,
                "width": 120,
                "height": 70
            },
            "img_link": "https://example.com/image-link"
        }
    ]
}
```

**Réponse non authentifiée (401 Unauthorized) :**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Réponse d'erreur (400/404/500) :**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 3. Obtenir les informations complètes d'un élément

Récupérer les informations complètes d'un élément spécifique, y compris toutes les URL de téléchargement.

**Point de terminaison :** `POST /item`

**Corps de la requête :**
```json
{
    "account": "example-account-name",
    "item_id": "item-id-1"
}
```

**Exemple de requête :**
```http
POST /item HTTP/1.1
Host: xxxxxx
x-api-key: <your-api-key>
Content-Type: application/json
Content-Length: <length>

{
    "account": "example-account-name",
    "item_id": "item-id-1"
}
```

**Réponse de succès (200 OK) :**
```json
{
    "status": "success",
    "data": {
        "id": "item-id-1",
        "name": "item name",
        "dimensions": {
                "unit": "cm", 
                "length": 100,
                "width": 100,
                "height": 50
        },
        "img_link": "https://example.com/image-link",
        "ar_link": "https://example.com/ar-link",
        "fbx_url": "https://example.com/download/fbx",
        "glb_url": "https://example.com/download/glb",
        "usdz_url": "https://example.com/download/usdz"
    }
}
```

**Réponse non authentifiée (401 Unauthorized) :**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Réponse d'erreur (400/404/500) :**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 4. Créer une tâche

Soumettre une tâche pour générer des modèles 3D ou des peintures à partir d'images/vidéos.

**Point de terminaison :** `POST /create`

**Format de requête :** `multipart/form-data`

**Champs du formulaire :**
- `account` (chaîne, requis) : Nom du compte
- `task_type` (entier, requis) : Type de tâche
  - `1` : Générer un modèle 3D à partir d'une photo (照片生3d模型)
  - `2` : Générer un modèle 3D à partir d'une vidéo (视频生3d模型)
  - `3` : Générer une peinture à partir d'une photo (照片生画)
- `data` (chaîne JSON, requis) : Configuration de la tâche
- `file` (fichier, requis) : Fichier d'entrée (image ou vidéo)

**Structure JSON des données :**
```json
{
    "name": "item name",
    "dimension": {
        "unit": "cm",  // "m", "inch", etc.
        "length": 100,  // Pour task_type=3 : correspond à "宽度 x" (largeur)
        "width": 200,   // Pour task_type=3 : correspond à "高度 y" (hauteur)
        "height": 1.0   // Pour task_type=3 : correspond à "厚度" (épaisseur)
    }
}
```

**Note :** Pour `task_type=3`, il n'y a pas de cadre par défaut.

**Exigences de fichier :**
- Pour `task_type=1` ou `3` : Fichier image (jpg, png, etc.)
- Pour `task_type=2` : Fichier vidéo (mp4, mov, etc.)

**Exemple de requête :**
```http
POST /create HTTP/1.1
Host: xxxxxx
x-api-key: <your-api-key>
Content-Type: multipart/form-data; boundary=<boundary-string>

------<boundary-string>
Content-Disposition: form-data; name="account"

example-account-name
------<boundary-string>
Content-Disposition: form-data; name="task_type"

1
------<boundary-string>
Content-Disposition: form-data; name="data"

{
    "name": "item name",
    "dimension": {
        "unit": "cm",
        "length": 100,
        "width": 200,
        "height": 50
    }
}
------<boundary-string>
Content-Disposition: form-data; name="file"; filename="input.jpg"
Content-Type: image/jpeg

<binary file data>
------<boundary-string>--
```

**Note :** La chaîne de délimitation est générée automatiquement par les clients HTTP. Toute chaîne unique peut être utilisée.

**Réponse de succès (200 OK) :**
```json
{
    "status": "success",
    "task_id": "task-id-12345"
}
```

**Réponse non authentifiée (401 Unauthorized) :**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Réponse d'erreur (400/404/500) :**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 5. Interroger le statut de la tâche

Vérifier le statut d'une tâche soumise.

**Point de terminaison :** `POST /task-status`

**Corps de la requête :**
```json
{
    "account": "example-account-name",
    "task_id": "task-id-12345"
}
```

**Exemple de requête :**
```http
POST /task-status HTTP/1.1
Host: xxxxxx
x-api-key: <your-api-key>
Content-Type: application/json
Content-Length: <length>

{
    "account": "example-account-name",
    "task_id": "task-id-12345"
}
```

**Réponse de succès (200 OK) - Tâche terminée :**
```json
{
    "status": "success",
    "task_status": "completed",
    "item_id": "item-id-56789"
}
```

**Réponse de succès (200 OK) - Tâche en cours :**
```json
{
    "status": "success",
    "task_status": "in_progress"
}
```

**Réponse de succès (200 OK) - Tâche échouée :**
```json
{
    "status": "success",
    "task_status": "failed",
    "message": "<error description>"
}
```

**Réponse non authentifiée (401 Unauthorized) :**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Réponse d'erreur (400/404/500) :**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

#### Valeurs de statut de tâche

- `completed` : La tâche s'est terminée avec succès. Retourne `item_id` de l'élément créé.
- `in_progress` : La tâche est toujours en cours de traitement.
- `failed` : La tâche a échoué. Retourne le message d'erreur dans le champ `message`.

---

### 6. Supprimer des éléments

Supprimer plusieurs éléments d'un compte. Chaque élément nécessite à la fois `item_id` et `item_name` pour vérification afin d'éviter les suppressions accidentelles.

**Point de terminaison :** `POST /delete-items`

**Corps de la requête :**
```json
{
    "account": "example-account-name",
    "items": [
        {
            "item_id": "item-id-1",
            "item_name": "item name 1"
        },
        {
            "item_id": "item-id-2",
            "item_name": "item name 2"
        }
    ]
}
```

**Exemple de requête :**
```http
POST /delete-items HTTP/1.1
Host: xxxxxx
x-api-key: <your-api-key>
Content-Type: application/json
Content-Length: <length>

{
    "account": "example-account-name",
    "items": [
        {
            "item_id": "item-id-1",
            "item_name": "item name 1"
        },
        {
            "item_id": "item-id-2",
            "item_name": "item name 2"
        }
    ]
}
```

**Réponse de succès (200 OK) :**
```json
{
    "status": "success",
    "received_count": 2,
    "successful_count": 1,
    "failed_count": 1,
    "failed_items": [
        {
            "item_id": "item-id-2",
            "item_name": "item name 2",
            "reason": "Item not found or name mismatch"
        }
    ]
}
```

**Note :** La réponse inclut :
- `received_count` : Nombre d'éléments reçus dans la requête
- `successful_count` : Nombre d'éléments supprimés avec succès
- `failed_count` : Nombre d'éléments qui ont échoué à supprimer
- `failed_items` : Tableau des éléments qui ont échoué à supprimer, avec les raisons

**Réponse non authentifiée (401 Unauthorized) :**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Réponse d'erreur (400/404/500) :**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

## Gestion des erreurs

Tous les points de terminaison suivent un format de réponse d'erreur cohérent :

```json
{
    "status": "error",
    "message": "<error description>"
}
```

Codes de statut HTTP courants :
- `400 Bad Request` : Paramètres ou format de requête invalide
- `401 Unauthorized` : Clé API manquante ou invalide
- `404 Not Found` : Ressource non trouvée (compte, élément, tâche)
- `500 Internal Server Error` : Erreur côté serveur






