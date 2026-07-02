# 生活场景 Scene 接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/scene`

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 场景名称 |
| `cover` | String | 否 | 封面图地址 |
| `likes` | Integer | 否 | 点赞数（**只读**，PC 端仅展示） |
| `dislikes` | Integer | 否 | 点踩数（**只读**，PC 端仅展示） |
| `sort` | Integer | 否 | 排序号，默认 `0` |
| `enabled` | Boolean | 否 | 是否启用 |
| `createTime` | String(Date) | 否 | 创建时间 |
| `updateTime` | String(Date) | 否 | 更新时间 |

> 📌 `likes` / `dislikes` 为**只读**：新增时后端置 `0`，修改时不影响（保持原值）。

---

## 接口详情

### 1. 分页查询

`GET /api/scene`

| Query 参数 | 类型 | 必填 | 默认 | 说明 |
|---|---|---|---|---|
| `page` | int | 否 | `1` | 页码 |
| `pageSize` | int | 否 | `20` | 每页条数 |
| `name` | String | 否 | - | 名称**模糊**查询 |
| `enabled` | Boolean | 否 | - | 启用筛选 |
| `sortBy` | String | 否 | `createdAt` | `createdAt` / `likes` / `dislikes` |
| `order` | String | 否 | `desc` | `asc` / `desc` |

```bash
curl "http://localhost:37050/api/scene?page=1&pageSize=10&name=聚餐&sortBy=likes&order=desc"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 3,
        "name": "家庭聚餐时刻",
        "cover": "https://images.unsplash.com/photo-1543353071-10c8ba85a904?w=400&h=300&fit=crop",
        "likes": 215, "dislikes": 2, "sort": 20, "enabled": true,
        "createTime": "2026-07-01T14:37:00", "updateTime": "2026-07-01T14:37:00"
      }
    ],
    "total": 1, "page": 1, "pageSize": 10
  },
  "message": null
}
```

### 2. 新增场景

`POST /api/scene` · **不要传** `likes` / `dislikes`，后端自动置 `0`

| Body 字段 | 类型 | 必填 | 默认 | 说明 |
|---|---|---|---|---|
| `name` | String | ✅ | - | 场景名称 |
| `cover` | String | ✅ | - | 封面图 |
| `enabled` | Boolean | 否 | `true`(DB) | 是否启用 |
| `sort` | Integer | 否 | `0` | 排序号 |

```bash
curl -X POST http://localhost:37050/api/scene \
  -H "Content-Type: application/json" \
  -d '{"name":"安静阅读角","cover":"https://x/s.jpg","sort":5,"enabled":true}'
```

成功返回含 id 的对象。

### 3. 修改场景

`PUT /api/scene/{id}` · 仅改 `name/cover/enabled/sort`，`likes/dislikes` 保持不变

```bash
curl -X PUT http://localhost:37050/api/scene/3 \
  -H "Content-Type: application/json" \
  -d '{"name":"家庭聚餐(改)","cover":"https://x/s.jpg","sort":20,"enabled":true}'
```

### 4. 删除场景（逻辑删除）

`DELETE /api/scene/{id}`

```bash
curl -X DELETE http://localhost:37050/api/scene/3
```

---

## curl 速查

```bash
curl "http://localhost:37050/api/scene?page=1&pageSize=10&name=聚餐&sortBy=likes&order=desc"
curl -X POST http://localhost:37050/api/scene -H "Content-Type: application/json" \
  -d '{"name":"安静阅读角","cover":"https://x/s.jpg","sort":5,"enabled":true}'
curl -X PUT http://localhost:37050/api/scene/1 -H "Content-Type: application/json" \
  -d '{"name":"安静阅读角","cover":"https://x/s.jpg","sort":5,"enabled":true}'
curl -X DELETE http://localhost:37050/api/scene/1
```

## 接口清单

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/scene` | 分页查询（name 模糊 / createdAt·likes·dislikes） |
| 2 | POST | `/api/scene` | 新增（likes/dislikes 默认 0） |
| 3 | PUT | `/api/scene/{id}` | 修改（仅 name/cover/enabled/sort） |
| 4 | DELETE | `/api/scene/{id}` | 逻辑删除 |

> 📋 待开发：`GET /api/scene/options`（小程序端）、`POST /api/scene/{id}/vote`（点赞/点踩）
