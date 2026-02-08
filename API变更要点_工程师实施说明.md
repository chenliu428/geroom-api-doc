# GeRoom API 变更要点 — 工程师实施说明

本文档对比 **旧版 API（API_ch.md）** 与 **新版 API（API_new_ch.md，RESTful v2.0）**，列出需由工程师完成的修改要点及具体实施方式。实施时**以新 API 文档为准**。

---

## 一、核心变更总览

| 维度 | 旧版 | 新版（实施依据） |
|------|------|------------------|
| 身份识别 | 每个请求必须带 `x-api-key` **且** 传 `account`（查询参数或请求体） | 仅需 `x-api-key`，**不再传** `account`，账户由服务端根据 Key 解析 |
| 风格 | 路径与动词混用（如 POST /item、POST /task-status） | 统一 RESTful：资源路径 + 标准 HTTP 方法（GET/POST/DELETE） |
| 错误格式 | 401 可能返回 `status: "unauthenticated"` | 所有错误统一为 `status: "error"` + `message` |
| 成功响应 | 部分接口不返回 account 信息 | 多数成功响应增加 `account_name` 便于客户端核对 |

---

## 二、按接口的修改要点与实施方式

### 1. 获取账户信息

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `GET /account-info` | `GET /account` |
| 请求 | 必须带查询参数 `account=xxx` | **不传**任何查询参数，仅需 Header `x-api-key` |
| 成功响应 | `data` 仅含 `credits`, `total_items` | `data` 增加 `account_name`（如 `"example-user-123"`） |

**实施要点：**

- 将请求从 `GET /account-info?account=xxx` 改为 `GET /account`，去掉所有 `account` 查询参数。
- 客户端解析响应时，若需展示“当前账户”，从 `data.account_name` 读取。

---

### 2. 获取全部物品

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `GET /items` | `GET /items`（不变） |
| 请求 | 必须带查询参数 `account=xxx` | **不传** `account`，仅需 `x-api-key` |
| 成功响应 | 无 `account_name` | 响应顶层增加 `account_name` |

**实施要点：**

- 请求从 `GET /items?account=xxx` 改为 `GET /items`，移除 `account` 查询参数。
- 若前端/客户端有依赖“当前账户名”的逻辑，改为使用响应中的 `account_name`。

---

### 3. 获取物品详情（单条完整信息）

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /item` | `GET /items/{item_id}` |
| 请求 | Body: `{"account":"xxx","item_id":"item-id-1"}` | **无 Body**，item_id 放在路径中，如 `GET /items/item-id-1` |
| 身份 | 依赖 Body 中的 `account` | 仅依赖 Header `x-api-key` |
| 成功响应 | 无 `account_name` | 响应顶层增加 `account_name` |

**实施要点：**

- 将“根据 item_id 查详情”从 **POST /item + JSON body** 改为 **GET /items/{item_id}**。
- 删除请求体中的 `account` 与 `item_id`，仅保留路径中的 `item_id`。
- 响应结构除增加 `account_name` 外，`data` 内字段与旧版一致（id, name, dimensions, img_link, ar_link, fbx_url, glb_url, usdz_url），按新文档字段名为准即可。

---

### 4. 创建任务

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /create` | `POST /tasks` |
| 请求 | multipart/form-data，含字段 `account` | **不再传** `account`，仅传 `task_type`、`data`、`file` |
| 成功响应 | 仅 `status`, `task_id` | 增加 `account_name` |
| 文件类型 | task_type=1: .webp, .jpeg, .png | task_type=1: jpg、png 等；task_type=3: **仅 .png** |

**实施要点：**

- 路径从 `POST /create` 改为 `POST /tasks`。
- 从表单中移除 `account` 字段，只保留 `task_type`、`data`（JSON 字符串）、`file`。
- 服务端成功创建任务后返回 **HTTP 201**，客户端应按 201 判断“创建成功”，并可从响应中读取 `account_name`、`task_id`。
- 若服务端有文件类型校验，按新文档：task_type=1 支持 jpg/png 等，task_type=3 仅 .png。

---

