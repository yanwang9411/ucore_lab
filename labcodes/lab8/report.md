
## 练习 1: 完成读文件操作的实现（需要编码）


### 首先了解打开文件的处理流程，然后参考本实验后续的文件读写操作的过程分析，编写在sfs_inode.c中sfs_io_nolock读文件中数据的实现代码

> > 
> > 实现：   

> > 
> > 在sfs_io_nolock函数中， 将读取的函数操作和sfs绑定，令sfs_buf_op = sfs_rbuf, sfs_block_op = sfs_rblock。
> > 
> > 先读写完整的数据块，通过sfs_rblock , 然后读写剩余的部分，通过sfs_rbuf。每部分中都调用sfs_bmap_load_nolock函数得到blkno对应的inode编号。完成后如果offset + alen > din->fileinfo.size（写文件时会出现这种情况，增加文件内容时） ， 则调整文件大小为offset + alen并设置dirty变量


###请在实验报告中给出设计实现”UNIX的PIPE机制“的概要设方案， 鼓励给出详细设计方案

> >
> > 不使用文件系统，可以通过分配页，并保存索引，同步互斥访问等方式来完成进程间的pipe通信。
> > 而使用文件系统，可以实现完整的pipe_fs文件系统，并将其add到vfs即可，首先类似sfs需要实现索引和目录，目录为进程pair和通信文件inode的对应，索引即inode。其次，需要实现读写方式，此处的读写频繁，且只需要保存未被目标进程读取的内容，即只需要存储buffer。类似生产消费者问题，可以实现读写互斥同步。


## 练习二：完成基于文件系统的执行程序机制的实现（需要编码）


### 请在实验报告中给出设计实现基于”UNIX的硬链接和软链接机制“的概要设方案， 鼓励给出详细设计方案

#### 硬链接

在lab8中，打开一个文件调用的函数如下：
```
  sysfile_open->file_open->vfs_open->vfs_lookup->vop_lookup(sfs_lookup)->sfs_lookup_once->sfs_load_inode->sfs_create_inode->vop_init(inode_init)
```    
> > 
> > 
在Linux中，将f2硬链接到f1后，f2和f1链接的inode节点相同。
lab8中打开文件时，在SFS层进行具体的文件inode创建。
> > 
> > 实现：
> > 
> > 
增加一个创建硬链接的函数，对某个path创建一个inode指针，指向要链接到的文件的inode，即两个文件链接到同一个inode。另外将对应的sfs_disk_inode的nlinks加一。如果要增加文件的删除功能，则需要对某个文件inode的硬链接数进行判断，如果为0，则最后删除该inode节点。

#### 软链接

> > 
> > 
根据path创建一个文件，然后新建inode和进行相关的初始化。openfile时，先检查文件是否存在，未存在则新创建。
> > 
> > 
创建软链接的函数：参数是被链接的文件和作为软连接的文件，获取这两个文件的inode，如果文件不存在则创建。然后将被链接文件inode中存储的相关信息，尤其是实际文件的位置信息写入软链接文件。
> > 
>  >
打开软链接文件：即打开被链接的文件,可以通过修改sfs inode，在其中增加成员变量表示是否是软链接，直接根据inode里的标识来进行操作。
