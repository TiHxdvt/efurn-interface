# VR 全景管理接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/vr`

本模块含主表 + 场景子表（一个 VR 含多张全景图，可在小程序内切换场景）。

---

## 数据模型

### VR Panorama

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 名称（应用层唯一） |
| `tags` | String[] | 否 | 标签数组，1-5 项，每项 1-10 字 |
| `scenes` | Scene[] | 否 | 场景列表，至少 1 个，须含 1 个默认 |
| `url` | String | 否 | 预览链接，**后端自动生成** `/vr-viewer/{id}`，不可改 |
| `coverThumb` | String | 是 | 封面缩略图（**响应注入**：默认场景的 image，无默认取首个） |
| `sort` | Integer | 否 | 排序号，默认 0 |
| `enabled` | Boolean | 否 | 是否启用 |
| `remark` | String | 是 | 备注 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

### Scene（场景）

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 场景名（客厅/主卧等） |
| `image` | String | 否 | 等距矩形全景图 URL |
| `isDefault` | Boolean | 否 | 是否默认场景 |
| `order` | Integer | 否 | 排序号 |

---

## 业务错误码

| code | 含义 | 触发场景 |
|---|---|---|
| `1001` | 名称重复 | name 已存在 |
| `1002` | 名称必填 | name 为空 |
| `1004` | 场景校验 | 场景为空 / 未设置默认场景 |
| `1005` | 标签校验 | 标签数非 1-5 个 / 单个标签非 1-10 字 |
| `404` | 不存在 | id 查不到 |

---

## 接口详情

### 1. 列表 `GET /api/vr`

| Query | 类型 | 说明 |
|---|---|---|
| `page` / `pageSize` | int | 默认 pageSize=20 |
| `name` | String | 名称模糊 |
| `enabled` | Boolean | 启用筛选 |
| `remark` | String | 备注模糊 |

排序：`sort` 升序、`createdAt` 降序。**列表每条带完整 `scenes` + `coverThumb`**。

```bash
curl "http://localhost:37050/api/vr?page=1&pageSize=10&enabled=true"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1, "name": "北欧极简客厅 89㎡",
        "tags": ["北欧", "极简", "客厅"],
        "scenes": [
          { "id": 1, "name": "客厅", "image": "https://...", "isDefault": true, "order": 1 },
          { "id": 2, "name": "阳台", "image": "https://...", "isDefault": false, "order": 2 }
        ],
        "url": "/vr-viewer/1", "coverThumb": "https://...",
        "sort": 0, "enabled": true, "remark": "样板间",
        "createdAt": "2026-07-01T14:18:11", "updatedAt": "2026-07-01T14:18:11"
      }
    ],
    "total": 3, "page": 1, "pageSize": 10
  },
  "message": null
}
```

### 2. 字典 `GET /api/vr/options`

```json
{ "code": 0, "data": { "enabledOptions": [ {"value": true, "label": "启用"}, {"value": false, "label": "禁用"} ] }, "message": null }
```

### 3. 详情 `GET /api/vr/{id}`

返回单条 VR（含 scenes + coverThumb）。

### 4. 新增 `POST /api/vr`

Body：
```json
{
  "name": "现代轻奢客厅",
  "tags": ["现代", "轻奢", "客厅"],
  "scenes": [
    { "name": "客厅", "image": "https://...", "isDefault": true },
    { "name": "阳台", "image": "https://...", "isDefault": false }
  ],
  "sort": 0,
  "enabled": true,
  "remark": "样板间"
}
```

- name 空 → `1002`；场景为空/无默认 → `1004`；标签不合法 → `1005`；name 重复 → `1001`
- **不要传 `url`**（后端生成），传了也会被忽略
- 成功返回含 id、url、scenes 的完整对象

### 5. 修改 `PUT /api/vr/{id}`

Body 同新增（全部字段可选，传啥改啥）。

- **`scenes` 整体替换**：传则先删旧场景再插新的（同需求梳理选项）；不传则保留原场景
- **`url` 不可改**（传了忽略）
- name 变更时校验唯一 → `1001`

```bash
curl -X PUT http://localhost:37050/api/vr/1 -H "Content-Type: application/json" \
  -d '{"remark":"样板间(改)","scenes":[{"name":"客厅","image":"https://x","isDefault":true}]}'
```

### 6. 删除 `DELETE /api/vr/{id}`（逻辑删除）

逻辑删除主表 + 其下所有场景。返回 `{ "id": <id> }`。

---

## curl 速查

```bash
curl "http://localhost:37050/api/vr?page=1&pageSize=10&enabled=true"
curl "http://localhost:37050/api/vr/options"
curl "http://localhost:37050/api/vr/1"
curl -X POST http://localhost:37050/api/vr -H "Content-Type: application/json" \
  -d '{"name":"现代轻奢客厅","tags":["现代","轻奢"],"scenes":[{"name":"客厅","image":"https://x","isDefault":true}],"sort":0,"enabled":true}'
curl -X PUT http://localhost:37050/api/vr/1 -H "Content-Type: application/json" \
  -d '{"remark":"改备注"}'
curl -X DELETE http://localhost:37050/api/vr/1
```

## 接口清单（6 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/vr` | 列表（name/enabled/remark + scenes + coverThumb） |
| 2 | GET | `/api/vr/options` | 启用状态字典 |
| 3 | GET | `/api/vr/{id}` | 详情 |
| 4 | POST | `/api/vr` | 新增（生成 url） |
| 5 | PUT | `/api/vr/{id}` | 修改（scenes 整体替换） |
| 6 | DELETE | `/api/vr/{id}` | 逻辑删除（主表 + 场景） |

> 📋 待开发（小程序端）：`GET /api/vr/list`、`GET /api/vr/detail/{id}`（C 端展示型，只返回启用项）
