2015年04月01日
===

## git中存储内容的地方

在git中,存储内容的地方有三个.
分别是当前目录下的`.git`文件夹,这个是所有版本存储的地方.
第二个是缓存区,也就是说的index.
第三个是working directory,也就是我们说看到的目录下面的内容.

一般进行代码修改的过程如下:

当我们修改代码的时候,首先修改的内容会在工作目录下面看到,然后使用`git add`可以将这些
内容的改变写到index中去,再使用`git commit`就会index中缓存的改变写到.git中去,这样就
产生了一个新的版本了.

当然,我们可以使用`git commit -a`直接将工作目录中的改变直接跳过index添加到.git的一个
新版本中去.

可以使用`git diff`来查看上面三个地方的当前内容的区别来确定是不是要进行commit.

`git diff`会比较工作目录和缓存中的内容,
`git diff --cached`会比较缓存和.git中的内容,
`git diff HEAD`会比较.git的内容和工作目录以及缓存中的内容的区别.

## git的branch
为了支持各种同时开发的功能,git中有branch的概念. 一个repo中可以同时存在很多个branch.
在一个时间,我们只会在其中一个branch中工作.

### 创建branch

	git branch [--set-upstream | --track | --no-track] [-l] [-f] <branchname> [<start-point>]

其中<branchname>我们想要创建的branch的名字,一般名字都会比较简短,但是能够形容这个branch的
功能.

<start-point>是可选的,表示我们新建的branch要从哪儿开始,其为一个commit,但是commit id
一般是记不住的,所以可以使用commit id,一个branch name,或者是一个tag name,反正要能够
表示一个commit. 如果其没有给出,那么默认是当前工作的branch的HEAD上.

当我们指定的

