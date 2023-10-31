# ch3总结报告

## 实现功能总结

本lab主要实现了sys_task_info功能，该函数主要包含以下三个功能：
1. 获取当前任务状态
2. 统计系统调用的次数
3. 统计运行时长

根据提示信息，在TaskControlBlock添加需要统计的信息：syscall_times， start_time
在os/src/task/mod.rs 添加相应函数的核心逻辑
+ get_current_task_status: 获取当前任务的状态比较，比较简单，直接参考上面的run_next_task函数的获取状态
+ get_syscall_times: 这里统计当前任务+=1, 结果保存在syscall_times数组中
+ get_current_run_time：根据提示，可以用get_time() 获取程序运行时间，查询时通过 当前时间 - 开始时间 得到目标结果

注意的坑：TaskControlBlock 中的变量记得需要初始化



## 简答作业

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容 (运行 Rust 三个 bad 测例 (ch2b_bad_*.rs) ， 注意在编译时至少需要指定 LOG=ERROR 才能观察到内核的报错信息) ， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。
    在低特权级使用更高特权级的指令 / 访问非法地址等都会触发异常
    [ERROR] [kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x80400408, core dumped.
    [ERROR] [kernel] IllegalInstruction in application, core dumped.
    [ERROR] [kernel] IllegalInstruction in application, core dumped.

2. 深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:
    + L40：刚进入 __restore 时，a0 代表了什么值。请指出 __restore 的两种使用情景
        a0 代表了一个指向 TrapContext 结构体的指针，该结构体保存了中断发生时的寄存器和状态信息。
        两种使用情景： 
        - 一种是在 trap_handler 函数的最后调用 __restore，这是正常的中断处理流程，trap_handler 负责处理中断并更新 TrapContext 中的 sepc 和 sstatus 等寄存器。
        - 另一种是在用户态程序执行系统调用时，内核会通过 sret 指令直接跳转到 __restore，而不经过 trap_handler。

    + L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
        - t0: 一个临时寄存器，用于保存和恢复 sstatus 的值。
        - t1: 一个临时寄存器，用于保存和恢复 sepc 的值。
        - t2: 一个临时寄存器，用于保存和恢复 sscratch 的值。
        - t0-t2仅作为临时变量，无特殊意义
        - sstatus: 一个控制和状态寄存器，用于保存和设置中断使能、特权级别、浮点状态等信息。
        - sepc: 一个控制和状态寄存器，通常指向中断发生时的下一条指令，或者系统调用返回后的下一条指令。
        - sscratch: 一个控制和状态寄存器，用于保存和设置用户态栈指针（sp）的值。
  
    + L50-L56：为何跳过了 x2 和 x4？
        - x2: 这个寄存器是栈指针（sp），它在中断发生时被保存到 sscratch 中
        - x4: 用不到，不需要保存

    + L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？
        sp 和 sscratch 分别指向 内核栈/用户栈 其中的一个和另一个, 执行之后他们的指向发生交换, 此处sp指向用户栈, sscratch指向进程内核栈

    + __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
    状态切换的指令：__restore 中发生状态切换的指令是 sret，它是一条特殊的返回指令，用于从特权态返回到低特权态。
     
    + L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？
        sp 和 sscratch 分别指向 内核栈/用户栈 其中的一个和另一个, 执行之后他们的指向发生交换, 此处sscratch指向用户栈, sp指向进程内核栈
    + 从 U 态进入 S 态是哪一条指令发生的？
        ecall

## 荣誉准则
1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：
    微信群的讨论

2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：
    http://jackming2271.xyz/2022/07/26/rcore%E5%AE%9E%E9%AA%8C%E6%8A%A5%E5%91%8A-%E5%85%A8Ch1-8/
    https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/6answer.html
3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。


