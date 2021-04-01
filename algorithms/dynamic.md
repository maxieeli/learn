# 动态规划算法

## 定义

动态规划是将较大的问题分解成较小问题，对这些较小的问题并不是对原问题明晰的分割，其中一部分是被重复求解的。动态规划其实是运筹学的一种最优化方法。

## 常见解决形式

首先动态规划问题的一般形式为求最值，例如最长递增子序列，最小编辑距离等。核心问题在于穷举，但是该穷举有些区别，因为该类存在 '重叠子问题', 如果单纯穷举效率会极其低下，所以需要 '备忘录' 或 'table'优化穷举，避免不必要的计算。而且动态规划问题一定具备 '最优子结构', 才能通过子问题的最值得到原问题的最值。

### 1.题目实例

给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。可以认为每种硬币的数量是无限的。

input: coins = [1, 2, 5], amount = 11
output: 3(11 = 5 + 5 + 1)

input: coins = [2], amount = 3
output: -1

input: coins = [1], amount = 0
output: 0

input: coins = [1], amount = 1
output: 1

input: coins = [1], amount = 2
output: 2

#### 解决思路

递归问题 --> 含有重叠子问题(操作重复) --> 记忆化搜索(自顶向下) --> 动态规划(自底向上)

#### 递归

```java
class Solution {
  int res = Integer.MAX_VALUE;
  public int coinChange(int[] coins, int amount) {
     if(coins.length == 0) {
        return -1;
     }
     findWay(coins, amount, 0);
     // 若没有任何一种硬币组合能组成总金额，返回-1
     if(res == Integer.MAX_VALUE) {
        return -1;
     }
     return res;
  }
  public void findWay(int[] coins, int amount, int count) {
     if(amount < 0) {
        return;
     }
     if(amount == 0) {
        res = Math.min(res, count);
     }
     for(int i = 0; i < coins.length; i++) {
        findWay(coins, amount - coins[i], count + 1);
     }
  }
}
```

#### 记忆化搜索

在进行递归时，有很多重复节点要进行操作。这样会浪费一些时间。可以使用数组memo[] 来保存节点的值。
在进行递归时，memo[n]被复制了，就不用继续递归，直接调用。

```java
class Solution {
   int[] memo;
   public int coinChange(int[] coins, int amount) {
      if(coins.length == 0) {
         return -1;
      }
      memo = new int[amount];
      return findWay(coins, amount);
   }
   // memo[n]表示钱币n可以被换取的最少硬币数，不能换取为-1
   // findWay函数目的是找到amount数量的零钱可以兑换的最少硬币数量，返回值int
   public int findWay(int[] coins, int amount) {
      if(amount < 0) {
         return -1;
      }
      if(amount == 0) {
         return 0;
      }
      // 记忆化的处理，memo[n]赋予值，就不用继续下面的循环
      // 直接返回memo[n]的最优值
      if(memo[amount - 1] != 0) {
         return memo[amount - 1];
      }
      int min = Integer.MAX_VALUE;
      for(int i = 0; i < coins.length; i++) {
         int res = findWay(coins, amount - coins[i]);
         if(res > 0 && res < min) {
            min = res + 1;
         }
      }
      memo[amount - 1] = (min == Integer.MAX_VALUE ? -1 : min);
      return memo[amount - 1];
   }
}
```

#### 动态规划

上面的记忆化搜索是从memo[amount - 1]开始，从上到下
而动态规划从memo[0]开始，从下到上

```java
class Solution {
   public int coinChange(int[] coins, int amount) {
      // 自底向上的动态规划
      if(coins.length == 0) {
         return -1;
      }
      // memo[n]的值，表示凑成总金额为n所需最少硬币个数
      int[] memo = new int[amount + 1];
      memo[0] = 0;
      for(int i = 1; i <= amount; i++) {
         int min = Integer.MAX_VALUE;
         for(int j = 0; j < coins.length; j++) {
            if(i - coins[j] >= 0 && memo[i - coins[j]] < min) {
               min = memo[i - coins[j]] + 1;
            }
         }
         memo[i] = min;
      }
      return memo[amount] == Integer.MAX_VALUE ? -1 : memo[amount];
   }
}
```

另一种实现

memo[i]有两种实现的方式，取两者的最小值。

+ 包含当前的coins[i], 那么剩余钱就是i - coins[i]， 这种操作要兑换的硬币数是memo[i - coins[j]] + 1
+ 不包含, 要兑换的硬币数是memo[i]

