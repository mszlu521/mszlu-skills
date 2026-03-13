## 非 Starter 方式使用指南

Spring AI Alibaba 可以不使用 Spring Boot Starter，直接引用核心库进行开发。

### 适用场景

| 场景 | 说明 |
|------|------|
| **Spring Boot 项目（手动配置）** | 使用 Spring Boot，但选择手动配置 Bean，而不是自动配置 |
| **Spring 项目（非 Boot）** | 传统 Spring 框架项目 |
| **纯 Java 项目** | 不使用 Spring 框架 |
| **需要细粒度控制** | 需要自定义配置逻辑，覆盖自动配置 |
| **轻量级应用** | CLI 工具、批处理程序等 |

### Starter vs 非 Starter 对比

| 特性 | Starter 方式 | 非 Starter 方式 |
|------|-------------|----------------|
| 配置方式 | `application.yml` + 自动配置 | Java 代码手动配置 |
| ChatModel | 自动注入 | 手动创建 Bean |
| ChatClient.Builder | 自动注入 | 手动创建或直接使用 ChatModel |
| 灵活性 | 配置驱动 | 代码驱动，完全可控 |
| 学习曲线 | 低 | 中高 |
| 适用场景 | 快速开发、标准场景 | 复杂定制、非标准场景 |

---

## 1. Spring Boot 项目使用非 Starter 方式

### 1.1 为什么选择非 Starter？

在 Spring Boot 项目中，你可能选择非 Starter 方式的原因：
- 需要动态切换模型配置
- 需要多模型实例管理
- 需要自定义连接池或重试逻辑
- 不想依赖自动配置，追求代码显式化
- 需要与遗留系统集成

### 1.2 依赖配置

```xml
<dependencies>
    <!-- Spring Boot 基础 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    
    <!-- 使用非 Starter 版本的核心库 -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-agent-framework</artifactId>
        <version>1.1.2.2</version>
    </dependency>
    
    <!-- DashScope 核心库（非 Starter） -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-dashscope</artifactId>
        <version>1.1.2.2</version>
    </dependency>
</dependencies>
```

### 1.3 手动配置类

```java
@Configuration
public class ManualAiConfig {
    
    @Value("${ai.dashscope.api-key}")
    private String apiKey;
    
    @Value("${ai.dashscope.model:qwen-max}")
    private String model;
    
    /**
     * 手动创建 ChatModel
     */
    @Bean
    public ChatModel chatModel() {
        DashScopeApi dashScopeApi = DashScopeApi.builder()
            .apiKey(apiKey)
            .build();
        
        return DashScopeChatModel.builder()
            .dashScopeApi(dashScopeApi)
            .defaultOptions(DashScopeChatOptions.builder()
                .model(model)
                .temperature(0.7)
                .maxTokens(2000)
                .build())
            .build();
    }
    
    /**
     * 手动创建 ChatClient.Builder
     */
    @Bean
    public ChatClient.Builder chatClientBuilder(ChatModel chatModel) {
        return ChatClient.builder(chatModel);
    }
    
    /**
     * 手动创建 MemorySaver
     */
    @Bean
    public MemorySaver memorySaver() {
        return new MemorySaver();
    }
    
    /**
     * 手动创建 ReactAgent
     */
    @Bean
    public ReactAgent reactAgent(ChatModel chatModel, MemorySaver memorySaver) {
        return ReactAgent.builder()
            .name("ManualAgent")
            .model(chatModel)
            .instruction("你是一个手动配置的 AI 助手")
            .saver(memorySaver)
            .enableLogging(true)
            .build();
    }
}
```

### 1.4 多模型配置示例

