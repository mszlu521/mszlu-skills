# 码神 Skills 集合

码神（[www.mszlu.com](https://www.mszlu.com)）的 Claude Skills 集合，用于在 Claude Code 中提供各类开发指导和最佳实践。

## Skills 列表

| Skill | 描述 |
|-------|------|
| [spring-ai-alibaba-dev-guide](skills/spring-ai-alibaba-dev-guide/) | Spring AI Alibaba 完整开发指导，涵盖 Agents、Models、Tools、Memory、Hooks、多智能体编排、RAG、Graph 等所有核心功能 |

> 持续更新中，敬请期待更多 Skills...

## 快速开始

### 安装 Skill

```bash
npx skills add https://github.com/mszlu521/mszlu-skills --skill spring-ai-alibaba-dev-guide
```

### 本地安装

```bash
# 克隆仓库
git clone https://github.com/mszlu521/mszlu-skills.git

# 复制到 Claude Code skills 目录
cp -r mszlu-skills/skills/spring-ai-alibaba-dev-guide ~/.claude/skills/
```

## Skill 详细说明

### spring-ai-alibaba-dev-guide

当用户需要开发基于 Spring AI Alibaba 的 AI 应用时，此 Skill 提供全面的开发指导。

**触发场景：**
- 用户询问如何使用 Spring AI Alibaba 开发 AI 应用
- 用户需要创建 Agent、ChatBot、工作流、RAG 系统
- 用户询问多智能体编排、工具调用、MCP 集成
- 用户需要配置模型、记忆管理、上下文工程
- 用户询问 Hooks、Interceptors、Skills 机制
- 用户需要 A2A、Graph 工作流、DeepResearch、Data Agent
- 用户遇到 Spring AI Alibaba 相关的任何开发问题
- 用户需要最佳实践、示例代码、项目结构指导

**涵盖内容：**
- Models（模型配置与调用）
- Messages（消息类型与对话管理）
- Memory（记忆存储方案）
- Agents（ReactAgent、Assistant Agent 等）
- Tools（工具定义与调用）
- Skills（可复用技能模块）
- Hooks & Interceptors（钩子与拦截器）
- 多智能体编排（Sequential、Parallel、Routing、Loop）
- 工作流与 Graph
- RAG（检索增强生成）
- A2A（Agent-to-Agent 通信）
- Chat UI & Admin
- DeepResearch & Data Agent

## 项目结构

```
mszlu-skills/
├── skills/
│   └── spring-ai-alibaba-dev-guide/
│       ├── SKILL.md              # Skill 主文件
│       ├── README.md             # Skill 说明
│       ├── evals/
│       │   └── evals.json        # 评估用例
│       └── ...
├── skills.json                   # Skills 索引
├── AGENTS.md                     # 仓库说明
└── README.md                     # 本文件
```

## 贡献

欢迎提交 Issue 和 PR 来完善 Skills 内容。

## 关于码神

- **网站**：[www.mszlu.com](https://www.mszlu.com)
- **GitHub**：[@mszlu521](https://github.com/mszlu521)

## 参考资源

- [Spring AI Alibaba 官方文档](https://java2ai.com/docs/overview)
- [Spring AI Alibaba GitHub](https://github.com/alibaba/spring-ai-alibaba)
- [skills.sh](https://skills.sh/) - Skills 平台
