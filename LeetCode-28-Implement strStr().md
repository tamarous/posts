# LeetCode-28-Implement strStr()
实现类似C语言中strStr()函数的效果, 即给出两个字符串A和B:(1)如果B为NULL,则返回0;(2)如果B比A长,则返回-1;(3)如果B在A中,则返回B在A中第一次出现的起始位置;(4)如果B不在A中,则返回-1.

这道题其实思路蛮清晰的,遍历A的每个字符,然后比较B的第一个字符和当前是否相同,如果相同则继续比较,如果把B遍历完了,那么就返回A的这个字符的索引.

顺着这样的思路,我写出了这样的算法:

```
int strStr(char* haystack, char* needle) {
    if(!needle) {
        return 0;
    }
    size_t lens = strlen(haystack);
    size_t lent = strlen(needle);
    if(lens < lent) {
        return -1;
    }
    for(int i = 0; i < lens;i++) {
        int j = 0;
        for(j = 0; j < lent;j++) {
            if(needle[j] != haystack[i+j]) {
                break;
            }
        }
        if (j == lent) {
            return i;
        }
    }
    return -1;
}
```

但是提交后却一直报错,让我思考了好一会,后来看了题解才意识到, i的索引范围应该是从0到lens-lent+1, 否则按照我原来那种写法,在haystack[i+j]中就会越界,导致错误的结果. 因此只要把这里改了就可以AC了. 正确的代码如下:

```
int strStr(char* haystack, char* needle) {
    if(!needle) {
        return 0;
    }
    size_t lens = strlen(haystack);
    size_t lent = strlen(needle);
    if(lens < lent) {
        return -1;
    }
    for(int i = 0; i < lens-lent+1;i++) {
        int j = 0;
        for(j = 0; j < lent;j++) {
            if(needle[j] != haystack[i+j]) {
                break;
            }
        }
        if (j == lent) {
            return i;
        }
    }
    return -1;
}
```

