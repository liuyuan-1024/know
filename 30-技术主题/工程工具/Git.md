# Git 使用指南

## 1. Git 基础概念

Git 是一个免费、开源的分布式版本控制系统，用于管理代码版本、追踪文件变化以及支持多人协作开发。

Git 是版本控制工具，GitHub、GitLab、GitCode 等是提供远程仓库管理与团队协作功能的代码托管平台。

常见代码托管平台：

- **GitHub**：国际开源代码托管平台。
- **GitLab**：支持云端托管和私有化部署的 DevOps 平台。
- **GitCode**：国内代码托管与开源协作平台。

### 1.1 Git 工作区域

Git 的基本工作区域包括：

1. **工作区（Working Directory）**：存放实际项目文件的目录。
2. **暂存区（Staging Area / Index）**：保存准备提交的文件变更。
3. **本地仓库（Local Repository）**：记录项目提交历史和版本信息。
4. **远程仓库（Remote Repository）**：存放在远程服务器上的 Git 仓库。

基本工作流程：

```text
工作区
  │
  │ git add
  ▼
暂存区
  │
  │ git commit
  ▼
本地仓库
  │
  │ git push
  ▼
远程仓库
```

从远程仓库获取代码：

```text
远程仓库
  │
  │ git fetch / git pull
  ▼
本地仓库
  │
  │ git pull（合并或变基）
  ▼
当前分支与工作区
```

其中：

- `git add`：将文件变更添加到暂存区。
- `git commit`：将暂存区内容提交到本地仓库。
- `git push`：将本地提交推送到远程仓库。
- `git fetch`：获取远程仓库更新，不自动合并到当前分支。
- `git pull`：获取远程仓库更新，并合并或变基到当前分支。

---

## 2. Git 环境配置

### 2.1 用户信息配置

```bash
# 设置全局用户名
git config --global user.name "用户名"

# 设置全局邮箱
git config --global user.email "邮箱"

# 查看全局配置
git config --global --list

# 查看当前仓库生效的配置
git config --list

# 查看配置及其来源
git config --list --show-origin
```

Git 配置具有不同作用范围：

| 配置级别 | 参数 | 作用范围 |
|---|---|---|
| 系统级 | `--system` | 当前计算机的所有用户 |
| 用户级 | `--global` | 当前用户的所有 Git 仓库 |
| 仓库级 | `--local` | 当前 Git 仓库 |

通常情况下，仓库级配置的优先级高于用户级配置，用户级配置的优先级高于系统级配置。

### 2.2 常用全局配置

```bash
# 设置新建仓库的默认分支为 main
git config --global init.defaultBranch main

# 设置 git pull 默认采用 rebase 方式
git config --global pull.rebase true

# 设置 git pull 默认采用 merge 方式
git config --global pull.rebase false

# 设置 git pull 只允许快进合并
git config --global pull.ff only

# 查看 pull.rebase 配置
git config --global --get pull.rebase
```

`git pull` 常见的整合方式：

- **merge**：通过合并方式整合远程提交，必要时生成合并提交。
- **rebase**：将本地提交重新应用到更新后的远程提交之上，使提交历史更加线性。
- **fast-forward only**：仅允许快进更新，无法快进时停止操作。

对于个人开发分支，可以根据团队规范选择 `rebase`。

对于已经与其他开发者共享的提交，应谨慎执行 `rebase`，避免改写公共提交历史。

---

## 3. Git 仓库管理

### 3.1 克隆远程仓库

```bash
# 使用 HTTPS 克隆
git clone https://github.com/用户名/仓库名.git

# 使用 SSH 克隆
git clone git@github.com:用户名/仓库名.git

# 克隆到指定目录
git clone 远程仓库地址 本地目录名

# 克隆指定分支
git clone -b 分支名 远程仓库地址
```

HTTPS 和 SSH 是 Git 访问远程仓库的两种常见方式。

- HTTPS：通常使用 443 端口，身份验证一般通过凭据管理器或访问令牌完成。
- SSH：通常使用 22 端口，通过 SSH 密钥进行身份验证。

对于已经配置 GitHub SSH 密钥的开发环境，可以使用 SSH 地址克隆仓库。

### 3.2 初始化本地仓库

适用于已有本地项目，需要将其纳入 Git 版本管理的情况。

