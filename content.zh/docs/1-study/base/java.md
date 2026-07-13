---
title: Java 基础
---

# Java Coding面试常用数据结构与方法

## 1. Array（数组）

### 基本操作
```java
// 声明和初始化
int[] arr = new int[10];
int[] arr2 = {1, 2, 3, 4, 5};

// 获取长度
int length = arr.length;

// 遍历
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}

// 增强for循环
for (int num : arr) {
    System.out.println(num);
}

// 排序
Arrays.sort(arr);  // 升序
Arrays.sort(arr, Collections.reverseOrder());  // 降序（需要Integer[]）

// 二分查找（必须先排序）
int index = Arrays.binarySearch(arr, target);

// 填充
Arrays.fill(arr, 0);  // 全部填充为0

// 深拷贝基本类型数组
int[] deepCopy = Arrays.copyOf(arr, arr.length);
int[] deepCopyRange = Arrays.copyOfRange(arr, 0, 3);

// 或者使用System.arraycopy
int[] deepCopy2 = new int[arr.length];
System.arraycopy(arr, 0, deepCopy2, 0, arr.length);
```

### 常用技巧
```java
// 数组转List
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

// 复制数组
int[] copy = Arrays.copyOf(arr, arr.length);
int[] copyRange = Arrays.copyOfRange(arr, 0, 3);

// 比较数组
boolean equals = Arrays.equals(arr1, arr2);
```

## 2. List（列表）

### ArrayList
```java
// 声明和初始化
List<Integer> list = new ArrayList<>();
List<Integer> list2 = new ArrayList<>(Arrays.asList(1, 2, 3));

// 基本操作
list.add(1);                    // 添加元素
list.add(0, 0);                // 在指定位置添加
list.set(0, 10);               // 设置指定位置的元素
int element = list.get(0);      // 获取元素
list.remove(0);                 // 删除指定位置的元素
list.remove(Integer.valueOf(1)); // 删除指定值的元素
int size = list.size();         // 获取大小
boolean isEmpty = list.isEmpty(); // 判断是否为空

// 查找
boolean contains = list.contains(1);
int index = list.indexOf(1);
int lastIndex = list.lastIndexOf(1);

// 遍历
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}

for (Integer num : list) {
    System.out.println(num);
}

// 排序
Collections.sort(list);  // 升序
Collections.sort(list, Collections.reverseOrder());  // 降序
list.sort(Comparator.naturalOrder());  // 升序
list.sort(Comparator.reverseOrder());  // 降序

// 反转
Collections.reverse(list);
```

### LinkedList
```java
LinkedList<Integer> linkedList = new LinkedList<>();
linkedList.addFirst(1);        // 在开头添加
linkedList.addLast(2);         // 在结尾添加
linkedList.removeFirst();       // 删除第一个
linkedList.removeLast();        // 删除最后一个
int first = linkedList.getFirst();  // 获取第一个
int last = linkedList.getLast();    // 获取最后一个
```

## 3. String（字符串）

### 基本操作
```java
String str = "Hello World";

// 长度
int length = str.length();

// 字符访问
char ch = str.charAt(0);

// 子串
String sub = str.substring(0, 5);  // "Hello"
String sub2 = str.substring(6);    // "World"

// 查找
int index = str.indexOf("o");      // 第一次出现的位置
int lastIndex = str.lastIndexOf("o"); // 最后一次出现的位置
boolean contains = str.contains("Hello");

// 替换
String replaced = str.replace("World", "Java");
String replacedAll = str.replaceAll("o", "0");

// 分割
String[] parts = str.split(" ");   // ["Hello", "World"]

// 连接
String joined = String.join("-", "Hello", "World"); // "Hello-World"

// 转换
String upper = str.toUpperCase();
String lower = str.toLowerCase();
String trimmed = str.trim();       // 去除首尾空格

// 比较
boolean equals = str.equals("Hello World");
boolean equalsIgnoreCase = str.equalsIgnoreCase("hello world");
int compare = str.compareTo("Hello Java");
```

### 常用技巧
```java
// 字符串转字符数组
char[] chars = str.toCharArray();

// 字符数组转字符串
String fromChars = new String(chars);

// 字符串构建器
StringBuilder sb = new StringBuilder();
sb.append("Hello");
sb.append(" ");
sb.append("World");
String result = sb.toString();

// 格式化
String formatted = String.format("Hello %s", "World");
```

## 4. char（字符）

### 基本操作
```java
char ch = 'A';

// 字符转换
int ascii = (int) ch;           // 转ASCII码
char fromAscii = (char) 65;     // ASCII码转字符

// 字符判断
boolean isLetter = Character.isLetter(ch);
boolean isDigit = Character.isDigit(ch);
boolean isUpperCase = Character.isUpperCase(ch);
boolean isLowerCase = Character.isLowerCase(ch);
boolean isWhitespace = Character.isWhitespace(ch);

// 字符转换
char upper = Character.toUpperCase(ch);
char lower = Character.toLowerCase(ch);

// 数字字符转数字
int digit = Character.getNumericValue('5');  // 5
```

## 5. Map（映射）

