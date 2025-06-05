# 目录
1. [查询数据（SELECT）](#一查询数据select)
   - [基本查询](#基本查询)
   - [查询所有字段](#查询所有字段)
   - [使用条件（WHERE）](#使用条件where)
   - [排序（ORDER BY）](#排序order-by)
   - [去重（DISTINCT）](#去重distinct)
   - [分页（LIMIT / OFFSET）](#分页limit--offset)
2. [插入数据（INSERT）](#二插入数据insert)
   - [插入单行](#插入单行)
   - [插入多行](#插入多行)
3. [更新数据（UPDATE）](#三更新数据update)
4. [删除数据（DELETE）](#四删除数据delete)
5. [聚合函数（GROUP BY + HAVING）](#五聚合函数group-by--having)
6. [连接查询（JOIN）](#六连接查询join)
7. [子查询（Subquery）](#七子查询subquery)
8. [CASE 表达式（类似 IF）](#八case-表达式类似-if)
9. [创建和管理表结构（DDL）](#九创建和管理表结构ddl)
10. [常用运算符速查](#十常用运算符速查)
11. [常见组合例子](#常见组合例子)

# SQL常用语法
## 一、查询数据（SELECT）
### 基本查询
```sql
SELECT column1, column2
FROM table_name;
```
### 查询所有字段
```sql
SELECT * FROM table_name;
```
### 使用条件（WHERE）
```sql
SELECT name, age
FROM users
WHERE age >= 18 AND gender = 'F';
```
### 排序（ORDER BY）
```sql
SELECT name, age
FROM users
ORDER BY age DESC, name ASC;
```
### 去重（DISTINCT）
```sql
SELECT DISTINCT city FROM users;
```
### 分页（LIMIT / OFFSET）
```sql
-- MySQL / PostgreSQL
SELECT * FROM users LIMIT 10 OFFSET 20;
```
## 二、插入数据（INSERT）
### 插入单行
```sql
INSERT INTO users (name, age)
VALUES ('Alice', 25);
```
### 插入多行
```sql
INSERT INTO users (name, age)
VALUES 
  ('Bob', 30),
  ('Carol', 22);
```
## 三、更新数据（UPDATE）
```sql
UPDATE users
SET age = 26
WHERE name = 'Alice';
```
## 四、删除数据（DELETE）
```sql
DELETE FROM users
WHERE age < 18;
```
## 五、聚合函数（GROUP BY + HAVING）
```sql
SELECT department, COUNT(*) AS total
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```
常见聚合函数：
| 函数        | 作用  |
| --------- | --- |
| `COUNT()` | 计数  |
| `SUM()`   | 求和  |
| `AVG()`   | 平均值 |
| `MAX()`   | 最大值 |
| `MIN()`   | 最小值 |
## 六、连接查询（JOIN）
```sql
SELECT u.name, o.order_id
FROM users u
JOIN orders o ON u.id = o.user_id;
```
## 七、子查询（Subquery）
```sql
SELECT name
FROM users
WHERE id IN (
  SELECT user_id FROM orders WHERE amount > 100
);
```
## 八、CASE 表达式（类似 IF）
```sql
SELECT name,
       CASE 
         WHEN age < 18 THEN 'minor'
         WHEN age < 65 THEN 'adult'
         ELSE 'senior'
       END AS age_group
FROM users;
```
## 九、创建和管理表结构（DDL）
```sql
-- 创建表
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  age INT
);

-- 修改表
ALTER TABLE users ADD email VARCHAR(255);

-- 删除表
DROP TABLE users;
```
## 十、常用运算符速查
| 类别   | 运算符/关键字                  |
| ---- | ------------------------ |
| 比较   | =, <>, >, <, >=, <=      |
| 范围   | `BETWEEN x AND y`        |
| 集合   | `IN (..)`                |
| 模糊匹配 | `LIKE '%abc%'`           |
| 空值检查 | `IS NULL`, `IS NOT NULL` |
| 逻辑运算 | `AND`, `OR`, `NOT`       |

## 常见组合例子
```sql
SELECT department, AVG(salary)
FROM employees
WHERE hire_date >= '2022-01-01'
GROUP BY department
ORDER BY AVG(salary) DESC
LIMIT 5;
```