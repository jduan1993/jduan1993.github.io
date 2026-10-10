---
title: JPQL 中 DISTINCT 与分页共存时的处理
---

## 问题描述

在 Spring Data JPA 中，使用 `@Query` 编写 JPQL 查询时，如果同时使用 `DISTINCT`、`JOIN FETCH` 和 `Pageable`，可能出现分页性能下降、查询结果不符合预期，甚至抛出异常等问题。

典型的查询方式如下：

```java
@Query("SELECT DISTINCT e FROM Entity e LEFT JOIN FETCH e.relatedEntities")
Page<Entity> findAllWithRelatedEntities(Pageable pageable);
```

其中：

* `DISTINCT`：用于消除查询结果中的重复实体。
* `JOIN FETCH`：在查询主实体时，同时加载关联实体。
* `Pageable`：用于指定分页参数，包括页码、每页大小和排序条件。

当关联属性是集合类型，例如 `@OneToMany` 或 `@ManyToMany` 时，这种查询方式需要特别注意。

**核心问题在于：数据库分页针对 SQL 查询结果行，而业务分页通常针对去重后的主实体。**

当一个主实体关联多个实体时，JOIN 操作可能产生多条对应的 SQL 结果行，导致分页与实体数量之间出现差异。

---

## 问题原因及分析

### 1. JOIN 操作可能产生重复行

假设主实体与关联实体之间存在一对多或多对多关系。

当执行 JOIN 操作时，一个主实体可能对应多条 SQL 结果行。

例如，某个主实体关联了多个实体，JOIN 后可能产生如下结果：

| 主实体 ID | 关联实体 ID |
| ------ | ------- |
| 1      | 101     |
| 1      | 102     |
| 1      | 103     |
| 2      | 201     |
| 3      | 301     |

虽然结果中包含五行数据，但实际上只有三个不同的主实体。

如果直接对 JOIN 结果执行分页，分页范围可能包含同一个主实体对应的多行记录，导致实际返回的主实体数量少于预期。

### 2. DISTINCT 的作用及执行顺序

在 SQL 中，`DISTINCT` 用于消除查询投影结果中的重复行。

通常可以将查询逻辑理解为：

```text
FROM / JOIN
    ↓
WHERE
    ↓
GROUP BY / HAVING（如果存在）
    ↓
SELECT / DISTINCT
    ↓
ORDER BY
    ↓
OFFSET / LIMIT
```

因此，对于普通 SQL 查询，数据库通常先完成 `DISTINCT` 去重，再应用分页限制。

但需要区分两个概念：

* **SQL 结果行去重**：数据库根据查询投影结果消除重复行。
* **ORM 实体去重**：JPA Provider 将查询结果转换为实体对象时，对重复实体进行处理。

二者并不完全等价。

在 JPQL 中，`SELECT DISTINCT e` 表示查询不同的实体结果。Hibernate 等 ORM 框架可能结合实体标识和自身的结果处理机制完成实体去重。

因此，不能简单地认为 `DISTINCT` 一定能解决所有分页问题。

### 3. JOIN FETCH 与集合关联的特殊性

`JOIN FETCH` 不仅用于关联查询，还要求 ORM 在当前查询中初始化指定的关联属性。

当关联属性是集合类型时，数据库返回的结果行可能包含同一个主实体的多条记录。

例如：

```java
@Query("SELECT DISTINCT e FROM Entity e LEFT JOIN FETCH e.relatedEntities")
List<Entity> findAllWithRelatedEntities();
```

该查询可以同时获取主实体及其关联集合，并通过 `DISTINCT` 表达去重意图。

但是，如果将返回类型改为 `Page<Entity>`，并传入 `Pageable`，问题就会变得复杂。

数据库的分页操作作用于 SQL 查询结果，而 Hibernate 需要保证集合关联能够正确组装。

如果直接在 SQL 层限制结果行数，可能会截断某个主实体对应的关联结果，导致关联集合无法完整加载。

为了避免这种情况，Hibernate 在部分版本和配置下，可能忽略数据库层面的分页限制，将符合条件的结果加载到内存后再进行分页。

这会带来以下风险：

* 数据库返回大量结果行。
* ORM 需要处理大量重复的实体数据。
* 内存占用增加。
* 查询性能随数据规模增长而下降。
* 在特定版本或配置下，可能产生警告或异常。

因此，**`DISTINCT` 可以解决结果重复问题，但不能从根本上解决集合关联 `JOIN FETCH` 的分页问题。**

### 4. COUNT 查询也需要考虑 DISTINCT

