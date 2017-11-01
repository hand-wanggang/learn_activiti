# Activiti的引擎、服务和API

一、activiti的引擎和服务结构![](/assets/import-engine-1.png)1、ProcessEngineConfiguration为初始化ProcessEngine提供了配置，以下是ProcessEngineConfiguration的配置：

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