```java
@Configuration
public class MultiModelConfig {
    
    @Value("${ai.dashscope.api-key}")
    private String dashscopeApiKey;
    
    @Value("${ai.openai.api-key}")
    private String openaiApiKey;
    
    /**
     * 主模型 - DashScope qwen-max
     */
    @Bean("primaryChatModel")
    @Primary
    public ChatModel primaryChatModel() {
        return createDashScopeModel("qwen-max", 0.7);
    }
    
    /**
     * 快速模型 - DashScope qwen-turbo
     */
    @Bean("fastChatModel")
    public ChatModel fastChatModel() {
        return createDashScopeModel("qwen-turbo", 0.5);
    }
    
    /**
     * 代码模型 - DashScope qwen-coder-plus
     */
    @Bean("codingChatModel")
    public ChatModel codingChatModel() {
        return createDashScopeModel("qwen-coder-plus", 0.3);
    }
    
    /**
     * OpenAI 备用模型
     */
    @Bean("openAiChatModel")
    public ChatModel openAiChatModel() {
        OpenAiApi openAiApi = new OpenAiApi(openaiApiKey);
        
        return OpenAiChatModel.builder()
            .openAiApi(openAiApi)
            .defaultOptions(OpenAiChatOptions.builder()
                .model("gpt-4")
                .temperature(0.7)
                .build())
            .build();
    }
    
    private ChatModel createDashScopeModel(String modelName, double temperature) {
        DashScopeApi dashScopeApi = DashScopeApi.builder()
            .apiKey(dashscopeApiKey)
            .build();
        
        return DashScopeChatModel.builder()
            .dashScopeApi(dashScopeApi)
            .defaultOptions(DashScopeChatOptions.builder()
                .model(modelName)
                .temperature(temperature)
                .build())
            .build();
    }
}
```

### 1.5 使用服务类

```java
@Service
public class AiService {
    
    @Autowired
    private ReactAgent agent;
    
    @Autowired
    @Qualifier("fastChatModel")
    private ChatModel fastModel;
    
    @Autowired
    @Qualifier("codingChatModel")
    private ChatModel codingModel;
    
    /**
     * 使用 Agent 进行对话
     */
    public String chatWithAgent(String message) {
        return agent.call(message);
    }
    
    /**
     * 使用快速模型进行摘要
     */
    public String quickSummary(String text) {
        return fastModel.call("请简要总结以下内容：" + text);
    }
    
    /**
     * 使用代码模型生成代码
     */
    public String generateCode(String requirement) {
        return codingModel.call("请生成代码：" + requirement);
    }
}
```

### 1.6 动态模型切换

```java
@Service
public class DynamicModelService {
    
    private final Map<String, ChatModel> models = new HashMap<>();
    
    @PostConstruct
    public void init() {
        // 初始化多个模型
        models.put("default", createModel("qwen-max", 0.7));
        models.put("creative", createModel("qwen-max", 0.9));
        models.put("precise", createModel("qwen-plus", 0.2));
    }
    
    public String chat(String modelType, String message) {
        ChatModel model = models.getOrDefault(modelType, models.get("default"));
        return model.call(message);
    }
    
    private ChatModel createModel(String modelName, double temperature) {
        // 创建模型实例
        DashScopeApi api = DashScopeApi.builder()
            .apiKey(System.getenv("AI_DASHSCOPE_API_KEY"))
            .build();
        
        return DashScopeChatModel.builder()
            .dashScopeApi(api)
            .defaultOptions(DashScopeChatOptions.builder()
                .model(modelName)
                .temperature(temperature)
                .build())
            .build();
    }
}
```

### 1.7 禁用自动配置（可选）

如果你同时引入了 Starter 依赖，但想使用手动配置，可以排除自动配置：

```java
@SpringBootApplication(exclude = {
    DashScopeAutoConfiguration.class
})
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

或在 `application.yml`：

```yaml
spring:
  ai:
    dashscope:
      enabled: false
```

---

## 2. 核心依赖（非 Starter）

### 2.1 基础依赖配置

```xml
<properties>
    <spring-ai-alibaba.version>1.1.2.2</spring-ai-alibaba.version>
    <spring-ai.version>1.1.2</spring-ai.version>
</properties>

