### 1 Git内部原理
#### 1.1 底层命令和高层命令

#### 1.5 Git 引用
我们可以借助类似于 git log 1a410e 这样的命令来浏览完整的提交历史，但为了能遍历那段历史从而找到所有相关对象，你仍须记住 1a410e 是最后一个提交。 我们需要一个文件来保存 SHA-1 值，并给文件起一个简单的名字，然后用这个名字指针来替代原始的 SHA-1 值。

在 Git 里，这样的文件被称为“引用（references，或缩写为 refs）”；你可以在 .git/refs 目录下找到这类含有 SHA-1 值的文件。 在目前的项目中，这个目录没有包含任何文件，但它包含了一个简单的目录结构：
```xml
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/tags
$ find .git/refs -type f
```









### 2 git merge conflict
http://blog.csdn.net/u012150179/article/details/14047183
##### 1、git智能自动合并
##### 2、用户手动合并(包括文件合并和树合并)


参考：

[Git 内部原理](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E5%BA%95%E5%B1%82%E5%91%BD%E4%BB%A4%E5%92%8C%E9%AB%98%E5%B1%82%E5%91%BD%E4%BB%A4)