# LeetCode-9-Palindrome Number
题目要求：判断一个数字是否是回文数字。
思路：通过`res = res * 10 + x % 10; x /= 10;`我们就能取出x的右半边逆序后的数字。当x的位数是奇数时，如x = 12321, 那么很容易验证循环退出时res = 123，此时判断x ?= res/10，如果相等则x是回文数，否则不是；当x的位数是偶数时，如x = 123321，那么循环退出时res = 123，此时判断x ?= res，如果相等则x是回文数。
代码如下：

```
bool isPalindrome(int x) {
    if (x < 0 || (x % 10 == 0 && x != 0)) {
        return false;LinkedList
    }
    int res = 0;
    while(x > res) {
        res = res * 10 + x % 10;
        x = x / 10;
    }
    return res == x || x == res/10;
}
```

