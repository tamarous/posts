# LeetCode-494-Target Sum
[这个题](https://leetcode.com/problems/target-sum/description/)也是一道动态规划题目。题意是：给出一系列数和一个目标数，使用`+`和`-`来使这些数字的和等于给定的目标数，求总共可行的方法数。

例子：输入`[1, 1, 1, 1, 1]`，`target = 3`，那么：

```
-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3
```
所以最后的输出结果是：5

思路：
这个题比较类似于之前做过的求[`二叉树上和为指定数值的路径`](http://www.tamarous.com/2017/11/26/path-sum/)。我们可以写一个辅助函数 

```
int process(vector<int> &nums, int target, int left, int right);
```
这个函数计算：以 nums 的位于 left 和 right 之间的元素为输入，使用`+`和`-`使得这些数字之和等于 target 的方法数。声明一个 int 型变量 res，初始时 `res = 0`，表示总的方法数。对于当前区间`[i,j)`来说，如果我们对第一个数 `nums[i]`使用`+`，那么剩余的目标数为 `target - nums[i]`，如果我们对第一个数 `nums[i]`使用`-`，那么剩余的目标数为 `target+nums[i]`，所以有：

```
res = process(nums,target-nums[i],left+1, right) + process(nums,target+nums[i],left+1,right)
```

如果`left > right`, 此时如果 `target = 0`的话，那么这种方法就可以使得和为 target，返回1，否则就返回0。

根据以上分析，不难写出如下的代码：

```
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int size = nums.size();
        if (size == 0) {
            return 0;
        }
        return process(nums, target, 0, size);
    }
    int process(vector<int> &nums, int target, int left, int right) {
        int size = nums.size();
        if (left >= size) {
            if (target == 0) {
                return 1;
            } else {
                return 0;
            }
        }
        int res = 0;
        int cur = nums[i];
        res = process(nums, target - nums[left], left + 1, right) + 
        process(nums, target + nums[left], left + 1, right);
        return res;
    }
};
```



