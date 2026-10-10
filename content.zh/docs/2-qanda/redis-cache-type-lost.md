---
title: Redis 缓存反序列化丢失类型信息
---

## 问题描述

在 Spring Boot 项目中，使用 Spring Cache 注解（如 `@Cacheable`）缓存 Java 对象时，可能出现缓存读取后的对象类型与原始类型不一致的问题。

例如，缓存方法返回一个 `List<DTO>`，首次调用时，方法正常返回 DTO 对象列表；再次调用时，Spring 从 Redis 中读取缓存数据，但列表中的元素却变成了 `LinkedHashMap`，而不是原本的 DTO 类型。

示例：

```java
@Cacheable(cacheNames = "exampleCache", key = "#id")
public List<ExampleDTO> getData(Long id) {
    return dataService.getData(id);
}
```

首次调用时，返回的数据类型为：

```java
List<ExampleDTO>
```

缓存命中后，读取到的实际数据却可能表现为：

```java
List<LinkedHashMap>
```

当后续代码尝试将列表元素直接转换为 DTO，或者调用 DTO 特有的方法时，就可能出现 `ClassCastException` 等异常。

需要注意的是，这并不是所有 Redis JSON 序列化配置都会出现的问题。具体表现取决于使用的序列化器、Jackson 配置、缓存数据结构及版本兼容性。

## 问题原因及分析

### 1. JSON 默认不保留完整的 Java 类型信息

JSON 是一种数据交换格式，主要描述对象的属性及其值，并不天然保留 Java 对象的完整类型信息。

例如，一个 Java 对象：

```java
public class ExampleDTO {

    private Long id;

    private String name;

    // getter、setter
}
```

序列化后可能得到：

```json
{
  "id": 1,
  "name": "example"
}
```

这段 JSON 描述了对象的数据，但没有明确标明它对应 Java 中的 `ExampleDTO` 类。

如果反序列化时没有提供目标类型，也没有可用的类型元数据，Jackson 可能将 JSON 对象恢复为通用的 `Map` 实现，例如 `LinkedHashMap`。

### 2. 缓存反序列化时，可能缺少明确的目标类型

在普通 Jackson 使用场景中，可以显式指定目标类型：

```java
ExampleDTO dto = objectMapper.readValue(
        json,
        ExampleDTO.class
);
```

因为代码明确指定了 `ExampleDTO.class`，Jackson 就知道应该创建哪一种对象。

但 Spring Cache 的通用缓存序列化场景有所不同。

例如：

```java
@Cacheable(cacheNames = "exampleCache", key = "#id")
public List<ExampleDTO> getData(Long id) {
    // 查询并返回数据
}
```

缓存框架使用配置的序列化器写入和读取缓存值。通用序列化器不一定能直接从缓存读取操作中获得方法声明的完整泛型返回类型。

因此，如果缓存中只有普通 JSON 数据，又没有足够的类型信息，反序列化器就可能无法恢复原始 DTO 类型。

### 3. 序列化器的类型处理配置可能不一致

Spring Data Redis 提供了多种序列化方式，例如：

* `StringRedisSerializer`
* `JdkSerializationRedisSerializer`
* `Jackson2JsonRedisSerializer`
* `GenericJackson2JsonRedisSerializer`

不同序列化器的行为并不相同。

其中，`GenericJackson2JsonRedisSerializer` 常用于通用 Java 对象的 JSON 序列化，但具体的类型信息处理行为还与构造方式、Jackson 配置及 Spring Data Redis 版本有关。

例如，使用自定义 `ObjectMapper` 时：

```java
GenericJackson2JsonRedisSerializer serializer =
        new GenericJackson2JsonRedisSerializer(objectMapper);
```

不能仅凭使用了 `GenericJackson2JsonRedisSerializer` 就认定类型信息一定会被正确保存和恢复。

如果序列化阶段没有写入所需的类型信息，或者反序列化阶段无法识别这些信息，就可能出现对象类型丢失的问题。

### 4. Redis 中可能残留旧格式的缓存数据

修改序列化配置后，Redis 中已经存在的缓存数据不会自动按照新规则重新序列化。

例如：

1. 旧配置将对象保存为普通 JSON。
2. 新配置启用了类型信息处理。
3. 应用读取之前写入的缓存数据。
4. 旧数据不符合新配置预期的格式，导致反序列化失败或无法恢复原始类型。

因此，修改缓存序列化配置后，需要考虑旧缓存数据的兼容性。

## 最佳实践及分析

### 1. 统一配置 Redis 专用 ObjectMapper

对于需要缓存多种 Java 对象的应用，可以创建专门用于 Redis 的 `ObjectMapper`，集中配置日期时间处理和类型信息处理。

示例：

