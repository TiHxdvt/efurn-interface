# 业主小程序案例接口文档（C 端）

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。
> PC 后台管理见 [case.md](./case.md)。

**路径前缀**：`/api/cases`（复数，与 PC 管理 `/api/case` 区分）

业主小程序案例列表 + 详情。

---

## 接口详情

### 1. 案例列表 `GET /api/cases`

仅上线案例，按 `sort` 升序，支持 style/area 筛选。

| Query | 类型 | 说明 |
|---|---|---|
| `style` | String | 风格筛选（如 `现代轻奢`），精确匹配 |
| `area` | String | 面积筛选（如 `150㎡`），精确匹配 |

```bash
curl "http://localhost:37050/api/cases?style=现代轻奢"
```

```json
{
  "code": 0,
  "data": [
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

---

## curl 速查

```bash
curl "http://localhost:37050/api/cases"
curl "http://localhost:37050/api/cases?style=现代轻奢&area=150㎡"
curl "http://localhost:37050/api/cases/1"
```

## 接口清单

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/cases` | 列表（style/area 筛选） |
| 2 | GET | `/api/cases/{id}` | 详情（content + images + panoramaId） |

> 📋 关联：首页精选案例用 `/api/home/featured`（前 6），本接口是完整列表/详情页。