### HashMap
```java
// 声明和初始化
Map<String, Integer> map = new HashMap<>();
Map<String, Integer> map2 = new HashMap<>(Map.of("a", 1, "b", 2));

// 基本操作
map.put("key", 1);              // 添加或更新
map.putIfAbsent("key", 2);      // 如果不存在则添加
Integer value = map.get("key");  // 获取值
Integer valueOrDefault = map.getOrDefault("key", 0); // 获取值，不存在返回默认值
map.remove("key");               // 删除键值对
map.clear();                     // 清空

// 查找
boolean containsKey = map.containsKey("key");
boolean containsValue = map.containsValue(1);
int size = map.size();
boolean isEmpty = map.isEmpty();

// 遍历
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

for (String key : map.keySet()) {
    System.out.println(key + ": " + map.get(key));
}

for (Integer val : map.values()) {
    System.out.println(val);
}

// 合并
map.merge("key", 1, Integer::sum);  // 如果存在则累加，不存在则添加
```

### TreeMap（有序）
```java
TreeMap<String, Integer> treeMap = new TreeMap<>();
treeMap.put("c", 3);
treeMap.put("a", 1);
treeMap.put("b", 2);

// 有序遍历
for (String key : treeMap.keySet()) {
    System.out.println(key + ": " + treeMap.get(key));
}

// 获取第一个和最后一个
String firstKey = treeMap.firstKey();
String lastKey = treeMap.lastKey();
Map.Entry<String, Integer> firstEntry = treeMap.firstEntry();
Map.Entry<String, Integer> lastEntry = treeMap.lastEntry();

// 获取小于或大于某个键的视图
SortedMap<String, Integer> headMap = treeMap.headMap("c");
SortedMap<String, Integer> tailMap = treeMap.tailMap("a");
```

## 6. Deque（双端队列）

### ArrayDeque
```java
Deque<Integer> deque = new ArrayDeque<>();

// 在开头操作
deque.addFirst(1);
deque.offerFirst(2);
deque.removeFirst();
deque.pollFirst();
Integer first = deque.getFirst();
Integer peekFirst = deque.peekFirst();

// 在结尾操作
deque.addLast(3);
deque.offerLast(4);
deque.removeLast();
deque.pollLast();
Integer last = deque.getLast();
Integer peekLast = deque.peekLast();

// 栈操作（LIFO）
deque.push(5);      // 压栈
Integer pop = deque.pop();  // 弹栈
Integer peek = deque.peek(); // 查看栈顶

// 队列操作（FIFO）
deque.offer(6);     // 入队
Integer poll = deque.poll(); // 出队
Integer element = deque.element(); // 查看队首

// 遍历
for (Integer num : deque) {
    System.out.println(num);
}
```

## 7. Set（集合）

### HashSet
```java
Set<Integer> set = new HashSet<>();
set.add(1);
set.add(2);
set.add(1);  // 重复元素不会添加

boolean contains = set.contains(1);
set.remove(1);
int size = set.size();

// 遍历
for (Integer num : set) {
    System.out.println(num);
}

// 集合操作
Set<Integer> set2 = new HashSet<>(Arrays.asList(2, 3, 4));
set.addAll(set2);           // 并集
set.retainAll(set2);        // 交集
set.removeAll(set2);        // 差集
```

### TreeSet（有序）
```java
TreeSet<Integer> treeSet = new TreeSet<>();
treeSet.add(3);
treeSet.add(1);
treeSet.add(2);

// 有序遍历
for (Integer num : treeSet) {
    System.out.println(num);
}

// 获取第一个和最后一个
Integer first = treeSet.first();
Integer last = treeSet.last();

// 获取小于或大于某个值的视图
SortedSet<Integer> headSet = treeSet.headSet(3);
SortedSet<Integer> tailSet = treeSet.tailSet(1);
```

## 8. 常用算法技巧

### 双指针
```java
// 快慢指针
int slow = 0, fast = 0;
while (fast < nums.length) {
    if (condition) {
        slow++;
    }
    fast++;
}

// 左右指针
int left = 0, right = nums.length - 1;
while (left < right) {
    if (condition) {
        left++;
    } else {
        right--;
    }
}
```

### 滑动窗口
```java
int left = 0, right = 0;
int sum = 0;
while (right < nums.length) {
    sum += nums[right];
    while (sum >= target) {
        // 更新结果
        sum -= nums[left];
        left++;
    }
    right++;
}
```

### 前缀和
```java
int[] prefixSum = new int[nums.length + 1];
for (int i = 0; i < nums.length; i++) {
    prefixSum[i + 1] = prefixSum[i] + nums[i];
}

// 计算区间和
int rangeSum = prefixSum[right + 1] - prefixSum[left];
```

## 9. 面试常见问题

### 时间复杂度
- 数组访问：O(1)
- 数组搜索：O(n)
- 数组排序：O(n log n)
- HashMap操作：平均O(1)，最坏O(n)
- TreeMap操作：O(log n)
- ArrayList添加/删除：O(1)（末尾），O(n)（中间）
- LinkedList添加/删除：O(1)（头尾），O(n)（中间）

### 空间复杂度
- 原地算法：O(1)
- 递归：O(递归深度)
- 动态规划：O(n) 或 O(n²)

### 常用技巧
1. **使用HashMap优化查找**：将O(n)的查找优化为O(1)
2. **双指针技巧**：解决数组中的查找、去重等问题
3. **滑动窗口**：解决子数组、子串等问题
4. **单调栈/队列**：解决下一个更大元素等问题
5. **前缀和**：快速计算区间和
6. **差分数组**：快速进行区间更新
