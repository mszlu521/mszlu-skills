## AgentScope 集成

AgentScope 是一个开源的多智能体框架，Spring AI Alibaba 提供了与 AgentScope 的集成支持。

### 1. 添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-starter-agentscope</artifactId>
    <version>1.1.2.2</version>
</dependency>
```

### 2. AgentScopeAgent

AgentScopeAgent 是 Spring AI Alibaba 对 AgentScope ReActAgent 的封装。

```java
@Configuration
public class AgentScopeConfig {
    
    @Bean
    public AgentScopeAgent agentScopeAgent(ChatModel chatModel) {
        return AgentScopeAgent.builder()
            .name("AgentScopeAssistant")
            .model(chatModel)
            .instruction("你是一个专业的 AI 助手，可以帮助用户解决各种问题。")
            .tools(List.of(searchTool, calculatorTool))
            .memorySaver(memorySaver())
            .build();
    }
    
    @Bean
    public MemorySaver memorySaver() {
        return new MemorySaver();
    }
}
```

### 3. AgentScope 路由

AgentScopeRoutingAgent 支持基于 LLM 的智能路由。

```java
@Bean
public AgentScopeRoutingAgent routingAgent(
        ChatModel chatModel,
        Map<String, AgentScopeAgent> agents) {
    
    return AgentScopeRoutingAgent.builder()
        .name("SmartRouter")
        .model(chatModel)
        .agents(agents)
        .routingInstruction("""
            根据用户查询内容，选择最合适的 Agent 来处理。
            可选的 Agent 有：
            - technical_agent: 处理技术问题
            - business_agent: 处理业务问题  
            - support_agent: 处理客户支持
            """)
        .build();
}
```

### 4. 与 StateGraph 集成

```java
@Service
public class AgentScopeWorkflow {
    
    public StateGraph<WorkflowState> buildGraph(
            AgentScopeAgent agent1,
            AgentScopeAgent agent2) {
        
        return new StateGraph<>(WorkflowState.class)
            .addNode("agent1", AgentScopeNode.builder()
                .agent(agent1)
                .inputKey("input")
                .outputKey("agent1_result")
                .build())
            .addNode("agent2", AgentScopeNode.builder()
                .agent(agent2)
                .inputKey("agent1_result")
                .outputKey("final_result")
                .build())
            .addEdge(START, "agent1")
            .addEdge("agent1", "agent2")
            .addEdge("agent2", END);
    }
}
```

### 5. AgentScope 会话管理

```java
@Service
public class AgentScopeSessionService {
    
    @Autowired
    private AgentScopeAgent agent;
    
    public String chatWithSession(String sessionId, String message) {
        SessionKey sessionKey = new SimpleSessionKey(sessionId);
        
        return agent.call(message, RunnableConfig.builder()
            .threadId(sessionId)
            .build());
    }
    
    public void clearSession(String sessionId) {
        agent.clearSession(new SimpleSessionKey(sessionId));
    }
}
```

### 6. 流式响应

```java
@Service
public class AgentScopeStreamingService {
    
    @Autowired
    private AgentScopeAgent agent;
    
    public Flux<String> streamChat(String message) {
        return agent.stream(message)
            .map(output -> output.getOutput().getText());
    }
}
```

### 7. 工具使用

AgentScope 支持标准的 Spring AI ToolCallback：

```java
@Component
public class AgentScopeTools {
    
    @Bean
    public List<ToolCallback> tools() {
        return List.of(
            FunctionToolCallback.builder("search", this::search)
                .description("Search for information")
                .inputType(SearchRequest.class)
                .build(),
            FunctionToolCallback.builder("calculate", this::calculate)
                .description("Perform calculations")
                .inputType(CalculateRequest.class)
                .build()
        );
    }
    
    private String search(SearchRequest request) {
        // 搜索实现
        return "Search results for: " + request.query();
    }
    
    private String calculate(CalculateRequest request) {
        // 计算实现
        return "Result: " + (request.a() + request.b());
    }
    
    public record SearchRequest(String query) {}
    public record CalculateRequest(int a, int b) {}
}
```

### 8. 与原生 AgentScope 对比

| 特性 | 原生 AgentScope | Spring AI Alibaba 集成 |
|------|----------------|----------------------|
| 模型支持 | 内置模型 | Spring AI 所有模型 |
| 配置方式 | 代码/配置文件 | Spring Boot 自动配置 |
| 工具集成 | AgentScope Tools | Spring AI Tools |
| 持久化 | 内置存储 | 多种存储选项 |
| 工作流 | 基础流程 | StateGraph 完整支持 |

### 9. 使用场景

**适合使用 AgentScope 集成的场景：**
- 已有 AgentScope 项目需要迁移到 Spring Boot
- 需要使用 Spring AI Alibaba 的持久化和工作流能力
- 希望统一使用 Spring AI 的工具生态

**示例：客服系统**

```java
@Configuration
public class CustomerServiceConfig {
    
    @Bean
    public AgentScopeRoutingAgent customerServiceRouter(
            ChatModel chatModel,
            AgentScopeAgent technicalAgent,
            AgentScopeAgent billingAgent,
            AgentScopeAgent generalAgent) {
        
        Map<String, AgentScopeAgent> agents = Map.of(
            "technical", technicalAgent,
            "billing", billingAgent,
            "general", generalAgent
        );
        
        return AgentScopeRoutingAgent.builder()
            .name("CustomerServiceRouter")
            .model(chatModel)
            .agents(agents)
            .routingInstruction("""
                根据用户问题类型路由：
                - 技术问题 -> technical
                - 账单问题 -> billing
                - 其他问题 -> general
                """)
            .build();
    }
    
    @Bean
    public AgentScopeAgent technicalAgent(ChatModel chatModel) {
        return AgentScopeAgent.builder()
            .name("TechnicalAgent")
            .model(chatModel)
            .instruction("你是技术支持专家，解决用户的技术问题。")
            .tools(List.of(logAnalysisTool, systemCheckTool))
            .build();
    }
    
    @Bean
    public AgentScopeAgent billingAgent(ChatModel chatModel) {
        return AgentScopeAgent.builder()
            .name("BillingAgent")
            .model(chatModel)
            .instruction("你是账单支持专家，帮助用户解决账单相关问题。")
            .tools(List.of(queryBillTool, refundTool))
            .build();
    }
}
```
