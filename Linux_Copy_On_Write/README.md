### Linux写时拷贝技术(copy-on-write)
源于网上资料

**COW技术初窥：**

**Copy-on-write** (sometimes referred to as "COW") is an optimization strategy used in computer programming. The fundamental idea is that if multiple callers ask for resources which are initially indistinguishable, they can all be given pointers to the same resource. This function can be maintained until a caller tries to modify its "copy" of the resource, at which point a true private copy is created to prevent the changes becoming visible to everyone else. All of this happens transparently to the callers. The primary advantage is that if a caller never makes any modifications, no private copy need ever be created.

意思上就是：在复制一个对象的时候并不是真正的把原先的对象复制到内存的另外一个位置上，而是在新对象的内存映射表中设置一个指针，指向源对象的位置，并把那块内存的Copy-On-Write位设置为1.

这样，在对新的对象执行读操作的时候，内存数据不发生任何变动，直接执行读操作；而在对新的对象执行写操作时，将真正的对象复制到新的内存地址中，并修改新对象的内存映射表指向这个新的位置，并在新的内存位置上执行写操作。

这个技术需要跟虚拟内存和分页同时使用，好处就是在执行复制操作时因为不是真正的内存复制，而只是建立了一个指针，因而大大提高效率。但这不是一直成立的，如果在复制新对象之后，大部分对象都还需要继续进行写操作会产生大量的分页错误，得不偿失。所以COW高效的情况只是在复制新对象之后，在一小部分的内存分页上进行写操作。

在Linux程序中，fork（）会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，linux中引入了“写时复制“技术，也就是只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

那么子进程的物理空间没有代码，怎么去取指令执行exec系统调用呢？

 在fork之后exec之前两个进程用的是
*相同的物理空间（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间*，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个。当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间，如果不是因为exec，内核会给子进程的数据段、堆栈段分配相应的物理空间（至此两者有各自的进程空间，互不影响），而代码段继续共享父进程的物理空间（两者的代码完全相同）。而如果是因为exec，由于两者执行的代码不同，子进程的代码段也会分配单独的物理空间。

在网上看到还有个细节问题就是，fork之后内核会通过将子进程放在队列的前面，以让子进程先执行，以免父进程执行导致写时复制，而后子进程执行exec系统调用，因无意义的复制而造成效率的下降。


