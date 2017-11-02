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
        <signal id="alertSignal" name="alert" />                         <!--定义信号-->

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

5、触发信号事件

信号的触发可以由流程实例的结点触发，也可以通过API触发

```
RuntimeService.signalEventReceived(String signalName);   // 发送给全局的处理器

RuntimeService.signalEventReceived(String signalName, String executionId); // 发送给局部的处理器
```

6、信号事件的捕获

信号事件可以被中间捕获信号事件或者边界事件捕获。

7、查询信号事件的订阅

```
List<Execution> executions = runtimeService.createExecutionQuery().signalEventSubscriptionName("alert").list();
```

8、信号事件的范围

默认信号会在流程引擎范围内进行广播。也就是说，在流程实例A中抛出一个信号事件，其他不同的流程B和C都可以监听到这个事件。

```
<signal id="alertSignal" name="alert" activiti:scope"processInstance"/>// 定义信号时设置信号的传播范围为实例流程内
```

9、消息事件的定义

* 消息事件的定义需要引用一个命名的消息
* 每个消息都必须有名称和内容
* 消息事件总会直接发送给一个接受者

```
<definitions id="definitions"
  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
  xmlns:activiti="http://activiti.org/bpmn"
  targetNamespace="Examples"
  xmlns:tns="Examples">

  <message id="newInvoice" name="newInvoiceMessage" />  <!--定义消息-->
  <message id="payment" name="paymentMessage" />        <!--定义消息-->

  <process id="invoiceProcess">

    <startEvent id="messageStart" >
        <messageEventDefinition messageRef="newInvoice" /> <!--定义消息事件-->
    </startEvent>
    ...
    <intermediateCatchEvent id="paymentEvt" >
        <messageEventDefinition messageRef="payment" />  <!--消息捕获-->
    </intermediateCatchEvent>
    ...
  </process>

</definitions>
```

10、触发消息事件

* 如果消息应该启动触发一个新流程，则使用以下方法，触发消息事件

```
ProcessInstance startProcessInstanceByMessage(String messageName);
ProcessInstance startProcessInstanceByMessage(String messageName, Map<String, Object> processVariables);
ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey, Map<String, Object> processVariables);
```

* 如果消息要在已经运行的实例中处理，则使用下面的方法触发事件

```
void messageEventReceived(String messageName, String executionId);
void messageEventReceived(String messageName, String executionId, HashMap<String, Object> processVariables);
```

11、查询消息事件的订阅

activiti支持消息开始事件和中间消息事件

* 消息开始事件的订阅查询

```
ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
      .messageEventSubscription("newCallCenterBooking")
      .singleResult();
```

* 中间消息事件的订阅查询

```
Execution execution = runtimeService.createExecutionQuery()
      .messageEventSubscriptionName("paymentReceived")
      .variableValueEquals("orderId", message.getOrderId())
      .singleResult();
```

12、空开始事件

空开始事件表示流程定义时没有指定启动流程的触发事件，流程的启动必须通过api接口实现。

13、定时开始事件

指定某个事件点创建流程实例，或者设定时间间隔来多次启动流程。

注意：

* 子流程不能使用定时开始事件
* 定时开始事件在流程发布后开始计算时间。
* 当流程部署新的版本时，旧的定时器将被删除，新的计时器开始计时。

14、消息开始事件

使用一个命名的消息来启动流程实例，可以通过使用消息名称来选择正确的开始事件。

注意事项：

* 消息开始事件的名称必须唯一，否则通过消息开始流程实例时activiti会抛出异常。
* 消息开始事件名称在所有已发布的流程定义中不能重复，否则通过消息开始流程实例时activiti会抛出异常。
* 流程版本，在发布新版本的流程定义时，之前订阅的消息订阅将会被取消，即便新版本中没有消息事件也会这样处理。

通过消息开启流程实例的api：

```
ProcessInstance startProcessInstanceByMessage(String messageName);
ProcessInstance startProcessInstanceByMessage(String messageName, Map<String, Object> processVariables);
ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey, Map<String, Object< processVariables);
```

注意：

* 消息开始事件只支持顶级流程，不支持内嵌子流程。
* 如果流程定义有多个消息开始事件和一个空开始事件，runtimeService.startProcessInstancdByKey\(...\)和runtimeService.startProcessInstanceById\(...\)会使用空开始事件启动流程实例。
* 如果流程定义没有空开始事件且有多个消息开始事件，使用runtimeService.startProcessInstancdByKey\(...\)和runtimeService.startProcessInstanceById\(...\)会抛出异常。
* 如果流程定义只有一个消息开始事件，runtimeService.startProcessInstancdByKey\(...\)和runtimeService.startProcessInstanceById\(...\)会使用这个消息开始事件启动流程实例。
* 如果流程被调用环节启动（callActivity），消息开始事件只支持【1：在消息开始事件以外，还有一个单独的空开始事件】【2：只有一个消息开始事件，没有空开始事件】

15、信号开始事件

描述：通过一个已命名的信号来启动流程实例。

触发方式：信号可以在流程实例内部使用“中间信号抛出事务”触发，也可以通过runtimeService.signalEventReceivedXXX方法触发。两种情况下，所有流程中拥有相同名称的signalStartEvent都会启动。

16、错误开始事件

描述：错误开始事件可以用来触发一个子流程。开始错误事件不能用了启动流程实例。

17、结束事件

结束事件表示流程（或子流程）的结束。结束事件都是触发事件。当流程到达结束事件，会触发一个结果。

18、空结束事件

描述：当空结束事件到达时，不会抛出结果，只是结束当前执行的分支，不做其它事情。

19、错误结束事件

描述：当流程执行到错误结束事件，流程的当前分支就会结束，并抛出一个错误。这个错误可以被对应的中间边界错误事件捕获。如果找不到匹配的边界错误事件，就会抛出一个异常。

20、取消结束事件

取消结束事件只能与BPMN事务子流程结合使用。当到达取消结束事件时，会抛出取消事件，它必须被取消边界事件捕获。取消边界事件会取消事务，并触发补偿机制。

21、边界事件

描述：边界事件都是捕获事件，它会附在一个环节上。当节点运行时，事件会监听对应的触发类型，当事件被捕获，节点就会中断，同时执行事件的后续连线。

22、定时边界事件

描述：定时边界事件就是一个暂停等待警告的时钟。当流程执行到绑定了边界的环节，就会启动一个定时器。当定时器触发时，环节就会中断。并沿着定时边界事件外出连线继续执行。

注意：

* 边界定时事件只能在job执行器启用时使用

23、错误边界事件

描述：结点边界上的中间捕获错误事件，捕获节点范围内抛出的错误。

使用：大多用于内嵌子流程，或者调用节点。错误是由错误结束事件抛出的，错误会传递给上层作用域，直到找到一个错误事件定义相匹配的边界错误事件。当捕获了错误事件，边界任务绑定的节点就会销毁，也会销毁内部的所有执行分支。流程执行会继续沿着边界事件的外出连线继续执行。





















