```java
class Solution {
   public int coinChange(int[] coins, int amount) {
      // 自底向上的动态规划
      if(coins.length == 0) {
         return -1;
      }
      // memo[n]的值：表示为凑成总金额为n所需的最少硬币个数
      int[] memo = new int[amount + 1];
      // 给memo赋予值，最多的硬币数就是全部使用面值1的硬币进行交换
      // amount + 1是不可能达到的换取数量，于是填充
      Arrays.fill(memo, amount + 1);
      memo[0] = 0;
      for(int i = 1; i <= amount; i++) {
         for(int j = 0; j < coins.length; j++) {
            if(i - coins[j] >= 0) {
               // 上述两种情况
               memo[i] = Math.min(memo[i], memo[i - coins[j]] + 1);
            }
         }
      }
   }
}
```

### 2.题目实例

给定一个非负整数数组，a1, a2, ..., an, 和一个目标数，S。现在有两个符号 + 和 -。对于数组中的任意一个整数，都可以从 + 或 -中选择一个符号添加在前面。返回可以使最终数组和为目标数 S 的所有添加符号的方法数。

输入：nums: [1, 1, 1, 1, 1], S: 3
输出：5
解释：
-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3
一共有5种方法让最终目标和为3。

### 解决思路

+ 回溯算法,利用回溯穷举所有结果，计数有多少种组合能计算出目标值

```java
class Solution {
   public int result = 0;
   public int findTargetSumWays(int[] nums, int target) {
      if(nums.length == 0) {
         return 0;
      }
      backTrack(nums, 0, target);
      return result;
   }
   public void backTrack(int[] nums, int i, int rest) {
      if(i == nums.length) {
         if(rest == 0) {
            result++;
         }
         return;
      }
      // 给nums[i]选择减号
      rest += nums[i];
      // 穷举nums[i + 1]
      backTrack(nums, i + 1, rest);
      // 撤销选择
      rest -= nums[i];
      // 给nums[i]选择加号
      rest -= nums[i];
      // 穷举nums[i + 1]
      backTrack(nums, i + 1, rest);
      // 撤销选择
      rest += nums[i];
   }
}
```

以上回溯算法可以解决该问题，时间复杂度为O(2^N)，N为数组大小，该算法就是根据二叉树的遍历。其中树的高度就是nums的长度。
所以时间复杂度就是二叉树的节点数。

+ 消除重叠子问题

如何发现重叠子问题，方式就是看是否可能出现重复的'状态',对于递归函数来说，函数参数中会变的参数就是'状态'.
对于backTrack函数来说，会变的参数就是 i 和 rest。这就是重叠子问题。只要能找到一个，那一定存在着很多的重叠子问题。

```java
class Solution {
   public int findTargetSumWays(int[] nums, int target) {
      if(nums.length == 0) {
         return 0;
      }
      return dp(nums, 0, target);
   }
   // 制作备忘录
   HashMap<String, Integer> memo = new HashMap<>();
   int dp(int[] nums, int i, int rest) {
      if(i == nums.length) {
         if(rest == 0) {
            return 1;
         }
         return 0;
      }
      // 转成字符串作为哈希表的键
      String key = i + "," + rest;
      // 避免重复计算
      if(memo.containsKey(key)) {
         return memo.get(key);
      }
      // 穷举
      int result = dp(nums, i + 1, rest - nums[i]) +
                  dp(nums, i + 1, rest + nums[i]);
      // 记录在备忘录
      memo.put(key, result);
      return result;
   }
}
```

+ 动态规划

