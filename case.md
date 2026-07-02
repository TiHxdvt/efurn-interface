# 案例管理接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/case`

单表 CRUD，含富文本 `content`。

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 案例名称（应用层唯一，1-50 字） |
| `cover` | String | 否 | 主图地址 |
| `tags` | String[] | 否 | 标签数组，1-5 项，每项 1-10 字 |
| `description` | String | 否 | 介绍（≤100 字） |
| `content` | String | 否 | 图文详情（富文本 HTML，去标签后非空） |
| `sort` | Integer | 否 | 排序号，默认 0 |
| `enabled` | Boolean | 否 | 是否启用 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

---

## 业务错误码

| code | 含义 | 触发场景 |
|---|---|---|
| `1001` | 名称重复 | name 已存在 |
| `1002` | 名称校验 | name 为空 / 超过 50 字 |
| `1003` | 主图必填 | cover 为空 |
| `1004` | 图文详情必填 | content 去标签后为空 |
| `1005` | 标签校验 | 标签数非 1-5 个 / 单个非 1-10 字 |
| `1006` | 介绍校验 | description 为空 / 超过 100 字 |
| `404` | 不存在 | id 查不到 |

> ⚠️ 案例模块的业务码语义与其它模块不同（1003=主图、1004=图文详情、1006=介绍），以上表为准。

---

## 接口详情

### 1. 列表 `GET /api/case`

| Query | 类型 | 说明 |
|---|---|---|
| `page` / `pageSize` | int | 默认 pageSize=20 |
| `name` | String | 名称模糊 |
| `enabled` | Boolean | 启用筛选 |

排序：`sort` 升序、`createdAt` 降序。返回完整字段（含 `content`）。

```bash
curl "http://localhost:37050/api/case?page=1&pageSize=10&enabled=true"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1, "name": "翡翠湾 150㎡ 现代轻奢",
        "cover": "https://...",
        "tags": ["现代", "轻奢", "大户型"],
        "description": "本案以现代轻奢为主题...",
        "content": "<p>富文本HTML...</p>",
        "sort": 0, "enabled": true,
        "createdAt": "2026-07-01T14:18:11", "updatedAt": "2026-07-01T14:18:11"
      }
    ],
    "total": 4, "page": 1, "pageSize": 10
  },
  "message": null
}
```

### 2. 字典 `GET /api/case/options`

```json
{ "code": 0, "data": { "enabledOptions": [ {"value": true, "label": "启用"}, {"value": false, "label": "禁用"} ] }, "message": null }
```

### 3. 详情 `GET /api/case/{id}`

返回单条案例完整字段。

### 4. 新增 `POST /api/case`

Body：
```json
{
  "name": "翡翠湾 150㎡ 现代轻奢",
  "cover": "https://images.unsplash.com/photo-1600607687939-ce8a6c25118c?w=750&q=80",
  "tags": ["现代", "轻奢", "大户型"],
  "description": "本案以现代轻奢为主题，融合金属、大理石、丝绒等材质。",
  "content": "<p><strong>设计理念：</strong>现代轻奢...</p>",
  "sort": 0,
  "enabled": true
}
```

校验顺序：cover(1003) → name(1002) → tags(1005) → description(1006) → content(1004) → name 唯一(1001)。成功返回含 id 的完整对象。

### 5. 修改 `PUT /api/case/{id}`

Body 同新增（全部字段可选，传啥改啥）。各字段提供时分别校验；name 变更时校验唯一 → `1001`。

```bash
curl -X PUT http://localhost:37050/api/case/1 -H "Content-Type: application/json" \
  -d '{"sort": 5, "enabled": false}'
```

### 6. 删除 `DELETE /api/case/{id}`（逻辑删除）

返回 `{ "id": <id> }`。

---

## curl 速查

```bash
curl "http://localhost:37050/api/case?page=1&pageSize=10&enabled=true"
curl "http://localhost:37050/api/case/options"
curl "http://localhost:37050/api/case/1"
curl -X POST http://localhost:37050/api/case -H "Content-Type: application/json" \
  -d '{"name":"新案例","cover":"https://x/c.jpg","tags":["现代"],"description":"介绍","content":"<p>详情</p>","sort":0,"enabled":true}'
curl -X PUT http://localhost:37050/api/case/1 -H "Content-Type: application/json" \
  -d '{"sort": 5, "enabled": false}'
curl -X DELETE http://localhost:37050/api/case/1
```

## 接口清单（6 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/case` | 列表（name/enabled 筛选） |
| 2 | GET | `/api/case/options` | 启用状态字典 |
| 3 | GET | `/api/case/{id}` | 详情 |
| 4 | POST | `/api/case` | 新增 |
| 5 | PUT | `/api/case/{id}` | 修改 |
| 6 | DELETE | `/api/case/{id}` | 逻辑删除 |

> 📋 待开发（小程序端）：`GET /api/case/list`（style/area 筛选）、`GET /api/case/detail/{id}`、`GET /api/case/featured`（首页前 6）
