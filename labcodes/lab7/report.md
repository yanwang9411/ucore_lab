

##练习1:

### 理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题,分析了解lab7
采用信号量的执行过程

> >
> >  init_main，调用了check_sync函数，check—sync函数实现基于信号量的哲学家问
题,和基于管程的哲学家就餐问题
> > 
> > 首先初始化一个互斥信号量,然后创建5个哲学家内核线程,每个内核线程完成了基于信号量的哲学家就餐.
###基于信号量的哲学家问题
> > 
> > 先创建多个哲学家线程，哲学家行为如下：
        do_sleep(SLEEP_TIME);
        phi_take_forks_sema(i); 
        /* 需要两只叉子，或者阻塞 */
        do_sleep(SLEEP_TIME);
        phi_put_forks_sema(i);
        /* 把两把叉子同时放回桌子 */

> > 
> > phi_take_forks_sema和phi_put_forks_sema的实现：

```c
        down(&mutex);         /* 进入临界区 */
        state_sema[i]=HUNGRY; /* 哲学家i饥饿 */
        phi_test_sema(i);     /* 请求得到两只叉子 */
        up(&mutex);           /* 离开临界区 */
        down(&s[i]);          /* 请求失败就阻塞 */
```
```c
        down(&mutex);           /* 进入临界区 */
        state_sema[i]=THINKING; /* 哲学家进餐结束 */
        phi_test_sema(LEFT);    /* 看一下左邻居现在是否能进餐 */
        phi_test_sema(RIGHT);   /* 看一下右邻居现在是否能进餐 */
        up(&mutex);             /* 离开临界区 */
        
```
##练习2:    
###设计实现
> > 
> >  管程机制,然后基于信号量实现完成条件变量实现,然后用管程机制实现哲学家就餐问题的解决方案

> > 
> > 运行时，先将状态设置为hungry，然后进行test，如果满足条件变量，并调用cond_signal,从在等待的队列中选取调度执行，
。条件不满足，那么会阻塞并且等待，调用函数cond_wait实现。运行结束后，如果有等待这个条件变量的线程，则唤醒，退出临界区。在放下餐具的时候
，测试左右两边的哲学家是否满足条件变量，如果满足条件变量，就进入调度运行。

```c
void phi_take_forks_condvar(int i) {
     down(&(mtp->mutex));
      state_condvar[i]=HUNGRY; 
      phi_test_condvar(i); 
      while (state_condvar[i] != EATING) {
          cprintf("phi_take_forks_condvar: %d didn't get fork and will wait\n",i);
          cond_wait(&mtp->cv[i]);
      }
//--------leave routine in monitor--------------
      if(mtp->next_count>0)
         up(&(mtp->next));
      else
         up(&(mtp->mutex));
}

void phi_put_forks_condvar(int i) {
     down(&(mtp->mutex));
      state_condvar[i]=THINKING;
      // test left and right neighbors
      phi_test_condvar(LEFT);
      phi_test_condvar(RIGHT);
//--------leave routine in monitor--------------
     if(mtp->next_count>0)
        up(&(mtp->next));
     else
        up(&(mtp->mutex));
}
```
###用管程实现信号量时，有两个函数的实现：
####cond_wait
> >
> > 1、如果进程A调用cond_wait函数,表示此进程等待某个条件C不为真,需要sleep，等待进程个数cv.count加一。
> > 
> > 2、monitor.next_count如果大于0,表示有进程执行cond_wait函数等待,存储在monitor.next信号量上。唤醒进程链表中的一个进程B。
> > 
> > monitor.next_count如果小于等于0,表示目前没有进程执行cond_signal函数且在等待,则唤醒由于互斥条件限制而无法进入管程的进程,即在monitor.mutex上的进程
> > 
> > 3、然后cv.sem等待,运行结束,则让cv.count减一,表示等待此条件的睡眠进程个数减少一个,可继续执行其他线程.

```c
    cvp->count++;
    if(cvp->owner->next_count>0){
        up(&(cvp->owner->next));
    }else up(&cvp->owner->mutex);
    down(&cvp->sem);
    cvp->count--;
```
####cond_signal
> >
> > 判断cv.count,如果不大于0,则表示当前没有执行cond_wait的进程，返回。
> > 
> > 如果大于0,这表示当前有执行cond_wait而睡眠的进程,因此需要唤醒等待在cv.sem上睡眠的进程A。monitor.next_count加一，当前进程(进程B)在信号量monitor.next上等待。运行结束后,monitor.next_count
减一。

```c
   if(cvp->count>0){
       cvp->owner->next_count++;
	   up(&(cvp->sem));
	   down(&(cvp->owner->next));
	   cvp->owner->next_count--;
   }
```


