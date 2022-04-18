---
title: 剑指Offer系列
date: 2022-04-06 19:10:42
tags:
  - 剑指Offer
categories:
  - 剑指Offer

---

# 剑指Offer系列题解

## 数组

### 1 数组中重复的数字

> https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/

#### 解一

数组排序后遍历

```java
public int findRepeatNumber(int[] nums) {
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 1; i++) {
        if (nums[i] == nums[i + 1]){
            return nums[i];
        }
    }
    return -1;
}
```

时间复杂度为 O(nlogn) 

#### 解二

使用哈希表

```java
public int findRepeatNumber(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int i = 0; i < nums.length; i++) {
        if (set.contains(nums[i])) {
            return nums[i];
        } else {
            set.add(nums[i]);
        }
    }
    return -1;
}
```

时间复杂度为O(n)，空间复杂度为O(n)

#### 解三

使用原地重排数组

```java
public int findRepeatNumber(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        while (nums[i] != i) {
            if (nums[i] == nums[nums[i]]) {
                return nums[i];
            }
            int tmp = nums[nums[i]];
            nums[nums[i]] = nums[i];
            nums[i] = tmp;
        }
    }
    return -1;
}
```

时间复杂度O(n)，空间复杂度O(1)

### 2 不修改数组找出重复的数字

> 保留

### 3 二维数组中的查找

> https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/

#### 解一

从左下往上搜索，小于目标值，往右，大于目标值，往上

```java
public boolean findNumberIn2DArray(int[][] matrix, int target) {
    int m = matrix.length - 1;
    int j = 0;
    while (m >= 0 && j < matrix[0].length) {
        if (target < matrix[m][j]) {
            m--;
        } else if (target > matrix[m][j]) {
            j++;
        } else {
            return true;
        }
    }
    return false;
}
```

#### 解二

从右上往下搜索，大于目标值，往左，小于目标值，往下

```java
public boolean findNumberIn2DArray(int[][] matrix, int target) {
    int m = 0;
    if (matrix.length == 0 || matrix[0].length == 0) {
        return false;
    }
    int j = matrix[0].length - 1;
    while (m < matrix.length && j >= 0) {
        if (target < matrix[m][j]) {
            j--;
        } else if (target > matrix[m][j]) {
            m++;
        } else {
            return true;
        }
    }
    return false;
}
```

## 字符串

### 4 替换空格

> https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/

#### 解一

遍历字符串，使用StringBuilder拼接字符

```java
public String replaceSpace(String s) {
    StringBuilder res = new StringBuilder();
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (c == ' ') {
            res.append("%20");
        } else {
            res.append(c);
        } 
    }
    return res.toString();
}
```

#### 解二

从后往前添加字符串

> 【举一反三】
> 在合并两个数组（包括字符串）时，如果从前往后复制每个值需要重复移动值，那么可以考虑从后往前复制，减少移动次数，提高效率。

## 链表

### 5 从尾到头打印链表

> https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/

> 在面试中，最好先问面试官是否允许修改输入的数据。

#### 解一

使用先进后出的数据结构【栈】

```java
public int[] reversePrint(ListNode head) {
    if (head == null) {
        return new int[0];
    }
    Stack<ListNode> stack = new Stack<>();
    ListNode cur = head;
    while (cur != null) {
        stack.push(cur);
        cur = cur.next;
    }
    int[] res = new int[stack.size()];
    int index = 0;
    while (!stack.isEmpty()) {
        res[index++] = stack.pop().val;
    }
    return res;
}
```

#### 解二

反转链表

```java
public int[] reversePrint(ListNode head) {
    // 头插法反转链表
    ListNode dummy = new ListNode(-1);
    ListNode cur = head;
    int size = 0;
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = dummy.next;
        dummy.next = cur;
        cur = next;
        size++;
    }
    int[] res = new int[size];
    cur = dummy.next;
    size = 0;
    while (cur != null) {
        res[size++] = cur.val;
        cur = cur.next;
    }
    return res;
}
```

