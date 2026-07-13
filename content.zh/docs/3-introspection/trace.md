---
title: OpenTelemetry 全链路追踪
---

在微服务架构中使用 OpenTelemetry 实现全链路追踪是提升可 observability（可观测性）和系统可维护性的关键步骤。以下是一些最佳实践，涵盖从设计、实现到部署运维的完整视角。

---

## 🌐 一、OpenTelemetry 简介

OpenTelemetry 是由 CNCF（Cloud Native Computing Foundation）托管的开放标准，支持 **分布式追踪（Tracing）**、**指标（Metrics）** 和 **日志（Logs）** 的采集与导出。它支持多语言，能够将数据导出至多种后端（如 Jaeger、Prometheus、Grafana Tempo、Datadog、Zipkin 等）。

---

## 📦 二、全链路追踪的核心概念

1. **Trace（追踪）**：一次完整的调用链（如用户下单请求）。
2. **Span（跨度）**：调用链中的单个操作（如订单服务调用库存服务）。
3. **Context（上下文）**：跨服务传递 trace 的元信息（如 trace id）。

---

## 🔧 三、在微服务架构中使用 OpenTelemetry 的最佳实践

### ✅ 1. **统一规范和自动化接入**

* **选择统一的 SDK 版本**：所有服务使用相同版本的 OpenTelemetry SDK。
* **使用自动注入的方式接入 HTTP / gRPC 框架**：例如 Spring Boot + Spring Cloud Sleuth，Python FastAPI + OpenTelemetry middleware，Go + OpenTelemetry HTTP 插件。
* **尽量避免手动创建 span，使用封装的 middleware 或框架支持**。

---

### 🔗 2. **统一上下文传递（Context Propagation）**

* 使用标准协议（如 W3C Trace Context）传递 trace 上下文。
* 在所有服务间调用（REST/gRPC/Kafka/NATS）时，显式或自动传递 `traceparent`、`baggage` 等 headers。
* 确保所有服务的网关、API 层、异步消息系统都能正确传递 trace context。

---

### 📤 3. **选择合适的后端（Exporter）**

常见导出目标：

* **Jaeger / Zipkin**（轻量部署）
* **Grafana Tempo**（可与 Loki、Prometheus 集成）
* **Datadog / New Relic / AWS X-Ray / Google Cloud Trace**（SaaS）

建议：

* 开发和测试使用 Jaeger，生产建议使用更具可扩展性和可集成性的后端。
* 将 Trace 数据与 Metrics、Logs 汇聚到统一平台（如 Grafana Stack）。

---

### 🧩 4. **整合日志与指标（Logs & Metrics）**

* 将 TraceId 注入日志上下文（如 Logback / Log4j 的 MDC，Python 的 logging filter）。
* 实现“从日志跳转到 Trace”，“从 Trace 跳转到 Metrics”的闭环观测。
* 示例日志格式：

  ```json
  {
    "timestamp": "2025-07-07T10:10:00Z",
    "level": "INFO",
    "trace_id": "abc123",
    "span_id": "def456",
    "message": "Order created successfully"
  }
  ```

---

### 📊 5. **采样策略（Sampling）设计**

* **Always on**：开发调试时使用。
* **Probabilistic sampling**：按概率采样，适合高流量生产环境。
* **Tail-based sampling**：先收集，再判断是否保留，适用于链路分析。
* **自定义策略**：对关键业务路径、异常链路提高采样率。

---

### 🧪 6. **异步调用与消息队列支持**

* Kafka、RabbitMQ 等中间件也需要传递 TraceContext。
* 使用消息头传递上下文，并在消费者手动创建新的 span，关联 parent span。
* 建议使用官方或社区提供的 instrumentation，例如：

  * `opentelemetry-instrumentation-kafka`
  * `opentelemetry-instrumentation-celery`

---

### 🚀 7. **部署与监控建议**

* 使用 sidecar 或 agent 模式部署 OTEL Collector（采集、过滤、导出）。
* 启用资源检测（host name, container id, region 等）便于分析。
* 配置熔断机制，防止追踪系统故障影响业务流。

---

## 🧰 四、工具链推荐

| 工具/组件             | 功能                    | 说明                                 |
| ----------------- | --------------------- | ---------------------------------- |
| OpenTelemetry SDK | Trace、Metrics、Logs 采集 | 多语言支持                              |
| OTEL Collector    | 中转、转码、过滤              | 可部署为 sidecar / DaemonSet / Gateway |
| Jaeger / Zipkin   | 可视化追踪链路               | 开源、轻量、易部署                          |
| Grafana Tempo     | 可扩展 Trace 存储          | 可与 Loki、Prometheus 配合使用            |
| Grafana / Kibana  | 可视化分析                 | 多数据源支持                             |

