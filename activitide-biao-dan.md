# Activiti的表单

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



