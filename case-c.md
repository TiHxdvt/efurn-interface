# 业主小程序案例接口文档（C 端）

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。
> PC 后台管理见 [case.md](./case.md)。

**路径前缀**：`/api/cases`（复数，与 PC 管理 `/api/case` 区分）

业主小程序案例列表 + 详情。

---

## 接口详情

### 1. 案例列表 `GET /api/cases`

仅上线案例，按 `sort` 升序，支持 style/area/layout 筛选，分页。

| Query | 类型 | 默认 | 说明 |
|---|---|---|---|
| `style` | String | - | 风格筛选，**模糊匹配**（如 `现代` 可匹配 `现代轻奢`、`现代简约`） |
| `area` | String | - | 面积筛选，精确匹配（如 `150㎡`） |
| `layout` | String | - | 户型筛选，匹配案例 `tags`（户型标签）中包含该项的案例 |
| `page` | int | `1` | 页码 |
| `pageSize` | int | `20` | 每页条数 |

```bash
curl "http://localhost:37050/api/cases?style=现代轻奢&page=1&pageSize=10"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1,
        "title": "翡翠湾 150㎡ 现代轻奢",
        "cover": "https://x/c1.jpg",
        "style": "现代轻奢",
        "area": "150㎡",
        "tags": ["现代", "轻奢", "大户型"],
        "description": "本案以现代轻奢为主题..."
      }
    ],
    "total": 4,
    "page": 1,
    "pageSize": 10
  },
  "message": null
}
```

**列表字段**（简化，不含富文本 content）

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 案例 id |
| `title` | String | 标题（= 后台 Case.name） |
| `cover` | String | 主图 |
| `style` | String | 风格 |
| `area` | String | 面积 |
| `tags` | String[] | 标签 |
| `description` | String | 介绍 |

### 2. 案例详情 `GET /api/cases/{id}`

返回完整案例（含富文本 content + 图片组 images + 关联 VR）。

```bash
curl "http://localhost:37050/api/cases/1"
```

```json
{
  "code": 0,
  "data": {
    "id": 1,
    "title": "翡翠湾 150㎡ 现代轻奢",
    "cover": "https://x/c1.jpg",
    "style": "现代轻奢",
    "area": "150㎡",
    "tags": ["现代", "轻奢", "大户型"],
    "description": "本案以现代轻奢为主题...",
    "content": "<p><strong>设计理念：</strong>...</p>",
    "images": ["https://x/c1.jpg", "https://x/c2.jpg"],
    "panoramaId": null
  },
  "message": null
}
```

**详情额外字段**

| 字段 | 类型 | 说明 |
|---|---|---|
| `content` | String | 图文详情（富文本 HTML） |
| `images` | String[] | 案例图片组（轮播） |
| `panoramaId` | Long | 关联 VR 全景 id，可为 `null`（用于跳转 VR 看） |

### 3. 筛选选项 `GET /api/cases/options`

返回所有可选的风格和户型，供小程序渲染筛选条件。数据**从已上线案例聚合去重**，后台新增带新风格/户型的案例后自动出现，无需单独维护。

```bash
curl "http://localhost:37050/api/cases/options"
```
```json
{
  "code": 0,
  "data": {
    "styles": ["现代轻奢", "新中式", "简约"],
    "layouts": ["三室两厅", "复式", "Loft"]
  },
  "message": null
}
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `styles` | String[] | 所有风格（来自已上线案例的 `style` 去重） |
| `layouts` | String[] | 所有户型（来自已上线案例的 `tags` 去重） |

---

## curl 速查

```bash
curl "http://localhost:37050/api/cases?page=1&pageSize=10"
curl "http://localhost:37050/api/cases?style=现代轻奢&layout=三室两厅"
curl "http://localhost:37050/api/cases/options"
curl "http://localhost:37050/api/cases/1"
```

## 接口清单

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/cases` | 列表（style/area/layout 筛选，分页） |
| 2 | GET | `/api/cases/{id}` | 详情（content + images + panoramaId） |
| 3 | GET | `/api/cases/options` | 筛选选项（所有风格 + 户型） |

> 📋 关联：首页精选案例用 `/api/home/featured`（前 6），本接口是完整列表/详情页。
> - 小程序端已对接：✅ `GET /api/cases` · `GET /api/cases/{id}` · `GET /api/cases/options`