```bash
# 初始化 Git 仓库，默认分支为 main
git init -b main

# 查看当前仓库状态
git status

# 添加当前目录下的文件变更
git add .

# 创建第一次提交
git commit -m "chore: initial commit"
```

### 3.3 关联远程仓库

首先在 GitHub、GitLab 等平台创建远程仓库。

然后执行：

```bash
# 添加远程仓库，origin 为远程仓库别名
git remote add origin 远程仓库地址

# 查看远程仓库
git remote -v

# 将本地 main 分支推送到远程仓库
git push -u origin main
```

`-u` 即 `--set-upstream`，用于建立本地分支与远程分支的跟踪关系。

建立跟踪关系后，通常可以直接执行：

```bash
git pull

git push
```

如果远程仓库已经包含 README 等初始提交，需要先处理远程与本地历史的关系，不能直接假定可以推送。

### 3.4 将已有本地仓库推送到新远程仓库

```bash
# 添加新的远程仓库
git remote add origin 远程仓库地址

# 推送 main 分支并建立跟踪关系
git push -u origin main
```

如果需要推送所有本地分支：

```bash
git push -u origin --all
```

如果需要推送所有本地标签：

```bash
git push origin --tags
```

注意：

- `--all` 推送所有本地分支，但不包括所有标签。
- `--tags` 推送所有标签。
- 仅需要同步当前开发分支时，不必推送所有分支。

### 3.5 远程仓库管理

```bash
# 查看所有远程仓库及其地址
git remote -v

# 查看指定远程仓库的详细信息
git remote show origin

# 添加远程仓库
git remote add 别名 仓库地址

# 修改远程仓库地址
git remote set-url origin 新URL

# 修改远程仓库别名
git remote rename origin 新别名

# 删除远程仓库配置
git remote remove 别名

# 获取所有远程仓库更新
git fetch --all

# 清理已不存在的远程分支引用
git fetch --prune
```

说明：

- `origin` 是远程仓库的常见默认别名，并非固定名称。
- `upstream` 常用于表示 Fork 项目的原始仓库。
- `git remote rename` 通常会同步更新相关的远程跟踪配置。

---

## 4. Git 日常开发流程

### 4.1 标准开发流程

日常开发通常按照以下顺序进行：

```text
拉取最新代码
    ↓
修改项目文件
    ↓
查看文件变更
    ↓
添加到暂存区
    ↓
提交到本地仓库
    ↓
同步远程更新
    ↓
推送到远程仓库
```

对应命令：

```bash
# 1. 拉取当前分支对应的远程更新
git pull --rebase

# 2. 修改项目文件

# 3. 查看工作区状态
git status

# 4. 查看尚未暂存的文件差异
git diff

# 5. 添加文件到暂存区
git add .

# 6. 查看暂存区差异
git diff --cached

# 7. 提交到本地仓库
git commit -m "feat: 添加新功能"

# 8. 提交前再次同步远程更新
git pull --rebase

# 9. 推送到远程仓库
git push
```

说明：

- 第一次执行 `git push` 前，需要确认本地分支已经建立远程跟踪关系。
- `git pull --rebase` 可能发生冲突，需要先解决冲突才能继续。
- 如果存在尚未提交的本地修改，`pull --rebase` 可能无法直接执行，应先提交、暂存或处理这些修改。

### 4.2 文件暂存操作

```bash
# 添加指定文件
git add 文件名

# 添加当前目录及子目录中的变更
git add .

# 添加整个工作树中的变更，包括删除操作
git add -A

# 交互式选择要暂存的修改
git add -p

# 查看工作区状态
git status

# 查看未暂存的修改
git diff

# 查看已经暂存的修改
git diff --cached

# 将文件从暂存区移除，保留工作区修改
git restore --staged 文件名
```

如果某个文件已经被 Git 跟踪，后来才添加到 `.gitignore`，需要先取消跟踪。

```bash
# 停止跟踪指定文件，保留本地文件
git rm --cached 文件名

# 停止跟踪指定目录，保留本地文件
git rm -r --cached 目录名/

# 提交变更
git commit -m "chore: stop tracking generated files"
```

`.gitignore` 只对未被 Git 跟踪的文件生效。

已经被跟踪的文件不会因为加入 `.gitignore` 而自动停止跟踪。

### 4.3 提交代码

