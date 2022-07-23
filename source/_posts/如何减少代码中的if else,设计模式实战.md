---
title: 如何减少代码中的if else,设计模式实战
id: '32'
tags: ['设计模式','Java','面向对象']
categories:
  - - develop
date: 2022-07-31 15:59:16
url: /design
---
## 前言

设计模式可能我们都听过，但是在实际开发过程中可能种种原因（时间紧，没有合适场景）很少使用。上周对接流程接口时，接口调用方的回调接口很有意思，也正好使用设计模式的设计理念来对整个内容设计开发下。

需求内容大概是这样的，我们系统有很多审批流程推送到客户系统，但是客户只要求我们提供唯一的回调接口。接口返回内容也只有请求id和审批通过/审批不通过。

## 设计

通常情况下，这个需求实现起来很容易，大概是这样的：
1. 先判断流程是审批通过/审批退回
2. 根据请求id查找出对应的流程
3. 对应流程执行相应的操作。

## 编码

### 快速编码

我们很容易根据设计写出以下的代码：
```java
    @RequestMapping("callbackoa")
    public String callBackOa(@RequestBody String param){
        JSONObject paramJson = JSONObject.parseObject(param);
        String requestId = paramJson.getString("requestId");
        String approvalStatus = paramJson.getString("approvalStatus");
        if ("1".equals(approvalStatus)){
            // search workflowId
            String workflowId = getWorkflowId(requestId);
            if ("111".equals(workflowId)){
                //do Corresponding Process
            }
            if ("222".equals(workflowId)){
                //do Corresponding Process
            }
            if ("333".equals(workflowId)){
                //do Corresponding Process
            }
        }
        else {
            String workflowId = getWorkflowId(requestId);
            if ("111".equals(workflowId)){
                //do Corresponding Process
            }
            if ("222".equals(workflowId)){
                //do Corresponding Process
            }
            if ("333".equals(workflowId)){
                //do Corresponding Process
            }

        }

        return "";
    }
```
在我们需求快速开发时期，这样写也没什么问题，也能很快的完成需求交付。但是很显然，这并不是一种**好的编码形式**

他可能有以下问题
1. 不符合开闭原则，如果想要修改代码，就要重新修改这个类，不利于拓展
2. 不符合面向对象编程，看上去更像是面向过程（调用函数来实现）
3. 每次打开这个类一行行看下来，找对应的代码逻辑非常困难，代码耦合太多，不符合单一职责原则。

### 改造第一阶段

既然上面的代码有这么多的问题，那我们开始优化一下。
1. 我们考虑到每次执行对应的流程都要先根据`resquestid`查询一下对应的`workflowid`，可以考虑将通用的方法抽象出来。
2. 可以考虑将每一种流程都抽取到单独的一个类里面。
3. 使用依赖注入来注入所有实现，统一使用抽象类来调用（抽象类本身不要添加@component注解）

我们使用**适配器模式**对代码优化
> 适配器模式：将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不     兼容而不能一起工作的那些类能一起工作。  
  适配器模式包含以下主要角色：  
    - 目标（Target）接口：当前系统业务所期待的接口，它可以是抽象类或接口。  
    - 适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口。  
    - 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。  
    
下面是改造后的代码。
    
```java
    # 回调接口
    @Autowired
    private ReceiveOAProcessService oaProcessService;

    @RequestMapping("callbackoa")
    public String callBackOa(@RequestBody String param){
        JSONObject paramJson = JSONObject.parseObject(param);
        String requestId = paramJson.getString("requestId");
        String approvalStatus = paramJson.getString("approvalStatus");
        if ("1".equals(approvalStatus)) {
            oaProcessService.revokeSuccCallback(jsonObject.getString("requestId"));
        }
        else if ("0".equals(approvalStatus)) {
            oaProcessService.revokeFailCallback(jsonObject.getString("requestId"));
        }
        return AjaxResult.error("流程传入标识为空！").toString();
    }
    
    # 抽象类
    public abstract class abstractWorkflowService{
     public WorkflowRequestInfo executeCallback(String requestId) {
        // search workflowid
     }
    } 

    public abstract void revokeSuccCallback(String requestId);

    public abstract void revokeFailCallback(String requestId);
    
    ## 具体实现
    public class workflowServiceImpl extends abstractWorkflowService {
     @Override
     public void revokeSuccCallback(String requestId) {
        WorkflowRequestInfo workflowRequestInfo = executeCallback(requestId);
        if (workflowId.equals(workflowRequestInfo.getWorkflowBaseInfo().getWorkflowId())){
           //do Corresponding Process
        }
     }
    }
```
### 改造第二阶段

