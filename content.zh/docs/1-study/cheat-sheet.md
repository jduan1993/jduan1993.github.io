---
title: Coding Cheat Sheet
---

## Java + Python 高频方法大全

> **版本说明：** Python 3.x；Java 8+。Java 9+ 和 Java 11+ 才支持的方法会特别注明。

# PART 1 — PYTHON

## 1. Built-in Functions（内置函数）

这些函数不需要额外导入模块。

| 方法              | 用法示例                | 作用                   |
| --------------- | ------------------- | -------------------- |
| ⭐ `len()`       | `len(arr)`          | 获取长度                 |
| ⭐ `range()`     | `range(n)`          | 生成整数序列               |
| ⭐ `enumerate()` | `enumerate(arr)`    | 同时获取索引和值             |
| ⭐ `zip()`       | `zip(a, b)`         | 同时遍历多个序列             |
| ⭐ `sorted()`    | `sorted(arr)`       | 返回排序后的新列表            |
| ⭐ `reversed()`  | `reversed(arr)`     | 反向迭代                 |
| ⭐ `sum()`       | `sum(arr)`          | 求和                   |
| ⭐ `min()`       | `min(arr)`          | 最小值                  |
| ⭐ `max()`       | `max(arr)`          | 最大值                  |
| ⭐ `abs()`       | `abs(x)`            | 绝对值                  |
| `round()`       | `round(x, 2)`       | 四舍五入到指定小数位           |
| `pow()`         | `pow(a, b)`         | 计算幂                  |
| `divmod()`      | `divmod(a, b)`      | 返回 `(a // b, a % b)` |
| `all()`         | `all(arr)`          | 所有元素都为真时返回 `True`    |
| `any()`         | `any(arr)`          | 至少一个元素为真时返回 `True`   |
| `map()`         | `map(int, arr)`     | 对每个元素应用函数            |
| `filter()`      | `filter(func, arr)` | 筛选满足条件的元素            |
| ⭐ `int()`       | `int("123")`        | 转整数                  |
| ⭐ `str()`       | `str(123)`          | 转字符串                 |
| `ord()`         | `ord('a')`          | 字符转 Unicode 整数       |
| `chr()`         | `chr(97)`           | Unicode 整数转字符        |

### 高频写法

```python
# 遍历索引和值
for i, x in enumerate(arr):
    pass

# 同时遍历两个数组
for a, b in zip(nums1, nums2):
    pass

# 排序
arr = sorted(arr)
arr = sorted(arr, reverse=True)

# 自定义排序
arr = sorted(words, key=len)
arr = sorted(intervals, key=lambda x: x[0])

# 求和、最大值、最小值
total = sum(arr)
maximum = max(arr)
minimum = min(arr)

# 初始化二维数组：每行独立
matrix = [[0] * n for _ in range(m)]

# 统计真假值
count = sum(x > 0 for x in arr)
```

**注意：** `sorted(arr)` 返回新列表，而 `arr.sort()` 直接修改原列表。

---

## 2. List（列表 / 动态数组）

Python 的 `list` 是 Coding Test 中最常用的数据结构之一。

| 方法           | 用法示例                | 作用              |
| ------------ | ------------------- | --------------- |
| ⭐ `append()` | `arr.append(x)`     | 末尾添加一个元素        |
| ⭐ `extend()` | `arr.extend(other)` | 在末尾添加多个元素       |
| `insert()`   | `arr.insert(i, x)`  | 在指定位置插入         |
| ⭐ `pop()`    | `arr.pop()`         | 删除并返回最后一个元素     |
| `pop(i)`     | `arr.pop(i)`        | 删除并返回索引 `i` 的元素 |
| `remove()`   | `arr.remove(x)`     | 删除第一个值为 `x` 的元素 |
| `clear()`    | `arr.clear()`       | 清空列表            |
| ⭐ `index()`  | `arr.index(x)`      | 查找元素第一次出现的位置    |
| `count()`    | `arr.count(x)`      | 统计指定元素出现次数      |
| ⭐ `sort()`   | `arr.sort()`        | 原地排序            |
| `reverse()`  | `arr.reverse()`     | 原地反转            |
| `copy()`     | `arr.copy()`        | 浅拷贝             |
| `len()`      | `len(arr)`          | 获取长度            |

### 高频操作

```python
# 访问元素
x = arr[0]
last = arr[-1]

# 切片
arr[:k]       # 前 k 个元素
arr[k:]       # 从 k 到末尾
arr[l:r]      # 索引 l 到 r-1
arr[::-1]     # 反转后的新列表
arr[::2]      # 每隔一个元素取值

# 合并
new_arr = arr1 + arr2

# 判断是否存在
if x in arr:
    pass

# 排序
arr.sort()
arr.sort(reverse=True)

# 自定义排序
arr.sort(key=lambda x: x[1])

# 删除最后一个元素
arr.pop()

# 列表推导式
squares = [x * x for x in arr]
evens = [x for x in arr if x % 2 == 0]

# 二维数组
matrix = [[0] * n for _ in range(m)]
```

**常见陷阱：**

```python
# 错误：多个行共享同一个内部列表
matrix = [[0] * n] * m

# 正确：每行是独立列表
matrix = [[0] * n for _ in range(m)]
```

---

## 3. String（字符串）

字符串处理在 Coding Test 中出现频率非常高。

| 方法             | 用法示例                  | 作用              |
| -------------- | --------------------- | --------------- |
| ⭐ `len()`      | `len(s)`              | 字符串长度           |
| ⭐ `split()`    | `s.split()`           | 按空白字符拆分         |
| `split(sep)`   | `s.split(",")`        | 按指定分隔符拆分        |
| ⭐ `join()`     | `"".join(chars)`      | 用指定字符串连接元素      |
| ⭐ `strip()`    | `s.strip()`           | 去除两端空白字符        |
| `lstrip()`     | `s.lstrip()`          | 去除左侧空白字符        |
| `rstrip()`     | `s.rstrip()`          | 去除右侧空白字符        |
| ⭐ `lower()`    | `s.lower()`           | 转小写             |
| `upper()`      | `s.upper()`           | 转大写             |
| ⭐ `replace()`  | `s.replace("a", "b")` | 替换子字符串          |
| ⭐ `find()`     | `s.find("abc")`       | 查找位置，不存在返回 `-1` |
| `rfind()`      | `s.rfind("a")`        | 从右侧查找           |
| `index()`      | `s.index("a")`        | 查找位置，不存在则抛异常    |
| ⭐ `count()`    | `s.count("a")`        | 统计出现次数          |
| `startswith()` | `s.startswith("ab")`  | 检查开头            |
| `endswith()`   | `s.endswith(".com")`  | 检查结尾            |
| `isdigit()`    | `s.isdigit()`         | 检查是否由数字字符组成     |
| `isalpha()`    | `s.isalpha()`         | 检查是否由字母组成       |
| `isalnum()`    | `s.isalnum()`         | 检查是否由字母或数字组成    |
| `isspace()`    | `s.isspace()`         | 检查是否全为空白字符      |