该问题可以转化为一个子集划分问题，而子集划分问题又是一个[背包问题](https://labuladong.gitbook.io/algo/dong-tai-gui-hua-xi-lie/bei-bao-lei-xing-wen-ti/bei-bao-wen-ti)。
首先，如果把nums划分成两个子集 A 和 B，分别代表分配 + 和 - 的数，那么它们和target存在以下关系

```javascript
sum(A) - sum(B) = target
sum(A) = target + sum(B)
sum(A) + sum(A) = target + sum(B) + sum(A)
2 * sum(A) = target + sum(nums)
```

可以推出 sum(A) = (target + sum(nums)) / 2，也就是把问题转化成: nums中存在几个子集A，使得A中元素的和为
(target + sum(nums)) / 2，根据以上结论可以得出一个处理动态规划问题的基本流程：

<strong>第一步明确两点, '状态'和'选择'</strong>.
   对于背包问题，这个都是一样的，状态就是「背包的容量」和「可选择的物品」，选择就是「装进背包」或者「不装进背包」。

<strong>第二步明确dp数组的定义</strong>.
   按照背包问题的套路，可以给出如下定义：
   dp[i][j] = x 表示，若只在前 i 个物品中选择，若当前背包的容量为 j，则最多有 x 种方法可以恰好装满背包。
   翻译成探讨的子集问题就是，若只在 nums 的前 i 个元素中选择，若目标和为 j，则最多有 x 种方法划分子集。
   根据这个定义，显然 dp[0][..] = 0，因为没有物品的话，根本没办法装背包；dp[..][0] = 1，因为如果背包的最大载重为 0，「什么都不装」就是唯一的一种装法。
   所求的答案就是 dp[N][sum]，即使用所有 N 个物品，有几种方法可以装满容量为 sum 的背包。

<strong>根据'选择'，思考状态转移逻辑</strong>
   回想刚才的 dp 数组含义，可以根据「选择」对 dp[i][j] 得到以下状态转移：
   如果不把 nums[i] 算入子集，或者说不把这第 i 个物品装入背包，那么恰好装满背包的方法数就取决于上一个状态 dp[i-1][j]，继承之前的结果。
   如果把 nums[i] 算入子集，或者说把这第 i 个物品装入了背包，那么只要看前 i - 1 个物品有几种方法可以装满 j - nums[i-1] 的重量就行了，所以取决于状态 dp[i-1][j-nums[i-1]]。
   PS：注意这里说的 i 是从 1 开始算的，而数组 nums 的索引时从 0 开始算的，所以 nums[i-1] 代表的是第 i 个物品的重量，j - nums[i-1] 就是背包装入物品 i 之后还剩下的容量。
   由于 dp[i][j] 为装满背包的总方法数，所以应该以上两种选择的结果求和，得到状态转移方程：

```javascript
dp[i][j] = dp[i - 1][j] + dp[i - 1][j - nums[i  - 1]]
```

根据状态转移方程写出动态规划算法：

```java
class Solution {
   public int subsets(int[] nums, int sum) {
      int n = nums.length;
      int[][] dp = new int[n + 1][sum + 1];
      for(int i = 0; i <= n; i++) {
         dp[i][0] = 1;
      }
      for(int i = 1; i <= n; i++) {
         for(int j = 0; j <= sum; j++) {
            if(j >= nums[i - 1]) {
               dp[i][j] = dp[i - 1][j] + dp[i - 1][j - nums[i - 1]];
            } else {
               dp[i][j] = dp[i - 1][j];
            }
         }
      }
      return dp[n][sum];
   }
}
```

然后，发现dp[i][j]只和前一行dp[i - 1][...]有关，可以优化成一维dp:

```java
class Solution {
   public int subsets(int[] nums, int sum) {
      int n = nums.length;
      int[] dp = new int[sum + 1];
      dp[0] = 1;
      for(int i = 1; i <= n; i++) {
         for(int j = sum; j >= 0; j--) {
            if(j >= nums[i - 1]) {
               dp[j] = dp[j] + dp[j - nums[i - 1]];
            } else {
               dp[j] = dp[j];
            }
         }
      }
      return dp[sum];
   }
}
```

<strong>对照二维 dp，只要把 dp 数组的第一个维度全都去掉就行了，唯一的区别就是这里的 j 要从后往前遍历，原因如下：</strong>

因为二维压缩到一维的根本原理是，dp[j] 和 dp[j-nums[i-1]] 还没被新结果覆盖的时候，相当于二维 dp 中的 dp[i-1][j] 和 dp[i-1][j-nums[i-1]]。

那么，就要做到：<strong>在计算新的 dp[j] 的时候，dp[j] 和 dp[j-nums[i-1]] 还是上一轮外层 for 循环的结果。</strong>

如果从前往后遍历一维 dp 数组，dp[j] 显然是没问题的，但是 dp[j-nums[i-1]] 已经不是上一轮外层 for 循环的结果了，这里就会使用错误的状态，当然得不到正确的答案。

-- END--

参考：

[494.目标和(leetcode)](https://leetcode-cn.com/problems/target-sum/)

[518. 零钱兑换 II(leetcode)](https://leetcode-cn.com/problems/coin-change-2/)

[动态规划和回溯算法](https://labuladong.gitbook.io/algo/dong-tai-gui-hua-xi-lie/dong-tai-gui-hua-ji-ben-ji-qiao/targetsum)
