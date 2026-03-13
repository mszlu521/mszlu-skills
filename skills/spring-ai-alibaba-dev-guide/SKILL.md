---
name: spring-ai-alibaba-dev-guide
description: |
  Spring AI Alibaba 开发指导 Skill。当用户需要开发基于 Spring AI Alibaba 的 AI 应用时，
  必须调用本 Skill 提供全面的开发指导。涵盖 Agents、Models、Messages、Tools、Memory、
  Hooks、Interceptors、Skills、Structured Output、上下文工程、人工介入、记忆管理、
  多智能体编排、智能体作为工具、工作流、RAG、A2A、Graph、Chat UI、Admin、DeepResearch、
  Data Agent、Assistant Agent、Tool Calling 等所有核心功能。

  触发场景：
  - 用户询问如何使用 Spring AI Alibaba 开发 AI 应用
  - 用户需要创建 Agent、ChatBot、工作流、RAG 系统
  - 用户询问多智能体编排、工具调用、MCP 集成
  - 用户需要配置模型、记忆管理、上下文工程
  - 用户询问 Hooks、Interceptors、Skills 机制
  - 用户需要 A2A、Graph 工作流、DeepResearch、Data Agent
  - 用户遇到 Spring AI Alibaba 相关的任何开发问题
  - 用户需要最佳实践、示例代码、项目结构指导
---

# Spring AI Alibaba 开发指导

## 版本信息

**当前版本**: `1.1.2.2`

所有示例代码均使用最新版本，Maven 依赖请使用：

```xml
<properties>
    <spring-ai-alibaba.version>1.1.2.2</spring-ai-alibaba.version>
</properties>
```

---

## Spring AI 基础

Spring AI Alibaba 基于 **Spring AI 1.1.2** 构建，理解 Spring AI 的核心概念对于开发 Spring AI Alibaba 应用至关重要。

### Spring AI 与 Spring AI Alibaba 的关系

```
┌─────────────────────────────────────────────────────────────┐
│                   Spring AI Alibaba                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Spring AI 1.1.2 (Base)                 │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌───────────────┐  │   │
│  │  │  ChatModel  │ │ ChatClient  │ │  VectorStore  │  │   │
│  │  └─────────────┘ └─────────────┘ └───────────────┘  │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌───────────────┐  │   │
│  │  │   Prompt    │ │    Tool     │ │   Advisor     │  │   │
│  │  └─────────────┘ └─────────────┘ └───────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         Spring AI Alibaba Extensions                │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌───────────────┐  │   │
│  │  │ReactAgent   │ │ StateGraph  │ │   A2A         │  │   │
│  │  └─────────────┘ └─────────────┘ └───────────────┘  │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌───────────────┐  │   │
│  │  │Multi-Agent  │ │DashScope    │ │   MCP         │  │   │
│  │  └─────────────┘ └─────────────┘ └───────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Spring AI 核心架构

Spring AI 提供了一套统一的抽象层，用于与各种 AI 模型和服务交互：

| 组件 | 说明 | 主要接口/类 |
|------|------|------------|
| **ChatModel** | 聊天模型核心接口 | `ChatModel`, `StreamingChatModel` |
| **ChatClient** | Fluent API 客户端 | `ChatClient`, `ChatClient.Builder` |
| **Prompt** | 提示词封装 | `Prompt`, `PromptTemplate` |
| **Messages** | 消息类型体系 | `UserMessage`, `SystemMessage`, `AssistantMessage`, `ToolResponseMessage` |
| **ChatOptions** | 模型调用选项 | `ChatOptions`, `FunctionOptions` |
| **Tools** | 函数/工具调用 | `ToolCallback`, `ToolDefinition` |
| **Advisor** | 拦截器/切面 | `Advisor`, `CallAdvisor`, `StreamAdvisor` |
| **VectorStore** | 向量存储 | `VectorStore`, `Document` |
| **Embedding** | 嵌入模型 | `EmbeddingModel`, `EmbeddingClient` |

### ChatModel API

`ChatModel` 是 Spring AI 的核心接口，定义了与 AI 模型交互的基础方法：

```java
public interface ChatModel extends Model<Prompt, ChatResponse>, StreamingChatModel {
    // 简单文本调用
    default String call(String message);

    // 多消息调用
    default String call(Message... messages);

    // Prompt 调用（核心方法）
    ChatResponse call(Prompt prompt);

    // 流式调用
    default Flux<ChatResponse> stream(Prompt prompt);

    // 获取默认选项
    default ChatOptions getDefaultOptions();
}
```

**使用示例：**

```java
@Service
public class ChatService {
    @Autowired
    private ChatModel chatModel;

    public String chat(String message) {
        // 简单调用
        return chatModel.call(message);
    }

    public String chatWithHistory(List<Message> messages) {
        // 带历史的对话
        Prompt prompt = new Prompt(messages);
        ChatResponse response = chatModel.call(prompt);
        return response.getResult().getOutput().getText();
    }

    public Flux<String> streamChat(String message) {
        // 流式输出
        return chatModel.stream(new Prompt(message))
            .map(r -> r.getResult().getOutput().getText());
    }
}
```

### ChatClient Fluent API

`ChatClient` 提供了更便捷的流式 API 用于构建对话：

```java
// 创建 ChatClient
ChatClient chatClient = ChatClient.builder(chatModel).build();

// 简单调用
String response = chatClient.prompt("你好").call().content();

// 带系统提示的调用
String response = chatClient.prompt()
    .system("你是一个专业的助手")
    .user("你好")
    .call()
    .content();

// 流式调用
Flux<String> stream = chatClient.prompt("你好").stream().content();

// 使用工具
String response = chatClient.prompt()
    .user("上海天气如何？")
    .tools(weatherTool)
    .call()
    .content();

// 结构化输出
ProductInfo product = chatClient.prompt()
    .user("分析这个产品的信息")
    .call()
    .entity(ProductInfo.class);
```

**ChatClient 完整配置：**

```java
@Configuration
public class ChatClientConfig {

    @Bean
    public ChatClient chatClient(ChatModel chatModel) {
        return ChatClient.builder(chatModel)
            // 默认系统提示
            .defaultSystem("你是一个专业的AI助手")
            // 默认选项
            .defaultOptions(
                ChatOptions.builder()
                    .temperature(0.7)
                    .maxTokens(2000)
                    .build()
            )
            // 默认工具
            .defaultTools(weatherTool, calculatorTool)
            // 默认 Advisors
            .defaultAdvisors(
                new SimpleLoggerAdvisor(),
                new MessageChatMemoryAdvisor(chatMemory)
            )
            .build();
    }
}
```

### Prompt 和 Messages

Spring AI 提供了完整的消息类型体系：

```java
// ===== 消息类型 =====

// 系统消息 - 设置AI角色和行为
SystemMessage systemMessage = new SystemMessage("你是专业助手");
SystemMessage systemMessage = SystemMessage.builder()
    .text("你是专业助手")
    .metadata(Map.of("version", "1.0"))
    .build();