### 高频写法

```python
# 字符串转字符列表
chars = list(s)

# 字符列表转字符串
s = "".join(chars)

# 拆分输入
words = s.split()

# 去除两端空白
s = s.strip()

# 反转字符串
rev = s[::-1]

# 检查回文
is_palindrome = s == s[::-1]

# 字母大小写统一
s = s.lower()

# 统计某个字符
count = s.count("a")

# 字符串中是否包含子串
if "abc" in s:
    pass

# 字符串转整数
num = int("123")

# 整数转字符串
s = str(123)
```

**重要：Python 字符串不可变。** 如果要逐步构造字符串，常用列表收集字符，最后使用 `''.join(chars)`。

---

## 4. Dictionary（字典 / HashMap）

字典是 Python 实现哈希表的主要方式，用于频率统计、查找、映射等问题。

| 方法             | 用法示例                  | 作用            |
| -------------- | --------------------- | ------------- |
| ⭐ `get()`      | `d.get(key, 0)`       | 获取值，不存在时返回默认值 |
| ⭐ `keys()`     | `d.keys()`            | 获取所有键         |
| `values()`     | `d.values()`          | 获取所有值         |
| ⭐ `items()`    | `d.items()`           | 获取键值对         |
| ⭐ `update()`   | `d.update(other)`     | 更新字典          |
| `setdefault()` | `d.setdefault(k, [])` | 不存在时设置默认值     |
| `pop()`        | `d.pop(key, None)`    | 删除并返回键对应的值    |
| `clear()`      | `d.clear()`           | 清空字典          |

### 高频写法

```python
# 初始化
d = {}

# 添加 / 更新
d[key] = value

# 安全读取
value = d.get(key, 0)

# 判断键是否存在
if key in d:
    pass

# 遍历键
for key in d:
    pass

# 遍历键值对
for key, value in d.items():
    pass

# 频率统计
freq[x] = freq.get(x, 0) + 1

# 常见的分组写法
d.setdefault(key, []).append(value)
```

### `Counter` 和 `defaultdict`

`collections` 中有两个非常值得掌握的工具。

```python
from collections import Counter, defaultdict

# 统计元素频率
freq = Counter(nums)
freq[x]       # x 出现的次数
freq.most_common(2)  # 出现次数最多的 2 个元素

# 默认值为 0
freq = defaultdict(int)
freq[x] += 1

# 默认值为空列表
groups = defaultdict(list)
groups[key].append(value)
```

**Coding Test 高频题型：** Two Sum、频率统计、Anagrams、分组、重复元素判断。

---

## 5. Set（集合 / HashSet）

集合适合去重和平均 O(1) 时间复杂度的成员查询。

| 方法                 | 用法示例                | 作用            |
| ------------------ | ------------------- | ------------- |
| ⭐ `add()`          | `s.add(x)`          | 添加元素          |
| ⭐ `remove()`       | `s.remove(x)`       | 删除元素，不存在则报错   |
| `discard()`        | `s.discard(x)`      | 删除元素，不存在也不会报错 |
| `pop()`            | `s.pop()`           | 删除并返回一个元素     |
| `clear()`          | `s.clear()`         | 清空集合          |
| ⭐ `union()`        | `a.union(b)`        | 并集            |
| ⭐ `intersection()` | `a.intersection(b)` | 交集            |
| ⭐ `difference()`   | `a.difference(b)`   | 差集            |
| `issubset()`       | `a.issubset(b)`     | 判断是否为子集       |
| `issuperset()`     | `a.issuperset(b)`   | 判断是否为超集       |

### 高频运算符

```python
# 去重
unique = set(nums)

# 是否存在
if x in s:
    pass

# 添加和删除
s.add(x)
s.discard(x)

# 并集
a | b

# 交集
a & b

# 差集
a - b

# 对称差集
a ^ b

# 子集判断
a <= b

# 超集判断
a >= b
```

注意：`set` 是无序集合，不能依靠索引访问元素。

---

## 6. Stack、Queue、Deque（栈、队列、双端队列）

### Stack：栈

Python 通常直接使用 `list` 实现栈。

```python
stack = []

stack.append(x)  # push
stack.pop()      # pop
stack[-1]        # peek：查看栈顶
```

### Queue：队列

普通列表不适合频繁从头部删除元素。BFS 等场景推荐使用 `deque`。

```python
from collections import deque

q = deque()

q.append(x)       # 队尾加入
q.popleft()       # 队首取出
q[0]              # 查看队首
```

### Deque：双端队列

| 方法                     | 作用     |
| ---------------------- | ------ |
| `append(x)`            | 右端添加   |
| `appendleft(x)`        | 左端添加   |
| `pop()`                | 删除右端元素 |
| `popleft()`            | 删除左端元素 |
| `extend(iterable)`     | 右端批量添加 |
| `extendleft(iterable)` | 左端批量添加 |
| `rotate(k)`            | 旋转队列   |

**典型应用：** BFS、单调队列、滑动窗口、栈模拟、括号匹配。

---

## 7. Heap / Priority Queue（堆 / 优先队列）

Python 使用 `heapq`，默认是**最小堆**。

```python
import heapq

heap = []

heapq.heappush(heap, x)  # 添加元素
x = heapq.heappop(heap)   # 弹出最小元素
x = heap[0]               # 查看最小元素
heapq.heapify(arr)        # 将列表原地转换为堆
```

### 常用方法

