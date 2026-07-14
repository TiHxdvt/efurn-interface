# 需求梳理模块接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/requirement`

本模块含 3 个子表：**分类 → 题目 → 选项**（一对多两层），共 14 个端点。

---

## 数据模型

### 分类 Category

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 分类名 |
| `stepNumber` | Integer | 否 | 步骤编号 1-6（应用层唯一） |
| `description` | String | 否 | 描述 |
| `order` | Integer | 否 | 排序号 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

### 题目 Question

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `questionId` | String | 否 | 业务键（应用层唯一，受保护不可改） |
| `step` | Integer | 否 | 步骤号（= 所属分类的 stepNumber） |
| `categoryId` | Long | 否 | 所属分类 id（受保护不可改） |
| `questionText` | String | 否 | 题干 |
| `questionType` | String | 否 | 题型：`single`/`multi`/`text` |
| `required` | Boolean | 否 | 是否必填 |
| `allowSkip` | Boolean | 否 | 是否允许跳过 |
| `withImage` | Boolean | 否 | 选项是否配图 |
| `order` | Integer | 否 | 排序号（正整数） |
| `config` | Object | 是 | 题型配置（见下） |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

**`config` 按题型**：
- `single`：无额外配置，`{}`
- `multi`：`{ "maxSelect": 3 }`（最多选几项）
- `text`：`{ "placeholder": "请输入", "maxLength": 200 }`

### 选项 Option

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `questionId` | Long | 否 | 关联题目 id |
| `value` | String | 否 | 选项值 |
| `label` | String | 否 | 选项文案 |
| `description` | String | 否 | 选项描述 |
| `imageUrl` | String | 否 | 选项配图 |
| `recommendationText` | String | 否 | 推荐语 |
| `allowCustom` | Boolean | 否 | 是否「其他」自由填空项 |
| `order` | Integer | 否 | 排序号 |

---

## 业务错误码

| code | 含义 | 触发场景 |
|---|---|---|
| `1001` | 唯一冲突 | 分类 stepNumber 重复 / 题目业务键 questionId 重复 |
| `1002` | 外键无效 | 题目 categoryId 指向不存在的分类 |
| `1003` | 已删不可改 | 编辑已逻辑删除的记录 |
| `1005` | 级联拒绝 | 删除分类时其下仍有未删题目 |
| `1006` | 排序非法 | 题目 order 不是正整数 |
| `404` | 不存在 | 分类/题目 id 查不到 |

---

## 接口详情

### 字典

#### `GET /api/requirement/options`

返回步骤、题型映射，供前端下拉/分组用。

```json
{
  "code": 0,
  "data": {
    "steps": [1, 2, 3, 4, 5, 6],
    "stepMap": { "1": "基础信息", "2": "家庭成员", "3": "空间需求", "4": "风格偏好", "5": "设备智能", "6": "预算时间" },
    "questionTypes": ["single", "multi", "text"],
    "questionTypeMap": { "single": "单选", "multi": "多选", "text": "文本" }
  },
  "message": null
}
```

> `stepMap` 由分类表动态生成（分类改名后自动同步）。

---

### 分类（6 个端点）

#### `GET /api/requirement/categories` — 列表

| Query | 类型 | 说明 |
|---|---|---|
| `page` / `pageSize` | int | 默认 pageSize=999（分类少，常全量） |
| `name` | String | 名称模糊 |
| `stepNumber` | Integer | 步骤筛选 |

排序：`order` 升序。返回 `PageResult<CategoryVO>`。

#### `GET /api/requirement/categories/{id}` — 详情

#### `POST /api/requirement/categories` — 新增

Body：`{ name, stepNumber, description?, order? }`（`order` 不传则自动取最大+1）
- `stepNumber` 重复 → `1001`
- 成功返回含 id 的对象

#### `PUT /api/requirement/categories/{id}` — 修改

Body 同新增。`stepNumber` 变更时校验唯一。

**上移/下移操作**：调用两次 `PUT` 交换两个分类的 `stepNumber` + `sortOrder`：

```bash
# 示例：分类 1（stepNumber=1）和分类 2（stepNumber=2）互换
curl -X PUT http://localhost:37050/api/requirement/categories/1 -H "Content-Type: application/json" \
  -d '{"stepNumber": 2, "sortOrder": 2}'
curl -X PUT http://localhost:37050/api/requirement/categories/2 -H "Content-Type: application/json" \
  -d '{"stepNumber": 1, "sortOrder": 1}'
```

#### `DELETE /api/requirement/categories/{id}` — 删除

- 分类下有题目 → `1005`
- 成功返回 `{ "id": <id> }`

#### `GET /api/requirement/categories/{id}/questions` — 题目数

返回 `{ "count": <n> }`（列表展示用）

---

### 题目（5 个端点）

#### `GET /api/requirement/questions` — 列表

| Query | 类型 | 说明 |
|---|---|---|
| `page` / `pageSize` | int | 默认 pageSize=20 |
| `questionId` | String | 业务键模糊 |
| `questionText` | String | 题干模糊 |
| `questionType` | String | `single`/`multi`/`text` |
| `categoryId` | Long | 所属分类 |
| `required` | Boolean | 是否必填 |

排序：`order` 升序。

#### `GET /api/requirement/questions/{id}` — 详情

#### `POST /api/requirement/questions` — 新增

