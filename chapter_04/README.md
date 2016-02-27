# 树和图

树和图是非线性数据结构。

树和图的题目有很多 ambiguous details，面试时最好向面试官问清以下问题：

1. 二叉树还是二叉搜索树？BST 有很好的性质，往往更好处理。
2. 平衡还是非平衡？非平衡的情况下，一定要注意 worst case 的算法复杂度分析。
3. Full and Complete。即满二叉树和完全二叉树。满二叉树是完全二叉树的特例。

## 二叉树遍历

先序、中序、后序这三种基本遍历的实现。层次遍历的实现。递归与非递归实现都要掌握。

## 树的平衡化

红黑树，AVL 树。属于比较复杂的数据结构，面试中直接要求实现的可能性小，但是要熟悉基本操作和性质。

## Trie

Trie 即字典树/单词查找树，hash 的思想，每个从根节点出发的路径表示一个单词。

## 图的遍历

DFS 和 BFS，DFS 往往容易用递归实现，BFS 通常借助队列实现。

---

## 题目 4.1

> 判断一棵二叉树是否为平衡树，平衡定义为树中任意节点的左右子树高度相差不超过 1 。

## 解答

利用树的递归定义。

```cpp
class Balance {
public:
    bool isBalance(TreeNode* root) {
        // write code here
        if (!root)
            return true;
        
        int left_d = depth(root->left);
        int right_d = depth(root->right);
        if (abs(left_d - right_d) > 1)
            return false;
        return isBalance(root->left) && isBalance(root->right);
    }
    
private:
    int depth(TreeNode *root) {
        if (!root)
            return 0;
        
        return max(depth(root->left), depth(root->right)) + 1;
    }
};
```

上面代码的缺点是需要重复遍历同一节点多次（求每个节点对应子树的深度时会重复遍历一些节点），效率不高。

如何优化使得每个节点只被遍历一次？利用后序遍历。后序遍历二叉树，在遍历某个节点的左右子树之后，可以根据其左右子树的深度判断它是不是平衡的，同时得到当前节点的深度。

```cpp
class Balance {
public:
    bool isBalance(TreeNode* root) {
        // write code here
        int depth = 0;
        return is_balanced(root, depth);
    }
    
private:
    bool is_balanced(TreeNode *root, int &depth) {
        if (root == NULL) {
            depth = 0;
            return true;
        }
        
        int left_depth = 0, right_depth = 0;
        if (is_balanced(root->left, left_depth) && 
            is_balanced(root->right, right_depth)) {
            if (abs(left_depth - right_depth) <= 1) {
                depth = max(left_depth, right_depth) + 1;
                return true;
            }
        }
        return false;
    }
};
```

---

## 题目 4.2

> Given a directed graph, design an algorithm to find out whether there is a route between two nodes.

> 给定一个有向图，设计算法判断两节点之间是否存在路径。


## 解答

给定一个有向图和起点终点，判断从起点开始是否存在一条路径可以到达终点。考察图的遍历，从图的起点开始遍历图，如果能访问到终点， 则说明起点与终点间存在路径。

这题面试时要问清楚图用什么来表示，邻接矩阵还是邻接表。这里用邻接矩阵来表示图。

利用 BFS 遍历图，遍历过程中进行判断即可：

```cpp
bool hasRoute(int src, int dst, bool graph[][MAX_SIZE])
{
    queue<int> Q;
    bool visited[MAX_SIZE];
    memset(visited, false, sizeof(visited));

    Q.push(src);
    visited[src] = true;

    int tmp;
    while (!Q.empty())
    {
        tmp = Q.front();
        Q.pop();
        for (int i = 0; i < MAX_SIZE; ++i)
        {
            if (graph[tmp][i] && !visited[i])
            {
                visited[i] = true;
                Q.push(i);

                if (i == dst)
                    return true;
            }
        }
    }

    return false;
}
```

---

## 题目 4.3 

> Given a sorted (increasing order) array, write an algorithm to create a binary tree with minimal height.

## 解答

