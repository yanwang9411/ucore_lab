##练习1:

###实现：
> >
> > 根据注释：
> >

> > 设置trapframe,使得进程从kernel返回user模式
> >
> >
cs  ＝ user_cs
> >
> >
ds es ss = user_ds
> >
> >
esp指向用户栈(堆顶)
> >
> >
eip指向elf文件定义的程序入口地址
> >
> >
eflags设置为允许中断
> >

###加载运行的过程，
> >
> >
执行kernel_execve，产生系统调用
> >
> >
sys_exec 调用 do_execuv()
> >
> >
do_exe 调用 load_icode 进行用户程序代码加载
> >
> >
在load_icode中实现了 分配内存，初始化堆栈
> >
> >
设置trapframe
> >
> >
trap返回后执行用户程序


##练习2:

###copy_range 的实现：

> >
> >
实现思路：
> >
> >

将page内容复制到npage
> >
> >
memcpy( addr1, addr2, page_size);
> >
> >
并建好映射
> >
> >
insert (npage);

###cow的实现：
> >
> >
在proc_struct 的mmstruct中保存所有父进程的page指针，在子进程进行写操作时，alloc_page  分配一个新的页，替换mm中的该页。


##练习3:

###说明fork/wait/exit是如何运行的？

> >
> >
####do_fork:
> >
> >
调用点：
> >
> >
创建新的线程时调用 kernel_thread ，kernel_thread 设置child process的trapframe，并调用do_fork
> >
> >
实现：
> >
> >
do_fork: 分配内存等资源，并将子线程加入到就绪队列（runnable）

####do_wait:
> >
> >
调用点：在init_main中，初始化用户线程之后，执行 do_wait, 直到所有用户线程都资源释放。
> >
> >
实现：
> >
> >
查找处在僵尸状态的子线程，返回0，直到没有子线程未释放返回-E_BAD_PROC

####do_exit:
> >
> >
调用点：
> >
> >
系统调用
> >
> >
实现：
> >
> >
释放资源，进入僵尸状态，唤醒父进程reclaim，放弃cpu

###请说明fork等在实现中是如何影响进程执行状态的？

> >
> >
fork：最后调用wakeup_proc，将程序放入就绪队列。
> >
> >
wait：当有子程序未返回，程序进入sleeping状态，等待子线程的返回。
> >
> >
exit： 设置当前进程状态未zombie


###调用关系图如下：
> >
> > 状态： runnable running sleep zombie
> >
> > runnable -> running ： schedule（）
> >
> > 进入runnable  : do_fork, do_execv
> >
> > running -> zombie : do_exit()
> >
> > running -> sleep : do_wait()