Body：
```json
{
  "categoryId": 2,
  "questionText": "是否有特殊人群需求？",
  "questionType": "single",
  "required": true,
  "withImage": true,
  "sortOrder": 2,
  "config": {}
}
```

- `questionId`（业务键）由后端自动生成，无需传入
- `allowSkip` 固定 `false`，无需传入
- `categoryId` 不存在 → `1002`
- `step` 自动跟随分类 stepNumber，无需传
- 同分类下 `questionText` 重复 → `1001`
- 成功返回含 id 的对象

#### `PUT /api/requirement/questions/{id}` — 修改

Body 同新增。⚠️ **受保护字段 `questionId`/`categoryId`/`step` 即使传入也会被忽略**。
- `order` 非正整数 → `1006`

**题目上移/下移**：调用两次 `PUT` 交换两个题目的 `order`：

```bash
# 示例：题目 1（order=1）和题目 2（order=2）互换
curl -X PUT http://localhost:37050/api/requirement/questions/1 -H "Content-Type: application/json" \
  -d '{"order": 2}'
curl -X PUT http://localhost:37050/api/requirement/questions/2 -H "Content-Type: application/json" \
  -d '{"order": 1}'
```

#### `DELETE /api/requirement/questions/{id}` — 删除（逻辑删除）

返回 `{ "id": <id> }`。题目下的选项保留（前端读取时空数组）。

---

### 选项（2 个端点）

#### `GET /api/requirement/questions/{qid}/options` — 选项列表

返回 `OptionVO[]`，按 `order` 升序。

```json
{
  "code": 0,
  "data": [
    { "id": 1, "questionId": 1, "value": "rough_new", "label": "毛坯新房", "description": "", "imageUrl": "", "recommendationText": "", "allowCustom": false, "order": 1 }
  ],
  "message": null
}
```

#### `PUT /api/requirement/questions/{qid}/options` — 批量替换

**整体替换**该题的所有选项（先逻辑删旧的，再插入新的）。Body：

```json
{
  "options": [
    { "value": "rough_new", "label": "毛坯新房", "allowCustom": false, "order": 1 },
    { "value": "other", "label": "其他", "allowCustom": true, "order": 2 }
  ]
}
```

- `qid` 不存在 → `404`
- 返回新选项数量（int）

---

## curl 速查

```bash
# 字典
curl "http://localhost:37050/api/requirement/options"

# 分类
curl "http://localhost:37050/api/requirement/categories"
curl -X POST http://localhost:37050/api/requirement/categories -H "Content-Type: application/json" \
  -d '{"name":"基础信息","stepNumber":1,"description":"房屋基础信息","order":1}'
curl -X PUT http://localhost:37050/api/requirement/categories/1 -H "Content-Type: application/json" \
  -d '{"name":"基础信息(改)","stepNumber":1,"order":1}'
curl -X DELETE http://localhost:37050/api/requirement/categories/1
curl "http://localhost:37050/api/requirement/categories/1/questions"

# 题目
curl "http://localhost:37050/api/requirement/questions?categoryId=1"
curl -X POST http://localhost:37050/api/requirement/questions -H "Content-Type: application/json" \
  -d '{"questionId":"q7_special","categoryId":2,"questionText":"特殊需求？","questionType":"single","required":true,"order":2,"config":{}}'
curl -X PUT http://localhost:37050/api/requirement/questions/7 -H "Content-Type: application/json" \
  -d '{"questionText":"特殊需求(改)","questionType":"multi","config":{"maxSelect":3}}'
curl -X DELETE http://localhost:37050/api/requirement/questions/7

# 选项
curl "http://localhost:37050/api/requirement/questions/1/options"
curl -X PUT http://localhost:37050/api/requirement/questions/1/options -H "Content-Type: application/json" \
  -d '{"options":[{"value":"a","label":"选项A","order":1},{"value":"other","label":"其他","allowCustom":true,"order":2}]}'
```

## 接口清单（14 个）

| 分组 | # | 方法 | 路径 | 功能 |
|---|---|---|---|---|
| 字典 | 1 | GET | `/api/requirement/options` | 步骤/题型映射 |
| 分类 | 2 | GET | `/api/requirement/categories` | 列表 |
|  | 3 | GET | `/api/requirement/categories/{id}` | 详情 |
|  | 4 | POST | `/api/requirement/categories` | 新增 |
|  | 5 | PUT | `/api/requirement/categories/{id}` | 修改 |
|  | 6 | DELETE | `/api/requirement/categories/{id}` | 删除（1005 级联拒绝） |
|  | 7 | GET | `/api/requirement/categories/{id}/questions` | 题目数 |
| 题目 | 8 | GET | `/api/requirement/questions` | 列表 |
|  | 9 | GET | `/api/requirement/questions/{id}` | 详情 |
|  | 10 | POST | `/api/requirement/questions` | 新增 |
|  | 11 | PUT | `/api/requirement/questions/{id}` | 修改（保护 questionId/categoryId/step） |
|  | 12 | DELETE | `/api/requirement/questions/{id}` | 删除 |
| 选项 | 13 | GET | `/api/requirement/questions/{qid}/options` | 列表 |
|  | 14 | PUT | `/api/requirement/questions/{qid}/options` | 批量替换 |

> 📋 小程序端文档：[requirement-c.md](./requirement-c.md)（5 个 C 端端点：config / submit / results / results/{id} / DELETE）
