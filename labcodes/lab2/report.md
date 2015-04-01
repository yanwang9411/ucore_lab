
#lab2 实验报告  
计24 2012011352  王妍

##[练习一]  
###实现 first-fit 连续物理内存分配算法  

>>
>>设计和实现:
>>
> >page中保存的值：
>>property：保存空闲的块
>>ref ： 保存是否被使用

>>alloc 函数中：

>>通过遍历freelist列表：找到第一个大小大于要求size的空闲的块：
>>需要进行的操作：
>>首先从freelist中删去该n个page，
>>将剪裁后的空闲块加入到freelist中
>>
>>在free 函数中：

>>需要完成的步骤：

>>根据首地址找到合适的插入位置；
>>设置 ref 为 0, 设置 property 为n
>>向前向后合并，如果可合并，则通过改变property进行实现

>>
###可改进的地方：

>>开始的实现方法：每个page保存其后最大的空闲空间, 即设property为其后最大的block数；
后来的改进：如参考答案所写，仅在空闲块起始page保存最大空闲空间； 可以将修改复杂度由 o(n) 降到 o(1)
>>

>>释放空间插入时的查找可使用二分查找或者 skip list 进行缩减时间

###与参考答案比较：  
思路基本相同。

##[练习2]  

###实验内容：通过设置页表和对应的页表项， 可建立虚拟内存地址和物理内存地址的对应关系。 其中的get_pte函数是设置页表项环节 中的一个重要步骤。 此函数找到一个虚地址对应的二级页表项的内核虚地址， 如果此二级页表项不存在， 则分配一个包含此项的二级页表。 需要补全get_pte函数 in kern/mm/pmm.c， 实现其功能。请在实验报告中简 要说明你的设计实现过程。 

>>设计实现：
> >getpte：给定虚拟地址，返回在二级页表中对应的项
>>查找页目录项
>>若需要创建，则分配一个页创建页表
>>设置页目录项的权限
>>返回该页表的线性地址


### 请描述页目 录项（Pag Director Entry） 和页表（Page Table Entry） 中每个组成部分的含义和以及对ucore而言的潜在用处。

>> 主要存储：页目录项保存对应的页表地址，页表项保存对应的页的地址

>>标志位：

‘’’
/* page table/directory entry flags */
#define PTE_P           0x001                   // 是否存在
#define PTE_W           0x002                  // 是否可写：只有一级二级页表的项都设置了用户写权限后，用户才能读写
#define PTE_U           0x004                   // 用户是否可访问
#define PTE_PWT         0x008                // Write-Through
#define PTE_PCD         0x010                // 是否可以缓存，（如每次IO得到不同的结果，此时就不可以缓存
#define PTE_A           0x020                   // 是否可获取
#define PTE_D           0x040                   // 是否是脏页： 进行页替换时，是否要写回
#define PTE_PS          0x080                   // 页大小
#define PTE_MBZ         0x180                   // 需要置为0的位：预留空闲的位，以兼容其他格式的页表项或页目录项
#define PTE_AVAIL       0xE00                   // 软件可用，可被用户程序设置
  ‘’’                                            

###如果ucore执行过程中访问 内存， 出现了页访问 异常， 请问 硬件要做哪些事情？
对外存进行IO操作，将修改过的内存内容写回等一系列io操作


##练习三：释放某虚地址所在的页并取消对应二级页表项的映射
###设计实现


>>查询页目录是否存在
>>找到对应的页，将引用次数减1，如果引用次数变为0了，则释放这个页，删除二级页表项
>>更新tlb：将tlb中对应的entry 设置为失效，仅当修改的页表在当前的进程中


###和标准答案的差别

 > >似乎没有

###数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？
‘’’
// page number field of address
#define PPN(la) (((uintptr_t)(la)) >> PTXSHIFT)
‘’’
>> la 是address，PPN(la) 是page数组的index，所以是地址右移PTXSHIFT位


###如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？

由以下代码可知：
/* *
 * PADDR - takes a kernel virtual address (an address that points above KERNBASE),
 * where the machine's maximum 256MB of physical memory is mapped and returns the
 * corresponding physical address.  It panics if you pass it a non-kernel virtual address.
 * */
#define PADDR(kva) ({                                                   \
            uintptr_t __m_kva = (uintptr_t)(kva);                       \
            if (__m_kva < KERNBASE) {                                   \
                panic("PADDR called with invalid kva %08lx", __m_kva);  \
            }                                                           \
            __m_kva - KERNBASE;                                         \
        })

/* *
 * KADDR - takes a physical address and returns the corresponding kernel virtual
 * address. It panics if you pass an invalid physical address.
 * */
#define KADDR(pa) ({                                                    \
            uintptr_t __m_pa = (pa);                                    \
            size_t __m_ppn = PPN(__m_pa);                               \
            if (__m_ppn >= npage) {                                     \
                panic("KADDR called with invalid pa %08lx", __m_pa);    \
            }                                                           \
            (void *) (__m_pa + KERNBASE);                               \
        })


physical address 和 virtual address的转换:
virt addr = linear addr  = phy addr + 0xc0000000
将kernbase定义为0 即可
