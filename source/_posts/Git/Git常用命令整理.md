---
title: Git常用命令整理
tags: Git
categories:
  - Git
date: 2026-04-21 09:31:32
---

# Git常用命令全面整理

本文全面整理Git日常开发中高频使用命令，按功能分类，每条命令附一句话说明，简洁易懂、可直接复制使用，覆盖95%\+日常开发场景，适合新手入门及老手快速查阅。

<!--more-->

## 一、基础配置（首次使用必配）

用于配置Git全局信息、个性化设置，仅需首次配置，后续可按需修改。

```bash
# 配置全局用户名（提交记录中显示的用户名）
git config --global user.name "你的用户名"

# 配置全局邮箱（与Git账号/代码托管平台账号绑定）
git config --global user.email "你的邮箱地址"

# 查看所有全局配置信息（验证配置是否生效）
git config --list

# 查看指定配置项（如单独查看用户名/邮箱）
git config user.name  # 查看用户名
git config user.email # 查看邮箱

# 配置默认文本编辑器（如vim，提交备注时使用）
git config --global core.editor vim

# 配置换行符自动转换（避免跨系统换行符冲突）
git config --global core.autocrlf true  # Windows系统推荐
git config --global core.autocrlf input # Mac/Linux系统推荐

# 取消某个全局配置项
git config --global --unset 配置项名（如user.email）
```

## 二、仓库操作（初始化与克隆）

用于创建本地仓库、关联远程仓库，是Git使用的基础步骤。

```bash
# 初始化本地Git仓库（在当前目录生成.git隐藏文件夹，用于存储版本信息）
git init

# 初始化本地仓库并指定目录（若目录不存在则自动创建）
git init 仓库目录名

# 克隆远程仓库到本地（完整复制远程仓库的代码、分支、提交历史）
git clone 远程仓库地址（如https://github.com/xxx/xxx.git）

# 克隆远程仓库并指定本地目录名（避免目录名与仓库名一致）
git clone 远程仓库地址 本地目录名

# 查看本地仓库的.git目录详情（隐藏目录，需开启显示隐藏文件）
ls -a .git  # Mac/Linux
dir .git    # Windows
```

## 三、工作区与暂存区操作（核心流程）

工作区（本地编辑文件的区域）、暂存区（临时存储修改的区域）、本地仓库（永久存储版本的区域）的交互，是日常提交代码的核心流程。

### 3\.1 查看状态与差异

```bash
# 查看工作区、暂存区的文件修改状态（新增/修改/删除，最常用）
git status

# 查看工作区与暂存区的文件差异（具体修改了哪些内容）
git diff

# 查看暂存区与本地仓库（当前分支最新版本）的差异
git diff --staged
git diff --cached  # 与--staged等价，兼容旧版本Git

# 查看工作区与本地仓库最新版本的差异（跳过暂存区）
git diff HEAD

# 查看两个指定文件的差异（可跨工作区/暂存区）
git diff 文件名1 文件名2
```

### 3\.2 添加到暂存区

```bash
# 添加单个文件到暂存区（指定具体文件名）
git add 文件名（如test.txt）

# 添加多个指定文件到暂存区（用空格分隔）
git add 文件名1 文件名2 文件名3

# 添加当前目录下所有修改、新增、删除的文件到暂存区（最常用）
git add .

# 添加当前目录下指定类型的文件到暂存区（如所有.md文件）
git add *.md

# 添加当前目录下所有子目录的文件到暂存区
git add -A
git add --all  # 与-A等价

# 交互式添加文件（可选择文件的部分修改内容添加到暂存区）
git add -i
```

### 3\.3 提交到本地仓库

```bash
# 提交暂存区内容到本地仓库，附带提交说明（必填，描述修改内容）
git commit -m "提交说明"（如：fix: 修复登录按钮点击无响应问题）

# 跳过暂存区，直接提交已跟踪文件的修改（未跟踪文件需先git add）
git commit -a -m "提交说明"
git commit --all -m "提交说明"  # 与-a等价

# 修改最近一次提交的说明（未推送到远程仓库时可用）
git commit --amend -m "新的提交说明"

# 合并最近多次提交（将当前分支最近n次提交合并为1次，n为数字）
git rebase -i HEAD~n

# 提交时忽略某些警告（如换行符警告，谨慎使用）
git commit --no-verify -m "提交说明"
```

