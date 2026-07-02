# 业主小程序首页接口文档（C 端）

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/home`

业主小程序首页聚合接口，**仅返回上线数据**，字段对齐 miniapp（`title/image` 而非 PC 的 `name/cover`）。

---

## 接口详情

### 1. 首页 Banner `GET /api/home/banners`

返回所有启用的 Banner，按 `sort` 升序。供小程序首页轮播。

```bash
curl "http://localhost:37050/api/home/banners"
```

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "title": "春日新品发布会",
      "image": "https://images.unsplash.com/photo-1506784983877-45594efa4cbe?w=800&h=450&fit=crop",
      "link": "/pages/package-1/index"
    },
    {
      "id": 2,
      "title": "业主专属福利月",
      "image": "https://x/b.jpg",
      "link": "/pages/package-2/index"
    }
  ],
  "message": null
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | Banner id |
| `title` | String | 标题（= 后台 Banner.name） |
| `image` | String | 图片（= 后台 Banner.cover） |
| `link` | String | 跳转链接，可为 `null` |

> 📌 字段映射：后台管理用 `name/cover`，小程序展示用 `title/image`，本接口已转换。

### 2. 首页生活场景 `GET /api/home/scenes`

返回所有启用的生活场景，按 `sort` 升序，含 `likes/dislikes` 供展示与点赞。

```bash
curl "http://localhost:37050/api/home/scenes"
```

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "title": "清晨第一缕阳光",
      "image": "https://x/s1.jpg",
      "likes": 128,
      "dislikes": 3
    },
    {
      "id": 3,
      "title": "家庭聚餐时刻",
      "image": "https://x/s3.jpg",
      "likes": 215,
      "dislikes": 2
    }
  ],
  "message": null
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 场景 id（点赞接口用） |
| `title` | String | 标题（= 后台 Scene.name） |
| `image` | String | 图片（= 后台 Scene.cover） |
| `likes` | Integer | 点赞数 |
| `dislikes` | Integer | 点踩数 |

> 📌 `likes/dislikes` 只读（小程序累计），后台不可改。

---

## 接口清单

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/home/banners` | 首页 Banner（仅上线） |
| 2 | GET | `/api/home/scenes` | 首页生活场景（仅上线 + likes/dislikes） |

> 📋 首页其它区块：
> - **功能入口金刚区**：前端写死（4 个入口：找案例/预约报价/需求测评/VR样板间），无需后端接口
> - **精选案例 / 整装套餐**：待开发 `GET /api/home/featured`（案例前 6）、`GET /api/home/packages`（套餐最多 3）
> - **场景点赞**：待开发 `POST /api/scene/{id}/vote`（like/dislike）
