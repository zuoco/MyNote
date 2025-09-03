---
title: "05 — Linux内核数据结构 — 红黑树 — 插入节点（代码）"
description: 
date: 2025-02-17
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



# 1. **插入节点的用户接口**   
红黑树第一篇讲到调用插入函数`rb_add`插入节点时会调用两个重要函数，`rb_link_node`，`rb_insert_color`。其中rb_link_node用来将新节点链接到树中，而rb_insert_color用来在插入后修复平衡。      
```c
// less: 用户节点的排序逻辑
static __always_inline void
rb_add(struct rb_node *node, struct rb_root *tree,
       bool (*less)(struct rb_node *, const struct rb_node *))
{
	struct rb_node **link = &tree->rb_node;
	struct rb_node *parent = NULL;

	// 如果 node 小于 parent，向左子树移动；否则向右子树移动，直到 *link 为 NULL，就是要插入的位置。
	while (*link) {
		parent = *link;
		if (less(node, parent))
			link = &parent->rb_left;
		else
			link = &parent->rb_right;
	}

	rb_link_node(node, parent, link);  	// 将节点连接到树中
	rb_insert_color(node, tree);	    // 修复平衡
}
```

---

```c
/**	
 *  将新节点链接到红黑树中，将新节点 node 链接到红黑树中，作为 parent 的子节点，并更新 rb_link 指向新节点。
 */
static inline void rb_link_node(struct rb_node *node, struct rb_node *parent, struct rb_node **rb_link)
{
	node->__rb_parent_color = (unsigned long)parent;   // 保存父节点信息
	node->rb_left = node->rb_right = NULL;             // 初始化新节点

	*rb_link = node;
}

// 在 RCU（Read-Copy-Update）同步模型下，将新节点 node 链接到红黑树中。
// 用于需要 RCU 保护的红黑树插入操作，如并发读写场景。
static inline void rb_link_node_rcu(struct rb_node *node, struct rb_node *parent, struct rb_node **rb_link)
{
	node->__rb_parent_color = (unsigned long)parent;
	node->rb_left = node->rb_right = NULL;

	rcu_assign_pointer(*rb_link, node);
}
```

---

**重点函数：**
```c
// tools/lib/rbtree.c  
// 新节点默认初始化为红色。
void rb_insert_color(struct rb_node *node, struct rb_root *root)
{ 
	__rb_insert(node, root, dummy_rotate);
}

/**	
 *  node，新插入的节点。
 *  augment_rotate，这是给增强型红黑树用的，在非增强型红黑树中只是传入一个占位符，无需理会。  
 */
static __always_inline void
__rb_insert(struct rb_node *node, struct rb_root *root,
	    void (*augment_rotate)(struct rb_node *old, struct rb_node *new))
{
	struct rb_node *parent = rb_red_parent(node);  // 获取当前节点的父节点指针。
	struct rb_node *gparent, *tmp；

	while (true) {
        /**
         *  Case1, 插入节点为根节点。 将其设为黑色，结束循环。
         *  unlikely使用了linux上的一种性能优化手段，我们暂时理解为将“!parent”转换为bool值。
         */
		if (unlikely(!parent)) {      
			rb_set_parent_color(node, NULL, RB_BLACK);
			break;
		}


		/**
		 * Case2, 父节点是黑色，无需修复（性质未被破坏），退出。
		 */
		if(rb_is_black(parent))
			break; 

		/**	
		 * Case3, 父节点是红色，代码能走到这里，说明父节点是红色（需修复）
		 * 这里有分为几种子情况讨论
		 */
		gparent = rb_red_parent(parent);
		tmp = gparent->rb_right; // 获取叔节点

		/* Case3-1： 父节点是祖父节点的左子节点 */ 
		if (parent != tmp) {  
			
			/* Case3-1-1： 叔叔是红色 */
			if (tmp && rb_is_red(tmp)) {   
				// 进行颜色平衡
				rb_set_parent_color(tmp, gparent, RB_BLACK);     // 将叔节点设置为黑色。  
				rb_set_parent_color(parent, gparent, RB_BLACK);  // 将父节点设置为黑色。  
				
				// 设置了叔节点和父节点的颜色，相应也要更新祖父节点的颜色
 
				node = gparent;                            // 当前节点更新到祖父节点
				parent = rb_parent(node); 
				rb_set_parent_color(node, parent, RB_RED); // 将祖父节点设置为红色。

				continue; // 上面是这种局部修复可能会导致整体的不平衡，所以还要继续向上递归检查。
			}

			/* Case3-1-2： 叔节点是黑色的 */
			tmp = parent->rb_right;
			/**	
			 * Case3-1-2-1：新插入的节点是父节点的右子节点；
			 * 注意了Case3-1-2-1和Case3-1-2-2的代码合起来才是完整的Case3-1-2-1逻辑；
			 * Case3-1-2-2包含了Case3-1-2-1需要的部分旋转以及节点提升的逻辑。
			 */ 
			if (node == tmp) {    
				tmp = node->rb_left;
				WRITE_ONCE(parent->rb_right, tmp);
				WRITE_ONCE(node->rb_left, parent);
				if (tmp) {
					rb_set_parent_color(tmp, parent, RB_BLACK); 
				}
				rb_set_parent_color(parent, node, RB_RED); 
				augment_rotate(parent, node);
				parent = node;
				tmp = node->rb_right;
			}

			/* Case3-1-2-2：新插入的节点是父节点的左子节点 */
			WRITE_ONCE(gparent->rb_left, tmp); 
			WRITE_ONCE(parent->rb_right, gparent);
			if (tmp) {
				rb_set_parent_color(tmp, gparent, RB_BLACK);
			}
			__rb_rotate_set_parents(gparent, parent, root, RB_RED); // 提升parent节点（parent和gparent交换了身份），以及对应的颜色变换
			augment_rotate(gparent, parent);
			break;
		} 
		else  /* Case3-2： 父节点是祖父节点的右子节点 */
		{
			tmp = gparent->rb_left;

			/* Case3-2-1:  叔叔节点是红色的, 只需要变色操作 */
			if (tmp && rb_is_red(tmp)) {
				rb_set_parent_color(tmp, gparent, RB_BLACK);
				rb_set_parent_color(parent, gparent, RB_BLACK);
				node = gparent;
				parent = rb_parent(node);
				rb_set_parent_color(node, parent, RB_RED);
				continue;
			}

			tmp = parent->rb_left;
			/*  Case3-2-2:  叔叔节点是黑色的, 需要“旋转 + 变色”操作 */
			if (node == tmp) {
				/* Case3-2-2-1:  新节点是父节点的左子节点, 需要“右旋 + 变色”操作。和Case3-1-2类似，这里的部分逻辑在Case3-2-2-2  */
				tmp = node->rb_right;
				WRITE_ONCE(parent->rb_left, tmp);
				WRITE_ONCE(node->rb_right, parent);
				if (tmp)
					rb_set_parent_color(tmp, parent, RB_BLACK);
				rb_set_parent_color(parent, node, RB_RED);
				augment_rotate(parent, node);
				parent = node;
				tmp = node->rb_left;
			}

			/* Case3-2-2-2:  新节点是父节点的右孩子, 需要“左旋 + 变色”操作  */
			WRITE_ONCE(gparent->rb_right, tmp);
			WRITE_ONCE(parent->rb_left, gparent);
			if (tmp)
				rb_set_parent_color(tmp, gparent, RB_BLACK);
			__rb_rotate_set_parents(gparent, parent, root, RB_RED);
			augment_rotate(gparent, parent);
			break;
		}
	}
}
```