// 用户消息 - 用户输入
UserMessage userMessage = new UserMessage("你好");
UserMessage userMessage = UserMessage.builder()
    .text("分析这张图片")
    .media(new Media(MimeTypeUtils.IMAGE_PNG, imageResource))
    .build();

// 助手消息 - AI回复
AssistantMessage assistantMessage = new AssistantMessage("你好！有什么可以帮助你？");

// 工具响应消息
ToolResponseMessage toolMessage = ToolResponseMessage.builder()
    .responses(List.of(
        new ToolResponseMessage.ToolResponse("call_123", "工具执行结果")
    ))
    .build();

// ===== 构建 Prompt =====

// 简单 Prompt
Prompt prompt = new Prompt("你好");

// 带选项的 Prompt
Prompt prompt = new Prompt(
    "你好",
    ChatOptions.builder()
        .temperature(0.7)
        .maxTokens(1000)
        .build()
);

// 多消息 Prompt
Prompt prompt = new Prompt(List.of(
    new SystemMessage("你是专业助手"),
    new UserMessage("你好")
));

// 使用 Builder
Prompt prompt = Prompt.builder()
    .messages(
        SystemMessage.builder().text("你是专业助手").build(),
        UserMessage.builder()
            .text("分析这张图片")
            .media(new Media(MimeTypeUtils.IMAGE_PNG, imageResource))
            .build()
    )
    .chatOptions(ChatOptions.builder().temperature(0.5).build())
    .build();
```

### ChatOptions 配置

```java
// 通用选项
ChatOptions options = ChatOptions.builder()
    .model("qwen-max")              // 模型名称
    .temperature(0.7)               // 随机性 (0.0-2.0)
    .maxTokens(2000)                // 最大生成token数
    .topP(0.9)                      // 核采样
    .topK(50)                       // Top-K采样
    .frequencyPenalty(0.5)          // 频率惩罚
    .presencePenalty(0.5)           // 存在惩罚
    .stopSequences(List.of("STOP")) // 停止序列
    .build();

// DashScope 特定选项
DashScopeChatOptions dashScopeOptions = DashScopeChatOptions.builder()
    .model("qwen-max")
    .temperature(0.7)
    .responseFormat(ResponseFormat.JSON)  // JSON 模式
    .multiModel(true)                      // 多模态
    .build();

// OpenAI 特定选项
OpenAiChatOptions openAiOptions = OpenAiChatOptions.builder()
    .model("gpt-4")
    .temperature(0.7)
    .tools(List.of("function1", "function2"))  // 指定可用工具
    .build();
```

### Advisor 机制

Advisor 是 Spring AI 的拦截器机制，用于在请求处理前后执行逻辑：

```java
// 使用内置 Advisors
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultAdvisors(
        // 记录日志
        new SimpleLoggerAdvisor(),
        // 聊天记忆 - 自动维护对话历史
        new MessageChatMemoryAdvisor(chatMemory, "conversationId", 10),
        // RAG 检索 - 自动检索相关文档
        new QuestionAnswerAdvisor(vectorStore),
        // 安全过滤
        new SafeGuardAdvisor(Set.of("敏感词")),
        // 结构化输出验证
        new StructuredOutputValidationAdvisor()
    )
    .build();

// 自定义 Advisor
@Component
public class LoggingAdvisor implements CallAdvisor {
    private static final Logger log = LoggerFactory.getLogger(LoggingAdvisor.class);

    @Override
    public String getName() {
        return "LoggingAdvisor";
    }

    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE;
    }

    @Override
    public ChatClientResponse adviseCall(AdvisedRequest request, CallAdvisorChain chain) {
        // 前置处理
        log.info("Request: {}", request.userText());
        long startTime = System.currentTimeMillis();

        // 执行链
        ChatClientResponse response = chain.next(request);

        // 后置处理
        long duration = System.currentTimeMillis() - startTime;
        log.info("Response time: {}ms", duration);

        return response;
    }
}
```

### Vector Store 和 RAG

Spring AI 提供了统一的 Vector Store 抽象：

```java
// 创建 Vector Store
@Bean
public VectorStore vectorStore(EmbeddingModel embeddingModel) {
    // 简单内存存储
    return SimpleVectorStore.builder(embeddingModel).build();

    // 或 Redis
    // return RedisVectorStore.builder(redisTemplate, embeddingModel).build();

    // 或 PostgreSQL (PGVector)
    // return PgVectorStore.builder(dataSource, embeddingModel).build();
}

// 文档索引
@Service
public class DocumentService {
    @Autowired
    private VectorStore vectorStore;

    public void indexDocuments(List<Resource> resources) {
        // 文本分割器
        TokenTextSplitter splitter = new TokenTextSplitter(
            500,  // chunk size
            50,   // overlap
            10,   // min chunk size
            1000, // max chunk size
            true  // keep separator
        );

        for (Resource resource : resources) {
            // 读取并分割文档
            String content = readResource(resource);
            List<String> chunks = splitter.split(content);

            // 创建 Document 并添加元数据
            List<Document> documents = chunks.stream()
                .map(chunk -> new Document(
                    chunk,
                    Map.of(
                        "source", resource.getFilename(),
                        "timestamp", Instant.now().toString()
                    )
                ))
                .toList();

            // 存入向量数据库
            vectorStore.add(documents);
        }
    }
}

// RAG 检索
@Bean
public VectorStoreDocumentRetriever documentRetriever(VectorStore vectorStore) {
    return VectorStoreDocumentRetriever.builder()
        .vectorStore(vectorStore)
        .similarityThreshold(0.7)
        .topK(5)
        .filterExpression("type == 'document'")
        .build();
}
```

### Tool/Function Calling

Spring AI 支持将 Java 方法暴露为 AI 可调用的工具：

```java
// 方式 1: 使用 @Tool 注解
@Component
public class WeatherTool {

    @Tool(name = "get_weather", description = "获取指定城市的天气信息")
    public WeatherInfo getWeather(
            @ToolParam(description = "城市名称，如：北京、上海") String city) {
        // 实现天气查询逻辑
        return new WeatherInfo(city, "晴天", 25);
    }
}

// 方式 2: 编程式创建 ToolCallback
@Bean
public ToolCallback searchTool() {
    return FunctionToolCallback.builder("search", (SearchRequest request) -> {
        // 执行搜索
        return searchService.search(request.query());
    })
    .description("搜索知识库")
    .inputType(SearchRequest.class)
    .build();
}

// 注册工具
@Bean
public ChatClient chatClient(ChatModel chatModel, WeatherTool weatherTool) {
    return ChatClient.builder(chatModel)
        .defaultTools(weatherTool)
        .build();
}

// 使用工具
@Service
public class ToolService {
    @Autowired
    private ChatClient chatClient;