```java
@Bean
public ObjectMapper redisObjectMapper() {
    ObjectMapper objectMapper = new ObjectMapper();

    // 支持 Java 8 日期时间类型
    objectMapper.registerModule(new JavaTimeModule());

    // 日期时间采用更易读的文本形式输出
    objectMapper.disable(
            SerializationFeature.WRITE_DATES_AS_TIMESTAMPS
    );

    // 限制允许参与多态类型处理的类
    BasicPolymorphicTypeValidator ptv =
            BasicPolymorphicTypeValidator.builder()
                    .allowIfSubType("com.example.")
                    .build();

    // 配置默认类型信息处理
    objectMapper.activateDefaultTyping(
            ptv,
            ObjectMapper.DefaultTyping.NON_FINAL
    );

    return objectMapper;
}
```

相关配置说明：

| 配置                                     | 作用                   |
| -------------------------------------- | -------------------- |
| `new ObjectMapper()`                   | 创建 Jackson JSON 处理对象 |
| `registerModule(new JavaTimeModule())` | 支持 Java 日期时间类型       |
| `WRITE_DATES_AS_TIMESTAMPS`            | 控制日期时间的序列化格式         |
| `BasicPolymorphicTypeValidator`        | 限制允许参与多态类型处理的类型      |
| `activateDefaultTyping()`              | 为符合规则的类型配置默认类型信息处理   |

需要注意：

* `com.example.` 只是示例包路径，应替换为项目中实际需要允许的包路径。
* `DefaultTyping.NON_FINAL` 并不意味着所有类型都会自动携带类型信息。
* 启用默认多态类型处理会影响反序列化行为，应限制可信类型范围，避免对不可信数据进行不受限制的反序列化。
* 应根据项目使用的 Jackson 和 Spring Data Redis 版本，确认相应配置方式的兼容性。

### 2. 为 RedisTemplate 配置统一的序列化器

如果项目使用 `RedisTemplate` 手动操作 Redis，可以为 key 和 value 配置不同的序列化器。

