---
name: release-tag
description: |
  为当前提交打 annotated tag 并自动生成中文 release notes，不 push。
  Use when: (1) 用户想打 tag/发版/release, (2) 用户说 "/release-tag"、"打标签"、"发布版本",
  (3) 紧跟 bump-version 之后发版。默认读版本文件当前版本作为 tag 名，也支持显式指定。
---

# Release Tag

为当前提交打 annotated tag，并根据提交历史归纳中文 release notes。**只打 tag，不 push。**

## 调用形式

- `/release-tag` —— 默认读版本文件当前版本作为 tag 名
- `/release-tag 2.1.0` —— 显式指定 tag 名

## 工作流程

### 1. 前置检查

- 运行 `git rev-parse --is-inside-work-tree`；非 git 仓库 → 中止并提示。

### 2. 确定 tag 名

- **默认**：从版本文件读出当前版本（探测优先级同 bump-version：`package.json` → `pyproject.toml` → `Cargo.toml` → `pom.xml`）。紧跟 `/bump-version` 之后运行可无缝衔接。
- **显式**：使用用户传入的 tag 名。
- **前缀风格**：运行 `git tag` 查看历史 tag；若历史 tag 均带 `v` 前缀（如 `v1.2.0`）则跟随加 `v`；无历史 tag 则用纯数字（默认偏好，无 `v`）。
- 运行 `git rev-parse <tag名>` 检查；tag 已存在 → 中止并提示。

### 3. 生成 release notes

- 找上一个 tag：`git describe --tags --abbrev=0`；无历史 tag 则取全部提交。
- 取区间提交：`git log <上个tag>..HEAD --pretty=format:"%s"`（无上个 tag 时用 `git log --pretty=format:"%s"`）。
- 按 Conventional Commits 前缀归类，归纳成**精炼中文** release notes，按分组罗列要点：

| 前缀 | 分组标题 |
|----|----|
| feat | ✨ 新功能 |
| fix | 🐛 修复 |
| docs | 📝 文档 |
| style | 💄 格式 |
| refactor | ♻️ 重构 |
| perf | ⚡ 性能 |
| test | ✅ 测试 |
| chore | 🔧 杂项 |

- 仅保留有内容的分组；无可识别前缀的提交归入末尾「其它」。

### 4. 打 tag

- 将 release notes 写入临时文件，运行：`git tag -a <tag名> --cleanup=whitespace -F <release notes 文件>`（annotated tag）。
- 行首避免以 `#` 开头（或保留上面的 `--cleanup=whitespace`），否则会被 git 当作注释行删除。

### 5. 输出

- tag 名
- release notes 预览
- 待执行的 push 命令：`git push origin <当前分支> <tag名>`（用 `git rev-parse --abbrev-ref HEAD` 取分支名）
- 明确提示：**未 push**。
