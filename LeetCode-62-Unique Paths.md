# LeetCode-62-Unique Paths
[这道题](https://leetcode.com/problems/unique-paths/description/)是一道典型的动态规划问题。如图：
![Robot](https://leetcode.com/static/images/problemset/robot_maze.png)

一个机器人的起始位置是左上方的网格，每次只能往下或者往右走。网格的长和高分别是 m 和 n，求机器人要走到图中的五角星处共有多少种走法？

思路：生成一个大小为 m*n 的二维数组，记为 dp，dp[i][j]表示走到图中第 i 行第 j 列个网格点共有多少种走法。首先考虑第一行。由于机器人只能往下和往右走，因此位于第一行的网格点只有一直往右这一种走法，因此 `dp[0][j] = 1`。同理位于第一列的网格点只有一直往下这一种走法，因此`dp[i][0] = 1`。

现在考虑第 i 行第 j 列的网格点。机器人要走到位于第 i 行第 j 列的网格点，只可能从它的左边节点[i][j-1]或是从它的上边的节点[i-1][j]移动一步到达，因此 `dp[i][j] = dp[i-1][j] + dp[i][j-1]`。这就是这道题的状态转移方程。根据该方程我们就能算出最后一个网格点 `dp[m-1][n-1]`的值，也就是我们要求的结果。

代码如下：

```
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int> > dp(m,vector<int>(n));
        dp[0][0] = 1;
        for(int i = 1; i < m;i++) {
            dp[i][0] = 1;
        }
        for(int i = 1; i < n; i++) {
            dp[0][i] = 1;
        }
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
};
```