先在纸上比划一下，发现要使构建出来的二叉树高度最小，那么对于树种任意节点，其左子树和右子树的节点数量应该差不多。最简单的方法就是取得有序数组中中间那个数，然后把小于它的放在它的左子树， 大于它的放在它的右子树。不断地递归操作即可构造这样一棵最小高度二叉树。

如果只是问最小高度二叉树的高度的话，不需要显式构造树：

```cpp
class MinimalBST {
public:
    int buildMinimalBST(vector<int> vals) {
        // write code here
        int n = vals.size();
        
        return (int)(log(n) / log(2)) + 1;
    }
};
```

递归构造代码：

```cpp
void create_min_height_bst(TreeNode *&node, const vector<int> &nums, int first, int last) {
    if (first > last)
        return;

    if (node == NULL)
    {
        int mid = (first + last) / 2;
        node = new TreeNode(nums[mid]);
        node->left = NULL;
        node->right = NULL;

        create_min_height_bst(node->left, nums, first, mid - 1);
        create_min_height_bst(node->right, nums, mid + 1, last);
    }
}

TreeNode* create_min_height_bst(const vector<int> &nums) {

    TreeNode *root = NULL;
    create_min_height_bst(root, nums, 0, nums.size() - 1);
    return root;
}
```

---

## 题目 4.4

> Given a binary search tree, design an algorithm which creates a linked list of all the nodes at each depth (i.e., if you have a tree with depth D, you’ll have D linked lists).

## 解答

这里先给出将某一层节点连成链表的代码：

```cpp
class TreeLevel {
public:
    ListNode* getTreeLevel(TreeNode* root, int dep) {
        // write code here
        if (root == NULL || dep <= 0)
            return 0;

        queue<pair<TreeNode*, int>> Q;
        Q.push(make_pair(root, 1));

        ListNode dummy_head(-1);
        ListNode *p = &dummy_head;

        while (!Q.empty())
        {
            TreeNode *node = Q.front().first;
            int cur_level = Q.front().second;
            Q.pop();

            if (cur_level == dep)
            {
                p->next = new ListNode(node->val);
                p = p->next;
            }

            if (cur_level > dep)
                break;

            if (node->left)
                Q.push(make_pair(node->left, cur_level + 1));
            if (node->right)
                Q.push(make_pair(node->right, cur_level + 1));
        }

        return dummy_head.next;
    }
};
```

---

## 题目 4.5

> Implement a function to check if a binary tree is a binary search tree.


## 解答

bst 的中序遍历一定是有序的，对二叉树进行中序遍历，记录前一个节点，和当前节点进行比较。

```cpp
class Checker {
public:
    bool checkBST(TreeNode* root) {
        // write code here
        int last = INT_MIN;
        return inorder(root, last);
    }
    
private:
    bool inorder(TreeNode *root, int &last) {
        if (!root)
            return true;
        
        bool left_is_bst = inorder(root->left, last);
        if (!left_is_bst)
            return false;
        
        if (root->val < last)
            return false;
        last = root->val;
        
        return inorder(root->right, last);
    }
};
```

另一种递归思路，根据bst定义，所有左子树的节点小于根节点，所有右子树的节点大于根节点，并且左右子树也是二叉查找树。这个思路很简单，但是很容易写错，要注意递归状态的传递。在递归的过程中，我们需要传递两个参数（当前根节点对应的二叉树的所有节点的最大值和最小值），同时不断的更新这两个参数，如果当前节点的值不在这两个数范围中，则直接返回false，否则接着递归便可。

```cpp
class Checker {
public:
    bool checkBST(TreeNode* root) {
        // write code here
        return check(root, INT_MIN, INT_MAX);
    }
    
private:
    bool check(TreeNode *root, int min, int max) {
        if (!root)
            return true;
        if (root->val < min || root->val > max)
            return false;
        
        return check(root->left, min, root->val) && check(root->right, root->val, max);
    }
};
```

---

## 题目 4.6

> Write an algorithm to find the ‘next’ node (i.e., in-order successor) of a given node in a binary search tree where each node has a link to its parent.

## 解答

最简单的方法，先得到 bst 的中序遍历序列，然后直接找给定节点的后继即可。

