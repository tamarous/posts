# 调整数组中元素的顺序
这个题是剑指 Offer 上的，题意要求是：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分。

**进阶要求**：要求奇数和奇数、偶数和偶数之间的相对顺序保持不变。

## 基础要求
### 思路分析
首先我们来考虑基础要求，也就是对调整后元素之间的相对顺序不作要求的这种情况。既然是将奇数和偶数调整到数组的前后两部分，也就是将数组分为 Odd 和 Even 两部分，这和快速排序中的 partition 算法的功能相似。因此我们可以仿照 partition 算法的实现来完成调整。具体过程如下：

1. 设置两个指针 left 和 right，分别指向数组的第一个和最后一个元素。
2. 当 left < right 时，让 left 从左向右移动，直到找到一个偶数 a。同时让 right 从右向左移动，直到找到一个奇数 b。
3. 交换奇数 a和偶数 b。
4. 如果 left < right，则进入第2步。否则，循环结束。

### 代码

```
class Solution {
public:
    void reOrderArray(vector<int> &array) {
        int size = array.size();
        if (size < 2) {
            return;
        }
        int left = 0; 
        int right = size - 1;
        while(left < right) {
            while(left < right && !isEven(array[left]) ) {
                left++;
            }
            while(left < right && isEven(array[right])) {
                right--;
            }
            if (left < right) {
                int temp = array[left];
                array[left] = array[right];
                array[right] = temp;
            }
        }
    }
    bool isEven(int n) {
        return (n & 1) == 0;
    }
};
```

## 进阶要求

### 思路分析
在进阶要求里，我们不仅需要将所有奇数调整到数组的前半部分，将偶数调整到数组的后半部分，而且还需要保证原来数组中元素的相对顺序不变。既然原题目中对于空间复杂度没有要求，那么我们可以额外开辟一个数组空间，先扫描一遍，将所有的奇数扫描进数组中，然后再扫描一遍，将所有的偶数按顺序扫描进数组中。这个方法的空间复杂度为 O(n)，而时间复杂度也为 O(n)。

### 代码

```
class Solution {
public:
    void reOrderArray(vector<int> &array) {
        int size = array.size();
        if (size < 2) {
            return;
        }
        vector<int> result;
        for(int i = 0; i < size; i++) {
            if (array[i] % 2 == 1) {
                result.push_back(array[i]);
            }
        }
        for(int i = 0; i < size; i++) {
            if (array[i] % 2 == 0) {
                result.push_back(array[i]);
            }
        }
        array = result;
    }
};
```






