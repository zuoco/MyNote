---
title: "03 — Linux内核数据结构 — 红黑树 — 移除节点（代码）"
description: 
date: 2025-02-09
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


- [1. 移除节点的用户接口](#1-移除节点的用户接口)
	- [2.1. **\_\_rb\_erase\_color()**](#21-__rb_erase_color)




# 1. 移除节点的用户接口
**rb_erase()**： include/linux/rbtree.h   
```c
extern void rb_erase(struct rb_node *, struct rb_root *);

/** 
 *  tools/lib/rbtree.c
 *  移除黑色节点会导致经过该节点的路径所包含的黑节点比其他路径少一个，破坏了平衡。 
 *  当被删除节点（node）有左右孩子时间，需要使用后继节点（successor）来填补node的位置，此时问题就变成了“删除后successor”。
 */
void rb_erase(struct rb_node *node, struct rb_root *root)
{
	struct rb_node *rebalance;
    /**
	 * rb_erase_augmented 完成两件事：  
     * 1. 执行节点删除操作，node；
     * 2. 该函数会完成一部分简单的平衡任务，如果没有完全平衡就返回一个节点地址，返回的节点地址有两种情况：   
	 *    a. node没有孩子，且是黑色，返回node的父节点。    
     *    b. node有左右孩子，并且后继节点是黑色，返回后继节点的父节点。     
	 */
	rebalance = __rb_erase_augmented(node, root, &dummy_callbacks);  // 完成部分平衡修复工作
	if (rebalance) {
		// 进一步完成平衡
		____rb_erase_color(rebalance, root, dummy_rotate); 
    }
}
```

---

**__rb_erase_augmented()**： 删除节点，并完成初步的平衡工作，尽可能在该函数内完成平衡，如果情况简单，该函数内部就完成了，就返回NULL，如果情况比较复杂，该函数内部只能完成部分，就返回待平衡节点地址。        
```c
/* tools/lib/rbtree.c */
static __always_inline struct rb_node *
__rb_erase_augmented(struct rb_node *node, struct rb_root *root,
		     const struct rb_augment_callbacks *augment)
{
	struct rb_node *child = node->rb_right;
	struct rb_node *tmp = node->rb_left;
	struct rb_node *parent, *rebalance;
	unsigned long pc;  

    /* if {没有孩子/只有右孩子} else if {只有左孩子} else {两个孩子都在} */
	if (!tmp) {
        /** 
         * Case 0: 被删除节点有右孩子或者没有孩子， 如果有孩子只能是红色。
		 * !tmp == true也就是tmp == NULL，没有左孩子，那就剩下两种情况了：“只有右孩子”、“没有孩子”。
         */

		pc = node->__rb_parent_color;                   // 获取被删除节点的父节点指针
		parent = __rb_parent(pc);                       // 获取被删除节点的父节点指针
		__rb_change_child(node, child, parent, root);   // parent的子节点从node改为child: (A -> B -> C) 变为 （A -> C）
		
        /* 根据有没有孩子决定要不要在本函数内部修复平衡 */
        if (child) {  
            // Case1: 只有红色右孩子，此时孩子变色即可，本函数内部修复。   
			child->__rb_parent_color = pc;              // child的父节点指针指向自己的祖父节点， 同时颜色被置为与父节点同色（黑色）。  
			rebalance = NULL;                           // 不再需要额外修复。     
		} else {
            // Case2: 没有孩子，且为节点为黑色（删除红色节点不会破环平衡），
			// 此时的平衡修复工作量大，所以返回rebalance给__rb_erase_color()，由__rb_erase_color()进行修复。
			rebalance = __rb_is_black(pc) ? parent : NULL;
        }   
        tmp = parent;      
	} else if (!child) {
        /** 
         *  Case 3: 被删除节点只有左孩子，此时孩子也只能是红色。
         */
		
		// 经过下面上行函数： parent <--> node <--> tmp 变为 parent <--> tmp
		tmp->__rb_parent_color = pc = node->__rb_parent_color; // 原本 node <-- tmp，现在 parent <-- tmp，并且tmp颜色设置为node同色(本函数内部恢复平衡)。
		parent = __rb_parent(pc);
		__rb_change_child(node, tmp, parent, root);            // 原本 parent --> node，现在 parent --> tmp
		
		rebalance = NULL; 
		tmp = parent;
	} else {
		/**
		 * Case 4:  被删除节点有左右孩子，此时需要使用“后继节点”替换将要被删除的节点。
		 * if-else 语句块处理被删除节点的右子树。
		 */
		struct rb_node *successor = child, *child2;   

		/* 选择后继节点，也就是右子树中的最大节点（最左节点）。 */
		tmp = child->rb_left;
		if (!tmp) {
			/*
			 * Case 4-1: 如下图，被删除节点（n），因为s没有左子树，所以n的右子树中最大就是s。
			 *
			 *    (n)          (s)
			 *    / \          / \
			 *  (x) (s)  ->  (x) (c)
			 *        \
			 *        (c)
			 */
			parent = successor;             // 上图中的 s
			child2 = successor->rb_right;   // 上图中的 c

			augment->copy(node, successor); 
		} else {
			/*
			 * Case 4-2: 如下图，被删除节点（n），因为s有左子树，所以n的右子树中最大就是s。
			 *
			 *    (n)          (s)
			 *    / \          / \
			 *  (x) (y)  ->  (x) (y)
			 *      /            /
			 *    (p)          (p)
			 *    /            /
			 *  (s)          (c)
			 *    \
			 *    (c)
			 */

			do {  // 在被删除节点的右子树中，沿着最左边沿一直找，找到最左边的节点，就是被删除节点的右子树中的最大节点。
				parent = successor;
				successor = tmp;
				tmp = tmp->rb_left;
			} while (tmp);  

			/** 
			 *  找到了后继节点，由于后继节点是最左节点，所以它没有左子树，只有右子树。  
			 *  parent    -->   p,
			 *  successor -->   s,
			 *  child2    -->   c,  
			 */
			child2 = successor->rb_right;   
			WRITE_ONCE(parent->rb_left, child2);     // p --> c 
			WRITE_ONCE(successor->rb_right, child);  // s --> y
			rb_set_parent(child, successor);         // child的父节点设置为successor，y --> s，同时保留y的颜色

			augment->copy(node, successor);
			augment->propagate(parent, successor);
		}

		/* 处理node的左子树 */
		tmp = node->rb_left;                        // 被删除节点的左子树的 
		WRITE_ONCE(successor->rb_left, tmp);        // 原本 node --> tmp，现在successor --> tmp
		rb_set_parent(tmp, successor);              // tmp --> successor

		pc = node->__rb_parent_color;
		tmp = __rb_parent(pc);                          // node的父节点
		__rb_change_child(node, successor, tmp, root);  // 原本 tmp(pc) --> node，现在 tmp(pc) --> successor。 
		
		/**	
		 *  因为 successor 是 node 的右子树中的最左节点，所以它必然没有左孩子，但是右孩子就不一定了。
		 *  如果有右孩子，那么这该节点一定是红色的，否则这棵树中经过node节点的路径说包含的黑色节点数量不同了。
		 *  如果没有右孩子，这个节点可能是红色，也可能是黑色：if {有孩子} else {没有孩子}。  
		 */  
		if (child2) {  
			// successor 有右孩子（child2是后继节点的右子树）
			successor->__rb_parent_color = pc;              // successor继承node的父节点和颜色： successor --> tmp， 颜色也设置为和node同色。 
			rb_set_parent_color(child2, parent, RB_BLACK);  // child2 --> parent，并将child2的颜色设置为黑色。
			rebalance = NULL;
		} else {
			// successor 没有右孩子
			unsigned long pc2 = successor->__rb_parent_color;
			successor->__rb_parent_color = pc;
			/**
			 *  如果successor是红色，删除它不影响平衡；
			 *  如果successor是黑色，删除它会破坏平衡，需要修复平衡。
			 */
			rebalance = __rb_is_black(pc2) ? parent : NULL;  
		}  
		tmp = successor;
	}

	augment->propagate(tmp, NULL);
	return rebalance;
}
```

---

**__rb_erase_color()**： 在__rb_erase_augmented()之后调用，进一步完成平衡工作。       
```c
static __always_inline void
____rb_erase_color(struct rb_node *parent, struct rb_root *root,
	void (*augment_rotate)(struct rb_node *old, struct rb_node *new))
{
    // 和 rb_insert_color() 的流程差不多
	// 根据parent的兄弟节点的颜色分布，分情况讨论。
}
```
