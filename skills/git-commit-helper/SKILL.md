---
name: git-commit-helper
description: |
  Automate git commit workflow with intelligent commit message generation.
  Use when: (1) User wants to commit changes, (2) User says "commit", "/commit",
  (3) User asks to summarize and commit changes, (4) User needs help writing commit messages.
  Analyzes git diff, generates conventional commit messages, and executes git commit.
---

# Git Commit Helper

Streamline git commits by analyzing changes and generating meaningful commit messages.

## 工作流程

1. **检查状态**: 运行 `git status` 查看暂存/未暂存的变更
2. **分析差异**: 运行 `git diff --staged`。暂存区非空时只提交已暂存的内容，不追加暂存其它文件（bump-version、release-tag 先只暂存目标文件再调用本 skill）；暂存区为空时运行 `git diff`，与用户确认要提交的文件后按文件名 `git add`
3. **生成消息**: 创建遵循 Conventional Commits 的提交消息
4. **确认消息**: 提交前向用户展示建议的消息
5. **执行提交**: 运行 `git commit -m "<消息>"`

## Commit Message Format

使用 Conventional Commits 格式：

```
<type>(<scope>): <中文描述>

[可选正文]

[可选脚注]
```

### Types（release-tag 按此归类 release notes）

| Type | 用途 |
|------|------|
| feat | 新功能 |
| fix | 修复 |
| docs | 文档 |
| style | 格式 |
| refactor | 重构 |
| perf | 性能 |
| test | 测试 |
| chore | 杂项/构建/依赖 |

### Guidelines

- Description: 使用中文，动词开头，不需要句号，最大 50 字符
- Body: 72 字符换行，说明做了什么以及为什么
- Scope: 可选，表示受影响的模块/组件

## Examples

**单文件变更：**
```
fix(auth): 修正密码验证逻辑
```

**多个相关变更：**
```
feat(api): 添加用户资料接口

- GET /api/users/:id
- PUT /api/users/:id
- 添加输入验证
```

**破坏性变更：**
```
feat(db)!: 迁移到 PostgreSQL

BREAKING CHANGE: 不再支持 SQLite
```