| 方法                  | 作用           |
| ------------------- | ------------ |
| `heappush()`        | 入堆           |
| `heappop()`         | 弹出最小元素       |
| `heapify()`         | 列表原地建堆       |
| `heappushpop()`     | 先加入再弹出最小值    |
| `heapreplace()`     | 先弹出最小值再加入新元素 |
| `nlargest(k, arr)`  | 获取最大的 k 个元素  |
| `nsmallest(k, arr)` | 获取最小的 k 个元素  |

### 高频写法

```python
# 最小堆
heap = []
heapq.heappush(heap, x)
smallest = heapq.heappop(heap)

# 前 k 大元素：直接获得结果
largest_k = heapq.nlargest(k, nums)

# 使用负数模拟最大堆
heap = []
heapq.heappush(heap, -x)
largest = -heapq.heappop(heap)
```

**常见题型：** Top K、合并有序列表、调度问题、动态维护最小值/最大值。

---

## 8. Binary Search（二分查找）

Python 的 `bisect` 模块适合在**有序列表**中查找插入位置。

```python
from bisect import bisect_left, bisect_right, insort

i = bisect_left(arr, x)
j = bisect_right(arr, x)
insort(arr, x)
```

| 方法                     | 作用             |
| ---------------------- | -------------- |
| `bisect_left(arr, x)`  | 第一个 `>= x` 的位置 |
| `bisect_right(arr, x)` | 第一个 `> x` 的位置  |
| `insort(arr, x)`       | 按顺序插入元素        |

### 判断元素是否存在

```python
i = bisect_left(arr, target)

if i < len(arr) and arr[i] == target:
    print("Found")
```

### 手写二分查找

```python
left, right = 0, len(arr) - 1

while left <= right:
    mid = left + (right - left) // 2

    if arr[mid] == target:
        break
    elif arr[mid] < target:
        left = mid + 1
    else:
        right = mid - 1
```

---

## 9. Math（数学函数）

```python
import math
```

| 方法 / 常量             | 作用                |
| ------------------- | ----------------- |
| `math.gcd(a, b)`    | 最大公约数             |
| `math.lcm(a, b)`    | 最小公倍数，Python 3.9+ |
| `math.sqrt(x)`      | 平方根               |
| `math.isqrt(x)`     | 不超过平方根的最大整数       |
| `math.ceil(x)`      | 向上取整              |
| `math.floor(x)`     | 向下取整              |
| `math.factorial(n)` | 阶乘                |
| `math.inf`          | 正无穷               |
| `math.isqrt(n)`     | 整数平方根             |

### 高频写法

```python
# 最大公约数
g = math.gcd(a, b)

# 最小公倍数
lcm = a * b // math.gcd(a, b) if a and b else 0

# 无穷大，用于初始化最小值
best = float("inf")

# 初始化最大值
best = float("-inf")

# Python 的整数可以自动扩展精度
big = 10**100
```

---

## 10. Other Useful Modules（其他常用模块）

### `itertools`

```python
from itertools import accumulate, product, combinations, permutations

# 前缀和
prefix = list(accumulate(nums))

# 笛卡尔积
for a, b in product(arr1, arr2):
    pass

# 组合
for pair in combinations(nums, 2):
    pass

# 排列
for p in permutations(nums, 2):
    pass
```

高频程度从上到下依次降低。前缀和最值得掌握；组合和排列在回溯及枚举题中比较有用。

### `collections`

```python
from collections import Counter, defaultdict, deque
```

最值得记忆的是 `Counter`、`defaultdict` 和 `deque`，前面已列出用法。

### `sys`：输入处理

```python
import sys

# 读取整行
line = sys.stdin.readline()

# 读取全部输入，按空白分割
data = sys.stdin.read().split()

# 递归较深时，某些题目可能需要调整上限
sys.setrecursionlimit(10**6)
```

### 高频输入写法

```python
# 一行整数
nums = list(map(int, input().split()))

# 多行输入
n = int(input())

for _ in range(n):
    x = int(input())
```

---

# PART 2 — JAVA

## 1. String（字符串）

Java 字符串是不可变对象。修改字符串内容时，通常会产生新的字符串。

| 方法                   | 用法示例                       | 作用                 |
| -------------------- | -------------------------- | ------------------ |
| ⭐ `length()`         | `s.length()`               | 获取长度               |
| ⭐ `charAt()`         | `s.charAt(i)`              | 获取指定位置的字符          |
| ⭐ `substring()`      | `s.substring(l, r)`        | 截取 `[l, r)`        |
| ⭐ `equals()`         | `s.equals(t)`              | 比较字符串内容            |
| `equalsIgnoreCase()` | `s.equalsIgnoreCase(t)`    | 忽略大小写比较            |
| ⭐ `compareTo()`      | `s.compareTo(t)`           | 字典序比较              |
| ⭐ `contains()`       | `s.contains("abc")`        | 判断是否包含子串           |
| ⭐ `indexOf()`        | `s.indexOf("a")`           | 查找首次出现的位置          |
| `lastIndexOf()`      | `s.lastIndexOf("a")`       | 查找最后一次出现的位置        |
| `startsWith()`       | `s.startsWith("ab")`       | 判断开头               |
| `endsWith()`         | `s.endsWith(".com")`       | 判断结尾               |
| ⭐ `split()`          | `s.split(",")`             | 按分隔符拆分             |
| ⭐ `replace()`        | `s.replace("a", "b")`      | 替换字符或子字符串          |
| `replaceAll()`       | `s.replaceAll("\\s+", "")` | 使用正则表达式替换          |
| `trim()`             | `s.trim()`                 | 去除两端 ASCII 控制字符及空格 |
| `toLowerCase()`      | `s.toLowerCase()`          | 转小写                |
| `toUpperCase()`      | `s.toUpperCase()`          | 转大写                |
| `isEmpty()`          | `s.isEmpty()`              | 判断长度是否为 0          |
| `toCharArray()`      | `s.toCharArray()`          | 转字符数组              |

### 高频写法

```java
// 长度
int n = s.length();

// 获取字符
char c = s.charAt(i);

// 截取 [l, r)
String sub = s.substring(l, r);

// 比较字符串内容
if (s.equals(t)) {
    // ...
}

// 判断是否包含
if (s.contains("abc")) {
    // ...
}

// 查找
int idx = s.indexOf("abc");

// 拆分
String[] parts = s.split(",");

// 去除两端空白
String cleaned = s.trim();

// 转字符数组
char[] chars = s.toCharArray();

// 字符串转数字
int num = Integer.parseInt("123");
long value = Long.parseLong("123456789");
```

