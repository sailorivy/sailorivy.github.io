---
title: Git常用命令
layout: post
categories: 工具
tags: Git
comments: yes
---

<h2>Git的环境变量</h2>
Git的环境变量可以在三个地方设置：

 - /etc/gitconfig文件：对所有用户适用，使用git config命令时指定‘–system’选项。
 - ～/.gitconfig文件：只适用于当前用户，使用git config命令时指定‘–global’选项。
 - 当前项目目录中的.git/config文件：仅对当前项目有效，不用指定任何选项。

每个级别的配置都会覆盖上层的相同配置，.git/config里的配置会覆盖/etc/gitconfig中的同名变量。
<h3>用户信息</h3>
配置个人的用户名称和电子邮件。Git在每次提交时都会引用这两条信息，说明是谁提交了，这两条配置会随更新内容一起被纳入历史记录。

    $ git config [--global] user.name <user_name>
    $ git config [--global] user.email <user_email>
<h3>查看配置信息</h3>
查看已有的配置信息

    $ git config --list
查看某个环境变量的设置，直接指定该环境变量的名称就可以：

    $ git config user.name
<h2>获取帮助</h2>

    $ git help <verb>
    $ git <verb> --help
    $ man git-<verb>
<h2>常用命令</h2>
<h3>本地操作</h3>
克隆现有仓库

    $ git clone <url>
查看当前文件的状态

    $ git status
跟踪新文件/暂存已修改的文件

    $ git add <file>
查看未暂存的改动

    $ git diff
查看已暂存的改动

    $ git diff --cached
删除文件

    $ git rm <file_path>
移动/重命名文件

&nbsp;&nbsp;&nbsp;&nbsp;Git的元数据并不跟踪移动/重命名操作，但它能推断出是移动/重命名操作。

    $ git mv <file_from> <file_to>
    或者
    $ mv <file_from> <file_to>
    $ git rm <file_from>
    $ git add <file_to>
提交更新（已暂存的改动）

    $ git commit -m '<commit_log>'
提交已经跟踪过的文件的更新（跳过git add步骤，一步就暂存并提交）

    $ git commit -a -m '<commit_log>'
查看提交历史

    $ git log
&nbsp;&nbsp;&nbsp;&nbsp;使用-p选项可以显示每次提交的内容差异。

&nbsp;&nbsp;&nbsp;&nbsp;另外还有--pretty选项，可以指定不同于默认格式的方式展示提交历史。

&nbsp;&nbsp;&nbsp;&nbsp;在--pretty选项指定为oneline或format的时候，结合--graph选项可以看到开头会多出一些ASCII字符串表示的简单图形，展示每个提交所在的分支及其分化衍合情况。

&nbsp;&nbsp;&nbsp;&nbsp;还可以指定--since和--until设置时间范围，用--author选项指定作者（只显示该作者的提交）

&nbsp;&nbsp;&nbsp;&nbsp;如果只关心某些文件或目录的历史提交，可以在git log选项的最后指定路径（path）

撤销commit

    $ git reset --hard <commit_id>
取消已经暂存的文件

    $ git reset HEAD <file>
取消对文件的修改

    $ git checkout -- <file>


<h3>分支操作</h3>
新建并切换分支

    $ git branch <branch_name>
    $ git checkout <branch_name>
    或者
    $ git checkout -b <branch_name>
&nbsp;&nbsp;&nbsp;&nbsp;在切换分支之前，需要保持一个清洁的工作区域（暂存区和工作目录），那些还没有提交的修改会和即将检出的分支产生冲突从而阻止Git为你切换分支。但现实工作中，工作区域不可能一直保持清洁，解决办法见后面的“冲突处理”。

推送本地新建分支到远程仓库

    $ git push --set-upstream <remote_name> <branch_name>

推送数据到远程仓库

    $ git push <remote_name> <branch_name>
    
