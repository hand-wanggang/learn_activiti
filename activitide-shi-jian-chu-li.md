# Activiti的事件处理

**一、activiti支持的事件类型**

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

**二、监听器的实现**

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

**三、事件**

1、在BPMN中事件被分为两大类：捕获和触发。

* 捕获事件：当执行到某个流程，事件会等待被触发（如：超时事件、边界出错事件）
* 触发时间：当执行到某个流程是，触发事件（如：执行完成事件、终止事件）

2、定时器事件的定义（定时器时间可用于开始事件、中间事件、边界事件）

定时器事件的触发方式：

* timeDate-根据特定的时间进行触发
* timeDuration-更具时间间隔触发
* timeCycle-使用重复执行间隔（可用Corn表达式指定）

注意：以上这些方式的参数都可以通过流程中的变量来设置，以达到动态的效果。

3、错误事件的定义

```
<endEvent id="myErrorEndEvent">
  <errorEventDefinition errorRef="myError" />
</endEvent>
```

引用相同error元素的错误事件处理器会捕获这个错误。本例中应用额错误是"myError"。

4、信号事件定义

信号事件会引用一个已命名的信号，信号全局范围的时间（广播）。会发送给所有激活的处理器。

下面的流程实例会抛出一个信号并被中间事件捕获

```
<definitions... >
        <!-- declaration of the signal -->
        <signal id="alertSignal" name="alert" />              <!--定义信号-->

        <process id="catchSignal">
                <intermediateThrowEvent id="throwSignalEvent" name="Alert">
                        <!-- signal event definition -->
                        <signalEventDefinition signalRef="alertSignal" /> <!--抛出信号-->
                </intermediateThrowEvent>
                ...
                <intermediateCatchEvent id="catchSignalEvent" name="On Alert">
                        <!-- signal event definition -->
                        <signalEventDefinition signalRef="alertSignal" /> <!--捕获信号-->
                </intermediateCatchEvent>
                ...
        </process>
</definitions>
```

























