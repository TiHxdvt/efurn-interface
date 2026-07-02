# 整装套餐接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/package`（路径风格特殊：列表 `/list`、详情 `/detail/{id}`，非标准 RESTful）

单表，含富文本 `content`。

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `cover` | String | 否 | 主图地址 |
| `name` | String | 否 | 套餐名称 |
| `tier` | String | 否 | 档位（尊享套餐/悦享套餐等） |
| `price` | Long | 否 | 价格（整数） |
| `subtitle` | String | 是 | 副标题 |
| `tags` | String[] | 是 | 标签数组 |
| `content` | String | 是 | 图文详情（富文本 HTML，用 `<h2>` 分块：套餐亮点/全包包含/合作品牌） |
| `bookable` | Boolean | 否 | 是否展示「立即预约」按钮 |
| `sort` | Integer | 否 | 排序号，默认 0 |
| `createdBy` | String | 否 | 创建者 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

---

## 业务错误码

| code | 含义 |
|---|---|
| `1404` | 套餐不存在或已删除（本模块专用码） |

> 套餐新增/修改**宽松不强制校验**必填（对齐 demo），前端表单自行保证。

---

## 接口详情

### 1. 列表 `GET /api/package/list`

| Query | 类型 | 说明 |
|---|---|---|
| `page` / `pageSize` | int | 默认 pageSize=20 |
| `keyword` | String | 模糊匹配 `name` / `tier` / `subtitle` |
| `bookable` | Boolean | 是否可预约筛选 |
| `sortField` | String | 排序字段：`sort`(默认) / `price` / `createdAt` |
| `sortOrder` | String | `asc`(默认) / `desc` |

```bash
curl "http://localhost:37050/api/package/list?page=1&pageSize=10&keyword=轻奢&sortField=price&sortOrder=asc"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1, "name": "精奢整装套餐", "tier": "尊享套餐",
        "cover": "https://...", "price": 299999,
        "subtitle": "一线品牌·全屋定制·拎包入住",
        "content": "<h2>套餐亮点</h2>...",
        "bookable": true, "sort": 10, "createdBy": "admin",
        "createdAt": "2026-07-01T14:18:11", "updatedAt": "2026-07-01T14:18:11"
      }
    ],
    "total": 3, "page": 1, "pageSize": 10
  },
  "message": null
}
```

### 2. 详情 `GET /api/package/detail/{id}`

返回单条套餐完整字段。

### 3. 新增 `POST /api/package`

Body：
```json
{
  "cover": "https://images.unsplash.com/photo-1600607687939-ce8a6c25118c?w=800&h=450&fit=crop",
  "name": "精奢整装套餐",
  "tier": "尊享套餐",
  "price": 299999,
  "subtitle": "一线品牌·全屋定制·拎包入住",
  "content": "<h2>套餐亮点</h2><p>主材：诺贝尔/圣象</p>",
  "bookable": true,
  "sort": 10
}
```

`createdBy` 后端自动填（admin）。成功返回含 id 的完整对象。

### 4. 修改 `PUT /api/package/{id}`

Body 同新增（全部字段可选）。不存在 → `1404`。

```bash
curl -X PUT http://localhost:37050/api/package/1 -H "Content-Type: application/json" \
  -d '{"price": 288888, "bookable": false}'
```

### 5. 删除 `DELETE /api/package/{id}`（逻辑删除）

不存在 → `1404`。成功返回 `{ "id": <id> }`。

---

## curl 速查

```bash
curl "http://localhost:37050/api/package/list?page=1&pageSize=10&keyword=轻奢"
curl "http://localhost:37050/api/package/detail/1"
curl -X POST http://localhost:37050/api/package -H "Content-Type: application/json" \
  -d '{"name":"新套餐","tier":"尊享","price":299999,"bookable":true,"sort":10}'
curl -X PUT http://localhost:37050/api/package/1 -H "Content-Type: application/json" \
  -d '{"price": 288888}'
curl -X DELETE http://localhost:37050/api/package/1
```

## 接口清单（5 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/package/list` | 列表（keyword + bookable + 通用排序） |
| 2 | GET | `/api/package/detail/{id}` | 详情 |
| 3 | POST | `/api/package` | 新增 |
| 4 | PUT | `/api/package/{id}` | 修改 |
| 5 | DELETE | `/api/package/{id}` | 逻辑删除 |

> 📋 待开发（小程序端）：`GET /api/package/home`（首页最多 3 个）、`GET /api/package/detail/{id}`（C 端）
