# LeetCode-9-Palindrome Number
这道题的题目是：判断一个数字是否是回文数，但是要求不能分配额外的空间。
思路很容易，就是看这个数字的前半部分和后半部分翻转后的值是否相同。我们可以利用取一个数字的各位上的数字的方法求出这个数字的后半部分翻转后的值：

```
    int res = 0;
    while(x > res) {
        res = res * 10 + x % 10;
        x = x / 10;
    }
```
注意，当x的位数是偶数时，直接比较res和x的值是否相同即可；但是当x的位数是奇数，如x=12321，当res=123、x=12，此时循环跳出，判断条件只要改成x==res%10即可。因此完整的代码是：

    bool isPalindrome(int x) {
        if (x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }
        int res = 0;
        while(x > res) {
            res = res * 10 + x % 10;
            x = x / 10;
        }
        return res == x || x == res/10;
    }

由这道题还可以引申出一道类似的题目：判断一个字符串是不是一个回文串。回文串的定义和回文数字类似，就是正着读倒着读效果是一样的字符串，如"charahc"。只需要用两个指针分别指向字符串的头和尾，比较当前这两个指针所指向的字符是否相同，若相同则继续移动这两个指针，直到这两个指针相遇为止。代码如下：

    bool isPalindrome(char *str) {
        if (str == NULL) {
            return false;
        }
    
        int len = strlen(str);
        if (len == 1) {
            return true;
        }
    
        char *front = str;
        char *end = str + len-1;
        while(front != end) {
            if (*front != *end) {
                return false;
            }
            front++;
            end--;
        }
        return true;
    }





