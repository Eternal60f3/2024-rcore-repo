---
mindmap-plugin: basic
---

+ why
	+ 安全性
	+ “有限”物理内存变成“无限”虚拟内存
	+ 统一的内存布局
+ [[Pasted image 20240506001839.png|需要思考的问题]]
+ [操作系统内核的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-raii)
+ [地址空间抽象的成因和用途](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#id8)
	+ [分段式实现方式](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#id7)
	+ [分页式实现方式](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#id8)
+ sv39多级页表 在操作系统端需要进行的一些操作
	+ [对于物理内存的管理](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#term-manage-phys-frame)
		+ 内核态虚拟地址和物理地址进行恒等映射。所以即使开启了分页模式，仍能直接使用物理内存进行进行访存
	+ [页表的设计](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#term-create-pagetable)
+ 本章涉及到的一些关键实现
	+ 硬件对与页表的支持
		+ 软件也页表机制的辅助
	+ 很妙的一个设计：trampoline的实现
+ 一些注意事项
	+ 对于当前使用unsafe来实现的 `PhysPageNum.get_*` 系列方法可以尝试着进行进一步改良，使得能够通过编译器来进行检查
	+ 在map和unmap之后为啥没有刷新TLB
		+ 由于应用和内核在不同的地址空间下，我们无需在每一次map/unmap之后都立即刷新TLB，只需在所有的操作结束后，即将切换回应用地址空间之前刷新一次TLB即可，这可以参考__restore的实现。这样做是由于刷新TLB是一个十分耗时的操作，需要尽可能避免不必要的刷新。
+ temp_question
	+ MapArea`data_frames`字段的意义，不是memoryset中的pagetable已经中已经存储存储了frames吗
		+ 答：便于管理
	+ 应用运行的时候，它的栈空间以及堆空间位于内核中的哪个位置。平时代码运行的时候，还遵循那个内存布局吗
		+ 答：
			+ 这个是ch3及之前才需要考虑的问题
				+ 在ch3中，内核栈和用户栈都是放在给定的变量处
			+ ch4引入地址空间后，内核地址空间和应用地址空间分离了，至于具体应用地址空间分配到那个物理页帧就不确定了
	+ rust如何对物理地址不连续的变量进行操作
		+ why: 为了处理`process.rs`中`sys_get_time`函数传入的ts指针指向的`TimeVal`跨越多页的情况
+ 待完成的任务
	+ 实现lazy分配
	+ 实现支持大页机制