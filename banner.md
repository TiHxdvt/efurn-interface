# 首页 Banner 接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/banner`

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | Banner 名称（应用层唯一，1-50 字） |
| `cover` | String | 否 | 封面图地址 |
| `link` | String | 是 | 点击跳转链接，可能为 `null` |
| `sort` | Integer | 否 | 排序号，0-9999，越小越靠前 |
| `enabled` | Boolean | 否 | 是否启用：`true` 上线 / `false` 下线 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

> 🔒 `isDeleted` 为内部字段，不返回。

---

## 业务错误码

| code | 含义 |
|---|---|
| `1001` | 该 banner 名称已存在 |
| `1002` | 名称必填 / 超过 50 字 |
| `1003` | 请上传主图（cover 为空） |
| `1004` | 排序值需 0-9999 |
| `404` | Banner 不存在 |

---

## 接口详情

### 1. 列表 `GET /api/banner`

| Query | 类型 | 默认 | 说明 |
|---|---|---|---|
| `page` / `pageSize` | int | 1 / 20 | 分页 |
| `name` | String | - | 名称模糊 |
| `enabled` | Boolean | - | `true`/`false` |
| `sortBy` | String | `sort` | `sort` / `createdAt` |
| `order` | String | `asc` | `asc` / `desc`（`sortBy=createdAt` 时固定 desc） |

```bash
curl "http://localhost:37050/api/banner?page=1&pageSize=10&enabled=true&sortBy=sort&order=asc"
```

### 2. 字典 `GET /api/banner/options`

```json
{ "code": 0, "data": { "enabledOptions": [ {"value": true, "label": "启用"}, {"value": false, "label": "禁用"} ] }, "message": null }
```

### 3. 详情 `GET /api/banner/{id}`

不存在 → `404`。

### 4. 新增 `POST /api/banner`

Body：`{ name, cover, link?, sort?, enabled? }`
- 校验顺序：cover(1003) → name(1002) → sort(1004) → name 唯一(1001)
- 成功返回含 id 的对象

```bash
curl -X POST http://localhost:37050/api/banner -H "Content-Type: application/json" \
  -d '{"name":"新活动","cover":"https://x/a.jpg","link":"/pages/x","sort":5,"enabled":true}'
```

### 5. 修改 `PUT /api/banner/{id}`

Body 同新增（全部可选，传啥改啥）。各字段提供时分别校验；name 变更时校验唯一 → `1001`。不存在 → `404`。

### 6. 删除 `DELETE /api/banner/{id}`（逻辑删除）

返回 `{ "id": <id> }`。

---

## curl 速查

```bash
curl "http://localhost:37050/api/banner?page=1&pageSize=10&enabled=true"
curl "http://localhost:37050/api/banner/options"
curl "http://localhost:37050/api/banner/1"
curl -X POST http://localhost:37050/api/banner -H "Content-Type: application/json" \
  -d '{"name":"新活动","cover":"https://x/a.jpg","sort":1,"enabled":true}'
curl -X PUT http://localhost:37050/api/banner/1 -H "Content-Type: application/json" \
  -d '{"sort":2,"enabled":false}'
curl -X DELETE http://localhost:37050/api/banner/1
```

## 接口清单（6 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/banner` | 列表（name/enabled/sortBy/order） |
| 2 | GET | `/api/banner/options` | 启用状态字典 |
| 3 | GET | `/api/banner/{id}` | 详情 |
| 4 | POST | `/api/banner` | 新增（返回含 id 对象） |
| 5 | PUT | `/api/banner/{id}` | 修改 |
| 6 | DELETE | `/api/banner/{id}` | 逻辑删除 |

> 📋 待开发（小程序端）：首页 Banner 聚合接口（仅上线，字段映射 name→title/cover→image）
