# LeetCode-63-Unique Paths Ⅱ
[这道题](https://leetcode.com/problems/unique-paths-ii/description/)是[Unique Path](http://www.tamarous.com/2018/01/09/unique-paths/)的续集。

题意：输入一个二维数组，这个二维数组的元素如果是0，表示对应网格处没有障碍，可以正常向下或向右移动；如果这个数字是1，表示对应网格处有个障碍，机器人🤖将不能通过。

思路：
（1）首先我们考虑一下特殊情况。如果第一个网格或最后一个网格对应的数字为1，那么意味着是不可达的，因此直接返回0即可。
（2）排除掉这两种情况后，我们还是先生成一个二维数组 dp，dp[i][j]表示走到图中第 i 行第 j 列个网格点共有多少种走法。首先考虑第一行。如果某个位置的 `obstacle[0][j] = 1`，那么这个网格点以及它右面和它下面的其他网格点都无法到达，因此我们需要一个标记 flag 来表示是否遇到了 obstacle = 1的情况。如果 `flag = true`，那么`dp[0][j] = 1`。只有当 `flag != true && obstacle[0][j] != 1` 时，有 `dp[0][j] = 1`。第一列也是类似的。
（3）然后考虑第 i 行第 j 列的网格点。对于 dp[i][j] 来说，如果 `obstacle[i][j] = 0`，那么 `dp[i][j] = dp[i-1][j] + dp[i][j-1]`；如果`obstacle[i][j] = 1`，那么 `dp[i][j] = 0`。这就是该题的状态转移方程。

代码如下：

```
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();
        
        vector<vector<int> > dp(m,vector<int>(n));
        bool flag = false;
        if(obstacleGrid[0][0] == 1) {
            return 0;
        }
        for(int i = 0; i < m; i++) {
            if (obstacleGrid[i][0] != 1 && !flag) {
                dp[i][0] = 1;
            } else {
                if (obstacleGrid[i][0] == 1) {
                    flag = true;
                    dp[i][0] = 0;
                } else if (flag) {
                    dp[i][1] = 0;
                }
            }
        }
        flag = false;
        for(int i = 1; i < n; i++) {
            if (obstacleGrid[0][i] != 1 && !flag) {
                dp[0][i] = 1;
            } else {
                if (obstacleGrid[0][i] == 1) {
                    flag = true;
                    dp[0][i] = 0;
                } else if (flag) {
                    dp[0][i] = 0;
                }
            }
        }
        
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                dp[i][j] = obstacleGrid[i][j] != 1 ? dp[i-1][j] + dp[i][j-1] : 0;
            }
        }
        return dp[m-1][n-1] = obstacleGrid[m-1][n-1] != 1?dp[m-1][n-1]:0;
    }
};
```