看上去我们的代码已经优化的差不多了，如果要增加流程只需要增加类继承原有的抽象类即可，修改对应流程也只需要修改对应的类。但事实上我们只是将if判断语句换了地方，写到了具体的类里面。这显然不符合我们的预期。

我们使用**策略模式** 对代码进行优化

> 策略模式定义了一系列的算法，并将每一个算法封装起来，使它们可以相互替换。策略模式通常包含以下角色：  
> - 抽象策略（Strategy）类：定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，环境角色使用这个接口调用不同的算法，一般使用接口或抽象类实现。    
> - 具体策略（Concrete Strategy）类：实现了抽象策略定义的接口，提供具体的算法实现。  
> - 环境（Context）类：持有一个策略类的引用，最终给客户端调用。   

根据策略模式，我们抽象出一个接口，并提供对外的访问的通用环境类。

```java

# 接口
public interface OaProcessStrategy
{
      void revokeSuccCallback(String requestId);
}

 # 实现
  public class firstworkflowServiceImpl extends abstractWorkflowService {
     @Override
     public void revokeSuccCallback(String requestId) {
        WorkflowRequestInfo workflowRequestInfo = executeCallback(requestId);
        //do Corresponding Process
        
     }
  }
  
  # 对外的环境类
  public class workflowStrategyContext{
      
      public void receiveWorkflowInfo(String workflowid){
          if("111".equals(workflowid)){
              new  firstworkflowServiceImpl();
          }
          if("222".equals(workflowid)){
              new  secorndworkflowServiceImpl();
          }
      }
      
      public void getcallMethod(){
          OaProcessStrategy oaprocessStrategy =  receiveWorkflowInfo(String workflowid);
          oaprocessStrategy.revokeSuccCallback(String requestId);
      }
      
      
  }
  
```
### 改造第三阶段

由以上的改造我们发现对应的实现类中没有成员变量，也就是实现类是无状态的，多次调用不影响类的实例没有影响。并且每次增加流程还需要修改环境类，不符合开闭原则。

因此我们使用**单例模式**进行对应的修改

> 单例模式[1-5]设计模式属于创建型模式，它提供了一种创建对象的最佳方式。  
> - 这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

```java


# 环境类修改
@Service
public class OaProcessStrategyContext
{
    private final static Map<String,OaProcessStrategy> oaProcessStrategyMap = new HashMap<>();


    private String OAWebserviceUrl = "https://testecology.mengtaigroup.com/services/WorkflowService?wsdl";

    @Autowired
    private ExtractDataSource dataSource;

    /**
     * 注册策略转换类
     * @param workflowId
     * @param oaProcessStrategy
     */
    public static void registerStrategy(String workflowId,OaProcessStrategy oaProcessStrategy){
        oaProcessStrategyMap.putIfAbsent(workflowId,oaProcessStrategy);
    }

    /**
     * 获取对应的策略类
     * @param workflowId
     * @return
     */
    public static OaProcessStrategy getStrategy(String workflowId){
       return oaProcessStrategyMap.get(workflowId);
    }
}
# 抽象类

public abstract class ReceiveOAProcessService implements OaProcessStrategy
{

    public void register(String workflowId){
        OaProcessStrategyContext.registerStrategy(workflowId,this);
    }

}

# 实现类

@Service
public class DelegationReceiveOAProcessServiceImpl extends ReceiveOAProcessService implements OaProcessStrategy
{

    private DelegationReceiveOAProcessServiceImpl() {
        register(workflowId);
    }

    @Override
    public void revokeSuccCallback(String requestId) {
    }

    
}

```

由此，我们实现了每个流程都能自己注册到环境类中，并且增加新流程完全不用修改原来的代码。同时代码之间的耦合性也大大降低。