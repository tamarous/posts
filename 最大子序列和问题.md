# 最大子序列和问题

最大子序列和问题是「数据结构与算法分析」一书开篇提出的问题，问题的描述是这样的：给定N个整数，求其中子序列之和的最大值。

那么最简单的方法，当然也是最低效的一种算法便是暴力求解法了，这种方法的思路是对数组进行双重遍历，将每一种可能性都计算出来与当前的最大值进行比较，如果大于该最大值，就将最大值更新为新计算出来的值。代码如下：

    int maxSubSequence(const int array[], int N) {
        int thisSum,maxSum,i,j,k;
        thisSum = maxSum = 0;
        for (i = 0; i < N;i++) {
            for(j = i;j < N;j++) {
                thisSum = 0;
                for (k = i;k <= j;k++) {
                    thisSum += array[k];
                }
                if (thisSum > maxSum) {
                    maxSum = thisSum;
                }
            }
        }
        return maxSum;
    }
    
易知该算法的时间复杂度为O(N^3)。

显然时间复杂度这么高的算法是不可以被接受的，因此有必要寻找一个更为简单高效的方法。
书中紧接着给出了一个递归算法，该算法的时间复杂度为O(NlogN)，思路如下：首先我们知道，一个数组的最大子序列，要么出现在这个数组的前一半，要么出现在这个数组的后一半，要么横跨这个数组的左右两半，由两半的最大值求和得到。因此我们就可以缩小这个问题的规模，将这个问题转化为：**求前后两半以及中间数组的子序列的最大值，然后将这三个值的最大值作为整个数组的最大值**。

实现如下：
    
    class Solution {
    public:
        static int maxSubSum(const int a[], int left, int right) {
            int leftSum,rightSum,maxLeftSum,maxRightSum,leftPartSum,rightPartSum;
            int center,i;
        
            // 说明数组中只有一个元素，因此直接返回它本身或者是0
            if (left == right) {
                return a[left];
            }
        
            center = (left+right)/2;
            leftSum = maxSubSum(a,left,center);
            rightSum = maxSubSum(a,center+1,right);
        
            leftPartSum = 0;
            maxLeftSum = INT_MIN;
            for ( i = center; i >= left; i--) {
                leftPartSum += a[i];
                if (leftPartSum >= maxLeftSum) {
                    maxLeftSum = leftPartSum;
                }
            }
        
            rightPartSum = 0;
            maxRightSum = INT_MIN;
            for ( i = center+1;i <= right;i++)  {
                rightPartSum += a[i];
                if (rightPartSum >= maxRightSum) {
                    maxRightSum = rightPartSum;
                }
            }
        
            return max(leftSum,max(rightSum,maxLeftSum+maxRightSum));
        }
        
        int maxSubsequenceSum(const int a[], int N) {
            return maxSubSum(a,0,N-1);
        }
    };

该算法思路清晰，实现也比较简单，可以说是一种很不错的算法了。

不过，如果使用动态规划算法的话，那么可以在 O(N) 的时间复杂度内解决此问题：

    class Solution {
    public:
        int FindGreatestSumOfSubArray(vector<int> array) {
            int size = array.size();
            if (size == 0) {
                return 0;
            }
            if (size == 1) {
                return array[0];
            }
            vector<int> dp(size);
            dp[0] = array[0];
            for(int i = 1; i < size; i++) {
                if (dp[i-1] <= 0) {
                    dp[i] = array[i];
                } else {
                    dp[i] = dp[i-1] + array[i];
                }
            }
            int result = INT_MIN;
            for(int i = 0; i < size; i++) {
                result = max(result,dp[i]);
            }
            return result;
        }
    };
    
我们声明一个长度和输入数组相同的数组dp, dp[i] 表示以第 i 个数作为结尾的子数组的最大和。那么，如果当第 i-1 个数作为结尾的子数组的和小于0，即 dp[i-1] <= 0，如果把这个负数和 array[i] 相加，那么结果会比 array[i] 还小，因此以第 i 个数作为结尾的子数组的最大和应该为 array[i]；如果 dp[i-1] > 0，那么它与 array[i] 相加得到的就是以第 i 个数作为结尾的子数组的最大和。

    












