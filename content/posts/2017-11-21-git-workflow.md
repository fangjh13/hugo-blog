---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-11-21T07:18:51+08:00'
description: Git工作流介绍与协同工作
lastmod: '2017-11-22T09:14:53+08:00'
showToc: true
tags: [Git]
title: Git工作流
---

[Git](https://git-scm.com/)是当今最流行的开源版本控制系统，使用git的每个团队应该也有固定的工作流。今天就介绍一个现有普遍使用的工作流**git-flow**。

![git-flow](https://img.fythonfang.com/GitFlowHotfixBranch.png)

## 分支（branch）

如上图一般分支有*`master`*、*`develop`*、*`feature/xxx`*、*`release/xxx`*、*`hotfixes`*，下面一一介绍。

### 长期分支

> 长期分支是跟着产品长期存在的不会删除。

*`master`*只能用来包括产品代码，不要在这上面做任何的改动。

*`develop`*开发主分支，所有新功能的分支从这里`checkout`，所以这是开发的基础分支。该分支也是汇集所有开发新功能后最后`merge`到*`master`*的分支。

![master-develop-branch](https://img.fythonfang.com/01-master-develop.png)



### 临时分支

> 临时分支只用来开发新功能、修复bug和发布产品用最后删除。

*`feature/xxx`*开发新功能的分支从*`develop`*分支`checkout`每个功能创建一个分支是一个良好的习惯。开发测试完后`merge`到*`develop`*分支进行更全面的测试。

*`release/xxx`*产品预发布分支，一般以版本号为分支名，需要合并到*`master`*分支发布。

*`hotfixes`*紧急修复bug的分支基于*`master`*分支，修复完后一定要合并到*`master`*分支和*`develop`*分支。

![feature-hotfix-branch](https://img.fythonfang.com/02-features-hotfix.png)

## Git-flow工作流程

### 基本步骤

1. 开发人员首先从*`develop`*分支`checkout -b feature/xxx`一个待开发的分支，开发和测试。
2. 当开发和测试都完成后将*`feature/xxx`*合并到*`develop`*并删除清理。当然其中有新功能时可以再新建一个*`feature`*分支并最后合并到开发分支。
3. 终于等到release了，现在开发分支汇集了所有新开发的功能并无重大bug，我们从*`develop`*分支`checkout`一个*`release/v1.0.0`*的分支进行更加全面的测试准备发布，有bug就在*`release`*分支修复，正式发布就是把这个*`release`*分支合并到*`master`*分支打上`tag`。当然不要忘记将发布分支合并到*`develop`*分支以保持和*`master`*代码同步一致。
4. 可能发布后有紧急的bug需要修复那就从*`master`*分支`checkout`一个*`hotfixes/missing-link`*分支修复bug并合并到*`master`*和*`develop`*最后删除。

### 总结一下

以上就是**Git-flow**方案的所有流程，优点是分支清晰各干各的看名字就知道，最后只留下*`master`*和*`develop`*分枝。缺点当然是步骤比较繁琐咯，开发要新建很多分支然后切换和删除。我们可以还有很多方案选择[GitHub flow](http://scottchacon.com/2011/08/31/github-flow.html)、[GitLab flow](https://docs.gitlab.com/ee/workflow/gitlab_flow.html)等，或者理解熟练了之后自创一套也是可以的。

## GitHub协同工作

身为一个开发者一定想为开源项目贡献自己的代码，下面是基本步骤可做参考。也可以作为团队协作的基本流程。

1. `Fork`到自己帐号下并`Clone`到本地
2. `git remote add upstream https://github.com/...` 添加上游项目地址（源项目地址）以便更新到最新的代码，添加完可用`git remote -v`查看除了origin应该还有upstream远程地址
3. `git pull upstream master`从刚才添加的upstream拉取最新代码
4. `git checkout -b feature/some-feature `新建一个你要添加新功能的分支，开发就在这上面进行
5. `git add . && git commit -m 'some feature add'`测试通过后提交代码
6. `git checkout master  && git pull upstream master`这主要是从源仓库拉取最新代码，在你开发期间，如果源项目有改动提交pull就能拉取下来
7. `git checkout feature/some-feature && git rebase master `然后执行一次变基更新代码 
8. `git push origin feature/some-feature`推送你开发完并更新到最新的分支到GitHub
9. 去自己GitHub帐号下 pull request等review合并

一具话就是先在本地新建一个分支开发最后提交这个分支发PR，需要注意的是提交到远程的时候记得先pull更新代码然后rebase使分支清晰可读。


### 参考

[https://datasift.github.io/](https://datasift.github.io/gitflow/IntroducingGitFlow.html)

[https://www.git-tower.com/](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/git-flow)

[http://www.ruanyifeng.com/](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)

