# LeetCode-48-Rotate Image

题意：给定一个 N*N 的矩阵，将这个矩阵顺时针旋转90°。

例如，输入：

```
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
] 
```

输出

```
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```



## 思路一 分圈处理
一种思路是类似于“转圈打印矩阵”，分圈对矩阵进行处理。

```
class Solution {
public:
	void rotateMatrix(vector<vector<int> > &matrix) {
        int tR = 0;
        int tC = 0;
        int dR = matrix.size() - 1;
        int dC = matrix[0].size() - 1;
        while(tR < dR) {
          	rotateEdge(matrix, tR++, tC++, dR++, dC++);
        }
	}
	
	void rotateEdge(vector<vector<int> > &matrix, int tR, int tC, int dR, int dC) {
    	int times = dC - tC;
    	int temp = 0;
    	for(int i = 0; i < times; i++) {
        	temp = m[tR][tC+i];
        	m[tR][tC+i] = m[dR - i][tC];
        	m[dR - i][tC] = m[dR][dC - i];
        	m[dR][dC - i] = m[tR+i][dC];
        	m[tR+i][dC] = temp;
    	}
	}
};
```



## 思路二 逆序交换

首先我们将输入逆序，也就是将

```
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
] 
```

逆序为

```
[
  [7,8,9],
  [4,5,6],
  [1,2,3]
] 
```

之后再将主对角线上的元素交换一下，就可以得到旋转90°后的矩阵了。

```
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
] 
```

代码如下：

```
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        std::reverse(matrix.begin(), matrix.end());
        for(int i = 0 ; i < matrix.size(); i++) {
            for (int j = i+1; j < matrix.size(); j++) {
                swap(matrix[i][j], matrix[j][i]);
            }
        }
    }
};
```

