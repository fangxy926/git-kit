# git-kit

版本号管理与 Git 发版三件套，打包为一个 Claude Code 插件。三个 skill 职责单一、可独立使用，也可组合成发版流水线：

```
git-commit-helper  →  bump-version  →  release-tag
   (怎么提交)          (改版本号+提交)    (打 tag + release notes)
```

三者均为**全局通用**，不绑定任何具体项目，且**默认不 push**——发版属于敏感的外向操作，push 留给用户手动确认。

## 包含的 skill

| Skill | 职责 | 动 commit | 动 tag | 会 push |
|---|---|---|---|---|
| `git-commit-helper` | 分析 diff，生成并执行 Conventional Commits 风格的中文提交 | ✅ 本身 | ❌ | ❌ |
| `bump-version` | 更新版本号，提交委托 git-commit-helper | ✅ 委托 | ❌ | ❌ |
| `release-tag` | 读提交历史归纳 release notes，打 annotated tag | ❌ | ✅ | ❌ |

## 安装

1. 将本仓库放入 Claude Code 的插件目录，或通过你的插件管理方式安装 `git-kit`。
2. 安装后三个 skill 自动可用：`/git-commit-helper`、`/bump-version`、`/release-tag`。
3. 若你之前在 `~/.claude/skills/` 下单独装过 `git-commit-helper`，请删除该独立副本，避免同名 skill 重复发现。

## 用法

### 典型发版流水线

```
/bump-version patch     # 1.2.3 → 1.2.4，只改版本文件并提交（chore: 升级版本号至 1.2.4）
/release-tag            # 默认用版本文件当前版本打 annotated tag + 生成 release notes
git push origin <分支> <tag>   # 用户手动 push
```

### 独立用法

- **只改版本不发版**：只跑 `/bump-version`。
- **给已提交代码补 tag**：只跑 `/release-tag`。
- **普通提交**：只跑 `/git-commit-helper`。

### bump-version 调用形式

- `/bump-version patch` —— 语义化关键词：`major` / `minor` / `patch`
- `/bump-version 2.1.0` —— 显式完整版本号
- `/bump-version` —— 无参数，交互询问

支持的版本文件（按优先级探测）：`package.json` → `pyproject.toml`（含包内 `__version__`）→ `Cargo.toml` → `pom.xml`。

### release-tag 调用形式

- `/release-tag` —— 默认读版本文件当前版本作为 tag 名
- `/release-tag 2.1.0` —— 显式指定 tag 名

tag 默认无 `v` 前缀；若仓库历史 tag 均带 `v` 则自动跟随。

## 设计约定

- **统一提交规范**：三个 skill 共享同一套 Conventional Commits type 约定（feat/fix/docs/style/refactor/perf/test/chore），release notes 按此分组。
- **松耦合**：三者通过「版本文件」「Conventional Commits 提交历史」两个隐式契约衔接，不互相硬依赖。
- **非目标**：不自动 push；不集成 GitHub/GitLab Release 发布；不维护 CHANGELOG 落盘文件（release notes 仅写入 tag）。
