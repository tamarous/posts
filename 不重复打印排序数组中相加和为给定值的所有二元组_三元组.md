# 不重复打印排序数组中相加和为给定值的所有二元组/三元组

## 不重复打印排序数组中相加和为给定值的所有二元组

### 题意

给定一个排好序的数组arr 和整数k，不重复打印出 arr 中所有相加和为 k 的不降序二元组。例如，arr = [-8,-4,3,0,1,2,4,5,8,9], k = 10, 打印结果为：[1,9]和[2,8]

### 思路

由于整个数组是排好序的，因此我们可以从数组的两端出发，向中间收紧。用一个左指针left和一个右指针right，左指针指向数组的第一个元素，右指针指向数组的最后一个元素，然后比较 left+right 与 k 的大小：

-   如果 left + right == k，那么打印 left 和 right, 然后 left++, right--;
-   如果 left + right > k, 那么right--;
-   如果 left + right < k, 那么left++;

因为还有不重复的要求，因此我们还要加一个判断：如果 left 和 left-1 的值相同，那么就跳过, left++。

### 代码

```
class Solution {
public:
	void printUniquePair(vector<int> &arr, int k) {
     	if (arr.size() < 2) {
        	return;
     	}
     	int left = 0;
     	int right = arr.size()-1;
     	while(left < right) {
        	int result = arr[left] + arr[right];
        	if (result > k) {
             	right--;
        	} else if (result < k) {
             	left++;
        	} else {
        		if (left == 0 || arr[left-1] != arr[left]) {
                 	printf("[%d,%d]\n",arr[left],arr[right]);
        		}
             	 left++;
             	 right--;
        	}
     	}
	}	
}; 
```



## 不重复打印排序数组中相加和为给定值的所有三元组

### 题意

给定一个排好序的数组arr 和整数k，不重复打印出 arr 中所有相加和为 k 的不降序三元组。例如，arr = [-8,-4,3,0,1,2,4,5,8,9], k = 10, 打印结果为：[-4,5,9],[-3,4,9],[-3,5,8],[0,1,9],[0,2,8],[1,4,5]

### 思路

和求二元组类似，用三个指针，记为 left, mid, right, 然后对于每一个left，寻找和为 k - left的二元组。

### 代码

```
class Solution {
public:
	void printUniqueTriple(vector<int> &arr, int k) {
     	if (arr.size() < 3) {
        	return;
     	}
     	for(int i = 0; i < arr.size() - 2; i++) {
         	if (i == 0 || arr[i] != arr[i-1]) {
             	printRest(arr, i, i+1, arr.size()-1, k - arr[i]);
         	}
     	}
	}	
	void printRest(vector<int> &arr, int left, int mid, int right, int k) {
     	while(mid < right) {
         	int result = arr[mid] + arr[right];
         	if (result < k) {
             	mid++;
         	} else if (result > k) {
             	right--;
         	} else {
             	if (mid = left + 1 || arr[mid] != arr[mid-1]) {
                 	printf("[%d, %d, %d]\n",arr[left],arr[mid],arr[right]);
             	}
         	}
     	}
	}
}; 
```

