---
title: "内核OOM机制"
description: "Out of Memory"
date: 2025-05-21T20:53:24+08:00
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - Linux内核
---

# 1. 什么是OOM
在内存不足时，内存管理系统会回收内核中可以释放的内存，当实在没有内存可用的时候，就会进入OOM（Out of Memory）状态，内存管理系统会执行OOM Killer，依据一定的规则杀死一些进程来释放内存，对于个人PC来说，这都不是事儿，但对于服务器，有可能就会将重要的业务进程给干死了，所以有的服务器会将sysctl的vm.panic_on_omc参数设为1，当发生OOM时强制关闭系统，如果设置为0(默认)，在OOM时就会运行OOM Killer。   
```bash
dmesg | grep -i "out of memory"  # 查看OOM事件
```

# 2. OOM Killer机制
OOM Killer机制依靠两个因素选择要杀的进程，`oom_score`和`oom_score_adj`，其中oom_score是内核通过进程的内存消耗计算出来的，oom_score_adj（取值-1000 ~ 1000）是用户用来干预oom的（用户权重），内核会向`oom_score + oom_score_adj`值最高的进程发送关闭信号。   

## 2.1. oom Killer代码
内核代码：linux-6.14.6/mm/oom_kill.c  
1. OOM杀手算法的评分机制:   
```c
long oom_badness(struct task_struct *p, unsigned long totalpages)
{
	long points;
	long adj;

  // 1. 排除不可杀进程
	if (oom_unkillable_task(p))
		return LONG_MIN;

	p = find_lock_task_mm(p);
	if (!p)
		return LONG_MIN;

  // 2. 获取用户权重，并判断该进程是否可杀
	adj = (long)p->signal->oom_score_adj;
	if (adj == OOM_SCORE_ADJ_MIN ||
			test_bit(MMF_OOM_SKIP, &p->mm->flags) ||
			in_vfork(p)) {
		task_unlock(p);
		return LONG_MIN;
	}

  // 3. 基础评分 = 物理内存 + 交换分区 + 页表内存
	points = get_mm_rss(p->mm) + get_mm_counter(p->mm, MM_SWAPENTS) +
		mm_pgtables_bytes(p->mm) / PAGE_SIZE;
	task_unlock(p);

  // 4. 归一化调整值 = OOM评分调整值 * 总内存/1000
	adj *= totalpages / 1000;
	// 5. 最终评分
  points += adj;

	return points;
}
```
2. 查找oom最大的进程
```c
static void select_bad_process(struct oom_control *oc)
{
	oc->chosen_points = LONG_MIN;

	if (is_memcg_oom(oc))
		mem_cgroup_scan_tasks(oc->memcg, oom_evaluate_task, oc);
	else {
		struct task_struct *p;

		rcu_read_lock();
    // 否则遍历系统所有进程，通过oom_evaluate_task评估每个进程，若返回true则立即停止遍历（找到候选进程）
		for_each_process(p)
			if (oom_evaluate_task(p, oc))
				break;
		rcu_read_unlock();
	}
}
```

3. 杀死进程
```c
static void oom_kill_process(struct task_struct *victim, const char *message)
{
  // 杀死进程，太多了，就不放了
}
```

# 3. /proc/sys/vm/oom_kill_allocating_task
写入1表示优先杀死导致内存不足的任务，而不是选择评分最高的任务。
