kmalloc()、kzalloc()、kcalloc() 和 vmalloc() 都是 Linux 内核中用于动态分配内存的函数。其中kmalloc()、kzalloc() 和 kcalloc() 是 slab 分配器的接口函数，主要用于分配内核对象的内存。他们的区别如下：
- kmalloc()：分配指定大小的内存，返回指向内存块的指针，该内存块的内容是未初始化的。
- kzalloc()：分配指定大小的内存，并将其内容全部设置为 0，返回指向内存块的指针。
- kcalloc()：分配指定大小的内存，并将其内容全部设置为 0，返回指向内存块的指针。

 相比之下，vmalloc() 是用于分配虚拟内存的函数，其分配的内存空间在物理内存中是不连续的，并且不同于 slab 分配器，它不会有内存碎片的问题。
 总结：
 - kmalloc()、kzalloc() 和 kcalloc() 适用于动态分配内核对象的内存，可以保证分配的内存块是连续的；
- vmalloc() 主要用于分配虚拟内存，适用于需要大块连续虚拟内存的场景，但在一些情况下可能会导致性能问题。


1. kmalloc 分配的内存大小有限制（128KB），而 vmalloc 没有限制；
2. kmalloc 可以保证分配的内存物理地址是连续的，但是 vmalloc 不能保证
3. kmalloc 分配内存的过程可以是原子过程（使用 GFP_ATOMIC），而 vmalloc 分配内存时则可能产生阻塞；
4.  kmalloc 分配内存的开销小，因此 kmalloc 比 vmalloc 要快；
5. kzalloc是清零的kmalloc
6. kcalloc是 kzalloc 的array

flags 的参考用法：
　|– 进程上下文，可以睡眠　　　　　GFP_KERNEL
　|– 进程上下文，不可以睡眠　　　　GFP_ATOMIC
　|　　|– 中断处理程序　　　　　　　GFP_ATOMIC
　|　　|– 软中断　　　　　　　　　　GFP_ATOMIC
　|　　|– Tasklet　　　　　　　　　GFP_ATOMIC
　|– 用于DMA的内存，可以睡眠　　　GFP_DMA | GFP_KERNEL
　|– 用于DMA的内存，不可以睡眠　　GFP_DMA |GFP_ATOMIC

```C
void *kmalloc(size_t size, gfp_t flags);
void *kzalloc(size_t size, gfp_t flags)
void *kcalloc(size_t n, size_t size, gfp_t flags);
void kfree(const void *objp);
#define __get_free_page(gfp_mask) __get_free_pages((gfp_mask), 0) // 128k

void *vmalloc(unsigned long size);
void vfree(const void *addr);

```




