# LeetCode 上几道经典的排列组合题

LeetCode 上有几道题都和排列组合有关，非常经典，值得放在一起总结一下。这几道题分别是：

* [Permutations](https://leetcode.com/problems/permutations/description/)。给定一组各不相同的数字，求这些数字的所有排列。
* [Permutations II](https://leetcode.com/problems/permutations-ii/)。给定一组数字，这些数字中可能有重复的，求这些数字的所有不重复的排列。
* [Next Permutation](https://leetcode.com/problems/next-permutation/description/)。给定一组数字的全排列中的一个排列，求这个排列的下一个排列。
* [Permutation Sequence](https://leetcode.com/problems/permutation-sequence/description/)。给定一组数字和一个数字 K，求这组数字的全排列中，按照字典序顺序排序的第 K 个排列。

## Permutations
对于一个集合来说，它的全排列是指集合内元素的所有可能的排列。以输入`[1,2,3]`为例，它的全排列是`[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`这六种情况。求一个集合的全排列有很多种方法，本文中将介绍一下最为常见的两种方法，字典序法和递归法。

### 字典序法
字典序法，就是将元素按照字典的顺序进行排列，而对于数字来说，就是将较小的数字排在较大的数字的前面。上面列出的`[1,2,3]`的全排列就是按照字典序的顺序排列的。由输入排列产生下一排列时，字典序法要求它与当前排列有尽可能长的公共前缀，因此针对输入的`1,2,3`，使用字典序法生成全排列，就是依次生成`1,3,2`，`2,1,3`，`2,3,1`，`3,1,2`，`3,2,1`。那么如何根据当前排列来生成下一排列呢？

已知当前排列，求下一排列的算法过程：

1. 对于给定排列nums,从左向右，找出第一个违反从左到右是递增顺序的数字，记为i。以`[6,8,7,4,3,2]`为例，从右向左一直是增加的，直到6的出现打破了这一规律，因此i = 6。
2. 从右向左，找出第一个比刚刚找到的数字大的数字，记为j。对于`[6,8,7,4,3,2]`，从右到左一次是2,3,4,7，因此这个数字是7，j=7。
3. 交换这两个数字。即将`[6,8,7,4,3,2]`中的6和7进行交换，变为`[7,8,6,4,3,2]`。
4. 将j右边的所有数字进行逆序，即将7右边的`[8,6,4,3,2]`逆序，`[7,8,6,4,3,2]`变为`[7,2,3,4,6,8]`。算法结束。

这里引用一下[水中的鱼](http://fisherlei.blogspot.com/2012/12/leetcode-next-permutation.html)的图片，可以对整个过程有更加直观的认识：
![水中的鱼](http://4.bp.blogspot.com/-4zN0u5JG0vs/UN0xPEkP5yI/AAAAAAAAG9Q/O48ZfwB1i_c/s1600/Picture4.png)

```
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        if (nums.size() <= 0) {
            return;
        }
        int size = nums.size();
        int violationIndex = size-1;
        while(violationIndex > 0) {
            if (nums[violationIndex] > nums[violationIndex-1]) {
                break;
            } 
            violationIndex --;
        } 
        if (violationIndex > 0) {
            violationIndex--;
            int changeIndex = size -1 ;
            for(;changeIndex>= 0 ;changeIndex--) {
                if (nums[changeIndex] > nums[violationIndex]) {
                    break;
                }
            }
            swap(nums[changeIndex],nums[violationIndex]);   
            violationIndex++;
        }
        reverse(nums.begin()+violationIndex, nums.end());
    }  
};
```
另外其实在 STL 中也有一个`next_permutation`算法，它的源代码如下：

```
template <class _Compare, class _BidirectionalIterator>
bool
__next_permutation(_BidirectionalIterator __first, _BidirectionalIterator __last, _Compare __comp)
{
    _BidirectionalIterator __i = __last;
    if (__first == __last || __first == --__i)
        return false;
    while (true)
    {
        _BidirectionalIterator __ip1 = __i;
        if (__comp(*--__i, *__ip1))
        {
            
            _BidirectionalIterator __j = __last;
            while (!__comp(*__i, *--__j))
                ;
            swap(*__i, *__j);
            _VSTD::reverse(__ip1, __last);
            return true;
        }
        if (__i == __first)
        {
            _VSTD::reverse(__first, __last);
            return false;
        }
    }
}
```
从代码中我们可以看出 STL 中的这个算法也是按照上述步骤来实现的。

通过对当前排列不断调用上述算法求出其下一排列，再令求出的排列为当前排列，就可以得到一个基于字典序的全排列了。

### 递归法

从集合中依次选出每一个元素，作为排列的第一个元素，然后对剩余的元素进行全排列，如此递归处理，从而得到所有元素的全排列。以对`[1,2,3]`进行全排列为例，我们可以这么做：

固定1，求后面23的排列：`[1,2,3]`，`[1,3,2]`；
固定2，求后面13的排列：`[2,1,3]`，`[2,3,1]`；
固定3，求后面21的排列：`[3,2,1]`，`[3,1,2]`。

```
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int> > result;
        if (nums.size() <= 0) {
            return result;
        }
        // 从索引号为0的元素开始，到最后一个元素截止
        permutation(result,nums,0,nums.size());
        return result;
    }
    
    // p 表示从索引为 p 的元素开始逐一与后面的元素交换，q 表示最后一个元素的索引
    void permutation(vector<vector<int> > &results,vector<int> &nums, int i, int j) {
        // 如果当前元素已经是最后一个元素了，那么nums 存放了一个全新的排列，此时应该回溯返回
        if(i == j-1) {
            results.push_back(nums);
            return;
        }
        for(int k = i; k < j; k++) {
            swap(nums, i, k);
            permutation(results,nums,i+1, j);
            swap(nums, i, k);
        }
    }
    
    void swap(vector<int> &nums, int i, int k) {
        int temp = nums[i];
        nums[i] = nums[k];
        nums[k] = temp;
    }
};
```

## Permutations II
我们选用递归法的框架。首先需要对输入进行排序，以将所有有重复的数字安排在相邻的位置。之后代码的处理过程就和上面的问题类似了，具体实现请看下面的代码和注释。

```
class Solution {
public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<vector<int> > results;
        sort(nums.begin(), nums.end());
        permutation(results, nums, 0, nums.size()-1);
        return results;
    }
    
    // 注意 nums 的参数传递方式是值传递而非引用传递
    void permutation(vector<vector<int> > &results, vector<int> nums, int i , int j )  {
        if (i == j) {
            results.push_back(nums);
            return;
        } else {
            for(int k = i; k < j; k++) {
            
                // 因为已经对输入进行排序，将重复的数字安排在相邻的位置，所以如果nums[i] == nums[k]，就表示遇到重复数字了，可以跳过
                if (i != k && nums[i] == nums[k]) {
                    continue;
                }
                swap(nums, i, k);
                backtrace(results, nums, i+1, j);
            }
        }
    }
    
    void swap(vector<int> &nums, int i, int k) {
        int temp = nums[i];
        nums[i] = nums[k];
        nums[k] = temp;
    }

};
```

## Next Permutation
给定一组数字的全排列中的一个排列，求这个排列的下一个排列。这个题其实就是全排列的字典序法中的一个子步骤，在上文中已经介绍过了，因此不再赘述。

## Permutation Sequence
对于一个集合`[1,2,3,...,n]`来说，它的全排列有n!种。现在给定n 和 k，求`[1,2,3,...,n]`的全排列中的第k个排列。

其实这个题目的解法非常简单，直接利用利用求下一排列的算法即可：维护一个从1开始的计数器，当计数器的数值小于 K 时，就重复计算下一排列并将计数器加一，当计数器数值为 k 时停止，此时计算出的排列就是第 K 个排列。

```
class Solution {
public:
	int maxOfK() {
		int result = 1;
		for(int i = 1; i < 10 ;i++) {
			result *= i;
		}
		return result;
	}
	string getString(int n) {
		int i = 1;
		string s = to_string(i);
		while(++i <= n) {
			s += to_string(i);
		}
		return s;
	}
	string getPermutation(int n, int k) {
		string s;
		if (k <= 0 || k > maxOfK()) {
			return s;
		}
		
		// 输入n,那么s="123...n"
		s = getString(n);
		int i = 1;
		while(i++ < k) {
		   // 通过 STL 函数来对 s 进行排列，这个排列是按照字典序进行的
			std::next_permutation(s.begin(),s.end());
		}
		return s;
	}
};
```