**最重要的 Java 字符串知识：**

```java
// 比较字符串内容：使用 equals
s.equals(t);

// 不要用 == 比较字符串内容
s == t;  // 比较的是对象引用是否相同
```

另外，Java 的 `substring(l, r)` 与 Python 的 `s[l:r]` 一样，都不包括右边界 `r`。

---

## 2. StringBuilder（字符串构造器）

当需要频繁拼接或修改字符串时，推荐使用 `StringBuilder`。

| 方法               | 用法示例                 | 作用          |
| ---------------- | -------------------- | ----------- |
| ⭐ `append()`     | `sb.append(x)`       | 追加内容        |
| `insert()`       | `sb.insert(i, x)`    | 指定位置插入      |
| `delete()`       | `sb.delete(l, r)`    | 删除 `[l, r)` |
| `deleteCharAt()` | `sb.deleteCharAt(i)` | 删除指定位置字符    |
| `reverse()`      | `sb.reverse()`       | 反转          |
| `setCharAt()`    | `sb.setCharAt(i, c)` | 修改指定字符      |
| `charAt()`       | `sb.charAt(i)`       | 获取字符        |
| `length()`       | `sb.length()`        | 获取长度        |
| ⭐ `toString()`   | `sb.toString()`      | 转为字符串       |

### 高频写法

```java
StringBuilder sb = new StringBuilder();

sb.append("hello");
sb.append(' ');
sb.append("world");

String result = sb.toString();

// 反转字符串
String reversed = new StringBuilder(s).reverse().toString();

// 修改字符
sb.setCharAt(0, 'H');
```

---

## 3. Arrays（数组工具类）

Java 的 `Arrays` 类是 Coding Test 的重要工具类，需要重点记忆。

先导入：

```java
import java.util.Arrays;
```

| 方法                        | 用法示例                            | 作用          |
| ------------------------- | ------------------------------- | ----------- |
| ⭐ `Arrays.sort()`         | `Arrays.sort(arr)`              | 排序          |
| ⭐ `Arrays.binarySearch()` | `Arrays.binarySearch(arr, x)`   | 二分查找        |
| ⭐ `Arrays.fill()`         | `Arrays.fill(arr, 0)`           | 填充数组        |
| `Arrays.copyOf()`         | `Arrays.copyOf(arr, n)`         | 复制并指定长度     |
| `Arrays.copyOfRange()`    | `Arrays.copyOfRange(arr, l, r)` | 复制 `[l, r)` |
| `Arrays.equals()`         | `Arrays.equals(a, b)`           | 比较数组内容      |
| `Arrays.toString()`       | `Arrays.toString(arr)`          | 将数组转为可读字符串  |
| `Arrays.deepToString()`   | `Arrays.deepToString(matrix)`   | 输出多维数组      |
| `Arrays.asList()`         | `Arrays.asList("a", "b")`       | 创建基于数组的列表视图 |

### 高频写法

```java
// 排序
Arrays.sort(arr);

// 二分查找：数组必须有序
int idx = Arrays.binarySearch(arr, target);

// 初始化数组
int[] arr = new int[n];
Arrays.fill(arr, -1);

// 复制数组
int[] copy = Arrays.copyOf(arr, arr.length);

// 截取 [l, r)
int[] sub = Arrays.copyOfRange(arr, l, r);

// 比较数组内容
boolean same = Arrays.equals(a, b);

// 打印数组
System.out.println(Arrays.toString(arr));

// 打印二维数组
System.out.println(Arrays.deepToString(matrix));
```

**注意：** `Arrays.binarySearch()` 找不到元素时会返回负数，其值还编码了插入位置；使用前必须确保数组已经排序。

---

## 4. ArrayList（动态数组）

先导入：

```java
import java.util.ArrayList;
import java.util.List;
```

| 方法               | 用法示例                              | 作用              |
| ---------------- | --------------------------------- | --------------- |
| ⭐ `add()`        | `list.add(x)`                     | 添加元素            |
| `add(i, x)`      | `list.add(i, x)`                  | 在指定位置插入         |
| ⭐ `get()`        | `list.get(i)`                     | 获取元素            |
| ⭐ `set()`        | `list.set(i, x)`                  | 修改指定位置          |
| ⭐ `remove()`     | `list.remove(i)`                  | 按索引删除           |
| `remove(Object)` | `list.remove(Integer.valueOf(x))` | 按值删除一个匹配元素      |
| ⭐ `size()`       | `list.size()`                     | 获取长度            |
| ⭐ `contains()`   | `list.contains(x)`                | 判断是否包含          |
| `indexOf()`      | `list.indexOf(x)`                 | 首次出现的位置         |
| `lastIndexOf()`  | `list.lastIndexOf(x)`             | 最后出现的位置         |
| `isEmpty()`      | `list.isEmpty()`                  | 判断是否为空          |
| `clear()`        | `list.clear()`                    | 清空列表            |
| `subList()`      | `list.subList(l, r)`              | 获取 `[l, r)` 的视图 |

### 高频写法

```java
List<Integer> list = new ArrayList<>();

// 添加
list.add(10);

// 获取
int x = list.get(0);

// 修改
list.set(0, 20);

// 长度
int n = list.size();

// 判断
if (list.contains(10)) {
    // ...
}

// 遍历
for (int x : list) {
    // ...
}

// 索引遍历
for (int i = 0; i < list.size(); i++) {
    int x = list.get(i);
}
```

### 一个非常容易踩的坑

```java
List<Integer> list = new ArrayList<>();

list.add(10);
list.add(20);

// 按索引删除第 0 个元素
list.remove(0);

// 按值删除整数 20
list.remove(Integer.valueOf(20));
```

因为 `remove(0)` 会被解释为按索引删除，而不是删除值为 `0` 的元素。

---

## 5. HashMap（哈希表）

常用于键值映射、频率统计、快速查找。

导入：

```java
import java.util.HashMap;
import java.util.Map;
```

