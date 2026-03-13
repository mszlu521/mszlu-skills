## 附录：补充功能详解

### A. Chat Client 高级用法

#### A.1 完整 Chat Client 配置示例

```java
@Configuration
public class ChatClientConfig {
    
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder, 
                                  ChatMemory chatMemory) {
        return builder
            .defaultSystem("""
                你是一个专业的 AI 助手。
                请用中文回答用户的问题。
                回答要简洁、准确、有帮助。
                """)
            .defaultOptions(DashScopeChatOptions.builder()
                .model("qwen-max")
                .temperature(0.7)
                .maxTokens(2000)
                .build())
            .defaultAdvisors(
                new MessageChatMemoryAdvisor(chatMemory),
                new SimpleLoggerAdvisor(),
                new SafeGuardAdvisor()
            )
            .build();
    }
}
```

#### A.2 Chat Client 与 Agent 结合

```java
@Service
public class ChatClientAgentService {
    
    @Autowired
    private ChatClient chatClient;
    
    public String agenticChat(String input, List<ToolCallback> tools) {
        return chatClient.prompt()
            .system("""
                你是一个智能助手，可以使用工具来完成任务。
                当需要调用工具时，请先分析需求，然后选择合适的工具。
                """)
            .user(input)
            .tools(tools)
            .advisors(new RetryAdvisor(3))  // 重试 3 次
            .call()
            .content();
    }
}
```

### B. 多模态（Multimodality）

Spring AI Alibaba 支持多模态输入（文本 + 图片 + 音频）。

#### B.1 图片理解

```java
@Service
public class ImageAnalysisService {
    
    @Autowired
    private ChatModel chatModel;
    
    public String analyzeImage(String description, Resource image) {
        // 创建包含图片的消息
        Media imageMedia = new Media(
            MimeTypeUtils.IMAGE_PNG,
            image
        );
        
        UserMessage userMessage = new UserMessage(
            description,
            List.of(imageMedia)
        );
        
        Prompt prompt = new Prompt(userMessage);
        ChatResponse response = chatModel.call(prompt);
        
        return response.getResult().getOutput().getText();
    }
}

// 使用 Chat Client 简化
@Service
public class ImageChatService {
    
    @Autowired
    private ChatClient chatClient;
    
    public String chatWithImage(String text, Resource image) {
        return chatClient.prompt()
            .user(u -> u.text(text).media(MimeTypeUtils.IMAGE_PNG, image))
            .call()
            .content();
    }
}
```

#### B.2 图片生成

```java
@Service
public class ImageGenerationService {
    
    @Autowired
    private ImageModel imageModel;
    
    public Resource generateImage(String prompt) {
        ImagePrompt imagePrompt = new ImagePrompt(
            prompt,
            DashScopeImageOptions.builder()
                .model("wanx-v1")
                .width(1024)
                .height(1024)
                .build()
        );
        
        ImageResponse response = imageModel.call(imagePrompt);
        return response.getResult().getOutput();
    }
}

// 在 Agent 中使用图片生成工具
@Component
public class ImageGenerationTool {
    
    @Autowired
    private ImageModel imageModel;
    
    @Tool(name = "generate_image", 
          description = "Generate an image based on text description")
    public String generateImage(@ToolParam(description = "Image description") String description) {
        ImagePrompt prompt = new ImagePrompt(description);
        ImageResponse response = imageModel.call(prompt);
        String imageUrl = response.getResult().getUrl();
        return "Image generated: " + imageUrl;
    }
}
```

#### B.3 音频处理

```java
@Service
public class AudioService {
    
    @Autowired
    private AudioTranscriptionModel transcriptionModel;
    
    @Autowired
    private AudioSpeechModel speechModel;
    
    // 语音转文字
    public String transcribeAudio(Resource audioFile) {
        AudioTranscriptionPrompt prompt = 
            new AudioTranscriptionPrompt(audioFile);
        
        AudioTranscriptionResponse response = 
            transcriptionModel.call(prompt);
        
        return response.getResult().getOutput();
    }
    
    // 文字转语音
    public Resource textToSpeech(String text) {
        AudioSpeechPrompt prompt = new AudioSpeechPrompt(
            text,
            DashScopeAudioSpeechOptions.builder()
                .model("sambert-zhimao-v1")
                .voice("zhimao")
                .build()
        );
        
        AudioSpeechResponse response = speechModel.call(prompt);
        return response.getResult().getOutput();
    }
}
```

#### B.4 语音 Agent

```java
@Component
public class VoiceAgent {
    
    @Autowired
    private ChatClient chatClient;
    
    @Autowired
    private AudioTranscriptionModel transcriptionModel;
    
    @Autowired
    private AudioSpeechModel speechModel;
    
    public Resource voiceChat(Resource audioInput) {
        // 1. 语音转文字
        String userText = transcribe(audioInput);
        
        // 2. AI 处理
        String aiResponse = chatClient.prompt()
            .user(userText)
            .call()
            .content();
        
        // 3. 文字转语音
        return textToSpeech(aiResponse);
    }
    
    private String transcribe(Resource audio) {
        AudioTranscriptionPrompt prompt = 
            new AudioTranscriptionPrompt(audio);
        return transcriptionModel.call(prompt)
            .getResult()
            .getOutput();
    }
    
    private Resource textToSpeech(String text) {
        AudioSpeechPrompt prompt = new AudioSpeechPrompt(text);
        return speechModel.call(prompt)
            .getResult()
            .getOutput();
    }
}
```

### C. MCP（Model Context Protocol）详解

