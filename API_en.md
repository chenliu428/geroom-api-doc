# GeRoom API Documentation

## Overview

All API endpoints require authentication using the `x-api-key` header and account information. An API management UI should be provided for:
- Creating API keys
- Deleting API keys
- Max number of API keys: 30
- Expiration: 120 days

## Authentication

All endpoints require the `x-api-key` header for authentication:

```
x-api-key: <your-api-key>
```

## Base URL

```
Host: https://open.ge-room.com/
```

---

## API Endpoints

### 1. Get Account Information

Retrieve account credits and total number of items.

**Endpoint:** `GET /account-info`

**Query Parameters:**
- `account` (string, required): Account name

**Request Example:**
```http
GET /account-info?account=example-account-name HTTP/1.1
Host: xxxxx
x-api-key: <your-api-key>
```

**Response Status Codes:**
- `200 OK` - Success
- `401 Unauthorized` - Unauthenticated (invalid or missing API key)
- `400 Bad Request` - Invalid request (missing account field, invalid format)
- `404 Not Found` - Account not found
- `500 Internal Server Error` - Server error

**Success Response (200 OK):**
```json
{
    "status": "success",
    "data": {
        "credits": 1000,
        "total_items": 42
    }
}
```

**Unauthenticated Response (401 Unauthorized):**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Error Response (400/404/500):**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 2. Get All Items

Retrieve a list of all items under an account (only basic information).

**Endpoint:** `GET /items`

**Query Parameters:**
- `account` (string, required): Account name

**Request Example:**
```http
GET /items?account=example-account-name HTTP/1.1
Host: xxxxx
x-api-key: <your-api-key>
```

**Success Response (200 OK):**
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

**Unauthenticated Response (401 Unauthorized):**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Error Response (400/404/500):**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 3. Get Single Item Complete Information

Retrieve complete information for a specific item, including all download URLs.

**Endpoint:** `POST /item`

**Request Body:**
```json
{
    "account": "example-account-name",
    "item_id": "item-id-1"
}
```

**Request Example:**
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

**Success Response (200 OK):**
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

**Unauthenticated Response (401 Unauthorized):**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Error Response (400/404/500):**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 4. Create Task

Submit a task to generate 3D models or paintings from images/videos.

**Endpoint:** `POST /create`

**Request Format:** `multipart/form-data`

**Form Fields:**
- `account` (string, required): Account name
- `task_type` (integer, required): Task type
  - `1`: Generate 3D model from photo (照片生3d模型)
  - `2`: Generate 3D model from video (视频生3d模型)
  - `3`: Generate painting from photo (照片生画)
- `data` (JSON string, required): Task configuration
- `file` (file, required): Input file (image or video)

**Data JSON Structure:**
```json
{
    "name": "item name",
    "dimension": {
        "unit": "cm",  // "m", "inch", etc.
        "length": 100,  // For task_type=3: corresponds to "宽度 x" (width)
        "width": 200,   // For task_type=3: corresponds to "高度 y" (height)
        "height": 1.0   // For task_type=3: corresponds to "厚度" (thickness)
    }
}
```

**Note:** For `task_type=3`, there is no frame by default.

**File Requirements:**
- For `task_type=1` or `3`: Image file (jpg, png, etc.)
- For `task_type=2`: Video file (mp4, mov, etc.)

**Request Example:**
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
        "height ": 50
    }
}
------<boundary-string>
Content-Disposition: form-data; name="file"; filename="input.jpg"
Content-Type: image/jpeg

<binary file data>
------<boundary-string>--
```

**Note:** The boundary string is automatically generated by HTTP clients. Any unique string can be used.

**Success Response (200 OK):**
```json
{
    "status": "success",
    "task_id": "task-id-12345"
}
```

**Unauthenticated Response (401 Unauthorized):**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Error Response (400/404/500):**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 5. Query Task Status

Check the status of a submitted task.

**Endpoint:** `POST /task-status`

**Request Body:**
```json
{
    "account": "example-account-name",
    "task_id": "task-id-12345"
}
```

**Request Example:**
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

**Success Response (200 OK) - Task Completed:**
```json
{
    "status": "success",
    "task_status": "completed",
    "item_id": "item-id-56789"
}
```

**Success Response (200 OK) - Task In Progress:**
```json
{
    "status": "success",
    "task_status": "in_progress"
}
```

**Success Response (200 OK) - Task Failed:**
```json
{
    "status": "success",
    "task_status": "failed",
    "message": "<error description>"
}
```

**Unauthenticated Response (401 Unauthorized):**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Error Response (400/404/500):**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

#### Task Status Values

- `completed`: Task has finished successfully. Returns `item_id` of the created item.
- `in_progress`: Task is still being processed.
- `failed`: Task has failed. Returns error message in `message` field.

---

### 6. Delete Items

Delete multiple items from an account. Each item requires both `item_id` and `item_name` for verification to prevent accidental deletions.

**Endpoint:** `POST /delete-items`

**Request Body:**
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

**Request Example:**
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

**Success Response (200 OK):**
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

**Note:** The response includes:
- `received_count`: Number of items received in the request
- `successful_count`: Number of items successfully deleted
- `failed_count`: Number of items that failed to delete
- `failed_items`: Array of items that failed to delete, with reasons

**Unauthenticated Response (401 Unauthorized):**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**Error Response (400/404/500):**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

## Error Handling

All endpoints follow a consistent error response format:

```json
{
    "status": "error",
    "message": "<error description>"
}
```

Common HTTP status codes:
- `400 Bad Request`: Invalid request parameters or format
- `401 Unauthorized`: Missing or invalid API key
- `404 Not Found`: Resource not found (account, item, task)
- `500 Internal Server Error`: Server-side error