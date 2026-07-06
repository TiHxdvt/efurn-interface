# 生活场景 Scene 接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/scene`

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 场景名称（应用层唯一，1-50 字） |
| `cover` | String | 否 | 封面图地址 |
| `likes` | Integer | 否 | 点赞数（**只读**，小程序累计，后台不可改） |
| `dislikes` | Integer | 否 | 点踩数（**只读**，小程序累计，后台不可改） |
| `sort` | Integer | 否 | 排序号，默认 0 |
| `enabled` | Boolean | 否 | 是否启用 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

> 📌 `likes`/`dislikes` 为**只读**：新增时后端置 `0`，修改时不影响（保持原值，由小程序点赞累计）。

---

## 业务错误码

| code | 含义 |
|---|---|
| `1001` | 该生活场景名称已存在 |
| `1002` | 名称必填 / 超过 50 字 |
| `1003` | 请上传主图（cover 为空） |
| `404` | 生活场景不存在 |

---

## 接口详情

### 1. 列表 `GET /api/scene`

| Query | 类型 | 默认 | 说明 |
|---|---|---|---|
| `page` / `pageSize` | int | 1 / 20 | 分页 |
| `name` | String | - | 名称模糊 |
| `enabled` | Boolean | - | 启用筛选 |
| `sortBy` | String | `createdAt` | `createdAt` / `likes` / `dislikes` |
| `order` | String | `desc` | `asc` / `desc` |

### 2. 字典 `GET /api/scene/options`

```json
{ "code": 0, "data": { "enabledOptions": [ {"value": true, "label": "启用"}, {"value": false, "label": "禁用"} ] }, "message": null }
```

### 3. 详情 `GET /api/scene/{id}`

不存在 → `404`。

### 4. 新增 `POST /api/scene`

Body：`{ name, cover, enabled?, sort? }`（**不要传** likes/dislikes，后端置 0）
- 校验：cover(1003) → name(1002) → name 唯一(1001)

### 5. 修改 `PUT /api/scene/{id}`

Body 同新增（全可选）。仅改 name/cover/enabled/sort；**likes/dislikes 不可改**（DTO 无该字段）。name 变更时校验唯一 → `1001`。不存在 → `404`。

### 6. 删除 `DELETE /api/scene/{id}`（逻辑删除）→ `{ "id": <id> }`

---

## curl 速查

```bash
curl "http://localhost:37050/api/scene?page=1&pageSize=10&sortBy=likes&order=desc"
curl "http://localhost:37050/api/scene/options"
curl "http://localhost:37050/api/scene/1"
curl -X POST http://localhost:37050/api/scene -H "Content-Type: application/json" \
  -d '{"name":"安静阅读角","cover":"https://x/s.jpg","sort":5,"enabled":true}'
curl -X PUT http://localhost:37050/api/scene/1 -H "Content-Type: application/json" \
  -d '{"sort":5,"enabled":false}'
curl -X DELETE http://localhost:37050/api/scene/1
```

## 接口清单（6 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/scene` | 列表（name/createdAt·likes·dislikes） |
| 2 | GET | `/api/scene/options` | 启用状态字典 |
| 3 | GET | `/api/scene/{id}` | 详情 |
| 4 | POST | `/api/scene` | 新增（likes/dislikes 默认 0） |
| 5 | PUT | `/api/scene/{id}` | 修改（仅 name/cover/enabled/sort） |
| 6 | DELETE | `/api/scene/{id}` | 逻辑删除 |

> 📋 小程序端已对接：✅ `POST /api/scene/{id}/vote`（点赞/点踩）

### C 端：投票 `POST /api/scene/{id}/vote`

Body：`{ "type": "like" }` 或 `{ "type": "dislike" }`

返回：`{ "code": 0, "data": { "id": 1, "likes": 129, "dislikes": 3 }, "message": null }`

```bash
curl -X POST http://localhost:37050/api/scene/1/vote -H "Content-Type: application/json" -d '{"type":"like"}'
```