    public String chatWithTools(String message) {
        return chatClient.prompt()
            .user(message)
            .call()
            .content();
    }
}
```

### 结构化输出

Spring AI 支持将模型输出映射到 Java 对象：

```java
// 定义输出结构
public record ProductInfo(
    String name,
    double price,
    List<String> features,
    String category
) {}

// 使用 ChatClient
@Service
public class StructuredOutputService {
    @Autowired
    private ChatClient chatClient;

    public ProductInfo extractProductInfo(String description) {
        return chatClient.prompt()
            .user(u -> u.text("""
                从以下描述中提取产品信息：
                {description}
                """
            ).param("description", description))
            .call()
            .entity(ProductInfo.class);
    }

    public List<String> extractKeywords(String text) {
        return chatClient.prompt()
            .user("从以下文本中提取关键词：" + text)
            .call()
            .entity(new ParameterizedTypeReference<List<String>>() {});
    }
}

// 使用 BeanOutputParser
@Service
public class ParserService {
    @Autowired
    private ChatModel chatModel;

    public ProductInfo parseWithBeanParser(String description) {
        BeanOutputParser<ProductInfo> parser = new BeanOutputParser<>(ProductInfo.class);

        String prompt = """
            提取产品信息。
            输出格式: {format}

            描述: {description}
            """;

        PromptTemplate template = new PromptTemplate(
            prompt,
            Map.of(
                "format", parser.getFormat(),
                "description", description
            )
        );

        ChatResponse response = chatModel.call(template.create());
        return parser.parse(response.getResult().getOutput().getText());
    }
}
```

### 多模型厂商支持

Spring AI Alibaba 对模型提供方的支持分为两类：

| 类型 | 说明 | 依赖来源 |
|------|------|----------|
| **内置支持** | DashScope（阿里云百炼） | `spring-ai-alibaba-starter-dashscope` |
| **Spring AI 原生支持** | OpenAI、DeepSeek、Ollama、Azure OpenAI 等 | Spring AI 官方 starters |

#### 模型厂商依赖配置

**1. DashScope（阿里云）- Spring AI Alibaba 内置**

```xml
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>
    <version>1.1.2.2</version>
</dependency>
```

```yaml
spring:
  ai:
    dashscope:
      api-key: ${AI_DASHSCOPE_API_KEY}
      chat:
        options:
          model: qwen-max  # 或其他模型：qwen-plus, qwen-turbo, qwen-vl-max 等
```

**2. OpenAI - Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      base-url: https://api.openai.com/v1  # 可选，用于代理或兼容接口
      chat:
        options:
          model: gpt-4o
          temperature: 0.7
```

**3. DeepSeek - Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-deepseek-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    deepseek:
      api-key: ${DEEPSEEK_API_KEY}
      base-url: https://api.deepseek.com/v1
      chat:
        options:
          model: deepseek-chat  # 或 deepseek-coder
```

**4. Ollama（本地模型）- Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434  # Ollama 服务地址
      chat:
        options:
          model: llama3.1  # 或 mistral, qwen2 等
```

**5. Azure OpenAI - Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-azure-openai-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    azure:
      openai:
        api-key: ${AZURE_OPENAI_API_KEY}
        endpoint: https://your-resource.openai.azure.com
        chat:
          options:
            deployment-name: gpt-4o
```

**6. Anthropic Claude - Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-anthropic-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    anthropic:
      api-key: ${ANTHROPIC_API_KEY}
      chat:
        options:
          model: claude-3-opus-20240229
```

**7. 智谱 AI (ZhiPu AI) - Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-zhipuai-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    zhipuai:
      api-key: ${ZHIPUAI_API_KEY}
      chat:
        options:
          model: glm-4  # 或 glm-3-turbo
```

**8. 月之暗面 (Moonshot/Kimi) - Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-moonshot-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    moonshot:
      api-key: ${MOONSHOT_API_KEY}
      chat:
        options:
          model: moonshot-v1-8k  # 或 moonshot-v1-32k, moonshot-v1-128k
```

**9. MiniMax - Spring AI 原生支持**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-minimax-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    minimax:
      api-key: ${MINIMAX_API_KEY}
      chat:
        options:
          model: abab6.5-chat
```

#### 多模型共存配置

可以在同一个项目中配置多个模型，按需注入使用：

```java
@Configuration
public class MultiModelConfig {

    // DashScope - 用于通用对话
    @Bean
    public ChatClient dashScopeChatClient(@Qualifier("dashscopeChatModel") ChatModel chatModel) {
        return ChatClient.builder(chatModel).build();
    }

    // OpenAI - 用于特定功能
    @Bean
    public ChatClient openAiChatClient(@Qualifier("openaiChatModel") ChatModel chatModel) {
        return ChatClient.builder(chatModel).build();
    }

    // Ollama - 用于本地隐私场景
    @Bean
    public ChatClient ollamaChatClient(@Qualifier("ollamaChatModel") ChatModel chatModel) {
        return ChatClient.builder(chatModel).build();
    }
}

@Service
public class MultiModelService {

    @Autowired
    @Qualifier("dashScopeChatClient")
    private ChatClient dashScopeClient;

    @Autowired
    @Qualifier("openAiChatClient")
    private ChatClient openAiClient;

    public String chatWithDashScope(String message) {
        return dashScopeClient.prompt(message).call().content();
    }

    public String chatWithOpenAI(String message) {
        return openAiClient.prompt(message).call().content();
    }
}
```

#### 模型选择建议

| 场景 | 推荐模型 | 说明 |
|------|----------|------|
| 国内部署 | DashScope (qwen-max) | 阿里云国内节点，访问稳定 |
| 中文处理 | DashScope、DeepSeek | 对中文优化更好 |
| 代码生成 | DeepSeek Coder、GPT-4o | 代码能力强 |
| 本地/离线 | Ollama + Llama3/Qwen2 | 数据不出域 |
| 长文本 | Moonshot (128k)、GPT-4o (128k) | 支持超长上下文 |
| 成本敏感 | DashScope (qwen-turbo) | 价格较低 |

---

## 快速开始

### 1. 环境要求

- **JDK**: 17+
- **构建工具**: Maven 3.6+ 或 Gradle 7+
- **AI 模型 API Key**: DashScope / OpenAI / DeepSeek 等

### 2. 基础依赖

```xml
<dependencies>
    <!-- Spring AI Alibaba Agent Framework -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-agent-framework</artifactId>
        <version>1.1.2.2</version>
    </dependency>

    <!-- 模型提供方（选择其一） -->
    <!-- DashScope（阿里云） -->
    <dependency>
        <groupId>com.alibaba.cloud.ai</groupId>
        <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>
        <version>1.1.2.2</version>
    </dependency>

    <!-- 或 OpenAI -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.1.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 3. 配置 API Key

**环境变量方式**（推荐）：
```bash
export AI_DASHSCOPE_API_KEY=your-api-key
```

**配置文件方式**：
```yaml
spring:
  ai:
    dashscope:
      api-key: ${AI_DASHSCOPE_API_KEY}
      chat:
        options:
          model: qwen-max  # 或其他模型
```