#### 解三

递归反转链表（修改输入数据）

```java
public int[] reversePrint(ListNode head) {
    ListNode dummy = reverseListNode(head);
    ListNode cur = dummy;
    int size = 0;
    while (cur != null) {
        cur = cur.next;
        size++;
    }
    int[] res = new int[size];
    cur = dummy;
    size = 0;
    while (cur != null) {
        res[size++] = cur.val;
        cur = cur.next;
    }
    return res;
}
public ListNode reverseListNode(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode next = reverseListNode(head.next);
    head.next.next = head;
    head.next = null;
    return next;
}
```

## 树

### 6 重建二叉树

#### 解一

根据先序遍历找根节点，然后根据根节点在中序遍历中划分左右子树

```java
public TreeNode buildTree(int[] preorder, int[] inorder) {
    //
    if (preorder.length == 0) {
        return null;
    }
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) {
        map.put(inorder[i], i);
    }
    return helper(preorder, 0, 0, inorder.length - 1, map);
}

private TreeNode helper(int[] preorder, int pStart, int iStart, int iEnd, Map<Integer, Integer> map) {
    if (pStart >= preorder.length || iStart > iEnd) {
        return null;
    }
    int rootVal = preorder[pStart];
    int rootIndex = map.get(rootVal);
    TreeNode root = new TreeNode(rootVal);
    // 左子树的长度为 rootIndex - iStart
    // 右子树的根索引为 pStart + (rootIndex - iStart) + 1
    root.left = helper(preorder, pStart + 1, iStart, rootIndex - 1, map);
    root.right = helper(preorder, pStart + (rootIndex - iStart) + 1, rootIndex + 1, iEnd, map);
    return root;
} 
```

## 栈和队列

### 7 用两个栈实现队列

#### 解一

```java
class CQueue {

    private LinkedList<Integer> inStack;
    private LinkedList<Integer> outStack;

    public CQueue() {
        inStack = new LinkedList();
        outStack = new LinkedList();
    }
    
    public void appendTail(int value) {
        inStack.push(value);
    }
    
    public int deleteHead() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) {
                outStack.push(inStack.poll());
            }
        }
        return outStack.isEmpty() ? -1 : outStack.poll();
    }
}
```

### 8 用两个队列实现栈

#### 解一

```java
class MyStack {

    private LinkedList<Integer> inQueue;
    private LinkedList<Integer> outQueue;
    private int size;
    public MyStack() {
        inQueue = new LinkedList<>();
        outQueue = new LinkedList<>();
        size = 0;
    }
    
    // 直接向队列1添加就行
    public void push(int x) {
        inQueue.add(x);
        size++;
    }
    
    public int pop() {
        if (size == 0) {
            return -1;
        }
        for (int i = 0; i < size - 1; i++) {
            outQueue.add(inQueue.remove());
        }
        int res = inQueue.remove();
        size--;
        for (int i = 0; i < size; i++) {
            inQueue.add(outQueue.remove());
        }
        return res;
    }
    
    public int top() {
        int res = pop();
        inQueue.add(res);
        size++;
        return res;
    }
    
    public boolean empty() {
        return size == 0;
    }
}
```

## 递归和循环

### 9 斐波那契数列

#### 解一

```java
public int fib(int n) {
    int a = 0;
    int b = 1;
    if (n == 0 || n == 1) {
        return n;
    }
    for (int i = 2; i <= n; i++) {
        int tmp = b % 1000000007;
        b = (a % 1000000007 + b % 1000000007) % 1000000007;
        a = tmp;
    }
    return b;
}
```

### 10 青蛙跳台阶问题

#### 解一

```java
public int numWays(int n) {
    int a = 1, b = 2;
    if (n == 0 || n == 1) {
        return 1;
    }
    for (int i = 2; i < n; i++) {
        int tmp = b % 1000000007;
        b = (a % 1000000007 + b % 1000000007) % 1000000007;
        a = tmp;
    }
    return b;
}
```

