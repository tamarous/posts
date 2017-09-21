# LeetCode-21-Merge Two Sorted Lists
这道题的要求是合并两个有序单链表。描述很简单，思路更加简单，就是先创建一个新的链表节点，然后用两个指针指向这两个单链表，比较这两个指针指向元素的大小，将较小的一个的值赋给新节点的元素。当这两个链表有一个已经遍历完之后，就直接把另一条链表的其余元素复制到新链表中去。
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


