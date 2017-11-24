# 由前序和中序遍历恢复二叉树结构
这是一道比较经典的题目，题目来源是[Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)。题目大意就是给出一个二叉树的前序和中序遍历的结果，让从这个结果中分析出二叉树的结构来。
容易知道，一个二叉树的前序遍历的第一个值，就是这个二叉树的根节点的值。知道了根节点后，中序遍历结果中所有位于根节点左侧的值就对应着根节点的左孩子及左孩子的子节点，而位于根节点右侧的值则对应着根节点的右孩子及右孩子的子节点。这样就把根节点找到了，左右孩子的范围也找到了，后续就可以采用递归的思想来依次寻找左右孩子的具体值了。

代码如下：

    class Solution
    {
    public:
        TreeNode *buildTree(vector<int>&preorder,vector<int>& inorder) {
            if (preorder.size() == 0 || inorder.size() == 0) {
                return NULL;
            }
            if (preorder.size() != inorder.size()) {
                return NULL;
            }
            int length = preorder.size();
            int *start_pre = &preorder[0];
            int *start_in = &inorder[0];
            return construct(start_pre,start_pre+length-1,start_in,start_in+length-1);
        }
    private:
        TreeNode *construct(int *start_pre,int *end_pre, int *start_in,int *end_in) {
            int value = start_pre[0];
            TreeNode *root = new TreeNode(0);
            root->val = value;
            root->left = root->right = NULL;
            if (start_pre == end_pre) {
                if (start_in == end_in && *start_pre == *start_in) {
                    return root;
                } else {
                    return NULL;
                }
            }
    
            int *rootInOrder = start_in;
            while(rootInOrder != end_in && *rootInOrder != value) {
                rootInOrder++;
            }
    
            if (rootInOrder == end_in && *rootInOrder != *end_in) {
                return NULL;
            }
    
            int leftLength = rootInOrder-start_in;
            if (leftLength > 0) {
                root->left = construct(start_pre+1,start_pre+leftLength,start_in,rootInOrder-1);
            } 
            if (leftLength < end_pre - start_pre) {
                root->right = construct(start_pre+1+leftLength,end_pre,rootInOrder+1,end_in);
            }
            return root;
        }
    };