---

## 核心概念

### 1. Models（模型）

Spring AI Alibaba 支持多种 Chat Model：

```java
// 自动注入配置的 ChatModel
@Autowired
private ChatModel chatModel;

// 直接调用
String response = chatModel.call("你好");

// 使用 Prompt
ChatResponse response = chatModel.call(
    new Prompt(new UserMessage("你好"))
);

// 流式调用
Flux<ChatResponse> stream = chatModel.stream(
    new Prompt(new UserMessage("你好"))
);
```

**支持的模型提供方**：
- DashScope（阿里云）
- OpenAI
- DeepSeek
- Ollama（本地模型）
- Azure OpenAI

### 2. Messages（消息）

```java
// 系统消息
SystemMessage systemMessage = new SystemMessage("你是助手");

// 用户消息
UserMessage userMessage = new UserMessage("你好");

// 助手消息
AssistantMessage assistantMessage = new AssistantMessage("你好！");

// 工具消息
ToolResponseMessage toolMessage = new ToolResponseMessage(
    List.of(new ToolResponseMessage.ToolResponse("call_123", "工具结果"))
);

// 构建对话历史
List<Message> messages = List.of(
    systemMessage,
    userMessage,
    assistantMessage
);
```

### 3. Memory（记忆）

#### 3.1 MemorySaver（内存存储）
```java
@Bean
public MemorySaver memorySaver() {
    return new MemorySaver();
}
```

#### 3.2 Redis 存储
```java
@Bean
public RedisSaver redisSaver(RedisTemplate<String, String> redisTemplate) {
    return new RedisSaver(redisTemplate);
}
```

#### 3.3 数据库存储
```java
// MySQL
@Bean
public MysqlSaver mysqlSaver(DataSource dataSource) {
    return new MysqlSaver(dataSource, CreateOption.CREATE_IF_NOT_EXISTS);
}

// PostgreSQL
@Bean
public PostgresSaver postgresSaver(DataSource dataSource) {
    return new PostgresSaver(dataSource, CreateOption.CREATE_IF_NOT_EXISTS);
}

// MongoDB
@Bean
public MongoSaver mongoSaver(MongoTemplate mongoTemplate) {
    return new MongoSaver(mongoTemplate);
}
```

---

## Agent 开发

### 1. 基础 ReactAgent

ReactAgent 是 Spring AI Alibaba 的核心 Agent 实现，支持推理-行动循环。

```java
@Configuration
public class AgentConfig {

    private static final String INSTRUCTION = """
        You are a helpful assistant named SAA.
        You have access to tools that can help you accomplish tasks.
        Think step by step and use tools when necessary.
        """;

    @Bean
    public ReactAgent simpleAgent(
            ChatModel chatModel,
            MemorySaver memorySaver) {
        return ReactAgent.builder()
            .name("SimpleAgent")
            .model(chatModel)
            .instruction(INSTRUCTION)
            .saver(memorySaver)
            .enableLogging(true)
            .build();
    }

    @Bean
    public MemorySaver memorySaver() {
        return new MemorySaver();
    }
}
```

### 2. 带工具调用的 Agent

```java
@Configuration
public class ToolAgentConfig {

    private static final String INSTRUCTION = """
        You are a data analysis assistant.
        Use the available tools to fetch and analyze data.
        """;

    @Bean
    public ReactAgent toolAgent(
            ChatModel chatModel,
            MemorySaver memorySaver,
            ToolCallback searchTool,
            ToolCallback calculateTool) {
        return ReactAgent.builder()
            .name("ToolAgent")
            .model(chatModel)
            .instruction(INSTRUCTION)
            .tools(searchTool, calculateTool)
            .saver(memorySaver)
            .maxToolCalls(10)  // 限制工具调用次数
            .enableLogging(true)
            .build();
    }
}
```

### 3. 流式输出 Agent

```java
@Service
public class StreamingAgentService {

    @Autowired
    private ReactAgent agent;

    public Flux<String> streamChat(String input) {
        return agent.stream(input)
            .map(output -> output.getOutput().getText());
    }
}
```

---

## Tools（工具）

### 1. Function Tool

```java
// 定义工具函数
public class SearchTool {

    public record SearchRequest(String query, int limit) {}
    public record SearchResult(List<String> results) {}

    @Tool(description = "Search for information")
    public SearchResult search(SearchRequest request) {
        // 实现搜索逻辑
        return new SearchResult(List.of("result1", "result2"));
    }
}

// 注册为 Spring Bean
@Bean
public ToolCallback searchTool() {
    return FunctionToolCallback.builder("search", new SearchTool())
        .description("Search for information in the knowledge base")
        .inputType(SearchTool.SearchRequest.class)
        .build();
}
```

### 2. Spring Bean Tool

```java
@Component
public class CalculatorTool {

    @Tool(name = "add", description = "Add two numbers")
    public double add(@ToolParam(description = "First number") double a,
                      @ToolParam(description = "Second number") double b) {
        return a + b;
    }

    @Tool(name = "multiply", description = "Multiply two numbers")
    public double multiply(double a, double b) {
        return a * b;
    }
}

// 自动扫描并注册
@Bean
public ToolCallbackResolver toolCallbackResolver(CalculatorTool calculator) {
    return ToolCallbackResolver.builder()
        .tool(calculator)
        .build();
}
```

### 3. Tool Calling（手动调用）

```java
@Service
public class ManualToolService {

    @Autowired
    private ChatModel chatModel;

    public String chatWithTools(String input, List<ToolCallback> tools) {
        // 创建 Prompt
        Prompt prompt = new Prompt(
            new UserMessage(input),
            PromptOptions.builder()
                .toolCallbacks(tools)
                .build()
        );

        // 第一次调用获取工具请求
        ChatResponse response = chatModel.call(prompt);

        // 检查是否有工具调用
        if (response.getResult().getOutput() instanceof AssistantMessage assistantMsg
                && assistantMsg.hasToolCalls()) {

            // 执行工具调用
            List<ToolResponseMessage> toolResponses = new ArrayList<>();
            for (ToolCall toolCall : assistantMsg.getToolCalls()) {
                String result = executeTool(toolCall, tools);
                toolResponses.add(new ToolResponseMessage(
                    List.of(new ToolResponseMessage.ToolResponse(toolCall.id(), result))
                ));
            }

            // 第二次调用获取最终回复
            Prompt followUpPrompt = new Prompt(List.of(
                new UserMessage(input),
                new AssistantMessage(assistantMsg.getText()),
                toolResponses.get(0)
            ));

            ChatResponse finalResponse = chatModel.call(followUpPrompt);
            return finalResponse.getResult().getOutput().getText();
        }

        return response.getResult().getOutput().getText();
    }
}
```

---

## Skills（技能）

Skills 是可复用的 Agent 能力模块。

### 1. 定义 Skill