```bash
# 提交暂存区内容
git commit -m "feat: 添加新功能"

# 添加提交标题和详细说明
git commit -m "feat: 添加新功能" -m "提交信息正文"

# 修改最近一次提交信息
git commit --amend -m "新的提交信息"
```

建议使用 Conventional Commits 风格：

```text
<type>(<scope>): <description>
```

示例：

```text
feat(auth): 添加用户身份验证
fix(api): 修复接口参数校验问题
docs(readme): 更新项目使用说明
refactor(core): 重构核心业务逻辑
perf(cache): 优化缓存查询性能
```

常见提交类型：

| 类型 | 说明 |
|---|---|
| feat | 新增功能 |
| fix | 修复缺陷 |
| docs | 修改文档 |
| style | 调整代码格式，不改变代码逻辑 |
| refactor | 重构代码，不新增功能或修复缺陷 |
| perf | 性能优化 |
| test | 添加或修改测试 |
| build | 修改构建系统或外部依赖 |
| ci | 修改持续集成配置 |
| chore | 日常维护及其他辅助性修改 |
| revert | 回滚已有提交 |

提交规范建议：

- 标题简洁，明确描述本次修改。
- 一次提交尽量只包含一个相对独立的逻辑变更。
- 复杂修改可以通过提交正文补充背景和实现细节。
- 避免将密钥、密码、访问令牌等敏感信息提交到仓库。

### 4.4 拉取远程代码

```bash
# 拉取当前分支跟踪的远程分支
git pull

# 指定远程仓库及分支
git pull origin main

# 使用 merge 方式整合远程代码
git pull --no-rebase origin main

# 使用 rebase 方式整合远程代码
git pull --rebase origin main

# 仅获取远程更新，不合并
git fetch origin

# 获取所有远程仓库更新
git fetch --all
```

`git fetch` 和 `git pull` 的主要区别：

| 命令 | 获取远程更新 | 更新当前分支 |
|---|---|---|
| git fetch | 是 | 否 |
| git pull | 是 | 是 |

`git fetch` 适合在整合远程代码前检查更新情况。

例如：

```bash
# 获取最新远程更新
git fetch origin

# 查看远程 main 分支的提交记录
git log --oneline origin/main

# 查看当前分支与远程分支的差异
git log --oneline HEAD..origin/main
```

#### 处理无关提交历史

如果执行 `git pull` 时出现：

```text
fatal: refusing to merge unrelated histories
```

表示 Git 检测到两个分支不存在共同的提交祖先。

在确认确实需要合并两个独立仓库的历史后，可以执行：

```bash
git pull origin main --allow-unrelated-histories --no-rebase
```

注意：

- 该参数允许合并没有共同祖先的提交历史。
- 该命令并非强制覆盖远程代码。
- 两个独立项目合并时可能产生大量文件冲突。
- 应先确认远程仓库地址和初始化过程是否正确。

### 4.5 推送代码

```bash
# 将当前分支推送到其跟踪的远程分支
git push

# 推送本地 main 分支到 origin 的 main 分支
git push origin main

# 首次推送并建立跟踪关系
git push -u origin main

# 推送所有本地分支
git push origin --all

# 推送所有本地标签
git push origin --tags
```

如果远程分支包含本地没有的提交，通常需要先拉取并整合远程更新，再执行推送。

---

## 5. Git 分支管理

Git 分支用于支持多个开发任务并行进行。

通过不同分支，可以在不直接影响主分支的情况下开发新功能、修复缺陷或开展实验。

### 5.1 查看分支

```bash
# 查看本地分支
git branch

# 查看远程跟踪分支
git branch -r

# 查看所有本地和远程跟踪分支
git branch -a

# 查看本地分支及其跟踪关系
git branch -vv

# 查看当前分支名称
git branch --show-current
```

### 5.2 创建与切换分支

推荐使用 `git switch`：

```bash
# 创建新分支
git branch dev

# 切换到指定分支
git switch dev

# 创建并切换到新分支
git switch -c dev

# 切换到 main 分支
git switch main
```

传统 `checkout` 写法：

```bash
# 创建并切换分支
git checkout -b dev

# 切换到指定分支
git checkout dev
```

`git switch` 主要用于分支切换，语义更加明确。

`git checkout` 同时支持分支切换和部分文件恢复操作。

### 5.3 重命名分支

```bash
# 修改指定本地分支名称
git branch -m 旧分支名 新分支名

# 修改当前分支名称
git branch -m 新分支名
```

