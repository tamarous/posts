# LeetCode-347-Top K Frequent Elements
这道题的题意是：给出一个数组和一个数字k，返回一个由输入数组中出现次数排在前K位的元素组成的数组。
思路：由题意，显然我们要将数组中的每个元素出现的次数记录下来，而这可以用哈希表来实现。在C++的标准库里，unordered_map是实现这一功能的库。于是我们可以先将每个元素出现的次数记录下来，然后把所有次数装进一个vector里面并排序。之后我们就可以从这个vector的末尾出发，去哈希表中找对应的键是什么，如果找到的话就把键加入到最终的结果数组中，直到遍历完vector的后K个元素。
代码如下：

```
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        vector<int> result;
        unordered_map<int,int> set;
        for(int i = 0; i < nums.size(); i++) {
            if (set.find(nums[i]) != set.end()) {
                set[nums[i]] ++;
            } else {
                set.insert({nums[i],1});
            }
        }
        vector<int> indexs;
        for(auto &index: set) {
            indexs.push_back(index.second);
        }
        sort(indexs.begin(), indexs.end());
        size_t j = indexs.size();
        for(int i = 0; i < k; i++) {
            int x = indexs[j-1];
            j--;
            unordered_map<int,int>::iterator it = set.begin();
            for(; it != set.end();it++) {
                if (it->second == x) {
                    result.push_back(it->first);
                    set.erase(it);
                    break;
                }
            }
        }
        return result;
    }
};
```


