
## wbox tlb

TLB是Translation Lookaside Buffer的简称，可翻译为“地址转换后援缓冲器”，也可简称为“快表”。简单地说，TLB就是页表的Cache，属于MMU的一部分，其中存储了当前最可能被访问到的页表项，其内容是部分页表项的一个副本。处理器在取指或者执行访问memory指令的时候都需要进行地址翻译，即把虚拟地址翻译成物理地址。而地址翻译是一个漫长的过程，需要遍历几个level的Translation table，从而产生严重的开销。为了提高性能，我们会在MMU中增加一个TLB的单元，把地址翻译关系保存在这个高速缓存中，从而省略了对内存中页表的访问。

### tlb命令寄存器
TLB_COMMAND_ADDR        0xF0110000
![[tlb_command.png]]
支持3种命令操作
- add entry ![[add_entry.png]]
- delete entry![[delete_entry.png]]
- share entry![[share_entry.png]]
### 输出状态寄存器
输出状态寄存器
![[out_status.png]]