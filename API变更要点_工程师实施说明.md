# GeRoom API 变更要点 — 工程师实施说明

本文档对比 **旧版 API（API_ch.md）** 与 **新版 API（API_new_ch.md，RESTful v2.0）**，列出需由工程师完成的修改要点及具体实施方式。实施时**以新 API 文档为准**。

---

## 一、核心变更总览

| 维度 | 旧版 | 新版（实施依据） |
|------|------|------------------|
| 身份识别 | 每个请求必须带 `x-api-key` **且** 传 `account`（查询参数或请求体） | 仅需 `x-api-key`，**不再传** `account`，账户由服务端根据 Key 解析 |
| 风格 | 路径与动词混用（如 POST /item、POST /task-status） | 统一 RESTful：资源路径 + 标准 HTTP 方法（GET/POST/DELETE） |
| 成功响应 | 多数带 `"status": "success"`，部分不返回 account | 新版成功响应**不再统一包含** `status` 字段，多数增加 `account_name` |
| 错误格式 | 401 可能返回 `status: "unauthenticated"` | 错误以 **HTTP 状态码** 为准，响应体**可能仅含** `message`（不强制含 `status`） |
| 删除能力 | 仅支持批量删除（需 item_id + item_name） | 新增**单条删除** `DELETE /items/{item_id}`；批量删除保留 |

---

## 二、按接口的修改要点与实施方式

### 1. 获取账户信息

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `GET /account-info` | `GET /account` |
| 请求 | 必须带查询参数 `account=xxx` | **不传**任何查询参数，仅需 Header `x-api-key` |
| 成功响应 | `status` + `data`（credits, total_items） | **无** 顶层 `status`，仅 `data`，且 `data` 内含 `account_name`、`credits`、`total_items` |

**实施要点：**

- 将请求从 `GET /account-info?account=xxx` 改为 `GET /account`，去掉所有 `account` 查询参数。
- 客户端解析响应时：从 `data.account_name` 取当前账户，从 `data.credits`、`data.total_items` 取积分与总数；**不要依赖** 顶层 `status` 判断成功（以 HTTP 200 为准）。

---

### 2. 获取全部物品

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `GET /items` | `GET /items`（不变） |
| 请求 | 查询参数 `account=xxx` | **不传** `account`；新增**分页**：`page`（默认 1）、`per_page`（默认 10） |
| 成功响应 | 无分页，无 `account_name`，无 `created_at` | 顶层 `account_name`；`data` 中每项增加 `created_at`；**无** 顶层 `status`；新增 **`pagination`** |

**实施要点：**

- 请求从 `GET /items?account=xxx` 改为 `GET /items`，移除 `account`；按需传 `page`、`per_page`，例如 `GET /items?page=1&per_page=10`。
- 响应结构：`account_name`、`data`（数组，每项含 id, name, dimensions, img_link, **created_at**）、**pagination**（`page`, `per_page`, `total`, `total_pages`）。
- 客户端需实现分页逻辑（翻页、总数展示等），并兼容 `data` 中的 `created_at`；成功以 HTTP 200 为准，不依赖 `status`。

---

### 3. 获取物品详情（单条完整信息）

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /item` | `GET /items/{item_id}` |
| 请求 | Body: `{"account":"xxx","item_id":"item-id-1"}` | **无 Body**，item_id 放在路径中，如 `GET /items/item-id-1` |
| 成功响应 | 有 `status`；data 无 `created_at` | **无** 顶层 `status`；`data` 增加 **created_at**；顶层有 `account_name` |

**实施要点：**

- 将“根据 item_id 查详情”从 **POST /item + JSON body** 改为 **GET /items/{item_id}**，删除 Body 中的 `account` 与 `item_id`。
- 响应：顶层 `account_name` + `data`（id, name, dimensions, img_link, ar_link, fbx_url, glb_url, usdz_url, **created_at**）；不以 `status` 判断成功。

---

### 4. 创建任务

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /create` | `POST /tasks` |
| 请求 | multipart/form-data，含字段 `account` | **不再传** `account`，仅传 `task_type`、`data`、`file` |
| 成功状态码 | 200 OK | **201 Created** |
| 成功响应 | `status` + `task_id` | **无** `status`，仅 `account_name` + `task_id` |
| 文件类型 | task_type=1: .webp, .jpeg, .png | task_type=1: jpg、png 等；task_type=3: **仅 .png** |

**实施要点：**

- 路径从 `POST /create` 改为 `POST /tasks`；表单移除 `account`，只保留 `task_type`、`data`（JSON 字符串）、`file`。
- 以 **HTTP 201** 判断创建成功；响应仅含 `account_name`、`task_id`，不要依赖 `status`。
- 文件类型：task_type=1 支持 jpg/png 等，task_type=3 仅 .png。

---

