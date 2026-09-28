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
| `release-tag` | 归纳 release notes，同步写入 CHANGELOG.md 并提交，打 annotated tag | ✅ 委托 | ✅ | ❌ |

## 安装

1. 将本仓库放入 Claude Code 的插件目录，或通过你的插件管理方式安装 `git-kit`。
2. 安装后三个 skill 自动可用：`/git-commit-helper`、`/bump-version`、`/release-tag`。
3. 若你之前在 `~/.claude/skills/` 下单独装过 `git-commit-helper`，请删除该独立副本，避免同名 skill 重复发现。

## 用法

```
/bump-version           # 默认日期版本（YYYY-MM-DD），也支持 major/minor/patch 或显式版本号
/release-tag            # 生成 release notes → 写入 CHANGELOG.md 并提交 → 打 annotated tag
git push origin <分支> <tag>   # 用户手动 push
```

三个 skill 也可单独使用：只改版本跑 `/bump-version`，给已提交代码补 tag 跑 `/release-tag`，普通提交跑 `/git-commit-helper`。参数、版本文件探测、tag 前缀等细节见各自的 `skills/*/SKILL.md`。

## 设计约定

- **统一提交规范**：三个 skill 共享同一套 Conventional Commits type 约定（feat/fix/docs/style/refactor/perf/test/chore），release notes 按此分组。
- **松耦合**：三者通过「版本文件」「Conventional Commits 提交历史」两个隐式契约衔接，不互相硬依赖。
- **不自动 push、不自动创建 Release**：远程是 GitHub 时 release-tag 只给出创建 Release 的指引，用户明确要求时才代为执行。
- **CHANGELOG 与 tag 一致**：CHANGELOG.md 由 release-tag 在打 tag 时维护（不存在则自动创建），tag 描述与 CHANGELOG 条目正文逐字一致，release notes 只生成一次、两处使用。