如果需要重命名已经推送的远程分支，通常需要：

1. 重命名本地分支。
2. 推送新分支。
3. 更新远程跟踪关系。
4. 根据需要删除旧远程分支。

### 5.4 删除分支

```bash
# 安全删除已经合并的本地分支
git branch -d 分支名

# 强制删除本地分支
git branch -D 分支名

# 删除远程分支
git push origin --delete 分支名
```

`-d` 会执行合并状态检查，`-D` 会跳过该检查，应谨慎使用。

### 5.5 远程分支跟踪关系

```bash
# 获取远程仓库最新信息
git fetch --all

# 查看所有远程跟踪分支
git branch -r

# 查看本地分支跟踪关系
git branch -vv

# 为当前分支设置远程跟踪分支
git branch --set-upstream-to=origin/dev

# 简写
git branch -u origin/dev
```

建立跟踪关系后，可以简化日常的 `git pull` 和 `git push` 操作。

### 5.6 合并分支

以将 `dev` 分支合并到 `main` 为例：

```bash
# 1. 切换到目标分支
git switch main

# 2. 拉取目标分支最新代码
git pull origin main

# 3. 合并 dev 分支
git merge dev

# 4. 如果发生冲突，解决冲突后暂存相关文件
git add <冲突文件>

# 5. 如果存在尚未完成的合并，完成合并提交
git merge --continue

# 6. 推送更新后的 main 分支
git push origin main
```

说明：

- 无冲突时，Git 通常会自动完成合并，不需要额外提交。
- 有冲突时，需要先解决冲突，再执行 `git add` 和 `git merge --continue`。
- 如果不希望继续合并，可以执行 `git merge --abort`。

### 5.7 分支合并冲突

当两个分支对相同文件的相同或重叠内容进行了不同修改时，Git 可能无法自动完成合并。

冲突文件通常包含以下标记：

```text
<<<<<<< HEAD
当前分支中的内容
=======
待合并分支中的内容
>>>>>>> dev
```

其中：

- `<<<<<<< HEAD`：当前分支内容的起始标记。
- `=======`：两个版本内容的分隔标记。
- `>>>>>>> dev`：待合并分支内容的结束标记。

解决冲突时，需要：

1. 打开冲突文件。
2. 确认应保留的代码内容。
3. 手动修改文件。
4. 删除冲突标记。
5. 将解决后的文件加入暂存区。
6. 完成合并操作。

```bash
# 查看冲突状态
git status

# 将已解决冲突的文件加入暂存区
git add 文件名

# 完成合并
git merge --continue
```

取消当前合并：

```bash
git merge --abort
```

### 5.8 Rebase 操作

`rebase` 用于将当前分支的提交重新应用到另一个分支的最新提交之后。

示例：

```bash
# 切换到开发分支
git switch dev

# 获取远程更新
git fetch origin

# 将 dev 分支变基到最新的 origin/main
git rebase origin/main
```

发生冲突时：

```bash
# 手动解决冲突后
git add <冲突文件>

# 继续 rebase
git rebase --continue
```

取消 rebase：

```bash
git rebase --abort
```

注意：

- `merge` 通常保留原有提交历史。
- `rebase` 会重新生成被重放的提交。
- 已经推送并被其他开发者使用的公共分支，应谨慎执行 `rebase`。

---

## 6. Git 版本查看与回退

### 6.1 查看提交历史

```bash
# 查看当前分支提交历史
git log

# 简洁查看提交历史
git log --oneline

# 图形化查看提交历史
git log --oneline --graph --all

# 查看指定分支提交历史
git log origin/main

# 查看最近 5 次提交
git log -5

# 查看指定提交详情
git show commit-hash
```

### 6.2 恢复文件修改

```bash
# 丢弃指定文件尚未暂存的修改
git restore 文件名

# 从暂存区移除文件，保留工作区修改
git restore --staged 文件名

# 同时恢复暂存区和工作区中的指定文件
git restore --source=HEAD --staged --worktree 文件名
```

注意：丢弃工作区修改通常无法直接通过 Git 恢复，应确认修改不再需要。

### 6.3 重置提交

```bash
# 重置到指定提交，并丢弃相关已跟踪文件的本地修改
git reset --hard commit-hash

# 获取远程最新状态
git fetch origin

# 将当前分支重置到远程 main 分支的状态
git reset --hard origin/main
```