## 查找和排序

### 11 旋转数组的最小数字

#### 解一

直接遍历

```java
public int minArray(int[] numbers) {
    int minNumber = numbers[0];
    for (int i = 1; i < numbers.length; i++) {
        if (numbers[i] < minNumber) {
            return numbers[i];
        }
    }
    return minNumber;
}
```

#### 解二

二分

不能用中间值与左值进行比较

```java
public int minArray(int[] numbers) {
    int len = numbers.length;
    if (len == 1) {
        return numbers[0];
    }
    int l = 0;
    int r = len - 1;
    // 第一次二分找到转折点
    while (l < r) {
        int mid = (l + r) / 2;
        if (numbers[mid] > numbers[r]) {
            l = mid + 1;
        } else if (numbers[mid] == numbers[r]) {
            r = r - 1;
        } else {
            r = mid;
        }
    }
    return numbers[l];
}
```

## 回溯法

### 12 矩阵中的路径

#### 解一

```java
public boolean exist(char[][] board, String word) {
    char[] wordArray = word.toCharArray();
    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[0].length; j++) {
            if (dfs(board, wordArray, i, j, 0)) {
                return true;
            }
        }
    }
    return false;
}

public boolean dfs(char[][] board, char[] word, int i, int j, int k) {
    // 剪枝
    if (i >= board.length || i < 0 || j >= board[0].length || j < 0 || board[i][j] != word[k]) {
        return false;
    }
    if (k == word.length - 1) {
        return true;
    }

    // 往四个方向继续走
    // mask当前值
    board[i][j] = '\0';
    boolean res =  dfs(board, word, i - 1, j, k + 1) || dfs(board, word, i + 1, j, k + 1) || dfs(board, word, i, j - 1, k + 1) || dfs(board, word, i, j + 1, k + 1);
    board[i][j] = word[k];
    return res;
}
```

### 13 机器人的运动范围

#### 解一

```java
class Solution {
    int res = 0;
    public int movingCount(int m, int n, int k) {
        boolean[][] visited = new boolean[m][n];
        dfs(m, n, 0, 0, k, visited);
        return res;
    }

    private void dfs(int m, int n, int i, int j, int k, boolean[][] visited) {
        // 剪枝
        if (i < 0 || j < 0 || i >= m || j >= n || visited[i][j]) {
            return;
        }
        if (getSum(i) + getSum(j) > k) {
            return;
        }
        visited[i][j] = true;
        res++;
        dfs(m, n, i - 1, j, k, visited);
        dfs(m, n, i + 1, j, k, visited);
        dfs(m, n, i, j - 1, k, visited);
        dfs(m, n, i, j + 1, k, visited);
        return;
    }

    private int getSum(int i) {
        int count = 0;
        while (i != 0) {
            count += (i % 10);
            i /= 10;
        }
        return count;
    }
}
```

## 动态规划和贪心算法

### 14 剪绳子

## 位运算

### 15 二进制中1的个数

### 16 数值的整数次方

#### 解一

注意的两个点：

- n为Integer.MIN_VALUE时，直接使用int转为正值会溢出
- 直接遍历n进行相乘会超时

```java
public double myPow(double x, int n) {
    if (x == 0) {
        return 0;
    }
    if (n == 0) {
        return 1;
    }
    long b = n;
    if (b < 0) {
        x = 1.0 / x;
        b = -b;
    }
    double res = 1.0;
    while (b > 0) {
        if ((b & 1) == 1) {
            res *= x;
        }
        x *=x;
        b >>= 1;
    }
    return res;
}
```

### 17 打印从1到最大的n位数

#### 解一

注意的两个点：

- 大数越界问题，可以使用String来解决

```java
public int[] printNumbers(int n) {
    int end = (int)Math.pow(10, n) - 1;
    int[] res = new int[end];
    for (int i = 0; i < end; i++) {
        res[i] = i + 1;
    }
    return res;
}
```

