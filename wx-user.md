# 微信小程序用户接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/wx-user`（⚠️ 注意：本模块路径不在 `/api/` 下，需在网关白名单放行）

微信小程序用户登录 / 登出。登录流程：小程序端调用 `wx.login()` 拿到 `code` → 后端通过 `code` 换取 `openid` → 查库判断是否新用户 → 注册/返回 token。

---

## ⚠️ 当前状态：基本可用

| 部分 | 状态 | 说明 |
|---|---|---|
| Controller | ✅ 已完成 | 3 个端点已定义 |
| Entity / DTO / VO | ✅ 已完成 | `WxUser` / `WxLoginModel` / `WxLoginVO` / `WxLoginUser` |
| Service 实现 | ✅ 基本可用 | 登录流程可工作（code → openid → 注册/查库 → 返回用户信息） |
| 数据库表 | ✅ 已完成 | `wx_user` |
| 网关白名单 | ✅ 已配置 | `/wx-user/login` 和 `/wx-user/callback` 已放行 |

**本模块使用 `R<T>` 响应格式**（主框架），注意与 wechat 业务模块的 `Result<T>` 不同：

| 对比 | 本模块 `R<T>` | 业务模块 `Result<T>` |
|---|---|---|
| 成功码 | `200` | `0` |
| 消息字段 | `msg` | `message` |
| 完整响应 | `{ code: 200, data: T, msg: null }` | `{ code: 0, data: T, message: null }` |

> 📋 **TODO**：统一为 `Result<T>`（code=0），与其它 wechat 接口一致。当前保留 `R<T>` 是因为与 auth 服务 token 签发链路耦合。

---

## 数据模型

### WxUser（数据库实体）

| 字段 | 类型 | 可空 | 说明 |
|---|---|---|---|
| `openId` | String | 否 | 微信 openid（**主键**） |
| `wxNickname` | String | 是 | 微信用户昵称 |
| `wxAvatarUrl` | String | 是 | 微信头像 URL |
| `phoneNum` | String | 是 | 手机号（绑定后填充） |
| `status` | String | 是 | 状态：`0` 正常 / `1` 禁用 |
| `createTime` | LocalDateTime | 是 | 创建时间 |
| `delFlag` | String | 是 | 删除标识 |

> 📌 `openId` 作为主键（非自增 Long），来自微信 `jscode2session` 接口。

### WxLoginModel（登录请求体，当前未使用）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | String | 是 | 微信登录 code（`wx.login()` 获取） |
| `nickName` | String | 否 | 用户昵称 |
| `avatarUrl` | String | 否 | 头像 URL |
| `gender` | Integer | 否 | 性别：0 未知 / 1 男 / 2 女 |

> 📌 当前 Controller 用 `@RequestParam String code` 接收，未使用 `@RequestBody WxLoginModel`。

### WxLoginUser（登录响应，跨服务模型）

| 字段 | 类型 | 说明 |
|---|---|---|
| `cacheKey` | String | Redis 缓存 key |
| `openId` | String | 微信 openid |
| `username` | String | 用户名（= wxNickname） |
| `status` | String | 用户状态 |
| `avatarUrl` | String | 头像 URL |
| `loginTime` | Long | 登录时间戳 |
| `expireTime` | Long | 过期时间戳 |

### WxLoginVO（登录响应 VO，当前未使用）

| 字段 | 类型 | 说明 |
|---|---|---|
| `userId` | Long | 用户 ID |
| `nickName` | String | 昵称 |
| `avatarUrl` | String | 头像 |
| `token` | String | 访问令牌 |
| `tokenType` | String | 令牌类型（`Bearer`） |
| `expiresIn` | Long | 过期时间（秒，默认 7200） |
| `isNewUser` | Boolean | 是否新注册用户 |

---

## 业务错误码

| code | 含义 | 触发场景 |
|---|---|---|
| `500` | 登录 code 为空 | 未传 `code` 参数 |
| `500` | 微信登录失败 | 调用微信 `jscode2session` 接口失败 |
| `500` | 获取微信 openid 失败 | 微信返回的 openid 为空 |
| `500` | 微信登录失败：{errmsg} | 微信返回业务错误 |

> 📋 TODO：统一为模块专属业务码（如 `1501` 微信登录失败），当前直接抛 `ServiceException` 走全局 `500`。

---

## 接口详情

### 1. 微信登录 `POST /wx-user/login`

小程序端核心登录接口。通过 `code` 换取 `openid`，自动注册新用户，返回 token。

| 参数 | 位置 | 类型 | 必填 | 说明 |
|---|---|---|---|---|
| `code` | Query | String | 是 | 微信 `wx.login()` 获取的 code |

**登录流程**：
1. 用 `code` 调用微信 `jscode2session` 接口 → 获取 `openid` + `session_key`
2. 查 `wx_user` 表，判断是否已注册
3. 未注册 → 插入新用户记录
4. 已注册 → 更新登录信息
5. 生成 token（UUID），存入 Redis（过期时间 7200s）
6. 返回 `WxLoginUser`（含 token）

```bash
curl -X POST "http://localhost:37050/wx-user/login?code=TEST_LOGIN_CODE"
```

**响应**：
```json
{
  "code": 200,
  "data": {
    "openId": "oXXXXXXX",
    "username": "微信用户"
  },
  "msg": null
}
```

---

### 2. 绑定手机号 `POST /wx-user/bindPhone`

小程序端通过微信 `getPhoneNumber` 授权拿到 code，后端调微信接口换取真实手机号并写入 `wx_user.phone_num`。**需登录态**（携带 `wxBearer` token）。

| 参数 | 位置 | 类型 | 必填 | 说明 |
|---|---|---|---|---|
| `code` | Query | String | 是 | `<button open-type="getPhoneNumber">` 回调拿到的 code |