把本地分支推送到某个命名不同的远程分支

    $ git push <remote_name> <branch_name>:<remote_branch_name>
合并分支

    $ git checkout <target_branch_name>
    $ git merge <source_branch_name>
冲突处理

 1. 如果在合并时或git pull时发生冲突，可以用git status查阅哪些文件出现了冲突。
 2. 接着手工定位并解决这些冲突（Git会在有冲突的文件里加入标准的冲突解决标记）。
 也可以运行git mergetool，调用一个可视化的合并工具来引导你解决所有冲突。
 3. 解决了所有文件里的所有冲突后，运行git add将把它们标记为已解决状态（暂存就表示冲突已经解决）。
 4. 再运行一次git status，确认所有冲突都已解决。
 5. 用git commit完成合并提交。

删除分支

    $ git branch -d <branch_name>
列出当前所有的分支，并显示当前所在的分支

    $ git branch
查看各个分支最后一个提交对象的信息

    $ git branch -v
查看哪些分支已被并入当前分支（一般来说，列表中没有*的分支通常都可以用git branch -d删掉）

    $ git branch --merged
查看尚未合并的分支

    $ git branch --no-merged
&nbsp;&nbsp;&nbsp;&nbsp;对于尚未合并的分支，删除的时候（git branch -d <branch_name>）会提示错误，确定改动没用的话可以强制删除（git branch -D <branch_name>）。

储藏

&nbsp;&nbsp;&nbsp;&nbsp;当你正在进行某一部分工作，里面的东西比较杂乱，这时候需要转到其他分支上进行一些工作，但又不能提交进行了一半的工作。这种情况怎么办？

    $ git stash
&nbsp;&nbsp;&nbsp;&nbsp;“储藏”可以获取工作目录的中间状态（也就是修改过的被追踪的文件和暂存的变更），并将它保存到一个未完结变更的堆栈中，以便以后重新应用。

查看现有的储藏

    $ git stash list

重新应用储藏

    $ git stash apply <stash_name>
    # 不指定stash_name的话，会重新应用最新的储藏
    # 这个命令只能重新应用文件的变更，没有重新暂存已被暂存的文件
重新应用被暂存的变更

    $ git stash apply --index
从堆栈中移除储藏

&nbsp;&nbsp;&nbsp;&nbsp;stash apply只尝试应用储藏的工作，储藏的内容仍然在栈上。

    $ git stash drop <stash_name>
重新应用储藏并从堆栈中移除储藏

    $ git stash pop
<h3>忽略某些文件</h3>
有些文件不需要纳入Git的管理，比如日志或临时文件等自动生成的内容。大家应该不希望这些文件在每次git status的时候都出现在未跟踪文件列表里，也不希望它们被纳入配置管理。可以创建一个名为.gitignore的文件，列出要忽略的文件模式。

.gitignore的格式规范如下:

 - 空行或者以注释符号#开头的行会被Git忽略。
 - 可以使用标准的glob模式匹配。
glob模式是指Shell使用的简化了的正则表达式。
星号（*）匹配零个或多个任意字符；[abc]匹配任何一个列在方括号中的字符；问号（?）匹配一个任意字符；如果在方括号中使用短横线分隔两个字符，表示这两个字符范围内的都可以匹配。
 - 匹配模式最后跟斜杠（/）说明忽略的是目录。
 - 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。
 
举例来说：

    $ cat .gitignore
    # 注释，被Git忽略
    # 忽略所有.log结尾的文件
    *.log
    # 但oom.log除外
    !oom.log
    # 忽略项目根目录下的test文件，不包括api/test
    /test
    # 忽略target目录下的所有文件
    target/
    # 忽略doc目录下的doc文件，但不包括doc/manual/*.doc
    doc/*.doc
    # 忽略doc目录中的所有txt文件
    doc/**/*.txt

     




<h3>初始化新仓库</h3>
对现有的某个项目开始用Git管理，只需到此项目所在的目录执行：

    $ git init

