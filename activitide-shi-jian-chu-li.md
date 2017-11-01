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

二、监听器的实现

通过实现org.activiti.engine.delegate.event.ActivitiEventListener接口，可以实现一个监听器。

![](/assets/import-event-1.png)

实现onEvent方法和isFailOnException\(\)方法即可。

注意：isFailOnException方法决定了当onEvent\(\)方法抛出异常时，是否将异常像上次抛出还是忽略异常。如果isFailOnException返回false则忽略，返回true则继续向上传播，导致当前命令失败。

三、监听器的配置与安装

1、设置监听的事件的类型进行配置（只有当配置的事件类型的事件被抛出时，监听器才会触发）

```
<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    ...
    <property name="typedEventListeners">
      <map>
        <entry key="JOB_EXECUTION_SUCCESS,JOB_EXECUTION_FAILURE" >
          <list>
            <bean class="org.activiti.engine.example.MyJobEventListener" />
          </list>
        </entry>
      </map>
    </property>
</bean>
```

2、不设置监听事件类型的配置（任何事件触发都会被触发监听）

```
<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    ...
    <property name="eventListeners">
      <list>
         <bean class="org.activiti.engine.example.MyEventListener" />
      </list>
    </property>
</bean>
```

3、运行时使用RuntimeService的api进行运行添加（运行时添加的监听，当activiti的引擎重启后无效）

```
    void addEventListener(ActivitiEventListener var1);

    void addEventListener(ActivitiEventListener var1, ActivitiEventType... var2);

    void removeEventListener(ActivitiEventListener var1);
```




















