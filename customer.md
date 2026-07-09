# 客户管理接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/customer`

PC 端客户管理，含 CRUD + 字典。

---

## 数据模型

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `id` | Long | 否 | 主键 |
| `name` | String | 否 | 客户姓名 |
| `phone` | String | 否 | 手机号（唯一） |
| `source` | String | 否 | 来源（见字典） |
| `status` | String | 否 | 状态（见字典） |
| `province` | String | 是 | 省份编码 |
| `city` | String | 是 | 城市编码 |
| `region` | String | 是 | 区域编码 |
| `assignedTo` | String | 是 | 跟进人 |
| `wechatOpenid` | String | 是 | 微信 openid |
| `remark` | String | 是 | 备注 |
| `lastFollowedAt` | String(Date) | 是 | 最后跟进时间 |
| `createdAt` | String(Date) | 否 | 创建时间 |
| `updatedAt` | String(Date) | 否 | 更新时间 |

---

## 字典

### `GET /api/customer/options`

```json
{
  "code": 0,
  "data": {
    "sourceMap": {
      "mini_register": "业主注册",
      "mini_quote": "预约报价",
      "mini_assess": "需求测评",
      "manual": "手动新增"
    },
    "statusMap": {
      "potential": "潜在",
      "following": "跟进中",
      "won": "已成交",
      "lost": "已流失"
    },
    "assigneeOptions": ["刘销售", "王销售", "张设计师"]
  }
}
```

---

## 接口详情

### 1. 列表 `GET /api/customer`

| Query | 类型 | 说明 |
|---|---|---|
| `page` / `pageSize` | int | 默认 20 |
| `name` | String | 姓名模糊 |
| `phone` | String | 手机号模糊 |
| `source` | String | 来源精确 |
| `status` | String | 状态精确 |
| `city` | String | 城市精确 |
| `region` | String | 区域精确 |
| `assignedTo` | String | 跟进人精确 |
| `createdAtStart` / `createdAtEnd` | String | 创建时间范围（ISO 格式） |

排序：`createdAt` 降序。

### 2. 详情 `GET /api/customer/{id}`

### 3. 新增 `POST /api/customer`

Body：`{ name, phone, source?, status?, province?, city?, region?, assignedTo?, wechatOpenid?, remark? }`

- phone 唯一校验 → `1001`
- 默认 `source=manual`，`status=potential`

### 4. 修改 `PUT /api/customer/{id}`

Body 同新增（全部可选）。phone 变更时校验唯一 → `1001`。不存在 → `404`。

### 5. 删除 `DELETE /api/customer/{id}`（逻辑删除）

返回 `{ "id": <id> }`。

---

## curl 速查

```bash
curl "http://localhost:37050/api/customer?page=1&pageSize=10&status=potential"
curl "http://localhost:37050/api/customer/options"
curl "http://localhost:37050/api/customer/1"
curl -X POST http://localhost:37050/api/customer -H "Content-Type: application/json" \
  -d '{"name":"张三","phone":"13800000001","source":"manual","status":"potential"}'
curl -X PUT http://localhost:37050/api/customer/1 -H "Content-Type: application/json" \
  -d '{"status":"following","assignedTo":"刘销售"}'
curl -X DELETE http://localhost:37050/api/customer/1
```

## 接口清单（6 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/customer` | 列表（多维筛选 + 分页） |
| 2 | GET | `/api/customer/options` | 字典（来源/状态/跟进人） |
| 3 | GET | `/api/customer/{id}` | 详情 |
| 4 | POST | `/api/customer` | 新增 |
| 5 | PUT | `/api/customer/{id}` | 修改 |
| 6 | DELETE | `/api/customer/{id}` | 逻辑删除 |
