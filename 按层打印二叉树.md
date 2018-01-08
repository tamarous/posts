## 按层打印二叉树
题意：给定一棵二叉树，将同一层上的节点打印在同一行上。

```
class Solution {
public:
    void printByLevel(TreeNode *head) {
        if (head == NULL) {
            return;
        }
        queue<TreeNode *> queue;
        
        // 当前层的节点个数
        int current = 1;
        queue.push(head);
        // 下一层的节点个数
        int next = 0;
        
        while(! queue.empty()) {
            head = queue.front();
            queue.pop();
            current--;
            printf("%d ",head->val);
            if (head->left != NULL) {
                queue.push(head->left);
                next++;
            }
            if (head->right != NULL) {
                queue.push(head->right);
                next++;
            }
            if (current == 0) {
                printf("\n");
                current = next;
                next = 0;
            }
        }
    }
};
```

## ZigZag 打印二叉树
代码：

```
class Solution {
public:
    void printZigZag(TreeNode *head) {
        if (head == NULL) {
            return NULL;
        }
        deque<TreeNode *> deque;
        int level = 1;
        bool lr = true;
        TreeNode *last = head;
        TreeNode *nLast = NULL;
        deque.push_front(last);
        printLevel(level++,last);
        while(! deque.empty()) {
            if (lr) {
                head = deque.pop_front();
                if (head->left != NULL) {
                    nLast = nLast == NULL ? head->left : nLast;
                    duque.push_back(head->left);
                } 
                if (head->right != NULL) {
                    nLast = nLast == NULL ? head->right: nLast;
                    deque.push_back(head->right);
                }
            } else {
                head = deque.pop_back();
                if (head->right != NULL) {
                    nLast = nLast == NULL ? head->right: nLast;
                    deque.push_front(head->right);
                }
                if (head->left != NULL) {
                    nLast = nLast == NULL? head->left: nLast;
                    deque.push_front(head->left);
                }
            }
            printf("%d ",head->val);
            if (head == last && !deque.empty()) {
                lr = !lr;
                last = nLast;
                nLast = NULL;
                printf("\n");
                printLevel(level++,lr);
            }   
        }
        printf("\n");
    }
    
    void printLevel(int level, bool lr) {
        printf("level %d from ",level);
        printf(lr?"left to right: ":"right to left: ");
    }
} 
```