| 方法                  | 用法示例                                               | 作用            |
| ------------------- | -------------------------------------------------- | ------------- |
| ⭐ `put()`           | `map.put(k, v)`                                    | 添加或更新键值       |
| ⭐ `get()`           | `map.get(k)`                                       | 获取值           |
| ⭐ `getOrDefault()`  | `map.getOrDefault(k, 0)`                           | 获取值，不存在时返回默认值 |
| ⭐ `containsKey()`   | `map.containsKey(k)`                               | 判断键是否存在       |
| `containsValue()`   | `map.containsValue(v)`                             | 判断值是否存在       |
| `putIfAbsent()`     | `map.putIfAbsent(k, v)`                            | 键不存在时才添加      |
| ⭐ `remove()`        | `map.remove(k)`                                    | 删除键           |
| `replace()`         | `map.replace(k, v)`                                | 替换已存在的值       |
| `merge()`           | `map.merge(k, 1, Integer::sum)`                    | 合并或更新值        |
| `computeIfAbsent()` | `map.computeIfAbsent(k, key -> new ArrayList<>())` | 键不存在时计算并保存值   |
| ⭐ `keySet()`        | `map.keySet()`                                     | 获取所有键         |
| ⭐ `values()`        | `map.values()`                                     | 获取所有值         |
| ⭐ `entrySet()`      | `map.entrySet()`                                   | 获取所有键值对       |
| `size()`            | `map.size()`                                       | 获取键值对数量       |
| `isEmpty()`         | `map.isEmpty()`                                    | 判断是否为空        |
| `clear()`           | `map.clear()`                                      | 清空            |

### 高频写法

```java
Map<Integer, Integer> map = new HashMap<>();

// 添加 / 更新
map.put(key, value);

// 安全获取
int value = map.getOrDefault(key, 0);

// 统计频率
map.put(x, map.getOrDefault(x, 0) + 1);

// 判断是否存在
if (map.containsKey(key)) {
    // ...
}

// 遍历键
for (int key : map.keySet()) {
    // ...
}

// 遍历键值对
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    int key = entry.getKey();
    int value = entry.getValue();
}

// Java 8+：频率统计
map.merge(x, 1, Integer::sum);
```

### HashMap 高频题型

Two Sum、Frequency Count、Grouping、Duplicate Detection、Prefix Sum + HashMap。

---

## 6. HashSet（集合）

适用于去重、成员判断、交集和差集。

```java
import java.util.HashSet;
import java.util.Set;
```

| 方法             | 用法示例                   | 作用         |
| -------------- | ---------------------- | ---------- |
| ⭐ `add()`      | `set.add(x)`           | 添加元素       |
| ⭐ `contains()` | `set.contains(x)`      | 判断元素是否存在   |
| ⭐ `remove()`   | `set.remove(x)`        | 删除元素       |
| `size()`       | `set.size()`           | 获取元素数量     |
| `isEmpty()`    | `set.isEmpty()`        | 判断是否为空     |
| `clear()`      | `set.clear()`          | 清空         |
| `addAll()`     | `set.addAll(other)`    | 批量添加，即并集效果 |
| `retainAll()`  | `set.retainAll(other)` | 只保留共有元素    |
| `removeAll()`  | `set.removeAll(other)` | 删除其他集合中的元素 |

### 高频写法

```java
Set<Integer> set = new HashSet<>();

// 添加
set.add(x);

// 查询
if (set.contains(x)) {
    // ...
}

// 去重
Set<Integer> unique = new HashSet<>(list);

// 并集
Set<Integer> union = new HashSet<>(a);
union.addAll(b);

// 交集
Set<Integer> intersection = new HashSet<>(a);
intersection.retainAll(b);

// 差集
Set<Integer> difference = new HashSet<>(a);
difference.removeAll(b);
```

---

## 7. Queue、Deque、Stack（队列、双端队列、栈）

Java 中推荐使用 `ArrayDeque` 实现普通栈和队列。

```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Queue;
```

### Queue：队列

```java
Queue<Integer> q = new ArrayDeque<>();

q.offer(x);  // 入队
q.poll();    // 出队，空队列时返回 null
q.peek();    // 查看队首，空队列时返回 null
q.isEmpty(); // 判断是否为空
q.size();    // 获取长度
```

| 方法          | 作用      |
| ----------- | ------- |
| `offer(x)`  | 添加到队尾   |
| `poll()`    | 删除并返回队首 |
| `peek()`    | 查看队首    |
| `isEmpty()` | 判断是否为空  |
| `size()`    | 获取元素数量  |

### Deque：双端队列

```java
Deque<Integer> deque = new ArrayDeque<>();

deque.offerFirst(x); // 左端加入
deque.offerLast(x);  // 右端加入
deque.pollFirst();   // 左端取出
deque.pollLast();    // 右端取出
deque.peekFirst();   // 查看左端
deque.peekLast();    // 查看右端
```

### Stack：使用 Deque 实现栈

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(x); // 入栈
stack.pop();   // 出栈
stack.peek();  // 查看栈顶
stack.isEmpty();
stack.size();
```

**建议：** 在 Java Coding Test 中，优先使用 `Deque` + `ArrayDeque`，而不是旧的 `Stack` 类。

**典型题型：** BFS、DFS、括号匹配、单调栈、滑动窗口。

---

## 8. PriorityQueue（优先队列 / 堆）

Java 的 `PriorityQueue` 默认是**最小堆**。

```java
import java.util.PriorityQueue;
```

| 方法           | 作用         |
| ------------ | ---------- |
| ⭐ `offer(x)` | 加入元素       |
| `add(x)`     | 加入元素       |
| ⭐ `poll()`   | 弹出优先级最高的元素 |
| ⭐ `peek()`   | 查看优先级最高的元素 |
| `size()`     | 获取数量       |
| `isEmpty()`  | 判断是否为空     |
| `clear()`    | 清空         |

### 高频写法

```java
// 最小堆
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

minHeap.offer(5);
minHeap.offer(2);
minHeap.offer(8);

int smallest = minHeap.poll(); // 2
```

### 最大堆

```java
PriorityQueue<Integer> maxHeap =
    new PriorityQueue<>((a, b) -> Integer.compare(b, a));
```

### 自定义优先级

```java
// 按二维数组第 2 列升序排列
PriorityQueue<int[]> pq =
    new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