**流程**：
1. 从 token 取当前登录用户 openId
2. 获取微信 access_token（Redis 缓存，7000s）
3. 用 code + access_token 调微信 `getuserphonenumber` 接口
4. 更新 `wx_user.phone_num`
5. 返回手机号字符串

```bash
curl -X POST "http://localhost:37050/wx-user/bindPhone?code=xxxx" \
  -H "Authorization: wxBearer <token>"
```

**响应**：`{ "code": 200, "data": "13800138000", "msg": null }`

> 📌 access_token 用 Redis key `wx:access_token` 缓存，避免重复调用微信接口。

---

### 3. 微信回调 `POST /wx-user/callback`

微信服务器回调通知接口（预留，当前返回空 `R.ok()`）。

```bash
curl -X POST http://localhost:37050/wx-user/callback
```

> 📌 用途：接收微信推送的事件通知（如用户信息更新、手机号绑定等），具体逻辑待实现。

---

### 4. 用户登出 `GET /wx-user/auth/logout`

小程序端用户登出。

| 参数 | 位置 | 类型 | 必填 | 说明 |
|---|---|---|---|---|
| `userId` | Query | Long | 是 | 用户 ID |

```bash
curl "http://localhost:37050/wx-user/auth/logout?userId=1"
```

**响应**：`{ "code": 200, "data": null, "msg": null }`

> 📌 当前仅返回空成功，完工后应清除 Redis 中的 token。

---

### 5. 获取个人信息 `GET /wx-user/info`

获取当前登录用户的个人信息。**需登录态**（携带 `wxBearer` token）。

```bash
curl "http://localhost:37050/wx-user/info" \
  -H "Authorization: wxBearer <token>"
```

**响应**：
```json
{
  "code": 200,
  "data": {
    "id": "oXXXXXXX",
    "nickname": "张三",
    "avatar": "https://cdn.xxx.com/avatar.jpg",
    "phone": "13800138000"
  },
  "msg": null
}
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 用户 ID（openId） |
| `nickname` | String | 昵称 |
| `avatar` | String | 头像 URL |
| `phone` | String | 手机号（未绑定时为 null） |

---

### 6. 更新个人信息 `PUT /wx-user/info`

更新当前登录用户的昵称和头像。**需登录态**（携带 `wxBearer` token）。

**请求体**：
```json
{
  "nickname": "新昵称",
  "avatar": "https://cdn.xxx.com/new-avatar.jpg"
}
```

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `nickname` | String | 否 | 昵称，传空则不更新 |
| `avatar` | String | 否 | 头像 CDN URL，先调上传接口获取 |

```bash
curl -X PUT "http://localhost:37050/wx-user/info" \
  -H "Authorization: wxBearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"nickname":"新昵称","avatar":"https://cdn.xxx.com/avatar.jpg"}'
```

**响应**：同获取个人信息接口，返回更新后的完整用户信息。

---

## 数据库表（`wx_user`）

```sql
CREATE TABLE wx_user (
    open_id       VARCHAR(64)  NOT NULL PRIMARY KEY COMMENT '微信openid',
    wx_nickname   VARCHAR(100) DEFAULT NULL          COMMENT '用户昵称',
    wx_avatar_url VARCHAR(500) DEFAULT NULL          COMMENT '头像URL',
    phone_num     VARCHAR(20)  DEFAULT NULL          COMMENT '手机号',
    status        VARCHAR(1)   DEFAULT '0'           COMMENT '状态 0正常 1禁用',
    create_time   DATETIME     DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    del_flag      VARCHAR(1)   DEFAULT '0'           COMMENT '删除标识'
);
```

> ⚠️ SQL 文件 `wechat_user.sql` 为 PostgreSQL 语法（BIGSERIAL、COMMENT ON），MySQL 版需适配。且 SQL 文件表名/字段名与 Entity 不完全一致（SQL 用 `user_id` 主键，Entity 用 `openId` 主键），以 Entity 为准。

---

## 跨服务调用（Feign）

`RemoteWxUserService` 定义了 Feign 接口，供 auth 服务等远程调用：

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
# 登录
curl -X POST "http://localhost:37050/wx-user/login?code=TEST_LOGIN_CODE"

# 绑定手机号
curl -X POST "http://localhost:37050/wx-user/bindPhone?code=xxxx" -H "Authorization: wxBearer <token>"

# 回调
curl -X POST http://localhost:37050/wx-user/callback

# 登出
curl "http://localhost:37050/wx-user/auth/logout?userId=1"

# 获取个人信息
curl "http://localhost:37050/wx-user/info" -H "Authorization: wxBearer <token>"

# 更新个人信息
curl -X PUT "http://localhost:37050/wx-user/info" \
  -H "Authorization: wxBearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"nickname":"新昵称","avatar":"https://cdn.xxx.com/avatar.jpg"}'
```

## 接口清单（6 个）

| # | 方法 | 路径 | 功能 | 状态 |
|---|---|---|---|---|
| 1 | POST | `/wx-user/login` | 微信登录（code → openid → 注册/查库 → 自动建三维家账号） | ✅ 可用 |
| 2 | POST | `/wx-user/bindPhone` | 绑定手机号（getPhoneNumber code → 写 phone_num） | ✅ 可用 |
| 3 | POST | `/wx-user/callback` | 微信回调通知（预留） | ⏳ 空实现 |
| 4 | GET | `/wx-user/auth/logout` | 用户登出 | ⏳ 空实现 |
| 5 | GET | `/wx-user/info` | 获取个人信息 | ✅ 可用 |
| 6 | PUT | `/wx-user/info` | 更新个人信息 | ✅ 可用 |
