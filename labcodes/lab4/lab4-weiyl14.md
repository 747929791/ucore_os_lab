# lab4 实验报告

## 练习0：填写已有实验

继续沿用之前的做法，用git diff结合Linux自带的patch命令来完成。 

	git diff <...> --relative=labcodes/lab3 | patch -d labcodes/lab4 -p1

注意到这里我们看上去仅重复了lab3当中的更改，但实际上因为lab3已经包含了更早的lab当中引入的修改，因此只需要这样做其实就已经把所有修改一起应用进来了。

## 练习1：分配并初始化一个进程控制块（需要编码）

**请在实验报告中简要说明你的设计实现过程。**

根据定义对各个成员变量进行初始化即可。参照实验指导书的说明，尚未初始化的进程，其状态是UNINIT；尚未分配pid，所以pid为-1。其余的量没有明确的默认值，因此都先置为0。

**请说明proc_struct中struct context context和struct trapframe *tf成员变量含义和在本实验中的作用是啥？（提示通过看代码和编程调试可以判断出来）**

context是代码运行的上下文，即各个寄存器的值。trapframe是中断栈帧。这两个变量在本实验当中的线程切换里会用到。

## 练习2：为新创建的内核线程分配资源（需要编码）

**请在实验报告中简要说明你的设计实现过程。**

遵照所给的步骤来完成即可。这里没有明说的一点是需要调用`get_pid()`来分配一个pid，并且`hash_proc()`的正确执行依赖已经正确赋值的`proc->pid`，因此这两个操作执行的顺序不能颠倒。

**请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。**

是。每次fork都会调用`get_pid()`分配pid，而该函数的实现确保了分配出来的pid是唯一的。

## 练习3：阅读代码，理解 proc_run 函数和它调用的函数如何完成进程切换的。（无编码工作）

**请在实验报告中简要说明你对proc_run函数的分析。**

`proc_run()`函数将当前运行的进程从全局变量`current`变为传入的参数`proc`。函数先后完成了这些事情：将`current`置为`proc`，切换内核堆栈，载入新进程的页表，最后调用`switch_to()`完成进程的切换。`switch_to()`在MOOC当中已经有分析，这里不再赘述。

**在本实验的执行过程中，创建且运行了几个内核线程？**

两个。分别是0号线程idleproc和1号线程initproc。

**语句local_intr_save(intr_flag);....local_intr_restore(intr_flag);在这里有何作用?请说明理由**

`local_intr_save(flag)`暂时屏蔽中断，并将此前是否允许中断的信息存入`flag`；`local_intr_restore(flag)`在传入参数`flag`为真时设置中断使能。因此用这两个操作包起一段代码，就是暂时屏蔽中断，然后在这段代码结束时恢复回原来的中断启用状态。这是为了确保关键代码区域（在这里是进程切换）操作的原子性。