<dependencies>
    <!-- ==================== 核心模块（必选） ==================== -->
    
    <!-- Graph Core - 工作流引擎 -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-graph-core</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- Agent Framework - Agent 开发框架 -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-agent-framework</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- ==================== 模型提供方（选择其一或多个） ==================== -->
    
    <!-- DashScope（阿里云）- 非 Starter 版本 -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-dashscope</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- OpenAI -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai</artifactId>
        <version>${spring-ai.version}</version>
    </dependency>
    
    <!-- Ollama（本地模型） -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-ollama</artifactId>
        <version>${spring-ai.version}</version>
    </dependency>
    
    <!-- ==================== 持久化（可选） ==================== -->
    
    <!-- Redis -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-graph-redis</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- MySQL -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-graph-mysql</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- PostgreSQL -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-graph-postgres</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- MongoDB -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-graph-mongo</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- ==================== 其他可选模块 ==================== -->
    
    <!-- 内置 Nodes -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-builtin-nodes</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
    
    <!-- AgentScope 集成 -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-agentscope</artifactId>
        <version>${spring-ai-alibaba.version}</version>
    </dependency>
</dependencies>
```

### 2.2 Gradle 配置

```groovy
ext {
    springAiAlibabaVersion = '1.1.2.2'
    springAiVersion = '1.1.2'
}

dependencies {
    // 核心模块
    implementation "com.alibaba.cloud.ai:spring-ai-alibaba-graph-core:${springAiAlibabaVersion}"
    implementation "com.alibaba.cloud.ai:spring-ai-alibaba-agent-framework:${springAiAlibabaVersion}"
    
    // 模型提供方
    implementation "com.alibaba.cloud.ai:spring-ai-alibaba-dashscope:${springAiAlibabaVersion}"
    
    // 持久化
    implementation "com.alibaba.cloud.ai:spring-ai-alibaba-graph-redis:${springAiAlibabaVersion}"
}
```

---

## 3. 手动配置 ChatModel

### 3.1 DashScope

```java
public class ChatModelConfig {
    
    public static ChatModel createDashScopeChatModel() {
        // 创建 API 实例
        DashScopeApi dashScopeApi = DashScopeApi.builder()
            .apiKey(System.getenv("AI_DASHSCOPE_API_KEY"))
            .baseUrl("https://dashscope.aliyuncs.com")  // 可选，默认即为官方地址
            .build();
        
        // 创建 ChatModel
        return DashScopeChatModel.builder()
            .dashScopeApi(dashScopeApi)
            .defaultOptions(DashScopeChatOptions.builder()
                .model("qwen-max")
                .temperature(0.7)
                .maxTokens(2000)
                .build())
            .build();
    }
    
    // 带重试和超时的配置
    public static ChatModel createDashScopeChatModelWithRetry() {
        DashScopeApi dashScopeApi = DashScopeApi.builder()
            .apiKey(System.getenv("AI_DASHSCOPE_API_KEY"))
            .build();
        
        return DashScopeChatModel.builder()
            .dashScopeApi(dashScopeApi)
            .defaultOptions(DashScopeChatOptions.builder()
                .model("qwen-max")
                .temperature(0.7)
                .build())
            .retryTemplate(RetryTemplate.builder()
                .maxAttempts(3)
                .retryOn(IOException.class)
                .build())
            .build();
    }
}
```

### 3.2 OpenAI

```java
public class OpenAiConfig {
    
    public static ChatModel createOpenAiChatModel() {
        OpenAiApi openAiApi = new OpenAiApi(
            System.getenv("OPENAI_API_KEY"),
            "https://api.openai.com"  // 可选，或使用代理地址
        );
        
        return OpenAiChatModel.builder()
            .openAiApi(openAiApi)
            .defaultOptions(OpenAiChatOptions.builder()
                .model("gpt-4")
                .temperature(0.7)
                .build())
            .build();
    }
}
```

### 3.3 Ollama（本地模型）

```java
public class OllamaConfig {
    
    public static ChatModel createOllamaChatModel() {
        OllamaApi ollamaApi = new OllamaApi("http://localhost:11434");
        
        return OllamaChatModel.builder()
            .ollamaApi(ollamaApi)
            .defaultOptions(OllamaOptions.builder()
                .model("llama2")
                .temperature(0.7)
                .build())
            .build();
    }
}
```

---

## 4. 手动创建和使用 Agent

### 4.1 基础 ReactAgent

```java
public class SimpleAgentExample {
    
