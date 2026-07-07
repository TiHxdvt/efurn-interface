# 业主小程序需求梳理接口文档（C 端）

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。
> PC 后台管理见 [requirement.md](./requirement.md)。

**路径前缀**：`/api/requirement`

业主小程序需求测评：拉取题目配置 → 作答 → 提交答卷 → 查看历史。

---

## 数据模型

### 分类 Category（与 PC 端一致）

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 主键 |
| `name` | String | 分类名 |
| `stepNumber` | Integer | 步骤编号 1-6 |
| `description` | String | 描述 |
| `sortOrder` | Integer | 排序号 |

### 题目 Question（与 PC 端一致）

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 主键 |
| `questionId` | String | 业务键（如 `q1_renovation_type`） |
| `step` | Integer | 步骤号 |
| `categoryId` | Long | 所属分类 id |
| `questionText` | String | 题干 |
| `questionType` | String | `single` / `multi` / `text` |
| `required` | Boolean | 是否必填 |
| `allowSkip` | Boolean | 是否允许跳过 |
| `withImage` | Boolean | 选项是否配图 |
| `sortOrder` | Integer | 排序号 |
| `config` | Object | 题型配置（`multi`→`{maxSelect}`，`text`→`{placeholder,maxLength}`） |

### 选项 Option（C 端专用，含 questionKey）

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 主键 |
| `questionId` | Long | 关联题目 id（数值型） |
| `questionKey` | String | 题目业务键（= 题目的 questionId，前端按此字段分组） |
| `value` | String | 选项值 |
| `label` | String | 选项文案 |
| `description` | String | 选项描述 |
| `imageUrl` | String | 选项配图 |
| `recommendationText` | String | 推荐语 |
| `allowCustom` | Boolean | 是否「其他」自由填空项 |
| `sortOrder` | Integer | 排序号 |

### 答卷 Result

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | Long | 主键 |
| `userId` | Long | 微信用户 id，可为 null |
| `answers` | Object | 答卷内容 `{ questionId: value }` |
| `createdAt` | String(Date) | 创建时间 |
| `updatedAt` | String(Date) | 更新时间 |

**answers 格式说明**：

| 题型 | 格式 | 示例 |
|---|---|---|
| 单选 | `{ qid: "value" }` | `{ "q1_renovation_type": "rough_new" }` |
| 单选「其他」 | `{ qid: "other", qid_other: "..." }` | `{ "q1_renovation_type": "other", "q1_renovation_type_other": "局部翻新厨房" }` |
| 多选 | `{ qid: ["v1", "v2"] }` | `{ "q6_members": ["single", "couple"] }` |
| 多选「其他」 | `{ qid: ["v1", "other"], qid_other: "..." }` | `{ "q6_members": ["single", "other"], "q6_members_other": "保姆" }` |
| 文本 | `{ qid: "..." }` | `{ "q5_location": "北京市朝阳区" }` |

---

## 接口详情

### 1. 全量题目配置 `GET /api/requirement/config`

一次请求返回分类 + 题目 + 选项，小程序端无需分步拉取。

```bash
curl "http://localhost:37050/api/requirement/config"
```

```json
{
  "code": 0,
  "data": {
    "categories": [
      { "id": 1, "name": "基础信息", "stepNumber": 1, "description": "房屋类型、面积、户型、楼层、地理位置等基础信息", "sortOrder": 1 }
    ],
    "questions": [
      { "id": 1, "questionId": "q1_renovation_type", "step": 1, "categoryId": 1, "questionText": "本次装修属于哪种类型?", "questionType": "single", "required": true, "allowSkip": false, "withImage": true, "sortOrder": 1, "config": {} }
    ],
    "options": [
      { "id": 1, "questionId": 1, "questionKey": "q1_renovation_type", "value": "rough_new", "label": "毛坯新房（第一次装修）", "description": "", "imageUrl": "", "recommendationText": "", "allowCustom": false, "sortOrder": 1 }
    ]
  },
  "message": null
}
```

> 📌 `questionKey` 是题目业务键字符串，前端按此字段将选项分组到对应题目。

---

### 2. 提交答卷 `POST /api/requirement/submit`

Header（可选）：`X-User-Id: <微信用户 id>`

Body：
```json
{
  "answers": {
    "q1_renovation_type": "rough_new",
    "q2_area": "100_120",
    "q3_layout": "three_two",
    "q6_members": ["single", "couple"],
    "q6_members_other": "",
    "q5_location": "北京市朝阳区"
  }
}
```

```bash
curl -X POST http://localhost:37050/api/requirement/submit \
  -H "Content-Type: application/json" \
  -H "X-User-Id: 1" \
  -d '{"answers":{"q1_renovation_type":"rough_new","q2_area":"100_120"}}'
```

响应：
```json
{ "code": 0, "data": { "id": 1 }, "message": null }
```

---

### 3. 我的答卷列表 `GET /api/requirement/results`

Header（可选）：`X-User-Id: <微信用户 id>`

传 `X-User-Id` 时返回该用户答卷；**未传时返回全部答卷**（按 `createdAt` 降序）。

```bash
curl -H "X-User-Id: 1" "http://localhost:37050/api/requirement/results"
```

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "userId": 1,
      "answers": { "q1_renovation_type": "rough_new", "q2_area": "100_120" },
      "createdAt": "2026-07-02T16:30:00",
      "updatedAt": "2026-07-02T16:30:00"
    }
  ],
  "message": null
}
```

---

### 4. 答卷详情 `GET /api/requirement/results/{id}`

```bash
curl "http://localhost:37050/api/requirement/results/1"
```

响应：单条答卷完整数据（同列表中单条结构）。不存在 → `404`。

---

### 5. 删除答卷 `DELETE /api/requirement/results/{id}`

```bash
curl -X DELETE "http://localhost:37050/api/requirement/results/1"
```

响应：`{ "code": 0, "data": { "id": 1 }, "message": null }`

不存在 → `404`。

---

## curl 速查

```bash
# 全量题目配置
curl "http://localhost:37050/api/requirement/config"

# 提交答卷
curl -X POST http://localhost:37050/api/requirement/submit \
  -H "Content-Type: application/json" -H "X-User-Id: 1" \
  -d '{"answers":{"q1_renovation_type":"rough_new"}}'

# 我的答卷列表
curl -H "X-User-Id: 1" "http://localhost:37050/api/requirement/results"

# 答卷详情
curl "http://localhost:37050/api/requirement/results/1"

# 删除答卷
curl -X DELETE "http://localhost:37050/api/requirement/results/1"
```

## 接口清单（5 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/requirement/config` | 全量题目配置（分类+题目+选项） |
| 2 | POST | `/api/requirement/submit` | 提交答卷 |
| 3 | GET | `/api/requirement/results` | 我的答卷列表 |
| 4 | GET | `/api/requirement/results/{id}` | 答卷详情 |
| 5 | DELETE | `/api/requirement/results/{id}` | 删除答卷 |

> 📋 关联：PC 端管理接口（分类/题目/选项 CRUD）见 [requirement.md](./requirement.md)，共 14 个端点。