## 四、分支管理（核心高频）

分支用于隔离开发功能、修复bug，避免影响主分支，是团队协作的核心操作。

### 4\.1 查看分支

```bash
# 查看本地所有分支（带*的为当前所在分支）
git branch

# 查看本地所有分支，并显示每个分支的最后一次提交记录
git branch -v

# 查看本地+远程所有分支（远程分支以remotes/origin/开头）
git branch -a

# 查看分支的详细信息（包括关联的远程分支）
git branch -vv

# 查看哪些分支已合并到当前分支
git branch --merged

# 查看哪些分支未合并到当前分支
git branch --no-merged
```

### 4\.2 创建与切换分支

```bash
# 创建新分支（基于当前分支，不切换到新分支）
git branch 分支名（如feature/login）

# 基于指定分支创建新分支（如基于main分支创建feature分支）
git branch 新分支名 目标分支名

# 切换到指定本地分支
git checkout 分支名

# 创建并直接切换到新分支（最常用，等价于git branch + git checkout）
git checkout -b 新分支名
git switch -c 新分支名  # Git 2.23+版本新增，与checkout -b等价

# 切换到上一个分支（快速切换，无需输入分支名）
git checkout -
git switch -  # Git 2.23+版本新增

# 基于远程分支创建本地分支（并关联远程分支）
git checkout -b 本地分支名 origin/远程分支名
git switch -c 本地分支名 origin/远程分支名  # Git 2.23+版本
```

### 4\.3 合并与删除分支

```bash
# 把指定分支合并到当前所在分支（如把feature分支合并到main分支）
git merge 目标分支名

# 合并分支时，强制使用快进合并（默认开启，无冲突时直接移动指针）
git merge --ff 目标分支名

# 合并分支时，禁止快进合并（强制生成一个新的合并提交记录）
git merge --no-ff -m "合并说明" 目标分支名

# 合并分支时，遇到冲突后中止合并（恢复到合并前状态）
git merge --abort

# 删除本地已合并到当前分支的分支（安全删除，未合并会提示）
git branch -d 分支名

# 强制删除本地分支（无论是否合并，谨慎使用，删除后无法恢复）
git branch -D 分支名

# 删除远程分支（本地分支删除后，同步删除远程分支）
git push origin --delete 远程分支名
git push origin :远程分支名  # 与上一条等价，兼容旧版本Git
```

### 4\.4 分支关联与同步

```bash
# 关联本地分支与远程分支（首次推送时常用，后续可直接git push/pull）
git branch --set-upstream-to=origin/远程分支名 本地分支名

# 查看本地分支与远程分支的关联关系
git branch -vv

# 取消本地分支与远程分支的关联
git branch --unset-upstream 本地分支名

# 拉取远程分支的最新代码到本地关联分支
git pull origin 远程分支名:本地分支名
git pull  # 关联后可省略分支名，默认拉取当前分支关联的远程分支
```

## 五、远程仓库操作

用于与远程代码托管平台（如GitHub、Gitee、GitLab）交互，实现代码同步、团队协作。

### 5\.1 关联与查看远程仓库

```bash
# 关联本地仓库与远程仓库（给远程仓库起别名，默认别名为origin）
git remote add origin 远程仓库地址

# 给远程仓库设置多个别名（如同时关联GitHub和Gitee）
git remote add gitee 远程仓库地址（Gitee）
git remote add github 远程仓库地址（GitHub）

# 查看所有关联的远程仓库信息（显示别名、远程地址）
git remote -v

# 查看指定远程仓库的详细信息（如查看origin的详情）
git remote show origin

# 修改远程仓库的别名（如把origin改为github）
git remote rename 旧别名 新别名

# 修改远程仓库的地址（如远程仓库地址变更后）
git remote set-url origin 新的远程仓库地址

# 删除关联的远程仓库（如删除gitee别名的远程仓库）
git remote remove gitee
git remote rm gitee  # 与上一条等价
```

### 5\.2 拉取与推送代码

