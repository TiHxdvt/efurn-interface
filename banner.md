# 首页 Banner 接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/banner`

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | Banner 名称 |
| `cover` | String | 否 | 封面图地址 |
| `link` | String | 是 | 点击跳转链接，可能为 `null` |
| `sort` | Integer | 否 | 排序号，默认 `0`，**越小越靠前** |
| `enabled` | Boolean | 否 | 是否启用：`true` 上线 / `false` 下线 |
| `createTime` | String(Date) | 否 | 创建时间 |
| `updateTime` | String(Date) | 否 | 更新时间 |

> 🔒 `isDeleted` 为内部字段，不返回。

---

## 接口详情

### 1. 分页查询

`GET /api/banner`

| Query 参数 | 类型 | 必填 | 默认 | 说明 |
|---|---|---|---|---|
| `page` | int | 否 | `1` | 页码 |
| `pageSize` | int | 否 | `20` | 每页条数 |
| `name` | String | 否 | - | 名称**模糊**查询 |
| `enabled` | Boolean | 否 | - | `true`/`false`；不传查全部 |
| `sortBy` | String | 否 | `sort` | `sort` / `createdAt` |
| `order` | String | 否 | `asc` | `asc` / `desc`（`sortBy=createdAt` 时固定 desc） |

```bash
curl "http://localhost:37050/api/banner?page=1&pageSize=10&enabled=true&sortBy=sort&order=asc"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1,
        "name": "春日新品发布会",
        "cover": "https://images.unsplash.com/photo-1506784983877-45594efa4cbe?w=800&h=450&fit=crop",
        "link": "/pages/package-1/index",
        "sort": 0,
        "enabled": true,
        "createTime": "2026-07-01T14:18:11",
        "updateTime": "2026-07-01T14:18:11"
      }
    ],
    "total": 3, "page": 1, "pageSize": 10
  },
  "message": null
}
```

### 2. 新增 Banner

`POST /api/banner` · `Content-Type: application/json`

| Body 字段 | 类型 | 必填 | 默认 | 说明 |
|---|---|---|---|---|
| `name` | String | ✅ | - | Banner 名称 |
| `cover` | String | ✅ | - | 封面图地址 |
| `link` | String | 否 | `null` | 跳转链接 |
| `sort` | Integer | 否 | `0` | 排序号 |
| `enabled` | Boolean | 否 | `true`(DB) | 是否启用 |

```bash
curl -X POST http://localhost:37050/api/banner \
  -H "Content-Type: application/json" \
  -d '{"name":"新活动","cover":"https://x/a.jpg","link":"/pages/x","sort":5,"enabled":true}'
```

成功返回**含 id 的对象**。校验失败：`{"code":-1,"data":null,"message":"Banner名称不能为空"}`

### 3. 修改 Banner

`PUT /api/banner/{id}` · body 不含 id，全量更新

| Body 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `name` | String | ✅ | 名称 |
| `cover` | String | ✅ | 封面图 |
| `link` | String | 否 | 跳转链接 |
| `sort` | Integer | 否 | 排序号 |
| `enabled` | Boolean | 否 | 是否启用 |

```bash
curl -X PUT http://localhost:37050/api/banner/5 \
  -H "Content-Type: application/json" \
  -d '{"name":"新活动(改)","cover":"https://x/b.jpg","sort":5,"enabled":false}'
```

### 4. 删除 Banner（逻辑删除）

`DELETE /api/banner/{id}`

```bash
curl -X DELETE http://localhost:37050/api/banner/5
```

---

## curl 速查

```bash
curl "http://localhost:37050/api/banner?page=1&pageSize=10&enabled=true&sortBy=sort&order=asc"
curl -X POST http://localhost:37050/api/banner -H "Content-Type: application/json" \
  -d '{"name":"新活动","cover":"https://x/a.jpg","link":"/pages/x","sort":1,"enabled":true}'
curl -X PUT http://localhost:37050/api/banner/1 -H "Content-Type: application/json" \
  -d '{"name":"新活动(改)","cover":"https://x/a.jpg","sort":1,"enabled":true}'
curl -X DELETE http://localhost:37050/api/banner/1
```

## 接口清单

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/banner` | 分页查询 |
| 2 | POST | `/api/banner` | 新增（返回含 id 对象） |
| 3 | PUT | `/api/banner/{id}` | 修改 |
| 4 | DELETE | `/api/banner/{id}` | 逻辑删除 |

> 📋 待开发：`GET /api/banner/options`（小程序端仅上线）、`GET /api/banner/{id}`（详情）