```java
// 创建 skill 目录和 SKILL.md
// src/main/resources/skills/data-analysis/SKILL.md

@Component
public class DataAnalysisSkill {

    private static final String SKILL_INSTRUCTION = """
        You are a data analysis expert.
        You can analyze CSV, JSON, and Excel files.
        Provide insights and visualizations.
        """;

    @Bean
    public ReactAgent dataAnalysisAgent(
            ChatModel chatModel,
            MemorySaver memorySaver) {
        return ReactAgent.builder()
            .name("DataAnalysisAgent")
            .model(chatModel)
            .instruction(SKILL_INSTRUCTION)
            .saver(memorySaver)
            .build();
    }

    @Bean
    public ToolCallback csvParserTool() {
        return FunctionToolCallback.builder("parse_csv", this::parseCsv)
            .description("Parse CSV file and return structured data")
            .inputType(CsvParseRequest.class)
            .build();
    }

    private CsvParseResult parseCsv(CsvParseRequest request) {
        // 实现 CSV 解析
        return new CsvParseResult();
    }
}
```

### 2. 使用 Skill

```java
@Service
public class SkillBasedAgent {

    @Autowired
    private ApplicationContext context;

    public ReactAgent loadSkill(String skillName) {
        // 从 Spring 上下文加载 Skill Agent
        return context.getBean(skillName + "Agent", ReactAgent.class);
    }
}
```

---

## Structured Output（结构化输出）

### 1. 使用 Bean Output Parser

```java
// 定义输出结构
public record ProductInfo(
    String name,
    double price,
    List<String> features,
    String category
) {}

@Service
public class StructuredOutputService {

    @Autowired
    private ChatModel chatModel;

    public ProductInfo extractProductInfo(String description) {
        BeanOutputParser<ProductInfo> parser = new BeanOutputParser<>(ProductInfo.class);

        String prompt = """
            Extract product information from the following description.
            {format}

            Description: {description}
            """;

        PromptTemplate template = new PromptTemplate(
            prompt,
            Map.of("format", parser.getFormat(), "description", description)
        );

        ChatResponse response = chatModel.call(template.create());
        return parser.parse(response.getResult().getOutput().getText());
    }
}
```

### 2. 使用 JSON Schema

```java
public JsonSchema getProductSchema() {
    return JsonSchema.builder()
        .type("object")
        .properties(Map.of(
            "name", JsonSchema.builder().type("string").build(),
            "price", JsonSchema.builder().type("number").build(),
            "features", JsonSchema.builder()
                .type("array")
                .items(JsonSchema.builder().type("string").build())
                .build()
        ))
        .required(List.of("name", "price"))
        .build();
}

// 使用 schema 获取结构化输出
ChatOptions options = DashScopeChatOptions.builder()
    .responseFormat(ResponseFormat.JSON)
    .jsonSchema(getProductSchema())
    .build();
```

---

## Hooks 和 Interceptors

### 1. Hooks（钩子）

Hooks 允许在 Agent 执行过程中插入自定义逻辑。

#### 1.1 自定义 Hook

```java
@Component
public class LoggingHook implements AgentHook {

    private static final Logger log = LoggerFactory.getLogger(LoggingHook.class);

    @Override
    public void beforeCall(AgentContext context) {
        log.info("Agent {} starting iteration {}",
            context.getAgentName(),
            context.getIteration());
    }

    @Override
    public void afterCall(AgentContext context, AgentOutput output) {
        log.info("Agent {} completed iteration {} with {} tool calls",
            context.getAgentName(),
            context.getIteration(),
            output.getToolCalls().size());
    }

    @Override
    public void onError(AgentContext context, Exception error) {
        log.error("Agent {} encountered error: {}",
            context.getAgentName(),
            error.getMessage());
    }
}
```

#### 1.2 使用 Hook

```java
@Bean
public ReactAgent agentWithHook(
        ChatModel chatModel,
        LoggingHook loggingHook) {
    return ReactAgent.builder()
        .name("AgentWithHook")
        .model(chatModel)
        .instruction(INSTRUCTION)
        .hooks(loggingHook)
        .build();
}
```

### 2. Interceptors（拦截器）

拦截器用于在消息处理前后执行操作。

```java
@Component
public class TokenLimitInterceptor implements MessageInterceptor {

    private final int maxTokens = 4000;

    @Override
    public List<Message> intercept(List<Message> messages) {
        int totalTokens = estimateTokens(messages);

        if (totalTokens > maxTokens) {
            // 压缩或截断消息
            return compressMessages(messages);
        }

        return messages;
    }

    private int estimateTokens(List<Message> messages) {
        // 估算 token 数量
        return messages.stream()
            .mapToInt(m -> m.getText().length() / 4)
            .sum();
    }
}
```

---

## 上下文工程

### 1. 上下文压缩

```java
@Bean
public ReactAgent agentWithCompaction(ChatModel chatModel) {
    return ReactAgent.builder()
        .name("CompactionAgent")
        .model(chatModel)
        .instruction(INSTRUCTION)
        .hooks(ContextCompactionHook.builder()
            .maxTokens(4000)
            .compactionStrategy(new SummaryCompactionStrategy())
            .build())
        .build();
}
```

### 2. 上下文编辑

```java
@Bean
public ReactAgent agentWithEditing(ChatModel chatModel) {
    return ReactAgent.builder()
        .name("EditingAgent")
        .model(chatModel)
        .instruction(INSTRUCTION)
        .hooks(ContextEditingHook.builder()
            .allowUserEdit(true)
            .editTrigger(ctx -> ctx.getIteration() > 5)
            .build())
        .build();
}
```

### 3. 动态工具选择

```java
@Bean
public ReactAgent agentWithDynamicTools(ChatModel chatModel) {
    return ReactAgent.builder()
        .name("DynamicToolAgent")
        .model(chatModel)
        .instruction(INSTRUCTION)
        .hooks(DynamicToolSelectionHook.builder()
            .toolSelector((query, availableTools) -> {
                // 根据查询选择相关工具
                return availableTools.stream()
                    .filter(t -> t.getDescription().contains("search"))
                    .collect(Collectors.toList());
            })
            .build())
        .build();
}
```

---

## 人工介入（Human-in-the-Loop）

### 1. 基础人工介入

```java
@Bean
public ReactAgent agentWithHumanApproval(ChatModel chatModel) {
    return ReactAgent.builder()
        .name("HumanLoopAgent")
        .model(chatModel)
        .instruction(INSTRUCTION)
        .hooks(HumanInTheLoopHook.builder()
            .triggerCondition(ctx -> ctx.getIteration() > 3)
            .approvalPrompt("请审核以下行动计划：")
            .build())
        .build();
}
```

### 2. 工具调用前确认

```java
@Bean
public ReactAgent agentWithToolConfirmation(ChatModel chatModel) {
    return ReactAgent.builder()
        .name("ToolConfirmAgent")
        .model(chatModel)
        .instruction(INSTRUCTION)
        .hooks(ToolConfirmationHook.builder()
            .requireConfirmationForTools(Set.of("delete", "update", "send"))
            .confirmationPrompt(tool -> "确认执行工具: " + tool.getName() + "?")
            .build())
        .build();
}
```

