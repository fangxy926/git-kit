---
name: bump-version
description: |
  更新项目版本号并提交（提交委托 git-commit-helper），不打 tag、不 push。
  Use when: (1) 用户想升级/更新版本号, (2) 用户说 "bump version"、"/bump-version"、"升版本",
  (3) 发版前需要先改版本号。支持日期版本（YYYY-MM-DD，默认首选）和传统 semver（major/minor/patch 关键词或显式版本号如 2.1.0）。
---

# Bump Version

更新项目版本号并通过 git-commit-helper 提交。**只改版本、只提交，不打 tag、不 push。**CHANGELOG.md 由 release-tag 在打 tag 时维护，本 skill 不涉及。

## 调用形式

支持日期（`YYYY-MM-DD(.N)`，默认首选）和 semver（`x.y.z`）两种版本方案：

- `/bump-version` 或 `/bump-version date` —— 日期方案：新版本 = 今天日期
- `/bump-version 2026-07-09` —— 显式日期版本号
- `/bump-version major|minor|patch` —— semver 关键词
- `/bump-version 2.1.0` —— 显式 semver 版本号

## 工作流程

### 1. 前置检查

- 运行 `git rev-parse --is-inside-work-tree`；非 git 仓库 → 中止并提示用户先 `git init`。
- 运行 `git status --porcelain`；若有未提交改动 → 警告并询问是否继续（避免把无关改动混入版本提交）。用户确认后才继续。

### 2. 探测版本文件

按以下优先级扫描**项目根目录**，读出当前版本号：

| 优先级 | 文件 | 版本字段 |
|----|----|----|
| 1 | `package.json` | 顶层 `"version"` |
| 2 | `pyproject.toml` | `[project]` 下 `version`；或包内 `__version__.py` / `__init__.py` 的 `__version__` |
| 3 | `Cargo.toml` | `[package]` 下 `version` |
| 4 | `pom.xml` | 顶层 `<version>` |

- **单一命中**：直接采用该文件。
- **多个命中（monorepo）**：列出所有命中文件 + 各自当前版本，让用户选择要更新哪个。
- **零命中**：询问用户版本文件路径与字段。

### 3. 确定新版本

当前版本匹配 `YYYY-MM-DD(.N)` 为日期方案，匹配 `x.y.z` 为 semver 方案。

- **无参数或 `date`**（日期方案，默认）→ 新版本 = 今天日期 `YYYY-MM-DD`：
  - 若当前版本已是今天（`今天` 或 `今天.N`）→ 追加/递增序号：`今天` → `今天.1`，`今天.N` → `今天.(N+1)`。
  - 若当前版本是 semver 格式 → 提示用户即将从 semver 切换到日期方案，确认后执行。
- **显式版本号** → 校验格式：匹配 `YYYY-MM-DD(.N)`（且为合法日期）或 `x.y.z` 即采用；新版本不得早于/低于当前同方案版本，否则警告确认。
- **semver 关键词**（`major` / `minor` / `patch`）→ 要求当前版本为 `x.y.z` 格式，按 semver 规则递增。
  - 若当前版本是日期方案 → 关键词不适用，提示用户改用日期方案（或显式给出 semver 版本号强制切换）。

向用户展示「旧版本 → 新版本」并确认。

### 4. 写入与提交

- 仅修改探测到文件中的 version 字段（结构化/定点替换，不改动其它内容）。
- `git add <版本文件>` —— **只暂存该版本文件**。
- **调用 git-commit-helper skill** 生成并执行提交。由于只暂存了版本文件，git-commit-helper 看到的 staged diff 即版本变更，自然生成形如 `chore: 升级版本号至 2.1.0` 的提交。

### 5. 输出

- 旧版本 → 新版本
- 改动的文件
- 生成的 commit（hash + message）
- 明确提示：**未打 tag、未 push**。如需打 tag 并更新 CHANGELOG，可接着运行 `/release-tag`。
