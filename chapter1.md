# Activiti的历史信息配置

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

