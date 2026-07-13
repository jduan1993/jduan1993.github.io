---
title: WebClient 响应体为空时如何识别和处理
---

有，**WebClient 确实有一个常见问题：响应体为空（empty body）时如何识别和处理**。这个在使用 `bodyToMono()` 时尤其容易遇到。

---

## 1. `bodyToMono()` 遇到空响应体会怎样？

例如：

服务端返回：

```
HTTP/1.1 200 OK
Content-Length: 0
```

没有 body。

代码：

```java
webClient.get()
    .uri("/user")
    .retrieve()
    .bodyToMono(User.class);
```

结果：

```java
Mono.empty()
```

也就是说：

* **不会返回 `null`**
* **不会报错**
* 返回的是一个没有数据的 Mono

例如：

```java
Mono<User> result = webClient.get()
        .retrieve()
        .bodyToMono(User.class);
```

实际状态：

```
Mono<User>
   |
   └── empty
```

---

## 2. 为什么容易踩坑？

比如：

```java
User user = webClient.get()
        .uri("/user")
        .retrieve()
        .bodyToMono(User.class)
        .block();

user.getName();
```

如果响应为空：

```java
user == null
```

然后：

```java
user.getName();
```

会：

```
NullPointerException
```

因为：

```java
block()
```

把：

```
Mono.empty()
```

转换成：

```
null
```

---

## 3. 怎么判断空响应？

### 方法1：`switchIfEmpty()`

推荐：

```java
webClient.get()
    .uri("/user")
    .retrieve()
    .bodyToMono(User.class)
    .switchIfEmpty(
        Mono.error(new RuntimeException("响应为空"))
    );
```

流程：

```
有body
 |
 User对象

无body
 |
 抛异常
```

---

### 方法2：返回默认值

例如：

```java
.bodyToMono(User.class)
.defaultIfEmpty(new User())
```

如果没有响应：

返回：

```java
new User()
```

---

### 方法3：使用 `hasElement()`

判断有没有数据：

```java
Mono<Boolean> exists =
    webClient.get()
        .uri("/user")
        .retrieve()
        .bodyToMono(User.class)
        .hasElement();
```

结果：

```
有body:
true

空body:
false
```

---

## 4. 还有一个容易混淆的问题：204 No Content

比如：

```
HTTP/1.1 204 No Content
```

这是合法响应。

例如删除接口：

```
DELETE /user/1

返回:
204
```

代码：

```java
.retrieve()
.bodyToMono(Void.class)
```

通常这样处理。

但是：

```java
.bodyToMono(User.class)
```

会得到：

```
Mono.empty()
```

---

## 5. `retrieve()` 对空 body 不敏感

很多人以为：

```java
.retrieve()
```

会判断：

"服务器有没有返回内容"

实际上：

`retrieve()` 主要处理：

* HTTP状态码
* 错误响应

例如：

```
404
500
```

而：

```java
bodyToMono()
```

负责：

* body解析
* 类型转换

所以：

```
HTTP 200 + 空body
```

不会被认为错误。

---

## 6. 实际项目中常见写法

例如调用第三方接口：

```java
return webClient.get()
        .uri(url)
        .retrieve()
        .bodyToMono(ResponseDTO.class)
        .switchIfEmpty(
            Mono.error(
                new BusinessException("接口返回为空")
            )
        )
        .timeout(Duration.ofSeconds(30))
        .doOnSuccess(resp ->
            log.info("调用成功: {}", resp)
        )
        .doOnError(e ->
            log.error("调用失败", e)
        );
```

这样：

| 情况           | 结果               |
| ------------ | ---------------- |
| 200 + 正常JSON | 返回对象             |
| 200 + 空body  | 抛业务异常            |
| 500          | WebClient异常      |
| 超时           | TimeoutException |
| 网络错误         | 连接异常             |

---

## 7. 一个更隐蔽的问题：空 JSON

注意：

空 body：

```
<empty>
```

和：

空 JSON：

```json
{}
```

不是一回事。

例如：

```json
{}
```

WebClient 会：

```java
User(
 id=null,
 name=null
)
```

不会触发：

```java
switchIfEmpty()
```

因为它有内容。

---

总结：

> WebClient 的 `bodyToMono()` 对空响应体的处理是返回 `Mono.empty()`，不会报错。需要用 `switchIfEmpty()`、`defaultIfEmpty()` 等显式处理，否则 `.block()` 后可能得到 `null` 引发问题。

在微服务调用、网关调用第三方接口时，这个确实是一个非常常见的坑。