MCP 是 Anthropic 提出的开放协议，用于标准化 AI 模型与外部工具的集成。

#### C.1 MCP 基础概念

MCP 包含三种类型：
- **Resources**: 可被模型读取的数据资源
- **Tools**: 可被模型调用的功能
- **Prompts**: 预定义的提示模板

#### C.2 配置 MCP Server

```java
@Configuration
public class McpConfig {
    
    @Bean
    public McpSyncServer mcpServer() {
        return McpServer.sync()
            .serverInfo("my-mcp-server", "1.0.0")
            .tool("search", "Search knowledge base", this::handleSearch)
            .resource("docs://{path}", "Documentation content", this::handleDocs)
            .build();
    }
    
    private McpSchema.CallToolResult handleSearch(McpSchema.CallToolRequest request) {
        String query = (String) request.arguments().get("query");
        // 执行搜索
        List<String> results = searchService.search(query);
        
        return new McpSchema.CallToolResult(
            results.stream()
                .map(content -> new McpSchema.TextContent(content))
                .collect(Collectors.toList()),
            false
        );
    }
}
```

#### C.3 使用 MCP Client

```java
@Service
public class McpClientService {
    
    @Autowired
    private McpSyncClient mcpClient;
    
    public String useMcpTool(String toolName, Map<String, Object> args) {
        McpSchema.CallToolRequest request = 
            new McpSchema.CallToolRequest(toolName, args);
        
        McpSchema.CallToolResult result = mcpClient.callTool(request);
        
        return result.content().stream()
            .map(c -> ((McpSchema.TextContent) c).text())
            .collect(Collectors.joining("\n"));
    }
    
    public String readMcpResource(String uri) {
        McpSchema.ReadResourceRequest request = 
            new McpSchema.ReadResourceRequest(uri);
        
        McpSchema.ReadResourceResult result = 
            mcpClient.readResource(request);
        
        return result.contents().get(0).toString();
    }
}
```

#### C.4 在 Agent 中集成 MCP

```java
@Configuration
public class McpAgentConfig {
    
    @Bean
    public List<ToolCallback> mcpTools(McpSyncClient mcpClient) {
        // 从 MCP Server 获取工具列表
        McpSchema.ListToolsResult tools = mcpClient.listTools();
        
        // 转换为 Spring AI Tools
        return tools.tools().stream()
            .map(tool -> McpToolCallback.builder()
                .client(mcpClient)
                .toolName(tool.name())
                .description(tool.description())
                .build())
            .collect(Collectors.toList());
    }
    
    @Bean
    public ReactAgent mcpAgent(ChatModel chatModel,
                                List<ToolCallback> mcpTools) {
        return ReactAgent.builder()
            .name("MCPAgent")
            .model(chatModel)
            .instruction("You have access to MCP tools. Use them when needed.")
            .tools(mcpTools)
            .build();
    }
}
```

#### C.5 MCP Server 部署

```yaml
# application-mcp.yml
spring:
  ai:
    mcp:
      server:
        enabled: true
        name: my-mcp-server
        version: 1.0.0
        transport: stdio  # 或 sse
      client:
        servers:
          filesystem:
            transport: stdio
            command: npx
            args: [-y, "@modelcontextprotocol/server-filesystem", "/tmp"]
          fetch:
            transport: stdio
            command: uvx
            args: [mcp-server-fetch]
```

#### C.6 常用 MCP Servers

| Server | 功能 | 安装命令 |
|--------|------|----------|
| filesystem | 文件系统操作 | `npx @modelcontextprotocol/server-filesystem /path` |
| fetch | HTTP 请求 | `uvx mcp-server-fetch` |
| git | Git 操作 | `npx @modelcontextprotocol/server-git` |
| postgres | PostgreSQL 数据库 | `npx @modelcontextprotocol/server-postgres postgres://...` |
| sqlite | SQLite 数据库 | `npx @modelcontextprotocol/server-sqlite /path/db.sqlite` |
| brave-search | Brave 搜索 | `npx @modelcontextprotocol/server-brave-search` |

#### C.7 自定义 MCP Tool

```java
@Component
public class CustomMcpTool {
    
    @McpTool(name = "calculate",
             description = "Perform mathematical calculations")
    public String calculate(
            @McpParam(description = "Mathematical expression") 
            String expression) {
        try {
            // 使用脚本引擎计算
            ScriptEngine engine = new ScriptEngineManager()
                .getEngineByName("JavaScript");
            Object result = engine.eval(expression);
            return "Result: " + result;
        } catch (ScriptException e) {
            return "Error: " + e.getMessage();
        }
    }
    
    @McpResource(uri = "config://app",
                 description = "Application configuration")
    public String getConfig() {
        return "{\"version\": \"1.0\", \"env\": \"prod\"}";
    }
}
```

### D. Chat Client 完整示例

```java
@SpringBootApplication
public class ChatClientApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(ChatClientApplication.class, args);
    }
    
    @Bean
    ChatClient chatClient(ChatClient.Builder builder) {
        return builder.defaultSystem("You are a helpful AI assistant.").build();
    }
}

@RestController
class ChatController {
    
    private final ChatClient chatClient;
    
    ChatController(ChatClient chatClient) {
        this.chatClient = chatClient;
    }
    
    @GetMapping("/chat")
    String chat(@RequestParam String message) {
        return chatClient.prompt()
            .user(message)
            .call()
            .content();
    }
    
    @GetMapping(value = "/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    Flux<String> chatStream(@RequestParam String message) {
        return chatClient.prompt()
            .user(message)
            .stream()
            .content();
    }
}
```
