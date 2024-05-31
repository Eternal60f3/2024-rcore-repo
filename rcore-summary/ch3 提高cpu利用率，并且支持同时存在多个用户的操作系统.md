---
mindmap-plugin: basic
---

+ why
	+ [协作式多任务](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/0intro.html#id3) (多道程序)
		+ 缘由：cpu运算速度提高，内容容量增大，但是io速度增长缓慢。
		+ 结果：app进行io操作时主动放弃cpu使用权，交给下一个cpu。
			+ 只能等待app主动放弃，至于app什么时候主动放弃取决于app的编写者
	+ [抢占式多任务](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/0intro.html#id4)（分时共享）
		+ 原因：想要让多个用户同时使用一台设备，所以不能让一个程序霸占cpu
		+ 结果：任务轮流使用一定时长的cpu，到时间直接切换到下一个。呈现多个app同时运行的状态。
			+ 操作系统可以主动打断应用程序
	+ 共同点
		+ 为了减少切换app的代价，可以事先把所有app加载到内存中。
	+ 本节的核心：
		+ 在trap控制流的基础上增加task抽象
		+ 以及系统调用的设计
+ [加载所有的app](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/1multi-loader.html)
	+ 注：目前应用程序的编址方式是基于绝对位置的，并没做到与位置无关，内核也没有提供相应的地址重定位机制。
+ [任务切换](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/2task-switching.html#id1)
	+ 相当于是从一个任务的trap控制流切换到另一个任务的trap控制流
	+ 任务的管理
		+ [任务管理器](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/3multiprogramming.html#term-coop-impl)
		+ 每个任务都分配了自己的内核栈和用户栈
		+ `task_context` 主要存储的是不同任务的trap控制流状态
+ [协作式多任务的实现](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/3multiprogramming.html)
	+ [加入yield系统调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/3multiprogramming.html#sys-yield-sys-exit)
+ [抢占式多任务](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html)
	+ 加入[时间中断](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#id6)
		+ [risc-v架构中的中断](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#risc-v)
