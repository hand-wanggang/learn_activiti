# Activiti基础知识及实例讲解（完结）

通过这部分内容的学习，可以快速了解工作流是什么，以及activiti引擎的结构、配置、api、事件、表单、数据库表结构以及流程的设计。

什么是工作流？

工作流就相当于一个水道，水道有各种各样的闸口控制水的流向。其中的水我们可以理解为数据。在工作流中我们可以按照特定的业务根据它的规则设计一个独特的水道图纸。当有业务数据需要处理时，我们就根据图纸建造一个实际的水道，并让业务数据按照水道的规则流动。

# chapter1：Activiti的引擎结构和基础API

**一、activiti的引擎和服务结构**![](/assets/import-engine-1.png)1、ProcessEngineConfiguration为初始化ProcessEngine提供了配置，以下是ProcessEngineConfiguration的配置：

```
#配置数据源
    @Bean
    public HikariConfig hikariConfig() {
        HikariConfig hikariConfig = new HikariConfig();
        hikariConfig.setJdbcUrl(dataSourceProperties.getJdbcUrl());
        hikariConfig.setDriverClassName(dataSourceProperties.getDriverClassName());
        hikariConfig.setUsername(dataSourceProperties.getUserName());
        hikariConfig.setPassword(dataSourceProperties.getPassword());
        hikariConfig.setMaximumPoolSize(dataSourceProperties.getMaximumPoolSize());
        hikariConfig.setMaxLifetime(dataSourceProperties.getMaxLifeTime());
        hikariConfig.setConnectionTimeout(dataSourceProperties.getConnectionTimeout());
        hikariConfig.setIdleTimeout(dataSourceProperties.getIdelTimeout());
        return hikariConfig;
    }
# activity的表单属性解析器。如果需要配置自定义的属性解析器，需要覆盖原有属性，如下DateFormType的时间解析格式    
        @Bean
    public FormTypes formTypes(){
        FormTypes formTypes = new FormTypes();
        formTypes.addFormType(new StringFormType());
        formTypes.addFormType(new LongFormType());
        formTypes.addFormType(new DateFormType("yyyy-MM-dd"));
        formTypes.addFormType(new BooleanFormType());
        formTypes.addFormType(new DoubleFormType());
        return formTypes;
    }

#配置ProcessEngineConfiguration    
    @Bean
    public SpringProcessEngineConfiguration processEngineConfiguration() {
        SpringProcessEngineConfiguration processEngineConfiguration = new SpringProcessEngineConfiguration();
        processEngineConfiguration.setDataSource(dataSource()); // 设置数据源
        processEngineConfiguration.setTransactionManager(dataSourceTransactionManager());// 配置事务管理
        // 配置数据库
        processEngineConfiguration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
        // 配置是否启动job执行和异步执行，默认为true
        processEngineConfiguration.setJobExecutorActivate(Boolean.FALSE);
        // 配置历史信息级别。默认为 audit
        processEngineConfiguration.setHistory(HistoryLevel.FULL.getKey());
        // 配置activity的表单属性解析器
        processEngineConfiguration.setFormTypes(formTypes());
        return processEngineConfiguration;
    }
```

2、RepoistoryService的基本功能

* 查询引擎中的发布包和流程定义。

* 暂停或激活发布包，对应全部和特定流程定义。 暂停意味着它们不能再执行任何操作了，激活是对应的反向操作。

* 获得多种资源，像是包含在发布包里的文件， 或引擎自动生成的流程图。

* 获得流程定义的pojo版本， 可以用来通过java解析流程，而不必通过xml。

3、RuntimeService的基本功能

* 查询和启动流程实例
* 查询和保存流程运行中的变量

4、TaskService的基本功能

* 分配和查询用户和用户组任务
* 创建独立运行的任务，这些任务与实例无关
* 认领完成一个任务。

5、IdentityService的功能

* CRUD用户和用户组

6、FormService（动态表单时是否有用）

* 提供启动表单服务（流程启动时的表单服务，可以获取表单属性）
* 提供任务表单服务（任务完成时的表单服务，可以获取表单属性）

7、HistoryService提供引擎的所有历史记录

* 流程启动时间
* 任务的参与者
* 任务完成时间
* 每个流程实例的执行路径

**二、完整的Activi流程**

