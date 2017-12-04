
#####练习markdown语法
**加粗**加粗了吗？


###第一章

1. 第一节
2. 第二节
    * 第一小节
    * [***链接***](http://baidu.com)
    * 插入图片
    * ![](img/2017-12-03_172341.png)
---

###表格
|   tables    |
|-------------|
| 感觉不好用啊 |

### 代码
``` java
public abstract class ProcessEngineConfigurationImpl extends ProcessEngineConfiguration {

    protected List<SessionFactory> customSessionFactories;
    protected Map<Class<?>, SessionFactory> sessionFactories;
 

    protected void initSessionFactories() {
        if (this.sessionFactories == null) {
            this.sessionFactories = new HashMap();
            if (this.dbSqlSessionFactory == null) {
                this.dbSqlSessionFactory = new DbSqlSessionFactory();
            }

            this.dbSqlSessionFactory.setDatabaseType(this.databaseType);
            this.dbSqlSessionFactory.setIdGenerator(this.idGenerator);
            this.dbSqlSessionFactory.setSqlSessionFactory(this.sqlSessionFactory);
            this.dbSqlSessionFactory.setDbIdentityUsed(this.isDbIdentityUsed);
            this.dbSqlSessionFactory.setDbHistoryUsed(this.isDbHistoryUsed);
            this.dbSqlSessionFactory.setDatabaseTablePrefix(this.databaseTablePrefix);
            this.dbSqlSessionFactory.setTablePrefixIsSchema(this.tablePrefixIsSchema);
            this.dbSqlSessionFactory.setDatabaseCatalog(this.databaseCatalog);
            this.dbSqlSessionFactory.setDatabaseSchema(this.databaseSchema);
            this.dbSqlSessionFactory.setBulkInsertEnabled(this.isBulkInsertEnabled, this.databaseType);
            this.dbSqlSessionFactory.setMaxNrOfStatementsInBulkInsert(this.maxNrOfStatementsInBulkInsert);
            this.addSessionFactory(this.dbSqlSessionFactory);
            this.addSessionFactory(new GenericManagerFactory(AttachmentEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(CommentEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(DeploymentEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(ModelEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(ExecutionEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(HistoricActivityInstanceEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(HistoricDetailEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(HistoricProcessInstanceEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(HistoricVariableInstanceEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(HistoricTaskInstanceEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(HistoricIdentityLinkEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(IdentityInfoEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(IdentityLinkEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(JobEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(ProcessDefinitionEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(ProcessDefinitionInfoEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(PropertyEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(ResourceEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(ByteArrayEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(TableDataManager.class));
            this.addSessionFactory(new GenericManagerFactory(TaskEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(VariableInstanceEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(EventSubscriptionEntityManager.class));
            this.addSessionFactory(new GenericManagerFactory(EventLogEntryEntityManager.class));
            this.addSessionFactory(new DefaultHistoryManagerSessionFactory());
            this.addSessionFactory(new UserEntityManagerFactory());
            this.addSessionFactory(new GroupEntityManagerFactory());
            this.addSessionFactory(new MembershipEntityManagerFactory());
        }

        if (this.customSessionFactories != null) {
            Iterator var1 = this.customSessionFactories.iterator();

            while(var1.hasNext()) {
                SessionFactory sessionFactory = (SessionFactory)var1.next();
                this.addSessionFactory(sessionFactory);
            }
        }

    }
}
```
<meta http-equiv="refresh" content="0.1">