注意：

- `git reset --hard` 会丢弃相关未提交的修改。
- `git reset --hard origin/main` 可能使本地独有提交不再被当前分支引用。
- 这些命令不会自动删除所有未跟踪文件。
- 执行前应确认是否需要备份本地代码。

### 6.4 撤销已有提交

如果需要撤销已经推送到公共分支的提交，通常推荐使用：

```bash
git revert commit-hash
```

`git revert` 会生成一个新的提交，用于撤销指定提交的变更。

与 `git reset` 不同，`git revert` 不直接改写已有提交历史，通常更适合团队协作场景。

---

## 7. Git 团队协作

### 7.1 团队内部协作

团队内部通常通过远程仓库的访问权限管理实现多人协作。

以 GitHub 为例：

1. 仓库管理员邀请协作者。
2. 协作者接受邀请。
3. 协作者克隆远程仓库。
4. 在开发分支完成修改。
5. 推送分支并创建 Pull Request。
6. 经过代码审查后合并到目标分支。

协作者的具体权限由仓库或组织权限配置决定，并非所有协作者都可以直接推送到受保护分支。

### 7.2 跨团队协作（Fork + Pull Request）

对于没有原仓库写入权限的开发者，通常使用 Fork 工作流。

基本过程：

1. Fork 原始仓库到个人账号。
2. 克隆个人 Fork 仓库。
3. 添加原始仓库作为 `upstream`。
4. 同步上游代码。
5. 在开发分支修改代码。
6. 推送到个人 Fork。
7. 向原始仓库发起 Pull Request。

远程仓库关系：

```text
原始仓库（upstream）
        │
        │ git fetch upstream
        ▼
     本地仓库
        │
        │ git push origin
        ▼
个人 Fork（origin）
        │
        │ Pull Request
        ▼
     原始仓库
```

配置示例：

```bash
# 克隆个人 Fork
git clone git@github.com:用户名/项目名.git

# 进入项目目录
cd 项目名

# 添加原始仓库
git remote add upstream git@github.com:原作者/项目名.git

# 查看远程仓库
git remote -v

# 获取原始仓库更新
git fetch upstream

# 切换到本地 main 分支
git switch main

# 将本地 main 快进到上游 main
git merge --ff-only upstream/main

# 将更新后的 main 推送到个人 Fork
git push origin main
```

实际开发时，通常在单独的功能分支上提交修改，再通过 Pull Request 申请合并。

---

## 8. GitHub SSH 配置

GitHub 支持通过 SSH 密钥进行 Git 身份验证。

配置 SSH 后，使用 SSH 远程地址通常不需要每次输入 GitHub 账号凭据。

### 8.1 生成 SSH 密钥

在 macOS、Linux 或 Windows Git Bash 中执行：

```bash
ssh-keygen -t ed25519 -C "GitHub邮箱"
```

根据提示选择密钥保存位置，并按需设置密钥口令。

默认密钥位置：

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

其中：

- `id_ed25519`：私钥，不能公开。
- `id_ed25519.pub`：公钥，可以添加到 GitHub。

如果当前环境不支持 Ed25519，可以使用 RSA 密钥。

### 8.2 将公钥添加到 GitHub

查看公钥：

```bash
cat ~/.ssh/id_ed25519.pub
```

macOS 可以直接复制到剪贴板：

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

进入 GitHub：

```text
Settings
    ↓
SSH and GPG keys
    ↓
New SSH key
```

添加生成的公钥。

### 8.3 测试 SSH 连接

```bash
ssh -T git@github.com
```

首次连接时，需要确认 GitHub 主机密钥指纹。

连接成功后，通常会出现类似信息：

```text
Hi 用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

说明 SSH 身份验证成功。

### 8.4 将远程仓库从 HTTPS 修改为 SSH

查看当前远程地址：

```bash
git remote -v
```

修改远程地址：

```bash
git remote set-url origin git@github.com:用户名/仓库名.git
```

验证连接：

```bash
git fetch origin
```

修改远程地址不会改变本地代码、分支和提交历史。

---

## 9. 自动将 GitHub HTTPS 地址转换为 SSH

### 9.1 配置

```bash
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

### 9.2 作用

