# Lab1 Report
## 总结实现
* 为`TaskControlBlock`添加`start_time: usize`和`syscall_times: [u32; MAX_SYSCALL_NUM]`属性，用于统计运行时间以及系统调用次数
* 运行时间方面，在任务第一次被调度时(`run_first_task`时、以及`run_next_task`且下一个`task`是第一次被调度，即`start_time`还是初值0时)，记录`start_time`为当前系统时间。提供`get_current_task_time`函数用于获取当前时间与开始时间之差，即运行时间
* 系统调用次数方面，提供`on_syscall(syscall_id: usize)`函数，当前系统调用+1。在触发`syscall`，还未分发到具体`syscall`处理时执行该函数。

## 简答题
### 1
* SBI version: `[rustsbi] RustSBI version 0.4.0-alpha.1, adapting to RISC-V SBI v2.0.0`
* `ch2b_bad_address`: 在用户态写入`0x0`地址，会触发`PageFault`，在`trap_handler`中会杀死当前进程，并切换到下一个。相关日志：`[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804803a4, kernel killed it.`
* `ch2b_bad_instructions`:在用户态执行`sret`这一`S特权级`命令，触发`IllegalInstruction`中断。相关日志：`[kernel] IllegalInstruction in application, kernel killed it.`
* `ch2b_bad_register`: 在用户态尝试获取`sstatus`这一特权寄存器，会触发`IllegalInstruction`中断。相关日志：`[kernel] IllegalInstruction in application, kernel killed it.`

### 2
* `__alltraps`: 作为CPU的`Trap`入口，被写到`stvec`寄存器中，当发生陷阱时CPU会跳转到此进行处理。而真正的`trap_handler`是调用`Rust`函数来分发、处理的，`__alltraps`只负责在调用`trap_handler`之前将`cpu`从应用态转换到内核态，以及保存用户态正在运行的应用的上下文到内核栈当中
* `__restore`: 用内核栈上Trap的上下文恢复回陷入内核态之前的状态，包括恢复保存寄存器和`ra`、内核栈退栈、对换内核栈与用户栈指针（`csrrw sp, sscratch, sp`），这样执行完`sret`后就能完美恢复到用户态控制流。与`__alltraps`是逆过程
1. a0存储内核栈`push_context`的返回值，也就是内核栈上`trap context`的地址，通过它可以得到要返回的地址、寄存器等信息。`__restore`两种使用场景：每个任务第一次执行时由于`task_context`的`goto_restore`手动初始化返回地址为`__restore`地址，导致`run_first_task`的`__switch`能够主动跳转到`__restore`执行，实现第一次跳转到用户态；之后`trap`到内核态时，`__alltraps`过程`call`完`trap_handler`之后并没有返回，所以会接着自动执行`__restore`，从而回到用户态
2. `sstatus`、`sepc`和`x2`(`sp`)；分别用于切换回用户特权级、回到用户态要执行的指令地址、恢复用户栈
3. `x2`是`sp`，在后面手动处理（弹栈、切换用户栈和内核栈）；`x4`是`thread pointer`，目前用不到
4. `sp`指向用户栈，而`sscratch`存储内核栈指针，等待下一次`trap`时使用
5. 最后一行`sret`，因为会自动从`CSR`寄存器中恢复到之前的特权级模式
6. `sp`指向内核栈，而`sscratch`存储用户栈指针，待`__restore`时调换回来
7. 用户态（U特权级）发生`ecall`，处理器接收到`Trap`时会自动切换为S特权级，然后才进入内核态的`__alltraps`过程[实现特权级的切换(rCore-Tutorial-Book-v3)](https://rcore-os.gitcode.host/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#)

## 荣誉准则
1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 ChatGPT、Microsoft Copilot 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

特权级CSR相关汇编指令的含义与使用

2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

* [rCore-Tutorial-Guide 2024A文档](https://learningos.cn/rCore-Camp-Guide-2024A/chapter3)
* [rCore-Tutorial-Book](https://rcore-os.cn/rCore-Tutorial-Book-v3)
* [汇编（一）：risc-v汇编语法 Assembler Directive(知乎)](https://zhuanlan.zhihu.com/p/588075416)
* 《计算机组成与设计 硬件/软件接口 RISC-V版》

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。