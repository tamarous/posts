# 二叉树的 Morris 遍历
> 转载自：[Morris Traversal方法遍历二叉树（非递归，不用栈，O(1)空间）](https://www.cnblogs.com/AnnieKim/archive/2013/06/15/MorrisTraversal.html)，有所修改。

Morris 遍历是由 Joseph Morris 于1979年发明的，这种遍历方法与递归以及非递归遍历方法相比，其时间复杂度为 O(n)，而空间复杂度只要 O(1)。在某些场合下，其空间复杂度为常数的特性非常有用，因此我们有必要了解和掌握它是如何实现的。

## 中序遍历

过程：
1. 如果当前节点的左孩子为空，则输出当前节点并将当前节点更新为它的右孩子。
2. 如果当前节点的左孩子不为空，在当前节点的左子树中找到它的最右节点。
3. (a) 如果最右节点的右孩子为空，将它的右孩子设置为当前节点，而当前节点更新为当前节点的左孩子。(b) 如果最右节点的右孩子为当前节点，那么将最右节点的右孩子设置为 NULL，输出当前节点，当前节点更新为当前节点的右孩子。
4. 重复上述过程，直到当前节点为 NULL。

```
class Solution {
public:
    void MorrisInOrder(TreeNode *root) {
        TreeNode *cur = head, *prev = NULL;
        while(cur != NULL) {
            if (cur->left == NULL) { // 1
                printf("%d ",cur->val);
                cur = cur->right;
            } else {
                prev = cur->left;
                // 2
                while(prev->right != NULL && prev->right != cur) {
                    prev = prev->right;
                }
                if (prev->right == NULL) { // 3.a
                    prev->right = cur;
                    cur = cur->left;
                } else {
                    prev->right = NULL;   // 3.b
                    printf("%d ",cur->val);
                    cur = cur->right;
                }
            }
        }
    }
};
```

借用下[这篇博客](http://www.cnblogs.com/anniekim/archive/2013/06/15/morristraversal.html)中的图片：

![Morris InOrder](http://images.cnitblog.com/blog/300640/201306/14214057-7cc645706e7741e3b5ed62b320000354.jpg)

我们来看下以这幅图中的树为例，对其进行 Morris 中序遍历的过程。
1. cur = 6，左孩子不为空所以进入步骤2，cur 的左子树的最右节点是5，5的右孩子为空，所以让5的右孩子指向6，然后让 cur = 2。
2. cur = 2，左孩子不为空进入步骤2，2的最右节点为1，1的右孩子为空，所以把1的右孩子设置为2，然后让 cur = 1。
3. cur = 1，1的左孩子为空，所以进入步骤1，输出1，其右孩子为2，所以重新让 cur = 2。
4. 2的最右节点是1，而1的右孩子为2，所以将1的右孩子重新设置为空，然后输出2，让 cur = 4。
5. cur = 4，左孩子不为空进入步骤2，4的最右节点为3，3的右孩子为空，所以把3的右孩子设置为4，然后让 cur = 3。
6. cur = 3，左孩子为空，所以进入步骤1，输出3，由于3的右孩子为4，所以重新让 cur = 4。
7. cur = 4，最右节点为3，3的右孩子为4，所以将3的节点重新设为空，然后输出4，让 cur = 5。
8. cur = 5，左孩子为空，所以输出5，5的右孩子为6，因此让 cur = 6。

经历了以上几步，这棵树的左子树已经全部遍历完了。右子树的遍历和左子树是类似的，结合图片和上述过程应该比较容易理解了，就不在解释了。

## 前序遍历
前序遍历和中序遍历的过程基本一样，唯一的不同点在于打印的位置不同。过程：
1. 如果当前节点的左节点为空，那么输出当前节点并将当前节点更新为它的右孩子。
2. 如果当前节点的左节点不为空，在当前节点的左子树中找到它的最右节点。
3. (a) 如果最右节点的右孩子为空，将它的右孩子设置为当前节点。**输出当前节点(与中序遍历唯一的不同点)**，将当前节点更新为当前节点的左孩子。(b) 如果最右节点的右孩子为当前节点，那么将它的右孩子重新设置为空，然后将当前节点更新为当前节点的右孩子。
4. 重复上述过程，直到当前节点为 NULL。

```
class Solution {
public:
    void MorrisPreOrder(TreeNode *root) {
        TreeNode *cur = head, *prev = NULL;
        while(cur != NULL) {
            if (cur->left == NULL) { // 1
                printf("%d ",cur->val);
                cur = cur->right;
            } else {
                prev = cur->left;
                
                while(prev->right != NULL && prev->right != cur) { // 2
                    prev = prev->right;
                }
                if (prev->right == NULL) { // 3.a
                    prev->right = cur;
                    printf("%d ",cur->val);
                    cur = cur->left;
                } else {
                    prev->right = NULL;   // 3.b
                    cur = cur->right;
                }
            }
        }
    }
};
```

![Morris PreOrder](http://images.cnitblog.com/blog/300640/201306/14221458-aa5f9e92cce743ccacbc735048133058.jpg)

## 后序遍历
后续遍历又是最为复杂的一个。我们需要建立一个额外节点 dummy，然后将根节点设置为dummy 的左孩子。过程：
1. 建立一个额外节点 dummy，然后将根节点设置为 dummy 的左孩子。
2. 如果当前节点的左孩子为空，则将其右孩子作为当前节点。
3. 如果当前节点的左孩子不为空，在当前节点的左子树中找到它的最右节点。
4. (a) 如果最右节点的右孩子为空，则将最右节点的右孩子设置为当前节点，而当前节点更新为当前节点的左孩子。(b) 如果最右节点的右孩子为当前节点，将它的右孩子重新设为空。**倒序输出从当前节点的左孩子到该最右节点路径上的所有节点**，然后将当前节点更新为当前节点的右孩子。
5. 重复以上过程，直到当前节点为NULL。

```
class Solution {
public:
    void MorrisPostOrder(TreeNode *root) {
        TreeNode *dummy = new TreeNode(0);
        dummy->left = root;
        TreeNode *cur = dummy, *prev = NULL;
        while(cur != NULL) {
            if (cur->left == NULL) {
                cur = cur->right;
            } else {
                prev = cur->left;
                
                while(prev->right != NULL && prev->right != cur) { 
                    prev = prev->right;
                }
                if (prev->right == NULL) { 
                    prev->right = cur;
                    cur = cur->left;
                } else {
                    printReverse(cur->left, prev);
                    prev->right = NULL;
                    cur = cur->right;
                }
            }
        }
    }
    
    void reverse(TreeNode *from, TreeNode *to) {
        if (from == to) {
            return;
        }
        TreeNode *x = from, *y = from->right, *z;
        while (true) {
            z = y->right;
            y->right = x;
            x = y;
            y = z;
            if (x == to)
                break;
        }
    }

    void printReverse(TreeNode* from, TreeNode *to) {
        reverse(from, to);
        
        TreeNode *p = to;
        while (true) {
            printf("%d ", p->val);
            if (p == from)
                break;
            p = p->right;
        }
        
        reverse(to, from);
    }
};
```

![Morris PostOrder](http://images.cnitblog.com/blog/300640/201306/15165951-7991525829134fb3beefed9fbf7e0536.jpg)

下面以图中的树为例，分析下 Morris 后序遍历的过程。
1. 新建一个额外节点0，将它设为当前节点，然后将根节点9设置为它的左孩子。
2. cur = 0，它的左孩子不为空，在0的左子树中寻找最右节点，为7。
3. 因为7的右孩子为空，所以将7的右孩子设置为当前节点0，然后将当前节点0更新为0的左孩子9。
4. cur = 9，由于9的左孩子不为空，因此在9的左子树中寻找它的最右节点，为3。
5. 因为3的右孩子为空，所以将3的右孩子设置为当前节点9，然后将当前节点9更新为9的左孩子5。
6. cur = 5，它的左孩子不为空，在5的左子树中找到最右节点，为1。
7. 因为1的右孩子为空，所以把1的右孩子设置为当前节点5，然后将当前节点5更新为5的左孩子1。
8. cur = 1，当前节点1的左孩子为空，所以将1的右孩子5设置为当前节点。
9. cur = 5，它的左孩子不为空，在5的左子树中寻找最后节点，为1。
10. 因为1的右孩子为5，是当前节点，因此将1的右孩子重新设为空，然后倒序输出从5的左孩子1到1的节点，即输出1，然后将当前节点5更新为当前节点的右孩子4。
11. 当前节点4的左孩子不为空，因此在4的左子树中寻找最右节点，为2。
12. 2的右孩子为空，因此将2的右孩子设置为当前节点4，然后将当前节点设置为当前节点的左孩子2。
13. cur = 2，因为当前节点的左孩子为空，所以将2的右孩子4设置为当前节点。
14. cur = 4，由11的结果，它的最右节点为2。
15. 因为2的右孩子为4，是当前节点，因此将2的右孩子重新设置为空，然后倒序输出从4的左孩子2到2的节点，即输出2，然后将当前节点4更新为当前节点的右孩子3。
16. cur = 3，因为当前节点的左孩子为空，因此将3的右孩子9设置为当前节点。
17. cur = 9，因为9的左孩子不为空，所以在它的左子树中寻找它的最右节点，为3。
18. 因为3的右孩子9为当前节点，因此将3的右孩子重新设置为空，然后倒序输出从9的左孩子5到3的节点，即输出3、4、5。
19. 至此，9的左子树已经遍历输出完毕。后续遍历右子树的过程和遍历左子树基本一样，因此不再赘述。