    public static void main(String[] args) {
        // 1. 创建 ChatModel
        ChatModel chatModel = ChatModelConfig.createDashScopeChatModel();
        
        // 2. 创建 MemorySaver
        MemorySaver memorySaver = new MemorySaver();
        
        // 3. 创建 Agent
        ReactAgent agent = ReactAgent.builder()
            .name("SimpleAgent")
            .model(chatModel)
            .instruction("你是一个有帮助的助手")
            .saver(memorySaver)
            .enableLogging(true)
            .build();
        
        // 4. 使用 Agent
        String response = agent.call("你好，请介绍一下自己");
        System.out.println(response);
    }
}
```

---

## 5. 纯 Java 项目（无 Spring）

### 5.1 Maven 依赖

```xml
<dependencies>
    <!-- 只需要核心模块 -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-graph-core</artifactId>
        <version>1.1.2.2</version>
    </dependency>
    
    <!-- 模型提供方 -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-dashscope</artifactId>
        <version>1.1.2.2</version>
    </dependency>
    
    <!-- 日志（可选） -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>2.0.9</version>
    </dependency>
</dependencies>
```

### 5.2 完整示例

```java
public class PureJavaExample {
    
    public static void main(String[] args) {
        // 1. 配置日志
        System.setProperty("org.slf4j.simpleLogger.defaultLogLevel", "debug");
        
        // 2. 创建 ChatModel
        ChatModel chatModel = createChatModel();
        
        // 3. 创建 Agent
        ReactAgent agent = ReactAgent.builder()
            .name("PureJavaAgent")
            .model(chatModel)
            .instruction("你是一个纯 Java 项目中的 AI 助手")
            .saver(new MemorySaver())
            .enableLogging(true)
            .build();
        
        // 4. 使用
        Scanner scanner = new Scanner(System.in);
        System.out.println("AI: 你好！我是 AI 助手。输入 'exit' 退出。");
        
        while (true) {
            System.out.print("You: ");
            String input = scanner.nextLine();
            
            if ("exit".equalsIgnoreCase(input)) {
                break;
            }
            
            String response = agent.call(input);
            System.out.println("AI: " + response);
        }
        
        scanner.close();
    }
    
    private static ChatModel createChatModel() {
        DashScopeApi dashScopeApi = DashScopeApi.builder()
            .apiKey(System.getenv("AI_DASHSCOPE_API_KEY"))
            .build();
        
        return DashScopeChatModel.builder()
            .dashScopeApi(dashScopeApi)
            .defaultOptions(DashScopeChatOptions.builder()
                .model("qwen-turbo")
                .build())
            .build();
    }
}
```

---

## 6. 常见问题

### Q: Spring Boot 项目为什么要使用非 Starter 方式？

**答**: 虽然 Starter 方式更方便，但非 Starter 方式提供：
1. **完全控制**: 手动管理所有 Bean 的创建和生命周期
2. **动态配置**: 运行时动态切换模型参数
3. **多实例**: 轻松管理多个模型实例
4. **自定义逻辑**: 在 Bean 创建时注入自定义逻辑

### Q: 不使用 Starter 会缺少哪些功能？

**答**: 不会缺少核心功能，但需要手动配置：
- 自动配置的 `ChatClient.Builder` → 手动创建 `ChatClient`
- 自动配置的 `ChatModel` → 手动创建 `ChatModel`
- `application.yml` 配置 → 代码配置（或使用 `@Value` 注入）

### Q: 如何迁移从 Starter 到非 Starter？

**答**: 
1. 将 `spring-ai-alibaba-starter-xxx` 替换为 `spring-ai-alibaba-xxx`
2. 将 `application.yml` 中的配置迁移到 Java 配置类
3. 手动创建 Bean 替代自动配置

### Q: 混合使用 Starter 和手动配置可以吗？

**答**: 可以。可以保留 Starter 的基础配置，同时通过 `@Primary` 或 `@Qualifier` 覆盖特定 Bean。

```java
@Bean
@Primary  // 覆盖 Starter 自动配置的 ChatModel
public ChatModel customChatModel() {
    // 自定义配置
}
```
