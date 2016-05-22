﻿# lab7 实验报告

## 练习0：填写已有实验

继续沿用之前的做法，用git diff结合Linux自带的patch命令来完成。 

	git diff <...> --relative=labcodes/lab6 | patch -d labcodes/lab7 -p1

## 练习1: 理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题（不需要编码）

**请在实验报告中给出内核级信号量的设计描述，并说其大致执行流流程。**

内核级信号量在实现上和原理课上所讲的区别不大，其结构包含一个整数和一个等待队列。

P操作：首先屏蔽中断，然后判断value是否为正。如果为正，说明不需要等待，将value减一，恢复中断，返回；否则，调用`wait_current_set()`把当前线程加入到等待队列中，恢复中断，调用`schedule()`主动放弃CPU使用权。对于后一种情况，当线程再有机会执行，是从调用`schedule()`的下一条语句开始执行，这时应当在屏蔽中断的条件下把当前线程从等待队列当中移除，然后退出函数。

V操作：整个函数都在屏蔽中断的环境下完成。判断是否有进程在该信号量上等待，如果有则将其唤醒，否则将value加一即可。

**请在实验报告中给出给用户态进程/线程提供信号量机制的设计方案，并比较说明给内核级提供信号量机制的异同。**

将上述实现用系统调用进行包装应当是可行的。

## 练习2: 完成内核级条件变量和基于内核级条件变量的哲学家就餐问题（需要编码）

**请在实验报告中给出内核级条件变量的设计描述，并说其大致执行流流程。**

ucore当中的条件变量基于前述内核级信号量来实现。其结构包含一个信号量，一个整数count表示在其上等待的线程个数，和一个指向其所属的管程的指针。

wait操作：将count加一。如果管程当中有正在等待的线程（即，有进程在信号量`monitor->next`上等待），就将其唤醒，否则释放管程互斥锁mutex。最后对条件变量的信号量执行`down()`，将自己加入等待队列。再次运行时，将count减一，退出。

signal操作：如果count不大于0，什么也不做。否则，count大于0，说明有线程在这个条件变量上等待，命当前线程在信号量`monitor->next`上等待，然后对条件变量的信号量进行`up()`操作唤醒在其上等待的线程。再次运行时，释放信号量`monitor->next`，退出。