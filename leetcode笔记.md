### 类型的概念

#### 链表

递归时，**不要头脑压栈进入递归函数里，而是理解递归函数的定义**，比如反转函数：

`reverse`:**输入一个head，将这个head为首的链表翻转，并返回翻转之后的头节点。**

以反转整个链表为例：

```js
// reveseList是反转传入的链表
var reverseList = function (head) {
  // 递归出口
  if (head === null || head.next === null) return head;
  // 直接假设得到了翻转的结果
  const last = reverseList(head.next);
  // 先把head指向翻转的结果
  head.next.next = head;
  // 把当前和head.next断开
  head.next = null;
  return last;
};
```

![image-20210304175127187](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20210304175127187.png)

![image-20210304175158279](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20210304175158279.png)

注意点

1. 递归要有`base case`（出口）
2. 当链表反转后，新的头节点是`last`，而`head.next`是尾。如上图。所以`head.next.next`才是获取传入的`head.next`，而不是`last.next`

---

#### 二叉树

二叉树遍历方式，注释部分为对应方式书写代码的：

```java
/* 二叉树遍历框架 */
void traverse(TreeNode root) {
    // 前序遍历
    traverse(root.left)
    // 中序遍历
    traverse(root.right)
    // 后序遍历
}
```

##### 递归

**写递归算法的关键是要明确函数的「定义」是什么，然后相信这个定义，利用这个定义推导最终结果，绝不要跳入递归的细节**。

**写树相关的算法，先搞清楚当前 `root` 节点该做什么，然后根据函数定义递归调用子节点**

---

#### 回溯算法

**解决一个回溯问题，实际上就是一个决策树的遍历过程**。你需要思考 3 个问题：

1、路径：也就是已经做出的选择。

2、选择列表：也就是你当前可以做的选择。

3、结束条件：也就是到达决策树底层，无法再做选择的条件。

回溯算法的框架：

```python
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

**其核心就是 for 循环里面的递归，在递归调用之前「做选择」，在递归调用之后「撤销选择」**

<img src="https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20210310113601498.png" alt="image-20210310113601498" style="zoom: 25%;" />

多叉树的遍历框架就是这样：

```java
void traverse(TreeNode root) {
    for (TreeNode child : root.childern)
        // 前序遍历需要的操作
        traverse(child);
        // 后序遍历需要的操作
}
```



前序遍历和后序遍历，他们只是两个很有用的时间点,**前序遍历的代码在进入某一个节点之前的那个时间点执行，后序遍历代码在离开某个节点之后的那个时间点执行**。

![image-20210310113711808](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20210310113711808.png)

![image-20210310113844720](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20210310113844720.png)

```python
for 选择 in 选择列表:
    # 做选择
    将该选择从选择列表移除
    路径.add(选择)
    backtrack(路径, 选择列表)
    # 撤销选择
    路径.remove(选择)
    将该选择再加入选择列表
```

**我们只要在递归之前做出选择，在递归之后撤销刚才的选择**，就能正确得到每个节点的选择列表和路径。

---

#### 动态规划

**动态规划问题的一般形式就是求最值**。比如求**最长**递增子序列呀，**最小**编辑距离呀等等。

**求解动态规划的核心问题是穷举**。因为要求最值，肯定要把所有可行的答案穷举出来，然后在其中找最值呗。

特点：

**存在「重叠子问题」**

一定会**具备「最优子结构」**

**正确的「状态转移方程」**

当前状态和上一个的状态相关

解法：

1、**确定 base case, 找到状态不断能靠近这个base case的dp**

2、**确定「状态」，也就是原问题和子问题中会变化的变量**

3、**确定「选择」，也就是导致「状态」产生变化的行为**

4、**明确 `dp` 函数/数组的定义**。

```python
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
       # dp函数
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```

---

#### 滑动窗口

```cpp
int left = 0, right = 0;

