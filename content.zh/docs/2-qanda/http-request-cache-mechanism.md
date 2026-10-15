---
title: HttpServletRequest 请求体缓存机制及最佳实践
---

## 问题描述

在 Spring Boot Web 应用中，过滤器（Filter）、拦截器或其他组件可能需要提前读取 HTTP 请求体（Request Body），以获取请求中的数据。

然而，`HttpServletRequest` 的请求体通常通过输入流 `ServletInputStream` 提供，而输入流具有一次性读取的特点。当请求体被读取后，后续组件再次尝试读取时，可能无法获取完整内容，从而导致请求参数解析失败。

为了解决这一问题，可以通过包装原始 `HttpServletRequest`，将请求体缓存到内存中，并重写 `getInputStream()` 和 `getReader()` 方法，使后续组件能够重新读取请求体。

需要注意的是，**并不是所有 HTTP 请求都需要缓存请求体。只有当请求体需要被提前读取，并且后续组件仍然需要读取时，才有必要考虑缓存机制。**

## 问题原因及分析

### 1. HTTP 请求体具有流式读取特性

在 Servlet API 中，可以通过以下两种方式读取请求体：

* `getInputStream()`：以字节流的形式读取请求体。
* `getReader()`：以字符流的形式读取请求体。

例如：

```java
ServletInputStream inputStream = request.getInputStream();
byte[] body = inputStream.readAllBytes();
```

当请求体被读取完毕后，底层输入流通常已经到达末尾。后续代码再次读取同一个输入流时，可能只能得到空数据。

例如：

```java
byte[] firstRead = request.getInputStream().readAllBytes();
byte[] secondRead = request.getInputStream().readAllBytes();
```

这里的 `firstRead` 通常包含完整请求体，而 `secondRead` 通常为空。

需要注意，这里指的是重复读取同一个请求流。调用 `getInputStream()` 本身并不意味着容器会自动为每次调用创建一个新的、可重新读取的请求体流。

### 2. 过滤器提前读取请求体可能影响后续处理

Spring MVC 处理请求时，通常需要通过 `HttpMessageConverter` 等机制解析请求体，例如将 JSON 数据转换为 Java 对象。

如果自定义过滤器提前读取了原始请求体，却没有提供可重新读取的请求流，后续请求处理组件可能无法获取完整数据。

典型执行过程如下：

```text
客户端发送 HTTP 请求
        |
        v
Spring Security 过滤器链
        |
        v
自定义过滤器读取 Request Body
        |
        v
原始输入流被消费
        |
        v
Spring MVC 尝试解析 Request Body
        |
        v
可能无法读取完整请求数据
```

因此，当过滤器确实需要读取请求体，并且后续处理流程仍需要使用请求体时，可以通过请求包装器解决这一问题。

### 3. HttpServletRequestWrapper 的作用

Servlet API 提供了 `HttpServletRequestWrapper`，用于包装原始请求，并允许开发者重写请求相关方法。

通过继承该类，可以将原始请求体读取到字节数组中，再通过新的输入流返回缓存数据。

核心实现思路如下：

```java
public class CachedBodyHttpServletRequest
        extends HttpServletRequestWrapper {

    private final byte[] cachedBody;

    public CachedBodyHttpServletRequest(
            HttpServletRequest request) throws IOException {

        super(request);

        this.cachedBody = request.getInputStream().readAllBytes();
    }

    @Override
    public ServletInputStream getInputStream() {

        ByteArrayInputStream inputStream =
                new ByteArrayInputStream(cachedBody);

        return new ServletInputStream() {

            @Override
            public boolean isFinished() {
                return inputStream.available() == 0;
            }

            @Override
            public boolean isReady() {
                return true;
            }

            @Override
            public void setReadListener(ReadListener readListener) {
                throw new UnsupportedOperationException(
                        "Asynchronous reading is not supported");
            }

            @Override
            public int read() {
                return inputStream.read();
            }
        };
    }

    @Override
    public BufferedReader getReader() {
        return new BufferedReader(
                new InputStreamReader(
                        getInputStream(),
                        StandardCharsets.UTF_8));
    }
}
```

该实现展示了基本的缓存原理：

1. 在构造方法中读取原始请求体，并将其保存到 `byte[]` 中。
2. 重写 `getInputStream()`，使每次调用都基于缓存数据创建新的输入流。
3. 重写 `getReader()`，使字符流同样基于缓存数据创建。
4. 后续组件读取包装后的请求时，可以重新读取完整请求体。

需要注意，这只是一个用于说明原理的同步读取示例，并不是适用于所有生产环境的通用实现。对于异步请求、非阻塞 I/O、超大请求体等场景，需要进一步考虑 Servlet API 的契约、请求大小限制及内存管理。

### 4. 缓存机制并非没有成本

