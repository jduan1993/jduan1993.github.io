---
title: 算法
---

# ✅ 20 个最常见算法题型 + 解题思路 & 模板（Java版）

> **适合力扣 0-200 题重点训练，也适用于面试快速准备**

---

## 🧩 1. 两数之和（哈希表）

**题型**：数组 + 查找

```java
// LC 1. Two Sum
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement))
            return new int[]{map.get(complement), i};
        map.put(nums[i], i);
    }
    return new int[0];
}
```

---

## 🧩 2. 三数之和（排序 + 双指针）

## ✅ 解题思路（最佳解）

### 1. **先排序（排序 + 双指针的前提）**

排序是为了：

* 方便跳过重复元素
* 使得双指针能正确移动（从两端收缩）

```java
Arrays.sort(nums);
```

---

### 2. **固定一个元素，用双指针扫另外两个**

核心思想：

* 外层固定一个数 `a = nums[i]`
* 内层用两个指针 `left` 和 `right` 找出 `b + c = -a`

### 3. **去重逻辑（关键！）**

* 如果当前 `nums[i] == nums[i - 1]`，跳过
* 如果 `nums[left] == nums[left - 1]`，跳过（避免重复组合）

---

### ✅ 代码模板（Java）：

```java
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    Arrays.sort(nums);

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // 跳过重复

        int left = i + 1;
        int right = nums.length - 1;

        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                res.add(Arrays.asList(nums[i], nums[left], nums[right]));

                // 去重
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;

                left++;
                right--;
            } else if (sum < 0) {
                left++;  // 需要更大的数
            } else {
                right--; // 需要更小的数
            }
        }
    }

    return res;
}
```

---

## 🧠 这题用到的“套路”组合是：

| 思路       | 原因或作用                   |
| -------- | ----------------------- |
| 排序       | 为了去重 & 双指针策略成立          |
| 固定 + 双指针 | 枚举 a，找 b+c              |
| 去重逻辑     | 避免重复结果项                 |
| 时间复杂度    | `O(n^2)`，比暴力 `O(n^3)` 优 |

---

## ✅ 总结套路式解题的重点：

| 题型识别   | 常用技巧          | 模板          |
| ------ | ------------- | ----------- |
| 3数求和   | 排序 + 双指针 + 去重 | Leetcode 15 |
| 两数之和   | 哈希表           | Leetcode 1  |
| 4数求和   | 双层循环 + 双指针    | Leetcode 18 |
| 有序数组找对 | 双指针           | 经典指针移动模式    |

---

## 🧩 3. 四数之和（套用三数之和思路）

```java
// LC 18. Four Sum
public List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> res = new ArrayList<>();
    Arrays.sort(nums);
    int n = nums.length;

    for (int i = 0; i < n - 3; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        for (int j = i + 1; j < n - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j - 1]) continue;

            int left = j + 1, right = n - 1;
            while (left < right) {
                long sum = (long) nums[i] + nums[j] + nums[left] + nums[right];
                if (sum == target) {
                    res.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;
                    left++;
                    right--;
                } else if (sum < target) left++;
                else right--;
            }
        }
    }

    return res;
}
```

---

## 🧩 4. 合并区间（排序 + 扫描）

```java
// LC 56. Merge Intervals
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    List<int[]> res = new ArrayList<>();
    for (int[] interval : intervals) {
        if (res.isEmpty() || res.get(res.size() - 1)[1] < interval[0])
            res.add(interval);
        else
            res.get(res.size() - 1)[1] = Math.max(res.get(res.size() - 1)[1], interval[1]);
    }
    return res.toArray(new int[res.size()][]);
}
```

---

## 🧩 5. 移动零（双指针）

```java
// LC 283. Move Zeroes
public void moveZeroes(int[] nums) {
    int index = 0;
    for (int num : nums)
        if (num != 0)
            nums[index++] = num;
    while (index < nums.length)
        nums[index++] = 0;
}
```

---

## 🧩 6. 滑动窗口最大值

```java
// LC 239. Sliding Window Maximum
public int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new LinkedList<>();
    int[] res = new int[nums.length - k + 1];
    for (int i = 0; i < nums.length; i++) {
        while (!deque.isEmpty() && deque.peekFirst() < i - k + 1)
            deque.pollFirst();
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i])
            deque.pollLast();
        deque.offerLast(i);
        if (i >= k - 1) res[i - k + 1] = nums[deque.peekFirst()];
    }
    return res;
}
```

---

## 🧩 7. 最长无重复子串（滑动窗口 + 哈希）

```java
// LC 3. Longest Substring Without Repeating Characters
public int lengthOfLongestSubstring(String s) {
    Set<Character> set = new HashSet<>();
    int left = 0, maxLen = 0;
    for (int right = 0; right < s.length(); right++) {
        while (set.contains(s.charAt(right)))
            set.remove(s.charAt(left++));
        set.add(s.charAt(right));
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

---

## 🧩 8. 二分查找模板

```java
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

