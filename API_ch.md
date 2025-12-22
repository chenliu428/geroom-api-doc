# GeRoom API 文档

## 概述

所有 API 端点都需要使用 `x-api-key` 请求头和账户信息进行身份验证。应提供一个 API 管理界面用于：
- 创建 API 密钥
- 删除 API 密钥
- 密钥数目：30
- 过期： 120天

## 身份验证

所有端点都需要 `x-api-key` 请求头进行身份验证：

```
x-api-key: <your-api-key>
```

## 基础 URL

```
Host: https://open.ge-room.com/
```

---

## API 端点

### 1. 获取账户信息

获取账户余额点数和商品总数。

**端点：** `GET /account-info`

**查询参数：**
- `account` (字符串, 必需): 账户名称

**请求示例：**
```http
GET /account-info?account=example-account-name HTTP/1.1
Host: xxxxx
x-api-key: <your-api-key>
```

**响应状态码：**
- `200 OK` - 成功
- `401 Unauthorized` - 未认证（无效或缺少 API 密钥）
- `400 Bad Request` - 无效请求（缺少账户字段，格式无效）
- `404 Not Found` - 账户未找到
- `500 Internal Server Error` - 服务器错误

**成功响应 (200 OK)：**
```json
{
    "status": "success",
    "data": {
        "credits": 1000,
        "total_items": 42
    }
}
```

**未认证响应 (401 Unauthorized)：**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**错误响应 (400/404/500)：**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 2. 获取所有商品列表

获取账户下所有商品的列表（仅基本信息）。

**端点：** `GET /items`

**查询参数：**
- `account` (字符串, 必需): 账户名称

**请求示例：**
```http
GET /items?account=example-account-name HTTP/1.1
Host: xxxxx
x-api-key: <your-api-key>
```

**成功响应 (200 OK)：**
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

**注意：** 数据数组包含完整的商品信息，对应数据库的完整一行。

**未认证响应 (401 Unauthorized)：**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**错误响应 (400/404/500)：**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 3. 获取单个商品完整信息

获取特定商品的完整信息，包括所有下载 URL。

**端点：** `POST /item`

**请求体：**
```json
{
    "account": "example-account-name",
    "item_id": "item-id-1"
}
```

**请求示例：**
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

**成功响应 (200 OK)：**
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

**注意：** 数据对象包含完整的商品信息，对应数据库的完整一行。

**未认证响应 (401 Unauthorized)：**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**错误响应 (400/404/500)：**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 4. 创建任务

提交任务以从图片/视频生成 3D 模型或画作。

**端点：** `POST /create`

**请求格式：** `multipart/form-data`

**表单字段：**
- `account` (字符串, 必需): 账户名称
- `task_type` (整数, 必需): 任务类型
  - `1`: 照片生成 3D 模型（照片生3d模型）
  - `2`: 视频生成 3D 模型（视频生3d模型）
  - `3`: 照片生成画作（照片生画）
- `data` (JSON 字符串, 必需): 任务配置
- `file` (文件, 必需): 输入文件（图片或视频）

**数据 JSON 结构：**
```json
{
    "name": "item name",
    "dimension": {
        "unit": "cm",  // "m", "inch" 等
        "length": 100,  // 当 task_type=3 时对应 "宽度 x" (width)
        "width": 200,   // 当 task_type=3 时对应 "高度 y" (height)
        "height": 50   // 当 task_type=3 时对应 "厚度" (thickness)
    }
}
```

**注意：** 对于 `task_type=3`，默认没有画框。

**文件要求：**
- 对于 `task_type=1` 或 `3`：图片文件（jpg, png 等）
- 对于 `task_type=2`：视频文件（mp4, mov 等）

**请求示例：**
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

**注意：** 边界字符串由 HTTP 客户端自动生成。可以使用任何唯一字符串。

**成功响应 (200 OK)：**
```json
{
    "status": "success",
    "task_id": "task-id-12345"
}
```

**未认证响应 (401 Unauthorized)：**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**错误响应 (400/404/500)：**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

### 5. 查询任务状态

检查已提交任务的状态。

**端点：** `POST /task-status`

**请求体：**
```json
{
    "account": "example-account-name",
    "task_id": "task-id-12345"
}
```

**请求示例：**
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

**成功响应 (200 OK) - 任务完成：**
```json
{
    "status": "success",
    "task_status": "completed",
    "item_id": "item-id-56789"
}
```

**成功响应 (200 OK) - 任务进行中：**
```json
{
    "status": "success",
    "task_status": "in_progress"
}
```

**成功响应 (200 OK) - 任务失败：**
```json
{
    "status": "success",
    "task_status": "failed",
    "message": "<error description>"
}
```

**未认证响应 (401 Unauthorized)：**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**错误响应 (400/404/500)：**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

#### 任务状态值

- `completed`: 任务已成功完成。返回所创建商品的 `item_id`。
- `in_progress`: 任务仍在处理中。
- `failed`: 任务已失败。在 `message` 字段中返回错误消息。

---

### 6. 删除商品

删除账户下的多个商品。每个商品需要同时提供 `item_id` 和 `item_name` 进行验证，以防止意外删除。

**端点：** `POST /delete-items`

**请求体：**
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

**请求示例：**
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

**成功响应 (200 OK)：**
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

**注意：** 响应包括：
- `received_count`: 请求中接收到的商品数量
- `successful_count`: 成功删除的商品数量
- `failed_count`: 删除失败的商品数量
- `failed_items`: 删除失败的商品数组，包含失败原因

**未认证响应 (401 Unauthorized)：**
```json
{
    "status": "unauthenticated",
    "message": "Invalid or missing API key"
}
```

**错误响应 (400/404/500)：**
```json
{
    "status": "error",
    "message": "<error description>"
}
```

---

## 错误处理

所有端点都遵循一致的错误响应格式：

```json
{
    "status": "error",
    "message": "<error description>"
}
```

常见的 HTTP 状态码：
- `400 Bad Request`: 无效的请求参数或格式
- `401 Unauthorized`: 缺少或无效的 API 密钥
- `404 Not Found`: 资源未找到（账户、商品、任务）
- `500 Internal Server Error`: 服务器端错误