- 使用 HTTPS 地址克隆 GitHub 仓库时，Git 自动将其转换为 SSH 地址进行连接。
- 已有项目即使配置的是 HTTPS 远程地址，Git 也会自动通过 SSH 连接。
- 配置对当前用户的所有 Git 仓库生效，无需逐个修改远程地址。
- 不会修改 `.git/config` 中保存的原始远程地址，仅在 Git 使用地址时进行转换。

### 9.3 示例

原始地址：

```text
https://github.com/deepseek-ai/deepseek-harness.git
```

实际使用的地址：

```text
git@github.com:deepseek-ai/deepseek-harness.git
```

### 9.4 查看配置

```bash
# 查看 URL 重写配置
git config --global --get-regexp 'url\..*\.insteadof'

# 查看 Git 解析后的远程地址
git remote get-url origin

# 查看 Git 配置中保存的原始地址
git config --get remote.origin.url
```

### 9.5 取消配置

```bash
git config --global --unset url."git@github.com:".insteadOf
```

### 9.6 注意事项

- 需要提前配置 GitHub SSH 密钥，并确保 SSH 连接正常。
- 该配置只匹配以 `https://github.com/` 开头的 Git URL。
- SSH 身份验证成功不代表拥有远程仓库的推送权限。
- 对于仅支持 HTTPS 的特殊网络环境，不适合启用此配置。
- 如果存在多个 URL 重写规则，Git 会按照匹配规则选择对应的替换地址。

---

## 10. Git 常见问题与注意事项

### 10.1 HTTPS 无法连接 GitHub

常见报错：

```text
fatal: unable to access 'https://github.com/...':
Failed to connect to github.com port 443
```

可能原因：

- HTTPS 网络连接异常。
- Git 代理配置错误。
- 网络代理或 VPN 分流规则异常。
- DNS 或网络路由问题。

查看当前仓库远程地址：

```bash
git remote -v
```

查看 Git 代理配置：

```bash
git config --show-origin --get-regexp 'proxy|insteadof'
```

查看环境代理变量：

```bash
env | grep -i proxy
```

测试 HTTPS 连接：

```bash
git ls-remote https://github.com/zed-industries/zed.git HEAD
```

如果 SSH 可以正常访问 GitHub，可以修改远程地址：

```bash
git remote set-url origin git@github.com:zed-industries/zed.git
```

或者设置全局 HTTPS 到 SSH 的自动转换规则：

```bash
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

### 10.2 Git 提示远程分支领先

常见报错：

```text
! [rejected] main -> main (non-fast-forward)
```

通常表示远程分支包含本地尚未整合的提交。

可以先执行：

```bash
git pull --rebase origin main
```

解决可能出现的冲突后，再推送：

```bash
git push origin main
```

不应为了绕过普通的非快进错误而直接强制推送。

### 10.3 Git 无法识别用户身份

常见报错：

```text
Author identity unknown
```

设置用户名和邮箱：

```bash
git config --global user.name "用户名"
git config --global user.email "邮箱"
```

### 10.4 本地修改阻止拉取

常见报错：

```text
Your local changes would be overwritten
```

表示当前工作区存在可能被拉取操作覆盖的本地修改。

可以选择：

- 提交当前修改。
- 使用 `git stash` 临时保存修改。
- 确认不再需要后丢弃修改。

临时保存示例：

```bash
# 保存当前已跟踪文件的修改
git stash push -m "临时保存修改"

# 拉取远程更新
git pull --rebase

# 恢复临时保存的修改
git stash pop
```

`git stash pop` 也可能发生冲突，需要根据实际情况解决。

如果还需要暂存未跟踪文件，可以使用：

```bash
git stash push -u -m "临时保存修改"
```

### 10.5 强制推送与镜像推送

以下命令具有较高风险：

```bash
# 强制更新远程分支
git push --force

# 镜像推送所有引用
git push --mirror
```

注意：

- `git push --force` 可能覆盖其他开发者的提交。
- `git push --mirror` 会镜像本地引用，可能强制更新或删除远程引用。
- 日常开发和普通代码同步不应使用 `--mirror`。
- 强制推送之前应确认远程分支状态及团队协作情况。

对于确实需要在 rebase 后强制更新个人开发分支的情况，通常优先考虑：

```bash
git push --force-with-lease
```

该命令会进行额外的远程引用状态检查，但仍可能改写远程分支历史，应谨慎使用。