---

## 🧩 9. 反转链表（迭代）

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    while (head != null) {
        ListNode next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

---

## 🧩 10. 合并两个有序链表

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(-1), cur = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val < l2.val) {
            cur.next = l1;
            l1 = l1.next;
        } else {
            cur.next = l2;
            l2 = l2.next;
        }
        cur = cur.next;
    }
    cur.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

---

## 🧩 11. LRU 缓存（LinkedHashMap 或 手写双向链表 + 哈希表）

```java
// LC 146. LRU Cache
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder=true
        this.capacity = capacity;
    }

    public int get(int key) {
        return super.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        super.put(key, value);
    }

    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
```

---

## 🧩 12. 最长回文子串（中心扩展法）

```java
// LC 5. Longest Palindromic Substring
public String longestPalindrome(String s) {
    int start = 0, maxLen = 0;
    for (int i = 0; i < s.length(); i++) {
        int len1 = expand(s, i, i);
        int len2 = expand(s, i, i + 1);
        int len = Math.max(len1, len2);
        if (len > maxLen) {
            start = i - (len - 1) / 2;
            maxLen = len;
        }
    }
    return s.substring(start, start + maxLen);
}

private int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) {
        l--; r++;
    }
    return r - l - 1;
}
```

---

## 🧩 13. 爬楼梯（斐波那契）

```java
// LC 70. Climbing Stairs
public int climbStairs(int n) {
    if (n <= 2) return n;
    int a = 1, b = 2;
    for (int i = 3; i <= n; i++) {
        int tmp = a + b;
        a = b;
        b = tmp;
    }
    return b;
}
```

---

## 🧩 14. 最小路径和（动态规划）

```java
// LC 64. Minimum Path Sum
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    for (int i = 1; i < m; i++) grid[i][0] += grid[i - 1][0];
    for (int j = 1; j < n; j++) grid[0][j] += grid[0][j - 1];
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            grid[i][j] += Math.min(grid[i - 1][j], grid[i][j - 1]);
    return grid[m - 1][n - 1];
}
```

---

## 🧩 15. 单词拆分（动态规划 + HashSet）

```java
// LC 139. Word Break
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> wordSet = new HashSet<>(wordDict);
    boolean[] dp = new boolean[s.length() + 1];
    dp[0] = true;

    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && wordSet.contains(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }

    return dp[s.length()];
}
```

---

## 🧩 16. 岛屿数量（DFS or BFS）

```java
// LC 200. Number of Islands
public int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++)
        for (int j = 0; j < grid[0].length; j++)
            if (grid[i][j] == '1') {
                dfs(grid, i, j);
                count++;
            }
    return count;
}

private void dfs(char[][] g, int i, int j) {
    if (i < 0 || j < 0 || i >= g.length || j >= g[0].length || g[i][j] == '0') return;
    g[i][j] = '0';
    dfs(g, i - 1, j); dfs(g, i + 1, j);
    dfs(g, i, j - 1); dfs(g, i, j + 1);
}
```

---

## 🧩 17. 括号匹配（栈）

```java
// LC 20. Valid Parentheses
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '[') stack.push(']');
        else if (c == '{') stack.push('}');
        else if (stack.isEmpty() || stack.pop() != c)
            return false;
    }
    return stack.isEmpty();
}
```

---

## 🧩 18. K 个一组反转链表（递归）

```java
// LC 25. Reverse Nodes in k-Group
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode node = head;
    for (int i = 0; i < k; i++) {
        if (node == null) return head;
        node = node.next;
    }
    ListNode newHead = reverse(head, node);
    head.next = reverseKGroup(node, k);
    return newHead;
}

private ListNode reverse(ListNode a, ListNode b) {
    ListNode prev = null, curr = a;
    while (curr != b) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

---

## 🧩 19. 二叉树层序遍历（BFS）

```java
// LC 102. Binary Tree Level Order Traversal
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);

    while (!q.isEmpty()) {
        int size = q.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = q.poll();
            level.add(node.val);
            if (node.left != null) q.offer(node.left);
            if (node.right != null) q.offer(node.right);
        }
        res.add(level);
    }

    return res;
}
```

---

## 🧩 20. 判断回文链表（快慢指针 + 链表反转）

```java
// LC 234. Palindrome Linked List
public boolean isPalindrome(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    ListNode second = reverse(slow);
    ListNode first = head;
    while (second != null) {
        if (first.val != second.val) return false;
        first = first.next;
        second = second.next;
    }
    return true;
}

private ListNode reverse(ListNode head) {
    ListNode prev = null;
    while (head != null) {
        ListNode next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

---

# ✅ 你可以怎么用这份资料？

1. **刷题时对照查阅**（能套模板就套）
2. **建立“题型 - 思路 - 模板”三位一体体系**
3. **反复讲给自己听**（越能解释，越会写）
4. 可用于面试前复习速览

---
