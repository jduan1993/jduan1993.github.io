---
title: Spring 同类内部调用导致事务失效
---

## 问题描述

在 Spring 项目中，使用 `@Transactional` 注解声明事务时，如果通过同一个类中的 `this` 调用另一个带有 `@Transactional` 注解的方法，可能导致事务增强不生效。

例如：

```java
@Service
public class ExampleService {

    public void execute() {
        this.save();
    }

    @Transactional
    public void save() {
        // 数据库操作
    }
}
```

当外部调用 `execute()` 时，虽然 `save()` 方法声明了 `@Transactional`，但由于内部调用没有经过 Spring 事务代理，`save()` 方法上的事务配置通常不会被单独拦截和应用。

需要注意，同类内部调用并不意味着任何情况下都没有事务。如果外部调用已经开启事务，内部方法仍可能在该事务上下文中执行。真正的问题是：**内部调用绕过了 Spring AOP 代理，导致目标方法上的事务拦截逻辑无法正常触发。**

## 问题原因及分析

### 1. Spring 事务依赖 AOP 代理机制

Spring 声明式事务主要通过 AOP 代理实现。Spring 容器会根据配置，为需要事务增强的 Bean 创建代理对象，并在方法调用过程中执行事务拦截逻辑。

当外部调用带有 `@Transactional` 注解的方法时，典型执行流程如下：

```text
外部调用者
    |
    v
Spring 代理对象
    |
    v
事务拦截器
    |
    +--> 开启或加入事务
    |
    v
目标对象的方法
    |
    +--> 执行业务逻辑
    |
    v
提交或回滚事务
```

事务拦截器会根据事务传播行为、事务属性及当前事务状态，决定如何处理事务。

因此，`@Transactional` 并不是简单地在方法执行前后自动添加事务代码，而是依赖 Spring 的代理机制对方法调用进行增强。

### 2. 同类内部调用绕过代理

当一个类中的方法通过 `this` 调用另一个方法时，调用直接发生在当前目标对象内部，不会重新经过 Spring 代理。

例如：

```java
@Service
public class ExampleService {

    public void execute() {
        this.save();
    }

    @Transactional
    public void save() {
        // 数据库操作
    }
}
```

外部调用 `execute()` 时，执行流程如下：

```text
外部调用者
    |
    v
Spring 代理对象
    |
    v
ExampleService.execute()
    |
    v
this.save()
    |
    v
ExampleService.save()
```

其中，`this.save()` 是目标对象内部的普通 Java 方法调用，不会再次进入 Spring 事务代理。

因此，`save()` 方法上的 `@Transactional` 注解不会因为这次内部调用而触发独立的事务拦截逻辑。

需要区分两种情况：

* **当前没有事务**：如果内部调用的方法依赖自身的 `@Transactional` 注解开启事务，由于调用绕过代理，事务可能不会开启。
* **当前已经存在事务**：如果外部方法已经通过代理开启事务，内部方法仍然可以在已有事务中执行，但其自身声明的事务传播行为、隔离级别等属性可能无法通过这次内部调用生效。

### 3. 事务是否生效还取决于其他条件

即使方法调用经过 Spring 代理，也不意味着事务一定按照预期生效，还需要检查以下因素：

* **方法可代理性**：基于 JDK 动态代理或 CGLIB 的代理方式不同，对可拦截方法的要求也有所区别。
* **方法可见性**：在 Spring 常见的代理模式下，`private` 方法不能通过代理正常实现事务增强；其他非公开方法的支持情况取决于代理方式及相关配置。
* **异常处理方式**：默认情况下，未处理的 `RuntimeException` 和 `Error` 通常会触发回滚，受检异常默认不会触发回滚，除非通过 `rollbackFor` 等配置调整规则。
* **事务管理器配置**：需要确认事务管理器正确配置，且相关数据库操作参与同一个事务。
* **代理对象使用方式**：调用必须经过 Spring 容器管理的代理对象，直接创建对象或绕过代理都可能导致事务增强失效。

因此，排查事务问题时，不能只检查是否添加了 `@Transactional`，还应确认方法调用链、代理方式、事务配置和异常处理逻辑。

## 最佳实践及分析

### 1. 将事务方法拆分到独立的 Service（推荐）

如果某个方法需要独立的事务增强，优先考虑将其提取到另一个 Spring Bean 中，通过依赖注入调用。

例如：