示例：

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(
        RedisConnectionFactory connectionFactory,
        @Qualifier("redisObjectMapper")
        ObjectMapper redisObjectMapper) {

    RedisTemplate<String, Object> template =
            new RedisTemplate<>();

    template.setConnectionFactory(connectionFactory);

    // Key 使用字符串序列化
    StringRedisSerializer stringSerializer =
            new StringRedisSerializer();

    // Value 使用 JSON 序列化
    GenericJackson2JsonRedisSerializer jsonSerializer =
            new GenericJackson2JsonRedisSerializer(
                    redisObjectMapper
            );

    template.setKeySerializer(stringSerializer);
    template.setHashKeySerializer(stringSerializer);

    template.setValueSerializer(jsonSerializer);
    template.setHashValueSerializer(jsonSerializer);

    template.afterPropertiesSet();

    return template;
}
```

这样配置后：

* 普通 key 使用字符串序列化。
* Hash 内部字段名使用字符串序列化。
* 普通 value 使用 JSON 序列化。
* Hash value 使用 JSON 序列化。

使用相同的 `ObjectMapper` 可以减少不同 Redis 操作入口之间的序列化配置差异。

不过，`RedisTemplate` 与 Spring Cache 是不同的使用入口。仅配置 `RedisTemplate` 并不能保证 `@Cacheable` 使用相同的序列化配置。

### 3. 为 Spring Cache 单独配置 RedisCacheManager

使用 `@Cacheable`、`@CachePut` 和 `@CacheEvict` 时，通常需要通过 `RedisCacheManager` 配置缓存序列化规则。

示例：

```java
@Bean
public RedisCacheConfiguration redisCacheConfiguration(
        @Qualifier("redisObjectMapper")
        ObjectMapper redisObjectMapper) {

    GenericJackson2JsonRedisSerializer serializer =
            new GenericJackson2JsonRedisSerializer(
                    redisObjectMapper
            );

    return RedisCacheConfiguration.defaultCacheConfig()
            .serializeKeysWith(
                    RedisSerializationContext.SerializationPair
                            .fromSerializer(
                                    new StringRedisSerializer()
                            )
            )
            .serializeValuesWith(
                    RedisSerializationContext.SerializationPair
                            .fromSerializer(serializer)
            );
}
```

然后配置缓存管理器：

```java
@Bean
public RedisCacheManager redisCacheManager(
        RedisConnectionFactory connectionFactory,
        RedisCacheConfiguration redisCacheConfiguration) {

    return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(redisCacheConfiguration)
            .build();
}
```

通过这种方式，Spring Cache 可以使用指定的 key 和 value 序列化规则。

**关键点：** 如果问题发生在 `@Cacheable` 的缓存读取过程中，应重点检查 `RedisCacheManager` 使用的 `RedisCacheConfiguration`，而不能只检查 `RedisTemplate`。

### 4. 区分全局 ObjectMapper 与 Redis 专用 ObjectMapper

在项目中，可以分别维护：

* 全局 `ObjectMapper`：用于普通 JSON 请求、响应及业务序列化。
* Redis 专用 `ObjectMapper`：用于 Redis 缓存数据的序列化和反序列化。

例如：

```java
@Qualifier("redisObjectMapper")
ObjectMapper redisObjectMapper
```

这种方式可以明确指定依赖，避免 Redis 配置意外影响其他 JSON 处理场景。

如果项目中存在多个 `ObjectMapper` Bean，建议使用 `@Qualifier` 明确注入目标 Bean。

### 5. 谨慎启用默认类型信息处理

启用：

```java
objectMapper.activateDefaultTyping(
        ptv,
        ObjectMapper.DefaultTyping.NON_FINAL
);
```

能够帮助 Jackson 在适用的场景中保存和恢复类型信息，但也会扩大反序列化行为的影响范围。

最佳实践包括：

1. 限制允许的类型范围。
2. 不要对不可信的外部 JSON 无条件启用多态反序列化。
3. 确保序列化和反序列化配置相互兼容。
4. 检查集合、数组、DTO、日期时间类型等常见数据结构。
5. 根据项目版本核对 `GenericJackson2JsonRedisSerializer` 的具体行为。

如果缓存类型范围固定，也可以考虑针对特定缓存使用显式目标类型的序列化器。这样可以减少对全局默认多态类型处理的依赖，但需要为不同缓存类型维护相应配置。

### 6. 修改配置后处理旧缓存

修改序列化规则后，应考虑旧缓存的兼容性。

建议：

1. 在开发或测试环境中清理对应的旧缓存。
2. 使用新配置重新写入缓存。
3. 再次读取缓存并检查返回对象类型。
4. 如果使用多实例部署，确保各实例使用兼容的序列化配置。
5. 如果需要平滑升级，考虑缓存版本化或主动失效策略。

不要默认认为旧 JSON 数据能够被新序列化配置自动兼容。

### 7. 建立缓存类型验证测试

建议编写测试验证缓存读写后的对象类型。

例如：

```java
@Test
void shouldPreserveDtoTypeAfterRedisSerialization() {
    ExampleDTO dto = new ExampleDTO();
    dto.setId(1L);
    dto.setName("example");

    GenericJackson2JsonRedisSerializer serializer =
            new GenericJackson2JsonRedisSerializer(
                    redisObjectMapper
            );

    byte[] bytes = serializer.serialize(dto);

    Object result = serializer.deserialize(bytes);

    assertNotNull(result);
    assertEquals(ExampleDTO.class, result.getClass());
}
```

对于集合，还应单独测试列表及其元素类型：

```java
@Test
void shouldPreserveDtoTypeInListAfterRedisSerialization() {
    List<ExampleDTO> list = new ArrayList<>();

    ExampleDTO dto = new ExampleDTO();
    dto.setId(1L);
    dto.setName("example");

    list.add(dto);

    GenericJackson2JsonRedisSerializer serializer =
            new GenericJackson2JsonRedisSerializer(
                    redisObjectMapper
            );

    byte[] bytes = serializer.serialize(list);

    Object result = serializer.deserialize(bytes);

    assertInstanceOf(List.class, result);

    List<?> resultList = (List<?>) result;

    assertFalse(resultList.isEmpty());
    assertEquals(
            ExampleDTO.class,
            resultList.get(0).getClass()
    );
}
```

以上示例用于说明测试思路。实际使用时，需要根据项目所用 Spring Data Redis 版本、序列化器 API 和 DTO 定义调整代码。

测试时应特别注意：仅验证结果是 `List` 并不够，还需要检查集合元素是否恢复成预期类型。

## 总结

Redis 缓存反序列化后出现 `LinkedHashMap`，通常与 JSON 数据缺少可用的 Java 类型信息、反序列化时无法获得明确的目标类型，或者序列化配置不一致有关。

对于使用 Spring Cache 的应用，可以从以下几个方面排查和改进：

1. 检查 `RedisCacheManager` 使用的序列化配置。
2. 根据需要配置 Redis 专用 `ObjectMapper`。
3. 确保序列化和反序列化使用兼容的规则。
4. 根据安全要求限制允许参与多态类型处理的类。
5. 修改配置后处理旧缓存数据。
6. 通过测试验证 DTO、集合及集合元素的实际类型。

需要牢记的是：**JSON 数据能够成功读取，不代表 Java 对象类型一定恢复正确。**

在通用缓存场景中，必须同时考虑数据内容、类型信息、序列化器配置和反序列化目标类型，才能保证缓存读写行为符合预期。
