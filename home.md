# 业主小程序首页接口文档（C 端）

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/home`

业主小程序首页聚合接口，**仅返回上线数据**，字段对齐 miniapp。

---

## 接口详情

### 1. 首页 Banner `GET /api/home/banners`

仅上线，按 `sort` 升序。字段 `title/image/link`。

```bash
curl "http://localhost:37050/api/home/banners"
```
```json
{ "code": 0, "data": [ { "id": 1, "title": "春日新品发布会", "image": "https://x/a.jpg", "link": "/pages/package-1/index" } ], "message": null }
```

### 2. 首页生活场景 `GET /api/home/scenes`

仅上线，按 `sort` 升序，含 likes/dislikes。字段 `title/image/likes/dislikes`。

```bash
curl "http://localhost:37050/api/home/scenes"
```
```json
{ "code": 0, "data": [ { "id": 1, "title": "清晨第一缕阳光", "image": "https://x/s1.jpg", "likes": 128, "dislikes": 3 } ], "message": null }
```

### 3. 首页精选案例 `GET /api/home/featured`

仅上线，按 `sort` 升序，**最多 6 个**。字段 `title/cover/style/area/tags`。

```bash
curl "http://localhost:37050/api/home/featured"
```
```json
{
  "code": 0,
  "data": [ { "id": 1, "title": "翡翠湾 150㎡ 现代轻奢", "cover": "https://x/c.jpg", "style": "现代轻奢", "area": "150㎡", "tags": ["现代","轻奢"] } ],
  "message": null
}
```

### 4. 首页整装套餐 `GET /api/home/packages`

仅上线（`bookable=true`），按 `sort` 升序，**最多 3 个**。字段 `label/name/price(千分位)/tags/bgImage`。

```bash
curl "http://localhost:37050/api/home/packages"
```
```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "label": "尊享套餐",
      "name": "精奢整装套餐",
      "price": "299,999",
      "tags": ["一线品牌", "全屋定制", "拎包入住"],
      "bgImage": "https://x/p.jpg"
    }
  ],
  "message": null
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 套餐 id |
| `label` | String | 档位标签（= 后台 Package.tier，如「尊享套餐」） |
| `name` | String | 套餐名称 |
| `price` | String | 价格千分位字符串（299999 → `299,999`） |
| `tags` | String[] | 标签 |
| `bgImage` | String | 卡片背景图（= 后台 Package.cover） |

---

## 接口清单

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/home/banners` | 首页 Banner（仅上线） |
| 2 | GET | `/api/home/scenes` | 首页生活场景（仅上线 + likes/dislikes） |
| 3 | GET | `/api/home/featured` | 精选案例（仅上线，最多 6 个） |
| 4 | GET | `/api/home/packages` | 整装套餐（仅上线，最多 3 个） |

> 📌 首页五个区块状态：
> - Banner / 生活场景 / 精选案例 / 整装套餐：✅ C 端接口已全部完成
> - 功能入口金刚区：前端写死（4 个入口），无需后端接口
> - 小程序端已对接：✅ `GET /api/home/banners` · `GET /api/home/scenes` · `GET /api/home/featured` · `GET /api/home/packages`
