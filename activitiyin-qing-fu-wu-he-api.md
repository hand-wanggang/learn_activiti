# Activiti的引擎、服务和API

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
```