```

**注意：** 不建议简单写 `(a, b) -> b - a` 来实现降序比较，因为整数相减可能溢出。使用 `Integer.compare(b, a)` 更稳妥。

---

## 9. Collections（集合工具类）

`Collections` 和 `Arrays` 不同：`Arrays` 主要处理数组，`Collections` 提供集合操作工具。

```java
import java.util.Collections;
```

| 方法                           | 用法示例                                | 作用       |
| ---------------------------- | ----------------------------------- | -------- |
| ⭐ `Collections.sort()`       | `Collections.sort(list)`            | 排序       |
| `Collections.reverse()`      | `Collections.reverse(list)`         | 反转列表     |
| `Collections.swap()`         | `Collections.swap(list, i, j)`      | 交换两个元素   |
| `Collections.max()`          | `Collections.max(list)`             | 最大值      |
| `Collections.min()`          | `Collections.min(list)`             | 最小值      |
| `Collections.frequency()`    | `Collections.frequency(list, x)`    | 统计出现次数   |
| `Collections.binarySearch()` | `Collections.binarySearch(list, x)` | 二分查找     |
| `Collections.fill()`         | `Collections.fill(list, x)`         | 用指定值填充列表 |
| `Collections.reverseOrder()` | `Collections.reverseOrder()`        | 返回逆序比较器  |

### 高频写法

```java
// 排序
Collections.sort(list);

// 反转
Collections.reverse(list);

// 最大值、最小值
int max = Collections.max(list);
int min = Collections.min(list);

// 自定义排序：按绝对值升序
list.sort((a, b) ->
    Integer.compare(Math.abs(a), Math.abs(b))
);

// 降序排序
list.sort(Collections.reverseOrder());
```

### 自定义排序：二维数组

```java
// 按第 0 列升序
Arrays.sort(intervals,
    (a, b) -> Integer.compare(a[0], b[0])
);

// 先按第 0 列升序，再按第 1 列升序
Arrays.sort(intervals, (a, b) -> {
    int cmp = Integer.compare(a[0], b[0]);

    if (cmp != 0) {
        return cmp;
    }

    return Integer.compare(a[1], b[1]);
});
```

**注意：** 基本类型数组，例如 `int[]`，不能直接传入需要 `Comparator` 的 `Arrays.sort()` 重载。自定义比较器通常用于 `Integer[]`、`String[]` 或对象数组。

---

## 10. Math（数学函数）

Java 数学函数主要来自 `Math` 类。

| 方法 / 常量             | 用法示例                | 作用               |
| ------------------- | ------------------- | ---------------- |
| ⭐ `Math.max()`      | `Math.max(a, b)`    | 最大值              |
| ⭐ `Math.min()`      | `Math.min(a, b)`    | 最小值              |
| ⭐ `Math.abs()`      | `Math.abs(x)`       | 绝对值              |
| `Math.pow()`        | `Math.pow(a, b)`    | 幂运算，结果为 `double` |
| `Math.sqrt()`       | `Math.sqrt(x)`      | 平方根              |
| `Math.ceil()`       | `Math.ceil(x)`      | 向上取整，返回 `double` |
| `Math.floor()`      | `Math.floor(x)`     | 向下取整，返回 `double` |
| `Math.round()`      | `Math.round(x)`     | 四舍五入到整数尺度        |
| `Math.random()`     | `Math.random()`     | 生成 `[0, 1)` 的随机数 |
| `Integer.MAX_VALUE` | `Integer.MAX_VALUE` | `int` 最大值        |
| `Integer.MIN_VALUE` | `Integer.MIN_VALUE` | `int` 最小值        |
| `Long.MAX_VALUE`    | `Long.MAX_VALUE`    | `long` 最大值       |

### 高频写法

```java
int max = Math.max(a, b);
int min = Math.min(a, b);
int abs = Math.abs(x);

double sqrt = Math.sqrt(x);

// 二维距离
double distance = Math.sqrt(dx * dx + dy * dy);

// 常见初始化
int smallest = Integer.MAX_VALUE;
int largest = Integer.MIN_VALUE;
```

Java 的 `Math` 类没有直接提供 `gcd()`。遇到最大公约数问题，通常手写欧几里得算法，或在适合的情况下使用 `BigInteger.gcd()`。

---

## 11. Character（字符处理）

当题目涉及字母、数字字符或大小写判断时，常用 `Character`。

| 方法                            | 用法示例                           | 作用       |
| ----------------------------- | ------------------------------ | -------- |
| ⭐ `Character.isDigit()`       | `Character.isDigit(c)`         | 是否为数字字符  |
| ⭐ `Character.isLetter()`      | `Character.isLetter(c)`        | 是否为字母    |
| `Character.isLetterOrDigit()` | `Character.isLetterOrDigit(c)` | 是否为字母或数字 |
| `Character.isWhitespace()`    | `Character.isWhitespace(c)`    | 是否为空白字符  |
| ⭐ `Character.toLowerCase()`   | `Character.toLowerCase(c)`     | 转小写      |
| `Character.toUpperCase()`     | `Character.toUpperCase(c)`     | 转大写      |

### 高频写法

```java
char c = s.charAt(i);

if (Character.isDigit(c)) {
    // 数字字符
}

if (Character.isLetter(c)) {
    // 字母
}

char lower = Character.toLowerCase(c);
```

---

## 12. Integer、Long 和类型转换

| 方法 / 常量                | 用法示例                    | 作用          |
| ---------------------- | ----------------------- | ----------- |
| ⭐ `Integer.parseInt()` | `Integer.parseInt(s)`   | 字符串转 `int`  |
| ⭐ `Long.parseLong()`   | `Long.parseLong(s)`     | 字符串转 `long` |
| `String.valueOf()`     | `String.valueOf(x)`     | 转字符串        |
| `Integer.toString()`   | `Integer.toString(x)`   | 整数转字符串      |
| `Integer.compare()`    | `Integer.compare(a, b)` | 比较两个整数      |
| `Long.compare()`       | `Long.compare(a, b)`    | 比较两个长整数     |
| `Integer.MAX_VALUE`    | `Integer.MAX_VALUE`     | `int` 上限    |
| `Long.MAX_VALUE`       | `Long.MAX_VALUE`        | `long` 上限   |

### 高频写法

```java
int x = Integer.parseInt("123");
long y = Long.parseLong("123456789");

