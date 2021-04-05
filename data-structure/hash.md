# 哈希表

## 概念

散列表(HashTable)的背后也是分治的思想。其实散列表本身很简单，它对数据的要求仅仅是可以计算特征。

在树型结构中，查找的效率依赖于查找过程中所进行的比较次数。理想的情况是希望不经过任何比较，一次存取便能得到所查记录，那就必须在记录的存储位置和它的关键字之间建立一个确定的对应关系 f, 使每个关键字和结构中一个唯一的存储位置相对应。因而在查找时，只要根据这个对应关系 f 找到给定值 K 的函数值 f(K)。若结构中存在关键字和 K 相等的记录。在此，我们称这个对应关系 f 为哈希 (Hash) 函数，按这个思想建立的表为哈希表。

哈希表使用O(N)空间复杂度存储数据，并且以O(1)的时间复杂度求解问题。

Java 中的 HashSet 用于存储一个集合，可以查找元素是否在集合中。如果元素有穷，并且范围不大，那么可以用一个布尔数组来存储一个元素是否存在。例如对于只有小写字符的元素，就可以用一个长度为 26 的布尔数组来存储一个字符集合，使得空间复杂度降低为 O(1)。

Java 中的 HashMap 主要用于映射关系，从而把两个元素联系起来。HashMap 也可以用来对元素进行计数统计，此时键为元素，值为计数。和 HashSet 类似，如果元素有穷并且范围不大，可以用整型数组来进行统计。在对一个内容进行压缩或者其它转换时，利用 HashMap 可以把原始内容和转换后的内容联系起来。

## 题目解析

### 数组中两个数的和为给定值

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。[LeetCode](https://leetcode-cn.com/problems/two-sum/description/)

nums = [2, 7, 11, 15], target = 9  => [0, 1]

可以先对数组进行排序，然后使用双指针方法或者二分查找方法，这样做时间复杂度为O(NlogN),空间复杂度为O(1)。

用 HashMap 存储数组元素和索引的映射，在访问到 nums[i] 时，判断 HashMap 中是否存在 target - nums[i]，如果存在说明 target - nums[i] 所在的索引和 i 就是要找的两个数。该方法的时间复杂度为 O(N)，空间复杂度为 O(N)，使用空间来换取时间。

```java
public int[] twoSum(int[] nums, int target) {
  HashMap<Integer, Integer> indexForNum = new HashMap<>();
  for(int i = 0; i < nums.length; i++) {
    if(indexForNums.containsKey(target - nums[i])) {
      return new int[]{indexForNum.get(target - nums[i]), i};
    } else {
      indexForNum.put(nums[i], i);
    }
  }
  return null;
}
```

### 判断数组是否含有重复元素

给定一个整数数组，判断是否存在重复元素。
如果存在一值在数组中出现至少两次，函数返回 true 。如果数组中每个元素都不相同，则返回 false 。[LeetCode](https://leetcode-cn.com/problems/contains-duplicate/description/)

[1,2,3,1] => true  [1,2,3,4] => false

```java
public boolean containsDuplicate(int[] nums) {
  Set<Integer> set = new HashSet<>();
  for(int num: nums) {
    set.add(num);
  }
  return set.size() < nums.length;
}
```

### 最长连续序列

给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。 可以设计并实现时间复杂度为 O(n) 的解决方案吗？[LeetCode](https://leetcode-cn.com/problems/longest-consecutive-sequence/description/)

[100,4,200,1,3,2] => 4 ==> [1,2,3,4]
[0,3,7,2,5,8,4,6,0,1] => 9

```java
public int longestConsecutive(int[] nums) {
  Map<Integer, Integer> countForNum = new HashMap<>();
  for(int num : nums) {
    countForNum.put(num, 1);
  }
  for(int num : nums) {
    forward(countForNum, num);
  }
  return maxCount(countForNum);
}

private int forward(Map<Integer, Integer> countForNum, int num) {
  if(!countForNum.containsKey(num)) {
    return 0;
  }
  int cnt = countForNum.get(num);
  if(cnt > 1) {
    return cnt;
  }
  cnt = forward(countForNum, num + 1) + 1;
  countForNum.put(num, cnt);
  return cnt;
}

private int maxCount(Map<Integer, Integer> countForNum) {
  int max = 0;
  for(int num: countForNum.keySet()) {
    max = Math.max(max, countForNum.get(num));
  }
  return max;
}
```