# 在数组中找到出现次数大于 N/K 的数

## 找到出现次数大于 N/2 的数

### 题意描述

给定一个整形数组 arr， 打印其中出现次数大于一半的数，如果没有这样的数字，则打印提示信息。要求：时间复杂度 O(N), 空间复杂度 O(1)。



```
class Solution {
public:
	void printHalfNumber(vector<int> &arr) {
    	int cand = 0;
    	int times = 0;
    	for(int i = 0; i < arr.size();i++) {
        	if (times == 0) {
            	cand = arr[i];
            	times = 1;
        	} else if (arr[i] == cand) {
             	times++;
        	} else {
            	times--;
        	}
    	}
    	times = 0;
    	for(int i = 0; i < arr.size(); i++) {
        	if (arr[i] == cand) {
            	times++;
        	}
    	}
    	if (times > arr.size()/2) {
        	printf("%d\n", cand);
    	} else {
        	printf("No such num\n");
    	}
	}
};
```



## 找到出现次数大于 N/K 的数

### 过程

1.  声明一个哈希表，代表最后出现次数可能超过 N/K 的数字，以数字为键，以数字出现的次数为值。那么这个哈希表的大小最多为 K-1。
2.  然后遍历数组，判断当前数 arr[i] 是否在这个哈希表中。如果在，就把那个数字的出现次数加1。否则进入第3步。
3.  先看当前哈希表的大小是否已经为 K-1了。如果是的话，说明哈希表中存的已经是所有出现次数可能超过 N/K 的数字了，把这个哈希表中的所有键对应的值减1；如果不是，那么当前数可能是出现次数超过 N/K 的数，把它以及次数1 加入到哈希表中。
4.  最后遍历这个哈希表，对每一个键出现的次数进行判断，如果大于 N/K，就把这个数字打印出来。

```
class Solution {
public:
	void printKTimesNum(vector<int> &arr, int k) {
    	if (k < 2) {
         	printf("The k is invalid\n");
         	return;
    	}
    	unordered_map<int,int> map;
    	for(int i = 0; i < arr.size(); i++) {
        	if (map.find(arr[i]) != map.end()) {
            	map.insert({arr[i], map[arr[i]]+1});
        	} else {
             	if (map.size() == k-1) {
                	allMinusOne(map);
             	} else {
                	map.insert({arr[i],1});
             	}
        	}
    	}
    	
    	unordered_map<int,int> reals = getReals(arr, map);
    	bool hasNumber = false;
    	for(auto it = maps.begin(); it != maps.end();it++) {
       		if (it->second > arr.size() / k) {
             	hasNumber = true;
             	printf("%d\n", it->first);
       		}
    	}
    	printf(hasNumber?"":"No such num\n");
	}
	
	void allMinusOne(unordered_map<int,int> &map) {
    	for(auto it = map.begin(); it != map.end(); ) {
        	if (it->second == 1) {
            	it = map.erase(it);
        	} else {
            	it->second -= 1;
            	it++;
        	}
    	}
	}
	
	unordered_map<int,int> getReals(vector<int> &arr, unordered_map<int,int> map) {
    	unordered_map<int,int> result;
    	for(int i = 0; i < arr.size(); i++) {
        	int curNum = arr[i];
        	if (map.find(curNum) != map.end()) {
           		result.insert({curNum, map[curNum]+1});
        	} else {
             	result.insert({curNum, 1});
        	}
    	}
    	return result;
	}
};
```