将请求体缓存到内存中，意味着应用程序需要为请求数据分配额外内存。

如果请求体较大，或者并发请求较多，可能造成明显的内存压力。

此外，缓存请求体并不意味着所有请求都应该被读取或重复读取。对于文件上传、流式传输等场景，直接将整个请求体加载到内存中可能并不合适。

因此，应当根据实际需求决定是否缓存，而不是默认缓存所有请求。

## 最佳实践及分析

### 1. 优先明确请求体的读取责任

设计请求处理流程时，应首先明确哪个组件负责解析请求体，以及后续组件是否还需要读取原始数据。

通常可以遵循以下原则：

* 如果请求体只需要解析一次，则直接使用正常的请求解析机制。
* 如果过滤器需要读取请求体，但后续组件不再需要原始请求体，可以根据实际需求直接处理，不必引入缓存包装器。
* 如果多个组件确实需要读取同一份请求体，可以考虑使用请求包装器或其他适当的缓存机制。
* 如果请求体较大，应避免不加限制地将全部数据加载到内存。

**优先减少重复读取需求，只有在确实需要时才引入请求体缓存。**

### 2. 在确有需要时使用请求包装器

当请求体必须被多个组件读取时，可以采用 `HttpServletRequestWrapper`。

需要保证：

* 原始请求体只在适当的位置读取一次。
* 包装器保存的数据能够被后续组件正常访问。
* `getInputStream()` 和 `getReader()` 的实现符合 Servlet API 的使用约定。
* 请求体大小受到合理限制。
* 不会因为重复缓存而造成不必要的内存开销。
* 异步处理、非阻塞 I/O 等场景得到适当支持。

对于生产环境，应优先评估框架或容器提供的现有能力，避免自行实现不完整的请求包装器。

### 3. 避免在认证过滤器中承担不必要的请求解析工作

在 Spring Security 项目中，建议让不同组件保持明确的职责边界：

* **认证过滤器**：负责身份凭证提取、认证校验和安全上下文设置。
* **授权机制**：负责判断当前身份是否有权访问目标资源。
* **请求解析组件**：负责将请求数据转换为应用程序需要的对象。
* **业务服务**：负责处理业务规则和业务数据校验。

如果认证过滤器只需要读取请求头中的认证凭证，就没有必要为了认证而缓存整个请求体。

将请求体解析与业务规则检查放在适当的处理层，可以降低过滤器的复杂度，也能避免不必要的内存开销。

### 4. 注意包装器的实际使用范围

创建请求包装器后，需要将包装后的请求传递给后续过滤器：

```java
CachedBodyHttpServletRequest wrappedRequest =
        new CachedBodyHttpServletRequest(request);

filterChain.doFilter(wrappedRequest, response);
```

如果后续过滤器仍然接收到原始请求，而不是包装后的请求，那么请求体缓存机制可能无法发挥预期作用。

此外，还应检查过滤器的执行条件。

如果某些路径通过 `shouldNotFilter()` 被排除，那么对应请求不会进入 `doFilterInternal()`，其中的请求体缓存逻辑自然也不会执行。

因此，在设计请求包装器时，需要同时考虑过滤器的执行范围、包装对象的传递方式，以及后续组件的请求体读取行为。

### 5. 谨慎处理敏感数据与异常情况

请求体可能包含敏感信息。缓存请求体时，应避免将完整内容直接写入日志，防止敏感数据泄露。

同时，应考虑以下问题：

* 请求体读取失败时如何处理。
* 空请求体如何处理。
* 非法编码或格式错误如何处理。
* 请求体超过允许大小时如何处理。
* 是否需要对不同 Content-Type 采用不同处理方式。
* 是否支持异步请求及非阻塞读取。

缓存请求体是一种技术手段，而不是必须应用于所有请求的默认方案。

## 总结

`HttpServletRequest` 的请求体通常通过一次性消费的输入流提供。当某个组件提前读取请求体，而后续组件仍然需要读取时，可能导致请求数据无法被重复获取。

通过继承 `HttpServletRequestWrapper`，将请求体缓存到字节数组中，并重写 `getInputStream()` 和 `getReader()`，可以让后续组件重新读取缓存的数据。

但请求体缓存也会增加内存消耗和实现复杂度，因此不应无条件使用。

在 Spring Boot 和 Spring Security 项目中，推荐遵循以下原则：

1. 明确请求体的读取责任，尽量避免重复读取。
2. 只有多个处理组件确实需要读取同一请求体时，才考虑缓存。
3. 根据请求大小、并发量和 I/O 模型评估缓存实现的安全性与性能。
4. 保持认证、授权、请求解析和业务处理之间的职责边界。
5. 确保包装后的请求能够正确传递给后续过滤器及请求处理组件。

**核心原则：请求体缓存用于解决重复读取问题，而不是所有 HTTP 请求的必备处理步骤。**
