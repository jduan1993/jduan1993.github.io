---
title: 重放攻击
---

重放攻击（**Replay Attack**）是一种网络攻击手段，攻击者**拦截并保存合法请求的数据包**，然后**在稍后“重发”这些数据包**到服务器，试图欺骗系统重复执行某些操作（如转账、登录等）。

---

## 🧨 一句话理解：

> **攻击者不是伪造请求，而是“复读”别人发的合法请求来骗系统重复操作！**

---

## 📦 举个例子：

你在网银发送了如下转账请求（加密传输）：

```
POST /transfer
{
  "to": "Alice",
  "amount": 100
}
```

攻击者通过某种方式（如网络监听）捕获了这条请求，然后在你不知情的情况下**原样重放**这条请求。

结果系统再次转账了 100 元到 Alice，**你损失了 200 元**。

---

## 🧠 为什么这条“旧的请求”还会被服务器接受？

因为很多系统：

* **没有识别请求是否重复**
* **没对请求加时间戳 / 唯一标识**
* **加密是对内容加密，不代表防止重发**

---

## 🔐 重放攻击常见于哪些协议场景？

| 场景              | 原因                |
| --------------- | ----------------- |
| HTTPS（0-RTT 模式） | 早期数据可能在握手前被接受     |
| API 接口（如支付接口）   | 攻击者重放旧接口实现重复操作    |
| 登录认证（token）     | 旧 token 被截获后仍可登录  |
| 智能设备通信          | 设备控制指令被复用（开锁、支付等） |

---

## 🛡️ 如何防御重放攻击？

### ✅ 1. 使用唯一请求 ID（nonce）

* 每个请求携带唯一的随机字符串
* 服务端校验是否已处理过该 ID（如 Redis 去重）

### ✅ 2. 添加时间戳并设有效期

* 请求中携带当前时间
* 服务器验证是否在允许的时间窗口内（如 2 分钟）

### ✅ 3. 使用 HMAC 签名请求参数

* 客户端对请求参数 + 时间戳生成签名
* 服务端验证签名是否正确、是否被篡改

### ✅ 4. 使用 TLS 防中间人监听（基础防线）

* 如果攻击者无法截获原始数据，就不能重放

### ✅ 5. 对 0-RTT 请求做幂等性校验（QUIC/TLS 1.3 特别注意）

* 拒绝非幂等类型（如 POST/PUT）使用 0-RTT
* 服务端缓存并验证历史 0-RTT 请求

---

## 💡 补充：幂等请求 vs 非幂等请求

| 请求类型   | 是否幂等 | 说明               |
| ------ | ---- | ---------------- |
| GET    | ✅    | 多次访问结果一样         |
| POST   | ❌    | 多次发送可能产生副作用（如转账） |
| PUT    | ✅    | 设置某值，多次是一样的      |
| DELETE | ✅/❌  | 一般幂等，但有特殊情况除外    |

---

## 🎯 总结一句话：

> **重放攻击 = 重发旧的有效请求，欺骗系统重复执行操作。**
>
> 防御关键在于**识别请求是否“独一无二、在有效时间内、未处理过”**。

---

# 最佳实践

生产环境中防止重放攻击的最佳实践主要围绕以下几个核心措施展开，目标是确保每个请求**唯一且不可重复执行**，同时保证安全性和性能：

---

## 1. 请求唯一性（使用 Nonce 或唯一请求 ID）

* **客户端生成唯一的随机字符串（Nonce）**，每次请求携带
* **服务端维护一张已使用 Nonce 的缓存表（如 Redis）**
* 收到请求时检查 Nonce 是否已被处理过：

  * 是 → 拒绝，防止重复执行
  * 否 → 继续处理，并记录 Nonce

> **注意**：缓存 Nonce 的有效期应合理，防止缓存爆炸。

---

## 2. 时间戳和有效期限制

* 请求携带时间戳（如 UNIX 时间戳）
* 服务端验证请求时间与当前时间差是否在允许窗口内（例如 ±5 分钟）
* 超时请求拒绝处理，避免老请求被重放

---

## 3. 请求签名（HMAC 或数字签名）

* 客户端对请求参数（包括 Nonce、时间戳等）进行签名，保证请求未被篡改
* 服务端用共享密钥验证签名
* 签名同时防止篡改和保证请求的完整性

---

## 4. 幂等接口设计