### 18 删除链表的节点

#### 解一

```java
public ListNode deleteNode(ListNode head, int val) {
    if (head == null) {
        return head;
    }
    if (head.val == val) {
        return head.next;
    }
    ListNode p = head;
    ListNode cur = p.next;
    // 从next开始遍历
    while (cur != null) {
        if (cur.val == val) {
            p.next = cur.next;
            return head;
        }
        p = p.next;
        cur = cur.next;
    }
    return head;
}
```

### 19 删除排序链表中的重复元素

#### 解一

```java
public ListNode deleteDuplicates(ListNode head) {
    if (head == null) {
        return head;
    }
    ListNode p = head;
    ListNode cur = p.next;
    while (cur != null) {
        if (cur.val == p.val) {
            cur = cur.next;
            p.next = cur;
        } else {
            p = p.next;
            cur = cur.next;
        }
    }
    return head;
}
```

### 20 删除排序链表中的所有重复元素

#### 解一

```java
public ListNode deleteDuplicates(ListNode head) {
    if (head == null) {
        return head;
    }
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode cur = dummy;
    while (cur.next != null && cur.next.next != null) {
        if (cur.next.val == cur.next.next.val) {
            int curVal = cur.next.val;
            while (cur.next != null && cur.next.val == curVal) {
                cur.next = cur.next.next;
            }
        } else {
            cur = cur.next;
        }
    }
    return dummy.next;
}
```

### 21 正则表达式匹配

【困难】

#### 解一

首先，定义dp

```java
/**
dp[i][j] 表示 s[:i]与p[:j]匹配
s[:i] 表示s的前i个字符
p[:j] 表示p的前j个字符
若 s[:i] == p[:j]， 那么下一状态就是判断添加一个字符s[i+1]能否匹配或者添加一个字符p[j+1]能否匹配
// 初始化 
dp[0][0] 表示空字符串，dp[0][0] == true
dp[0][j] 此时，当p的偶数位为'*'才能匹配，需要保持p为空字符串

当p[j-1]为'*'时，可以匹配dp[i][j-2] || dp[i-1][j]


*/    
```



```java
public boolean isMatch(String s, String p) {
    int m = s.length() + 1;
    int n = p.length() + 1;
    boolean[][] dp = new boolean[m][n];
    dp[0][0] = true;
    // 初始化首行
    for (int j = 2; j < n; j += 2) {
        dp[0][j] = dp[0][j - 2] && p.charAt(j - 1) == '*';
    }
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            if (p.charAt(j - 1) == '*') {
                dp[i][j] = dp[i][j - 2] || (dp[i - 1][j] && (s.charAt(i - 1) == p.charAt(j - 2) || p.charAt(j - 2) == '.'));
            } else {
                dp[i][j] = dp[i - 1][j - 1] && ((s.charAt(i - 1) == p.charAt(j - 1)) || (p.charAt(j - 1) == '.'));
            }
        }
    }
    return dp[m - 1][n - 1];
}
```

### 21 调整数组顺序使奇数位于偶数前面

#### 解一

【注意】

注意边界

```java
public int[] exchange(int[] nums) {
    int l = 0;
    int r = nums.length - 1;
    while (l < r) {
        while (l < r && ((nums[r] & 1) == 0)) {
            r--;
        }
        while (l < r && ((nums[l] & 1) == 1)) {
            l++;
        }
        int t = nums[l];
        nums[l] = nums[r];
        nums[r] = t;
    }
    return nums;
}
```

### 22 链表中倒数第K个节点

#### 解一

双指针

```java
public ListNode getKthFromEnd(ListNode head, int k) {
    ListNode fast = head;
    ListNode slow = head;
    while (fast != null) {
        fast = fast.next;
        k--;
        if (k < 0) {
            slow = slow.next;
        }
    }
    return slow;
}
```

### 23 链表中环的入口

#### 解一

双指针遍历，能走完说明没有环