如果现在要求不是bst，而是普通的二叉树，可以利用中序遍历实现。假设节点的值没有重复，只要在中序遍历的过程中，当某个节点的值等于p，则告诉递归者下一个要遍历的节点便是我们要找的节点，这个需要有个信号在多次递归之间进行传递消息，c++里用引用完全可以实现，只需要传递一个bool类型的引用便可。

```cpp
class Successor {
public:
    int findSucc(TreeNode* root, int p) {
        // write code here
        bool flag = false;
        
        return findSucc(root, p, flag);
    }
    
private:
    int findSucc(TreeNode *root, int p, bool &flag) {
        if (!root)
            return -1;
        
        int left_ret = findSucc(root->left, p, flag);
        if (left_ret != -1)
            return left_ret;
        
        if (flag == true)
            return root->val;
        
        if (root->val == p)
            flag = true;
        
        return findSucc(root->right, p, flag);
    }
};
```

---

## 题目 4.7

> Design an algorithm and write code to find the first common ancestor of two nodes in a binary tree. Avoid storing additional nodes in a data structure. NOTE: This is not necessarily a binary search tree.

## 解答

首先假设二叉树是 BST。BST 的性质：位于左子树的节点值都比父节点值小，位于右子树的节点值都比父节点值大。我们只需要从树的根节点开始和两个输入节点进行比较。如果当前节点的值比两个节点的值都大，那么最低公共祖先一定在当前节点的左子树中，下一步遍历当前节点的左子节点。如果当前节点的值比两个节点的值都小，那么下一步遍历当前节点的右子节点。这样在树中从上到下找到的第一个在两个输入节点的值之间的节点就是要求的最低公共祖先。

```cpp
// https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/

/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (!root)
            return NULL;
            
        if (root->val > p->val && root->val > q->val) 
            return lowestCommonAncestor(root->left, p, q);
        if (root->val < p->val && root->val < q->val)
            return lowestCommonAncestor(root->right, p, q);
            
        return root;
    }
};
```

非递归版本：

```cpp
class Solution { 
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        TreeNode* cur = root;
        while (true) {
            if (cur == NULL) return NULL;
            if (p -> val < cur -> val && q -> val < cur -> val)
                cur = cur -> left;
            else if (p -> val > cur -> val && q -> val > cur -> val)
                cur = cur -> right;
            else return cur; 
        }
    }
};
```

如果是普通的二叉树，不是BST怎么办？如果树中的节点含有指向父节点的指针，那么可以转化为求两个链表的第一个公共节点来做。如果树节点没有指向父节点的指针呢？这里借助递归（DFS）可以写出非常简洁的代码：

```cpp
// https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/

/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == p || root == q || root == NULL)
            return root;
        
        TreeNode *left = lowestCommonAncestor(root->left, p, q);
        TreeNode *right = lowestCommonAncestor(root->right, p, q);
        
        return left && right ? root : (left ? left : right);
    }
};
```

这是 CTCI 官方解答使用的方法，尽管没有显式开辟空间，但由于使用了递归，实际空间复杂度为 O(log(n))。

不用递归的话：

1. 找从根节点到 node1 的路径，存储在一个数组中
2. 找从根节点到 node2 的路径，存储在一个数组中
3. 遍历两条路径，直到遇到一个不同的节点，则前面的那个为最低公共祖先。

时间复杂度和空间复杂度仍然都为 O(log(n))，最坏情况可达 O(n)。

这道题还有一个版本：

> 有一棵无穷大的满二叉树，其结点按根结点一层一层地从左往右依次编号，根结点编号为1。现在有两个结点a，b。请设计一个算法，求出a和b点的最近公共祖先的编号。
给定两个int a,b。为给定结点的编号。请返回a和b的最近公共祖先的编号。注意这里结点本身也可认为是其祖先。

满二叉树的子节点与父节点之间的关系：root = child / 2。如果 a != b ，就令其中较大的数除以2，直到 a == b。这样就得到了最低公共祖先。