### 5. 查询任务状态

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /task-status` | `GET /tasks/{task_id}` |
| 请求 | Body: `{"account":"xxx","task_id":"task-id-12345"}` | **无 Body**，task_id 在路径中，如 `GET /tasks/task-id-12345` |
| 成功响应 | 有 `status`；无 `account_name`、无 `created_at` | **无** 顶层 `status`；增加 `account_name`、**created_at**；仍含 `task_status`、`item_id`（完成时）、`message` |

**实施要点：**

- 从 **POST /task-status + JSON body** 改为 **GET /tasks/{task_id}**，不再传 `account` 与 Body。
- 响应字段：`account_name`、`task_status`、`item_id`、**created_at**、`message`；成功以 HTTP 200 为准，不依赖 `status`。

---

### 6. 删除物品（单条）— 新增

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | **无此接口**（仅支持批量删除） | **DELETE /items/{item_id}** |
| 请求 | — | 无 Body，仅路径中的 item_id + Header `x-api-key` |
| 成功响应 | — | 响应体为空或 `{}`，HTTP 200 |

**实施要点：**

- 新版支持**按 ID 单条删除**：调用 `DELETE /items/{item_id}`，无需传 item_name 或 account。
- 客户端若此前只能“批量删”，可新增单删能力，简化交互；成功以 200 与空体为准。

---

### 7. 批量删除物品

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /delete-items` | `DELETE /items` |
| 请求 | Body 含 `account` + `items` 数组 | Body **不含** `account`，仅 `items` 数组（每项仍为 item_id + item_name） |
| 成功响应 | 有 `status`；无 `account_name` | **无** 顶层 `status`；增加 `account_name`；其余同上（received_count, successful_count, failed_count, failed_items） |

**实施要点：**

- 路径从 `POST /delete-items` 改为 **DELETE /items**，方法改为 DELETE；请求体从 `{"account":"xxx", "items":[...]}` 改为 `{"items":[...]}`。
- 响应含 `account_name`、`received_count`、`successful_count`、`failed_count`、`failed_items`；成功以 HTTP 200 为准。

---

## 三、错误处理与状态码

- **错误判断**：以 **HTTP 状态码**（4xx/5xx）为准，不依赖响应体中的 `status` 字段。
- **错误响应体**：新文档约定为“可能为” `{ "message": "具体错误描述" }`，即可能**只有** `message`，不强制出现 `status: "error"`。客户端应兼容：有 `message` 则展示，无则按状态码处理。
- 旧版 401 可能返回 `status: "unauthenticated"`，新版不再使用该字段，统一通过 401 状态码识别未授权。
- 状态码含义：400 请求错误、401 未授权、404 未找到、500 服务器错误；**创建任务成功为 201**。

---

## 四、实施检查清单（按新 API 文档执行）

- [ ] **认证**：所有请求只带 `x-api-key`，不再在任何接口传 `account`（查询参数或 Body）。
- [ ] **1. 账户信息**：`GET /account`，无查询参数；解析 `data.account_name`、`data.credits`、`data.total_items`；不依赖 `status`。
- [ ] **2. 全部物品**：`GET /items`，无 account；支持 `page`、`per_page` 分页；解析 `account_name`、`data`、`pagination` 及每项 `created_at`；不依赖 `status`。
- [ ] **3. 物品详情**：`GET /items/{item_id}`，无 Body；解析 `account_name`、`data`（含 `created_at`）；不依赖 `status`。
- [ ] **4. 创建任务**：`POST /tasks`，表单无 account；以 **201** 判断成功；解析 `account_name`、`task_id`；task_type=3 仅 .png。
- [ ] **5. 任务状态**：`GET /tasks/{task_id}`，无 Body；解析 `account_name`、`task_status`、`item_id`、`created_at`、`message`；不依赖 `status`。
- [ ] **6. 单条删除**：新增 `DELETE /items/{item_id}`，无 Body，成功为 200 + 空体或 `{}`。
- [ ] **7. 批量删除**：`DELETE /items`，Body 仅 `items`；解析 `account_name` 及删除结果字段；不依赖 `status`。
- [ ] **错误处理**：以 HTTP 状态码判断错误，错误体按 `message` 兼容；不再依赖 `status: "unauthenticated"` 或 `status: "error"`。

---

## 五、简要路径与参数对照表

| 功能 | 旧版 | 新版 |
|------|------|------|
| 账户信息 | GET /account-info?account= | GET /account |
| 全部物品 | GET /items?account= | GET /items?page=1&per_page=10（可选分页） |
| 物品详情 | POST /item Body{account, item_id} | GET /items/{item_id} |
| 创建任务 | POST /create Form+account | POST /tasks Form 无 account |
| 任务状态 | POST /task-status Body{account, task_id} | GET /tasks/{task_id} |
| 单条删除 | （无） | **DELETE /items/{item_id}** |
| 批量删除 | POST /delete-items Body{account, items} | DELETE /items Body{items} |

实施时以 **API_new_ch.md（RESTful v2.0）** 为准；若有未列出的字段或边界情况，以新文档描述与示例为准。
