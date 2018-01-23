# LeetCode-437-Path Sum Ⅲ
在 [Path Sum](http://www.tamarous.com/2017/11/26/path-sum/) 和 [Path Sum Ⅱ](http://www.tamarous.com/2017/11/26/path-sum-%e2%85%a1/)之后，Path Sum 大家庭又迎来了第三位成员。那么这个[新成员](https://leetcode.com/problems/path-sum-iii/description/)又提出了什么样的要求呢？

题意是这样的：给出一棵二叉树和一个数值 sum，计算这棵树上的节点之和等于sum 的所有路径的条数。和之前不同的是，这道题中路径可以从树中的任何一个节点开始。

思路：二叉树的题目，大部分都可以用递归的思想来解决，这道题也不例外。基本思路和之前类似，可以参照前面的两篇博文。但是要注意的是，由于路径可以从任何地方开始，在任何地方结束，因此：root 处存在和为 sum 的路径，root->left 处存在和为 sum 的路径以及 root->right 存在和为 sum 的路径这三种情况是需要累加到一起的。我们可以写一个辅助函数`int rootSum(TreeNode *root, int sum)` 来计算从某个节点 node 处开始和为 sum 的路径条数，那么最后的总条数为 `rootSum(root,sum) + pathSum(root->left, sum) + pathSum(root->right, sum)`。

代码如下：

```
class Solution {
public:
    int rootSum(TreeNode *root, int sum) {
        if (root == NULL) {
            return 0;
        }
        if (sum == root->val) {
            return 1 + rootSum(root->left, 0) + rootSum(root->right,0);
        } else {
            return rootSum(root->left, sum - root->val) + rootSum(root->right, sum - root->val);
        }
    }
    int pathSum(TreeNode* root, int sum) {
        if (root == NULL) {
            return 0;
        }
        return rootSum(root, sum) + pathSum(root->left, sum) + pathSum(root->right, sum);
    }
};
```




