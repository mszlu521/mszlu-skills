# Skills 集合说明

本仓库包含 Spring AI Alibaba 相关的 Skills，用于在 Claude Code 中提供开发指导。

## 目录结构

```
skills/
└── spring-ai-alibaba-dev-guide/    # Spring AI Alibaba 开发指导
    ├── SKILL.md                    # Skill 主文件
    ├── README.md                   # 说明文档
    └── evals/                      # 评估用例
        └── evals.json
```

## 使用方式

### 本地安装

将 skill 复制到 Claude Code 的 skills 目录：

```bash
# Claude Code
cp -r skills/spring-ai-alibaba-dev-guide ~/.claude/skills/
```

### 通过 npx skills 安装

```bash
npx skills add mszlu/mszlu-skills
```

## Skill 清单

| Skill | 描述 | 触发场景 |
|-------|------|----------|
| spring-ai-alibaba-dev-guide | Spring AI Alibaba 完整开发指导 | 开发 AI Agent、工作流、RAG 系统 |

## 开发规范

1. **SKILL.md 必须包含:**
   - `name`: skill 的唯一标识
   - `description`: 描述和触发场景
   - `metadata`: 作者、版本、标签等元数据

2. **目录命名:**
   - 使用 kebab-case（短横线连接）
   - 例如: `spring-ai-alibaba-dev-guide`
