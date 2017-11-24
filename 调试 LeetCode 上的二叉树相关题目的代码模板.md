# 调试 LeetCode 上的二叉树相关题目的代码模板

在 LeetCode 上刷题时，只需要提供一个可以完成功能的函数就行了，但是这给我们的调试带来了不便。如果在出错时我们可以在本地 IDE 上调试我们的代码，那么就可以快速定位出问题所在。以下是一份可以用来直接运行的代码模板，针对的是LeetCode 上**树**这部分的题目。

    #include <iostream>
    #include <algorithm>
    #include <vector>
    #include <queue>
    #include <sstream>
    
    using namespace std;
    struct TreeNode {
        int val;
        TreeNode *left;
        TreeNode *right;
        TreeNode(int x): val(x), left(NULL), right(NULL) {}
    };
    
    
    class Solution {
    public:
        TreeNode *func(TreeNode *root) {
            return NULL;
        }    
    };
    
    void trimLeftTrailingSpaces(string &input) {
        input.erase(input.begin(), find_if(input.begin(), input.end(), [](int ch) {
            return !isspace(ch);
        }));
    }
    
    TreeNode* stringToTreeNode(string input) {
        input = input.substr(1, input.length() - 2);
        if (!input.size()) {
            return nullptr;
        }
        
        string item;
        stringstream ss;
        ss.str(input);
        
        getline(ss, item, ',');
        TreeNode* root = new TreeNode(stoi(item));
        queue<TreeNode*> nodeQueue;
        nodeQueue.push(root);
        
        while (true) {
            TreeNode* node = nodeQueue.front();
            nodeQueue.pop();
            
            if (!getline(ss, item, ',')) {
                break;
            }
            
            trimLeftTrailingSpaces(item);
            if (item != "null") {
                int leftNumber = stoi(item);
                node->left = new TreeNode(leftNumber);
                nodeQueue.push(node->left);
            }
            
            if (!getline(ss, item, ',')) {
                break;
            }
            
            trimLeftTrailingSpaces(item);
            if (item != "null") {
                int rightNumber = stoi(item);
                node->right = new TreeNode(rightNumber);
                nodeQueue.push(node->right);
            }
        }
        return root;
    }
    
    string treeNodeToString(TreeNode* root) {
        string output = "";
        if (root == nullptr) {
            return output;
        }
        
        queue<TreeNode*> q;
        q.push(root);
        while(!q.empty()) {
            TreeNode* node = q.front();
            q.pop();
            
            if (node == nullptr) {
                output += "null, ";
                continue;
            }
            
            output += to_string(node->val) + ", ";
            q.push(node->left);
            q.push(node->right);
        }
        return "[" + output.substr(0, output.length() - 2) + "]";
    }
    
    int main() {
        string line;
        while (getline(cin, line)) {
            TreeNode* root = stringToTreeNode(line);
            
            TreeNode* ret = Solution().func(root);
            
            string out = treeNodeToString(ret);
            cout << out << endl;
        }
        return 0;
    }



