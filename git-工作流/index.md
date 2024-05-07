# Git 工作流

## Git flow

Git 版本管理同样需要操作规范。荷兰程序员 Vincent 在 2010 年提出的方案被称为 git flow，如下图所示：

![git_flow](/git_flow/git_flow.png)

但大部分观点认为，以上流程只适用于作者所说的需要多版本并存的情况，即开发团队要同时维护多个有客户使用的版本。对于小型团队则比较复杂。与此类似的还有 Trunk-Based、Aone Flow 和最简单的 github flow 等。

## 通用工作流

这里考虑最简单的情况，熟悉以下流程，其他类型的 git flow 基本都能理解。假设要修改的仓库分支是 master：

1. clone 要完成的远程分支到本地，新建并切换到分支 `git switch -c feature-branch`。
2. 本地编辑后使用 `git diff` 查看修改，并使用 `git add <changed_file>` 提交修改到本地暂存区 (stage)。不建议直接用 `git add .` 的方式，因为一次性添加所有修改会造成不同文件的修改带有混乱的 comment。
3. 使用 `git commit -m "comment"` 提交修改到本地，此时本地新增一个提交记录。
4. 使用 `git push origin feature-branch` 提交到远程仓库。
5. 假设此时 master 仓库有更新，这时需要把远程的修改同步到本地验证。
   1. 先 switch 到本地 master 分支
   2. 运行 `git pull origin master` 同步远程 master 分支的更新
   3. 切回本地 feature-branch，运行 `git rebase master`，如果有冲突需要手动解决。
   4. 提交合并 master 修改后的代码到 remote，使用 `git push -f origin feature-branch`
6. 现在将 feature-branch 的代码同步到 master 中，进行 pull request。
   1. 负责同步 feature-branch 代码的管理者在 master 分支上进行 `squash and merge` 将多个连续提交合并为一个
   2. 删除远程的 feature-branch 分支
7. **确定无误后**，删除本地 feature-branch 分支 `git branch -D feature-branch`。
8. 最后切到 master 分支同步 `git pull origin master`。

## 各阶段命令常用汇总

![git-flow-commands](/git_flow/git-flow-commands-without-flow.png)

## 撤销操作

![git_revocation](/git_flow/git_revocation.png)

## merge / rebase

- Merge 操作会将两个分支的修改合并为一个新的提交，这个提交有两个父节点，分别来自于要合并的两个分支。适合用于合并公共分支、团队协作，以及当你希望保留每个分支的完整历史记录时。
- Rebase 操作会将当前分支的提交“挪动”到另一个分支的最新提交之后，形成一条直线的提交历史。它会改写提交的历史记录。适合用于保持干净的提交历史、清理提交记录、以及消除分支之间的分叉，使得整个提交历史更加线性清晰。

![merge_rebase](/git_flow/m_r.jpg)

## git 配置文件

git 的配置文件 Linux 下在 `~/.gitconfig`，Windows 下一般在 `C:\Users\username\.gitconfig`。命令行执行的有关用户名和邮箱的信息也被保存在这个位置。另外最常用的操作是设置代理，方便 clone 和 pull github 上的项目源码。通过在终端运行下面的命令完成：

```shell
git config --global url."https://github.moeyy.xyz/https://github.com".insteadOf "https://github.com"
```

url 后面的是代理地址。或者直接修改 `.gitconfig` 文件中的对应位置。

```text
[url "https://mirror.ghproxy.com/https://github.com"]
	insteadOf = https://github.com
```

## github

github 需要通过访问令牌验证用户，在个人设置的 `settings --> Developer Settings --> Personal access tokens` 中。生成令牌后用其代替密码输入，或者 clone 仓库后修改仓库的远端地址：

```bash
git remote set-url origin https://${token_seq}@github.com/kali20177/hugo-blog-src.git
```

## 参考

1. [如何撤销 Git 操作？](https://ruanyifeng.com/blog/2019/12/git-undo.html)
2. [Git Flow](https://www.twle.cn/l/yufei/git/git-basic-git-flow.html)
3. [Gitlab 基本原理](https://changsiyuan.github.io/2015/04/26/2015-4-26-gitlab-explain/)
4. [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
5. [十分钟学会正确的 github 工作流，和开源作者们使用同一套流程](https://www.bilibili.com/video/BV19e4y1q7JJ)
6. [git rebase VS git merge？更优雅的 git 合并方式值得拥有](https://www.cnblogs.com/FraserYu/p/11192840.html)
