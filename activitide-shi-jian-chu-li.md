# Activiti的事件处理

一、activiti支持的事件类型

activiti支持的事件类型在org.activiti.engine.delegate.event.ActivitiEventType中利用枚举值的方式列出。常用的事件类型有：

1、JOB\__EXECUTION_\_SUCCESS作业执行成功

2、JOB\_EXECUTION\_FAILURE作业执行失败

3、TIMER\_FIRED触发了定时器

4、ACTIVITY\_STARTED一个节点开始执行

5、ACTIVITY\_COMPLETED 一个节点成功结束

6、ACTIVITY\_SIGNALED一个节点接收到一个信号

7、TASK\_ASSIGNED一个任务被分配给一个人员

8、TASK\_CREATE任务被创建

9、TASK\_COMPLETED任务被完成

10、TASK\_TIMEOUT 任务超时

