# 品牌介绍接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/brand`（路径风格特殊：列表 `/list`、详情 `/detail/{id}`、标记当前 `/{id}/mark-current`）

版本状态机：`draft`（草稿）→ `current`（当前使用）→ `archived`（已归档），**全表仅 1 条 current**。

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `version` | String | 否 | 版本号，**自动生成**（v1.0/v1.1/v1.2…，末位 +1） |
| `title` | String | 否 | 标题 |
| `summary` | String | 否 | 描述 |
| `content` | String | 否 | 富文本内容（HTML） |
| `status` | String | 否 | 状态：`draft` / `current` / `archived` |
| `isCurrent` | Boolean | 否 | 是否当前使用（仅 current 那条为 true） |
| `createdBy` | String | 否 | 创建者 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

### 状态流转

```
新建 ──► draft ──(mark-current)──► current ──(被新 current 替换)──► archived
                 ↑ 仅 draft 可编辑/可删       ↑ current 不可删
```

---

## 业务错误码

| code | 含义 |
|---|---|
| `1400` | 标题、描述、内容均不能为空 |
| `1401` | 该版本已是当前使用（mark-current 时目标已是 current） |
| `1402` | 当前使用版本不可删除（需先标记其他版本为当前） |
| `1403` | 仅草稿状态可编辑（修改非 draft 版本） |
| `1404` | 该版本不存在或已删除 |

---

## 接口详情

### 1. 列表 `GET /api/brand/list`

| Query | 类型 | 默认 | 说明 |
|---|---|---|---|
| `page` / `pageSize` | int | 1 / 20 | 分页 |
| `keyword` | String | - | 模糊匹配 `title` / `summary` |
| `status` | String | `all` | `all` / `current` / `draft` / `archived` |
| `sortBy` | String | `updatedAt` | `updatedAt` / `createdAt` / `version` |
| `sortOrder` | String | `desc` | `asc` / `desc` |

```bash
curl "http://localhost:37050/api/brand/list?page=1&pageSize=10&status=current"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 3, "version": "v1.2", "title": "服务体系 - 四大承诺",
        "summary": "推出四大承诺",
        "content": "<h2>四大承诺</h2><ol>...</ol>",
        "status": "current", "isCurrent": true, "createdBy": "admin",
        "createdAt": "2026-07-02T09:50:00", "updatedAt": "2026-07-02T09:50:00"
      }
    ],
    "total": 1, "page": 1, "pageSize": 10
  },
  "message": null
}
```

### 2. 标记为当前使用 `POST /api/brand/{id}/mark-current`

**事务原子操作**：把指定版本设为 `current`，原 current 自动降级为 `archived`。

- 目标已是 current → `1401`
- 不存在 → `1404`
- 成功返回升级后的版本对象

```bash
curl -X POST http://localhost:37050/api/brand/4/mark-current
```

### 3. 详情 `GET /api/brand/detail/{id}`

不存在 → `1404`。

### 4. 新增 `POST /api/brand`

Body（三字段必填）：
```json
{
  "title": "新版品牌故事",
  "summary": "2026 品牌升级",
  "content": "<h2>品牌故事</h2><p>...</p>"
}
```

- 三字段任一为空 → `1400`
- **不要传** `version`/`status`/`isCurrent`（后端自动：version 末位+1、status=draft、isCurrent=false）
- 成功返回含 id、version 的完整对象

### 5. 修改 `PUT /api/brand/{id}`

Body 同新增。**仅 `draft` 状态可改**。
- 非 draft → `1403`
- 三字段空 → `1400`
- 不存在 → `1404`

```bash
curl -X PUT http://localhost:37050/api/brand/4 -H "Content-Type: application/json" \
  -d '{"title":"新版品牌故事(改)","summary":"2026 升级","content":"<p>内容</p>"}'
```

### 6. 删除 `DELETE /api/brand/{id}`（逻辑删除）

- `current` 不可删 → `1402`（需先 mark-current 其他版本）
- 不存在 → `1404`
- 成功返回 `{ "id": <id> }`

---

## curl 速查

```bash
curl "http://localhost:37050/api/brand/list?status=current"
curl "http://localhost:37050/api/brand/detail/3"
curl -X POST http://localhost:37050/api/brand -H "Content-Type: application/json" \
  -d '{"title":"新版本","summary":"描述","content":"<p>内容</p>"}'
curl -X POST http://localhost:37050/api/brand/4/mark-current
curl -X PUT http://localhost:37050/api/brand/4 -H "Content-Type: application/json" \
  -d '{"title":"改","summary":"描述","content":"<p>内容</p>"}'
curl -X DELETE http://localhost:37050/api/brand/4
```

## 接口清单（6 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/brand/list` | 列表（keyword + status + 通用排序） |
| 2 | GET | `/api/brand/detail/{id}` | 详情 |
| 3 | POST | `/api/brand` | 新增（draft，自动 version） |
| 4 | PUT | `/api/brand/{id}` | 修改（仅 draft） |
| 5 | DELETE | `/api/brand/{id}` | 删除（current 不可删） |
| 6 | POST | `/api/brand/{id}/mark-current` | 标记当前（事务，原 current 降级） |

> 📋 待开发（小程序端）：`GET /api/brand/current`（返回当前 current 版本）
