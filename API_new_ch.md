# GeRoom API 文档（RESTful v2.0）

## 概述

GeRoom API 支持 3D 模型生成与账户管理的无缝集成。

* **认证**：所有接口通过请求头中的 `x-api-key` 识别用户身份。
* **无状态**：请求中无需再传入 `account` 名称，系统会根据请求头解析您的身份。

## 认证

所有请求必须包含以下请求头：

```http
x-api-key: <your-api-key>

```

## 基础地址

```
https://open.ge-room.com

```

---

## API 接口

### 1. 获取账户信息

获取与 API Key 关联账户的积分及使用统计。

**接口：** `GET /account`

**请求示例：**

```http
GET /account HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**成功响应（200 OK）：**

```json
{
    "data": {
        "account_name": "example-user-123",
        "credits": 1000,
        "total_items": 42
    }
}

```

---

### 2. 获取全部物品

获取当前认证账户下物品列表（基础信息），结果分页返回。

**接口：** `GET /items`

**查询参数：**

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `page` | 整数 | 1 | 页码（从 1 开始）。 |
| `per_page` | 整数 | 10 | 每页条数（如 10）。 |

**请求示例：**

```http
GET /items?page=1&per_page=10 HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**成功响应（200 OK）– 分页：**

```json
{
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
            "img_link": "https://example.com/image-link",
            "created_at": "2025-02-08T10:30:00Z"
        }
    ],
    "pagination": {
        "page": 1,
        "per_page": 10,
        "total": 42,
        "total_pages": 5
    }
}
```

当物品数量较多时，响应始终为分页形式，可通过 `page` 与 `per_page` 翻页。

---

### 3. 获取物品详情（完整信息）

获取指定物品的完整信息，包括所有下载链接。

**接口：** `GET /items/{item_id}`

**请求示例：**

```http
GET /items/item-id-1 HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**成功响应（200 OK）：**

```json
{
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
        "usdz_url": "https://example.com/download/usdz",
        "created_at": "2025-02-08T10:30:00Z"
    }
}

```

---

### 4. 创建任务

提交图片或视频，用于生成 3D 模型或画作。

**接口：** `POST /tasks`

**Content-Type：** `multipart/form-data`

**表单字段：**

- `task_type`（整数，必填）：
  - `1`：照片转 3D
  - `2`：视频转 3D
  - `3`：照片转画作

- `data`（JSON 字符串，必填）：
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

* `file`（文件，必填）：源图片或视频文件。

**说明：** `task_type=3` 时默认无画框。

**文件要求：**
- `task_type=1`：图片文件（jpg、png 等）
- `task_type=3`：图片文件（仅 .png）
- `task_type=2`：视频文件（mp4、mov 等）

**请求示例：**

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

**成功响应（201 Created）：**

```json
{
    "account_name": "example-user-123",
    "task_id": "task-id-12345"
}

```

---

### 5. 查询任务状态

查询指定后台任务的执行状态。

**接口：** `GET /tasks/{task_id}`

**请求示例：**

```http
GET /tasks/task-id-12345 HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**成功响应（200 OK）- 任务已完成：**

```json
{
    "account_name": "example-user-123",
    "task_status": "completed",
    "item_id": "item-id-56789",
    "created_at": "2025-02-08T10:30:00Z",
    "message": ""
}
```

* `task_status` 取值：`in_progress`、`completed`、`failed`。`message` 在需要时包含错误描述。

**任务状态说明**

* `completed`：任务已成功完成，返回所创建物品的 `item_id`。
* `in_progress`：任务仍在处理中。
* `failed`：任务失败，错误信息在 `message` 字段中返回。

---

### 6. 删除物品

按 ID 删除单个物品。

**接口：** `DELETE /items/{item_id}`

**请求示例：**

```http
DELETE /items/item-id-1 HTTP/1.1
Host: open.ge-room.com
x-api-key: <your-api-key>
```

**成功响应（200 OK）：**

响应体为空，或：

```json
{}
```

---

### 7. 批量删除物品

批量删除多个物品。需同时提供 ID 与名称以确认操作意图。

**接口：** `DELETE /items`

**请求体：**

```json
{
    "items": [
        { "item_id": "item-id-1", "item_name": "item name 1" },
        { "item_id": "item-id-2", "item_name": "item name 2" }
    ]
}
```

**请求示例：**

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

**成功响应（200 OK）：**

```json
{
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

**说明：** 响应中包含：

- `received_count`：本次请求收到的物品数量
- `successful_count`：成功删除的物品数量
- `failed_count`：删除失败的物品数量
- `failed_items`：删除失败的物品列表及失败原因

---

## 错误处理

所有错误均返回统一的 JSON 结构。

| 状态码 | 含义 |
| --- | --- |
| **400** | 请求错误（缺少文件或 JSON 格式错误） |
| **401** | 未授权（`x-api-key` 无效或缺失） |
| **404** | 未找到（物品或任务不存在） |
| **500** | 服务器内部错误 |

**错误响应格式：**

HTTP 状态码表示错误类型（如 4xx、5xx）。响应体可能为：

```json
{
    "message": "Specific error description here"
}
```