* 尽量设计幂等的接口，保证重复请求不会造成副作用（如支付接口多次请求只扣一次）
* 对非幂等操作，通过上面 Nonce 和签名严格限制重复请求

---

## 5. 使用安全协议（TLS 1.3）

* 通过加密防止中间人抓包窃听和篡改
* 减少攻击者获取请求内容的可能

---

## 6. 对 0-RTT 请求特别防护（针对 QUIC/TLS 1.3）

* 限制 0-RTT 只用在幂等操作
* 服务器维护已接受的 0-RTT 请求缓存，防止重复执行
* 对非幂等请求禁用 0-RTT

---

## 7. 监控与告警

* 对异常重复请求频繁的 IP 或用户，触发告警
* 结合风控策略限制请求速率和行为异常

---

## 8. 具体技术栈示例

| 技术        | 推荐方案                 |
| --------- | -------------------- |
| 缓存        | Redis，支持快速去重缓存 Nonce |
| 签名算法      | HMAC-SHA256，RSA 数字签名 |
| 时间同步      | NTP 保证服务器时间准确        |
| 防火墙 & WAF | 监测并阻断异常请求            |

---

# 总结

| 防重放手段     | 作用          | 实施难度 | 备注        |
| --------- | ----------- | ---- | --------- |
| Nonce 唯一值 | 保证请求不可重复执行  | 中    | 需管理缓存有效期  |
| 时间戳校验     | 限制请求有效时间窗口  | 低    | 防止旧请求被重放  |
| 请求签名      | 保证请求完整性与合法性 | 中    | 需安全管理密钥   |
| 幂等设计      | 降低重复请求风险    | 设计层面 | 尽量设计业务幂等  |
| 0-RTT 限制  | 防止早期数据重放攻击  | 高    | 仅支持幂等请求   |
| 安全传输（TLS） | 防止中间人抓包     | 低    | 基础防线      |
| 监控告警      | 实时发现异常行为    | 中    | 配合自动化防御策略 |

---

# 案例

## 1. 请求体设计

假设这是前端或客户端发出的请求 JSON：

```json
{
  "userId": "12345",
  "action": "transfer",
  "amount": 100,
  "timestamp": 1687645200,          // UNIX时间戳，秒级
  "nonce": "a1b2c3d4e5f6",          // 唯一请求ID，防重放
  "signature": "abcdef1234567890"   // 请求签名，防篡改
}
```

---

## 2. 签名规则

> 签名 = HMAC\_SHA256(secret\_key, 拼接所有请求参数（除 signature）)

常见拼接方式（参数按字典序）：

```
action=transfer&amount=100&nonce=a1b2c3d4e5f6&timestamp=1687645200&userId=12345
```

---

## 3. 伪代码示范

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.util.*;
import java.nio.charset.StandardCharsets;

public class ReplayProtection {

    private static final String SECRET_KEY = "YOUR_SECRET_KEY";

    // 计算签名
    public static String generateSignature(Map<String, String> params) throws Exception {
        // 排序参数名
        List<String> keys = new ArrayList<>(params.keySet());
        Collections.sort(keys);

        // 拼接字符串，排除 signature 字段
        StringBuilder sb = new StringBuilder();
        for (String key : keys) {
            if (key.equals("signature")) continue;
            sb.append(key).append("=").append(params.get(key)).append("&");
        }
        sb.deleteCharAt(sb.length() - 1); // 去掉最后的&

        // HMAC-SHA256 计算
        Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secret_key = new SecretKeySpec(SECRET_KEY.getBytes(StandardCharsets.UTF_8), "HmacSHA256");
        sha256_HMAC.init(secret_key);
        byte[] hash = sha256_HMAC.doFinal(sb.toString().getBytes(StandardCharsets.UTF_8));

        // 转16进制字符串
        StringBuilder result = new StringBuilder();
        for (byte b : hash) {
            result.append(String.format("%02x", b));
        }
        return result.toString();
    }

    // 验证请求
    public static boolean verifyRequest(Map<String, String> params, Set<String> nonceCache, long allowedTimeWindowSeconds) throws Exception {
        // 1. 验证时间戳
        long timestamp = Long.parseLong(params.get("timestamp"));
        long now = System.currentTimeMillis() / 1000;
        if (Math.abs(now - timestamp) > allowedTimeWindowSeconds) {
            System.out.println("请求时间戳超时");
            return false;
        }

        // 2. 验证nonce唯一性（防重放）
        String nonce = params.get("nonce");
        if (nonceCache.contains(nonce)) {
            System.out.println("请求nonce重复，疑似重放攻击");
            return false;
        }
        nonceCache.add(nonce);

        // 3. 验证签名
        String clientSign = params.get("signature");
        String serverSign = generateSignature(params);
        if (!serverSign.equals(clientSign)) {
            System.out.println("签名校验失败");
            return false;
        }

        return true;
    }

