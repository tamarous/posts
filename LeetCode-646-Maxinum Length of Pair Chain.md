# LeetCode-646-Maxinum Length of Pair Chain
题意：给出 n 组数，每组有两个数，第一个数总是比第二个数字小。如果两组数 (a,b) 和 (c,d) 之间，有 b < c，那么就称 (a,b)和 (c,d) 之间形成了一个chain。求出这 n 组数中能形成的最长的 chain 的长度。

例子：输入`[[1,2], [2,3], [3,4]]`，因为`[1,2] -> [3,4]`，所以返回2。

思路：又是一道动态规划题目。因为第一个数字总是小于第二个数字，所以我们可以先将这些组按第一个数字来排序。然后我们声明一个长度为n 的数组，`dp[i]` 表示以 `pairs[i]` 作为 chain 的最后一个元素的情况下，能形成的 chain 的最大长度。于是对于 `pairs[i]`，在索引范围在 `0~i-1` 的所有 pair 中，如果 `pairs[j]` 满足 `pairs[j][1] < pairs[i][0]`，那么 `pairs[i]` 就可以连到 `pairs[j]` 上；如果 `pairs[j']` 是满足条件的 j 中 `pairs[j][1]` 最大的，那么这样形成的 chain 就是最长的。因此我们可以得到状态转移方程为：`dp[i] = {max(dp[i], dp[j]+1), j >= 0 && j < i, pairs[j][0] < pairs[i][0]}`。

在排序时，我们可以使用 `std::sort` 并传入自定义的比较函数，配合 lambda 语法可以使得排序非常简单。

代码如下：

```
class Solution {
public:
    int findLongestChain(vector<vector<int>>& pairs) {
        if (pairs.size() == 0 || pairs[0].size() < 2) {
            return 0;
        }
        
        // 将 pairs 按照第一个数字的大小顺序升序排列
        std::sort(pairs.begin(), pairs.end(), [](auto &left, auto &right){
            return left[0] < right[0];
        });
        int size = pairs.size();
        vector<int> dp(size,1);
        for (int i = 1; i < size; i++) {
            for (int j = 0; j < i; j++) {
                if (pairs[j][1] < pairs[i][0]) {
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
        }
        int len = 0;
        for (int i = 0; i < size; i++) {
            if (dp[i] > len) {
                len = dp[i];
            }
        }
        return len;
    }
};
```


