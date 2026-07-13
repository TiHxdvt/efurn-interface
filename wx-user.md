# 微信小程序用户接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/wx-user`（⚠️ 注意：本模块路径不在 `/api/` 下，需在网关白名单放行）

微信小程序用户登录 / 登出。登录流程：小程序端调用 `wx.login()` 拿到 `code` + 用户填写昵称 → 后端通过 `code` 换取 `openid` → 查库判断是否新用户 → 注册/返回用户信息。

---

## 响应格式

本模块使用 `R<T>` 响应格式（主框架）：

| 对比 | 本模块 `R<T>` | 业务模块 `Result<T>` |
|---|---|---|
| 成功码 | `200` | `0` |
| 消息字段 | `msg` | `message` |
| 完整响应 | `{ code: 200, data: T, msg: null }` | `{ code: 0, data: T, message: null }` |

---

## 数据模型

### WxLoginModel（登录请求体）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | String | 是 | 微信登录 code（`wx.login()` 获取） |
| `nickName` | String | 否 | 用户昵称（前端通过 `<input type="nickname">` 获取） |
| `avatarUrl` | String | 否 | 头像 URL |

> 📌 `nickName` 为空时后端默认填 `"微信用户"`，保证后续业务字段不为空。

### WxLoginUser（登录响应）

| 字段 | 类型 | 说明 |
|---|---|---|
| `openId` | String | 微信 openid |
| `username` | String | 用户名（= wxNickname） |

### WxUser（数据库实体）

| 字段 | 类型 | 说明 |
|---|---|---|
| `openId` | String | 微信 openid（主键） |
| `wxNickname` | String | 用户昵称 |
| `wxAvatarUrl` | String | 头像 URL |
| `phoneNum` | String | 手机号（后续绑定手机号功能填充） |
| `status` | String | 状态：`0` 正常 / `1` 禁用 |
| `createTime` | LocalDateTime | 创建时间 |
| `delFlag` | String | 删除标识 |

---

## 接口详情

### 1. 微信登录 `POST /wx-user/login`

通过 `code` 换取 `openid`，自动注册新用户，返回用户信息。

**请求**：`@RequestBody WxLoginModel`

```json
{
  "code": "wx.login()获取的code",
  "nickName": "用户填写的昵称",
  "avatarUrl": "头像URL（可空）"
}
```

**登录流程**：
1. 用 `code` 调用微信 `jscode2session` 接口 → 获取 `openid`
2. 查 `wx_user` 表，判断是否已注册
3. 未注册 → 插入新用户（昵称默认"微信用户"）
4. 已注册 → 同步更新昵称/头像
5. 返回 `WxLoginUser`（含 `openId` 和 `username`）

```bash
curl -X POST http://localhost:37050/wx-user/login \
  -H "Content-Type: application/json" \
  -d '{"code":"TEST_CODE","nickName":"张三"}'
```

**响应**：
```json
{
  "code": 200,
  "data": {
    "openId": "oXXXXXXX",
    "username": "张三"
  },
  "msg": null
}
```

**错误响应**：
- `code` 为空：`{ "code": 500, "msg": "登录code不能为空" }`
- 微信接口失败：`{ "code": 500, "msg": "微信登录失败：{errmsg}" }`

---

### 2. 微信回调 `POST /wx-user/callback`

微信服务器回调通知接口（预留）。

```bash
curl -X POST http://localhost:37050/wx-user/callback
```

---

### 3. 用户登出 `GET /wx-user/auth/logout`

| 参数 | 位置 | 类型 | 必填 | 说明 |
|---|---|---|---|---|
| `userId` | Query | Long | 是 | 用户 ID |

```bash
curl "http://localhost:37050/wx-user/auth/logout?userId=1"
```

---

## 前端调用示例

```typescript
// 1. wx.login() 获取 code
const { code } = await wx.login()

// 2. 获取用户昵称（通过 <input type="nickname">）
const nickName = '用户填写的昵称'

// 3. 调用登录接口
const res = await wx.request({
  url: 'http://10.113.192.106:37050/wx-user/login',
  method: 'POST',
  data: { code, nickName }
})

// 4. 保存 openId 和 username
if (res.data.code === 200) {
  const { openId, username } = res.data.data
  // 存入本地，后续请求通过 X-User-Id 头传递
}
```

---

## 数据库表（`wx_user`）

```sql
CREATE TABLE wx_user (
    open_id       VARCHAR(64)  NOT NULL PRIMARY KEY COMMENT '微信openid',
    wx_nickname   VARCHAR(100) DEFAULT '微信用户'   COMMENT '用户昵称',
    wx_avatar_url VARCHAR(500) DEFAULT ''           COMMENT '头像URL',
    phone_num     VARCHAR(20)  DEFAULT NULL         COMMENT '手机号',
    status        VARCHAR(1)   DEFAULT '0'          COMMENT '状态 0正常 1禁用',
    create_time   TIMESTAMP    DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    del_flag      SMALLINT     DEFAULT 0            COMMENT '删除标识'
);
```

---

## 跨服务调用（Feign）

Auth 服务通过 Feign 调用本接口获取用户信息，用于签发 JWT token：

```java
@FeignClient(value = "efurn-wechat", fallbackFactory = RemoteWxUserFallbackFactory.class)
public interface RemoteWxUserService {
    @PostMapping("/wx-user/login")
    R<WxLoginUser> getUserInfo(@RequestParam String code);
}
```

---

## curl 速查

```bash
# 登录（JSON Body）
curl -X POST http://localhost:37050/wx-user/login \
  -H "Content-Type: application/json" \
  -d '{"code":"TEST_CODE","nickName":"张三"}'

# 回调
curl -X POST http://localhost:37050/wx-user/callback

# 登出
curl "http://localhost:37050/wx-user/auth/logout?userId=1"
```

## 接口清单（3 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | POST | `/wx-user/login` | 微信登录（code → openid → 注册/返回用户信息） |
| 2 | POST | `/wx-user/callback` | 微信回调通知（预留） |
| 3 | GET | `/wx-user/auth/logout` | 用户登出 |
