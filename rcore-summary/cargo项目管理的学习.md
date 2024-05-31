---
mindmap-plugin: basic
---
#### 项目结构
+ 根目录下的[build.rs文件](https://course.rs/cargo/reference/build-script/intro.html)
	+ 在项目被构建之前，Cargo 会将构建脚本编译成一个可执行文件，然后运行该文件并执行相应的任务。可以更好的链接c依赖库等

#### 调试项目
+ 可以通过rust-objdump工具反汇编内核or应用程序可执行文件
+ [gdb 调试rcore](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/4first-instruction-in-kernel2.html#gdb)
	+ [GDB命令整理](https://www.cnblogs.com/JCpeng/p/15139424.html)
	+ [gdb调试riscv](https://gitlab.eduxiji.net/educg-group-17064-1466468/202310006101080-2675/-/blob/fc9aaa441729a7e6d53a70f12f9abf11cdc5bad3/docs/RISCV%E5%AF%84%E5%AD%98%E5%99%A8%E4%B8%8Egdb%E8%B0%83%E8%AF%95%E6%8C%87%E5%8D%97.md)
```shell
# qemu 启动
qemu-system-riscv64 \
    -machine virt \
    -nographic \
    -bios ../bootloader/rustsbi-qemu.bin \
    -device loader,file=target/riscv64gc-unknown-none-elf/debug/os.bin,addr=0x80200000 \
    -s -S
    
# gdb调试
riscv-linux-gnu-gdb \
    -ex 'file target/riscv64gc-unknown-none-elf/debug/os' \
    -ex 'set arch riscv:rv64' \
    -ex 'target remote localhost:1234'
```


gdb 调试rcore时出现传入的参数混乱的情况，并且执行下一条指令的时候不符合预期
+ 运行ci-user中的`make test`之后会把os文件夹下的makefile覆盖掉，此时就不会`make run`就不会生成os.bin文件只生成os文件，而gdb是运行os.bin文件，所以gdb一直再用之前的os.bin文件在运行