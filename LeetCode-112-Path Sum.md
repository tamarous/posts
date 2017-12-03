# LeetCode-112-Path Sum
题意描述：给出一棵二叉树和一个数值sum，判断这棵树从树根到某个叶子的路径上的数值之和是否等于sum。
实例：
给出下面这棵树和一个数字22，因为5->4->11->2这条路径上的数值之和为22，所以返回true。

```
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
```
思路：这个仍然是一个可以用递归思路解决的问题。首先是写一个辅助函数helper，第一个参数毫无疑问应该留给当前节点，第二个参数应该是与给出的数值有关的量,由于是递归调用，所以如果有返回值的话不好处理，因此我们让helper返回void，那么需要有一个变量来记录是否满足题目要求了，所以第三个参数是一个引用类型的布尔变量，用来标识是否有这样一条路径。
对于当前节点来说，如果是叶子节点，并且`路径上前些节点数值之和+当前节点数值 == sum`或者`sum - 路径上前些节点数值之和 == 当前节点数值`，那么就说明是存在这样一条路径的，返回true；如果不是叶子节点，那么就更新`路径上前些节点数值之和`，并分别在自己的左右孩子上调用helper方法。
代码如下（采用的是减法的思路）：

```
class Solution {
    public:
        bool hasPathSum(TreeNode* root, int sum) {
            bool hasSum = false;
            helper(root, sum, hasSum);
            return hasSum;
        }
    private:
        void helper(TreeNode *root, int sum, bool &flag) {
            if (root == NULL) {
                return;
            }
            sum -= root->val;
            if (root->left == NULL && root->right == NULL) {
                if (sum == 0) {
                    flag = true;
                }
            } else {
                if (root->left != NULL) {
                    helper(root->left, sum, flag);
                } 
                if (root->right != NULL) {
                    helper(root->right, sum, flag);
                }
            }
        }
        sum += root->val;
};
```
不过在看了讨论区的解法之后我发觉我写的还是太复杂了，他们的解法如下，更加简洁一些：

```
class Solution {
public:
    bool hasPathSum(TreeNode* root, int sum) {
        if (root == NULL) {
            return false;
        }
        if (root->left == NULL && root->right == NULL && sum - root->val == 0) {
            return true;
        }
        return hasPathSum(root->left, sum - root->val) || hasPathSum(root->right, sum - root->val);
    }
};
```

