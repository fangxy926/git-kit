---
name: release-tag
description: |
  自动生成中文 release notes，同步写入 CHANGELOG.md 并提交，然后打 annotated tag，不 push。
  Use when: (1) 用户想打 tag/发版/release, (2) 用户说 "/release-tag"、"打标签"、"发布版本",
  (3) 紧跟 bump-version 之后发版。默认读版本文件当前版本作为 tag 名，也支持显式指定。
---

# Release Tag

根据提交历史归纳中文 release notes，写入项目根目录的 CHANGELOG.md 并提交，再打 annotated tag。**tag 描述与 CHANGELOG 条目内容保持一致；不 push。**

## 调用形式

- `/release-tag` —— 默认读版本文件当前版本作为 tag 名
- `/release-tag 2.1.0` —— 显式指定 tag 名

## 工作流程

### 1. 前置检查

- 运行 `git rev-parse --is-inside-work-tree`；非 git 仓库 → 中止并提示。
- 运行 `git status --porcelain`；若有未提交改动 → 警告并询问是否继续（本 skill 会产生一个 CHANGELOG 提交，避免混入无关改动）。

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
- 区间内没有任何提交 → 正文写「本次仅更新版本号」。
- 向用户展示 release notes 确认。**这份内容是唯一事实来源**：下一步写入 CHANGELOG.md 的条目正文与 tag 描述均使用它，逐字一致。

### 4. 写入 CHANGELOG.md 并提交

- 位置：**项目根目录** `CHANGELOG.md`；文件不存在 → **主动创建**，以 `# Changelog` 作为一级标题开头。
- 新条目插入在 `# Changelog` 标题之后、所有旧条目之前（最新版本在最上面），正文即上一步的 release notes：

```markdown
## [<版本>] - <YYYY-MM-DD>

### ✨ 新功能

- 要点一

### 🐛 修复

- 要点二
```

- 条目标题：semver 版本用 `## [x.y.z] - YYYY-MM-DD`；日期版本（如 `2026-07-09`）本身就是日期，只写 `## [2026-07-09]`，避免重复。
- 若 CHANGELOG.md 中**已存在同版本条目**：以现有条目为准作为 tag 描述（保证两处一致），并询问用户是否要用新生成的 notes 覆盖它。
- 不改动其它历史条目。
- `git add CHANGELOG.md`（只暂存该文件），**调用 git-commit-helper skill** 提交，形如 `docs: 更新 CHANGELOG（<版本>）`。

### 5. 打 tag

- 将 release notes 写入临时文件，运行：`git tag -a <tag名> --cleanup=whitespace -F <release notes 文件>`（annotated tag，指向刚才的 CHANGELOG 提交）。
- tag 描述正文必须与 CHANGELOG 条目正文**逐字一致**（仅条目标题 `## [...]` 行是 CHANGELOG 独有的，不进 tag 描述）。
- 行首避免以 `#` 开头（或保留上面的 `--cleanup=whitespace`），否则会被 git 当作注释行删除。

### 6. 输出

- tag 名 + 指向的 commit
- release notes 预览（即 CHANGELOG 新条目内容）
- CHANGELOG 提交（hash + message）
- 待执行的 push 命令：`git push origin <当前分支> <tag名>`（用 `git rev-parse --abbrev-ref HEAD` 取分支名）
- 明确提示：**未 push**。
