---
title: Git Hooks 简介
date: 2023-03-11 09:07:31
tags: 工具
categories: 工具
description:  Git Hooks 是那些在Git执行特定事件（如commit、push、receive等）后触发运行的脚本
---

## 概念

如同其他许多的版本控制系统一样，Git 也具有在特定事件发生之前或之后执行特定脚本代码功能（从概念上类比，就与监听事件、触发器之类的东西类似）。 **Git Hooks 就是那些在Git执行特定事件（如commit、push、receive等）后触发运行的脚本** ，挂钩是可以放置在挂钩目录中的程序，可在git执行的某些点触发动作。没有设置可执行位的钩子将被忽略。

默认情况下，hooks目录是 `$GIT_DIR/hooks`，但是可以通过 core.hooksPath 配置变量来更改（请参见 [git-config](https://git-scm.com/docs/git-config)）。


## Git Hooks 能做什么

Git Hooks是定制化的脚本程序，所以它实现的功能与相应的git动作相关,如下几个简单例子：

1. 多人开发代码语法、规范强制统一
2. commit message 格式化、是否符合某种规范
3. 如果有需要，测试用例的检测
4. 服务器代码有新的更新的时候通知所有开发成员
5. 代码提交后的项目自动打包（git receive之后） 等等...

更多的功能可以按照生产环境的需求写出来


## Git Hooks 是如何工作的

每一个使用了 git 的工程下面都有一个隐藏的 .git 文件夹。

挂钩都被存储在 .git 目录下的 hooks 子目录中，即大部分项目中的 .git/hooks。

Git 默认会放置一些脚本样本在这个目录中，除了可以作为挂钩使用，这些样本本身是可以独立使用的。所有的样本都是shell脚本，其中一些还包含了Perl的脚本。不过，任何正确命名的可执行脚本都可以正常使用 ，也可以用Ruby或Python，或其他脚本语言。

```
- .git
    | - hooks
        | - applypatch-msg.sample
        | - commit-msg.sapmle
        | - post-update.sample
        | - pre-applypatch.sample
        | - pre-commit.sample
        | - pre-push.sample
        | - pre-rebase.sample
        | - pre-commit-msg.sample
```

上述文件是 git 初始化的时候生成的默认钩子，已包含了大部分可以使用的钩子，但是 .sample 拓展名防止它们默认被执行。为了安装一个钩子，你只需要去掉 .sample 拓展名。或者你要写一个新的脚本，你只需添加一个文件名和上述匹配的新文件，去掉.sample拓展名。把一个正确命名且可执行的文件放入 Git 目录下的 hooks子目录中，可以激活该挂钩脚本，之后他一直会被 Git 调用。


## 一个简单的 Hooks 例子

使用shell 这里尝试写一个简单的钩子，安装一个prepare-commit-msg钩子。去掉脚本的.sample拓展名，在文件中加上下面这两行：

```sh
#!/bin/sh

echo "# Please include a useful commit message!" > $1
```

接下来你每次运行git commit时，你会看到默认的提交信息都被替换了。

内置的样例脚本是非常有用的参考资料，因为每个钩子传入的参数都有非常详细的说明（不同钩子不一样）。

## 常用的钩子

### 客户端钩子

客户端钩子只影响它们所在的本地仓库。有许多客户端挂钩，以下把他们分为：提交工作流挂钩、电子邮件工作流挂钩及其他客户端挂钩。

#### 提交工作流挂钩

commit操作有 4个挂钩被用来处理提交的过程，他们的触发时间顺序如下：pre-commit、prepare-commit-msg、commit-msg、post-commit

##### pre-commit

pre-commit 挂钩在键入提交信息前运行，最先触发运行的脚本。被用来检查即将提交的代码快照。例如，检查是否有东西被遗漏、运行一些自动化测试、以及检查代码规范。当从该挂钩返回非零值时，Git 放弃此次提交，但可以用git commit --no-verify来忽略。该挂钩可以被用来检查代码错误，检查代码格式规范，检查尾部空白（默认挂钩是这么做的），检查新方法（译注：程序的函数）的说明。

pre-commit 不需要任何参数，以非零值退出时将放弃整个提交。这里，我们用 “强制代码格式校验” 来说明。

##### prepare-commit-msg

prepare-commit-msg 挂钩在提交信息编辑器显示之前，默认信息被创建之后运行，它和 pre-commit 一样，以非零值退出会放弃提交。因此，可以有机会在提交作者看到默认信息前进行编辑。该挂钩接收一些选项：拥有提交信息的文件路径，提交类型。例如和提交模板配合使用，以编程的方式插入信息。提交信息模板的提示修改在上面已经看到了，现在我们来看一个更有用的脚本。在处理需要单独开来的bug时，我们通常在单独的分支上处理issue。如果你在分支名中包含了issue编号，你可以使用prepare-commit-msg钩子来自动地将它包括在那个分支的每个提交信息中。

```python
#!/usr/bin/env python

import sys, os, re
from subprocess import check_output

# 收集参数
commit_msg_filepath = sys.argv[1]
if len(sys.argv) > 2:
    commit_type = sys.argv[2]
else:
    commit_type = ''
if len(sys.argv) > 3:
    commit_hash = sys.argv[3]
else:
    commit_hash = ''

print "prepare-commit-msg: File: %s\nType: %s\nHash: %s" % (commit_msg_filepath, commit_type, commit_hash)

# 检测我们所在的分支
branch = check_output(['git', 'symbolic-ref', '--short', 'HEAD']).strip()
print "prepare-commit-msg: On branch '%s'" % branch

# 用issue编号生成提交信息
if branch.startswith('issue-'):
    print "prepare-commit-msg: Oh hey, it's an issue branch."
    result = re.match('issue-(.*)', branch)
    issue_number = result.group(1)

    with open(commit_msg_filepath, 'r+') as f:
        content = f.read()
        f.seek(0, 0)
        f.write("ISSUE-%s %s" % (issue_number, content))
```

首先，上面的prepare-commit-msg 钩子告诉你如何收集传入脚本的所有参数。接下来，它调用了git symbolic-ref --short HEAD来获取对应HEAD的分支名。如果分支名以issue-开头，它会重写提交信息文件，在第一行加上issue编号。比如你的分支名issue-224，下面的提交信息将会生成:

```txt
ISSUE-224 

# Please enter the commit message for your changes. Lines starting 
# with '#' will be ignored, and an empty message aborts the commit. 
# On branch issue-224 
# Changes to be committed: 
# modified:   test.txt
```

有一点要记住的是即使用户用-m传入提交信息，prepare-commit-msg也会运行。也就是说，上面这个脚本会自动插入ISSUE-[#]字符串，而用户无法更改。你可以检查第二个参数是否是提交类型来处理这个情况。但是，如果没有-m选项，prepare-commit-msg钩子允许用户修改生成后的提交信息。所以这个脚本的目的是为了方便，而不是推行强制的提交信息规范。如果你要这么做，你需要下面所讲的commit-msg钩子。

##### commit-msg

commit-msg钩子和prepare-commit-msg钩子很像，但它会在用户输入提交信息之后被调用。这适合用来提醒开发者他们的提交信息不符合你团队的规范。传入这个钩子唯一的参数是包含提交信息的文件名。如果它不喜欢用户输入的提交信息，它可以在原地修改这个文件（和prepare-commit-msg一样），或者它会以非零值退出，放弃这个提交。比如说，下面这个脚本确认用户没有删除prepare-commit-msg脚本自动生成的ISSUE-[#]字符串。

```python
#!/usr/bin/env python

import sys, os, re
from subprocess import check_output

# 收集参数
commit_msg_filepath = sys.argv[1]

# 检测所在的分支
branch = check_output(['git', 'symbolic-ref', '--short', 'HEAD']).strip()
print "commit-msg: On branch '%s'" % branch

# 检测提交信息，判断是否是一个issue提交
if branch.startswith('issue-'):
    print "commit-msg: Oh hey, it's an issue branch."
    result = re.match('issue-(.*)', branch)
    issue_number = result.group(1)
    required_message = "ISSUE-%s" % issue_number

    with open(commit_msg_filepath, 'r') as f:
        content = f.read()
        if not content.startswith(required_message):
            print "commit-msg: ERROR! The commit message must start with '%s'" % required_message
            sys.exit(1)
```

##### post-commit

post-commit 挂钩在整个提交过程完成后运行，他不会接收任何参数，但可以运行git log来获得最后的提交信息。总之，该挂钩是作为通知之类使用的。虽然可以用post-commit来触发本地的持续集成系统，但大多数时候你想用的是post-receive这个钩子。它运行在服务端而不是用户的本地机器，它同样在任何开发者推送代码时运行。那里更适合进行持续集成。

提交工作流的客户端挂钩脚本可以在任何工作流中使用，他们经常被用来实施某些策略，但值得注意的是，这些脚本在clone期间不会被传送。可以在服务器端实施策略来拒绝不符合某些策略的推送，但这完全取决于开发者在客户端使用这些脚本的情况。所以，这些脚本对开发者是有用的，由他们自己设置和维护，而且在任何时候都可以覆盖或修改这些脚本，后面讲如何把这部分东西也集成到开发流中。

#### E-mail工作流挂钩

有3个可用的客户端挂钩用于e-mail工作流。当运行 `git am` 命令时，会调用他们，因此，如果你没有在工作流中用到此命令，可以跳过本节。如果你通过e-mail接收由git format-patch 产生的补丁，这些挂钩也许对你有用。

首先运行的是 applypatch-msg 挂钩，他接收一个参数：包含被建议提交信息的临时文件名。如果该脚本非零退出，Git 放弃此补丁。可以使用这个脚本确认提交信息是否被正确格式化，或让脚本编辑信息以达到标准化。

下一个在git am 运行期间调用是 pre-applypatch 挂钩。该挂钩不接收参数，在补丁被运用之后运行，因此，可以被用来在提交前检查快照。你能用此脚本运行测试，检查工作树。如果有些什么遗漏，或测试没通过，脚本会以非零退出，放弃此次git am的运行，补丁不会被提交。

最后在git am运行期间调用的是post-applypatch 挂钩。你可以用他来通知一个小组或获取的补丁的作者，但无法阻止打补丁的过程。

#### 其他客户端挂钩

##### pre-rebase

pre-rebase挂钩在衍合前运行，脚本以非零退出可以中止衍合的过程。你可以使用这个挂钩来禁止衍合已经推送的提交对象，pre-rebase挂钩样本就是这么做的。该样本假定next是你定义的分支名，因此，你可能要修改样本，把next改成你定义过且稳定的分支名。

比如说，如果你想彻底禁用rebase操作，你可以使用下面的pre-rebase脚本：

```sh
#!/bin/sh

# 禁用所有rebase
echo "pre-rebase: Rebasing is dangerous. Don't do it."
exit 1
```

每次运行git rebase，你都会看到下面的信息：

```text
pre-rebase: Rebasing is dangerous. Don't do it.
The pre-rebase hook refused to rebase.
```

内置的pre-rebase.sample脚本是一个更复杂的例子。它在何时阻止rebase这方面更加智能。它会检查你当前的分支是否已经合并到了下一个分支中去（也就是主分支）。如果是的话，rebase可能会遇到问题，脚本会放弃这次rebase。

##### post-checkout

由git checkout命令调用，在完成工作区更新之后执行。该脚本由三个参数：之前HEAD指向的引用，新的HEAD指向的引用，一个用于标识此次检出是否是分支检出的值（0表示文件检出，1表示分支检出）。也可以被git clone触发调用，除非在克隆时使用参数--no-checkout。在由clone调用执行时，三个参数分别为null, 1, 1。这个脚本可以用于为自己的项目设置合适的工作区，比如自动生成文档、移动一些大型二进制文件等，也可以用于检查版本库的有效性。

最后，在 merge 命令成功执行后，post-merge 挂钩会被调用。他可以用来在 Git 无法跟踪的工作树中恢复数据，诸如权限数据。该挂钩同样能够验证在 Git 控制之外的文件是否存在，因此，当工作树改变时，你想这些文件可以被复制。

### 服务器端 Hooks
除了客户端挂钩，作为系统管理员，你还可以使用两个服务器端的挂钩对项目实施各种类型的策略。这些挂钩脚本可以在提交对象推送到服务器前被调用，也可以在推送到服务器后被调用。推送到服务器前调用的挂钩可以在任何时候以非零退出，拒绝推送，返回错误消息给客户端，还可以如你所愿设置足够复杂的推送策略。

#### pre-receive

处理来自客户端的推送（push）操作时最先执行的脚本就是 pre-receive。它从标准输入（stdin）获取被推送引用的列表；如果它退出时的返回值不是0，所有推送内容都不会被接受。利用此挂钩脚本可以实现类似保证最新的索引中不包含非fast-forward 类型的这类效果；抑或检查执行推送操作的用户拥有创建，删除或者推送的权限或者他是否对将要修改的每一个文件都有访问权限。

```python
#!/usr/bin/env python

import sys
import fileinput

# 读取用户试图更新的所有引用
for line in fileinput.input():
    print "pre-receive: Trying to push ref: %s" % line

# 放弃推送
# sys.exit(1)
```

#### post-receive

post-receive 挂钩在整个过程完结以后运行，可以用来更新其他系统服务或者通知用户。它接受与 pre-receive相同的标准输入数据。应用实例包括给某邮件列表发信，通知实时整合数据的服务器，或者更新软件项目的问题追踪系统 —— 甚至可以通过分析提交信息来决定某个问题是否应该被开启，修改或者关闭。该脚本无法组织推送进程，不过客户端在它完成运行之前将保持连接状态；所以在用它作一些消耗时间的操作之前请三思。

#### update

update脚本和pre-receive脚本十分类似。不同之处在于它会为推送者更新的每一个分支运行一次。假如推送者同时向多个分支推送内容，pre-receive 只运行一次，相比之下update 则会为每一个更新的分支运行一次。它不会从标准输入读取内容，而是接受三个参数：索引的名字（分支），推送前索引指向的内容的 SHA-1 值，以及用户试图推送内容的 SHA-1 值。如果 update 脚本以退出时返回非零值，只有相应的那一个索引会被拒绝；其余的依然会得到更新。




## 最后

参考链接：

- https://www.cnblogs.com/jiaoshou/p/12222665.html
- https://www.jianshu.com/p/f3f83872a801