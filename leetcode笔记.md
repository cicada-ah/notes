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

