## 内置 Nodes（内置节点）

Spring AI Alibaba 提供了丰富的内置节点，用于快速构建 Graph 工作流。

### 1. LlmNode（LLM 节点）

调用大语言模型进行推理。

```java
// 创建 LLM 节点
LlmNode llmNode = LlmNode.builder()
    .chatClient(chatClient)
    .systemPrompt("你是一个专业助手")
    .userPrompt("请分析以下数据：{{data}}")
    .outputKey("analysis_result")
    .build();

// 添加到 Graph
graph.addNode("analyze", llmNode);
```

**配置参数**：
- `systemPrompt`: 系统提示
- `userPrompt`: 用户提示
- `params`: 模板参数
- `messagesKey`: 从状态中获取消息列表的 key
- `outputKey`: 输出结果存储的 key
- `toolCallbacks`: 工具列表
- `stream`: 是否流式输出

### 2. ToolNode（工具节点）

执行工具调用。

```java
ToolNode toolNode = ToolNode.builder()
    .toolCallbacks(List.of(searchTool, calculatorTool))
    .inputKey("tool_input")
    .outputKey("tool_result")
    .build();
```

### 3. AgentNode（Agent 节点）

在工作流中嵌入 Agent。

```java
AgentNode agentNode = AgentNode.builder()
    .agent(reactAgent)
    .inputKey("user_query")
    .outputKey("agent_response")
    .build();

graph.addNode("process_with_agent", agentNode);
```

### 4. KnowledgeRetrievalNode（知识检索节点）

执行 RAG 检索。

```java
KnowledgeRetrievalNode retrievalNode = KnowledgeRetrievalNode.builder()
    .vectorStore(vectorStore)
    .queryKey("user_query")
    .topK(5)
    .similarityThreshold(0.7)
    .outputKey("retrieved_docs")
    .build();
```

### 5. HttpNode（HTTP 节点）

发起 HTTP 请求。

```java
HttpNode httpNode = HttpNode.builder()
    .url("https://api.example.com/data")
    .method(HttpMethod.GET)
    .headers(Map.of("Authorization", "Bearer token"))
    .queryParamsKey("params")
    .outputKey("api_response")
    .build();
```

### 6. McpNode（MCP 节点）

调用 MCP 工具。

```java
McpNode mcpNode = McpNode.builder()
    .mcpClient(mcpClient)
    .toolName("filesystem")
    .argumentsKey("mcp_args")
    .outputKey("mcp_result")
    .build();
```

### 7. QuestionClassifierNode（问题分类节点）

对输入问题进行分类。

```java
QuestionClassifierNode classifierNode = QuestionClassifierNode.builder()
    .chatClient(chatClient)
    .categories(List.of(
        Category.builder().name("technical").description("技术问题").build(),
        Category.builder().name("business").description("业务问题").build(),
        Category.builder().name("other").description("其他问题").build()
    ))
    .inputKey("user_question")
    .outputKey("category")
    .build();

// 条件分支
graph.addConditionalEdge("classify", classifierNode,
    Map.of(
        "technical", "tech_handler",
        "business", "biz_handler",
        "other", "general_handler"
    ));
```

### 8. TemplateTransformNode（模板转换节点）

使用模板转换数据。

```java
TemplateTransformNode transformNode = TemplateTransformNode.builder()
    .template("将以下内容总结为3点：{{content}}")
    .inputKey("raw_content")
    .outputKey("summary")
    .chatClient(chatClient)
    .build();
```

### 9. DocumentExtractorNode（文档提取节点）

从文档中提取内容。

```java
DocumentExtractorNode extractorNode = DocumentExtractorNode.builder()
    .extractor(new PdfDocumentExtractor())
    .filePathKey("document_path")
    .outputKey("document_text")
    .build();
```

### 10. IterationNode（迭代节点）

循环执行直到满足条件。

```java
IterationNode iterationNode = IterationNode.builder()
    .subGraph(iterationSubGraph)
    .maxIterations(10)
    .exitCondition(state -> state.getData().containsKey("complete"))
    .inputKey("iteration_input")
    .outputKey("iteration_result")
    .build();
```

### 11. AssignerNode（赋值节点）

在状态间赋值数据。

```java
AssignerNode assignerNode = AssignerNode.builder()
    .assignments(List.of(
        Assignment.from("source_key").to("target_key"),
        Assignment.from("input.user_id").to("user.id")
    ))
    .build();
```

### 12. AnswerNode（回答节点）

生成最终回答。