String s = String.valueOf(x);

// 比较大小
int cmp = Integer.compare(a, b);

// 长整数
long total = 0L;
```

**Coding Test 中常见的选择：**

* `int`：一般整数范围内的计算。
* `long`：可能超过 `int` 范围的和、乘积、前缀和等。
* `double`：需要小数运算时。

---

## 13. TreeMap 和 TreeSet（有序映射和有序集合）

它们不像 `HashMap` 和 `HashSet` 那样主要用于快速无序查询，而是能保持键或元素的排序顺序。

```java
import java.util.TreeMap;
import java.util.TreeSet;
```

### TreeMap 常用方法

| 方法                 | 作用             |
| ------------------ | -------------- |
| `firstKey()`       | 最小的键           |
| `lastKey()`        | 最大的键           |
| `floorKey(x)`      | 小于或等于 `x` 的最大键 |
| `ceilingKey(x)`    | 大于或等于 `x` 的最小键 |
| `lowerKey(x)`      | 严格小于 `x` 的最大键  |
| `higherKey(x)`     | 严格大于 `x` 的最小键  |
| `pollFirstEntry()` | 删除并返回最小键值对     |
| `pollLastEntry()`  | 删除并返回最大键值对     |

### TreeSet 常用方法

| 方法            | 作用              |
| ------------- | --------------- |
| `first()`     | 最小元素            |
| `last()`      | 最大元素            |
| `floor(x)`    | 小于或等于 `x` 的最大元素 |
| `ceiling(x)`  | 大于或等于 `x` 的最小元素 |
| `lower(x)`    | 严格小于 `x` 的最大元素  |
| `higher(x)`   | 严格大于 `x` 的最小元素  |
| `pollFirst()` | 删除并返回最小元素       |
| `pollLast()`  | 删除并返回最大元素       |

**使用场景：** 需要维护有序数据、查询上下界、动态寻找最接近某个值的元素。

---

# PART 3 — JAVA VS PYTHON 对照表

这张对照表特别适合放在 Cheat Sheet 中间，方便你在两种语言之间快速切换。

| 功能          | Python                       | Java                                                 |
| ----------- | ---------------------------- | ---------------------------------------------------- |
| 获取数组 / 列表长度 | `len(arr)`                   | `arr.length`（数组）；`list.size()`（列表）                   |
| 获取字符串长度     | `len(s)`                     | `s.length()`                                         |
| 访问字符串字符     | `s[i]`                       | `s.charAt(i)`                                        |
| 数组排序        | `arr.sort()` / `sorted(arr)` | `Arrays.sort(arr)`                                   |
| 列表排序        | `arr.sort()`                 | `Collections.sort(list)` / `list.sort(...)`          |
| 反转          | `arr[::-1]`                  | `Collections.reverse(list)`；数组也可手动交换                 |
| 添加列表元素      | `arr.append(x)`              | `list.add(x)`                                        |
| 删除最后一个列表元素  | `arr.pop()`                  | 通常手动按索引处理，或用 `Deque`                                 |
| 哈希表         | `dict`                       | `HashMap`                                            |
| 哈希集合        | `set`                        | `HashSet`                                            |
| 安全获取默认值     | `d.get(k, 0)`                | `map.getOrDefault(k, 0)`                             |
| 检查键是否存在     | `k in d`                     | `map.containsKey(k)`                                 |
| 检查元素是否存在    | `x in arr` / `x in s`        | `list.contains(x)` / `set.contains(x)`               |
| 频率统计        | `Counter(arr)`               | `HashMap` + `getOrDefault()`                         |
| 栈入栈         | `stack.append(x)`            | `stack.push(x)`                                      |
| 栈出栈         | `stack.pop()`                | `stack.pop()`                                        |
| 队列入队        | `q.append(x)`                | `q.offer(x)`                                         |
| 队列出队        | `q.popleft()`                | `q.poll()`                                           |
| 优先队列        | `heapq`                      | `PriorityQueue`                                      |
| 最小堆         | `heapq` 默认                   | `PriorityQueue` 默认                                   |
| 最大堆         | 负数模拟                         | 传入逆序比较器                                              |
| 二分查找        | `bisect_left()`              | `Arrays.binarySearch()` 或手写                          |
| 获取最大值       | `max(arr)`                   | `Math.max(a, b)` 或遍历数组                               |
| 获取最小值       | `min(arr)`                   | `Math.min(a, b)` 或遍历数组                               |
| 求和          | `sum(arr)`                   | 通常遍历累加                                               |
| 去重          | `set(arr)`                   | `new HashSet<>(list)`                                |
| 字符串拆分       | `s.split()`                  | `s.split(...)`                                       |
| 字符串拼接       | `"".join(chars)`             | `StringBuilder`                                      |
| 字符串转整数      | `int(s)`                     | `Integer.parseInt(s)`                                |
| 整数转字符串      | `str(x)`                     | `String.valueOf(x)`                                  |
| 最大公约数       | `math.gcd(a, b)`             | 通常手写算法或用 `BigInteger.gcd()`                          |
| 无限大初始值      | `float("inf")`               | `Double.POSITIVE_INFINITY`；整数题常用 `Integer.MAX_VALUE` |

---

# PART 4 — Coding Test 高频算法模板

严格来说，这部分不是内置方法，但在实际 OA 和 LeetCode 中，**比记住更多 API 更重要**。

## 1. Frequency Count — 频率统计

**Python**

```python
from collections import Counter

freq = Counter(nums)
```

**Java**

```java
Map<Integer, Integer> freq = new HashMap<>();

for (int x : nums) {
    freq.put(x, freq.getOrDefault(x, 0) + 1);
}
```

## 2. Two Pointers — 双指针

适用于有序数组、回文判断和部分子数组问题。

**Python**

```python
left, right = 0, len(arr) - 1

while left < right:
    # 根据题目判断如何移动指针
    left += 1
    right -= 1
```

**Java**

```java
int left = 0, right = arr.length - 1;

while (left < right) {
    // 根据题目判断如何移动指针
    left++;
    right--;
}
```

## 3. Sliding Window — 滑动窗口

常用于连续子数组、最长不重复子串等问题。

**Python**

```python
left = 0

