# GeRoom API Documentation (RESTful v2.0)

## Overview

The GeRoom API allows for seamless integration of 3D model generation and account management.

* **Authentication**: All endpoints identify the user via the `x-api-key`.
* **Statelessness**: You no longer need to provide an `account` name in the request; the system resolves your identity from the header.

## Authentication

All requests must include the following header:

```http
x-api-key: <your-api-key>

```

## Base URL

```
https://open.ge-room.com

```

---

## API Endpoints

### 1. Get Account Information

Retrieve credits and usage statistics for the account associated with the API key.

**Endpoint:** `GET /account`

**Request Example:**

```http
GET /account HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**Success Response (200 OK):**

```json
{
    "status": "success",
    "data": {
        "account_name": "example-user-123",
        "credits": 1000,
        "total_items": 42
    }
}

```

---

### 2. Get All Items

Retrieve a list of all items (basic info) owned by the authenticated account.

**Endpoint:** `GET /items`

**Request Example:**

```http
GET /items HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**Success Response (200 OK):**

```json
{
    "status": "success",
    "account_name": "example-user-123",
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
        }
    ]
}

```

---

### 3. Get Item Details Complete Information

Retrieve complete information for a specific item, including all download URLs.

**Endpoint:** `GET /items/{item_id}`

**Request Example:**

```http
GET /items/item-id-1 HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**Success Response (200 OK):**

```json
{
    "status": "success",
    "account_name": "example-user-123",
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

---

### 4. Create Task

Submit an image or video to generate 3D models or paintings.

**Endpoint:** `POST /tasks`

**Content-Type:** `multipart/form-data`

**Form Fields:**

- `task_type` (integer, required):
  - `1`: Photo to 3D
  - `2`: Video to 3D
  - `3`: Photo to Painting

- `data` (JSON string, required):
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

* `file` (file, required): The source image or video.

**Note:** For `task_type=3`, there is no frame by default.

**File Requirements:**
- For `task_type=1` : Image file (jpg, png, etc.)
- For `task_type=3` : Image file (.png only)
- For `task_type=2` : Video file (mp4, mov, etc.)

**Request Example:**

```http
POST /tasks HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="task_type"

1
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="data"

{"name": "item name", "dimension": {"unit": "cm", "length": 100, "width": 200, "height": 50}}
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

<binary file data>
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

**Success Response (201 Created):**

```json
{
    "status": "success",
    "account_name": "example-user-123",
    "task_id": "task-id-12345"
}

```

---

### 5. Get Task Status

Check the status of a specific background task.

**Endpoint:** `GET /tasks/{task_id}`

**Request Example:**

```http
GET /tasks/task-id-12345 HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**Success Response (200 OK) - Task Completed:**

```json
{
    "status": "success",
    "account_name": "example-user-123",
    "task_status": "completed", // values: in_progress, completed, failed
    "item_id": "item-id-56789",
    "message": "" // <possible error description> 
}
```

**Task Status Values**

* `completed`: Task has finished successfully. Returns `item_id` of the created item.
* `in_progress`: Task is still being processed.
* `failed`: Task has failed. Returns error message in `message` field.

---

### 6. Bulk Delete Items

Delete multiple items. Requires both ID and Name to verify intent.

**Endpoint:** `DELETE /items`

**Request Body:**

```json
{
    "items": [
        { "item_id": "item-id-1", "item_name": "item name 1" },
        { "item_id": "item-id-2", "item_name": "item name 2" }
    ]
}

```

**Request Example:**

```http
DELETE /items HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
Content-Type: application/json

{
    "items": [
        { "item_id": "item-id-1", "item_name": "item name 1" },
        { "item_id": "item-id-2", "item_name": "item name 2" }
    ]
}
```

**Success Response (200 OK):**

```json
{
    "status": "success",
    "account_name": "example-user-123",
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

---

## Error Handling

All errors return a standard JSON object.

| Status Code | Meaning |
| --- | --- |
| **400** | Bad Request (missing files or malformed JSON) |
| **401** | Unauthorized (invalid or missing `x-api-key`) |
| **404** | Not Found (item or task does not exist) |
| **500** | Internal Server Error |

**Error Format:**

```json
{
    "status": "error",
    "message": "Specific error description here"
}

```
