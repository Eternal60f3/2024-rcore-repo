---
mindmap-plugin: basic
---

#### [[risc-V|risc-V 指令集]]


#### ch1 构造能够直接在裸机上运行的程序
+ 构造一个最小程序，以便于在***裸机***上执行  (注: 此时还没有操作系统的出现，是直接将程序运行在裸机上)
	+ [对源程序进行的修改](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/2remove-std.html)
		+ 背景知识: [[Pasted image 20240501234747.png|应用程序执行环境]]
		+ [具体步骤](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/2remove-std.html)
			+ 移除标准库
				+ 标准库里面包含了系统调用，因此需要操作系统的支持
			+ 移除main函数
				+ main函数依赖于标准库，在执行main函数之前还会进行很多初始化操作。现在没了标准库，也就无法适用main函数了
			+ 提供 panic_handler
				+ rust 会手动 or 自动调用`panic!`宏来处理出错信息，没了标准库, 也就没了错误处理，需要自定义一个
		+ 注：虽然移除了标准库，但rust 中还有一个`core`库提供一些基础操作(不涉及到系统调用，只是一些普通函数)
	+ 设置源程序的[编译流程](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#id8)中的具体操作
		+ 通过[链接脚本](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/4first-instruction-in-kernel2.html#id4)调整链接器生成的最终可执行文件的[内存布局](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#id7)
			+ 内存布局中的首地址0x80200000，可以理解为第一条指令的编号。后续将可执行程序加载到qemu裸机时，再把这个可执行程序加载裸机内存的0x80200000位置
	+ [嵌入汇编来执行程序的第一条指令](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/4first-instruction-in-kernel2.html#id3)
		+ [分配并使用栈](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#jump-practice)
			+ 这个栈的设计主要目的是给 `sp` 寄存器赋予合法的值
+ qemu 模拟裸机
	+ [qemu的了解](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#qemu)
	+ [qemu的启动流程](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#id5)
		+ 一阶段：执行`0x1000`处的几条简单的指令，跳转到`0x80000000` (特权级：m态)
		+ 二阶段：执行`0x80000000`处的bootloader (也就是 rustsbi)，进行一些初始化工作，如：设置将 u 态的异常交给 s 态处理(默认是 m 态进行处理) (特权级: m态)
		+ 三阶段：执行`0x80200000`处的os内核 (特权级: s 态)
			+ 也就是 ch1 的代码全称在内核态运行

#### ch2 开始出现操作系统来管理应用程序，区分内核态和用户态
+ 整体项目视角
	+ 构造包含操作系统内核和多个应用程序的单一执行程序，并在qemu模拟的裸机上运行
		+ 将应用程序作为执行程序的数据
	+ 该执行程序，应用态和内核态分离, ***本节操作系统的核心再于trap陷入的设计***。
		+ 应用态不能执行内核态的***指令***，但反过来可以
	+ 操作系统的作用：串行处理每一个应用程序，当前应用程序出错时，就执行下一个
+ [用户态](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/2application.html#id1)
	+ 调整内存布局，将首地址设为0x80400000
		+ 和操作系统的约定, 具体见后面内核态的设置
	+ 设计系统调用（是对ecall指令的封装）
+ 内核态
	+ 对应用程序的管理
		+ [把应用程序链接到内核的数据段](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/3batch-system.html#id3)
		+ 使用全局变量 [`AppManager` 结构体](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/3batch-system.html#id4)进行管理
			+ 当需要执行某个应用程序的时候，将应用程序二进制镜像从操作系统数据段 加载到 可以执行代码的text段`0x80400000`
			+ 用到了 `lazy_static!` 宏 + `UPSafeCell`类型(基于 `RefCell` 创建)
	+ [对于trap陷入的设计](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html)
		+ [硬件层面的特权级机制]([Fetching Title#mcpv](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html))  [riscv特权级机制](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#risc-v)
			+ [[Pasted image 20240503230707.png|寄存器]]
			+ [[Pasted image 20240505225310.png|作为中转的sscratch CSR]]
		+ 遇到异常(触发trap)的时候，会执行stvec寄存器处指定的trap处理代码的入口地址
			+ [运行前需要保存上下文, 运行后恢复上下文](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id8)
				+ 涉及到[内核栈和用户栈的设置](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id7)
					+ 注：ch1中分配那部分栈空间，在第一次执行`__restore`之后就不会再用到了，之后sp只在rust程序中新设置的`KernelStack`和`UserStack`中切换
		+ 利用[恢复上下文的过程从内核态转到用户态](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#ch2-app-execution)(执行下一个用户程序)
			+ 刚开始执行`rust_main`的时候处于内核态 (原理见上方qemu的启动流程)，当第一次执行`__restore`的时候转入用户态
	+ [系统调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id10)
		+ 目前的系统调用还非常简单

#### [[ch3 提高cpu利用率，并且支持同时存在多个用户的操作系统]]


#### [[ch4 加入了内存空间的操作]]




#### [[cargo项目管理的学习]]