Spring Data JPA 在执行分页查询时，通常需要获取两部分数据：

1. 当前页的实体内容。
2. 满足查询条件的总记录数。

第二部分通常通过 `COUNT` 查询完成。

如果查询中存在 JOIN，并且 JOIN 会使同一个主实体出现多次，那么直接使用：

```sql
COUNT(e.id)
```

可能得到重复计数。

在这种情况下，应根据实际查询结构考虑：

```sql
COUNT(DISTINCT e.id)
```

但是，如果分页查询本身不需要 JOIN，那么更好的方式通常是直接统计主实体数量，避免引入不必要的关联查询。

对于复杂 JPQL，建议显式声明 `countQuery`，确保总数统计与主查询的筛选条件一致。

### 5. DISTINCT 不应与所有 JOIN 查询机械地绑定

是否需要 `DISTINCT`，取决于 JOIN 的类型、查询投影以及 ORM 的结果处理方式。

| 查询方式                     | 是否通常需要 DISTINCT    | 分页风险    |
| ------------------------ | ------------------ | ------- |
| 仅查询主实体，不关联集合             | 通常不需要              | 较低      |
| 普通 JOIN，且关联结果可能重复主实体     | 视查询结果而定            | 需要评估    |
| 普通 JOIN，并使用 DISTINCT     | 通常可以消除重复结果         | 取决于查询结构 |
| 集合关联的 JOIN FETCH，返回 Page | 即使使用 DISTINCT 也有风险 | 较高      |
| 先分页查询主实体，再批量查询关联数据       | 主查询通常不需要           | 较低      |

---

## 最佳实践及分析

### 1. 优先采用两阶段查询

对于需要分页查询主实体，同时加载集合关联的场景，推荐采用两阶段查询：

1. 第一次查询只负责分页获取主实体。
2. 第二次查询根据当前页主实体的 ID，批量加载关联实体。

这种方式可以避免集合关联 `JOIN FETCH` 与数据库分页之间的冲突。

#### 第一步：分页查询主实体

```java
@Query(
    value = """
        SELECT e
        FROM Entity e
        WHERE e.deleted = false
        """,
    countQuery = """
        SELECT COUNT(e.id)
        FROM Entity e
        WHERE e.deleted = false
        """
)
Page<Entity> findEntityPage(Pageable pageable);
```

这里仅查询主实体，不使用集合关联的 `JOIN FETCH`。

由于查询不会因集合关联而产生重复行，因此通常不需要 `DISTINCT`。

显式声明 `countQuery`，可以让分页总数统计逻辑更加清晰。

#### 第二步：批量加载关联实体

```java
@Query("""
    SELECT DISTINCT e
    FROM Entity e
    LEFT JOIN FETCH e.relatedEntities
    WHERE e.id IN :ids
    """)
List<Entity> findWithRelatedEntitiesByIds(
    @Param("ids") Collection<Long> ids
);
```

该查询只针对当前页的主实体加载关联集合，不负责分页，因此不存在集合关联导致的数据库分页问题。

当 ID 集合为空时，应跳过第二次查询，避免空集合 `IN` 查询的兼容性问题。

#### 第三步：确保分页结果正确组装

需要注意，第二次查询返回的是根据 ID 查询得到的实体列表，其顺序不一定与第一次分页查询的顺序一致。

此外，如果两次查询处于同一个持久化上下文中，JPA Provider 通常会对相同数据库标识的实体使用同一个受管实体实例。

因此，不能简单地假设第二次查询返回的列表会自动替换第一次查询中的列表。

推荐做法包括：

* 在同一事务中执行两次查询。
* 将第二次查询结果转换为以实体 ID 为键的 Map。
* 按第一次分页查询的实体顺序重新组装结果。
* 或者使用 DTO 投影，将分页数据和关联数据显式组装。

通过这种方式，可以在避免 N+1 查询的同时，保留正确的分页顺序。

### 2. 使用 EntityGraph 控制关联加载

如果希望通过声明式方式指定查询时需要加载的关联属性，可以考虑 `@EntityGraph`。

例如：

```java
@EntityGraph(attributePaths = {"relatedEntities"})
@Query("SELECT e FROM Entity e WHERE e.deleted = false")
Page<Entity> findAllWithRelatedEntities(Pageable pageable);
```

需要注意，`@EntityGraph` 只是声明需要加载哪些关联属性，并不保证一定采用单条 SQL，也不保证一定避免集合关联的分页问题。

当 EntityGraph 导致 Hibernate 对集合关联进行 JOIN FETCH 时，仍然可能触发内存分页。

因此：

