---
title: Redis 中 Jackson 序列化 Instant 失败
---

## 问题现象

Redis 保存对象时发生 JSON 序列化异常：

```text
SerializationException: Could not write JSON:
Java 8 date/time type java.time.Instant not supported by default
```

异常涉及对象中的 `Instant` 类型字段。

---

## 根本原因

项目使用：

```java
GenericJackson2JsonRedisSerializer
```

进行 Redis JSON 序列化。

虽然项目已经引入：

```gradle
implementation 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310'
```

并配置了：

```java
objectMapper.registerModule(new JavaTimeModule());
```

但原来的 Redis 配置使用的是：

```java
new GenericJackson2JsonRedisSerializer();
```

这个 Serializer **不会自动使用 Spring 容器中的 `ObjectMapper` Bean**，而是内部使用自己的 Jackson 配置。

因此实际执行 Redis 序列化时，`ObjectMapper` 没有注册 `JavaTimeModule`，导致：

```text
Instant
↓
Jackson 序列化
↓
没有 JavaTimeModule
↓
不支持 Java 8 Date/Time
↓
SerializationException
```

---

### 最终解决方案

核心不是修改业务对象中的 `Instant`，而是让 Redis Serializer **明确使用已经配置好的 `ObjectMapper`**。

也就是：

```text
Spring ObjectMapper
        ↓
JavaTimeModule
        ↓
GenericJackson2JsonRedisSerializer
        ↓
RedisTemplate / RedisCacheManager
        ↓
Redis
```

Redis 配置中的：

```java
new GenericJackson2JsonRedisSerializer()
```

改为使用注入的：

```java
new GenericJackson2JsonRedisSerializer(objectMapper)
```

并让 `RedisTemplate` 和 `RedisCacheManager` 使用同一套 Serializer。

---

### 另外一个需要注意的点

如果项目没有其他特殊 Jackson 配置，**不建议自己创建一个裸的 `ObjectMapper` Bean**：

```java
@Bean
public ObjectMapper objectMapper() {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.registerModule(new JavaTimeModule());
    return objectMapper;
}
```

因为 Spring Boot 本身已经会自动配置 Jackson，包括 Java 8 时间类型支持。

更推荐：

1. 保留 `jackson-datatype-jsr310` 依赖；
2. 删除自己创建的 `JacksonConfig`；
3. 直接在 `RedisConfig` 中注入 Spring Boot 自动配置的 `ObjectMapper`；
4. 显式传给 `GenericJackson2JsonRedisSerializer`。

这样既能解决 Redis 的问题，也不会因为自己创建 `ObjectMapper` 而意外影响 Spring MVC / Controller 的 JSON 序列化配置。

**一句话总结：**

> 问题不是 `Instant` 不能存 Redis，而是 Redis 使用的 Jackson Serializer 没有拿到配置了 `JavaTimeModule` 的 `ObjectMapper`；解决方案是让 Redis Serializer 显式使用 Spring 管理的 `ObjectMapper`。