---

## 🧠 五、总结

| 关键点        | 建议做法                      |
| ---------- | ------------------------- |
| 上下文传递      | 使用 W3C Trace Context / B3 |
| SDK 接入     | 使用中间件或框架集成方式              |
| Trace 数据存储 | 选择高可用、高压缩比存储方案            |
| 日志关联       | trace\_id 注入日志上下文         |
| 性能开销控制     | 配置采样策略、批量导出机制             |
| 异常链路监控     | 重点服务配置高采样或全采样             |

---

## ☕ Java 示例（Spring Boot + OpenTelemetry + Jaeger）

### 1️⃣ Maven 依赖（Spring Boot 3+）

```xml
<dependency>
    <groupId>io.opentelemetry.instrumentation</groupId>
    <artifactId>opentelemetry-spring-boot-starter</artifactId>
    <version>1.30.0</version>
</dependency>
```

### 2️⃣ application.yml 配置

```yaml
otel:
  exporter:
    otlp:
      endpoint: http://localhost:4317
  resource:
    attributes:
      service.name: order-service
  tracing:
    enabled: true
    sampling:
      probability: 1.0  # 开发全采样，生产建议设置为 0.1 或配置 tail-based sampling
```

### 3️⃣ 控制器 + 调用外部服务示例

```java
@RestController
public class OrderController {

    private final RestTemplate restTemplate;

    public OrderController(RestTemplateBuilder builder) {
        this.restTemplate = builder.build();
    }

    @GetMapping("/order")
    public String createOrder() {
        // 自动创建 span 并关联 traceId
        String response = restTemplate.getForObject("http://inventory-service/inventory", String.class);
        return "Order placed. Inventory Response: " + response;
    }
}
```

### 4️⃣ 日志注入 TraceId

使用 Logback + MDC 自动注入：

```xml
<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - traceId=%X{trace_id} spanId=%X{span_id} %msg%n</pattern>
```

---

## 🐍 Python 示例（FastAPI + OpenTelemetry）

### 1️⃣ 安装依赖

```bash
pip install opentelemetry-sdk opentelemetry-instrumentation-fastapi \
            opentelemetry-exporter-otlp opentelemetry-instrumentation-requests
```

### 2️⃣ 初始化 OTEL（main.py）

```python
from fastapi import FastAPI
from opentelemetry import trace
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import SERVICE_NAME, Resource

# 设置 TracerProvider
trace.set_tracer_provider(
    TracerProvider(
        resource=Resource.create({SERVICE_NAME: "inventory-service"})
    )
)

# 配置导出器
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="http://localhost:4318/v1/traces"))
)

app = FastAPI()

# 自动注入中间件
FastAPIInstrumentor.instrument_app(app)
RequestsInstrumentor().instrument()

@app.get("/inventory")
def check_inventory():
    return {"status": "in-stock"}
```

### 3️⃣ 调用链上下游传播

调用 `inventory-service`：

```python
import requests
@app.get("/order")
def place_order():
    r = requests.get("http://localhost:8001/inventory")
    return {"order_status": "created", "inventory": r.json()}
```

上下文将通过 `traceparent` header 自动传递。

---

## 🎯 最佳实践体现

| 实践点                | 实现说明                                                            |
| ------------------ | --------------------------------------------------------------- |
| 自动 instrumentation | 使用了 `FastAPIInstrumentor` / Spring Boot Starter                 |
| tracecontext 传递    | `requests` 和 `RestTemplate` 均通过自动集成传递 `traceparent`             |
| trace-id 注入日志      | Java 通过 Logback 的 `%X{}` ；Python 可用 `structlog` + 自定义 processor |
| 批量导出 + 性能优化        | 使用 `BatchSpanProcessor` 避免每次请求立即导出                              |
| 可配置资源标签            | 使用 `service.name` 标识服务名，便于追踪分析                                  |
| 导出到集中平台            | 配置 OTLP 导出，可接入 Jaeger、Tempo、Datadog 等                           |

---

## 📦 推荐部署组合

| 组件                      | 说明                  |
| ----------------------- | ------------------- |
| OpenTelemetry Collector | 统一采集、转换、转发 trace 数据 |
| Jaeger / Grafana Tempo  | 可视化链路追踪图            |
| Prometheus + Grafana    | 指标观测                |
| Loki / ELK              | 日志观测，traceId 关联分析   |

---

## ✅ 一、什么是上下文自动传递？

在微服务架构中，一个请求通常会跨多个服务。例如：