```java
AnswerNode answerNode = AnswerNode.builder()
    .answerTemplate("根据分析结果：{{result}}")
    .inputKey("final_result")
    .build();
```

### 13. VariableAggregatorNode（变量聚合节点）

聚合多个变量。

```java
VariableAggregatorNode aggregatorNode = VariableAggregatorNode.builder()
    .sourceKeys(List.of("result1", "result2", "result3"))
    .targetKey("aggregated_results")
    .aggregationStrategy(AggregationStrategy.LIST)
    .build();
```

### 14. ListOperatorNode（列表操作节点）

对列表进行操作。

```java
ListOperatorNode<String> listNode = ListOperatorNode.<String>builder()
    .inputKey("items")
    .outputKey("filtered_items")
    .operation(ListOperation.FILTER)
    .filter(item -> item.length() > 5)
    .build();
```

### 15. ParameterParsingNode（参数解析节点）

解析参数。

```java
ParameterParsingNode parsingNode = ParameterParsingNode.builder()
    .inputKey("raw_input")
    .schema(ParameterSchema.builder()
        .addParam("name", String.class, true)
        .addParam("age", Integer.class, false)
        .build())
    .outputKey("parsed_params")
    .build();
```

### 16. CodeExecutorNode（代码执行节点）

执行代码。

```java
CodeExecutorNodeAction codeNode = CodeExecutorNodeAction.builder()
    .language("python")
    .codeKey("code_to_execute")
    .outputKey("execution_result")
    .timeout(30)  // 秒
    .build();
```

### 17. HumanNode（人工节点）

等待人工输入。

```java
// HumanNode 通常用于需要人工确认的场景
// 具体实现需要结合前端或消息队列
```

---

## 使用示例：完整工作流

```java
@Service
public class DocumentAnalysisWorkflow {
    
    public CompiledGraph<AnalysisState> buildWorkflow() {
        return new StateGraph<>(AnalysisState.class)
            // 1. 提取文档内容
            .addNode("extract", DocumentExtractorNode.builder()
                .extractor(new PdfDocumentExtractor())
                .filePathKey("document_path")
                .outputKey("document_text")
                .build())
            
            // 2. 分类文档类型
            .addNode("classify", QuestionClassifierNode.builder()
                .chatClient(chatClient)
                .categories(List.of(
                    Category.builder().name("contract").description("合同文档").build(),
                    Category.builder().name("report").description("报告文档").build()
                ))
                .inputKey("document_text")
                .outputKey("doc_type")
                .build())
            
            // 3. 根据类型处理
            .addNode("process_contract", LlmNode.builder()
                .chatClient(chatClient)
                .systemPrompt("你是一个合同审查专家")
                .userPrompt("请审查以下合同：{{document_text}}")
                .outputKey("analysis")
                .build())
            
            .addNode("process_report", LlmNode.builder()
                .chatClient(chatClient)
                .systemPrompt("你是一个数据分析专家")
                .userPrompt("请分析报告内容：{{document_text}}")
                .outputKey("analysis")
                .build())
            
            // 4. 生成摘要
            .addNode("summarize", TemplateTransformNode.builder()
                .chatClient(chatClient)
                .template("请总结以下内容：{{analysis}}")
                .outputKey("summary")
                .build())
            
            // 5. 输出结果
            .addNode("output", AnswerNode.builder()
                .answerTemplate("文档分析完成：\n{{summary}}")
                .build())
            
            // 定义边
            .addEdge(START, "extract")
            .addEdge("extract", "classify")
            .addConditionalEdge("classify", 
                state -> state.getData().get("doc_type"),
                Map.of("contract", "process_contract", "report", "process_report"))
            .addEdge("process_contract", "summarize")
            .addEdge("process_report", "summarize")
            .addEdge("summarize", "output")
            .addEdge("output", END)
            .compile();
    }
}
```

---

## 自定义 Node

如果内置节点不满足需求，可以实现自定义 Node：

```java
@Component
public class CustomProcessingNode implements NodeAction {
    
    @Override
    public Map<String, Object> execute(OverAllState state) {
        // 获取输入
        String input = (String) state.getData().get("input_key");
        
        // 处理逻辑
        String processed = process(input);
        
        // 返回输出
        return Map.of("output_key", processed);
    }
    
    private String process(String input) {
        // 自定义处理逻辑
        return input.toUpperCase();
    }
}
```

然后在 Graph 中使用：

```java
graph.addNode("custom", new CustomProcessingNode());
```
