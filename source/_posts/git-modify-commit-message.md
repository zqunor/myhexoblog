---
title: 【Git】修改已经提交的commit内容
date: 2018-08-13 17:39:15
tags:
  - git
category:
  - 【开发工具】
---

通过 Git 进行版本管理时，对于已经提交但没有 push 的 message 信息，发现提交信息填写错误后，如何进行修改？
对于已经 push 的 message 信息如何修改？通过`git rebase -i`进行分支管理，以及重新操作已经提交的分支信息[reword,edit,squash 等]。此次用到的主要是`reword`修改已经提交的`message`信息。

<!--more-->

## 修改已经 commit 但没有 push 的 commit message

### 查看提交历史

`git log --oneline -10`

> `--onlien`的方式能够显示精简的日志信息

显示的信息[当前分支为`zzz`]：

```bash
15af769 (HEAD -> zzz) 10-15 通过模型自动写入时间戳[补充order模型隐藏字段的设置]
197fcdd 10-13 测试下14 测试下单接口, 修改程序错误
fdeb6af 10-13 一对多关系的新增操作[完成下单接口方法]
da0bd4e 10-13 订单创建[添加订单信息到order order_product表]
5ab5068 10-11 订单快照的实现
09c2116 10-10 订单快照的业务分析
8493571 10-9 下单接口说明文档补充注释
6edda7e (origin/develop) 下单接口业务模型
60b8f01 10-7 编写一个复杂的验证器
3e8375c 10-4|5|6 下单与支付流程 + 重构权限控制前置方法
```

发现提交的信息中：

- `6edda7e`的信息中没有加标题序号

- `da0bd4e`的信息中标题序号错误

- `197fcdd`的信息中标题序号和内容有误

### 通过`git rebase -i`编辑提交的历史

> `git-rebase - Reapply commits on top of another base tip` [重新应用提交到另一个基础提示之上]

在上面的日志中可以看到`6edda7e`为已经 push 的分支了，暂时不介绍这个，现在需要修改`da0bd4e`和 `197fcdd`两个提交的分支上的`message`内容。

（1）编辑最久远的需要修改的分支的前一个分支上

`git rebase -i 60b8f01`

显示的内容：

```bash
pick 6edda7e 下单接口业务模型
pick 8493571 10-9 下单接口说明文档补充注释
pick 09c2116 10-10 订单快照的业务分析
pick 5ab5068 10-11 订单快照的实现
pick da0bd4e 10-13 订单创建[添加订单信息到order order_product表]
pick fdeb6af 10-13 一对多关系的新增操作[完成下单接口方法]
pick 197fcdd 10-13 测试下14 测试下单接口, 修改程序错误
pick 15af769 10-15 通过模型自动写入时间戳[补充order模型隐藏字段的设置]
```

并且下方会提示修改建议：

```bash
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
```

我们需要操作的是重新编辑已经提交的分支记录的 message 信息，所以对应的应该是`reword`,简写为`r`。

（2）修改显示的内容，将`pick`修改为`reword` [保留提交的分支记录，但是编辑提交的信息]

```bash
r 6edda7e 下单接口业务模型
pick 8493571 10-9 下单接口说明文档补充注释
pick 09c2116 10-10 订单快照的业务分析
pick 5ab5068 10-11 订单快照的实现
r da0bd4e 10-13 订单创建[添加订单信息到order order_product表]
pick fdeb6af 10-13 一对多关系的新增操作[完成下单接口方法]
r 197fcdd 10-13 测试下14 测试下单接口, 修改程序错误
pick 15af769 10-15 通过模型自动写入时间戳[补充order模型隐藏字段的设置]
```

将需要修改的记录前的 `pick` 改为 `r`，然后`:wq`保存退出后，会**按顺序**自动进入需要编辑的提交信息框

```bash
下单接口业务模型

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Aug 8 20:08:03 2018 +0800
```

然后将第一行的提交信息修改为需要设置的信息，然后`:wq`保存退出，进入下一条需要编辑的提交记录。将全部需要修改的分支信息依次修改完成后，保存退出后会出现下面的信息，表示提交成功。

```bash
[detached HEAD 91d973f] 10-8|9 下单接口业务模型
Date: Wed Aug 8 20:08:03 2018 +0800
3 files changed, 252 insertions(+), 1 deletion(-)
create mode 100644 application/api/service/Order.php
[detached HEAD b007935] 10-12 订单创建[添加订单信息到order order_product表]
4 files changed, 179 insertions(+), 23 deletions(-)
create mode 100644 application/api/model/Order.php
create mode 100644 application/api/model/OrderProduct.php
[detached HEAD a5449fc] 10-14 测试下单接口, 修改程序错误
4 files changed, 99 insertions(+), 7 deletions(-)
Successfully rebased and updated refs/heads/develop.
```

再次执行`git log --oneline -10`命令后，即可看到分支的信息为修改后的提交信息

## 修改已经 push 的 commmit message

对于已经提交的信息的分支信息操作步骤同上，只是在推送`push`的时候需要加`--force`，强制覆盖远程分支上的提交信息。

【声明】

我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：[https://cloud.tencent.com/developer/support-plan?invite_code=89fda9dsh3d0](https://cloud.tencent.com/developer/support-plan?invite_code=89fda9dsh3d0)