### 3. WebSocket 实时介入

```java
@Service
public class WebSocketHumanLoopService {

    public ReactAgent createInteractiveAgent(
            ChatModel chatModel,
            SimpMessagingTemplate messaging) {

        return ReactAgent.builder()
            .name("InteractiveAgent")
            .model(chatModel)
            .instruction(INSTRUCTION)
            .hooks(new HumanInTheLoopHook() {
                @Override
                public boolean shouldInterrupt(AgentContext context) {
                    return context.getIteration() > 2;
                }

                @Override
                public String getUserInput(AgentContext context) {
                    // 通过 WebSocket 发送请求并等待用户输入
                    messaging.convertAndSend("/topic/pause", context);
                    return waitForUserResponse(context.getConversationId());
                }
            })
            .build();
    }
}
```

---

## 多智能体编排

### 1. SequentialAgent（顺序执行）

```java
@Bean
public SequentialAgent pipelineAgent(
        ReactAgent dataExtractionAgent,
        ReactAgent analysisAgent,
        ReactAgent reportAgent) {

    return SequentialAgent.builder()
        .name("DataPipeline")
        .agents(List.of(
            dataExtractionAgent,
            analysisAgent,
            reportAgent
        ))
        .stateAggregator((states) -> {
            // 合并各 Agent 的状态
            Map<String, Object> aggregated = new HashMap<>();
            states.forEach(s -> aggregated.putAll(s.getData()));
            return aggregated;
        })
        .build();
}

// 使用
@Service
public class PipelineService {

    @Autowired
    private SequentialAgent pipelineAgent;

    public String runPipeline(String input) {
        return pipelineAgent.call(input);
    }
}
```

### 2. ParallelAgent（并行执行）

```java
@Bean
public ParallelAgent parallelAnalysisAgent(
        ReactAgent sentimentAgent,
        ReactAgent entityAgent,
        ReactAgent summaryAgent) {

    return ParallelAgent.builder()
        .name("ParallelAnalyzer")
        .agents(List.of(
            sentimentAgent,
            entityAgent,
            summaryAgent
        ))
        .maxConcurrency(3)
        .resultMerger((results) -> {
            // 合并并行执行结果
            StringBuilder merged = new StringBuilder();
            results.forEach(r -> merged.append(r).append("\n"));
            return merged.toString();
        })
        .build();
}
```

### 3. RoutingAgent（路由选择）

```java
@Bean
public RoutingAgent smartRouterAgent(
        ChatModel chatModel,
        Map<String, ReactAgent> specializedAgents) {

    return RoutingAgent.builder()
        .name("SmartRouter")
        .model(chatModel)
        .instruction("""
            Analyze the user query and route to the most appropriate agent:
            - technical: Technical support agent
            - billing: Billing agent
            - general: General inquiry agent
            """)
        .agents(specializedAgents)
        .routingStrategy((query, agents) -> {
            // 自定义路由逻辑
            if (query.contains("price") || query.contains("bill")) {
                return agents.get("billing");
            } else if (query.contains("error") || query.contains("bug")) {
                return agents.get("technical");
            }
            return agents.get("general");
        })
        .build();
}
```

### 4. LoopAgent（循环执行）

```java
@Bean
public LoopAgent iterativeRefinementAgent(
        ReactAgent refinementAgent,
        ChatModel chatModel) {

    return LoopAgent.builder()
        .name("RefinementLoop")
        .agent(refinementAgent)
        .maxIterations(5)
        .exitCondition((context, output) -> {
            // 当质量达到要求时退出
            return output.getText().contains("FINAL")
                || context.getIteration() >= 5;
        })
        .iterationPrompt((iteration, previousOutput) ->
            "Iteration " + iteration + ". Previous: " + previousOutput)
        .build();
}
```

---

## 智能体作为工具

将一个 Agent 作为工具提供给另一个 Agent 使用。

```java
@Configuration
public class AgentAsToolConfig {

    // 定义专门的研究 Agent
    @Bean
    public ReactAgent researchAgent(ChatModel chatModel) {
        return ReactAgent.builder()
            .name("ResearchAgent")
            .model(chatModel)
            .instruction("You are a research specialist. Deep dive into topics.")
            .build();
    }

    // 将 Research Agent 包装为工具
    @Bean
    public ToolCallback researchTool(ReactAgent researchAgent) {
        return AgentToolCallback.builder()
            .agent(researchAgent)
            .name("deep_research")
            .description("Perform deep research on a given topic")
            .inputType(ResearchRequest.class)
            .build();
    }

    // 主 Agent 使用 Research Agent 作为工具
    @Bean
    public ReactAgent mainAgent(
            ChatModel chatModel,
            ToolCallback researchTool) {
        return ReactAgent.builder()
            .name("MainAgent")
            .model(chatModel)
            .instruction("You are a general assistant with research capabilities.")
            .tools(researchTool)
            .build();
    }
}
```

---

## 工作流（Workflow）

### 使用 StateGraph 构建复杂工作流

```java
@Service
public class ComplexWorkflowService {

    // 定义工作流状态
    public record WorkflowState(
        String input,
        String processedData,
        String analysisResult,
        String finalOutput,
        Map<String, Object> metadata
    ) {}

    public CompiledGraph<WorkflowState> buildWorkflow() {
        return new StateGraph<>(WorkflowState.class)
            // 定义节点
            .addNode("preprocess", this::preprocessNode)
            .addNode("analyze", this::analysisNode)
            .addNode("enrich", this::enrichmentNode)
            .addNode("format", this::formattingNode)

            // 定义条件分支
            .addConditionalEdge("preprocess", this::routeByComplexity,
                Map.of(
                    "simple", "format",
                    "complex", "analyze"
                ))
            .addEdge("analyze", "enrich")
            .addEdge("enrich", "format")

            // 入口和出口
            .addEdge(START, "preprocess")
            .addEdge("format", END)

            // 编译
            .compile();
    }

    private WorkflowState preprocessNode(WorkflowState state) {
        // 数据预处理逻辑
        String processed = state.input().toUpperCase();
        return new WorkflowState(
            state.input(),
            processed,
            state.analysisResult(),
            state.finalOutput(),
            state.metadata()
        );
    }

    private String routeByComplexity(WorkflowState state) {
        return state.input().length() > 100 ? "complex" : "simple";
    }

    // 其他节点实现...
}
```

---

## RAG（检索增强生成）

### 1. 基础 RAG Agent

