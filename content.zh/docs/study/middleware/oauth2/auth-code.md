---
title: 授权码模式
---

## 1. 用户授权与鉴权机制设计要点

### 授权 vs 鉴权

* **授权（Authorization）**：确定用户或客户端是否有权限访问某资源或执行某操作。
* **鉴权（Authentication）**：验证用户身份是否真实。

---

### 常见技术与方案

| 技术                   | 作用      | 说明                               |
| -------------------- | ------- | -------------------------------- |
| OAuth2               | 授权框架    | 支持第三方应用代表用户访问资源，常用于开放API和SSO场景。  |
| JWT (JSON Web Token) | 鉴权令牌    | 无状态Token，包含用户信息和权限，便于分布式系统鉴权。    |
| Session              | 鉴权状态管理  | 服务器端保存用户登录状态，适合单体或小型应用。          |
| Token刷新              | 保持鉴权状态  | 使用刷新Token获取新的访问Token，减少用户重新登录频率。 |
| 权限模型                 | 细粒度权限控制 | RBAC（基于角色）和ACL（基于访问控制列表）是常见模型。   |

---

## 2. 设计建议

* **认证使用OAuth2 + JWT**：
  OAuth2提供标准授权流程，JWT用于访问令牌，支持无状态鉴权和微服务调用。

* **Token设计**：

  * **Access Token**：有效期较短，携带用户身份及权限。
  * **Refresh Token**：有效期较长，用于获取新的Access Token。

* **权限控制**：

  * **RBAC**：用户分配角色，角色对应权限，简单易维护。
  * **ACL**：对资源和用户的访问权限做细粒度控制，适合复杂场景。

* **安全加固**：

  * HTTPS传输所有Token。
  * Token签名和加密。
  * 定期刷新Token。
  * 防止Token泄露和重放攻击。

---

## 3. OAuth2 授权码模式（Authorization Code Grant）详细步骤

授权码模式主要用于服务器端应用，流程如下：

1. **用户请求授权**
   用户在客户端（如浏览器）访问需要授权的功能，客户端将用户重定向到授权服务器的授权端点，附带客户端ID、回调地址、请求权限范围（scope）等参数。

2. **用户登录并授权**
   授权服务器要求用户登录（如果未登录），并请求用户同意客户端申请的权限范围。

3. **授权码返回客户端**
   用户同意后，授权服务器将授权码（Authorization Code）通过浏览器重定向发送到客户端预先注册的回调地址。

4. **客户端用授权码换取访问Token**
   客户端后台用授权码向授权服务器的Token端点请求Access Token和Refresh Token。请求需包含客户端ID和密钥，保证安全。

5. **授权服务器颁发Token**
   授权服务器验证授权码和客户端信息，颁发Access Token和Refresh Token给客户端。

6. **客户端携带Access Token访问资源服务器**
   客户端使用Access Token请求受保护的资源。资源服务器验证Token有效性并返回数据。

7. **Access Token过期后使用Refresh Token刷新**
   当Access Token过期，客户端用Refresh Token向授权服务器请求新的Access Token。

---

### 流程图简述：

```
用户 → 客户端 → 授权服务器 (请求授权码)
用户登录并授权
授权服务器 → 客户端 (授权码)
客户端 → 授权服务器 (换取Token)
授权服务器 → 客户端 (Access Token + Refresh Token)
客户端 → 资源服务器 (携带Access Token访问资源)
```

---

### 优点

* 授权码在浏览器传输，避免Token泄露。
* Access Token和Refresh Token分别管理，安全性高。
* 支持第三方应用安全访问资源。

---

## 4. 用户登录后，服务器将 Access Token 绑定到用户 Session，详细说明绑定和存储查找流程

### 一般流程

1. **用户登录成功后，服务器生成 Session ID**

   * 服务器创建一个唯一的 Session ID（通常是一个随机、难以猜测的字符串，比如 UUID）。
   * Session ID 通过 Set-Cookie 发送给客户端浏览器（Cookie 通常设置为 HttpOnly、Secure、SameSite）。

2. **服务器在 Session 存储中（如 Redis）建立 Session 记录**

   * 以 Session ID 作为 Key，存储 Session 相关数据，典型内容包括：

     * 用户ID（user\_id）
     * Access Token（JWT 或自定义 Token）
     * 用户权限/角色信息
     * 登录时间、过期时间等元数据

   例如，Redis 中的键值对：

   ```
   Key: session:<session_id>  
   Value: {
     "user_id": "12345",
     "access_token": "<JWT或其他token>",
     "roles": ["admin", "user"],
     "expire_at": "2025-06-30T12:00:00Z"
   }
   ```

3. **后续请求客户端携带 Session ID Cookie**

   * 服务器通过请求的 Cookie 获取 Session ID。
   * 利用 Session ID 查 Redis 获取用户身份和 Access Token。
   * 服务器根据 Access Token 验证权限或业务需求。

---

### 绑定关系的存储和查找

* **存储位置**：集中式 Session 存储（Redis、Memcached 或数据库）。
* **查找方式**：

  * HTTP 请求时，服务器从 Cookie 获取 Session ID。
  * 使用 Session ID 作为 Key，查询存储获取用户数据和 Access Token。
* **数据结构**：典型哈希结构存储用户会话信息，支持快速读写和过期管理。

---

## 5. 防止用户伪造 Session

### 如何保证 Session 不被伪造？

* **Session ID 的安全设计**

  * 生成方式随机且不可预测（使用安全随机数生成器）。
  * 长度足够，避免暴力破解。
  * 只通过 Cookie 发送，且设置 `HttpOnly` 和 `Secure` 标记，防止JS访问和劫持。

* **Cookie 传输安全**

  * 使用 HTTPS，防止中间人攻击抓包窃取 Session ID。
  * 设置 `SameSite=Strict` 或 `Lax`，减少 CSRF 风险。

* **服务端验证**

  * 服务器拒绝无效或不存在的 Session ID。
  * 对 Session 设置过期时间，定期清理。

* **二次验证机制（可选）**

  * 结合用户 IP 地址、User-Agent 进行绑定校验。
  * 若异常则强制重新登录。
  * 使用双因素认证提升安全性。

* **攻击检测和防护**

  * 监控异常登录行为和请求频率。
  * 触发风控策略（如限流、验证码）。

---

### 为什么不直接存 Access Token 在客户端？

* 将 Access Token 存在客户端（如 LocalStorage）容易被 XSS 攻击窃取。
* 将 Token 保存在服务端 Session 中，通过难以猜测的 Session ID 间接标识用户，提高安全性。
* 这样客户端只有 Session ID，单靠 Session ID 无法伪造身份，必须持有有效的 Session 记录。

---

### 总结

| 防伪造措施               | 具体做法                           |
| ------------------- | ------------------------------ |
| **安全随机 Session ID** | 使用强随机数，长度长，不易猜测                |
| **安全 Cookie 设置**    | 设置 HttpOnly、Secure、SameSite 属性 |
| **HTTPS 传输**        | 确保所有通信加密，防止窃取 Session ID       |
| **服务端 Session 校验**  | 拒绝无效或过期 Session                |
| **异常行为检测**          | 绑定 IP/User-Agent，防止会话劫持和CSRF   |