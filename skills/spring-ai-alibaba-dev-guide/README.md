# Spring AI Alibaba 开发指导 Skill

本目录包含完整的 Spring AI Alibaba 开发指导文档，帮助开发者快速构建 AI 应用。

## 文档结构

```
spring-ai-alibaba-dev-guide/
├── SKILL.md                    # 主文档 - 核心功能完整指南（Spring Boot Starter 方式）
├── SKILL_APPENDIX.md           # 附录 - Chat Client、多模态、MCP 详解
├── SKILL_BUILT_IN_NODES.md     # 内置节点 - Graph 工作流预置节点
├── SKILL_AGENTS_SCOPE.md       # AgentScope 集成指南
├── SKILL_NON_STARTER.md        # 非 Starter 方式使用指南
├── README.md                   # 本文件 - 文档索引
└── evals/
    └── evals.json              # 测试用例
```

## 快速导航

### 新手入门
1. 阅读 [SKILL.md](SKILL.md) 的「快速开始」章节
2. 了解「核心概念」：Models、Messages、Memory
3. 学习「Agent 开发」：基础 ReactAgent

### 快速选择使用方式

| 项目类型 | 推荐文档 | 说明 |
|---------|---------|------|
| **Spring Boot 项目** | [SKILL.md](SKILL.md) | 使用 Starter，自动配置 |
| **Spring 项目（非 Boot）** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 7 | 手动配置 Bean |
| **纯 Java 项目** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 6 | 无 Spring 框架 |
| **Gradle 项目** | [SKILL.md](SKILL.md) 或 [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) | 两种都支持 |

### 功能模块

| 功能 | 文档位置 | 说明 |
|------|---------|------|
| **Chat Client** | [SKILL_APPENDIX.md](SKILL_APPENDIX.md) - A | 高级 API 简化模型调用 |
| **Chat Models** | [SKILL.md](SKILL.md) - 核心概念 | 多模型配置和使用 |
| **Multimodality** | [SKILL_APPENDIX.md](SKILL_APPENDIX.md) - B | 图片理解/生成、音频处理 |
| **MCP** | [SKILL_APPENDIX.md](SKILL_APPENDIX.md) - C | Model Context Protocol |
| **Tools** | [SKILL.md](SKILL.md) - 工具系统 | Function Tool、Spring Bean Tool |
| **Memory** | [SKILL.md](SKILL.md) - 记忆 | 多种存储方案 |
| **Hooks** | [SKILL.md](SKILL.md) - Hooks | 执行过程扩展点 |
| **Skills** | [SKILL.md](SKILL.md) - Skills | 可复用能力模块 |
| **多智能体编排** | [SKILL.md](SKILL.md) - 多智能体编排 | Sequential、Parallel、Routing、Loop |
| **Graph** | [SKILL.md](SKILL.md) - Graph | 状态图工作流 |
| **内置 Nodes** | [SKILL_BUILT_IN_NODES.md](SKILL_BUILT_IN_NODES.md) | 17+ 预置工作流节点 |
| **AgentScope** | [SKILL_AGENTS_SCOPE.md](SKILL_AGENTS_SCOPE.md) | AgentScope 框架集成 |
| **A2A** | [SKILL.md](SKILL.md) - A2A | Agent-to-Agent 通信 |
| **RAG** | [SKILL.md](SKILL.md) - RAG | 检索增强生成 |
| **上下文工程** | [SKILL.md](SKILL.md) - 上下文工程 | 压缩、编辑、动态工具 |
| **人工介入** | [SKILL.md](SKILL.md) - 人工介入 | Human-in-the-Loop |

### 非 Starter 方式

| 主题 | 文档位置 | 说明 |
|------|---------|------|
| **核心依赖** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 1 | Maven/Gradle 非 Starter 依赖 |
| **手动配置 ChatModel** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 2 | DashScope、OpenAI、Ollama 配置 |
| **手动创建 Agent** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 3 | 非自动配置方式 |
| **手动配置持久化** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 4 | Redis、MySQL、PostgreSQL |
| **纯 Java 项目** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 6 | 无 Spring 完整示例 |
| **Spring 非 Boot** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 7 | 传统 Spring 项目 |
| **模块化依赖** | [SKILL_NON_STARTER.md](SKILL_NON_STARTER.md) - 8 | 按需选择依赖 |

