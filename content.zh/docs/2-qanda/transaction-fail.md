---
title: 同一个类中事务失效问题
---

是的，在 **Spring 事务（`@Transactional`）场景下**，同一个类中通过 `this` 调用另一个方法，通常会导致事务失效。

原因不是 `this` 本身有什么问题，而是 **Spring 事务依赖代理机制（AOP）实现**。

---

## 1. 正常事务调用（事务生效）

例如：

```java
@Service
public class UserService {

    @Transactional
    public void saveUser() {
        // 保存用户
    }
}
```

Spring 启动后，并不是直接把你的 `UserService` 放进容器，而是创建一个代理对象：

```
调用者
  |
  v
UserService代理对象
  |
  |  开启事务
  v
真正的UserService对象
  |
  saveUser()
```

调用：

```java
userService.saveUser();
```

实际上走的是：

```
代理对象.saveUser()
```

代理负责：

1. 开启事务
2. 调用真实方法
3. 提交/回滚事务

---

## 2. 同类内部 this 调用（事务失效）

例如：

```java
@Service
public class UserService {

    public void test() {
        this.saveUser();
    }


    @Transactional
    public void saveUser() {
        // 数据库操作
    }
}
```

调用：

```java
userService.test();
```

流程：

```
调用者
 |
 v
代理对象.test()
 |
 v
真实UserService.test()
 |
 v
this.saveUser()
```

关键在这里：

```java
this.saveUser();
```

调用的是：

```
真实对象.saveUser()
```

而不是：

```
代理对象.saveUser()
```

所以：

```
没有经过Spring事务拦截器
```

事务不会开启。

---

## 3. 图示理解

### 正常调用：

```
Controller
    |
    v
UserService代理
    |
    | @Transactional拦截
    v
UserService.save()
```

事务生效。

---

### this调用：

```
Controller
    |
    v
UserService代理
    |
    v
UserService.test()
    |
    v
this.save()
    |
    v
UserService.save()
```

绕过代理。

事务失效。

---

## 4. 怎么解决？

### 方法1：拆成两个 Service（推荐）

例如：

```java
@Service
public class UserService {

    @Autowired
    private OrderService orderService;


    public void test() {
        orderService.saveOrder();
    }
}
```

另一个：

```java
@Service
public class OrderService {

    @Transactional
    public void saveOrder() {

    }
}
```

调用：

```
UserService
    |
    v
OrderService代理
    |
    v
@Transactional方法
```

事务正常。

---

### 方法2：注入自己（不推荐，但常见）

```java
@Service
public class UserService {

    @Autowired
    private UserService self;


    public void test() {
        self.saveUser();
    }


    @Transactional
    public void saveUser() {

    }
}
```

此时：

```
self
 |
 v
代理对象
 |
 v
saveUser()
```

事务生效。

---

### 方法3：通过 AopContext 获取代理

需要开启：

```java
@EnableAspectJAutoProxy(exposeProxy = true)
```

然后：

```java
((UserService) AopContext.currentProxy())
    .saveUser();
```

可以绕回代理。

---

## 5. 注意几个容易踩坑的点

### 私有方法事务无效

```java
@Transactional
private void save(){

}
```

无效。

因为 Spring 默认代理无法拦截 private 方法。

---

### final 方法事务可能无效

```java
@Transactional
public final void save(){

}
```

因为 CGLIB 无法重写 final 方法。

---

### 同类调用不是所有情况都失效？

准确说：

> **只有需要经过 Spring AOP 代理增强的功能会失效。**

比如：

* `@Transactional` ❌
* `@Cacheable` ❌
* `@Async` ❌

这些依赖代理。

普通 Java 方法调用：

```java
this.xxx();
```

当然没有问题。

---

一句话记忆：

> **Spring 的注解功能靠代理实现；同一个类里 `this.xxx()` 是直接调用原对象，绕过代理，所以事务、缓存、异步等 AOP 功能会失效。**
