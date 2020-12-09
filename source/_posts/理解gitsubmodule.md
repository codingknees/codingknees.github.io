---
title: 理解gitsubmodule
date: 2020-12-09 21:27:09
tags: git,gitsubmodule
---

git submodule最好的理解，就是指针。主仓库中，某个特定的目录，在版本管理模型中，不被标识为目录，而是一个指向其他仓库某个具体提交(commit-ish)的指针。
因此，对于子仓库目录而言，git不关心里面的具体内容，而只关心他的“指向”，即commit-ish。目录在，则子仓库在；目录被删除，则认为不再引用该子仓库。而目录在，不管里面的内容有还是没有（有，且被修改是另外一回事），都认为对子仓库的引用不变。这里之所以说这里多，是因为这对子仓库添加，拉取，删除的理解非常重要。

<!--more-->

子仓库信息在主仓库的三个地方进行了记录：
    * .gitmodule文件，这个会被添加进仓库
    * 子仓库对应的目录，也会被添加到仓库，只是不是以目录的形式，而是一个commit-ish
    * .git/config文件，很明显，这不会被加入到代码库中
三个地方的信息互有联系，又各司其职。下面依次用git submodule的各个子命令来理解这里面的联系和区别，以及他们的职责。

1. `git submodule ` 读取HEAD的tree-ish中，子仓库的目录已经对应的commit-ish
2. `git submodule add xx.git xx` 同时更新上面列到的三个地方

在添加子仓库之后，代码库会出现两个新的变更。一个是.gitmodule文件，一个是子仓库目录。提交这两个变更就算是完成了子仓库的添加。

3. `git clone ` 拉取仓库代码，明显，.gitmodule和子仓库目录（空）会拉取到本地
4. `git submodule init xx` 从git的输出来看，这一步是注册子仓库。这里对注册的理解，是把.gitmodule的内容“搬”到.git/config目录中，“搬”加了引号，因为不是简单的拷贝，当子仓库地址是相对文件路径时，需要计算出一个新的路径，可以直接进行子仓库的clone的新地址。
5. `git submodule update xx` 依据HEAD中记录的子仓库的commit-ish，以及.git/config的子仓库地址，在子仓库的目录中checkout出对应的内容。如果子仓库中已经有代码，似乎什么都不做。如果要放弃那些变更，可以添加-f选项。但是对于未加入代码跟踪的内容，-f也无法移除。似乎只能进入对应目录，执行`git clean -fd`进行删除。

到这里，对于`git clone` 之后，还需要`git submodule init`和`git submodule update`就很好理解了。子仓库代码的管理需要.git/config，因此必须先进行init。拉取子仓库的代码，则是用update命令进行。如果嫌操作麻烦，`git clone --recursive`选项也提供好了，一条命令执行clone以及全部子仓库的init和update。
很明显，init和update并不会产生新的变更，他们只是初始化了当前工作区对子仓库的操作环境，后面就可以通过git submodule的子命令对子仓库进行管理（虽然也没啥可管理的）。

最后，如何删除子仓库就可以推导出来了：
    * 删除.gitmodule文件中的条目
    * 删除子仓库对应的目录
提交上述两个操作。这样新clone仓库的人，子仓库就彻底从他们视线里消失了。
但是还有一些事情需要注意：
    * .git/config还留着个尾巴，还需要手动删除（尽管这个尾巴似乎不会造成什么问题）
    * 对于已经clone人来说，pull操作会让他们的子仓库目录变成untracked状态。
从上面的理解来看，这里缺少的其实是对.git/config和子目录文件的维护。命令准备好了，就是这个：

6. `git submodule deinit xx` 相当于是init和update的逆向操作，即删除子仓库目录下的所有内容，删除.git/config文件中，子仓库的配置。执行这条操作之后，去除了当前工作区对子仓库的操作环境，无法再通过git submodule 子命令对子仓库进行操作。

操作之后发现，先删除子仓库目录，再执行deinit会报找不到的错误。因此，应该先执行deinit，然后再进行删除。
对于其他人来说，先deinit再pull固然可以，但是操作上不现实。目前看，只能手动删除子仓库目录和维护.git/config了。

理解完这些之后，剩下的git submodule子命令就好办了。啃[官方文档](https://git-scm.com/docs/git-submodule)也更带劲了。