```cpp
class LCA {
public:
    int getLCA(int a, int b) {
        // write code here
        while (a != b) {
            if (a > b)
                a /= 2;
            else
                b /= 2;
        }
        
        return a;
    }
};
```

---

## 题目 4.8

> You have two very large binary trees: T1, with millions of nodes, and T2, with hundreds of nodes. Create an algorithm to decide if T2 is a subtree of T1. 

## 解答

利用树的递归定义。

```cpp
/**
 * Definition of TreeNode:
 * class TreeNode {
 * public:
 *     int val;
 *     TreeNode *left, *right;
 *     TreeNode(int val) {
 *         this->val = val;
 *         this->left = this->right = NULL;
 *     }
 * }
 */
class Solution {
public:
    /**
     * @param T1, T2: The roots of binary tree.
     * @return: True if T2 is a subtree of T1, or false.
     */
    bool isSubtree(TreeNode *T1, TreeNode *T2) {
        // write your code here
        if (T2 == NULL)
            return true;
        
        if (T1 == NULL)
            return false;
        
        bool result = false;
        if (T1->val == T2->val) 
            result = is_same_tree(T1, T2);
            
        if (!result)
            result = isSubtree(T1->left, T2);
        if (!result)
            result = isSubtree(T1->right, T2);
        
        return result;
    }
    
private:
    bool is_same_tree(TreeNode *T1, TreeNode *T2) {
        if (T1 == NULL && T2 == NULL)
            return true;
        
        if (T1 == NULL && T2 != NULL ||
            T1 != NULL && T2 == NULL)
            return false;
        
        if (T1->val != T2->val)
            return false;
            
        return is_same_tree(T1->left, T2->left) && 
            is_same_tree(T1->right, T2->right);
    }
};
```

---

## 题目 4.9

> You are given a binary tree in which each node contains a value. Design an algorithm to print all paths which sum up to that value. Note that it can be any path in the tree - it does not have to start at the root.

## 解答

求二叉树中和为指定值的路径。注意这里路径可以从任意节点开始，而不一定要从根节点开始。

先考虑路径只能从根节点开始的情况。

```cpp
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    vector<vector<int> > FindPath(TreeNode* root,int expectNumber) {
		vector<vector<int>> paths;
        vector<int> path;
        
        find_path(root, expectNumber, path, paths);
        
        return paths;
    }
    
private:
    void find_path(TreeNode *root, int sum, vector<int> &path, vector<vector<int> > &paths) {
        if (!root)
            return;
        
        if (root->left == NULL && root->right == NULL) {
            if (root->val == sum) {
                path.push_back(root->val);
                paths.push_back(path);
                path.pop_back();
            }
        }
        
        if (root->left) {
            path.push_back(root->val);
            find_path(root->left, sum - root->val, path, paths);
            path.pop_back();
        }
        
        if (root->right) {
            path.push_back(root->val);
            find_path(root->right, sum - root->val, path, paths);
            path.pop_back();
        }
    }
};
```

注意状态扩展与撤销，尤其是 `path.pop_back()`。

如果不要求路径从根节点开始，递归左右子树时只要再考虑不向路径中加入当前节点的情况即可：

```cpp
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    vector<vector<int> > FindPath(TreeNode* root,int expectNumber) {
		vector<vector<int>> paths;
        vector<int> path;
        
        find_path(root, expectNumber, path, paths);
        
        return paths;
    }
    
private:
    void find_path(TreeNode *root, int sum, vector<int> &path, vector<vector<int> > &paths) {
        if (!root)
            return;
        
        if (root->left == NULL && root->right == NULL) {
            if (root->val == sum) {
                path.push_back(root->val);
                paths.push_back(path);
                path.pop_back();
            }
        }
        
        if (root->left) {
            path.push_back(root->val);
            find_path(root->left, sum - root->val, path, paths);
            path.pop_back();
            
            find_path(root->left, sum, path, paths);
        }
        
        if (root->right) {
            path.push_back(root->val);
            find_path(root->right, sum - root->val, path, paths);
            path.pop_back();
            
            find_path(root->right, sum, path, paths);
        }
    }
};
```
