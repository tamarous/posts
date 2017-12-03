# LeetCode-198-House Robber
LeetCode上有一些题目，初看描述时会让人觉得丈二和尚摸不着头脑，绞尽脑汁也想不出解决方法，但看了题解之后会大呼过瘾和巧妙，让人感受到算法的神奇和魅力。今天要说的这个[题目](https://leetcode.com/problems/house-robber/description/)就具有以上这些特点。
题目描述是这样的：街上有一排房子，每个房子里都有一定的钱财，你作为一个歹徒要进行规划来抢到最多的钱，而规则是不能进入相邻的两个屋子，否则就会引来警察导致失败。
本题的讨论区中给出了一个相当简单的解法：

```
class Solution {
public:
    int rob(vector<int>& nums) {
        int rob = 0;
        int notrob = 0;
        for(int i = 0; i < nums.size(); i++) {
            int currob = notrob + nums[i];
            notrob = max(notrob, rob);
            rob = currob;
        }
        return max(rob,notrob);
    }
};
```
下面我就把这个解法给分析一下。
首先定义两个变量来存储最后的结果。在抢劫到第k个房子时，rob表示抢劫了第(k-1)个房子所获得的钱财总数，notrob表示不抢劫第(k-1)个房子获得的钱财总数。对于当前这个房子，只有抢与不抢两种可能，而且**当抢了的时候，第(k-1)个房子就不能抢，当不抢时，第(k-1)个房子可抢可不抢**。
分情况分析：
（1）抢了这个房子的话，那么现在获得的总钱数currob应该是不抢劫第(k-1)个房子获得的钱财总数加上第k个房子的钱财，即`currob = notrob+nums[k]`
（2）不抢这个房子的话，那么不抢第k个房子获得的钱财数应该是不抢第(k-1)个房子的钱财数与抢了第(k-1)个房子的钱财数中的最大值，即`notrob = max(notrob,rob)`
在进行下一次循环前，更新一下rob的值。循环结束后，返回rob和notrob中的最大值，即为抢劫了一条街后获得的最大钱财数。
怎么样，是不是有点豁然开朗了？





