# LeetCode-100-Same Tree及572-Subtree of Another Tree
之所以把这两道题放到一起，是因为第一道题的算法是第二道题的一块小零件，如果理解第一题的解法的话，那么看到第二题应该就很快有思路了。
## Same Tree
本题的要求是判断两棵树是否相同。根据题目描述，两棵树相同的条件是这两棵树的结构相同，并且对应位置上的节点的值也相同。
我们将这两棵树分别称为 s和 t。首先先处理一些特殊的情况：

* 当 s 和 t 中有一个不为空，也就意味着这两棵树的结构不同，因此应该返回 false。
* 当 s 和t 全为空时，应该返回 **true**。
* 当 s 和 t 全不为空时，此时先比较 s 和 t 的值是否相同，如果不同，则返回false, 如果相同，则递归地比较s 和 t 的左右孩子是否相同。

顺着以上思路，不难写出如下代码：

    class Solution {
    public:
        bool isSameTree(TreeNode* p, TreeNode* q) {
            if (!p && !q) {
                return true;
            }
            if (p == NULL || q == NULL) {
                return false;
            }
            if (p->val != q->val) {
                return false;
            }
            return isSameTree(p->left,q->left) && isSameTree(p->right,q->right);
        }
    };  
    
### Subtree of Another Tree
本题的要求是判断 t 是否是 s 的子树，也就是说如果 t 和 s 的某一部分的结构和对应节点的值相同的话，那么 t 就是 s 的一棵子树。经过思考我们发现，其实这道题的核心思路就是判断 t 和 s 的某一部分是否是相同的，因此可以借用上面的算法。因此这个题的思路就很明了了：首先判断 s和 t 是否相同，相同则返回 true；否则判断 t 是否是 s 的左孩子或右孩子的子树。代码如下：

    class Solution {
    public:
        bool isSubtree(TreeNode* s, TreeNode* t) {
            if ( s == NULL) {
                return false;
            }
            if (isSameTree(s,t)) {
                return true;
            }
            return isSubtree(s->left,t) || isSubtree(s->right,t);
        }
        
    private:
        bool isSameTree(TreeNode *s, TreeNode *t) {
            if (!s && !t) {
                return true;
            }
            if (s == NULL || t == NULL) {
                return false;
            }
            if (s->val != t->val) {
                return false;
            } else {
                return isSameTree(s->left,t->left) && isSameTree(s->right,t->right);
            }
        }
    };