```
用户请求 → 网关 → 用户服务 → 订单服务 → 库存服务
```

我们希望从用户开始的一次请求能贯穿整个链路，能在每个服务看到相同的 `trace_id`，这就需要：

> **将 trace 上下文（如 trace\_id、span\_id）自动传递到下一个服务，并被自动识别和继续追踪。**

---

## 🔧 二、自动上下文传递的核心机制

OpenTelemetry 使用标准协议（如 [W3C Trace Context](https://www.w3.org/TR/trace-context/)）来在不同服务之间传递 trace。

### 💡 关键 Header 示例（W3C Trace Context）

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

* `trace-id`: 整个调用链的唯一 ID
* `span-id`: 当前调用的唯一 ID
* `trace-flags`: 采样标记

---

## ⚙️ 三、如何启用自动传递（分语言说明）

---

### ☕ Java（Spring Boot / OpenTelemetry）

#### ✅ 使用 OpenTelemetry Starter（推荐）

```xml
<dependency>
  <groupId>io.opentelemetry.instrumentation</groupId>
  <artifactId>opentelemetry-spring-boot-starter</artifactId>
  <version>1.30.0</version>
</dependency>
```

它会自动完成以下工作：

* 通过拦截器自动读取 `traceparent` header
* 自动在使用 `RestTemplate`, `WebClient`, `gRPC` 时注入上下文
* 自动关联 `trace_id` 与 MDC 日志上下文

📍你只需确保调用时使用框架标准组件（如 `RestTemplate`, `WebClient`），它就会自动附带 trace header。

---

### 🐍 Python（FastAPI / Flask）

#### ✅ 使用自动 Instrumentation：

```bash
pip install opentelemetry-instrumentation-fastapi opentelemetry-instrumentation-requests
```

#### 初始化：

```python
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor

app = FastAPI()
FastAPIInstrumentor.instrument_app(app)
RequestsInstrumentor().instrument()
```

* 自动为 FastAPI 路由创建 span，并提取 trace 上下文（`traceparent`）
* 使用 `requests.get()` 时自动注入 trace header

---

### 🐹 Go（HTTP 或 gRPC）

#### ✅ 使用 OTEL HTTP 插件：

```go
import (
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
    "net/http"
)

handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // handler 自动获取 trace 上下文
})
http.Handle("/my-api", otelhttp.NewHandler(handler, "my-api-handler"))
```

#### 调用下游服务：

```go
client := http.Client{
    Transport: otelhttp.NewTransport(http.DefaultTransport),
}
req, _ := http.NewRequest("GET", "http://other-service", nil)
client.Do(req)
```

这样会自动将 tracecontext 插入 HTTP 请求头。

---

## 📥 四、在消息队列 / Kafka / gRPC 中传递上下文

### 📨 Kafka（Java 示例）

1. 在 Producer 中添加 traceparent 到消息头：

   ```java
   ProducerRecord<String, String> record = new ProducerRecord<>(topic, key, value);
   GlobalOpenTelemetry.getPropagators().getTextMapPropagator().inject(
       Context.current(), record.headers(), setter);
   ```

2. 在 Consumer 中提取并继续追踪：

   ```java
   Context extractedContext = propagators.extract(Context.current(), record.headers(), getter);
   Span span = tracer.spanBuilder("kafka-consume").setParent(extractedContext).startSpan();
   ```

---

## ✅ 五、部署环境要求

* 所有服务都要统一使用支持 OpenTelemetry 的 SDK 和传播机制
* 建议通过 **OpenTelemetry Collector** 中转 trace 数据，简化配置
* 在 Istio / Linkerd 等服务网格中使用时，也可以通过 mesh 自动处理 trace header

---

## 🧪 六、验证自动上下文是否传递成功

你可以在每个服务打印当前 trace\_id 或在日志中观察 trace\_id 是否一致：

Java 示例（Logback）：

```xml
%X{trace_id} %X{span_id} %msg%n
```

Python 示例（structlog）：

```python
log = structlog.get_logger().bind(trace_id=trace.get_current_span().get_span_context().trace_id)
```

---

## 🧠 七、小结

| 项目          | 最佳实践                              |
| ----------- | --------------------------------- |
| 使用标准 Header | 建议使用 `W3C Trace Context`（默认）      |
| HTTP 调用     | 使用框架集成组件（如 RestTemplate、requests） |
| 消息队列 / 异步   | 通过 headers 手动传递上下文                |
| 服务统一 SDK 配置 | 使用相同版本的 OTEL SDK 和 propagator     |
| 日志与追踪关联     | 将 trace\_id 注入日志上下文               |

