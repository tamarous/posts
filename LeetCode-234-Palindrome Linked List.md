# LeetCode-234-Palindrome Linked List
题意：判断一个链表是否是回文链表。

## 方法一 使用栈

我们先遍历这个链表，然后将他的所有元素压入栈中。由于栈具有后入后出的特性，因此再依次出栈时便刚好是逆序遍历了原来的链表。我们再用一个指针从链表头开始遍历，如果对应的值相等，则说明是回文链表，否则则不是。

```
// 使用栈
class Solution {
public:
    bool isPalindrome(ListNode *head) {
        ListNode *ptr = head;
        stack<ListNode *> s;        
        while (ptr != NULL) {
            s.push(ptr);
            ptr = ptr->next;
        }
        ptr = head;
        ListNode *cur = NULL;
        while(ptr != NULL && ! s.empty()) {
            cur = s.top();
            s.pop();
            if (cur->val != ptr->val) {
                return false;
            }
            ptr = ptr->next;
        }
        return true; 
    }
};
```

## 方法二 使用栈
和方法一差不多，不同的是只需要将链表的后一半元素进栈就可以了，这样可以省掉一半的空间。

```
class Solution {
public:
    bool isPalindrome(ListNode *head) {
        if (head == NULL || head->next == NULL) {
            return true;
        }
        
        ListNode *right = head->next;
        ListNode *cur = head;
        while(cur->next && cur->next->next) {
            right = right->next;
            cur = cur->next->next;
        }
        
        // right 此时指向链表后一半的第一个元素
        
        stack<ListNode *> stack;
        while(right != NULL) {
            stack.push(right);
            right = right->next;
        }
        while(! stack.empty()) {
            ListNode *p = stack.top();
            stack.pop();
            if (p->val != head->val) {
                return false;
            }
            head = head->next;
        }
        return true;
    }
};
```

### 方法三 调整指针指向
原理参见左程云的《程序员代码面试指南》，具体过程如下：
1. 首先改变链表右半区的结构，使整个右半区反转，最后指向中间节点。
2. leftStart和rightStart同时向中间点移动，移动每一步时都比较这两者的值是否相同，如果不同则返回false。
3. 将链表恢复成原样。
4. 返回结果

```
class Solution {
public:
    bool isPalindrome(ListNode *head) {
        if (head == NULL || head->next == NULL) {
            return true;
        }
        ListNode *right = head;
        ListNode *cur = head;
        while(cur->next && cur->next->next) {
            right = right->next;
            cur = cur->next->next;
        }
        cur = right->next; // right 为右半区的第一个节点
        
        right->next = NULL;
        // 反转右半区
        ListNode *ptr = NULL;
        while(cur != NULL) {
            ptr = cur->next;
            cur->next = right;
            right = cur;
            cur = ptr;
        }
        ptr = right;
        cur = head;
        bool flag = true;
        while(right && cur) {
            if (right->val != cur->val) {
                flag = false;
                break;
            }
            right = right->next;
            cur = cur->next;
        }
        right = ptr->next;
        ptr->next = NULL;
        while(right != NULL) {
            cur = right->next;
            right->next = ptr;
            ptr = right;
            right = cur;
        }
        return flag;
    }
};
```

