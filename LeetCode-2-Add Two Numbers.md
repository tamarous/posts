# LeetCode-2-Add Two Numbers

一个非常典型的错误代码如下：

```
class Solution {
public:
    ListNode * addTwoNumbers(ListNode *l1, ListNode *l2) {
        long a = listToNumber(l1);
        long b = listToNumber(l2);
        long c = a + b;
        ListNode *result = numberToList(c);
        return result;
    }
    
    long listToNumber(ListNode *list) {
        if (list == NULL) {
            return 0;
        }
        long sum = 0;
        ListNode *ptr = list;
        int i = 0;
        while(ptr != NULL) {
            sum += ptr->val * (long)pow(10,i++);
            ptr = ptr->next;
        }
        return sum;
    }

    ListNode * numberToList(long number) {
        if (number == 0) {
            ListNode *head = new ListNode(number);
            head->next = NULL;
            return head;
        }
        ListNode *head = NULL;
        ListNode *last = NULL;
        bool firstTime = true;
        while(number) {
            long digit = number % 10;
            ListNode *ptr = new ListNode(digit);
            if (firstTime) {
                head = ptr;
                last = ptr;
                firstTime = false;
            } else {
                last->next = ptr;
            }
            last = ptr;
            number /= 10;
        }
        last->next = NULL;
        return head;
    }
};
```

上面这段代码的思路是：将这两个链表先转换成数字，然后对数字进行相加，最后再把这个数字用链表表示出来。看起来呢这种算法比较直观简单，如果用一些比较短的链表来测试的话结果也是对的，但是它的问题在于：如果链表很长很长，比如说有100个元素，那么就不能用基本类型来表示，否则就会产生溢出，造成不正确的计算结果。所以呢这个题其实是一个非常经典的大数问题。面对大数问题，我们只能一位一位地进行处理。

解决大数问题的最核心的过程就是对于进位的处理了。为了叙述的方便，假设两个输入的链表分别是A和B，输出的链表是C，那么C的第n位的数字应该是A的第n位数字加上B的第n位数字再加上前一位产生的进位。因此我们需要用一个变量标记当前位相加是否产生了进位，并且在每次相加后更新这个变量的值。剩下的内容就是一些边界情况的处理，这部分很繁琐，因此必须要非常小心，不过尝试几次后应该就可以通过了。

```
class Solution {
public:
    ListNode * addTwoNumbers(ListNode *l1, ListNode *l2) {
        if (! l1 || !l2) {
            return NULL;
        }
        ListNode *head = NULL;
        bool firstTime = true;
        ListNode *ptr1 = l1;
        ListNode *ptr2 = l2;
        bool needAddOne = false;
        ListNode *last = NULL;
        while(ptr1 && ptr2) {
            int sum = 0;
            if (needAddOne) {
                sum = ptr1->val + ptr2->val + 1;
            } else {
                sum = ptr1->val + ptr2->val;
            }
            if (sum >= 10) {
                needAddOne = true;
                sum = sum - 10;
            } else {
                needAddOne = false;
            }
            ListNode *ptr = new ListNode(sum);
            if (firstTime) {
                head = ptr;
                last = ptr;
                firstTime = false;
            }
            last->next = ptr;
            last = ptr;
            ptr1 = ptr1->next;
            ptr2 = ptr2->next;
        }
        if (!ptr1 && !ptr2) {
            if (needAddOne) {
                ListNode *ptr = new ListNode(1);
                last->next = ptr;
                last = ptr;
                ptr->next = NULL;
            } else {
                last->next = NULL;
            }
        } else {
            if (ptr1) {
                while (ptr1) {
                    int sum = 0;
                    if (needAddOne) {
                        sum = ptr1->val + 1;
                    } else {
                        sum = ptr1->val;
                    }
                    if (sum >= 10) {
                        sum -= 10;
                        needAddOne = true;
                    } else {
                        needAddOne = false;
                    }
                    ListNode *ptr = new ListNode(sum);
                    last->next = ptr;
                    last = ptr;
                    ptr1 = ptr1->next;
                }
                if (needAddOne) {
                    ListNode *ptr = new ListNode(1);
                    last->next = ptr;
                    last = ptr;
                    ptr->next = NULL;
                } else {
                    last->next = NULL;
                }
            } else if (ptr2) {
                while(ptr2) {
                    int sum = 0;
                    if (needAddOne) {
                        sum = ptr2->val+1;
                    } else {
                        sum = ptr2->val;
                    }
                    if (sum >= 10) {
                        sum -= 10;
                        needAddOne = true;
                    } else {
                        needAddOne = false;
                    }
                    ListNode *ptr = new ListNode(sum);
                    last->next = ptr;
                    last = ptr;
                    ptr2 = ptr2->next;
                }
                if (needAddOne) {
                    ListNode *ptr = new ListNode(1);
                    last->next = ptr;
                    last = ptr;
                    ptr->next = NULL;
                } else {
                    last->next = NULL;
                }
            }
        }
        return head;
    }
};
```

**UPDATE** 好吧又被别人的算法虐了：别人的方法只要19行，而我的却要103行。。。
好吧，差距还是很大的，虽然我的方法也能通过，但是在面试时如果有时间要求的话那么我的那种方法就很难写正确了。仔细看了一下，实现的差异在于对两条链表是否为空这种情况的判断：我是按照：两者均不为空、l1为空、l2为空、两者均为空这四种情况来处理的，而别人将这几个条件浓缩在了(p != NULL || q != NULL)这一个条件判断中，因此简化了很多。另外可以看到别人设置进位标记的那行代码：`carry = sum / 10`，比我采用的判断方法要简洁很多。

```
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}
```