```bash
# 拉取远程仓库指定分支的代码，并合并到本地当前分支（最常用）
git pull origin 远程分支名

# 拉取远程仓库代码，不自动合并（先查看差异，再手动合并）
git fetch origin 远程分支名
git fetch  # 拉取所有远程分支的最新代码

# 拉取远程分支代码，并强制覆盖本地分支（谨慎使用，会丢失本地修改）
git fetch --all
git reset --hard origin/远程分支名

# 推送本地当前分支到远程关联分支（关联后可省略分支名）
git push origin 本地分支名:远程分支名
git push  # 关联后简化写法

# 首次推送本地分支到远程仓库，并关联远程分支（后续可直接git push）
git push -u origin 本地分支名
git push --set-upstream origin 本地分支名  # 与-u等价

# 强制推送本地分支到远程仓库（覆盖远程分支，谨慎使用，团队协作时禁止随意使用）
git push -f origin 本地分支名
git push --force origin 本地分支名  # 与-f等价

# 推送本地所有分支到远程仓库
git push --all origin
```

## 六、撤销与回退（安全操作）

用于撤销工作区、暂存区的修改，或回退本地仓库的版本，避免误操作导致的代码丢失。

```bash
# 撤销工作区指定文件的修改（恢复到上次提交/暂存的状态，不丢失暂存区内容）
git checkout -- 文件名
git restore 文件名  # Git 2.23+版本新增，与checkout -- 等价

# 撤销工作区所有文件的修改（恢复到上次提交状态）
git checkout .
git restore .  # Git 2.23+版本新增

# 撤销暂存区指定文件的修改（将文件从暂存区放回工作区，不丢失修改内容）
git reset HEAD 文件名
git restore --staged 文件名  # Git 2.23+版本新增

# 撤销暂存区所有文件的修改（所有文件放回工作区）
git reset HEAD
git restore --staged .  # Git 2.23+版本新增

# 回退到上一个版本（保留工作区和暂存区的修改，仅回退本地仓库指针）
git reset --soft HEAD^
git reset --soft HEAD~1  # 与HEAD^等价，~1表示上1个版本，~n表示上n个版本

# 回退到上一个版本（保留工作区修改，清空暂存区）
git reset --mixed HEAD^  # 默认选项，可省略--mixed

# 强制回退到上一个版本（丢弃工作区、暂存区所有修改，谨慎使用）
git reset --hard HEAD^

# 回退到指定版本（版本号通过git log查看，取前7位即可）
git reset --hard 版本号（如a1b2c3d）

# 查看所有操作历史（包括回退记录，可通过此找到误删的版本号）
git reflog

# 撤销最近一次提交，并将提交的内容放回暂存区（未推送到远程时可用）
git reset --soft HEAD~1

# 撤销某次指定的提交（生成一个新的提交，抵消原提交的修改，推荐使用）
git revert 版本号（如a1b2c3d）
```

## 七、日志与历史查看

用于查看提交历史、分支合并记录，定位问题、追溯修改记录。

```bash
# 查看本地仓库完整提交日志（按时间倒序，显示提交者、时间、提交说明、版本号）
git log

# 查看简洁版提交日志（一行一条，显示版本号前7位和提交说明，最常用）
git log --oneline

# 查看提交日志，并显示分支合并图谱（直观查看分支关系）
git log --graph
git log --graph --oneline  # 简洁版图谱，更易查看

# 查看指定分支的提交日志
git log 分支名

# 查看最近n次提交日志（n为数字，如查看最近5次）
git log -n 5
git log --oneline -5  # 简洁版最近5次

# 查看提交日志，并显示每次提交修改的文件
git log --name-only

# 查看提交日志，并显示每次提交的文件修改详情（差异）
git log -p

# 查看指定作者的提交日志（筛选某个人的提交记录）
git log --author="作者名"

# 查看指定时间段的提交日志（格式：YYYY-MM-DD）
git log --since="2024-01-01" --until="2024-12-31"

# 查看所有操作历史（含回退、撤销记录，可找回误删版本）
git reflog
```

## 八、标签管理（发布版本用）

用于给重要的版本（如发布版本）打标签，方便后续追溯、回退到指定版本。

