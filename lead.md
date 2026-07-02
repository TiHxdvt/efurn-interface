# 线索管理接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/lead`

本模块含线索主表 + 动作记录 + 跟进记录子表 + 转客户联动，共 12 个端点。

---

## 数据模型

### Lead 线索

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 客户姓名 |
| `phone` | String | 否 | 手机号（**跨线索+客户唯一**） |
| `source` | String | 否 | 来源（见字典） |
| `status` | String | 否 | `pending`(待跟进) / `following`(跟进中) / `converted`(已转客户) / `lost`(已流失) |
| `assignedTo` | String | 是 | 跟进人 |
| `remark` | String | 是 | 备注 |
| `lastActionAt` | String(Date) | 是 | 最后动作时间 |
| `lastFollowedAt` | String(Date) | 是 | 最后跟进时间 |
| `customerId` | Long | 是 | 转客户后的客户 id |
| `createdAt` / `updatedAt` | String(Date) | 否 | 时间 |

### Follow 跟进记录（人工，可增改删）

`id, leadId, content, channel, nextFollowAt, createdBy, createdAt`

### Action 动作记录（系统，只增不改删）

`id, leadId, type, description, meta(Object), createdAt`

---

## 字典（`GET /api/lead/options`）

```json
{
  "statuses": ["pending","following","converted","lost"],
  "sources": ["mini_register","mini_quote","mini_assess","douyin_im","xiaohongshu_comment","manual"],
  "actionTypes": ["mini_register","mini_quote","mini_assess","douyin_im","xiaohongshu_comment","manual_create"],
  "followChannels": ["phone","wechat","visit","other"],
  "assignees": ["刘销售","王销售","张设计师"],
  "statusMap": { "pending": "待跟进", "following": "跟进中", "converted": "已转客户", "lost": "已流失" },
  "sourceMap": { "mini_register": "业主注册", "mini_quote": "预约报价", "manual": "手动新增", ... },
  "actionTypeMap": { ... },
  "followChannelMap": { "phone": "电话", "wechat": "微信", "visit": "到店", "other": "其他" }
}
```

---

## 业务错误码

| code | 含义 |
|---|---|
| `1001` | 手机号已是线索/客户（新增/改手机号/转客户校验） |
| `1002` | 该手机号已是客户（convert 时） |
| `1003` | 终态不可改（converted/lost 不可编辑、不可转客户） |
| `1004` | 必填项为空（手机号、跟进内容） |
| `404` | 线索/跟进记录不存在 |

---

## 接口详情

### 线索 CRUD

#### 1. 列表 `GET /api/lead`

| Query | 说明 |
|---|---|
| `page` / `pageSize` | 默认 20 |
| `name` / `phone` | 模糊 |
| `status` / `source` / `assignedTo` | 精确 |
| `lastFollowedStart` / `lastFollowedEnd` | 最后跟进时间范围 |
| `lastActionStart` / `lastActionEnd` | 最后动作时间范围 |
| `createdAtStart` / `createdAtEnd` | 创建时间范围 |

排序：`createdAt` 降序。

#### 2. 详情 `GET /api/lead/{id}`

#### 3. 新增 `POST /api/lead`
Body：`{ name?, phone, source?, assignedTo?, remark? }`
- phone 必填 → `1004`
- phone 跨线索+客户唯一 → `1001`
- `source=manual` 时自动插一条 `manual_create` 动作并置 `lastActionAt`
- 新建 `status=pending`

#### 4. 编辑 `PUT /api/lead/{id}`
- 终态（converted/lost）→ `1003`
- 改 phone 时校验唯一 → `1001`
- **不能改** status / lastFollowedAt / lastActionAt / customerId（系统维护）

#### 5. 删除 `DELETE /api/lead/{id}`（逻辑删除）→ `{ "id": <id> }`

### 动作 / 跟进

#### 6. 动作列表 `GET /api/lead/{id}/actions` → `ActionVO[]`（按时间倒序）

#### 7. 跟进列表 `GET /api/lead/{id}/follows` → `FollowVO[]`（按时间倒序）

#### 8. 新增跟进 `POST /api/lead/{id}/follows`
Body：`{ content, channel?, nextFollowAt? }`
- content 必填 → `1004`
- **副作用**：更新线索 `lastFollowedAt`；`pending` 自动转 `following`

#### 9. 编辑跟进 `PUT /api/lead/{id}/follows/{followId}`

#### 10. 删除跟进 `DELETE /api/lead/{id}/follows/{followId}` → `{ "id": <followId> }`

### 转客户

#### 11. 转客户 `POST /api/lead/{id}/convert`（**事务**）
Body：`{ province, city, region }`

逻辑：校验非终态（`1003`）+ 手机号非客户（`1002`）→ 写入 `customer_info`（status=`potential`）→ 线索 `status=converted` + `customerId` → 返回 `{ lead, customer }`

```bash
curl -X POST http://localhost:37050/api/lead/2/convert -H "Content-Type: application/json" \
  -d '{"province":"SH","city":"SH","region":"Pudong"}'
```

---

## curl 速查

```bash
curl "http://localhost:37050/api/lead/options"
curl "http://localhost:37050/api/lead?page=1&pageSize=10&status=pending"
curl -X POST http://localhost:37050/api/lead -H "Content-Type: application/json" \
  -d '{"name":"周杰","phone":"13900138099","source":"manual","assignedTo":"刘销售"}'
curl -X PUT http://localhost:37050/api/lead/4 -H "Content-Type: application/json" \
  -d '{"remark":"已联系"}'
curl -X POST http://localhost:37050/api/lead/4/follows -H "Content-Type: application/json" \
  -d '{"content":"电话沟通，约周末到店","channel":"phone","nextFollowAt":"2026-07-05 10:00:00"}'
curl "http://localhost:37050/api/lead/1/follows"
curl -X POST http://localhost:37050/api/lead/2/convert -H "Content-Type: application/json" \
  -d '{"province":"SH","city":"SH","region":"Pudong"}'
```

## 接口清单（12 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/lead/options` | 字典 |
| 2 | GET | `/api/lead` | 列表（多维筛选） |
| 3 | GET | `/api/lead/{id}` | 详情 |
| 4 | POST | `/api/lead` | 新增（manual 自动动作） |
| 5 | PUT | `/api/lead/{id}` | 编辑（终态不可改） |
| 6 | DELETE | `/api/lead/{id}` | 删除 |
| 7 | GET | `/api/lead/{id}/actions` | 动作列表 |
| 8 | GET | `/api/lead/{id}/follows` | 跟进列表 |
| 9 | POST | `/api/lead/{id}/follows` | 新增跟进（副作用：lastFollowedAt + 转跟进中） |
| 10 | PUT | `/api/lead/{id}/follows/{followId}` | 编辑跟进 |
| 11 | DELETE | `/api/lead/{id}/follows/{followId}` | 删除跟进 |
| 12 | POST | `/api/lead/{id}/convert` | 转客户（事务，写 customer_info） |

> 📋 关联：`customer_info` 表为最小版（转客户写入），客户管理模块后续扩展。
