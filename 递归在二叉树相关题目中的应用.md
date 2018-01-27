# 递归在二叉树相关题目中的应用
最近集中刷了几道和二叉树有关的题目。由于二叉树本身就是递归定义的，因此使用递归算法解决二叉树的相关问题，不仅思路清晰，算法简单，而且代码量也比非递归的解法要小很多。在此将这些题目一并总结和记录一下。在介绍一般性的做法之前，先说一下自己总结出的经验。
**经验一**：使用一个辅助函数，在这个辅助函数中进行递归的操作，可以使代码变得更加简单。
**经验二**：在递归函数的开始处先处理特殊情况，也就是所谓的递归返回点。

下面是本文中二叉树的数据结构定义:
    
    struct TreeNode {
        int val;
        TreeNode *left;
        TreeNode *right;
        TreeNode(int x):val(x)，left(NULL)，right(NULL){}
    };

* 合并两个二叉树，[LeetCode-617-Merge Two Binary Trees](https://leetcode.com/problems/merge-two-binary-trees/)。给出两个二叉树，将这个二叉树合并成一个新的二叉树。合并规则是对应位置上的值相加。最后返回新的二叉树的根节点。
思路: 使用递归算法，在合并当前节点之前，先合并当前节点的左孩子和右孩子。

        class Solution {
        public:
            TreeNode *mergeTrees(TreeNode *t1, TreeNode *t2) {
                if( !t1 && !t2) {
                    return NULL;
                }
                struct TreeNode *node = (TreeNode *)malloc(sizeof(TreeNode));
                if (!t1 && t2) {
                    node->val = t2->val;
                    node->left = mergeTrees(NULL, t2->left);
                    node->right = mergeTrees(NULL, t2->right);
                } 
                if (t1 && !t2) {
                    node->val = t1->val;
                    node->left = mergeTrees(t1->left, NULL);
                    node->right = mergeTrees(t1->right, NULL);
                }
                if (t1 && t2) {
                    node->val = t1->val + t2->val;
                    node->left = mergeTrees(t1->left, t2->left);
                    node->right = mergeTrees(t1->right, t2->right);
                }
                return node;
            }
        };
                
* 前中后各种遍历。使用递归算法时这三种遍历的思路都是一样的，区别只是访问root节点的顺序。另外这几种遍历方法也可以使用非递归的方式实现，不过需要借助栈、队列等额外的数据结构，详见我的另一篇[博客](http://www.tamarous.com/2017/10/09/several-ways-of-travel-tree/)。

1. 前序遍历

        class Solution {
        public:
            void preOrder(TreeNode *root) {
                if(root == NULL) {
                    return;
                }
                cout << root->val << endl;
                preOrder(root->left);
                preOrder(root->right);
            }
        }

2. 中序遍历

        class Solution {
        public:
            void inOrder(TreeNode *root) {
                if(root == NULL) {
                    return;
                }
                inOrder(root->left);
                cout << root->val << endl;
                inOrder(root->right);
            }
        }
    
3. 后序遍历

        class Solution {
        public:
            void postOrder(TreeNode *root) {
                if(root == NULL) {
                    return;
                }
                postOrder(root->left);
                postOrder(root->right);
                cout << root->val << endl;
            }
        }

* 判断两个二叉树是否是相同的，[LeetCode-100-Same Tree](https://leetcode.com/problems/same-tree/)。 两个二叉树相同的定义是这两个二叉树的结构相同，并且对应位置上的节点的数值也相同。

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

* 判断一棵二叉树是否是对称的，[LeetCode-101-Symmetric Tree](https://leetcode.com/problems/symmetric-tree/description/)。

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

* 将一棵二叉树展平成一个单链表，[Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/description/)。
    
        class Solution {
        public:
            void flatten(TreeNode* root) {
                if(root == NULL) {
                    return;
                }
                flatten(root->right);
                flatten(root->left);
                root->right = prev;
                root->left = NULL;
                prev = root;
            }   
        private:
            TreeNode *prev;
        };
        
* 判断一棵树上是否存在一条路径，使得该路径上的所有节点的值的和等于给定的数字，[LeetCode-112-Path Sum](https://leetcode.com/problems/path-sum/)。

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
                    
                    sum += root->val;
                }
        };

* 和上道题非常类似，但是要求求出所有的符合条件的路径，[LeetCode-437-Path Sum II](https://leetcode.com/problems/path-sum-ii/description/)。

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
                 }
                 else {
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

* 修剪一棵二叉搜索树，[LeetCode-669-Trim a Binary Search Tree](https://leetcode.com/problems/trim-a-binary-search-tree/description/)。题干明确说明了该树是一棵二叉搜索树，所以具有如下的性质：某个节点的左子树上的每个节点的值都比该节点值小，而右子树上的每个节点的值都比该节点值大。因此，如果判断出当前节点的值在[L,R]之间，那么该节点就是一个子树的根节点；如果当前节点的值大于 R，那么根节点需要从它的左子树中寻找；如果当前节点的值小于 L，那么根节点需要从它的右子树中寻找。

        class Solution {
        public:
           TreeNode *trimBST(TreeNode *root ,int L, int R) {
               if (root == NULL) {
                   return NULL;
               }
               if (root->val <= R && root->val >= L) {
                   root->left = trimBST(root->left, L, R);
                   root->right = trimBST(root->right, L, R);
                   return root;
               } else {
                   if(root->val > R) {
                       return trimBST(root->left,L,R);
                   } else if (root->val < L) {
                       return trimBST(root->right, L, R);
                   }
               }
           }
        };

* 求一棵二叉树的最大深度，[LeetCode-104-Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/description/)。

        class Solution {
        public:
            int maxDepth(TreeNode* root) {
                if (root == NULL) {
                    return 0;
                }
                return 1 + max(maxDepth(root->left),maxDepth(root->right));
            }
        };
        
* 求一棵二叉树的最小深度，[LeetCode-111-Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/description/)。
思路与上一题类似，不过要注意的是如果一个节点的左孩子或者右孩子为空的话，那么以这个节点为根节点的子树的 minDepth 为它的左孩子或右孩子的 minDepth。

        class Solution {
        public:
            int minDepth(TreeNode* root) {
                if (root == NULL) {
                    return 0;
                }
                int left = minDepth(root->left);
                int right = minDepth(root->right);
                return (left == 0 || right == 0) ? (left+right+1):min(left,right)+1;
            }
        };

* 翻转一棵二叉树，[LeetCode-226-Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/description/)。

        class Solution {
        public:
            TreeNode *invertTree(TreeNode *root) {
                if (root == NULL) {
                    return NULL;
                }
                // 先递归翻转该节点的左右子树
                root->right = invertTree(root->right);
                root->left = invertTree(root->left);
                // 再交换该节点的左右孩子
                TreeNode *node = root->left;
                root->left = root->right;
                root->right = node;
                
                
                return root;
            }
        };
        
        
        
* 将一棵二叉搜索树变为一棵大树，[LeetCode-538-Convert BST to Greater Tree](https://leetcode.com/problems/convert-bst-to-greater-tree/description/)。也就是将一个节点的值更新为所有值大于该节点的节点的值之和。

        class Solution {
        public:
            TreeNode * convertBST(TreeNode *root) {
                vector<int> nodes;
                record(root, nodes);
                helper(root, nodes);
                return root;
            }
        
        private:
            void record(TreeNode *root, vector<int> &result) {
                if (root == NULL) {
                    return;
                }
                result.push_back(root->val);
                record(root->left, result);
                record(root->right, result);
            }
            void helper(TreeNode *root, vector<int> &result) {
                if (root == NULL) {
                    return;
                }
                int value = root->val;
                vector<int>::iterator iter = result.begin();
                for(iter; iter != result.end();iter++) {
                    if(*iter > value) {
                        root->val += *iter;
                    }
                }
                helper(root->left,result);
                helper(root->right,result);
            }
        };

* 求一棵二叉树所有左叶子节点的数值之和，[LeetCode-404-Sum of Left Leaves](https://leetcode.com/problems/sum-of-left-leaves/description/)。
    
         class Solution {
         public:
             int sumOfLeftLeaves(TreeNode *root) {
                 if(root == NULL) {
                     return 0;
                 }
                 
                 // 当前节点的左孩子为叶子节点
                 if (root->left && !root->left->left && !root->left->right) {
                     return root->left->val + sumOfLeftLeaves(root->right);
                 }
                 
                 return sumOfLeftLeaves(root->left)+sumOfLeftLeaves(root->right);
             }
         };
         
* 对一棵二叉树进行层序遍历，[LeetCode-102-Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/description/)。

        class Solution {
        public:
            vector<vector<int>  > levelOrder(TreeNode *root) {
                vector<vector<int> > result;
                levelTravel(root, 1, result);
                return result;
            }
        private:
            void levelTravel(TreeNode *root, int level, vector<vector<int> > &result) {
                if (root == NULL) {
                    return;
                }
                if (level > result.size()) {
                    result.push_back(vector<int>());
                }
                result[level-1].push_back(root->val);
                levelTravel(root->left, level+1, result);
                levelTravel(root->right, level+1, result);
            }
        }
* 求一棵二叉树的根节点到各个叶子节点的路径表示的值的和，[LeetCode-129-Sum Root to Leaf Numbers](https://leetcode.com/problems/sum-root-to-leaf-numbers/description/)。


```
class Solution {
public:
    int sumNumbers(TreeNode* root) {
        return dfs(root, 0);
        
    }
    int dfs(TreeNode *root, int sum) {
        if (root == NULL) {
            return 0;
        }
        if (! root->left && ! root->right) {
            return sum * 10 + root->val;
        }
        return dfs(root->left, sum * 10 + root->val) + dfs(root->right, sum * 10 + root->val);
    }
};
```

        


