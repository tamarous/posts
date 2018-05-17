# LeetCode上几道和字符串有关的动态规划问题

## [Word Break](https://leetcode.com/problems/word-break)

### 题意

给出一个字符串s和一个词典，判断字符串s是否可以被空格切分成一个或多个出现在字典中的单词。 给出

s = **"leetcode"**，dict = **["leet","code"]**，返回 true 因为**"leetcode"**可以被空格切分成**"leet code"**。

## 思路

凡是和字符串有关的题，在应用动态规划方法时主要有两种思路：

* 用`dp[i][j]`表示子串`s[i,j]`满足某一要求
* 用`dp[i]`表示子串`s[0,i]`满足某一要求

本题中选择第二种思路，即用`dp[i]`表示字符串的(0, i)之间的子串是否可以被空格切分成一个或多个出现在字典中的单词，那么对于`dp[j]`(j > i)，如果`s[i,j]`代表的子串在词典中出现过，并且`dp[i] = true`，则有`dp[j] = true`。

### 实现

```
class Solution {
public:
    /*
     * @param s: A string
     * @param dict: A dictionary of words dict
     * @return: A boolean
     */
    bool wordBreak(string &s, unordered_set<string> &dict) {
        int size = s.size();
        vector<bool> dp(size+1,false);
        int maxlength = 0;
        for(auto it = dict.begin(); it != dict.end(); it++) {
            maxlength = maxlength < (*it).length()? (*it).length():maxlength;
        }
        dp[0] = true;
        for(size_t i = 1; i <= size; i++) {
        
        	// 在往前搜寻时子串(j,i)的长度肯定要小于等于词典中最长串的长度
        	// 如果不加这个判断，LeetCode可以通过，但LintCode会超时
            for(int j = i-1; j >= 0 && i - j <= maxlength; j--) {
                if (dp[j] && dict.find(s.substr(j, i-j)) != dict.end()) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[size];
    }
};
```



## [Word Break II](https://leetcode.com/problems/word-break-ii)

### 题意

给出一字串s和单词的字典dict，在字符串中增加空格来构建一个句子，并且所有单词都来自字典， 返回所有有可能的句子。 如

```
输入s = "catsanddog"，wordDict = ["cat", "cats", "and", "sand", "dog"]
输出
[
  "cats and dog",
  "cat sand dog"
]
```

### 思路

我们观察到，这道题很明显是需要判断`s[i,j]`是否满足某一条件，所以思路二比较符合。另外我们在[前一篇博客](http://www.tamarous.com/leetcode-backtrace-problems/)中也提到

> 如果题目要求所有符合某一条件的集合，那么就可以考虑用回溯法来解决。

因此本题我们可以使用二维动态规划，配合回溯法来解决。

### 实现

```
class Solution {
public:
    vector<string> wordBreak(string s, vector<string>& wordDict) {
    
    	
        vector<bool> f(s.length()+1, false);
        vector<vector<bool> > prev(s.length()+1, vector<bool>(s.length()));
        f[0] = true;
        
        for(size_t i = 1; i <= s.length(); i++) {
            for (int j = i-1; j >= 0; j--) {
                if (f[j] == true && find(wordDict.begin(), wordDict.end(), s.substr(j, i-j)) != wordDict.end()) {
                    f[i] = true;
                    prev[i][j] = true;
                }
            }
        }
        
        vector<string> result;
        vector<string> path;
        backtrace(s, prev, s.length(), path, result);
        return result;
        
    }
    
    // result是可行解的集合, path是一个可能的解
    void backtrace(string &s, vector<vector<bool> > &prev, int cur, vector<string> &path, vector<string> &result) {
        if (cur == 0) {
            string tmp;
            for (auto iter = path.rbegin(); iter != path.rend(); iter++) {
                tmp += *iter + " ";
            }
            tmp.erase(tmp.end()-1);
            result.push_back(tmp);
        }
        for(size_t i = 0; i < s.size(); i++) {
            if (prev[cur][i]) {
                path.push_back(s.substr(i,cur-i));
                backtrace(s, prev, i, path, result);
                path.pop_back();
            }
        }
    }
};
```

## [最长回文子串](https://leetcode.com/problems/longest-palindromic-substring/description/)

### 题意

给定一个字符串s，找出s中的最长回文子串。如输入s = "bacad"，返回"aca"。

### 思路

我们看到实例中的"aca"是位于s的中部的，因此思路二显然不太适合，所以我们可以想办法往思路一上凑凑。我们可以用`dp[j][i]`表示`s[j,i]`是否是一个回文串，那么计算dp的方法如下：

`dp[j][i] = (s[j] == s[i] && (i - j < 2 || dp[j+1][i-1])`

### 实现

```
class Solution {
public:
    string longestPalindrome(string s) {
        int size = s.size();
        bool f[size][size];
        fill_n(&f[0][0], size * size, false);
        size_t max_len = 1, start = 0;
        for(size_t i = 0; i < size; i++) {
            f[i][i] = true;
            for(size_t j = 0; j < i; j++) {
                f[j][i] = (s[j] == s[i] && (i == j+1 || f[j+1][i-1]));
                if (f[j][i] && max_len < (i-j+1)) {
                    max_len = i-j+1;
                    start = j;
                }
            }
        }
        return s.substr(start, max_len);
    }
};
```



