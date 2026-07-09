---
name: bump-version
description: |
  更新项目版本号、生成 CHANGELOG.md 并提交（提交委托 git-commit-helper），不打 tag、不 push。
  Use when: (1) 用户想升级/更新版本号, (2) 用户说 "bump version"、"/bump-version"、"升版本",
  (3) 发版前需要先改版本号。支持日期版本（YYYY-MM-DD，默认首选）和传统 semver（major/minor/patch 关键词或显式版本号如 2.1.0）。
---

# Bump Version

更新项目版本号，分析与上一版本的差异并更新 CHANGELOG.md，最后通过 git-commit-helper 提交。**只改版本和 changelog、只提交，不打 tag、不 push。**

## 版本方案

支持两种版本号方案，**优先使用日期方案**：

1. **日期方案（首选）**：`YYYY-MM-DD`，如 `2026-07-09`；同一天多次 bump 时追加序号：`2026-07-09.1`、`2026-07-09.2`……
2. **传统 semver 方案**：`x.y.z`，按 major/minor/patch 递增。

根据当前版本号自动识别所属方案：匹配 `YYYY-MM-DD(.N)` 为日期方案，匹配 `x.y.z` 为 semver 方案。

## 调用形式

- `/bump-version` —— 无参数，**默认走日期方案**：新版本 = 今天日期
- `/bump-version date` —— 显式指定日期方案，新版本 = 今天日期
- `/bump-version 2026-07-09` —— 显式日期版本号
- `/bump-version patch` —— semver 关键词：`major` / `minor` / `patch`
- `/bump-version 2.1.0` —— 显式 semver 版本号

## 工作流程

### 1. 前置检查

- 运行 `git rev-parse --is-inside-work-tree`；非 git 仓库 → 中止并提示用户先 `git init`。
- 运行 `git status --porcelain`；若有未提交改动 → 警告并询问是否继续（避免把无关改动混入版本提交）。用户确认后才继续。

### 2. 探测版本文件（方案 A：内置固定文件类型探测器）

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

- **无参数或 `date`**（日期方案，默认）→ 新版本 = 今天日期 `YYYY-MM-DD`：
  - 若当前版本已是今天（`今天` 或 `今天.N`）→ 追加/递增序号：`今天` → `今天.1`，`今天.N` → `今天.(N+1)`。
  - 若当前版本是 semver 格式 → 提示用户即将从 semver 切换到日期方案，确认后执行。
- **显式版本号** → 校验格式：匹配 `YYYY-MM-DD(.N)`（且为合法日期）或 `x.y.z` 即采用；新版本不得早于/低于当前同方案版本，否则警告确认。
- **semver 关键词**（`major` / `minor` / `patch`）→ 要求当前版本为 `x.y.z` 格式，按 semver 计算：
  - `major`: `X.y.z` → `(X+1).0.0`
  - `minor`: `x.Y.z` → `x.(Y+1).0`
  - `patch`: `x.y.Z` → `x.y.(Z+1)`
  - 若当前版本是日期方案 → 关键词不适用，提示用户改用日期方案（或显式给出 semver 版本号强制切换）。

向用户展示「旧版本 → 新版本」并确认。

### 4. 分析与上一版本的差异，更新 CHANGELOG.md

**确定对比起点**（按优先级取第一个可用的）：

1. 上一个 tag：`git describe --tags --abbrev=0`
2. 无 tag → 上一次修改版本文件的提交：`git log -1 --format=%H -- <版本文件>`（即上一次 bump 提交）
3. 都没有 → 取全部提交历史

**收集差异**：

- `git log <起点>..HEAD --pretty=format:"%s"`（无起点时用 `git log --pretty=format:"%s"`）。
- 按 Conventional Commits 前缀归类，归纳成**精炼中文**要点（分组标题与 release-tag 保持一致）：

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
- 若区间内没有任何提交（如刚初始化的仓库），条目正文写「本次仅更新版本号」。

**写入 CHANGELOG.md（项目根目录）**：

- 文件不存在 → **主动创建**，以 `# Changelog` 作为一级标题开头。
- 新条目插入在 `# Changelog` 标题之后、所有旧条目之前（最新版本在最上面）。
- 条目标题：semver 方案用 `## [x.y.z] - YYYY-MM-DD`；日期方案版本号本身就是日期，只写 `## [2026-07-09]`，避免重复。
- 完整示例（semver 方案）：

```markdown
## [<新版本>] - <YYYY-MM-DD>

### ✨ 新功能

- 要点一
- 要点二

### 🐛 修复

- 要点三
```

- 不改动已有的历史条目；若 CHANGELOG.md 中已存在同版本号条目，询问用户是覆盖还是追加。
- 写入后向用户展示新条目内容确认。

### 5. 写入版本号与提交

- 仅修改探测到文件中的 version 字段（结构化/定点替换，不改动其它内容）。
- `git add <版本文件> CHANGELOG.md` —— **只暂存版本文件和 CHANGELOG.md**。
- **调用 git-commit-helper skill** 生成并执行提交。由于只暂存了这两个文件，git-commit-helper 看到的 staged diff 即版本变更，自然生成形如 `chore: 升级版本号至 2.1.0` 的提交。

### 6. 输出

- 旧版本 → 新版本
- 改动的文件（版本文件 + CHANGELOG.md）
- CHANGELOG 新条目摘要
- 生成的 commit（hash + message）
- 明确提示：**未打 tag、未 push**。如需打 tag，可接着运行 `/release-tag`。
