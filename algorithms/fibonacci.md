# 斐波那契数列


## 题目描述

求斐波那契数列的第n项， n <= 39.

f(n) = 0, n = 0;
f(n) = 1, n = 1;
f(n) = f(n - 1) + f(n - 2) n > 1;

## 解题思路

如果使用递归求解，会重复计算一些子问题，例如f(4)需要计算f(3)和f(2), 计算f(3)需要计算f(2)和f(1),那么f(2)被重复计算了。

递归是将一个问题划分成多个子问题求解，动态规划也是如此，但是动态规划会把子问题的解缓存起来，从而避免重复求解子问题。

```java
public int Fibonacci(int n) {
  if(n <= 1) {
    return n;
  }
  int[] fib = new int[n + 1];
  fib[1] = 1;
  for(int i = 2; i <= n; i++) {
    fib[i] = fib[i - 1] + fib[i - 2];
  }
  return fib[n];
}
``` 

考虑到第i项只与第i - 1项和 第i - 2项有关，因此只需要存储前两项的值就能求解第i项，从而将空间复杂度由O(N)降低到O(1).

```java
public int Fibonacci(int n) {
  if(n <= 1) {
    return n;
  }
  int pre2 = 0, pre1 = 1;
  int fib = 0;
  for(int i = 2; i <= n; i++) {
    fib = pre2 + pre1;
    pre2 = pre1;
    pre1 = fib;
  }
  return fib;
}
```

由于待求解的n小于40，因此可以将前40项的结果先进行计算，之后就能以O(1)时间复杂度得到第n项的值。

```java
public class Solution {
  private int[] fib = new int[40];
  public Solution() {
    fib[1] = 1;
    for(int i = 2; i < fib.length; i++) {
      fib[i] = fib[i - 1] + fib[n - 2];
    }
  }
  public int Fibonacci(int n) {
    return fib[n];
  }
}
```