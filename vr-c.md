# 业主小程序 VR 全景接口文档（C 端）

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。
> PC 后台管理见 [vr.md](./vr.md)。

**路径前缀**：`/api/vr`（C 端接口与 PC 管理共用前缀，通过路径区分）

业主小程序 VR 样板间列表 + 详情 + 全景数据。

---

## 接口详情

### 1. VR 样板间列表 `GET /api/vr/list`

仅返回上线（`enabled=true`）的 VR 全景，按 `sort` 升序，分页。

| Query | 类型 | 默认 | 说明 |
|---|---|---|---|
| `page` | int | `1` | 页码 |
| `pageSize` | int | `20` | 每页条数 |

```bash
curl "http://localhost:37050/api/vr/list?page=1&pageSize=10"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1,
        "name": "现代轻奢·125㎡大平层",
        "description": "现代轻奢风格，三室两厅",
        "cover": "https://...",
        "style": "轻奢",
        "area": "125㎡",
        "viewerUrl": "/vr-viewer/1"
      }
    ],
    "total": 4,
    "page": 1,
    "pageSize": 10
  },
  "message": null
}
```

**响应字段**

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | VR 全景 id |
| `name` | String | 名称 |
| `description` | String | 描述（= PC 端 remark） |
| `cover` | String | 封面图（= 默认场景的 image） |
| `style` | String | 风格标签（取 tags 第一个） |
| `area` | String | 面积标签（取 tags 第二个） |
| `viewerUrl` | String | VR 查看器链接（= 后端生成的 `/vr-viewer/{id}`） |

---

### 2. VR 样板间详情 `GET /api/vr/detail/{id}`

返回单条 VR 完整信息（含场景列表）。

```bash
curl "http://localhost:37050/api/vr/detail/1"
```

```json
{
  "code": 0,
  "data": {
    "id": 1,
    "name": "现代轻奢·125㎡大平层",
    "viewerUrl": "/vr-viewer/1",
    "scenes": [
      { "id": 1, "name": "客厅", "image": "https://...", "isDefault": true, "sortOrder": 1 },
      { "id": 2, "name": "阳台", "image": "https://...", "isDefault": false, "sortOrder": 2 }
    ]
  },
  "message": null
}
```

**响应字段**

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | VR 全景 id |
| `name` | String | 名称 |
| `viewerUrl` | String | VR 查看器链接 |
| `scenes` | Scene[] | 场景列表，按 sortOrder 升序 |

**Scene**

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 场景 id |
| `name` | String | 场景名（客厅/主卧等） |
| `image` | String | 等距矩形全景图 URL |
| `isDefault` | Boolean | 是否默认场景 |
| `sortOrder` | Integer | 排序号 |

> 仅返回 `enabled=true` 的 VR；不存在或已下线 → `404`。

---

## curl 速查

```bash
curl "http://localhost:37050/api/vr/list"
curl "http://localhost:37050/api/vr/detail/1"
```

## 接口清单

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/vr/list` | VR 样板间列表（仅上线） |
| 2 | GET | `/api/vr/detail/{id}` | VR 样板间详情（含场景列表） |

> 📋 关联：案例详情中的"进入 VR 全景"按钮通过 `panoramaId` 跳转；独立 VR 查看器用 web-view 加载 `viewerUrl`。
