##练习1:
###分析sched_class函数指针的用法，并结合RR调度算法描述ucore的执行过程

####sched_class :


struct sched_class {
   const char *name;
   void (*init)(struct run_queue *rq);
   void (*enqueue)(struct run_queue *rq, struct proc_struct *proc);
   void (*dequeue)(struct run_queue *rq, struct proc_struct *proc);
   struct proc_struct *(*pick_next)(struct run_queue *rq);
   void (*proc_tick)(struct run_queue *rq, struct proc_struct *proc);
 };

> > 
> > init : 初始化 进程队列
> > 
> > enqueue：入队
 > > 
> > dequeue: 出队
> > 
> > pick_next: 调度时选择下一个就绪的进程
> > 
> > proc_tick : 处理时间中断


###ucore的执行过程：

> > 
> > 将static variable sched_class 设置为 default 即 RR scheduler
> > 
> > 在调度点调用schedule(), 
> > 
> > 进一步调用 sched_class_enqueue 等函数

###RR 算法的实现：
> > 
> > enqueue：将当前运行的进程加到队尾
> > 
> > dequeue：将队首的进程移出
 > >
> > pick：选择队首的进程
> > 
> > proc_tick ： 将当前运行进程的timeslice －1 ， 若时间片用光，则进入调度

###多级反馈队列调度算法：

#### 数据结构： 将run_queue *rq
> >
> > 改为rq数组，大小等于优先级数，用于存储不同优先级的进程队列，其中优先级越高的rq，timeslice越短。

> > 

#### sched_class的函数实现：

> > 
> > enqueue：
将当前进程加到proc->priority相应的优先级队列
> > 
> > pick_next:
> > 从优先级最高的队列中顺序选取（即队首）；若队列为空，搜索更低优先级的队列

 > >
> > dequeue：
> > 
> > 从优先级最高的队列中顺序移除；若队列为空，搜索更低优先级的队列

> > 
> > proc_tick:
> > 
> > timeslice 减1； 若为0，则设置need_sched为1， 并将该进程的优先级＋1，即降低优先级。




## 练习2：

### stride_schedule的设计实现：

#### enqueue：
>  > 
>  > 使用 skew heap， 定义比较函数后，可直接插入并排序
>  > 
>  > 设proc->rq = rq; number + 1;
>  >

#### dequeue：
> > 
>  > 移出堆顶; number -1;

####pick_next:
> > 
> > 选取 lab6_run_pool 的堆顶

####proc_tick:

> > 将timeslice－1 若为0，则将当前进程设置为需要调度
