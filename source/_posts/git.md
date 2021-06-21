---
title: git进阶
date: 2021-06-18 09:21:06
updated: 2021-06-18 14:55:06
excerpt: git cherry-pick, git revert, git 头指针分离, git rebase交互式变基...
index_img: /img/git/base.gif
tags: git
categories: 
- 工具
---
记录自己对git的理解, 得益于此篇[推文](https://mp.weixin.qq.com/s/65XK7vpmLhFjQsBB9SBZdA), 推开了git世界的大门, 文中动图来自于此推文; 其他代码图片, 借助[carbon](https://carbon.now.sh/)生成
# detached HEAD state
头指针分离: HEAD 指针指向非最新的commit_id, 如果有需要可以创建分支以保存; 
应用场景: 个人常用于测试某些不确定性的功能, 测试可以则保存，测试不满意则丢弃
![](/img/git/head.png)
# git merge
## --no-ff
当前分支相比于我们要合并的分支有额外的提交, 会长生 merge_commit
![](/img/git/nff.gif)

## --ff
当前分支相比于我们要合并的分支没有额外的提交
![](/img/git/ff.gif)

# git rebase
git rebase -i (start_commit_id end_commit_id], start end的区间是**前开后闭**的;
**变基的前提, 要变基的分支没有合并到其他分支**
## CRUD
**更改commit/message/删除commit/修改某个commit/合并多个commit**
`git rebase -i start_commit_id end_commit_id`
- pick		use commit                    
保留此commit与commit-message
- reword	use commit but edit message   
保留此commit与commit-message但编辑commit-message
- edit		use commit but stop amending  
保留此commit与commit-message但对文件内容进行编辑 (类似`git commit --amend`)
- squash	use commit but meld into previous commit          
保留此commit与commit-message, 但把更改合并到前一个commit
- fixup		like squash but discard this commit's log commit  
保留此commit,把更改合并到前一个commit
- drop		remove commit                 
去除此commit与commit-message

## copy
**复制一段连续的commit到指定分支** `git rebase -i start_commit_id end_commit_id --onto branch_name`
![](/img/git/rebase-copy.png)
`git cherry-pick commit_id` 可以复制某个commit_id到当前分支, 多个用空格隔开
## 避免merge_commit
`git rebase branch` 把当前分支的提交复制到某分支上, 如果想保持分支的整洁线性 可以使用 git rebase, 在合并分支之前解决冲突, 可以保证不会产生merge_commit
![](/img/git/base.gif)
`git merge` 与 `git rebase` 哪个更好, 个人偏向于`git merge` 因为保留了很多细节功能迭代历史, 如果项目组管理只需要关注版本迭代的话, 确实`git rebase`更好

# git reset
## git reset --mixed
**用于重置暂存区的文件与上一次的提交(commit)保持一致，工作区文件内容保持不变**
## git reset --hard
**丢弃commit, HEAD指针指向指定commit**
![](/img/git/reset.gif)

## git reset --soft
**丢弃commit, HEAD指针指向指定commit, 但保存丢弃的commit的修改**
![](/img/git/soft-reset.gif)

# git revert
当要丢弃当前分支过去的某commit_id的时候,除了rebase-drop之外, 还可以使用`git revert`(git log会记录此操作)
曾遇到场景: 丢弃CRM分页条件查询限制
![](/img/git/revert.gif)

最后, 可以通过第三方包规范git提交格式, 比如`cz-conventional-changelog`