```java
@Configuration
public class RagConfig {

    @Bean
    public VectorStore vectorStore(EmbeddingModel embeddingModel) {
        // 使用 SimpleVectorStore 或 Redis/PGVector
        return SimpleVectorStore.builder(embeddingModel).build();
    }

    @Bean
    public DocumentRetriever documentRetriever(VectorStore vectorStore) {
        return VectorStoreDocumentRetriever.builder()
            .vectorStore(vectorStore)
            .similarityThreshold(0.7)
            .topK(5)
            .build();
    }

    @Bean
    public ReactAgent ragAgent(
            ChatModel chatModel,
            DocumentRetriever retriever) {

        String ragInstruction = """
            You are a helpful assistant with access to a knowledge base.
            Use the retrieved documents to answer questions accurately.
            If the documents don't contain the answer, say so clearly.
            """;

        return ReactAgent.builder()
            .name("RagAgent")
            .model(chatModel)
            .instruction(ragInstruction)
            .tools(RetrievalTool.builder()
                .retriever(retriever)
                .name("retrieve_documents")
                .description("Retrieve relevant documents from knowledge base")
                .build())
            .build();
    }
}
```

### 2. 文档加载和索引

```java
@Service
public class DocumentIndexingService {

    @Autowired
    private VectorStore vectorStore;

    public void indexDocuments(List<Resource> documents) {
        TokenTextSplitter splitter = new TokenTextSplitter(
            500,  // chunk size
            50,   // overlap
            10,   // min chunk size
            1000, // max chunk size
            true  // keep separator
        );

        for (Resource doc : documents) {
            // 读取文档
            String content = readDocument(doc);

            // 分块
            List<String> chunks = splitter.split(content);

            // 创建 Document 对象并添加元数据
            List<Document> documents = chunks.stream()
                .map(chunk -> new Document(
                    chunk,
                    Map.of(
                        "source", doc.getFilename(),
                        "timestamp", Instant.now().toString()
                    )
                ))
                .toList();

            // 存入向量数据库
            vectorStore.add(documents);
        }
    }
}
```

### 3. 多查询 RAG

```java
@Bean
public ReactAgent multiQueryRagAgent(ChatModel chatModel) {
    return ReactAgent.builder()
        .name("MultiQueryRagAgent")
        .model(chatModel)
        .instruction(INSTRUCTION)
        .tools(MultiQueryRetrievalTool.builder()
            .baseRetriever(documentRetriever)
            .queryGenerator((originalQuery) -> {
                // 生成多个相关查询
                return List.of(
                    originalQuery,
                    "What is " + originalQuery,
                    "Explain " + originalQuery,
                    originalQuery + " definition"
                );
            })
            .fusionStrategy(new RRFFusionStrategy())  // Reciprocal Rank Fusion
            .build())
        .build();
}
```

---

## Graph（图工作流）

### 1. 基础 Graph

```java
@Service
public class GraphWorkflowService {

    public CompiledGraph<Map<String, Object>> createSimpleGraph() {
        return new StateGraph<Map<String, Object>>(Map.class)
            .addNode("step1", (state) -> {
                state.put("step1_result", "processed");
                return state;
            })
            .addNode("step2", (state) -> {
                state.put("step2_result", "completed");
                return state;
            })
            .addEdge(START, "step1")
            .addEdge("step1", "step2")
            .addEdge("step2", END)
            .compile();
    }
}
```

### 2. 带持久化的 Graph

```java
@Bean
public CompiledGraph<AppState> persistentGraph(
        BaseCheckpointSaver saver) {

    return new StateGraph<>(AppState.class)
        .addNode("process", this::processNode)
        .addNode("review", this::reviewNode)
        .addEdge(START, "process")
        .addEdge("process", "review")
        .addConditionalEdge("review", this::shouldContinue,
            Map.of("continue", END, "revise", "process"))
        .compile(
            CompileOptions.builder()
                .checkpointSaver(saver)
                .build()
        );
}

// 使用时间旅行恢复
public AppState resumeFromCheckpoint(
        CompiledGraph<AppState> graph,
        String threadId,
        int checkpointId) {

    return graph.invoke(
        new AppState(),
        RunnableConfig.builder()
            .threadId(threadId)
            .checkpointId(checkpointId)
            .build()
    );
}
```

### 3. 子图（Subgraph）

```java
public StateGraph<ParentState> createParentGraph() {
    // 创建子图
    StateGraph<ChildState> childGraph = new StateGraph<>(ChildState.class)
        .addNode("child_process", this::childProcess)
        .addEdge(START, "child_process")
        .addEdge("child_process", END);

    // 父图使用子图作为节点
    return new StateGraph<>(ParentState.class)
        .addNode("parent_start", this::parentStart)
        .addSubgraph("child_workflow", childGraph, this::stateMapper)
        .addNode("parent_end", this::parentEnd)
        .addEdge(START, "parent_start")
        .addEdge("parent_start", "child_workflow")
        .addEdge("child_workflow", "parent_end")
        .addEdge("parent_end", END);
}
```

---

## A2A（Agent-to-Agent）

A2A 允许分布式 Agent 相互通信协作。

### 1. 配置 A2A

```java
@Configuration
@EnableA2A
public class A2AConfig {

    @Bean
    public AgentRegistry agentRegistry(NamingService namingService) {
        return new NacosAgentRegistry(namingService);
    }

    @Bean
    public A2AServer a2aServer(AgentRegistry registry) {
        return A2AServer.builder()
            .port(8080)
            .registry(registry)
            .build();
    }
}
```

### 2. 注册 Agent

```java
@Service
public class AgentRegistrationService {

    @Autowired
    private AgentRegistry registry;

    @PostConstruct
    public void register() {
        AgentDescriptor descriptor = AgentDescriptor.builder()
            .name("DataAnalysisAgent")
            .description("Performs data analysis tasks")
            .endpoint("http://localhost:8080/a2a")
            .capabilities(List.of(
                Capability.builder()
                    .name("analyze_csv")
                    .description("Analyze CSV files")
                    .build()
            ))
            .build();

        registry.register(descriptor);
    }
}
```

### 3. 调用远程 Agent

```java
@Service
public class A2AClientService {

    @Autowired
    private A2AClient a2aClient;

    public String callRemoteAgent(String agentName, String task) {
        // 发现 Agent
        AgentDescriptor agent = a2aClient.discover(agentName);

        // 构建任务请求
        TaskRequest request = TaskRequest.builder()
            .id(UUID.randomUUID().toString())
            .message(Message.builder()
                .role("user")
                .content(task)
                .build())
            .build();

        // 发送任务并获取结果
        TaskResponse response = a2aClient.sendTask(
            agent.getEndpoint(),
            request
        );

        return response.getResult();
    }
}
```

---

## Agent Chat UI

Spring AI Alibaba 提供内置的 Chat UI。

### 1. 添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-studio</artifactId>
    <version>1.1.2.2</version>
</dependency>
```

### 2. 配置 Chat UI

```yaml
spring:
  ai:
    alibaba:
      studio:
        enabled: true
        path: /chatui
        agent:
          name: MyAgent
          description: A helpful assistant
```

### 3. 自定义 UI

```java
@Configuration
public class ChatUiConfig {

