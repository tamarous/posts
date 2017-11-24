# LeetCode-191-Number of 1 Bits

[这道题](https://leetcode.com/problems/number-of-1-bits/description)是一道很有意思的题目，题意是求一个uint32_t 类型数字的二进制表示中字符1的个数。如数字11，它的二进制表示是`00000000000000000000000000001011`，因此1的个数是3。

当说到二进制表示时，我们可能首先就会想到这道题可以用位运算来解决。只要把输入的每一位依次拿出来和1做与运算，如果结果为1，则说明该位上的数字为1，否则为0。所以可以很快写出下面的解法：

    class Solution {
    public:
        int hammingWeight(uint32_t num) {
            int count = 0;
            while (num) {
                if (num & 1) {
                    count++;
                }
                n >> 1;
            }
            return count;
        }
    };
    
那么当输入不是无符号整数，而是有符号数时，上面的解法还适用嘛？
我们知道，当右移一个无符号整数时，会用0来填补左边的空位，因此上述解法的判断可以正常退出，但是当右移的是一个有符号数时，如果它是一个正数，则会用0来填补左边空位；如果是一个负数，则会用符号位也就是1来填补左边的空位，因此最后该数字会变成0xFFFFFFFF，从而陷入无限循环中。

我们可以换个思路，不让数字右移，而是将掩码位左移。

    class Solution {
    public:
        int hammingWeight(uint32_t num) {
            int mask = 1;
            int count = 0;
            while(mask) {
                if (num & mask) {
                    count ++;
                }
                mask << 1;
            }
            return count;
        }
    };

当左移时，将会在右侧补上0，因此如果掩码左移了32位，那么它将会变成0，所以循环就可以退出了。

这还不算完，在这道题的讨论区里，还有人提了这样一个解法：当该数不为0时，将该数减去1，所得的结果再和原数做与运算，然后再赋值给这个数，这样循环进行，直到退出。

    class Solution {
    public:
        int hammingWeight(uint32_t num) {
            int count = 0;
            while(num) {
                num = num & (num-1);
            }
            return count;
        }
    };
    
至于原理，可以用下面👇这张图来解释：

![191_Number_Of_Bits](https://leetcode.com/media/original_images/191_Number_Of_Bits.png)

我觉得一般能发现第一个方法中的不足，从而写出第二个算法的人已经是很厉害了，而第三个方法则显得有些 tricky，因此想不到也是很正常的，不过从中还是能够体会出位运算的神奇之处。



    


