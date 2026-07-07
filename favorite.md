# 用户收藏接口文档（C 端）

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/favorite`

用户收藏管理，当前仅支持案例收藏（`targetType=case`）。

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `userId` | Long | 否 | 微信用户 id |
| `targetType` | String | 否 | 收藏类型：`case` |
| `targetId` | Long | 否 | 收藏目标 id |
| `createdAt` | String(Date) | 否 | 收藏时间 |

---

## 接口详情

### 1. 收藏列表 `GET /api/favorite/list`

Header：`X-User-Id: <微信用户 id>`

| Query | 类型 | 必填 | 默认 | 说明 |
|---|---|---|---|---|
| `targetType` | String | 是 | - | 收藏类型（`case`） |
| `page` | int | 否 | `1` | 页码 |
| `pageSize` | int | 否 | `20` | 每页条数 |

返回该用户收藏的目标 id 列表（按时间降序，分页）。未登录返回空数组。

```bash
curl -H "X-User-Id: 1" "http://localhost:37050/api/favorite/list?targetType=case&page=1&pageSize=10"
```

```json
{ "code": 0, "data": { "list": [3, 1, 5], "total": 3, "page": 1, "pageSize": 10 }, "message": null }
```

---

### 2. 检查是否收藏 `GET /api/favorite/check`

Header：`X-User-Id: <微信用户 id>`

| Query | 类型 | 必填 | 说明 |
|---|---|---|---|
| `targetType` | String | 是 | 收藏类型 |
| `targetId` | Long | 是 | 目标 id |

```bash
curl -H "X-User-Id: 1" "http://localhost:37050/api/favorite/check?targetType=case&targetId=1"
```

```json
{ "code": 0, "data": { "favorited": true }, "message": null }
```

---

### 3. 添加收藏 `POST /api/favorite`

Header：`X-User-Id: <微信用户 id>`

Body：`{ "targetType": "case", "targetId": 1 }`

已收藏时幂等返回（不报错）。未登录 → `code: -1`。

```bash
curl -X POST http://localhost:37050/api/favorite -H "Content-Type: application/json" \
  -H "X-User-Id: 1" -d '{"targetType":"case","targetId":1}'
```

```json
{ "code": 0, "data": { "id": 1 }, "message": null }
```

---

### 4. 取消收藏 `DELETE /api/favorite`

Header：`X-User-Id: <微信用户 id>`

Body：`{ "targetType": "case", "targetId": 1 }`

```bash
curl -X DELETE http://localhost:37050/api/favorite -H "Content-Type: application/json" \
  -H "X-User-Id: 1" -d '{"targetType":"case","targetId":1}'
```

```json
{ "code": 0, "data": { "id": 1 }, "message": null }
```

---

## curl 速查

```bash
# 收藏列表
curl -H "X-User-Id: 1" "http://localhost:37050/api/favorite/list?targetType=case"

# 检查是否收藏
curl -H "X-User-Id: 1" "http://localhost:37050/api/favorite/check?targetType=case&targetId=1"

# 添加收藏
curl -X POST http://localhost:37050/api/favorite -H "Content-Type: application/json" \
  -H "X-User-Id: 1" -d '{"targetType":"case","targetId":1}'

# 取消收藏
curl -X DELETE http://localhost:37050/api/favorite -H "Content-Type: application/json" \
  -H "X-User-Id: 1" -d '{"targetType":"case","targetId":1}'
```

## 接口清单（4 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/favorite/list` | 收藏 ID 列表 |
| 2 | GET | `/api/favorite/check` | 检查是否已收藏 |
| 3 | POST | `/api/favorite` | 添加收藏 |
| 4 | DELETE | `/api/favorite` | 取消收藏 |
