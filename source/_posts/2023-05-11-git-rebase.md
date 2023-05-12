---
title: git rebase
date: 2023-05-11 18:09:01
updated: 2023-05-11 18:09:01
tags: Git
description: about git rebase
---

----
> The `git rebase` command allows you to easily change a series of commits, modifying the history of your repository. You can reorder, edit, or squash commits together.

通过 `git rebase` 能完成：
- Edit previous commit messages
- Combine multiple commits into one
- Delete or revert commits that are no longer necessary

`git rebase` 是一个变基的过程，有些命令可以直接完成，有些需要额外的操作，`git commit -amend`，`git rebase --continue`

### Rebasing commits against a branch

----
`git rebase --interactive OTHER-BRANCH-NAME`

### Rebasing commits against a point in time

---
`git rebase --interactive commit-id`

### rebase可选的操作

---

| command | desc                                                                                                    | 
|---------|---------------------------------------------------------------------------------------------------------|
| pick    | pick commits，此时可以 delte 、reorder commits                                                                |
| reword  | 和 pick 很像，但是会停止 rebase，给你修改 commit message 的机会， 任何当前 commit 做的代码修改都不受影响                                 |
| edit    | 如果选择 edit commit，此时有机会 amend commit，也就是说，可以增加或改变当前 commit ，甚至在继续进行 rebase 时，添加更多commit或者删除当前 commit 的错误代码 |
| squash  | 把多个 commit 合并成一个，会把当前命令之上的 commit 合并到当前 squash 的 commit 上                                               |
| fixup   | 和 squash 很像                                                                                             |
| exec    | This lets you run arbitrary shell commands against a commit.                                            |

### An example of using `git rebase`

---
使用 rebase 时，会使用配置的编辑器打开一个文件描述选择要 rebase 的 commits 范围的详细信息
```text
pick 1fc6c95 Patch A
pick 6b2481b Patch B
pick dd1475d something I want to split
pick c619268 A fix for Patch B
pick fa39187 something to add to patch A
pick 4ca2acc i cant' typ goods
pick 7b36971 something to move before patch B

# Rebase 41a72e6..7b36971 onto 41a72e6
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```
- 文件列出了7个 commits，说明从 rebase 的 starting point 到 当前分支状态直接有 7 次提交
- The commits you chose to rebase are sorted in the order of the oldest changes (at the top) to the newest changes (at the bottom).
- Each line lists a command (by default, pick), the commit SHA, and the commit message.整个 rebase 过程都是围绕这些命令完成的
- After the commits, Git tells you the range of commits we're working with (41a72e6..7b36971).
- Finally, Git gives some help by telling you the commands that are available to you when rebasing commits.

### Demo 

---
```shell
mkdir demo
cd demo
git init

# previous cmmit
touch README.md
git add .
git commit -m "history commit"

# first commit
touch main.py
git add .
git commit -m "1. Patch A"

# second commit
touch utils.py
git add .
git commit -m "2. Patch B"

# third commit
touch long_long_file.py
git add .
git commit -m "3. something i want to split"

# forth commit
echo "import time" >> utils.py
git add .
git commit -m "4. A fix for Patch B"

# fifth
echo "#" >> main.py
git add .
git commit -m "5. something to add to patch A"

# sixth
touch config.py
git add .
git commit -m "5. i cant' typ goods"

# seventh
echo "remove" >> utils.py
git add .
git commit -m "7. something to move before patch B"
```
### 目标

---
1. **squash** squash 第五个 bd089b9 commit 到第一个 commit 
2. **pick** 把第七个 commit 移到第二个 commit 之前  
3. **fixup** 合并第四个 commit 到 第二个 commit    
4. **edit** 把第三个 commit 分成更小的 commit
5. **reword** 修改 bd089b9 commit message

### actions

---
要完成上面五个目的，只需合理的修改 rebase 命令
```shell
git rebase -i HEAD~7

pick 0777897 1. Patch A
pick 2d01a11 2. Patch B
pick 497320a 3. something i want to split
pick 02ba6e9 4. A fix for Patch B
pick bd089b9 5. something to add to patch A
pick f691b1d 5. i cant' typ goods
pick 792fdd1 7. something to move before patch B
```

定义 rebase 行为
```shell
pick 0777897 1. Patch A
squash bd089b9 5. something to add to patch A, reword commit message
pick 792fdd1 7. something to move before patch B
pick 2d01a11 2. Patch B
fixup 02ba6e9 4. A fix for Patch B
edit bd089b9 5. something to add to patch A
reword f691b1d 5. i cant' typ goods
```
1. This file is Git's way of saying, "Hey, here's what I'm about to do with this squash." It lists the first commit's message ("Patch A"), and the second commit's message ("something to add to patch A"). If you're happy with these commit messages, you can save the file, and close the editor. Otherwise, yo

When the editor is closed, the rebase continues:

2. Git processes the two pick commands, It also processes the fixup command, since it doesn't require any interaction,fixup merges the changes from 02ba6e9 into the commit before it, 2d01a11. Both changes will have the same commit message: "Patch B".
3. Git gets to the edit opration, stop, prints the following message to the terminal
    ```shell
   You can amend the commit now, with

        git commit --amend

    Once you are satisfied with your changes, run

        git rebase --continue
   
   此时可以对代码做任意的修改，添加 commit，然后 git commit --amend，git rebase --continue继续进行rebase
   
   ```
4. Git then gets to the reword command, 打开编辑器让你修改 commit message

### 推送 rebase 到 远程
```shell
# Don't override changes
git push origin main --force-with-lease

# Override changes
git push origin main --force
```

注意：demo 有文件冲突跑不通，要做一些修改。

**github getting started demo**