    @Bean
    public ChatUiCustomizer chatUiCustomizer() {
        return builder -> builder
            .title("My Custom Chat")
            .theme("dark")
            .welcomeMessage("Welcome! How can I help you today?")
            .logo("/custom-logo.png");
    }
}
```

访问 `http://localhost:8080/chatui/index.html` 使用界面。

---

## Admin

Spring AI Alibaba Admin 提供可视化的 Agent 开发平台。

### 部署 Admin

```bash
# Docker 部署
docker run -p 8080:8080 \
  -e AI_DASHSCOPE_API_KEY=${AI_DASHSCOPE_API_KEY} \
  springai/spring-ai-alibaba-admin:1.1.2.2
```

### Admin 功能

- **可视化 Agent 开发**: 拖拽式界面设计 Agent
- **工作流编排**: 可视化设计复杂工作流
- **MCP 管理**: 管理 Model Context Protocol 服务
- **可观测性**: 查看 Agent 执行轨迹和性能
- **评估**: 测试和评估 Agent 性能
- **导出项目**: 将可视化设计导出为独立 Java 项目

---

## DeepResearch

DeepResearch 是基于 Spring AI Alibaba 的深度研究 Agent。

### 使用 DeepResearch

```java
@Service
public class DeepResearchService {

    @Autowired
    private DeepResearchAgent researchAgent;

    public ResearchReport conductResearch(String topic) {
        return researchAgent.research(
            ResearchRequest.builder()
                .topic(topic)
                .depth(Depth.COMPREHENSIVE)
                .sources(List.of(
                    SourceType.WEB,
                    SourceType.ACADEMIC,
                    SourceType.NEWS
                ))
                .build()
        );
    }
}
```

### 自定义 Research Agent

```java
@Bean
public DeepResearchAgent customResearchAgent(
        ChatModel chatModel,
        List<ResearchTool> tools) {

    return DeepResearchAgent.builder()
        .name("CustomResearcher")
        .model(chatModel)
        .tools(tools)
        .maxIterations(10)
        .outputFormat(ResearchReport.class)
        .build();
}
```

---

## Data Agent

Data Agent 用于自然语言查询数据库。

### 1. 配置 Data Agent

```java
@Configuration
public class DataAgentConfig {

    @Bean
    public DataAgent dataAgent(
            ChatModel chatModel,
            DataSource dataSource) {

        return DataAgent.builder()
            .name("SQLAgent")
            .model(chatModel)
            .database(DatabaseInfo.builder()
                .dataSource(dataSource)
                .schema("public")
                .excludedTables(List.of("users_passwords"))
                .build())
            .maxRetries(3)
            .confirmationRequired(true)  // SQL 执行前确认
            .build();
    }
}
```

### 2. 使用 Data Agent

```java
@Service
public class DataQueryService {

    @Autowired
    private DataAgent dataAgent;

    public QueryResult query(String naturalLanguageQuery) {
        return dataAgent.query(
            "查询上个月销售额最高的前 10 个产品"
        );
    }
}
```

---

## Assistant Agent

Assistant Agent 是通用的智能助手实现。

```java
@Bean
public AssistantAgent assistantAgent(
        ChatModel chatModel,
        List<ToolCallback> tools,
        MemorySaver memorySaver) {

    return AssistantAgent.builder()
        .name("Assistant")
        .model(chatModel)
        .instruction("""
            You are a general-purpose AI assistant.
            You can help with various tasks including:
            - Answering questions
            - Performing calculations
            - Searching information
            - Writing and editing text

            Be helpful, accurate, and concise.
            """)
        .tools(tools)
        .memory(memorySaver)
        .enableFileAccess(true)
        .enableCodeExecution(true)
        .build();
}
```

---

## 完整示例项目结构

```
my-spring-ai-app/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/myapp/
│   │   │       ├── MyApplication.java
│   │   │       ├── config/
│   │   │       │   ├── AgentConfig.java
│   │   │       │   ├── ToolConfig.java
│   │   │       │   └── PersistenceConfig.java
│   │   │       ├── agents/
│   │   │       │   ├── CustomerServiceAgent.java
│   │   │       │   └── TechnicalSupportAgent.java
│   │   │       ├── tools/
│   │   │       │   ├── SearchTool.java
│   │   │       │   └── CalculatorTool.java
│   │   │       ├── workflow/
│   │   │       │   └── TicketWorkflow.java
│   │   │       └── service/
│   │   │           └── AgentService.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── skills/
│   │       │   └── data-analysis/
│   │       │       └── SKILL.md
│   │       └── agents/
│   │           └── customer-service.md
│   └── test/
│       └── java/
│           └── com/example/myapp/
│               └── AgentTest.java
```

---

## 生产环境最佳实践

### 1. 配置管理

```yaml
spring:
  ai:
    alibaba:
      agent:
        default-max-iterations: 10
        default-max-tool-calls: 20
        logging-enabled: true
      graph:
        persistence:
          type: redis  # redis, mysql, postgres
          redis:
            host: ${REDIS_HOST}
            port: 6379
```

### 2. 监控和可观测性

```java
@Bean
public AgentObservationCustomizer observationCustomizer() {
    return (observation) -> observation
        .lowCardinalityKeyValue("agent.name", "ProductionAgent")
        .highCardinalityKeyValue("conversation.id", getConversationId());
}
```

### 3. 安全配置

```java
@Configuration
public class AgentSecurityConfig {

    @Bean
    public ToolCallback secureTool(ToolCallback originalTool) {
        return new SecureToolWrapper(originalTool,
            tool -> {
                // 验证工具调用权限
                return SecurityContextHolder.getContext()
                    .getAuthentication()
                    .getAuthorities()
                    .contains(new SimpleGrantedAuthority("TOOL:" + tool.getName()));
            });
    }
}
```

### 4. 性能优化

- 使用连接池管理模型客户端
- 启用响应缓存
- 异步处理非关键路径
- 限制上下文窗口大小

---

## 参考资源

- **官方文档**: https://java2ai.com/docs/overview
- **GitHub**: https://github.com/alibaba/spring-ai-alibaba
- **示例项目**: https://github.com/alibaba/spring-ai-alibaba/tree/main/examples
- **钉钉群**: 130240015687
- **微信公众号**: 搜索 "Spring AI Alibaba"

---

## 开发检查清单

创建 Spring AI Alibaba 应用时，请确认：

- [ ] 已添加核心依赖（agent-framework、模型 starter）
- [ ] 已配置 API Key（环境变量或配置文件）
- [ ] 已选择合适的开发模式（ReactAgent、Workflow、Graph）
- [ ] 已定义清晰的系统提示（Instruction）
- [ ] 已配置必要的工具（Tools）
- [ ] 已配置持久化方案（开发：MemorySaver，生产：Redis/DB）
- [ ] 已添加适当的 Hooks（监控、限制、人工介入）
- [ ] 已启用日志记录（开发阶段）
- [ ] 已编写单元测试和集成测试
- [ ] 已配置监控和可观测性（生产环境）
- [ ] 已实现安全控制（工具调用权限、输入验证）
