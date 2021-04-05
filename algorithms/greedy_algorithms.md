# 贪心算法思想

保证每次操作都是局部最优的，并且最后得到的结果是全局最优的。

## 题目解析

### 数组中两个数的和为给定值

假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。

对每个孩子 i，都有一个胃口值 g[i]，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 j，都有一个尺寸 s[j] 。如果 s[j] >= g[i]，可以将这个饼干 j 分配给孩子 i ，这个孩子会得到满足。目标是尽可能满足越多数量的孩子，并输出这个最大数值。[LeetCode](https://leetcode-cn.com/problems/assign-cookies/description/)

g = [1,2,3], s = [1,1]  ==> 1
有三个孩子和两块小饼干，3个孩子的胃口值分别是：1,2,3。虽然有两块小饼干，由于他们的尺寸都是1，只能让胃口值是1的孩子满足。所以应该输出1。

g = [1,2], s = [1,2,3]  == >2
有两个孩子和三块小饼干，2个孩子的胃口值分别是1,2。拥有的饼干数量和尺寸都足以让所有孩子满足。所以应该输出2.

+ 给一个孩子的饼干应当尽量小并且又能满足该孩子，这样大饼干才能拿来满足度比较大的孩子。
+ 因为满足度最小的孩子最容易得到满足。所以先满足满足度最小的孩子。

在以上的解法中，只在每次分配时饼干时选择一种看起来是当前最优的分配方法。但无法保证这种局部最优的分配方法最后能得到全局最优解。假设能得到全局最优解，并使用反证法进行证明。即假设存在一种比我们使用的贪心策略更优的最优策略。如果不存在这种最优策略，表示贪心策略就是最优策略，得到的解也就是全局最优解。

证明：假设在某次选择中，贪心策略选择给当前满足度最小的孩子分配第 m 个饼干，第 m 个饼干为可以满足该孩子的最小饼干。假设存在一种最优策略，可以给该孩子分配第 n 个饼干，并且 m < n。我们可以发现，经过这一轮分配，贪心策略分配后剩下的饼干一定有一个比最优策略来得大。因此在后续的分配中，贪心策略一定能满足更多的孩子。也就是说不存在比贪心策略更优的策略，即贪心策略就是最优策略。

```java
public int findContentChildren(int[] grid, int[] size) {
  if(grid == null || size == null) {
    return 0;
  }
  Arrays.sort(grid);
  Arrays.sort(size);
  int gi = 0, si = 0;
  while(gi < grid.length && si < size.length) {
    if(grid[gi] <= size[si]) {
      gi++;
    }
    si++;
  }
  return gi;
}
```

### 无重叠区间个数

给定一个区间的集合，找到需要移除区间的最小数量，使剩余区间互不重叠。[LeetCode](https://leetcode-cn.com/problems/non-overlapping-intervals/description/)

[[1,2], [2,3], [3,4], [1,3]] => 1 ==> [1,3]
[[1,2], [1,2], [1,2]] => 2 ==> [1,2], [1,2]
[[1,2], [2,3]] => 0

+ 计算让一组区间不重叠所需要移除的区间个数。
+ 先计算最多能组成的不重叠区间个数，然后用区间总个数减去不重叠区间的个数。
+ 在每次选择中，区间的结尾最为重要，选择的区间结尾越小，留给后面的区间的空间越大，那么后面能够选择的区间个数也就越大。
+ 按区间的结尾进行排序，每次选择结尾最小，并且和前一个区间不重叠的区间。

```java
public int eraseOverlapIntervals(int[][] intervals) {
  if(intervals.length == 0) {
    return 0;
  }
  Arrays.sort(intervals, Comparator.comparingInt(o -> o[1]));
  int cnt = 1;
  int end = intervals[0][1];
  for(int i = 1; i < intervals.length; i++) {
    if(intervals[i][0] < end) {
      continue;
    }
    end = intervals[i][1];
    cnt++;
  }
  return intervals.length - cnt;
}
```

### 分隔字符串使同种字符出现在一起

字符串 S 由小写字母组成。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。返回一个表示每个字符串片段的长度的列表。[LeetCode](https://leetcode-cn.com/problems/partition-labels/description/)

S = "ababcbacadefegdehijhklij" => [9,7,8]
划分结果为 "ababcbaca", "defegde", "hijhklij"。
每个字母最多出现在一个片段中。
像 "ababcbacadefegde", "hijhklij" 的划分是错误的，因为划分的片段数较少。

```java
public List<Integer> partitionLabels(String S) {
  int[] lastIndexsOfChar = new int[26];
  for(int i = 0; i < S.length(); i++) {
    lastIndexsOfChar[char2Index(S.charAt(i))] = i;
  }
  List<Integer> partitions = new ArrayList<>();
  int firstIndex = 0;
  while(firstIndex < S.length()) {
    int lastIndex = firstIndex;
    for(int i = firstIndex; i < S.length() && i <= lastIndex; i++) {
      int index = lastIndexOfChar[char2Index(S.charAt(i))];
      if(index > lastIndex) {
        lastIndex = index;
      }
    }
    partitions.add(lastIndex - firstIndex + 1);
    firstIndex = lastIndex + 1;
  }
  return partitions;
}

private int char2Index(char c) {
  return c - 'a';
}
```