demo的git地址:[https://github.com/hand-wanggang/activiti-demo.git](https://github.com/hand-wanggang/activiti-demo.git)

1、流程模型的设计

流程的模型设计，可以通过XML文件的方式来设计（比较复杂）也可以通过可视化的流程设计器来实现。

2、流程模型的部署

* [ ] 通过文件的方式部署

```
repositoryService.createDeployment()
  .name("expense-process.bar")
  .addClasspathResource("org/activiti/expenseProcess.bpmn20.xml")
  .addClasspathResource("org/activiti/expenseProcess.png")
  .deploy();
```

* [ ] 通过输入流的形式来部署

```
    @GetMapping("/deploy/{modelId}")
    public void deploy(@PathVariable("modelId") String modelId) throws IOException {
        Model modelData = repositoryService.getModel(modelId);
        ObjectNode modelNode = (ObjectNode) new ObjectMapper()
                .readTree(repositoryService.getModelEditorSource(modelData.getId()));
        byte[] bpmnBytes = null;
        BpmnModel model = new BpmnJsonConverter().convertToBpmnModel(modelNode);
        bpmnBytes = new BpmnXMLConverter().convertToXML(model);
        String processName = modelData.getName() + ".bpmn20.xml";
        System.out.println(new String(bpmnBytes,"UTF-8"));
        repositoryService.createDeployment().name(modelData.getName())
                .addString(processName, new String(bpmnBytes, "UTF-8")).deploy();
    }
```

3、根据流程定义启动流程实例

* [ ] 使用runtimeService.startProcessInstanceByKey启动

```
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("financialReport");
```

* [ ] 使用FormService的submitStartFormData来启动

```
ProcessInstance instance = formService.submitStartFormData(processDefinitionId, values);
```

4、获取和执行流程任务

* [ ] 查询流程任务

```
List<Task> tasks = taskService.createTaskQuery().taskCandidateUser("kermit").list();// 查询个人任务


List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup("accountancy").list();// 用户组任务
```

* [ ] 执行流程任务

```
taskService.complete(task.getId()); // 完成任务，也可以在完成任务前获取任务节点的表单


// 获取节点的表单
@GetMapping("/form")
public ResponseEntity taskForm(@RequestParam String taskId) {
        List<ReadOnlyFormData> data = dynamicFormPropertiesAndValue(taskId);
        List<FormProperty> properties = formService.getTaskFormData(taskId).getFormProperties();
        FormData formData = new FormData();
        formData.setRead(data);
        formData.setWrite(properties);
        return ResponseEntity.ok(formData);
}


// 判断一个流程是否结束
ProcessInstance rpi = processEngine.getRuntimeService()//
                .createProcessInstanceQuery()//创建流程实例查询对象
                .processInstanceId(pi.getId())
                .singleResult();
// 如果rpi为空则表示流程结束，否则还未结束
```

# chapter2：Activiti的历史信息级别及其配置

**一、activiti的历史信息级别**

1、none:忽略所有的历史存档。性能最好，但是没有任何历史信息可用。

2、activity保存所有流程实例信息和活动实例信息，在流程实例结束时，最后一个流程实例中的最新变量将赋值给历史变量。不会保存过程中的详细信息。

3、audit：默认的历史信息级别，保存所有实例信息、活动信息、保证所有变量和提交的表单属性保持同步这样所有用户交互信息都可以追溯，可以用来审计。

4、full：最高级别的历史信息存档，同时最慢。这个级别存储发生在审核以及所有细节信息，和流程变量信息都会保存。

**二、配置activiti的历史信息级别**

```
processEngineConfiguration.setHistory(HistoryLevel.FULL.getKey());
```

**三、历史信息使用实例**

```
    @GetMapping("/formProperties/{processInstanceId}")
    public ResponseEntity formProperties(@PathVariable("processInstanceId") String processInstanceId) {
        List<HistoricDetail> details = historyService.createHistoricDetailQuery().processInstanceId(processInstanceId)
                .formProperties().list();
        return ResponseEntity.ok(details);
    }
```

以上代码，可以用来查询具体的一个实例在运行过程中的表单属性。这些属性和值都是作为历史信息存储在"act-hi-deatil表中的。

# chapter3：Activiti的表单

activiti支持表单的方式有两种：内置表单（动态表单）和外置表单两种

**一、内置表单**

1.内置表单又叫动态表单。内置表单是在流程的定义阶段就存在的。表单的所有属性字段都存储在流程定义的xml文件中。

下面的例子中,在结点task上定义了一个表单。表单属性有：room，duration，speaker，street。

```
<userTask id="task">
  <extensionElements>
    <activiti:formProperty id="room" />
    <activiti:formProperty id="duration" type="long"/>
    <activiti:formProperty id="speaker" variable="SpeakerName" writable="false" />
    <activiti:formProperty id="street" expression="#{address.street}" required="true" />
  </extensionElements>
</userTask>
```

2、activiti当前默认的表单属性的类型有string,long,enum,date,boolean。这些类型可能不能满足我们的需求，所有activiti为我们提供了扩展这些属性的方法，继承AbstractFormType，并在ProcessEngineConfiguration中注册自定义的类型，即可。

3、内置表单属性的获取

* 获取开始节点表单属性

```
formService.getStartFormData(task.getProcessDefinitionId()).getFormProperties()
```

* 获得Task节点的表单属性

```
formService.getTaskFormData(taskId).getFormProperties()
```

* 表单属性的结构

```
public interface FormProperty extends Serializable {
    String getId();    // 表单属性的唯一标识

    String getName();  // 表单属性名称

    FormType getType(); // 属性类型

    String getValue();  // 属性值

    boolean isReadable();

    boolean isWritable();

    boolean isRequired();
}
```

4、内置表单的渲染

内置表单的渲染需要前端的UI进行渲染。

**二、外置表单**

外置表单需要我们将自定义的表单模板作为资源文件存储到数据库中，并通过formKey，获取到该模板。

```
#获取外置表单的唯一标识
FormService.getStartFormData(String processDefinitionId).getFormKey()

FormService.getTaskFormData(String taskId).getFormKey()


#通过formKey获取到外置表单模板
InputStream RepositoryService.getResourceAsStream(String deploymentId, String resourceName)
```

**三、自定义表单属性类型**

1、继承AbstractFormType

```
public class JavascriptFormType extends AbstractFormType
{
    @override
    public string getName()
    { return "javascript";}

    @override 
    public Object convertFormValueToModelValue(String propertyValue)
    { return propertyValue;}

    @override 
    public String convertModelValueToFormValue(Object modelValue)
    { return (String) modelValue;}
}
```

2、注册该自定义类型

```
    @Bean
    public AbstractFormType customerFormType(){
        return new JavascriptFormType();
    }

    @Bean
    public SpringProcessEngineConfiguration processEngineConfiguration() {
        SpringProcessEngineConfiguration processEngineConfiguration = new SpringProcessEngineConfiguration();
        processEngineConfiguration.setDataSource(dataSource());
        processEngineConfiguration.setTransactionManager(dataSourceTransactionManager());
        processEngineConfiguration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
        processEngineConfiguration.setJobExecutorActivate(Boolean.FALSE);
        processEngineConfiguration.setHistory(HistoryLevel.FULL.getKey());
        processEngineConfiguration.setFormTypes(formTypes());
        processEngineConfiguration.setCustomFormTypes(new ArrayList<>());      // 此处注册用户自定义的表单字段类型
        processEngineConfiguration.getCustomFormTypes().add(customerFormType());
        return processEngineConfiguration;
    }
```

# chapter4：Activiti的事件及事件监听

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

**三、监听器的配置与安装**

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

**四、Activiti流程设计中的事件**

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

# chapter5：Activiti的数据表及其用处

activiti的持久层框架使用的是mybatis。支持的数据库：DB2、H2、Oracle、MySQL、MS SQL、PostgerSQL等。

**一、数据库表的命名规程**

1、历史信息等级对数据表的影响

默认情况下activiti会在数据库中生成23张表，那是因为它的默认历史信息等级为audit。这种历史等级下，不会创建**ACT\_HI\_VARIABLE**和**ACT\_HI\_VARINST**表，这两张表保存中流程中每一个节点涉及到的变量及其值。

2、Activiti中标的规则

* **ACT\_RE\_**\*: 'RE'表示repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。

* **ACT\_RU\_**\*: 'RU'表示runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。

* **ACT\_ID\_**\*: 'ID'表示identity。 这些表包含身份信息，比如用户，组等等。

* **ACT\_HI\_**\*: 'HI'表示history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。

* **ACT\_GE\_**\*: 通用数据， 用于不同场景下。

**二、不同表的作用说明**

|  | **表名** | **解释** |
| :--- | :--- | :--- |
| 一般数据 | ACT\_GE\_BYTEARRAY | 通用的流程定义和流程资源 |
|  | ACT\_GE\_PROPERTY | 系统相关属性 |
| 流程历史记录 | ACT\_HI\_ACTINST | 历史的流程实例 |
|  | ACT\_HI\_ATTACHMENT | 历史的流程附件 |
|  | ACT\_HI\_COMMENT | 历史的说明性信息 |
|  | ACT\_HI\_DETAIL | 历史的流程运行中的细节信息 |
|  | ACT\_HI\_IDENTITYLINK | 历史的流程运行过程中用户关系 |
|  | ACT\_HI\_PROCINST | 历史的流程实例 |
|  | ACT\_HI\_TASKINST | 历史的任务实例 |
|  | ACT\_HI\_VARINST | 历史的流程运行中的变量信息 |
| 用户用户组表 | ACT\_ID\_GROUP | 身份信息-组信息 |
|  | ACT\_ID\_INFO | 身份信息-组信息 |
|  | ACT\_ID\_MEMBERSHIP | 身份信息-用户和组关系的中间表 |
|  | ACT\_ID\_USER | 身份信息-用户信息 |
| 流程定义表 | ACT\_RE\_DEPLOYMENT | 部署单元信息 |
|  | ACT\_RE\_MODEL | 模型信息 |
|  | ACT\_RE\_PROCDEF | 已部署的流程定义 |
| 运行实例表 | ACT\_RU\_EVENT\_SUBSCR | 运行时事件 |
|  | ACT\_RU\_EXECUTION | 运行时流程执行实例 |
|  | ACT\_RU\_IDENTITYLINK | 运行时用户关系信息 |
|  | ACT\_RU\_JOB | 运行时作业 |
|  | ACT\_RU\_TASK | 运行时任务 |
|  | ACT\_RU\_VARIABLE | 运行时变量表 |