否则，让一个指针重新从头开始遍历，另一个继续遍历，相等时即为环的入口

```java
public ListNode detectCycle(ListNode head) {
    if (head == null || head.next == null) {
        return null;
    }
    ListNode fast = head, slow = head;
    while (fast != null) {
        slow = slow.next;
        if (fast.next != null) {
            fast = fast.next.next;
        } else {
            return null;
        }
        if (fast == slow) {
            fast = head;
            while (slow != fast) {
                slow = slow.next;
                fast = fast.next;
            }
            return fast;
        }
    }
    return null;
}
```

### 24 反转链表

#### 解一

递归

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode returnNode = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return returnNode;
}
```

#### 解二

使用栈进行反转

### 25 合并排序链表

#### 解一

使用队列来保存

```java
public ListNode mergeKLists(ListNode[] lists) {
    ListNode cur = new ListNode(0);
    ListNode res = cur;
    int n = lists.length;
    PriorityQueue<ListNode> pq = new PriorityQueue<>((n1, n2) -> n1.val - n2.val);
    for (ListNode node : lists) {
        if (node != null) {
            pq.offer(node);
        }
    }
    while (!pq.isEmpty()) {
        ListNode curNode = pq.poll();
        res.next = curNode;
        res = res.next;
        if (curNode.next != null) {
            pq.offer(curNode.next);
        }
    }
    return cur.next;
}
```

### 26 树的子结构

#### 解一

先判断当前的节点A和B是否相同，然后比较A.l和B.l 、A.r和B.r

【注意】判断double类型的变量是否相等时，需要判断两数之差是否在一个范围内，不能直接使用==

```java
public boolean isSubStructure(TreeNode A, TreeNode B) {
    if (B == null || A == null) {
        return false;
    }
    return hasSubStructure(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
}

private boolean hasSubStructure(TreeNode A, TreeNode B) {
    if (B == null) {
        return true;
    }
    if (A == null || A.val != B.val) {
        return false;
    }
    return hasSubStructure(A.left, B.left) && hasSubStructure(A.right, B.right);
}
```

### 27 二叉树的镜像

#### 解一

```java
public TreeNode mirrorTree(TreeNode root) {
    if (root == null) {
        return null;
    }
    TreeNode tmp = root.left;
    root.left = mirrorTree(root.right);
    root.right = mirrorTree(tmp);
    return root;
}
```

### 28 对称的二叉树

#### 解一

构造一个辅助函数，进行对比

```java
public boolean isSymmetric(TreeNode root) {
    return check(root, root);
}

private boolean check(TreeNode p, TreeNode q) {
    if (p == null && q == null) {
        return true;
    }
    if ((p == null && q != null) || (p != null && q == null) || p.val != q.val) {
        return false;
    }
    return check(p.left, q.right) && check(p.right, q.left);
}
```

### 29 顺时针打印矩阵

#### 解一

```java
public int[] spiralOrder(int[][] matrix) {
    int m = matrix.length;
    if (m == 0) {
        return new int[0];
    }
    int n = matrix[0].length;
    int l = 0, r = n - 1, t = 0, b = m - 1, x = 0;
    int[] res = new int[m * n];
    while (true) {
        for (int i = l; i <= r; i++) {
            res[x++] = matrix[t][i];
        }
        if (++t > b) {
            break;
        }
        for (int i = t; i <= b; i++) {
            res[x++] = matrix[i][r];
        }
        if (l > --r) {
            break;
        }
        for (int i = r; i >= l; i--) {
            res[x++] = matrix[b][i];
        }
        if (t > --b) {
            break;
        }
        for (int i = b; i >= t; i--) {
            res[x++] = matrix[i][l];
        }                     
        if (++l > r) {
            break;
        }   
    }
    return res;
}
```

### 30 包含main函数的栈

#### 解一

另开一个最小栈，用于存放当前栈的最小值，当新push的值比原最小值小时，在最小栈中push新值，否则继续push原来的最小值

```java
class MinStack {

    private Stack<Integer> stack;
    private Stack<Integer> minStack;

    /**
     * initialize your data structure here.
     */
    public MinStack() {
        stack = new Stack<>();
        minStack = new Stack<>();
    }

    public void push(int x) {
        stack.push(x);
        if (minStack.isEmpty()) {
            minStack.push(x);
        } else {
            if (minStack.peek() > x) {
                minStack.push(x);
            } else {
                minStack.push(minStack.peek());
            }
        }
    }

    public void pop() {
        stack.pop();
        minStack.pop();
    }

    public int top() {
        return stack.peek();
    }

    public int min() {
        return minStack.peek();
    }
}
```

### 31 栈的压入、弹出序列

#### 解一

使用辅助栈

```java
public boolean validateStackSequences(int[] pushed, int[] popped) {
    // 辅助栈
    Stack<Integer> stack = new Stack<>();
    int i = 0;
    for (int num : pushed) {
        stack.push(num);
        while (!stack.isEmpty() && stack.peek() == popped[i]) {
            stack.pop();
            i++;
        }
    }
    return stack.isEmpty();
}
```

### 32 从上到下打印二叉树

#### 解一

```java
public int[] levelOrder(TreeNode root) {
    if (root == null) {
        return new int[0];
    }
    List<Integer> res = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            res.add(node.val);
            if (node.left != null) {
                queue.offer(node.left);
            }
            if (node.right != null) {
                queue.offer(node.right);
            }
        }
    }
    int[] ans = new int[res.size()];
    for (int i = 0; i < res.size(); i++) {
        ans[i] = res.get(i);
    }
    return ans;
}
```

### 33 二叉搜索树的后序遍历序列

#### 解一

根据后序遍历“左右根”的顺序，取出根的值，从前往后遍历找到第一个大于根的节点作为左右子树的分界，当右子树的所有制都大于根时，才为二叉搜索树

```java
class Solution {
    public boolean verifyPostorder(int[] postorder) {
        if (postorder.length < 2) {
            return true;
        }
        return helper(postorder, 0, postorder.length - 1);
    }