```bash
# 创建轻量标签（仅包含版本号，无额外信息）
git tag 标签名（如v1.0.0）

# 创建附注标签（包含标签说明、作者、时间，推荐使用）
git tag -a v1.0.0 -m "版本说明（如：首次正式发布）"

# 给指定版本打标签（不是当前分支的最新版本，需指定版本号）
git tag -a v1.0.0 版本号（如a1b2c3d） -m "版本说明"

# 查看所有本地标签（按字母顺序排列）
git tag

# 查看指定标签的详细信息（附注标签的说明、作者等）
git show 标签名

# 推送指定标签到远程仓库
git push origin 标签名

# 推送所有本地标签到远程仓库
git push origin --tags

# 删除本地标签
git tag -d 标签名

# 删除远程仓库的标签（先删除本地标签，再推送删除指令）
git tag -d 标签名
git push origin --delete 标签名
git push origin :refs/tags/标签名  # 与上一条等价

# 检出标签对应的版本（切换到标签指向的版本，处于 detached HEAD 状态）
git checkout 标签名

# 基于标签创建新分支（在标签版本基础上继续开发）
git checkout -b 新分支名 标签名
```

## 九、暂存管理（临时切换分支用）

用于临时保存工作区的修改，方便切换分支（无需提交代码），后续可恢复修改。

```bash
# 暂存当前工作区的所有修改（包括新增、修改、删除，不提交）
git stash

# 暂存当前工作区的修改，并添加备注（方便区分多个暂存记录）
git stash save "备注信息（如：未完成的登录功能）"

# 查看所有暂存记录（按暂存时间排序，最新的在最上面）
git stash list

# 恢复最近一次暂存的修改（恢复后，删除该暂存记录）
git stash pop

# 恢复指定的暂存记录（通过git stash list查看暂存索引，如stash@{0}）
git stash pop stash@{0}

# 恢复最近一次暂存的修改（恢复后，保留该暂存记录）
git stash apply

# 恢复指定的暂存记录（保留暂存记录）
git stash apply stash@{0}

# 删除最近一次暂存记录（不恢复修改）
git stash drop

# 删除指定的暂存记录
git stash drop stash@{0}

# 删除所有暂存记录（谨慎使用）
git stash clear

# 查看暂存记录的修改详情（如查看最近一次暂存的差异）
git stash show
git stash show -p  # 显示详细的文件修改内容
```

## 十、文件操作（删除、重命名、忽略）

```bash
# 删除工作区文件，并将删除操作添加到暂存区（后续需git commit提交）
git rm 文件名

# 删除工作区文件，但不删除本地文件（仅从Git跟踪中移除）
git rm --cached 文件名

# 重命名工作区文件，并将重命名操作添加到暂存区（后续需git commit提交）
git mv 旧文件名 新文件名

# 创建.gitignore文件（用于忽略不需要Git跟踪的文件/目录）
touch .gitignore  # Mac/Linux
echo "" > .gitignore  # Windows

# 查看.gitignore文件的内容
cat .gitignore  # Mac/Linux
type .gitignore  # Windows

# 强制添加被.gitignore忽略的文件到暂存区（特殊需求时使用）
git add -f 被忽略的文件名

# 查看.gitignore忽略的文件列表
git ls-files --others --ignored --exclude-standard
```

## 十一、高级常用命令（进阶场景）

```bash
# 查看Git版本（确认当前Git版本，部分命令需高版本支持）
git --version

# 清理工作区未跟踪的文件/目录（谨慎使用，删除后无法恢复）
git clean -f  # 删除未跟踪的文件
git clean -fd # 删除未跟踪的文件和目录

# 强制清理工作区未跟踪的文件/目录（包括.gitignore忽略的文件）
git clean -fx

# 查看Git命令的帮助文档（如查看git add的用法）
git help add
git add --help  # 与上一条等价

# 查看当前Git仓库的根目录路径
git rev-parse --show-toplevel

# 合并多个提交记录（交互式，可编辑提交信息、删除提交）
git rebase -i HEAD~n（n为要合并的提交次数）

# 查看两个分支之间的差异（如查看main分支和feature分支的差异）
git diff 分支1 分支2

# 查看指定文件在不同分支中的差异
git diff 分支1 分支2 -- 文件名

# 统计代码提交量（指定时间段、指定作者）
git log --since="2024-01-01" --until="2024-12-31" --author="作者名" --oneline | wc -l

# 查看Git配置文件的存储路径（全局配置文件）
git config --global --edit
```
