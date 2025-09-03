---
title: "06 — Linux内核数据结构 — 红黑树 — 遍历（代码）"
description: 
date: 2025-02-24
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - "数据结构"
    - "Linux内核"
---

- [1. **中序遍历**](#1-中序遍历)
- [2. **后序遍历**](#2-后序遍历)



二叉树遍历中，以根节点的顺序为准，例如“根 —> 左 —> 右”就是前序遍历，“左 —> 右 —> 根”就是后序遍历，从左到右一层一层的遍历就是层序遍历。Linux红黑树主要提供中序和后序两种方法。  

# 1. **中序遍历**  
左 —> 根 —> 右   
```c
struct rb_node *rb_first(const struct rb_root *);  //  中序遍历的最小节点，最左边
struct rb_node *rb_last(const struct rb_root *);   //  中序遍历的最大节点，最右边
struct rb_node *rb_next(const struct rb_node *);   //  中序遍历的后继节点
struct rb_node *rb_prev(const struct rb_node *);   //  中序遍历的前驱节点
```

---
**找到最小节点：**   
```c
struct rb_node *rb_first(const struct rb_root *root)
{
	struct rb_node	*n;

	n = root->rb_node;
	if (!n)
		return NULL;
	while (n->rb_left)  // 没有左孩子就对了
		n = n->rb_left;
	return n;
}
```

---
**找到最大节点：**  
```c
struct rb_node *rb_last(const struct rb_root *root)
{
	struct rb_node	*n;

	n = root->rb_node;
	if (!n)
		return NULL;
	while (n->rb_right)  // 没有右孩子就对了
		n = n->rb_right;
	return n;
}
```

---
**node的后继节点：** 
```c
struct rb_node *rb_next(const struct rb_node *node)
{
	struct rb_node *parent;

	if (RB_EMPTY_NODE(node))
		return NULL;

	if (node->rb_right) {
        // 1. 如果node有右子树，下一个节点就是右子树中最左节点。
		node = node->rb_right;   
		while (node->rb_left)
			node = node->rb_left;
		return (struct rb_node *)node;
	}

    // 2. 没有右子树，需要向上回溯，并更新当前节点以及其父节点，直到找到一个父节点，当前node节点是父节点的左孩子，这个父节点就是要找到后继节点，这是搜索二叉树的排序性质。  
	while ((parent = rb_parent(node)) && node == parent->rb_right)
		node = parent;

	return parent;
}
```

---  

**前驱节点：**  
```c
struct rb_node *rb_prev(const struct rb_node *node)
{
	struct rb_node *parent;

	if (RB_EMPTY_NODE(node))
		return NULL;

	if (node->rb_left) {
        // 1. 如果node有左子树，左子树中的最右节点就是后继节点
		node = node->rb_left;
		while (node->rb_right)
			node = node->rb_right;
		return (struct rb_node *)node;
	}

    // 2. node没有左子树，需要向上回溯，并更新当前节点以及其父节点，直到找到一个父节点，当前的node节点是父节点的右节点，这个父节点就是要找到后继节点。  
	while ((parent = rb_parent(node)) && node == parent->rb_left)
		node = parent;

	return parent;
}
```

# 2. **后序遍历**
左 —> 右 —> 根  
```c
struct rb_node *rb_first_postorder(const struct rb_root *);  // 获取红黑树后序遍历的第一个节点，要符合“左 —> 右 —> 根”的规则
struct rb_node *rb_next_postorder(const struct rb_node *);   // 获取当前节点在后序遍历中的后继节点
```

---  

**获取红黑树后序遍历的第一个节点：** 和中序遍历是一个节点。
```c
struct rb_node *rb_first_postorder(const struct rb_root *root)
{
	if (!root->rb_node)
		return NULL;

	return rb_left_deepest_node(root->rb_node); // 找到树左侧的最深节点。
}
```

---  

**后序遍历中的后继节点**  
```c
struct rb_node *rb_next_postorder(const struct rb_node *node)
{
	const struct rb_node *parent;
	if (!node)
		return NULL;
	parent = rb_parent(node);

	if (parent && node == parent->rb_left && parent->rb_right) {
        // node是父节点的左孩子，由于 “左 —> 右 —> 根” 的要求，所以到父节点的右子树中，寻找左侧的最深节点
		return rb_left_deepest_node(parent->rb_right);
	} else {
        // 1. 父节点是NULL，说明node是根节点，此时返回NULL。  
        // 2. 父节点没有右子树，后继节点就是父节点。  
        // 3. node是父节点的右孩子，后继节点就是父节点。  
		return (struct rb_node *)parent;
    }
}
```

---

**给定一个节点，找到该节点左侧的最深节点**  
```c
static struct rb_node *rb_left_deepest_node(const struct rb_node *node)
{
    // 遍历中，如果节点有左子树，那就进入左子树，如果没有左子树，那就进入右子树，直到找到一个没有孩子的节点
	for (;;) {
		if (node->rb_left)
			node = node->rb_left;
		else if (node->rb_right)
			node = node->rb_right;
		else
			return (struct rb_node *)node;
	}
}
```  

内核中没有用搜到后序遍历的使用，等以后需要的时候再去研究。   


