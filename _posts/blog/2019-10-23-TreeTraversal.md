---
layout: post
title: "Tree Traversal"
modified:
categories: blog
excerpt:
tags: [leetcode]
image:
  feature:
date: 2019-10-23

---

Tree traversal algorithms: pre_order, in_order, post_order

```c++
void preorder_traversal_iteratively(TreeNode* root)
{
	if (root == 0)
		return;
	stack<TreeNode*> s;
	s.push(root);
	cout << root->val << ' '; // visit root
	TreeNode* last_pop = root;
	while (!s.empty())
	{
		TreeNode* top = s.top();
		if (top->left != 0 && top->left != last_pop && top->right != last_pop) // push_left
		{
			s.push(top->left);
			cout << top->left->val << ' '; // visit top->left
		}
		else if (top->right != 0 && top->right != last_pop && (top->left == 0 || top->left == last_pop)) // push_right
		{
			s.push(top->right);
			cout << top->right->val << ' '; // visit top->right
		}
		else // pop
		{
			s.pop();
			last_pop = top;
		}
	}
}

void inorder_traversal_iteratively(TreeNode* root)
{
	if (root == 0)
		return;
	stack<TreeNode*> s;
	s.push(root);
	TreeNode* last_pop = root;
	while (!s.empty())
	{
		TreeNode* top = s.top();
		if (top->left != 0 && top->left != last_pop && top->right != last_pop) // push_left
		{
			s.push(top->left);
		}
		else if (top->right != 0 && top->right != last_pop && (top->left == 0 || top->left == last_pop)) // push_right
		{
			s.push(top->right);
			cout << top->val << ' '; // visit top
		}
		else // pop
		{
			s.pop();
			last_pop = top;
			if (top->right == 0)
				cout << top->val << ' '; // visit top
		}
	}
}

void postorder_traversal_iteratively(TreeNode* root)
{
	if (root == 0)
		return;
	stack<TreeNode*> s;
	s.push(root);	
	TreeNode* last_pop = root;
	while (!s.empty())
	{		
		TreeNode* top = s.top();
		if (top->left != 0 && top->left != last_pop && top->right != last_pop) // push_left
		{
			s.push(top->left);
		}
		else if (top->right != 0 && top->right != last_pop && (top->left == 0 || top->left == last_pop)) // push_right
		{
			s.push(top->right);
		}
		else // pop
		{
			s.pop();
			last_pop = top;
			cout << top->val << ' '; // visit top
		}
	}
}
```