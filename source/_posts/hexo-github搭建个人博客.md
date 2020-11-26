---
title: hexo+github搭建个人博客
date: 2020-11-26 18:00:42
tags: hexo
---

最近工作中学的新东西挺多的，重开博客记录一下。博客采用hexo+github page的方式搭建。
主要参考这篇[文章](https://zhuanlan.zhihu.com/p/26625249)，进行操作。
用的是这个[主题](https://github.com/wzpan/hexo-theme-freemind)的readable配色方案。
支持搜索，留言功能，但是顶部的分类和标签按钮不可用。对布局和颜色也不是特别满意，后面有需要再折腾这些吧。

整个过程还算是比较顺利，下面简单记录一下：
<!--more-->
1. 配置hexo的_config.yaml和theme的_config.yaml，由于我还深度改了一下theme的一些文案，~~就单独fork了这个theme的仓库，然后用的是我fork的这个仓库~~。直接将该仓库添加到当前项目。
2. hexo g 的原理似乎是把source代码转换之后，放到public目录中。hexo d则是把public的代码复制到.deploy_git目录中，然后在推到xx.github.io仓库，而且似乎是push -f ??
3. 参考[这里的做法](https://www.zhihu.com/question/21193762/answer/79109280)将source代码push到该仓库的hexo分支，以期望对source进行管理。
4. 测试，在另一台电脑上clone仓库，checkout到hexo分支，执行npm install之后，就可以用hexo进行管理了。



------

2020-11-26 20:00 继续折腾



1. 看到另外一篇文章提到nexT主题，发现更好看，果断换。
2. 参考[这篇](http://theme-next.iissnan.com/getting-started.html)和[这篇](https://theme-next.js.org/docs/getting-started/configuration.html)，发现nexT主题直接通过npm下载安装到node_modules目录，同时配置文件放在外面的做法更加优雅。
3. 但是，freemind那个主题比这个看起更加“技术”一点。[苦笑]