    public static void main(String[] args) throws Exception {
        // 模拟请求参数
        Map<String, String> requestParams = new HashMap<>();
        requestParams.put("userId", "12345");
        requestParams.put("action", "transfer");
        requestParams.put("amount", "100");
        requestParams.put("timestamp", String.valueOf(System.currentTimeMillis() / 1000));
        requestParams.put("nonce", UUID.randomUUID().toString().replace("-", ""));

        // 生成签名并加入请求参数
        String signature = generateSignature(requestParams);
        requestParams.put("signature", signature);

        // 模拟服务器缓存nonce（实际可用 Redis 等存储）
        Set<String> nonceCache = new HashSet<>();

        // 验证请求
        boolean valid = verifyRequest(requestParams, nonceCache, 300);
        System.out.println("请求是否有效：" + valid);

        // 模拟重放请求（同一个nonce）
        boolean replay = verifyRequest(requestParams, nonceCache, 300);
        System.out.println("重放请求是否有效：" + replay);
    }
}
```

---

## 4. 说明

* **timestamp** 防止旧请求被重放，服务器只接受一定时间窗口内的请求
* **nonce** 保证请求唯一，服务端缓存已用 nonce，重复使用判定为重放攻击
* **signature** 用 HMAC-SHA256 绑定所有参数，防止篡改
* **nonceCache** 这里用 HashSet 模拟，生产中应用 Redis 或其他分布式缓存并设置过期

---

## 5. 生产中需要考虑

* **nonce 缓存大小和过期策略**：防止内存/缓存爆满
* **时间同步**：客户端和服务器时间要同步（NTP）
* **签名算法和密钥安全**：密钥要安全管理，避免泄露
* **接口幂等设计**：即使请求未被重放，也要保证多次执行不出错
* **失败重试和并发控制**

---

需要我帮你写一个基于Spring Boot的拦截器示例，或者其他语言版本的实现吗？

很好的问题！关于**为什么用HMAC，以及生产环境的最佳实践**，我详细说说。

---

# HMAC

## 1. 为什么用 HMAC？

**HMAC（Hash-based Message Authentication Code）** 是一种基于密钥的消息认证码算法，常用来保证消息的完整性和真实性。它的优势包括：

* **防篡改性强**
  HMAC 用一个**共享的密钥**和哈希算法（如 SHA-256）结合生成签名，只有知道密钥才能正确计算和验证签名，防止消息被篡改。

* **计算效率高**
  相比公钥签名（RSA/ECDSA），HMAC 速度快、性能开销小，适合高并发场景。

* **简单易用**
  只需共享密钥，客户端和服务端都能快速实现，无需复杂的公私钥管理。

* **抗碰撞强**
  结合哈希算法的抗碰撞性，保证签名唯一且安全。

---

## 2. 生产环境的最佳实践是用 HMAC 吗？

**大部分互联网公司和金融系统都采用 HMAC 作为请求签名和防重放的核心手段。**

原因：

| 优势       | 解释                |
| -------- | ----------------- |
| 性能优异     | 适合海量请求的快速签名和验证    |
| 安全性足够强   | 对称密钥机制，在密钥管理得当下安全 |
| 实现和维护成本低 | 几乎所有语言和平台都有标准实现   |
| 容易集成     | 适用于移动端、前端、服务器端签名  |

---

## 3. 什么时候不用 HMAC？

* **需要非对称签名**（公钥/私钥）场景
  如 OAuth、JWT、数字证书签名，保证签名者身份公开验证。

* **多方交互，不能共享密钥**
  需要公私钥体系保证身份不可伪造。

---

## 4. 生产中其他常见做法

| 方案                 | 适用场景         | 备注           |
| ------------------ | ------------ | ------------ |
| HMAC（SHA-256等）     | 内部服务调用、API签名 | 高性能，简单       |
| RSA/ECDSA 数字签名     | 跨服务或第三方交互    | 身份验证，非对称更安全  |
| JWT + 签名           | 无状态认证        | 集成身份和权限，方便传递 |
| OAuth 2.0 / OpenID | 统一认证授权       | 标准协议，多方授权    |

---

## 5. 总结

| 特点     | HMAC              | 公钥签名（RSA/ECDSA） |
| ------ | ----------------- | --------------- |
| 安全性    | 高（对称密钥，密钥需安全）     | 更高（非对称，可公开验证）   |
| 性能     | 快                 | 慢，计算开销大         |
| 实现复杂度  | 低                 | 高，需密钥管理和分发      |
| 典型使用场景 | 内部 API 签名，防篡改，防重放 | 跨系统身份认证、电子签名    |

---

如果你要设计的系统是**公司内部服务、API接口调用、移动端调用**，HMAC几乎是最实用且广泛认可的方案。

需要，我可以帮你写一份**HMAC密钥管理最佳实践**或更详细的**公钥签名对比分析**。你感兴趣吗？

这是个非常关键且实用的问题！我帮你拆解讲清楚：

---

# FAQ

## 1. **前后端都要知道HMAC密钥吗？**

### 理想设计原则：**不应该让前端知道完整的 HMAC 服务器密钥**

* **HMAC密钥是“共享密钥”，只要泄露，攻击者就能伪造签名。**
  所以，**服务器端密钥必须严格保密，不能暴露给客户端（浏览器、移动端）**。

* **前端无法持有真正的服务器密钥，但需要某种机制签名请求，防止篡改和重放。**

---

## 2. 前端如何做签名？

* **做法一：前端使用客户端密钥（Client Secret）生成签名**
  适用于移动App或可信客户端。

  * 服务器为每个客户端分配唯一密钥
  * 前端用自己的密钥计算签名，后端校验对应密钥
  * 密钥隐藏在客户端代码中，防止泄漏（但有一定风险）

* **做法二：前端不签名，服务器端对请求做签名验证 + 防重放策略**
  适合Web前端。

  * 前端传递必要参数（含nonce、timestamp）
  * 服务器端校验nonce唯一和时间戳有效，
  * 结合身份认证（Token、Session）限制重复请求

* **做法三：API Gateway / 网关负责签名和校验**

  * 网关和后端服务共享密钥
  * 前端请求经过网关统一验证
  * 后端服务只信任网关转发的合法请求

---

## 3. **微服务中如何用HMAC防重放完成支付流程？**

### 典型流程示例：

```
[客户端] → 发起支付请求 → [API网关] → [支付服务] → [支付网关]
```

---

### 详细步骤

| 步骤             | 描述                                        |
| -------------- | ----------------------------------------- |
| 1. 客户端发起支付请求   | 前端发送支付参数（金额、订单号、时间戳、nonce等）到API网关         |
| 2. API网关签名请求   | API网关用**自身的密钥**对请求参数计算 HMAC 签名，附加到请求头或参数中 |
| 3. 支付服务验证签名    | 支付服务从API网关接收请求，使用共享密钥验证签名，确保请求未被篡改        |
| 4. 支付服务校验nonce | 检查请求中的nonce是否已使用，防止重复支付请求                 |
| 5. 支付服务验证时间戳   | 时间戳检查，防止过期请求                              |
| 6. 支付执行        | 通过验证后，调用支付网关完成支付                          |
| 7. 返回结果        | 支付服务返回结果给API网关，API网关反馈给客户端                |

---

### 关键点说明

* **HMAC密钥仅存在于API网关和支付服务之间，不暴露给客户端**
* **客户端不需要知道密钥，也不直接计算HMAC签名**
* **API网关是可信代理，负责对请求签名和校验**
* **支付服务通过nonce和时间戳保证请求唯一且及时**
* **对nonce缓存一般存储在分布式缓存（Redis）中，设置过期时间**
* **时间同步依赖NTP，防止时间差过大导致请求被误拒**

---

## 4. 典型架构图示意

```
客户端
  |
  | 请求（无HMAC签名，带时间戳+nonce）
  ↓
API网关 (生成HMAC签名)
  |
  | 带签名的请求
  ↓
支付服务 (验证签名 + 防重放)
  |
  ↓
支付网关
```

---

## 5. 总结

| 角色    | 是否知道HMAC密钥 | 负责什么               |
| ----- | ---------- | ------------------ |
| 客户端   | 否          | 生成请求参数（时间戳、nonce）  |
| API网关 | 是          | 计算并添加 HMAC 签名      |
| 支付服务  | 是          | 验证签名，检查 nonce 和时间戳 |

---
