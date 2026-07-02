# 预约记录接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

业主小程序提交报价预约，PC 后台查看/跟进。PC 端管理与小程序端共用 `/api/reservation`。

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 姓名 |
| `phone` | String | 否 | 手机号（11 位） |
| `rooms` | Object | 否 | 户型 `{shi, ting, wei, chu}`，各值 0-10 |
| `area` | Integer | 否 | 房屋面积（㎡） |
| `budget` | String | 否 | 预算区间（10万以下 / 10-20万 / 20-30万 / 30万以上） |
| `remark` | String | 是 | 备注（≤100 字） |
| `status` | String | 否 | 状态：`pending`(待联系) / `contacted`(已联系) / `measured`(已量房) |
| `wechatUserId` | Long | 是 | 提交者微信用户 id（登录态关联，可空） |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

**`rooms` 示例**：`{ "shi": 3, "ting": 2, "wei": 1, "chu": 1 }`（3 室 2 厅 1 卫 1 厨）

---

## 业务错误码

| code | 含义 |
|---|---|
| `1400` | 参数校验失败（户型数值非 0-10、状态非法等） |
| `404` | 预约记录不存在 |

---

## 接口详情

### PC 端管理

#### 1. 列表 `GET /api/reservation`

| Query | 类型 | 说明 |
|---|---|---|
| `page` / `pageSize` | int | 默认 20 |
| `name` | String | 姓名模糊 |
| `phone` | String | 手机号模糊 |
| `status` | String | `pending` / `contacted` / `measured` |

排序：`createdAt` 降序（最新优先）。

```bash
curl "http://localhost:37050/api/reservation?page=1&pageSize=10&status=pending"
```

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1, "name": "张先生", "phone": "13800138001",
        "rooms": { "shi": 3, "ting": 2, "wei": 1, "chu": 1 },
        "area": 150, "budget": "20-30万", "remark": "希望使用环保材料",
        "status": "measured", "wechatUserId": null,
        "createdAt": "2026-07-02T10:00:00", "updatedAt": "2026-07-02T10:00:00"
      }
    ],
    "total": 5, "page": 1, "pageSize": 10
  },
  "message": null
}
```

#### 2. 详情 `GET /api/reservation/{id}`

不存在 → `404`。

#### 3. 更新状态 `PUT /api/reservation/{id}/status`

Body：`{ "status": "contacted" }`

- status 非 pending/contacted/measured → `1400`
- 不存在 → `404`
- 成功返回更新后的预约对象

```bash
curl -X PUT http://localhost:37050/api/reservation/3/status -H "Content-Type: application/json" \
  -d '{"status":"contacted"}'
```

#### 4. 删除 `DELETE /api/reservation/{id}`（逻辑删除）

返回 `{ "id": <id> }`。

---

### 小程序端

#### 5. 提交预约 `POST /api/reservation`

Header（可选）：`X-User-Id: <微信用户 id>`（登录态关联提交者）

Body：
```json
{
  "name": "张先生",
  "phone": "13800138001",
  "rooms": { "shi": 3, "ting": 2, "wei": 1, "chu": 1 },
  "area": 150,
  "budget": "20-30万",
  "remark": "希望使用环保材料"
}
```

校验：name/phone/rooms/area/budget 必填；phone 11 位手机号；rooms 各值 0-10；remark ≤100 字。
新增后 `status` 固定 `pending`。成功返回 `{ "id": <id> }`。

#### 6. 我的预约 `GET /api/my-reservations`

Header：`X-User-Id: <微信用户 id>`

返回该用户提交的所有预约（按 `createdAt` 降序）。未传 `X-User-Id` 返回空数组。

```bash
curl -H "X-User-Id: 1" "http://localhost:37050/api/my-reservations"
```

---

## curl 速查

```bash
# PC 端
curl "http://localhost:37050/api/reservation?page=1&pageSize=10&status=pending"
curl "http://localhost:37050/api/reservation/1"
curl -X PUT http://localhost:37050/api/reservation/1/status -H "Content-Type: application/json" -d '{"status":"contacted"}'
curl -X DELETE http://localhost:37050/api/reservation/1

# 小程序端
curl -X POST http://localhost:37050/api/reservation -H "Content-Type: application/json" \
  -d '{"name":"张先生","phone":"13800138001","rooms":{"shi":3,"ting":2,"wei":1,"chu":1},"area":150,"budget":"20-30万","remark":"环保材料"}'
curl -H "X-User-Id: 1" "http://localhost:37050/api/my-reservations"
```

## 接口清单（6 个）

| 端 | # | 方法 | 路径 | 功能 |
|---|---|---|---|---|
| PC | 1 | GET | `/api/reservation` | 列表（name/phone/status） |
| PC | 2 | GET | `/api/reservation/{id}` | 详情 |
| PC | 3 | PUT | `/api/reservation/{id}/status` | 更新状态 |
| PC | 4 | DELETE | `/api/reservation/{id}` | 删除 |
| 小程序 | 5 | POST | `/api/reservation` | 提交预约 |
| 小程序 | 6 | GET | `/api/my-reservations` | 我的预约 |

> 📋 待对接：小程序登录态 token → 自动解析 `wechatUserId`（当前用 `X-User-Id` 头联调）
