# LeetCode-113-Path Sum Ⅱ
这道题与[上一篇博客](http://www.tamarous.com/2017/11/26/path-sum/)的那道题非常类似，比那个题更近一步，要求找出节点之和等于给定数值的所有路径。
思路：与上一题类似，定义一个递归用的辅助函数helper，第一个参数是当前节点，第二个参数是与sum有关的值，第三个参数是一个vector的vector，用来存储所有路径，而第四个参数是一个用来存储当前路径的vector。为了能对路径进行修改，后两个参数都是引用类型。
函数定义如下：
`void helper(TreeNode *node, int sum, vector<vector<int> > & paths, vector<int> &path);`
代码实现也与上一题的非常类似，这里就不再仔细说明思路了。要注意最后两行代码，表示的含义是这条路径已经走到了叶子节点处理完了，因此要回到叶子节点的父节点，并且在回到那个节点之前我们要把减掉的叶子节点的数值再加回来。
代码如下：

```
 class Solution {
 public:
     vector<vector<int> > pathSum(TreeNode *root, int sum) {
         vector< vector<int> > results;
         vector<int> result;
         helper(root, sum, results,result);
         return results;
     }

 private:
     void helper(TreeNode *root, int sum, vector<vector<int> > &paths,vector<int>& path) {
         if (root == NULL) {
             return;
         }
         sum -= root->val;
         path.push_back(root->val);
         if (root->left == NULL && root->right == NULL) {
             if (sum == 0) {
                 paths.push_back(path);
             }
         } else {
             if (root->left != NULL) {
                 helper(root->left, sum, paths,path);
             }
             if (root->right != NULL) {
                 helper(root->right, sum, paths,path);
             }
         }
         sum += root->val;
         path.pop_back();
     }
 };
```




