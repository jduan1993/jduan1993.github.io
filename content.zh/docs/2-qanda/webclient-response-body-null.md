---
title: WebClient 响应体为空时的识别与处理
---

## 问题描述

在 Spring WebFlux 中使用 `WebClient` 发起 HTTP 请求时，如果服务端返回成功状态码，但响应体为空，调用 `bodyToMono()` 通常会得到 `Mono.empty()`，而不是抛出异常。

例如：

```java
User user = webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(User.class)
        .block();
```

当服务端返回 HTTP 200，但没有响应体时，`bodyToMono(User.class)` 会产生一个空的 `Mono`。调用 `block()` 后，得到的结果为 `null`。

如果后续代码直接访问对象属性，就可能触发 `NullPointerException`。

需要注意，**HTTP 请求成功、响应体存在、响应体能够成功反序列化，是三个不同的概念**。HTTP 状态码正常，并不意味着一定能够获取到预期的 Java 对象。

此外，空响应体与内容为 `{}` 的 JSON 响应并不相同，两者需要采用不同的判断方式。

## 问题原因及分析

### 1. `bodyToMono()` 遇到空响应体时返回 `Mono.empty()`

`WebClient` 是基于 Reactor 的响应式 HTTP 客户端。调用 `bodyToMono(Class<T>)` 时，会尝试将响应体转换为指定类型的对象，并通过 `Mono` 表示结果。

当响应体为空时，典型行为如下：

```java
Mono<User> result = webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(User.class);
```

此时，`result` 表示一个没有发出数据的 `Mono`。

需要区分以下三种状态：

* **有数据**：`Mono` 发出一个 `User` 对象。
* **无数据**：`Mono.empty()`，不发出任何对象。
* **执行失败**：`Mono.error(exception)`，以错误信号终止。

这三种状态在 Reactor 中具有不同的语义，不能简单地将空结果视为异常。

### 2. `block()` 会将空结果转换为 `null`

在响应式编程中，`Mono.empty()` 本身不包含 `null` 值。

但是，当调用 `block()` 将响应式结果转换为同步返回值时，如果 `Mono` 没有发出数据，`block()` 就会返回 `null`。

例如：

```java
User user = webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(User.class)
        .block();
```

如果响应体为空，实际结果为：

```java
user == null
```

继续执行：

```java
user.getName();
```

就会产生空指针异常。

因此，问题的根源不是 `Mono.empty()` 自动产生了空指针异常，而是调用方没有正确处理响应式流可能不发出数据的情况。

### 3. `retrieve()` 不会默认将空响应体视为错误

`retrieve()` 主要用于获取响应并处理 HTTP 状态码对应的错误情况。

默认情况下，对于 HTTP 4xx 和 5xx 响应，`retrieve()` 会按照其错误处理规则生成异常；对于正常状态码，即使响应体为空，也不会仅因为没有内容就自动抛出异常。

例如：

```java
webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(User.class);
```

如果服务端返回 HTTP 200 和空响应体，通常得到的是 `Mono.empty()`。

如果接口约定成功响应必须包含对象，就需要调用方显式判断空结果，并将其转换为符合接口语义的异常或其他结果。

### 4. 空响应体与空 JSON 对象的区别

以下两种响应并不等价。

**空响应体：**

```http
HTTP/1.1 200 OK
Content-Length: 0
```

在没有其他响应内容的情况下，`bodyToMono(User.class)` 通常得到 `Mono.empty()`。

**包含空 JSON 对象的响应体：**

```json
{}
```

此时响应体包含实际内容。Jackson 等 JSON 反序列化器通常会将其转换为一个 `User` 对象，其属性值可能为 `null` 或对应的默认值。

因此：

```java
.bodyToMono(User.class)
.switchIfEmpty(Mono.error(
        new IllegalStateException("响应体为空")
))
```

只能处理没有发出数据的情况，不能判断反序列化得到的对象是否缺少必要字段。

如果接口还要求校验对象内容，应在反序列化之后增加独立的字段校验逻辑。

## 最佳实践及分析

### 1. 使用 `switchIfEmpty()` 明确处理空响应

如果接口约定成功响应必须包含有效对象，可以使用 `switchIfEmpty()` 将空结果转换为异常。

```java
Mono<User> result = webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(User.class)
        .switchIfEmpty(
                Mono.error(
                        new IllegalStateException("响应体为空")
                )
        );
```

其处理逻辑如下：

```text
HTTP 响应
    |
    v
bodyToMono(User.class)
    |
    +--> 有数据 --> 返回 User 对象
    |
    +--> 无数据 --> Mono.empty()
                         |
                         v
                   switchIfEmpty()
                         |
                         v
                      Mono.error()
```

这种方式能够在响应式链路中明确区分正常数据与空结果，避免空值问题延迟到后续代码才暴露。

实际开发中，应根据项目的异常体系选择合适的异常类型，而不是统一使用通用运行时异常。

### 2. 使用 `defaultIfEmpty()` 提供默认值

如果接口允许响应体为空，并且可以使用合理的默认对象，也可以通过 `defaultIfEmpty()` 处理。

