# power (x,n)
下面几道题都是和求 x的 n 次方形式类似的，在这里一起总结一下。

## LeetCode-50-Pow(x,n)
这个题目是要自己实现求解 x 的 n 次方，由于有公式`x^n = x^(n/2)*x^(n/2)*x^(n%2)`，所以求 x 的 n 次方可以转化为求 x 的 n/2次方，求 x 的n/2次方可以转化为求 x的 n/4方，因此该问题可以转化为很多个和原问题类似的子问题，而只要把这些子问题的解合并起来就是原问题的解了。
代码如下：

    class Solution {
    public:
        double myPow(double x, int n) {
            if (n < 0) {
                return 1.0/power(x,n);
            }
            return power(x,n);
        }
        
    private:
        double power(double x, int n) {
            if ( n == 0) {
                return 1.0f;
            }
            double v = power(x,n/2);
            if (n % 2== 0) {
                return v * v;
            } else {
                return v * v * x;
            }
            
        }    
    };

## Leetcode-231-Power of Two
这道题是让判断一个数字是否是2的幂次方。一个数字A，如果它是2的幂次方，那么A/2也一定是2的幂次方。
因此一种思路是：如果要判断的数字A是奇数，那么直接返回false，否则就返回以A/2作为参数递归调用这个函数的结果；另一种思路是不用递归，而是使用循环来解决。在性能上循环的解法会好很多。
递归代码如下：

    class Solution {
    public:
        bool isPowerOfTwo(int n) {
            if (n <= 0) {
                return false;
            }
            if (n == 1) {
                return true;
            }
            if (n % 2 == 1) {
                return false;
            } else {
                return isPowerOfTwo(n >> 1);
            }
        }
    };
    
循环代码如下：

```   
class Solution {
public:
    bool isPowerOfThree(int n) {
        if (n < 1) {
            return false;
        }
        while ( n % 2 == 0) {
            n /= 2;
        }

        return n == 1;
    }
};
```
  

## LeetCode-326-Power of Three
这道题是让判断一个数字是否是3的幂次方。做法也是和上面类似的，代码如下：
  
```   
class Solution {
public:
    bool isPowerOfThree(int n) {
        if (n < 1) {
            return false;
        }
        while ( n % 2 == 0) {
            n /= 2;
        }

        return n == 1;
    }
};
```

但是呢这个题还有个附加题，那就是尝试不用循环或者递归来解决。那么这就必须借助一些数学上的小技巧了。
我们知道如果一个数字n是3的i次方，那么`i = log3(n)`，而由换底公式,`log3(n) = log10(n)/log10(3)`，所以只要判断这个除法是不是整除即可。代码如下：

```
class Solution {
public:
    bool isPowerOfThree(int n) {
        return (log10(n)/log10(3)) % 1 == 0;
    }
};
```

## LeetCode-342-Power of Four
这个题和之前的两个题类似，让判断一个数字是否是4的幂次方。有了前面的三种方法做铺垫，相信要想写出正确的代码已经不难了，这里就不再贴出代码了。



  
  
  