while (right < s.size()) {`
    // 增大窗口
    window.add(s[right]);
    right++;

    while (window needs shrink) {
        // 缩小窗口
        window.remove(s[left]);
        left++;
    }
}
```

```cpp
/* 滑动窗口算法框架 */
void slidingWindow(string s, string t) {
    connst need = {}, window = {};
    for (char c : t) need[c]++;

    int left = 0, right = 0;
    int valid = 0; 
    while (right < s.size()) {
        // c 是将移入窗口的字符
        char c = s[right];
        // 右移窗口
        right++;
        // 进行窗口内数据的一系列更新
        ...

        /*** debug 输出的位置 ***/
        printf("window: [%d, %d)\n", left, right);
        /********************/

        // 判断左侧窗口是否要收缩
        while (window needs shrink) {
            // d 是将移出窗口的字符
            char d = s[left];
            // 左移窗口
            left++;
            // 进行窗口内数据的一系列更新
            ...
        }
    }
}
```

**滑动窗口算法的思路是这样**：

1、我们在字符串 `S` 中使用双指针中的左右指针技巧，初始化 `left = right = 0`，把索引**左闭右开**区间 `[left, right)` 称为一个「窗口」。

2、我们先不断地增加 `right` 指针扩大窗口 `[left, right)`，直到窗口中的字符串符合要求（包含了 `T` 中的所有字符）。

3、此时，我们停止增加 `right`，转而不断增加 `left` 指针缩小窗口 `[left, right)`，直到窗口中的字符串不再符合要求（不包含 `T` 中的所有字符了）。同时，每次增加 `left`，我们都要更新一轮结果。

4、重复第 2 和第 3 步，直到 `right` 到达字符串 `S` 的尽头。

**第 2 步相当于在寻找一个「可行解」，然后第 3 步在优化这个「可行解」，最终找到最优解，**也就是最短的覆盖子串。左右指针轮流前进，窗口大小增增减减，窗口不断向右滑动，这就是「滑动窗口」这个名字的来历。

---

### Tag

---

#### Array

**1. 最大子序和(Easy)**

1. 两层暴力便利
2. 动态规划

动态规划

1. 确定base case：Dp(0) = nums[0]
2. 确定状态：num
3. 确定选择：for const num of nums
4. 明确dp：dp(i) = max(dp(i - 1) + nums[i] / dp(i - 1), nums[i])

**2. 最大回文字串**

1. 两层暴力便利
2. 动态规划

动态规划

1. 确定状态：`dp[i][j]`: 字符串i到j是否回文。

1. 转移方程：dp[i]\[j] = dp[i+1]\[j-1] && s[i] === s[j]
   1. Dp[i+1]\[j-1] 代表上一个字串是回文的
   2. s[i] === s[j]代表当前是回文的
2. 特殊：j - i < 2，数量小于2，长度为0/1

```js
var longestPalindrome = function (s) {
  let n = s.length;
  let res = "";
  let dp = Array.from(new Array(n), () => new Array(n).fill(0));
  for (let i = n - 1; i >= 0; i--) {
    for (let j = i; j < n; j++) {
      dp[i][j] = s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1]);
      if (dp[i][j] && j - i + 1 > res.length) {
        res = s.substring(i, j + 1);
      }
    }
  }
  return res;
};
```



**3. 爬楼梯**

动态规划

1. 确定状态：dp[n]: 到当前层楼梯有多少种方法
2. 转移方程：dp[n] = dp[n - 1] + dp[n - 2]
   1. dp[n - 1]代表上一次是一步过来的，n-2代表两步
3. 特殊：n === 1 || n === 2

```js
var climbStairs = function (n) {
  let dp = Array.of(n);
  for (let i = 0; i <= n; i++) {
    dp[i] = i === 1 ? 1 : i === 2 ? 2 : dp[i - 1] + dp[i - 2];
  }
  return dp[n];
};
```



**4. 零钱兑换**

动态规划

1. Base case: amount 为 0，此时不需要硬币
2. 确定状态：amount，和楼梯阶层一个意思,amount是唯一不断靠近base case的变量
3. 转移方程：dp[n] = min(res, dp[n - count] + 1),dp[n]为当前amount时，所需最小硬币数量，res存储着，三种硬币对应的状态，最后取得最少的那个。res类似存储着两步，一步跳楼梯时的次数。

```js
// 暴力解，部分超时O(n^k)
var coinChange = function (coins, amount) {
  const dp = (n) => {
    if (n === 0) return 0;
    if (n < 0) return -1;
    let res = Infinity;
    for (const coin of coins) {
      pervCount = dp(n - coin);
      if (pervCount === -1) continue;
      res = Math.min(res, pervCount + 1);
    }
    return res !== Infinity ? res : -1;
  };
  return dp(amount);
};
```

```js
// 哈希记录，dp结果；O(nk)
var coinChange = function (coins, amount) {
  const hasMap = {};
  const dp = (n) => {
    if (hasMap[n]) return hasMap[n];
    if (n === 0) return 0;
    if (n < 0) return -1;
    let res = Infinity;
    for (const coin of coins) {
      pervCount = dp(n - coin);

      if (pervCount === -1) continue;
      res = Math.min(res, pervCount + 1);
    }
    hasMap[n] = res !== Infinity ? res : -1;
    return hasMap[n];
  };
  return dp(amount);
};
```

#### 链表

1. 翻转整个链表

   1. 迭代
   2. 递归

   递归：

   ```js
   var reverseList = function (head) {
     if (head === null || head.next === null) return head;
     const last = reverseList(head.next);
     head.next = head;
     head.next = null;
     return last;
   };
   ```

2. 翻转整个链表
   1. 迭代
   2. 递归

递归：

```js
var reverseList = function (head) {
  if (head === null || head.next === null) return head;
  const last = reverseList(head.next);
  head.next = head;
  head.next = null;
  return last;
};
```

