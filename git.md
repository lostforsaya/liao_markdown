#git学习笔记

* See __*'git help COMMAND'*__ for more information on a specific command


创建一个git库
	
	mkdir learngit
	cd learngit
	git init

在learngit下创建一个文件，并加入git提交列表（此时还未提交）

    git add readme.txt

提交到git仓库

    git commit -m "wrote a readme file"

查看当前git状态
    
    git status（learngit目录下无文件变更没有未commit的add列表动作）
    # On branch master
    nothing to commit (working directory clean)

查看当前文件变更（对比仓库上文件）

    git diff 
    diff --git a/readme.txt b/readme.txt
    index fecff14..c36662f 100644
    --- a/readme.txt
    +++ b/readme.txt
    @@ -1,3 +1,4 @@
     Git is a version control system.
     Git is free software.
     Git is very excuete
    +Git is bad

查看当前版本号/回退到历史版本号

    git reset --hard / git reset --hard version_id

查看操作记录

    git reflog


git提交工程一般分为三个阶段:

工作区learngit -> 暂存区（add) ->  版本库(commit)


命令git checkout -- readme.txt意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：

* 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
* 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次git commit或git add时的状态。

如果要回到add之前呢

    git reset HEAD file   # HEAD表示最新的版本

	[root@node1 learngit]# git reset HEAD readme.txt
	Unstaged changes after reset:
	M	readme.txt

执行完之后暂存区的修改记录就会被去掉:

    [root@node1 learngit]# git status
    # On branch master
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   readme.txt
    #


只会留下文件的修改记录:

	git checkout -- readme.txt

用完之后就会把文件修改记录还原

删除工作区文件:

1.	决定不要它了，删完工作区的，还要删除版本库里的，git rm filename; git commit -m "remove filename"
2.	删错了，后悔了，想还原  git checkout -- filename

添加远程库

1.本地建立一个库，再与远程库关联

    mkdir mysite
    git init
    git remote add origin git@github.com:AmiyaLiao/mysite.git


