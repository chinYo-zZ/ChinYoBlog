---
title: C++ 算法模版
date: 2024-08-07 15:09:24
tags: C++
categories: C++
sitemap: true
---

 - [AVL树](#avl树)
 - [二分查找](#二分查找)

 ## AVL树
```C++ 
#include <iostream>
#include <string>
#include <stack>
#include <algorithm>
#include <vector>
#include <utility>
#include <queue>
using namespace std;

/* AVL 树节点类 */
struct TreeNode
{
	int Val{}; // 节点值
	int height = 0; // 节点高度
	TreeNode* left{};
	TreeNode* right{};
	TreeNode() = default;
	explicit TreeNode(int x) :Val(x) {
	}
};
/* 获取节点高度 */
int Height(TreeNode* node) {
	return node == nullptr ? -1 : node->height;
}
/* 更新节点高度 */
void UpdateHeight(TreeNode* node) {
	node->height = max(Height(node->left), Height(node->right)) + 1;
}

/* 获取平衡因子 */
int BalanceFactor(TreeNode* node) {
	if (node == nullptr) {
		return 0;
	}
	return Height(node->left) - Height(node->right);
}

/* 右旋操作 */
TreeNode* RightRotate(TreeNode* node) {
	TreeNode* child = node->left;
	TreeNode* grandChild = child->right;

	child->right = node;
	node->left = grandChild;

	UpdateHeight(node);
	UpdateHeight(child);
	return child;
}

/* 左旋操作 */
TreeNode* LeftRotate(TreeNode* node) {
	TreeNode* child = node->right;
	TreeNode* grandChild = child->left;

	child->left = node;
	node->right = grandChild;

	UpdateHeight(node);
	UpdateHeight(child);
	return child;
}

/* 执行旋转操作，使该子树重新恢复平衡 */
TreeNode* Rotate(TreeNode* node) {
	int _balancefactory = BalanceFactor(node);
	if (_balancefactory > 1) {
		if (BalanceFactor(node->left) >= 0) {
			return RightRotate(node);
		}
		else
		{
			node->left = LeftRotate(node->left);
			return RightRotate(node);
		}
	}
	if (_balancefactory < -1) {
		if (BalanceFactor(node->right) <= 0) {
			return LeftRotate(node);
		}
		else
		{
			node->right = RightRotate(node->right);
			return LeftRotate(node);
		}
	}

	return node;
}

TreeNode* root;
TreeNode* InsertHelper(TreeNode* node, int val);
TreeNode* RemoveHelper(TreeNode* node, int val);

void Insert(int val) {
	root = InsertHelper(root, val);
}

TreeNode* InsertHelper(TreeNode* node, int val) {
	if (node == nullptr) {
		return new TreeNode(val);
	}
	if (val < node->Val) {
		node->left == InsertHelper(node->left, val);
	}
	else if (val > node->Val) {
		node->right = InsertHelper(node->right, val);
	}
	else
	{
		return node;
	}
	UpdateHeight(node);
	node = Rotate(node);

	return node;
}

void Remove(int val) {
	root = RemoveHelper(root, val);
}

TreeNode* RemoveHelper(TreeNode* node, int val) {
	if (node == nullptr) {
		return nullptr;
	}
	if (val < node->Val) {
		node->left = RemoveHelper(node->left, val);
	}
	else if (val > node->Val) {
		node->right = RemoveHelper(node->right, val);
	}
	else
	{
		if (node->left == nullptr || node->right == nullptr) {
			TreeNode* child = node->left != nullptr ? node->left : node->right;
			if (child == nullptr) {
				delete node;
				return nullptr;
			}
			else
			{
				delete node;
				node = child;
			}
		}
		else
		{
			TreeNode* temp = node->right;
			while (temp->left != nullptr) {
				temp = temp->left;
			}
			int tempVal = temp->Val;
			node->right = RemoveHelper(node->right,temp->Val);
			node->Val = tempVal;
		}
	}

	UpdateHeight(node);
	node = Rotate(node);

	return node;
}

/* 查找节点 */
TreeNode* Search(int num) {
	TreeNode* cur = root;
	// 循环查找，越过叶节点后跳出
	while (cur != nullptr) {
		// 目标节点在 cur 的右子树中
		if (cur->Val < num)
			cur = cur->right;
		// 目标节点在 cur 的左子树中
		else if (cur->Val > num)
			cur = cur->left;
		// 找到目标节点，跳出循环
		else
			break;
	}
	// 返回目标节点
	return cur;
}

int main() {
	root = new TreeNode(6);

	Insert(7);
	Insert(54);
	Insert(3);
	Insert(75);

	TreeNode* res = Search(54);
	return 0;
}
```
## 二分查找
```C++
#include <iostream>
#include <vector>
using namespace std;
int BinarySearch(vector<int> &nums, int Target)
{
    int i = 0, j = nums.size() - 1;
    while (i <= j)
    {
        int mid = i + (j - i) / 2;
        if (nums[mid] < Target)
        {
            i = mid + 1;
        }
        else if (nums[mid] > Target)
        {
            j = mid - 1;
        }
        else
        {
            return mid;
        }
    }
    return -1;
}

int main()
{
    vector<int> vec{5, 10, 15, 20, 25, 30, 35, 40, 45, 50};
    cout << BinarySearch(vec, 30);
    return 0;
}
```