### 5. 查询任务状态

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /task-status` | `GET /tasks/{task_id}` |
| 请求 | Body: `{"account":"xxx","task_id":"task-id-12345"}` | **无 Body**，task_id 在路径中，如 `GET /tasks/task-id-12345` |
| 身份 | 依赖 Body 的 `account` | 仅依赖 Header `x-api-key` |
| 成功响应 | 无 `account_name` | 响应中增加 `account_name`；结构仍含 `task_status`、`item_id`（完成时）、`message`（失败时） |

**实施要点：**

- 将“查任务状态”从 **POST /task-status + JSON body** 改为 **GET /tasks/{task_id}**。
- 删除 Body，不再传 `account` 与 `task_id`，仅用 URL 路径传 `task_id`。
- 客户端解析逻辑可保持不变：仍根据 `task_status`（in_progress / completed / failed）、`item_id`、`message` 处理；若需展示当前账户，使用响应中的 `account_name`。

---

### 6. 批量删除物品

| 项目 | 旧版 | 新版（按此实现） |
|------|------|------------------|
| 方法 + 路径 | `POST /delete-items` | `DELETE /items` |
| 请求 | Body 含 `account` + `items` 数组 | Body **不含** `account`，仅保留 `items` 数组 |
| 成功响应 | 无 `account_name` | 响应顶层增加 `account_name` |

**实施要点：**

- 路径从 `POST /delete-items` 改为 **DELETE /items**，请求方法改为 DELETE。
- 请求体从 `{"account":"xxx", "items":[...]}` 改为 `{"items":[...]}`，删除 `account` 字段。
- 响应中 `received_count`、`successful_count`、`failed_count`、`failed_items` 语义不变，按新文档为准；新增对 `account_name` 的兼容即可。

---

## 三、错误处理与状态码

- **统一错误体**：所有错误（含 401）均按新文档返回：
  ```json
  { "status": "error", "message": "具体错误描述" }
  ```
- 工程师需注意：旧版 401 可能返回 `status: "unauthenticated"`，新版统一为 `status: "error"`，客户端若此前依赖 `unauthenticated`，需改为判断 HTTP 状态码 401 或统一按 `status === "error"` 处理。
- 状态码含义与旧版一致：400 请求错误、401 未授权、404 未找到、500 服务器错误；创建任务成功为 **201**。

---

## 四、实施检查清单（按新 API 文档执行）

- [ ] **认证**：所有请求只带 `x-api-key`，不再在任何接口传 `account`（查询参数或 Body）。
- [ ] **1. 账户信息**：调用 `GET /account`，去掉 `account` 查询参数；处理 `data.account_name`。
- [ ] **2. 全部物品**：调用 `GET /items`，去掉 `account` 查询参数；处理顶层 `account_name`。
- [ ] **3. 物品详情**：改为 `GET /items/{item_id}`，不再使用 POST /item 与 Body；处理 `account_name`。
- [ ] **4. 创建任务**：改为 `POST /tasks`，表单去掉 `account`；按 201 判断成功；task_type=3 仅接受 .png。
- [ ] **5. 任务状态**：改为 `GET /tasks/{task_id}`，不再使用 POST /task-status 与 Body；处理 `account_name`。
- [ ] **6. 批量删除**：改为 `DELETE /items`，Body 仅保留 `items`；处理 `account_name`。
- [ ] **错误处理**：统一按 `status: "error"` 和 `message` 解析，401 不再依赖 `unauthenticated`。

---

## 五、简要路径与参数对照表

| 功能 | 旧版 | 新版 |
|------|------|------|
| 账户信息 | GET /account-info?account= | GET /account |
| 全部物品 | GET /items?account= | GET /items |
| 物品详情 | POST /item Body{account, item_id} | GET /items/{item_id} |
| 创建任务 | POST /create Form+account | POST /tasks Form 无 account |
| 任务状态 | POST /task-status Body{account, task_id} | GET /tasks/{task_id} |
| 批量删除 | POST /delete-items Body{account, items} | DELETE /items Body{items} |

实施时以 **API_new_ch.md（RESTful v2.0）** 为准；若有未列出的字段或边界情况，以新文档描述与示例为准。