* 对于单值关联，例如 `@ManyToOne` 和 `@OneToOne`，EntityGraph 通常比较适合与分页结合使用。
* 对于集合关联，例如 `@OneToMany` 和 `@ManyToMany`，应确认实际生成的 SQL 和分页行为。
* 如果需要稳定、可预测的分页性能，优先考虑两阶段查询。

### 3. 使用批量加载减少 N+1 查询

如果不希望使用 JOIN FETCH，也可以考虑 Hibernate 的批量加载机制。

例如，在实体关联属性上配置：

```java
@BatchSize(size = 50)
@OneToMany(mappedBy = "parent", fetch = FetchType.LAZY)
private List<RelatedEntity> relatedEntities;
```

当访问多个实体的懒加载关联时，Hibernate 可以尝试将多个关联加载请求合并为批量查询。

也可以根据实际情况使用 `hibernate.default_batch_fetch_size` 配置全局批量加载大小。

这种方式通常能够减少 N+1 查询，但它与两阶段查询的行为不同：

* 两阶段查询明确执行批量关联查询。
* 批量懒加载依赖后续对关联属性的访问，以及 Hibernate 的批量加载机制。

应根据关联数据的使用方式和实际 SQL 执行情况选择合适的方案。

### 4. 明确声明 countQuery

对于带有分页参数的复杂查询，建议显式定义 `countQuery`。

例如：

```java
@Query(
    value = """
        SELECT e
        FROM Entity e
        WHERE e.status = :status
        """,
    countQuery = """
        SELECT COUNT(e.id)
        FROM Entity e
        WHERE e.status = :status
        """
)
Page<Entity> findByStatus(
    @Param("status") Status status,
    Pageable pageable
);
```

这样可以明确区分：

* 内容查询负责返回当前页数据。
* 总数查询负责计算满足条件的记录数量。

如果内容查询使用了 JOIN，并可能产生重复主实体，则需要重新审视 `countQuery` 是否需要使用 `COUNT(DISTINCT e.id)`。

不要机械地将内容查询中的所有 JOIN 和 DISTINCT 复制到总数查询中。应根据实际的筛选条件和关联关系，构造满足业务语义的计数查询。

### 5. 分页查询必须具有稳定排序

即使解决了 DISTINCT 和 JOIN FETCH 的问题，分页查询仍然需要稳定的排序条件。

例如：

```java
Pageable pageable = PageRequest.of(
    page,
    size,
    Sort.by(Sort.Direction.ASC, "id")
);
```

如果排序字段不是唯一的，建议追加唯一字段作为次级排序条件。

稳定排序可以降低翻页时出现重复记录或遗漏记录的风险。

对于数据量较大、频繁发生插入或删除的场景，还需要考虑 OFFSET 分页的性能及数据变化带来的影响，必要时可以使用基于唯一键的游标分页。

### 6. 在开发环境中检测内存分页

对于使用 Hibernate 的项目，可以开启以下配置：

```properties
spring.jpa.properties.hibernate.query.fail_on_pagination_over_collection_fetch=true
```

开启后，当 Hibernate 检测到对集合关联进行内存分页时，可以抛出异常，而不是静默地执行可能存在性能问题的查询。

该配置适合用于开发和测试环境，帮助及早发现不合理的查询设计。

在升级 Hibernate 版本或修改关联映射后，也建议通过 SQL 日志和集成测试确认实际行为。

---

## 总结

`DISTINCT`、`JOIN FETCH` 和 `Pageable` 分别解决不同的问题：

* `DISTINCT`：用于消除查询结果中的重复实体或重复投影结果。
* `JOIN FETCH`：用于在查询过程中加载关联属性。
* `Pageable`：用于定义分页范围、排序及分页元数据。

当三者同时使用，并且 `JOIN FETCH` 加载的是集合关联时，需要特别注意数据库分页与 ORM 实体组装之间的差异。

**最推荐的实践是：分页查询主实体，再根据当前页实体的 ID 批量加载关联集合。**

这种方式具有以下优点：

1. 分页基于主实体执行，不会因为关联集合的数量不同而影响每页的主实体数量。
2. 不需要在分页查询中使用集合关联的 `JOIN FETCH`。
3. 可以通过批量查询避免 N+1 查询。
4. 可以将总数统计与内容查询分开设计，降低查询复杂度。
5. 更容易控制 SQL 执行次数、分页顺序和整体性能。

最后需要明确：**`DISTINCT` 并不是分页问题的通用解决方案。对于集合关联的分页查询，应优先从查询结构上规避问题，而不是单纯依赖 DISTINCT。**