    private boolean helper(int[] postorder, int start, int end) {
        if (start > end) {
            return true;
        }
        int root = postorder[end];
        int maxIndex = start;
        int i = start;
        while (postorder[i] < root) {
            i++;
        }
        int j = i;
        while (postorder[i] > root) {
            i++;
        }
        boolean left = helper(postorder, start,  j - 1);
        boolean right = helper(postorder, j, end - 1);
        return i == end && left && right;
    }
}
```

### 34 二叉树中和为某一值的路径

#### 解一

回溯遍历

```java
class Solution {
    public List<List<Integer>> pathSum(TreeNode root, int target) {
        List<List<Integer>> res = new ArrayList<>();
        LinkedList<Integer> list = new LinkedList<>();

        path(root, target, res, list);
        return res;
    }

    private void path(TreeNode root, int target, List<List<Integer>> res, LinkedList<Integer> list) {
        if (root == null) {
            return;
        }
        int val = root.val;
        list.add(val);
        if (target - val == 0 && root.left == null && root.right == null) {
            res.add(new ArrayList(list));
        }
        path(root.left, target - val, res, list);
        path(root.right, target - val, res, list);
        list.removeLast();
    }
}
```

### 35 复杂链表的复制

#### 解一

要使用map记录老节点与新节点的对应关系

```java
public Node copyRandomList(Node head) {
    if (head == null) {
        return head;
    } 
    Node cur = head;
    Node res = new Node(0);
    Node p = res;
    Map<Node, Node> map = new HashMap<>();
    while (cur != null) {
        Node newNode = new Node(cur.val);
        p.next = newNode;
        map.put(cur, newNode);
        p = p.next;
        cur = cur.next;
    }
    cur  = head;
    p = res.next;
    while (cur != null) {
        p.random = map.get(cur.random);
        cur = cur.next;
        p = p.next;
    }
    return res.next;
}
```

