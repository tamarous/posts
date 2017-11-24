# LeetCode-70-Climbing Stairs
You are climbing a stair case. It takes n steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

Note: Given n will be a positive integer.

Example 1:

```
Input: 2
Output:  2
Explanation:  There are two ways to climb to the top.

1. 1 step + 1 step
2. 2 steps
```

Example 2:

```
Input: 3
Output:  3
Explanation:  There are three ways to climb to the top.

1. 1 step + 1 step + 1 step
2. 1 step + 2 steps
3. 2 steps + 1 step
```

思路：很基础的动态规划问题，在剑指offer上有原题。
代码：

```
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 1) {
            return 1;
        }
        vector<int> vec(n+1);
        vec[0] = 1; vec[1] = 1;
        for(int i = 2; i <= n; i++) {
            vec[i] = vec[i-2] + vec[i-1];
        }
        return vec[n];
    }
};
```


