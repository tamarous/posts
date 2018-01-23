# LeetCode-120-Triangle
这道题的题意是：给定一个数组形成的三角形，在从三角形的顶部出发的路径中，求节点数值相加结果为最小的和。

例子：输入

```
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
```
那么从顶部到底部最小的一条路径和为：`2+3+5+1=11`。

思路：这是一道逆序的二维动态规划题，和之前看过的一道题--[《龙与地下城》](http://blog.csdn.net/yu280265067/article/details/50854944)非常相似，都是逆序地从下往上求 DP数组。代码很简单，就不多做分析了，相信大家一看就能看懂。

代码如下：

```
class Solution {
public:
    int minimumTotal(vector<vector<int>>& triangle) {
        for (int i = triangle.size() - 2; i >= 0; i--) {
            for (int j = 0; j < i + 1; j++) {
                triangle[i][j] = min(triangle[i + 1][j], triangle[i + 1][j + 1]) + triangle[i][j];
            }
        }
        return triangle[0][0];
    }
};
```



