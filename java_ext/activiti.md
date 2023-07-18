# 1. Activiti介绍

# 2. 基础操作

## 2.1 初始化流程引擎

```java
// 使用代码或配置文件获取ProcessEngineConfiguration
    ProcessEngineConfiguration cfg = new StandaloneProcessEngineConfiguration()
      .setJdbcUrl("jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000")
      .setJdbcUsername("sa")
      .setJdbcPassword("")
      .setJdbcDriver("org.h2.Driver")
      .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
//调用buildProcessEngine()方法, 会在数据库里创建名称为act_*_*的表, 引擎初始化完毕
    ProcessEngine processEngine = cfg.buildProcessEngine();
```

## 2.2 部署流程定义

*.bpmn20.xml文件格式

```xml
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
             xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
             xmlns:dc="http://www.omg.org/spec/DD/20100524/DC"
             xmlns:di="http://www.omg.org/spec/DD/20100524/DI"
             targetNamespace="http://example.com/bpmn">

  <process id="myProcess" name="My Process">
    <!-- 流程定义的其他元素和逻辑 -->
  </process>

</definitions>
```

部署代码

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
RepositoryService repositoryService = processEngine.getRepositoryService();

//将*.bpmn20.xml文件描述的流程定义部署到引擎中
Deployment deploy = repositoryService.createDeployment()
                .name("出差申请流程") // 流程名称
                .addClasspathResource("onboarding.bpmn20.xml")
                .deploy();
// 部署后, 每个ProcessDefinition都有id(id和*.bpmn20.xml文件中的id相同)
ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
        .deploymentId(deployment.getId()).singleResult();
```

## 2.3 运行流程实例

```java
    RuntimeService runtimeService = processEngine.getRuntimeService();
    ProcessInstance processInstance = runtimeService
        .startProcessInstanceByKey("onboarding"); // 参数key == ProcessDefinition的id
//每个流程实例有一个id和流程实例对应的流程业务
instance.getBusinessKey()
processInstance.getProcessInstanceId()
//获取流程实例对应的流程定义的id
processInstance.getProcessDefinitionKey()
```

## 2.4 查询流程中角色的TodoList并完成Task

```java
	//1、获取流程引擎
	ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
	//2、获取taskService
	TaskService taskService = processEngine.getTaskService();
	//3、根据流程key 和 流程中的角色(一个角色可能负责流程中多个节点的任务) 查询任务该角色的待做任务
	List<Task> taskList = taskService.createTaskQuery()
                .processDefinitionKey("myEvection") //流程Key
                .taskAssignee("zhangsan")  //要查询的角色
		//.taskCandidateUser("wangwu")如果一个任务有多个负责人时, 可以使用该接口指定包含特定候选负责人的任务
                .list();

	//4、task相关字段
	for (Task task : taskList) {
            System.out.println("task所在的流程实例id="+task.getProcessInstanceId());
            System.out.println("任务Id="+task.getId());
            System.out.println("任务负责人="+task.getAssignee());
            System.out.println("任务名称="+task.getName());
	    taskService.complete(task.getId()) //完成任务, 流程实例走向下一个节点
	}
```


# 3. 进阶操作

## 3.1 传递变量

一个流程实例在遍历流程过程中可以使用Map传递变量

```java
//流程实例拥有的变量map
Map<String,Object> variables = new HashMap<>();
//设定任务的负责人
variables.put("person",new Person());
// 变量map.get("x")在经过流程节点时, 会赋值给变量值为${x}的变量
variables.put("assignee0","李四");
variables.put("assignee1","王经理");
variables.put("assignee2","杨总经理");
variables.put("assignee3","张财务");
//启动流程
runtimeService.startProcessInstanceByKey("myProcessKey",variables);
```

## 3.2 监听器
