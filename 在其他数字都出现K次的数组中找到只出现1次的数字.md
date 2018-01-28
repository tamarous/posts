# 在其他数字都出现K次的数组中找到只出现M次的数字

## 在其他数字都出现偶数次的数组中找到只出现奇数次的1个数字

### 题目描述

给定一个数组 arr，其中只有一个数字出现了奇数次，其他数字都出现了偶数次，找出这个数字。

### 思路

使用异或运算符。我们可以声明一个变量 x，用它去和数组中的每个元素做异或运算，由于 n ^ n = 0, n ^ 0 = n，那么那些出现偶数次的数字和自身异或，结果为0。最后x中存的就是数组中只出现奇数次次的数字。
### 代码

```
class Solution {
public:
	int findOddTimesNum(vector<int> &nums) {
    	int x = 0;
    	for(int i = 0; i < nums.size() ;i ++) {
          	x ^= nums[i];
    	}
    	return x;
	}
};
```



## 在其他数字都出现偶数次的数组中找到只出现奇数次的两个数字

### 题目描述

给定一个整数数组 arr，已知这个数组中有两个数字只在数组中出现了奇数次，而其他数字都出现了偶数次。找到这两个数字。

### 思路

我们首先声明两个变量 x 和 y，让它们的初始值为0。如果该数组中的 a 和 b 都出现了奇数次，那么用 x 和数组中的每个元素异或的最终结果是 x = a ^ b，这个数字不为 0，所以 x 肯定有一个 bit 位是不为 0 的，假设是第 k 位不为 0，那么 a 和 b 中肯定有一个的第 k 位是1，而另外一个的第 k 位是 0。接下来用 y 去和数组中第 k 位为 1 的那些元素来做异或运算，当遍历完成时 y 中的值就是 a 和 b 中的某一个，而 x ^ y 就是另外一个。

### 代码

```
class Solution {
public:
    vector<int> singleNumber(vector<int>& nums) {
        vector<int> result;
        if(nums.size() < 0) {
            return result;
        }

        int x = 0, y = 0;
        int size = nums.size();
        for(int i = 0; i < size; i++) {
            x ^= nums[i];
        }
        int temp = x &(~x + 1);
        for(int i = 0; i < size; i++) {
            if(temp & nums[i]) {
                y ^= nums[i];
            }
        }

        x = x ^ y;
        result.push_back(x);
        result.push_back(y);
        return result;
    }
};
```



## 在其他数字都出现K次的数组中找到只出现一次的一个数字

### 题目描述

给定一个整数数组 arr 和一个大于 1 的整数 k。已知 arr 中只有 1 个数出现了 1 次，其他的数字都出现了 k 次。找出这个出现了 1 次的数字。

### 思路

声明一个长度为32的整形数组 digit，将这个数组初始化为0。然后遍历数组，计算 arr 中的每一个数字的第i位相加的和，存放在 digit[i] 中。如果我们让 digit[i] 对 K 取模，那么 digit[i] % k 就是 arr 中只出现 1 次的数字第 i 位上的值。因此我们可以从 digit 数组中恢复出要找的那个数字。举个例子：

```
X=***1***1***

Z=***0***0***

X=***1***1***

Y=***0***1***

X=***1***1***

Z=***0***0***

Z=***0***0***

     P   Q
```

X，Y，Z 是数组中的三个数字，X 和 Z 都出现了3次，而 Y 只出现了1次。将这三个数第 P 位上的数字相加，得到3，3 % 3 =0，即是 Y 第 P 位上的数；将第 Q 位上的数字相加，得到4，4 % 3 = 1，即是 Y 第 Q 位上的数字。

### 代码
```c++
int singleNumber(vector<int>& s) 
 {
     vector<int> t(32);////Made a array contain 32 elements.
     int sz = s.size();
     int i, j, n;
     for (i = 0; i < sz; ++i)
     {
     	n = s[i];
     	for (j = 31; j >= 0; --j)
     	{
     		t[j] += n & 1;
     		n >>= 1;
     		if (!n)
     			break;
     		}
     }
	int res = 0;
	for (j = 31; j >= 0; --j)
	{
		n = t[j] % 3;//"3" represents k times. 
		if (n) {
         	res += 1 << (31 - j);
		}
	}
	return res;
}
```

下面这种[解法](https://leetcode.com/problems/single-number-ii/discuss/43294/)更加神奇：

```
int singleNumber(vector<int> & A) {
    int b0 = 0, b1 = 0;
    for(int i = 0; i < A.size(); i++){
        b0 = (b0 ^ A[i]) & ~b1;
        b1 = (b1 ^ A[i]) & ~b0;
    }
    return b0;
}
```
具体解释可参看上面的链接。