```java
@Service
public class TaskService {

    private final TransactionService transactionService;

    public TaskService(TransactionService transactionService) {
        this.transactionService = transactionService;
    }

    public void execute() {
        transactionService.save();
    }
}
```

事务方法定义在独立的 Service 中：

```java
@Service
public class TransactionService {

    @Transactional
    public void save() {
        // 数据库操作
    }
}
```

调用流程如下：

```text
TaskService
    |
    v
TransactionService 代理对象
    |
    v
事务拦截器
    |
    v
TransactionService.save()
```

这种方式能够让方法调用自然地经过 Spring 代理，事务边界更加清晰。

同时，拆分后的组件职责更明确，也更便于单元测试和事务行为测试。

### 2. 通过自身代理对象调用（有条件使用）

如果不适合拆分 Service，也可以通过 Spring 注入的自身代理对象调用事务方法。

例如：

```java
@Service
public class ExampleService {

    private final ExampleService self;

    public ExampleService(@Lazy ExampleService self) {
        this.self = self;
    }

    public void execute() {
        self.save();
    }

    @Transactional
    public void save() {
        // 数据库操作
    }
}
```

这里的 `self` 应当引用 Spring 代理对象，而不是当前目标对象本身。

使用这种方式需要注意循环依赖、代理注入方式及应用上下文配置等问题，因此通常不如拆分 Service 直观。

### 3. 通过 AopContext 获取当前代理对象

Spring 也提供了通过 `AopContext.currentProxy()` 获取当前代理对象的方式。

首先，需要启用代理暴露：

```java
@EnableAspectJAutoProxy(exposeProxy = true)
```

然后通过代理对象调用目标方法：

```java
@Service
public class ExampleService {

    public void execute() {
        ExampleService proxy =
                (ExampleService) AopContext.currentProxy();

        proxy.save();
    }

    @Transactional
    public void save() {
        // 数据库操作
    }
}
```

这种方式可以让内部调用重新经过代理，但它依赖特定的 AOP 配置，并且要求当前调用处于有效的 Spring AOP 代理调用上下文中。

由于这种方式会增加代码对 Spring AOP 的耦合，一般不作为首选方案。

### 4. 正确理解事务传播行为

事务传播行为决定当前方法被调用时，应该如何处理已有事务。

例如：

* `Propagation.REQUIRED`：存在事务时加入当前事务；不存在时创建新事务。
* `Propagation.REQUIRES_NEW`：挂起当前事务，并创建一个新的独立事务。
* `Propagation.NESTED`：在支持嵌套事务的事务管理器中，尝试在当前事务内创建嵌套事务。

需要注意，**事务传播行为只有在事务拦截器实际执行时才能按配置生效。**

如果同类内部调用绕过代理，即使方法声明了 `REQUIRES_NEW`，也不会仅凭该注解自动创建新事务。

此外，`REQUIRES_NEW` 和 `NESTED` 的实际行为还取决于事务管理器及底层资源的支持情况。

### 5. 正确处理异常与回滚规则

事务是否回滚不仅取决于事务是否成功开启，还与异常类型和异常传播方式有关。

例如：

```java
@Transactional
public void save() {
    // 数据库操作

    throw new IllegalStateException("operation failed");
}
```

默认情况下，未处理的运行时异常通常会触发事务回滚。

如果方法捕获异常后没有继续抛出，也没有通过其他方式明确标记事务回滚，则事务可能正常提交。

因此，应结合实际异常处理逻辑检查事务结果，避免将所有事务异常都归因于同类内部调用。

## 总结

Spring 声明式事务依赖 AOP 代理机制实现。同一个类中通过 `this.xxx()` 调用另一个带有 `@Transactional` 注解的方法时，调用会直接进入目标对象，而不会重新经过 Spring 事务代理，因此目标方法自身的事务增强通常不会生效。

但这不代表内部方法一定处于无事务状态。如果调用者已经开启事务，内部方法仍可能参与当前事务，只是其自身声明的事务属性可能无法按预期应用。

解决此类问题时，优先将需要事务增强的方法拆分到独立的 Spring Bean 中，通过代理对象调用。同时，应检查方法可代理性、事务传播行为、异常处理方式及事务管理器配置。

**核心原则：`@Transactional` 注解能否生效，不仅取决于是否添加注解，更取决于方法调用是否经过 Spring 事务代理，以及事务配置是否满足生效条件。**
