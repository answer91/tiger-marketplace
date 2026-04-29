---
name: git-worktree-skill
description: 高效管理 Git worktree，支持并行开发多个分支，独立工作区，避免频繁切换。提供创建、删除、同步、列出等核心操作及常见问题处理。
---

# Skill: git-worktree-skill

`git worktree` 允许你在同一个 Git 仓库中同时检出多个分支到不同目录，每个目录拥有独立的工作区、暂存区和分支指针，共享同一个 `.git` 目录（通过符号链接或文件引用）。该技能帮助你在不中断当前工作流的情况下，并行处理多个任务（如开发新功能、修复紧急 bug、代码审查等）。

## 核心操作

### 创建worktree

**基本语法**：
```bash
git worktree add <路径> <分支>
```

**常用变体**

```bash
# 基于现有分支创建
git worktree add ../hotfix hotfix/urgent

# 新建分支并创建（相当于 checkout -b）
git worktree add -b feature/new-ui ../new-ui main

# 基于远程分支创建
git worktree add ../remote-feature origin/feature

# 基于指定 commit 创建 detached 工作树
git worktree add ../review 1a2b3c4d
```

**注意事项**
- worktree 目录优先在主仓库的 `.claude/worktrees/` 下创建；若主仓库不存在 `.claude` 目录，则在 `.worktree/` 下创建；若 `.worktree/` 也不存在，先创建 `.worktree/` 目录再创建 worktree
- 当前会话若已在 worktree 中，必须使用**主仓库绝对路径**创建，避免嵌套目录（例如：`git worktree add -b <branch> /path/to/main-repo/.claude/worktrees/<name> master`）
- 可通过 `git worktree list` 查看主仓库的绝对路径
- 同一个分支不能同时被两个工作树检出（除非使用 --force）
- 创建时会自动执行 git checkout，新目录内容与指定分支完全一致

### 删除worktree

**安全删除（工作树必须干净，无未提交变更）**

```bash
git worktree remove <路径>
```

**强制删除（丢弃未提交变更）**

```bash
git worktree remove --force <路径>
```

**手动删除目录后的清理**

```bash
rm -rf <路径>
git worktree prune   # 清理 .git/worktrees 中的残留元数据
```

**注意事项**
- 删除前确保工作树没有未提交的更改，否则会丢失数据
- 删除后需要执行 prune 清理无效的工作树引用

### 同步worktree
工作树之间不会自动同步更新，因为每个工作树有独立的工作区。同步操作本质上是将某个工作树中的提交拉取/变基到另一个工作树中。

**场景 A：将主分支的更新同步到 feature 工作树**

```bash
# 在 feature 工作树目录中
cd ../feature-worktree
git fetch origin              # 获取远程更新
git rebase origin/main        # 或者 git merge origin/main
```

**场景 B：将 feature 工作树的提交合并到主分支**

```bash
# 在主工作树中
cd /path/to/main-worktree
git fetch --all               # 获取所有工作树的更新（共享对象库）
git merge ../feature-worktree  # 直接合并另一个工作树的分支
# 或者使用普通分支名
git merge feature-branch
```

**场景 C：在多个工作树之间 cherry-pick**

```bash
# 从工作树 A 的提交哈希
cd ../worktree-B
git cherry-pick <commit-hash-from-A>
```

提示：因为所有工作树共享同一个 .git 目录，所以任何 git commit 产生的对象对所有工作树立即可见，只需执行 git fetch（或直接使用分支名）即可同步引用。

### 列出worktree list

```bash
git worktree list
```

### 其它管理操作

**锁定worktree**

```bash
# 防止被 prune 删除
git worktree lock <路径> [--reason "原因"]
```

**解锁worktree**

```bash
git worktree unlock <路径>
```

**移动worktree**

```bash
# 物理移动目录并更新引用
git worktree move <旧路径> <新路径>
```

**修复worktree**

```bash
# 修复 .git 文件损坏或路径变更问题
git worktree repair <路径>
```

### 使用场景示例

**场景 1：紧急修复生产 bug**

```bash
# 当前正在开发 feature，突然需要修复 hotfix
git worktree add ../hotfix main
cd ../hotfix
git checkout -b hotfix/bug-123
# 修复、测试、提交
git push origin hotfix/bug-123
# 删除工作树
cd .. && git worktree remove hotfix
```

**场景 2：同时运行多个构建配置**

```bash
git worktree add ../build-release v1.2.3
git worktree add ../build-debug v1.2.3
# 在两个目录中分别运行不同编译选项，互不影响
```

**场景 3：代码审查**

```bash
git worktree add ../pr-review origin/pull/123/head
cd ../pr-review
# 运行测试、审查代码
```
