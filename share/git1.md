###3、Git 内部原理 - Git References
---
####3、1 References

####3、2 HEAD
现在的问题是，当你执行 git branch (分支名称) 这条命令的时候，Git 怎么知道最后一次提交的 SHA-1 值呢？答案就是 HEAD 文件。HEAD 文件是一个指向你当前所在分支的引用标识符。这样的引用标识符——它看起来并不像一个普通的引用——其实并不包含 SHA-1 值，而是一个指向另外一个引用的指针

###4、时光机穿梭
---
####4.1、git status
git status后出现：
- Changes to be committed:
  	(use "git reset HEAD <file>..." to unstage)



####4.2、版本回退
1. 在实际工作中，我们脑子里怎么可能记得一个几千行的文件每次都改了什么内容，不然要版本控制系统干什么。版本控制系统肯定有某个命令可以告诉我们历史记录，在Git中，我们用git log命令查
2. 首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，也就是最新的提交3628164...882e1e0（注意我的提交ID和你的肯定不一样），上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。
3. 现在，我们要把当前版本“append GPL”回退到上一个版本“add distributed”，就可以使用git reset命令：

####4.5、撤销修改
- 你可以发现，Git会告诉你，git checkout -- file可以丢弃工作区的修改
- Git同样告诉我们，用命令git reset HEAD file可以把暂存区的修改撤销掉（unstage），重新放回工作区

###6、分支管理
---
####6.1、创建与合并分支

####6.2、解决冲突

####6.3、分支管理策略
通常，合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。
如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。
下面我们实战一下--no-ff方式的git merge：
```
$ git merge --no-ff -m "merge with no-ff" dev
```
合并后，我们用git log看看分支历史：
```
$ git log --graph --pretty=oneline --abbrev-commit
```

如果你只是想看看本地分支和远程分支的差异，你可以使用下面的命令：:
```
git diff master origin/master
```

跟踪分支 (tracking branch)。跟踪分支是一种和某个远程分支有直接联系的本地分支

####6.4、Bug分支

####6.5、Feature分支

####6.6、多人协作

####6.7、分支交叉（branch diverged）



####10、Git图形界面
gitk 和 git-gui



参考：

- [Reference](https://git-scm.com/docs)
- [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013760174128707b935b0be6fc4fc6ace66c4f15618f8d000)