### 高级特性
- **结构化输出**：[SKILL.md](SKILL.md) - Structured Output
- **Interceptors**：[SKILL.md](SKILL.md) - Interceptors
- **智能体作为工具**：[SKILL.md](SKILL.md) - 智能体作为工具
- **持久化和状态管理**：[SKILL.md](SKILL.md) - 持久化
- **流式处理**：[SKILL.md](SKILL.md) - 流式处理

### 平台和工具
- **Chat UI**：[SKILL.md](SKILL.md) - Agent Chat UI
- **Admin**：[SKILL.md](SKILL.md) - Admin
- **DeepResearch**：[SKILL.md](SKILL.md) - DeepResearch
- **Data Agent**：[SKILL.md](SKILL.md) - Data Agent
- **Assistant Agent**：[SKILL.md](SKILL.md) - Assistant Agent

### 生产环境
- **配置管理**：[SKILL.md](SKILL.md) - 生产环境最佳实践
- **监控和可观测性**：[SKILL.md](SKILL.md) - 监控
- **安全配置**：[SKILL.md](SKILL.md) - 安全
- **性能优化**：[SKILL.md](SKILL.md) - 性能优化
- **开发检查清单**：[SKILL.md](SKILL.md) - 开发检查清单

## 版本信息

**当前版本**: `1.1.2.2`

所有示例代码均使用最新版本，请确保 Maven 依赖版本与此一致。

## 触发场景

本 Skill 在以下情况自动触发：
- 用户询问如何使用 Spring AI Alibaba 开发 AI 应用
- 用户需要创建 Agent、ChatBot、工作流、RAG 系统
- 用户询问多智能体编排、工具调用、MCP 集成
- 用户需要配置模型、记忆管理、上下文工程
- 用户询问 Hooks、Interceptors、Skills 机制
- 用户需要 A2A、Graph 工作流、DeepResearch、Data Agent
- 用户遇到 Spring AI Alibaba 相关的任何开发问题
- 用户需要最佳实践、示例代码、项目结构指导
- 用户不使用 Spring Boot Starter，需要手动配置

## 完整功能列表

本 Skill 涵盖 Spring AI Alibaba 的所有核心功能：

### 基础功能
- ✅ Chat Client
- ✅ Chat Models（DashScope、OpenAI、DeepSeek、Ollama 等）
- ✅ Messages（SystemMessage、UserMessage、AssistantMessage、ToolMessage）
- ✅ Memory（MemorySaver、Redis、MySQL、PostgreSQL、MongoDB）

### Agent 相关
- ✅ ReactAgent
- ✅ Tools（Function Tool、Spring Bean Tool、Tool Calling）
- ✅ Skills
- ✅ Structured Output
- ✅ Hooks
- ✅ Interceptors

### 多智能体
- ✅ SequentialAgent
- ✅ ParallelAgent
- ✅ RoutingAgent
- ✅ LoopAgent
- ✅ 智能体作为工具

### 工作流
- ✅ StateGraph
- ✅ 内置 Nodes（17+ 种）
- ✅ 子图（Subgraph）
- ✅ 持久化

### 其他功能
- ✅ RAG
- ✅ A2A
- ✅ 上下文工程
- ✅ 人工介入
- ✅ 多模态（图片、音频）
- ✅ MCP
- ✅ Chat UI
- ✅ Admin
- ✅ DeepResearch
- ✅ Data Agent
- ✅ Assistant Agent
- ✅ AgentScope 集成

### 部署方式
- ✅ Spring Boot Starter（自动配置）
- ✅ 非 Starter 手动配置
- ✅ 纯 Java 项目（无 Spring）
- ✅ Spring 非 Boot 项目

## 参考资源

- **官方文档**: https://java2ai.com/docs/overview
- **GitHub**: https://github.com/alibaba/spring-ai-alibaba
- **钉钉群**: 130240015687
- **微信公众号**: 搜索 "Spring AI Alibaba"

## 更新日志

### 1.1.2.2
- 初始版本
- 完整覆盖 Spring AI Alibaba 1.1.2.2 的所有功能
- 包含主文档、附录、内置节点、AgentScope 集成、非 Starter 指南五个文档
