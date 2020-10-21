# Git 的使用

## 版本控制系统

### 什么是版本控制

版本控制是用于对某一组文件进行版本的记录，即在某一时刻为这一组文件提供一个快照，并将此快照进行存储。当你需要查看一个之前的快照（称之为“版本”）时，版本控制系统可以显示出当前版本与上一个版本之间的所有改动的细节。

### 为什么需要进行版本控制

1. 存储文件副本

   可能你平时在进行coding的时候，需要对当前的这个文件进行备份，避免你写的代码使得原来的文件混乱了，导致回不到原来的结果。尤其在修复bug的时候，是非常重要的。

2. 版本控制

   避免版本管理混乱。这是使用版本管理的最主要原因，也是版本管理的目的所在。你肯定不会希望在本地手动备份了多个副本后，到头来却不知道那个备份是最新的，那个备份进行了什么修改，修改日期是什么时候等等一切你记不清的问题。而版本管理软件能解决这些问题，它有详细的日志，能记住你的每一次提交、每一次改动，并且能够比较查看不同版本之间的异同，并且可以恢复到之前的任一版本。

3. 提高协同效率

   在进行多人开发的时候，使用版本控制系统可以非常好的避免在代码合并的时候可能出现的文件混乱等现象，例如你在手动合并代码的时候，可能会漏掉某些文件，也可能某个文件的某个段落漏掉了等等，这在两个人协同开发的时候就容易出现问题，三个，四个人，会有非常大的概率出现这些问题。

4. 明确分工责任

   使用版本控制系统可以非常容易定位出现bug的代码是谁写的，可以评估每个人的代码能力，代码风格等，明确责任。

### 版本控制软件的发展历史

最开始没有版本控制管理工具，代码的整合和备份都靠手工完成，手工备份历史文件，非常容易出错。

#### 手动管理时代

早期(1970年到1980年代)的软件开发大部分是愉快的个人创作。比如UNIX下的sed是L. E. McMahon写的，Python的第一个编译器是Guido写的，Linux最初的内核是Linus写的 (好吧，awk是个例外，它的名字是三位作者的首字母，但也只是三个人)。这些程序员可以用手工的方式进行备份，并以注释或者新建文本文件来记录变动。

正如现在普通用户常做的，当时的程序员常用cp备份:

```shell
$ cp dev.c dev.bak
```

更有条理一些的程序员会加上一个时间标记，比如:

```shell
$ cp dev.c dev.bak.19890908
```

程序员很可能会用vi创建一个LOG文件来做日志:

```
1989-09-08 02:00:00
Old input method is stupid
Add command-line input function
```

在一个版本发布的时候，程序员可能做一个tar归档，将所有的文件归为同一个.tar文件。

```shell
$ tar -cf project_v1.0.tar project
```

上面的工具构成了一套人工VCS。上面的这套组合也非常符合UNIX的模块化理念：让每个应用专注于一个小的功能，使用者根据需要，将这些功能连接起来。你还可以写一个shell脚本，将上面的功能都写在里面。当需要的时候，调用该脚本就可以了。



再说一下早期的合作开发模式。通常来说，当时的程序员通过电子邮件接收补丁(patch)，并将补丁应用到原来的代码文件。实际上，一个补丁(patch)的主要功能是描述两个文件的改变(change, or file delta)。 假设我们有两个文件`a.c`和`b.c`内容分别为:

```c
// a.c (有bug的代码)
int sum(int a, int b)
{
  int c;
  c = a + 1;
  return c;
} 
```

```c
// b.c  (修正后的代码)
int sum(int a, int b)
{
  int c;
  c = a + b;
  return c;
}
```

在UNIX系统下，运行

```shell
$ diff a b > iss01.patch
```

`iss01.patch`就是一个补丁文件，它看起来如下：

```
4c4
<   c = a + 1;
---
>   c = a + b;
```

这个补丁表示，更改原文件第四行的`c = a + 1;`，改为`c = a + b;`，更改后的这一行位于新的文件的第四行。

使用patch命令将`iss01.patch`应用到`a.c`上，相当于将 `b.c-a.c` 的改变作用在a上，`a.c`将和`b.c`有一样的内容：

```shell
$ patch a.c < iss01.patch
```

当我发现`a.c`的代码有错误时，可以将我修改后的`b.c`与原来的`a.c`做diff获得补丁文件，并将补丁发给程序员，并告诉他该补丁是为了修正`a.c`代码中的加法错误。程序员确认之后，就可以使用patch应用该补丁了。在后面我们将看到，这种diff-patch的工作方式被VCS不同程度的采用。



