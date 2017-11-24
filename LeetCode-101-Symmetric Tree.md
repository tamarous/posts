# LeetCode-101-Symmetric Tree
这道题的题意是判断一棵二叉树是不是对称的，如下图，当一棵树呈现这样的结构时，就可以称作是对称二叉树。

![Symetric Tree](http://www.ideserve.co.in/learn/img/mirrorTree_0.gif)

我采用的是递归的方法，根据题意，如果输入的根节点为空的话，那么需要返回 NULL；然后调用一个辅助函数来处理根节点的左右子树。在这个辅助函数里面，我们将左右子树作为参数传入，先判断左右子树的结构是否相同，相同的前提下判断这两个镜像位置的节点的数值是否相同，然后递归调用。

    class Solution {
        public:
            bool isSymmetric(TreeNode *root) {
                if (root == NULL) {
                    return true;
                }
                return isSymmetricHelper(root->left, root->right);
            }
        private:
            bool isSymmetricHelper(TreeNode *p,  TreeNode *q) {
                if (!p && !q) {
                    return true;
                }
                if (p == NULL || q == NULL) {
                    return false;
                }
                return (p->val == q->val) && isSymmetricHelper(p->left, q->right) && isSymmetricHelper(p->right, q->left);
            }
        };

今天刷剑指 offer，发现这道题还有另一种解法：首先定义另一种前序遍历A，也就是访问过父节点后，先遍历右子节点再遍历左子节点。那么一棵对称二叉树的A遍历和它的正常前序遍历结果是一样的。根据这种思路，可以写出如下的算法：

    class Solution {
    public:
        bool isSymmetric(TreeNode *root) {
            return isSymmetrical(root,root);
        }    
        bool isSymmetrical(TreeNode *root1, TreeNode *root2) {
            if (root1 == NULL && root2 == NULL) {
                return true;
            }
            if (root1 == NULL || root2 == NULL) {
                return false;
            }
            if (root1->val != root2->val) {
                return false;
            }
            return isSymmetrical(root1->left, root2->right) && isSymmetrical(root1->right, root2->left);
        }
    };

