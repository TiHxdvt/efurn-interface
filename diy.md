# AI轻设计（三维家）接口文档

> 通用约定（响应格式 / 分页 / 鉴权 / 错误码 / 数据类型）见 [00-公共约定.md](./00-公共约定.md)。

**路径前缀**：`/api/diy`

本模块对接三维家开放平台，实现企业免密登录 → 跳转轻设计 H5。用户登录小程序时自动创建并绑定三维家账号。

---

## 整体流程

```
小程序登录 → 自动创建三维家账号+绑定 → DIY页 → GET /api/diy/login-url → web-view加载免密登录URL → 跳转轻设计H5
```

**登录时自动动作**（`WxUserServiceImpl.wechatLogin`）：
1. 调三维家创建账号接口（outUserId = openId）
2. 调三维家绑定账号接口
3. 失败不影响登录（try-catch 兜底）

---

## 配置（Nacos `efurn-wechat-*.yaml`）

```yaml
sanweijia:
  app-id: 2619602550074        # 三维家应用ID
  app-secret: xxx              # 凭证方式换取 access_token 的密钥
  dept-id: 865295400398110729  # 账号所属部门ID（创建账号必填）
  redirect-uri: https://pre-layoutai.3vjia.com/editor/h5  # 轻设计H5地址（预发布）
```

> ⚠️ 正式环境 redirect-uri 改为 `https://layoutai.3vjia.com/editor/h5`

---

## 三维家认证方式

采用**凭证方式（Access-Token）**：
1. 用 App ID + App Secret 调 `https://graph.3vjia.com/oauth/token` 获取 access_token（有效期 2h）
2. token 缓存在 Redis（key `sanweijia:access_token`，留 100s 余量）
3. 业务接口请求头带 `Access-Token: <token>`

---

## 接口详情

### 1. 获取免密登录URL `GET /api/diy/login-url`

**需登录态**（携带 `wxBearer` token），从 token 取当前用户 openId 作为免密登录的 userid。

**响应**：
```json
{
  "code": 0,
  "data": {
    "loginUrl": "https://sso.3vjia.com/JointLogin/Index?userid=xxx&appid=xxx&time=xxx&sign=xxx&redirect_uri=https%3A%2F%2Fpre-layoutai.3vjia.com%2Feditor%2Fh5"
  },
  "message": null
}
```

**免密登录URL参数**：
| 参数 | 说明 |
|---|---|
| `userid` | 企业系统用户ID（= openId，已绑定三维家账号） |
| `appid` | 三维家应用ID |
| `time` | Unix 时间戳（10位，±5分钟） |
| `sign` | `MD5(userid + appid + time + appSecret)` |
| `redirect_uri` | 轻设计H5地址（urlencode） |

> 📌 小程序用 `web-view` 加载 loginUrl 即可免密跳转。

---

## 小程序对接

- `pages/diy/index.vue` — web-view 加载 loginUrl
- `api/modules/diy.ts` — `getDiyLoginUrl()`

---

## 接口清单（1 个）

| # | 方法 | 路径 | 功能 |
|---|---|---|---|
| 1 | GET | `/api/diy/login-url` | 获取三维家免密登录URL（从登录态取 userid） |

---

## 注意事项

1. **小程序后台配置 web-view 业务域名白名单**：`sso.3vjia.com`、`pre-layoutai.3vjia.com`
2. **轻设计权限**：需联系三维家开通轻设计模块访问权限
3. **token 前缀**：小程序用 `wxBearer`，PC 后台用 `Bearer`，后端 `SecurityUtils` 同时支持
4. **三维家账号用户名规则**：`efurn_` + openId 后 8 位