#### 早期的版本控制

早在70年代末80年代初，VCS的概念已经存在，比如UNIX平台的RCS (Revision Control System)。RCS是由Walter F. Tichy使用C开发。RCS对文件进行集中式管理，主要目的是避免多人合作情况下可能出现的冲突。如果多用户同时写入同一个文件，其写入结果可能相互混合和覆盖，从而造成结果的混乱。你可以将文件交给RCS管理。RCS允许多个用户同时读取文件，但只允许一个用户锁定(locking)并写入文件 (类似于多线程的mutex)。这样，当一个程序员登出(check-out，见RCS的co命令)某个文件并对文件进行修改的时候。只有在这个程序完成修改，并登入(check-in，见RCS的ci命令)文件时，其他程序员才能登出文件。基本上RCS用户所需要的，就是co和ci两个命令。在co和ci之间，用户可以对原文件进行许多改变(change, or file delta)。一旦重新登入文件，这些改变将保存到RCS系统中。通过check-in将改变永久化的过程叫做提交(commit)。

RCS的互斥写入机制避免了多人同时修改同一个文件的可能，但代价是程序员长时间的等待，给团队合作带来不便。如果某个程序员登出了某个文件，而忘记登入，那他就要面对队友的怒火了。(从这个角度上来说，RCS造成的问题甚至大于它所解决的问题……)

文件每次commit都会创造一个新的版本(revision)。RCS给每个文件创建了一个追踪文档来记录版本的历史。这个文档的名字通常是原文件名加后缀,v (比如main.c的追踪文档为main.c,v)。追踪文档中包括：最新版本的文件内容，每次check-in的发生时间和用户，每次check-in发生的改变。在最新文档内容的基础上，减去历史上发生的改变，就可以恢复到之前的历史版本。这样，RCS就实现了备份历史和记录改变的功能。

相对与后来的版本管理软件，RCS纯粹线性的开发方式非常不利于团队合作。但RCS为多用户写入冲突提供了一种有效的解决方案。RCS的版本管理功能逐渐被其他软件(比如CVS)取代，但时至今日，它依然是常用的系统管理工具。

#### 一个新的软件诞生了

1986年，Dick Grune写了一系列的shell脚本用于版本管理，并最终以这些脚本为基础，构成了CVS (Concurrent Versions System)。CVS后来用C语言重写。CVS是开源软件。在当时，Stallman刚刚举起GNU的大旗，掀起开源允许的序幕。CVS被包含在GNU的软件包中，并因此得到广泛的推广，最终击败诸多商业版本的VCS，呈一统天下之势。

CVS继承了RCS的集中管理的理念。在CVS管理下的文件构成一个库(repository)。与RCS的锁定文件模式不同，CVS采用复制-修改-合并(copy-modify-merge)的模式，来实现多线开发。CVS引进了分支(branch)的概念。多个用户可以从主干(也就是中心库)创建分支。分支是主干文件在本地复制的副本。用户对本地副本进行修改。用户可以在分支提交(commit)多次修改。用户在分支的工作结束之后，需要将分支合并到主干中，以便让其他人看到自己的改动。所谓的合并，就是CVS将分支上发生的变化应用到主干的原文件上。比如下面的过程中，我们从r1.1分支出rb1.1.2.*，并最终合并回主干，构成r1.2

CVS与RCS类似，使用,v文件记录改变，以便追踪历史。在合并的过程中，CVS将两个change应用于r1.1，就得到了r1.2:

```
r1.2 = r1.1 + change(rb1.1.2.2 - rb1.1.2.1) + change(rb1.1.2.1-r1.1)
```

上面的两个改变都记录在,v文件中，所以很容易提取。

在多用户情况下，可以创建多个分支进行开发，比如：

