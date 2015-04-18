##练习一：

###实现思路：

  > >完成对proc_struct的初始化
 > >
 > >Process state 为 未初始化
 > >
 > >Process ID 未分配，置为1
 > >
 > >running times of Proces 置为0
 > >
 > >Process kernel stack 置为0
 > >
 > >bool value: need to be rescheduled to release CPU： 不需要
 > > 
> >struct proc_struct *parent;  struct mm_struct *mm;      均未分配，置为空
 > >
 > >struct context context;              将context区域清空
 > >
 > >struct trapframe *tf;                   还未产生中断
 > >
 > >CR3 register:   设为全局的基地址 boot_cr3
 > >
 > >uint32_t flags;                         清空
 > >
 > >char name[PROC_NAME_LEN + 1];          清空

###请说明 context 和 trapframe 成员变量的作用是什么
 > >
 > >context： 切换的上下文
 > >
 > >trapframe： 当前中断保存恢复现场用
 > >
 > >context： 完成内核线程的上下文切换
 > >
 > >trapframe： 在本实验中似乎没用到


##练习2: 为新建的线程分配资源
 
###实现思路：
 > >
 > > 1、分配一个进程控制块
 > >
 > > 2、分配内核堆栈
 > >
 > > 3、分配或共享内存空间
 > >
 > > 4、设置tf 和 context
 > >
 > > 5、向进程链表中添加当前线程
 > >
 > > 6、将状态切换成runnable
 > >

 > > 7、返回孩子的进程编号



###请说明分配给每个fork线程的id是否唯一?

 > >
 > > 唯一。分析get_pid函数的实现：
 > >
 > > last_pid, next_safe 两个变量
 > >
 > > last_pid 为最小没出现的pid
 > >
 > > next_safe 为访问过的proc中大于last_pid的最小的pid
 > >
  > > 遍历proc_list ：
 > >
  > > 当 last_pid 等于当前pid时，last_pid + 1
 > >
 > > 当 last_pid ＋ 1 后大于等于保存的next_safe时，需要重新遍历proc_list, 以保证last_pid和之前访问过的proc_id 不重复


##练习三：理解proc_run

### 切换过程
 > >
 > > 将全局变量current 设为 要切换的线程 proc
 > >
 > > current = proc;
 > >
 > >加载proc的stack
 > >
 > >load_esp0(next->kstack + KSTACKSIZE);
 > >
 > >加载proc的页目录表首地址
 > >
 > >lcr3(next->cr3);
 > >
 > >切换上下文：
 > >
 > >switch_to(&(prev->context), &(next->context));


###在本实验中创建了几个内核线程？
 > >
 > > 创建了2个内核线程，idle 和 init_main， idle 为第一个线程， init_main 通过函数kernel_thread创建
 
###local_intr_save(intr_flag)在这里有什么作用？
 > >
 > > 代码中
 > >
 > > \#define spin_lock_irqsave(l, f) local_intr_save(f)

 > > \#define spin_unlock_irqrestore(l, f) local_intr_restore(f)
 > > 

 > > 在单cpu的情况下，通过将中断关闭，能够实现独占cpu处理。在该实验中，scheduled，switch 
context， proc_run 等函数使用了该方法。防止切换过程被打断，数据被修改。