```java
Mono<User> result = webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(User.class)
        .defaultIfEmpty(new User());
```

当响应体为空时，`Mono` 会发出默认对象。

需要注意，这种方式会将“没有返回数据”转换为“返回默认对象”，可能掩盖服务端未按约定返回内容的问题。

因此，只有在接口契约明确允许这种行为时，才建议使用默认值。

### 3. 使用 `hasElement()` 判断是否存在数据

如果只需要判断响应体是否产生了对象，可以使用 `hasElement()`。

```java
Mono<Boolean> hasData = webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(User.class)
        .hasElement();
```

结果如下：

| 响应情况                    | 结果      |
| ----------------------- | ------- |
| 成功反序列化并产生对象             | `true`  |
| 响应体为空，产生 `Mono.empty()` | `false` |
| 请求或反序列化失败               | 以错误信号终止 |

需要注意，`hasElement()` 判断的是响应式流是否发出元素，而不是 HTTP 响应体的原始字节数，也不是对象字段是否完整。

此外，如果后续还需要使用实际对象，不建议为了判断是否存在数据而单独发起一次 HTTP 请求。应尽可能在同一条响应式链路中完成判断和处理。

### 4. 根据接口语义选择响应类型

并非所有接口都要求返回 JSON 对象。

例如，某些操作成功后只返回 HTTP 204 No Content，或者接口约定不需要返回任何数据。

对于此类接口，可以使用 `bodyToMono(Void.class)`：

```java
Mono<Void> result = webClient.delete()
        .uri("/resource")
        .retrieve()
        .bodyToMono(Void.class);
```

这种方式适用于调用方不需要读取响应体内容的场景。

如果接口必须返回对象，则应使用对应的 DTO 类型，并明确处理空响应。

选择响应类型时，应根据服务端接口契约决定，而不是为了避免空指针而一律使用 `Void.class`。

### 5. 区分 HTTP 错误、空响应与调用超时

在使用 `WebClient` 时，应区分不同的失败情况：

| 场景                 | 典型处理方式                    |
| ------------------ | ------------------------- |
| HTTP 2xx，响应体包含有效内容 | 反序列化并返回对象                 |
| HTTP 2xx，响应体为空     | 根据接口契约处理 `Mono.empty()`   |
| HTTP 4xx 或 5xx     | 根据 `retrieve()` 的错误处理规则处理 |
| 响应体格式错误或类型转换失败     | 处理反序列化异常                  |
| 请求超时               | 处理超时异常                    |
| 网络连接失败             | 处理网络或连接异常                 |
| JSON 对象缺少必要字段      | 在反序列化后进行字段校验              |

可以通过 `onStatus()` 定制 HTTP 错误处理，通过 `switchIfEmpty()` 处理空结果，并通过 `timeout()` 为请求设置超时限制。

例如：

```java
return webClient.get()
        .uri("/resource")
        .retrieve()
        .bodyToMono(ResponseDTO.class)
        .switchIfEmpty(
                Mono.error(
                        new IllegalStateException("响应体为空")
                )
        )
        .timeout(Duration.ofSeconds(30));
```

这里的超时时间仅为示例，应根据接口的实际响应时间要求进行配置。

需要注意，`timeout()` 用于限制响应式流在指定时间内没有产生相应信号的情况，并不专门用于判断 HTTP 响应体是否为空。空响应通常会正常完成，因此需要使用 `switchIfEmpty()` 单独处理。

### 6. 避免在响应式链路中不必要地调用 `block()`

`block()` 会阻塞当前线程，等待 `Mono` 发出数据、完成或发生错误。

在 WebFlux 等响应式应用中，应优先通过响应式操作符传递和处理结果，而不是过早将响应式流转换为同步对象。

例如：

```java
public Mono<User> getUser() {
    return webClient.get()
            .uri("/resource")
            .retrieve()
            .bodyToMono(User.class)
            .switchIfEmpty(
                    Mono.error(
                            new IllegalStateException("响应体为空")
                    )
            );
}
```

这种方式保留了响应式链路，也能够让调用方继续通过 `map()`、`flatMap()`、`onErrorResume()` 等操作符处理结果。

## 总结

`WebClient` 使用 `bodyToMono()` 处理 HTTP 响应体时，如果响应体为空，通常会返回 `Mono.empty()`，而不是直接抛出异常。调用 `block()` 后，空的 `Mono` 会转换为 `null`，如果调用方没有进行空值处理，就可能导致空指针异常。

排查和处理此类问题时，需要区分 HTTP 状态码、响应体是否存在、反序列化是否成功，以及反序列化后的对象是否满足接口约定。

对于必须返回对象的接口，推荐使用 `switchIfEmpty()` 显式处理空响应；对于允许空响应的接口，可以根据实际语义选择 `defaultIfEmpty()` 或 `Mono<Void>` 等方式。

**核心原则：不要将 HTTP 请求成功等同于响应体一定存在，也不要将 `Mono.empty()` 与包含空字段的对象混为一谈。应根据接口契约明确处理空结果，并尽可能在响应式链路中完成校验和异常处理。**