![多分支](https://media.githubusercontent.com/media/HaoKunT/GitDemo/main/figure/%E5%A4%9A%E5%88%86%E6%94%AF.png)

在这样的多分支合并的情况下，有可能出现冲突(colliding)。比如上图中，第一次合并和第二次合并都对r1.1文件的同一行进行了修改，那么r1.3将不知道如何去修改这一行 (第二次合并比图示的要更复杂一些，分支需要先将主干拉到本地，合并过之后传回主干，但这一细节并不影响我们这里的讨论)。CVS要求冲突发生时的用户手动解决冲突。用户可以调用编辑器，对文件发生合并冲突的地方进行修改，以决定最终版本(r1.3)的内容。

 

CVS管理下的每个文件都有一系列独立的版本号(比如上面的r1.1,r1.2,r1.3)。但每个项目中往往包含有许多文件。CVS用标签(tag)来记录一个集合，这个集合中的元素是一对(文件名：版本号)。比如我们的项目中有三个文件(file1, file2, file3)，我们创建一个v1.0的标签：

tag v1.0 (file1:r1.3) (file2:r1.1) (file3:r1.5)

v1.0的tag中包括了r1.3版本的文件file1，r1.1版本的file2…… 一个项目在发布(release)的时候，往往要发布多个文件。标签可以用来记录该次发布的时候，是哪些版本的文件被发布。

 

CVS应用在许多重要的开源项目上。在90年代和00年代初，CVS在开源世界几乎不二选择 (RCS也是开源的，但正如我们已经提到的，RCS无法与CVS媲美)。时至今天，尽管CVS已经长达数年没有发布新版本，我们依然可以在许多项目中看到CVS的身影。


#### SVN
CVS也有许多常常被人诟病的地方，比如下面几条：

- 合并不是原子操作(atomic operation)：如果有两个用户同时合并，那么合并结果将是某种错乱的混合体。如果合并的过程中取消合并，不能撤销已经应用的改变。
- 文件的附加信息没有被追踪：一旦纳入CVS的管理，文件的附加信息(比如上次读取时间)就被固定了。CVS不追踪它所管理文件的附加信息的变化。
- 主要用于管理ASCII文件：不能方便的管理Binary文件和Unicode文件
- 分支与合并需要耗费大量的时间：CVS的分支和合并非常昂贵。分支需要复制，合并需要计算所有的改变并应用到主干。因此，CVS鼓励尽早合并分支。

CVS还有其它一些富有争议的地方。随着时间，人们对CVS的一些问题越来越感到不满 (而且程序员喜欢新鲜的东西)，Subversion应运而生。Subversion的开发者Karl Fogel和Jim Blandy是长期的CVS用户。赞助开发的CollabNet, Inc.希望他们写一个CVS的替代VCS。这个VCS应该有类似于CVS的工作方式，但对CVS的缺陷进行改进，并提供一些CVS缺失的功能。

总体上说，Subversion在许多方面沿袭CVS，也是集中管理库，通过记录改变来追踪历史，允许分支和合并，但并不鼓励过多分支。Subversion在一些方面得到改善。Subversion的合并是原子操作。它可以追踪文件的附加信息，并能够同样的管理Binary和Unicode文件。但CVS和Subversion又有许多不同：

与CVS的,v文件存储模式不同，Subversion采用关系型数据库来存储改变集。VCS相关数据变得不透明。
CVS中的版本是针对某个文件的，CVS中每次commit生成一个文件的新版本。Subversion中的版本是针对整个文件系统的(包含多个文件以及文件组织方式)，每次commit生成一个整个项目文件系统树的新版本。
Subversion依赖类似于硬连接(hard link)的方式来提高效率，避免过多的复制文件本身。Subversion不会从库下载整个主干到本地，而只是下载主干的最新版本。

 

在Subversion刚刚诞生的时候，来自CVS用户的抱怨不断。他们觉得在Subversion中有太多的改动，有些改动甚至是相对于CVS的倒退。比如CVS中的tag，在Subversion中被改为直接复制版本的文件系统树到一个特殊的文件夹。然而，随着时间的推移，Subversion逐渐推广 (Subversion已经是Apache中自带的一个模块了，Subversion应用于GCC、SourceForge，新浪APP Engine等项目)，并依然有活跃的开发，而CVS则逐渐沉寂。事实上，许多UNIX的参考书的新版本中，都缩减甚至删除了CVS的内容。



#### Git

git的作者是Linus Torvald。对，就是写Linux Kernel的那个Linus Torvald。Linus在贡献了最初的Linux Kernel源代码之后，一直领导着Linux Kernel的开发。Linus Torvald本人相当厌恶CVS(以及Subversion)。然而，操作系统内核是复杂而庞大的代码“怪兽” （2012年的Linux Kernel有1500万行代码，Windows的代码不公开，估计远远超过这一数目）。Linux内核小组最初使用.tar文件来管理内核代码，但这远远无法匹配Linux内核代码的增长速度。Linus转而使用BitKeeper作为开发的VCS工具。BitKeeper是一款分布式的VCS工具，它可以快速的进行分支和合并。然而由于使用证书方面的争议(BitKeeper是闭源软件，但给Linux内核开发人员发放免费的使用证书)，Linus最终决定写一款开源的分布式VCS软件：git。



对于一个开发项目，git会保存blob, tree, commit和tag四种对象。

- 文件被保存为blob对象。
- 文件夹被保存为tree对象。tree对象保存有指向文件或者其他tree对象指针。

上面两个对象类似于一个UNIX的文件系统，构成了一个文件系统树。

- 一个commit对象代表了某次提交，它保存有修改人，修改时间和附加信息，并指向一个文件树。这一点与Subversion类似，即每次提交为一个文件系统树。
- 一个tag对象包含有tag的名字，并指向一个commit对象。

![Git的存储结构](https://media.githubusercontent.com/media/HaoKunT/GitDemo/main/figure/Git的存储结构.png)

每个对象的内容的checksum校验(checksum校验可参阅[IP头部的checksum](http://www.cnblogs.com/vamei/archive/2012/12/02/2796988.html))都经过SHA1算法的HASH转换。每个对象都对应一个40个字符的HASH值。每个对象对应一个HASH值。两个内容不同的对象不会有相同的HASH值(SHA1有可能发生碰撞，但概率非常非常非常低)。这样，git可以随时识别各个对象。通过HASH值，我们可以知道这个对象是否发生改变。



在整个开发过程中，可能会有许多次提交(commit)。每次commit的时候，git并不总是复制所有的对象。git会检验所有对象的HASH值。如果该对象的HASH值已经存在，说明该对象已经保存过，并且没有发生改变，所以git只需要调整新建tree或者commit中的指针，让它们指向已经保存过的对象就可以了。git在commit的时候，只会新建HASH值发生改变的对象。如下图所示，我们创建新的commit的时候，只需要新建一个commit对象，一个tree对象和一个blob对象就足够了，而不需要新建整个文件系统树。

![Git的commit](https://media.githubusercontent.com/media/HaoKunT/GitDemo/main/figure/Git%E7%9A%84commit.png)

可以看到，与CVS,Subversion保存改变(file delta)的方式形成对照，git保存的不是改变，而是此时的文件本身。由于不需要遵循改变路径来计算历史版本，所以git可以快速的查阅历史版本。git可以直接提取两个commit所保存的文件系统树，并迅速的算出两个commit之间的改变。



同样由于上面的数据结构，git可以很方便的创建分支(branch)。实际上，git的一个分支是一个指向某个commit的指针。合并时，git检查两个分支所指的两个commit，并找到它们共同的祖先commit。git会分别计算每个commit与祖先发生的改变，然后将两个改变合并(同样，针对同一行的两个改变可能发生冲突，需要手工解决冲突)。整个过程中，不需要复制和遵循路径计算总的改变，所以效率提高很多。



由于git创建、合并和删除分支的成本极为低廉，所以git鼓励根据需要创建多个分支。实际上，如果分支位于不同的站点(site)，属于不同的开发者，那么就构成了分布式的多分支开发模式。每个开发者都在本地复制有自己的库，并可以基于本地库创建多个本地分支工作。开发者可以在需要的时候，选取某个本地分支与远程分支合并。git可以方便的建立一个分布式的小型开发团队。比如我和朋友两人各有一个库，各自开发，并相互拉对方的库到本地库合并(如果上面master，develop代表了两个属于不同用户的分支，就代表了这一情况)。当然，git也允许集中式的公共仓库存在，或者多层的公共仓库，每个仓库享有不同的优先级。git的优势不在于引进了某种开发模式，而是给了你设计开发模式的自由。



版本控制软件分为两大类，集中式版本控制和分布式版本控制

#### 集中式版本控制（VCS）

CVS和SVN是集中式版本控制的典型代表，其具有的特点是：

- 完整的代码版本存储在服务器上

- 具备文件历史版本的管理能力和分支能力

- 自动化集成，效率更高

集中式版本控制的缺点是只有服务器有完整的代码版本，因此容易产生单点故障，即若服务器挂掉了，所有的人都无法进行代码推送，拉取，合并等操作，如果服务器的代码库丢失了，整个软件版本历史也就丢失了。



![集中式版本控制](https://media.githubusercontent.com/media/HaoKunT/GitDemo/main/figure/%E9%9B%86%E4%B8%AD%E5%BC%8F%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6.jpg)



#### 分布式版本控制（DCS）

最具代表性的就是Git

- 自动完成代码整合和版本管理
- 分支管理
- 服务器和客户端都有完整的版本库
- 即使服务器宕机了，仅在本地也可以完成版本管理的操作

像Git这种分布式版本控制方式的缺点是会占用较大的磁盘空间，但是随着现在硬盘的大小越来越大，也不再是一个问题。另一个缺点是，保密性较差，因为客户端也有整个项目的所有版本历史，因此一旦在客户端处泄密，整个项目的所有版本源码就都被泄露出去了。



![分布式版本控制](https://media.githubusercontent.com/media/HaoKunT/GitDemo/main/figure/%E5%88%86%E5%B8%83%E5%BC%8F%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6.jpg)



### Git与GitHub的区别

可能有人会想问Git和GitHub的区别，其实GitHub只是一个基于Git所搭建的代码存储平台，即一个专门用于管理代码的平台。在GitHub上存储有非常多的Git仓库，这些仓库以一定的权限进行组织。GitHub还提供了关于项目的讨论平台（可以替代早期的邮件交流），代码版本的可视化，产品的发布，wiki等与项目有关的内容，旨在帮助程序员更好的编写自己的代码和使用他人的代码，也是为开源软件的代码管理提供一个非常好的平台。

简而言之，用一个不太恰当的比喻，Git好比是一个纯命令行软件，GitHub则将之进行了可视化（实际的执行操作还是使用的命令行软件，实际上这样的软件还有很多，例如PicGo，Vray2N等），还加上了很多和项目管理相关的内容。

> GitHub也被戏称为“全球最大的同性恋交友网站”，这是因为GitHub目前已有千万级的开发者注册，非常多的著名开源项目都托管在上面，例如大名鼎鼎的Linux内核
>
> GitHub已于2018年被微软收购。


## 操作其他远程仓库

Git 除了可以操作自己的本地仓库和 origin 远程仓库，也可以添加其他仓库进行操作。如果其他仓库开放了写入权限，还可以直接更改其他仓库。

本节涉及如下知识点：

1. 管理远程仓库列表
2. 从其他远程仓库合并
3. 将代码推送到其他远程仓库

### 管理远程仓库列表

每个 Git 本地仓库都可以记录许多远程仓库，每个远程仓库记录（简称“远程记录”）都可以起一个名字（简称“远程记录名”），这些远程仓库的地址都可以通过 `git remote` 命令进行操作。常用的操作有增、删、查、改。

**增**：如果要增加一个远程仓库，地址是 https://gitlab.gwmodel.whu.edu.cn/HPDell/GitDemo.git ，并命名为 gitlab ，则使用如下命令

```bash
git remote add gitlab http://gitlab.gwmodel.whu.edu.cn/HPDell/GitDemo.git
## 格式
# git remote add <远程记录名> <远程仓库地址>
```

远程仓库地址可以是 http https ssh 等类型的地址。

**查**：如果要查看本地仓库添加了哪些远程，则使用如下命令

```bash
git remote -v
## 输出
# origin  git@github.com:HPDell/GitDemo.git (fetch)
# origin  git@github.com:HPDell/GitDemo.git (push)
# gitlab  http://gitlab.gwmodel.whu.edu.cn/hpdell/GitDemo.git (fetch)
# gitlab  http://gitlab.gwmodel.whu.edu.cn/hpdell/GitDemo.git (push)
```

每个远程仓库对应了一个 fetch 地址和一个 push 地址，一般情况下 fetch 和 push 地址是一样的。因为我们添加远程的时候就指定了一个地址。

如果不加 `-v` 参数，那么只会输出所有远程记录名

**改**：修改远程仓库涉及到两个修改，即“重命名远程仓库”和“修改远程仓库地址”。

重命名远程仓库使用 `git remote rename` ，例如把 gitlab 远程名重命名为 lab ，使用如下命令

```bash
git remote rename gitlab lab
## 格式
# git remote rename <远程记录名> <新名称>
```

修改远程仓库地址使用 `git remote set-url` ，例如把 gitlab 远程仓库的地址修改为 ssh 的地址，使用如下命令

```bash
git remote set-rul gitlab git@gitlab.gwmodel.whu.edu.cn:hpdell/GitDemo.git
## 格式
# git remote set-url <远程记录名> <新地址>
```

**删**：如果要删除一个远程记录，如 gitlab ，则使用如下命令

```bash
git remote remove gitlab 
## 格式
# git remote remove <远程记录名>
```

总结起来，这套命令其实非常好记，基本格式就是

```bash
git remote <动作> <远程记录名> [其他参数]
```

不同的动作及其含义和参数总结如下表

| 动作      | 动作含义               | 参数                     |
| --------- | ---------------------- | ------------------------ |
| `add`     | 新增远程记录           | 远程仓库地址             |
| `rename`  | 重命名远程记录         | 新的记录名               |
| `set-url` | 设置远程记录的仓库地址 | 远程仓库地址             |
| `remove`  | 删除远程记录           |                          |
|           | 显示远程记录列表       | 有 `-v` 表示显示详细信息 |

### 从其他远程仓库合并

这个情况一般发生在当别人的仓库中做了一些更改，想要合并到自己的仓库中时。例如：

- 张三和李四的仓库克隆自同一个仓库，张三做了一些修改，李四想要合并张三的修改
- 张三克隆了 Vue 的仓库，但尤雨溪合并了其他人提交的 Pull Request ，张三需要将这些更新合并到自己的仓库中

比如我要合并 HaoKunT/GitDemo 的仓库，执行这样操作的完整流程是这样的

```bash
git remote add HaoKunT git@github.com:HaoKunT/GitDemo.git
git fetch HaoKunT
git checkout HaoKunT/master 
git checkout -b master-HaoKunT
git checkout master
git merge --no-ff master-HaoKunT
```

或者省略到中间 `git checkout` 的操作，直接合并“远程分支”。

```bash
git remote add HaoKunT git@github.com:HaoKunT/GitDemo.git
git fetch HaoKunT
git merge --no-ff HaoKunT/master
```

总结起来有四步：添加远程、获取远程、签出分支、合并分支。签出分支一步可以省略，此时合并分支直接合并远程分支。

**添加远程**：添加一个远程记录，地址是远程仓库。和之前所讲的一样。

**获取远程**：获得远程仓库中的提交记录。这里使用的是 `git fetch` 命令，后面可以加一个远程记录名作为参数，用于将远程仓库的记录获取到本地仓库的 .git 文件夹中，而不对代码文件作修改。如果要获取一个远程记录中的仓库 HuYigong/GitDemo 的递交记录，使用方法是

```bash
git fetch HuYigong/GitDemo
## 格式
# git fetch <远程记录名>
```

而如果不加参数，则认为是获取 origin 远程的提交记录。

**签出分支**：签出同样也是使用的 `git checkout` ，后面加分支名。这个分支可以是一个远程分支。如果签出的分支名是远程分支名，那么签出的结果是使 HEAD 指向一个提交记录，此时需要再执行下列命令创建一个本地分支，这个分支拥有的提交记录和远程一样。

```bash
git checkout -b master-HaoKunT
## 格式
# git checkout -b <新分支名>
```

在签出之前可以查看一下远程仓库的分支等信息，使用的是 `git branch ` 命令，后面加上 `-r` 参数可以查看远程分支，加上 `-a` 参数可以查看远程和本地分支

```bash
git branch -a -v 
## 输出
#  main                4566a89 Create README.md
#* master-HaoKunT      4566a89 Create README.md
#  remotes/origin/HEAD -> origin/main
#  remotes/origin/main 4566a89 Create README.md
#  remotes/qqrepo/main 160531f Update README.md
```

> 三个签出的操作，可以使用另一套方法完成：新建分支、设置上游、拉取代码。
>
> ```bash
> git checkout -b master-HaoKunT
> git branch --set-upstream-to=HaoKunT/master 
> git pull --rebase
> ```
>
> 其中，`git branch` 所加的选项 `--set-upstream-to=HaoKunT/master` 称为设置上游。

**合并分支**：签出到要合并到的分支后，同样也是使用 `git merge ` 命令来合并新的分支。后面可以加一个本地分支，也可以加一个远程分支。

> 之所以本地仓库可以直接合并远程分支，是因为经过 `git fetch` 操作后，远程仓库中的所有提交记录就已经下载到本地仓库中了，此时这些分支就和本地仓库中未签出的分支一样，可以进行同样的操作。所以他们虽然叫做远程分支，但是是存储在本地仓库中的。

### 推送到其他远程仓库

如果要推送到的远程仓库有权限，那么可以直接使用 `git push` 命令推送到远程仓库，后面跟上要推送到的远程记录名，远程会创建一个相同的分支。例如，将 master 分支推送到 HaoKunT/master 

```bash
git push HaoKunT master
```

如果后面不加 `master` 参数，则指定推送当前分支。

