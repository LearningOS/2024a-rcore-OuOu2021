# Lab1 Report
## 总结实现
* 为`TaskControlBlock`添加`start_time: usize`和`syscall_times: [u32; MAX_SYSCALL_NUM]`属性，用于统计运行时间以及系统调用次数
* 运行时间方面，在任务第一次被调度时(`run_first_task`时、以及`run_next_task`且下一个`task`是第一次被调度，即`start_time`还是初值0时)，记录`start_time`为当前系统时间。提供`get_current_task_time`函数用于获取当前时间与开始时间之差，即运行时间
* 系统调用次数方面，提供`on_syscall(syscall_id: usize)`函数，当前系统调用+1。在触发`syscall`，还未分发到具体`syscall`处理时执行该函数。

## 简答题
#TODO 