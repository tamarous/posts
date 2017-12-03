# LeetCode-21-Merge Two Sorted Lists
这道题的要求是合并两个有序单链表。描述很简单，思路更加简单，就是先创建一个新的链表节点，然后用两个指针指向这两个单链表，比较这两个指针指向元素的大小，将较小的一个的值赋给新节点的元素，如果这两个元素相同，那随便赋谁的值都可以。当这两个链表有一个已经遍历完之后，就直接把另一条链表的其余元素复制到新链表中去。
代码如下：

    class Solution {
    public:
        ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
            if (l1 == NULL && l2 == NULL) {
            return NULL;
        } else if (! l1 && l2) {
            return l2;
        } else if (l1 && !l2) {
            return l1;
        } else {
            ListNode *ptrToA = l1, *ptrToB = l2;
            ListNode *cur = (ListNode *)malloc(sizeof(ListNode));
            ListNode *head = cur;
            while(ptrToA && ptrToB) {
                while(ptrToA && ptrToB && (ptrToA->val < ptrToB->val)) {
                    ListNode *newCell = (ListNode *)malloc(sizeof(ListNode));
                    newCell->val = ptrToA->val;
                    cur->next = newCell;
                    cur = newCell;
                    ptrToA = ptrToA->next;
                }
                while(ptrToA && ptrToB && (ptrToA->val > ptrToB->val)) {
                    ListNode *newCell = (ListNode *)malloc(sizeof(ListNode));
                    newCell->val = ptrToB->val;
                    cur->next = newCell;
                    cur = newCell;
                    ptrToB = ptrToB->next;
                }            
                while(ptrToA && ptrToB && (ptrToA->val == ptrToB->val)) {
                    ListNode *newCellA = (ListNode *)malloc(sizeof(ListNode));
                    newCellA->val = ptrToA->val;
                    cur->next = newCellA;
                    cur = newCellA;
                    ListNode *newCellB = (ListNode *)malloc(sizeof(ListNode));
                    newCellB->val = ptrToB->val;
                    cur->next = newCellB;
                    cur = newCellB;
                    ptrToA = ptrToA->next;
                    ptrToB = ptrToB->next;
                }
            }
            if(!ptrToA && ptrToB) {
                while(ptrToB) {
                    ListNode *newCell = (ListNode *)malloc(sizeof(ListNode));
                    newCell->val = ptrToB->val;
                    cur->next = newCell;
                    cur = newCell;
                    ptrToB = ptrToB->next;
                }
                cur->next = NULL;
            } else if(ptrToA && !ptrToB) {
                while(ptrToA) {
                    ListNode *newCell = (ListNode *)malloc(sizeof(ListNode));
                    newCell->val = ptrToA->val;
                    cur->next = newCell;
                    cur = newCell;
                    ptrToA = ptrToA->next;
                }
                cur->next = NULL;
            } else if (!ptrToA && !ptrToB ) {
                cur->next = NULL;
            }
            return head->next;
        }
        }
    };

**Update**: 在讨论区中看到了这道题的另外两种解法，并且附上了很详细的分析，[传送门](https://leetcode.com/articles/merged-two-sorted-lists/)。相比于我自己的方法，下面的方法显得更加优美简单，并且在时间和空间复杂度上都更有优势。
 递归法：
 
    class Solution {
    public:
        ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
            if (l1 == NULL) {
                return l2;
            } else if (l2 == NULL) {
                return l1;
            } else if (l1->val < l2->val) {
                l1->next = mergeTwoLists(l1->next,l2);
                return l1;
            } else {
                l2->next = mergeTwoLists(l1, l2->next);
                return l2;
            }
        }
    };
    
时间复杂度上，合并的总次数显然与L1和L2的长度有关，因此是O(m+n)；空间复杂度上，由于这是递归调用的，而第一次调用mergeTwoLists的返回条件是l1和l2均为NULL，也就是说返回时递归调用了`mergeTwoLists` (m+n)次，所以空间复杂度也为O(m+n)。

迭代法：

```
class Solution {
public:
    ListNode * mergeTwoLists(ListNode *l1, ListNode *l2) {
        ListNode *prehead = new ListNode(-1);
        ListNode *prev = prehead;
    
        while(l1 != NULL && l2 != NULL) {
            if (l1->val <= l2->val) {
                prev->next = l1;
                l1 = l1->next;
            } else {
                prev->next = l2;
                l2 = l2->next;
            }
            prev = prev->next;
        }
    
        prev->next = l1 == NULL ? l2:l1;
        return prehead->next;
    }
};
```

这种方法的时间复杂度仍为O(m+n)；由于只分配了两个额外的指针，因此空间复杂度为O(1)。
    