for right in range(len(nums)):
    # 加入 nums[right]

    while False:  # 替换为题目的窗口收缩条件
        # 移除 nums[left]
        left += 1
```

**Java**

```java
int left = 0;

for (int right = 0; right < nums.length; right++) {
    // 加入 nums[right]

    while (false) { // 替换为题目的窗口收缩条件
        // 移除 nums[left]
        left++;
    }
}
```

## 4. Prefix Sum — 前缀和

**Python**

```python
prefix = [0]

for x in nums:
    prefix.append(prefix[-1] + x)

# nums[l:r+1] 的区间和
total = prefix[r + 1] - prefix[l]
```

**Java**

```java
int[] prefix = new int[nums.length + 1];

for (int i = 0; i < nums.length; i++) {
    prefix[i + 1] = prefix[i] + nums[i];
}

// nums[l...r] 的区间和
int total = prefix[r + 1] - prefix[l];
```

如果累加结果可能超出 `int` 范围，Java 中应考虑将前缀和数组改为 `long[]`。

## 5. BFS — 广度优先搜索

**Python**

```python
from collections import deque

q = deque([start])
visited = {start}

while q:
    node = q.popleft()

    for nei in graph[node]:
        if nei not in visited:
            visited.add(nei)
            q.append(nei)
```

**Java**

```java
Queue<Integer> q = new ArrayDeque<>();
Set<Integer> visited = new HashSet<>();

q.offer(start);
visited.add(start);

while (!q.isEmpty()) {
    int node = q.poll();

    for (int nei : graph.get(node)) {
        if (visited.add(nei)) {
            q.offer(nei);
        }
    }
}
```

这里假设图使用邻接表表示；Java 示例还假设 `graph` 中每个节点对应的邻居列表都已初始化。

## 6. DFS — 深度优先搜索

**Python**

```python
def dfs(node):
    if node in visited:
        return

    visited.add(node)

    for nei in graph[node]:
        dfs(nei)
```

**Java**

```java
void dfs(int node) {
    if (visited.contains(node)) {
        return;
    }

    visited.add(node);

    for (int nei : graph.get(node)) {
        dfs(nei);
    }
}
```

递归深度可能很大时，需要注意语言的递归限制或使用显式栈实现 DFS。

## 7. Binary Search — 二分查找

**Python**

```python
left, right = 0, len(nums) - 1

while left <= right:
    mid = left + (right - left) // 2

    if nums[mid] == target:
        break
    elif nums[mid] < target:
        left = mid + 1
    else:
        right = mid - 1
```

**Java**

```java
int left = 0, right = nums.length - 1;

while (left <= right) {
    int mid = left + (right - left) / 2;

    if (nums[mid] == target) {
        break;
    } else if (nums[mid] < target) {
        left = mid + 1;
    } else {
        right = mid - 1;
    }
}
```

## 8. BFS / DFS / Heap / HashMap 之外，也要熟悉的主题

准备 Coding Test 时，建议同时熟悉这些常见算法和数据结构：

| 主题                  | 核心内容              |
| ------------------- | ----------------- |
| Sorting             | 升序、降序、自定义排序       |
| Binary Search       | 查找目标、下界、上界、答案二分   |
| Two Pointers        | 对撞指针、快慢指针         |
| Sliding Window      | 固定窗口、可变窗口         |
| Prefix Sum          | 一维、二维、前缀和 + 哈希表   |
| Stack               | 括号匹配、单调栈          |
| Queue / Deque       | BFS、滑动窗口最大值       |
| Heap                | Top K、合并有序序列      |
| HashMap / HashSet   | 频率、查重、快速查询        |
| DFS / BFS           | 树、图、网格遍历          |
| Backtracking        | 子集、排列、组合          |
| Dynamic Programming | 一维 DP、二维 DP、背包类问题 |
| Greedy              | 区间调度、局部最优策略       |
| Union-Find          | 连通分量、动态合并集合       |
| Graph Algorithms    | 拓扑排序、最短路径、连通性     |

---

# PART 5 — 高频精简版

### PYTHON — MUST MEMORIZE

```python
# Built-ins
len(arr)
range(n)
enumerate(arr)
zip(a, b)
sorted(arr, reverse=True)
sum(arr)
min(arr), max(arr), abs(x)

# List
arr.append(x)
arr.extend(other)
arr.pop()
arr.sort()
arr[::-1]
arr[l:r]

# String
s.split()
"".join(chars)
s.strip()
s.lower()
s.replace(a, b)
s.find(x)
s[::-1]

# Dict / Set
d.get(key, 0)
d.items()
freq[x] = freq.get(x, 0) + 1
x in s
s.add(x)

# Collections
Counter(nums)
 (int)
defaultdict(list)
deque()
q.append(x)
q.popleft()

# Heap / Binary Search
heapq.heappush(heap, x)
heapq.heappop(heap)
heapq.heapify(arr)
bisect_left(arr, x)
bisect_right(arr, x)

# Math
math.gcd(a, b)
math.sqrt(x)
math.ceil(x)
math.floor(x)
float("inf")
```

### JAVA — MUST MEMORIZE

```java
// String
s.length()
s.charAt(i)
s.substring(l, r)
s.equals(t)
s.compareTo(t)
s.contains(x)
s.indexOf(x)
s.split(",")
s.toCharArray()

// StringBuilder
sb.append(x)
sb.reverse()
sb.toString()

// Arrays / List
Arrays.sort(arr)
Arrays.binarySearch(arr, x)
Arrays.fill(arr, 0)
Arrays.copyOf(arr, n)
list.add(x)
list.get(i)
list.set(i, x)
list.remove(i)
list.size()

// HashMap / HashSet
map.put(k, v)
map.getOrDefault(k, 0)
map.containsKey(k)
map.keySet()
map.entrySet()
set.add(x)
set.contains(x)
set.remove(x)

// Stack / Queue / Heap
stack.push(x)
stack.pop()
stack.peek()
q.offer(x)
q.poll()
q.peek()
pq.offer(x)
pq.poll()
pq.peek()

// Collections / Math
Collections.sort(list)
Collections.reverse(list)
Math.max(a, b)
Math.min(a, b)
Math.abs(x)
Integer.parseInt(s)
Long.parseLong(s)
String.valueOf(x)
Integer.MAX_VALUE
Long.MAX_VALUE
```
