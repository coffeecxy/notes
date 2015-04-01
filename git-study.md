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

当我们指定的<start-point>]是一个remote的branch的话,那么默认情况下我们新建的这个branch
就会track那个remote的branch,这个一般也是我们需要的.


## git push

	git push [option] [<repository> [<refspec>...]]

将本地的repo推到remote去,让remote同步上.

如果没有指定remote的repository,那么其会使用.git/config中找到branch.*.remote中设置的值,一般
这个值都是origin,所以如果没有能够找到这个配置的话,使用的就是origin.

如果没有指定refspec,那么会使用branch.*.push的配置,如果没有给出,那么会使用push.default的配置.

--all,如果使用这个选项,那么就不用指定refspec了,因为其会将本地所有没有同步的 *refs/heads/* 都给推送到
repo中去. 也就是让remote的每个branch都和本地的一样.
我比较喜欢使用这个选项,因为方便简单,而且能完成我们一般想做的事情. 
如果本地的有些branch还没有被创建的话,那么repo中也会创建.

--mirror,不用一个一个的指定ref,这个选项会导致remote变成本地repository的一个镜像,
意思是成功之后,那么remote repository就变成和本地的是一模一样的了. 这个选项用得不多,
特别是多人协作的工程,因为remote中包含 *refs/heads,refs/remote,refs/tag*都会被完全替换了.

## refspec
在使用push,pull,fetch等要和remote进行同步的操作的时候,一般都会用到refspec.

refspec包含三个部分,第一个是可选的+,第二个是<src>,然后跟上一个:,第三个是<dst>.

对于push,src在前面,用来标识本地的ref. 一般的,都是用来指明要同步的本地的branch.

dst用来指明remote方要被同步的ref.

如果只是指定了src,没有指定dst,那么就是从remote中删除dst指定的ref,比如

	git push origin :learn-git

那么会导致remote中的learn-git这个branch被删除.

对于`:`这个特殊的refspec,那么会将所有匹配的branch都推送到remote. 也就是
对于任何存在于本地的branch,如果remote中也有这个branch存在,那么这个branch会被同步.

	git push origin HEAD
会将本地当前正在工作的branch推送到origin这个remote去,如果remote中还没有这个branch的话,那么remote会创建这个branch.

