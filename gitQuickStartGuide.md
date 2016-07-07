title: svn迁移git快速上手指南
speaker: 王佳
transition: slide3
theme: moon
usemathjax: yes

[slide]
# svn迁移git快速上手指南
## 1 起步
### 1.1 Git简介
 * git是用于Linux内核开发的版本控制工具。与CVS、Subversion一类的集中式版本控制工具不同，它采用了分布式版本库的作法，不需要服务器端软件，就可以运作版本控制，使得源代码的发布和交流极其方便。git的速度很快，这对于诸如Linux内核这样的大项目来说自然很重要。git最为出色的是它的合并追踪（merge tracing）能力。 {:&.rollIn}
 * 设计目标：高度易用、速度、简单的设计、非线性开发模式、完全分布式、有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）。
 * 作为开源自由原教旨主义项目，git没有对版本库的浏览和修改做任何的权限限制，通过其他工具也可以达到有限的权限控制

[slide]
### 1.2 基本概念对照表
 概念              | SVN                 | GIT 
------------------|----------------------|---------------------
某个版本或某次提交   |revision(递增的数字)  |commit(40位的SHA摘要值)
中央服务器          |svn.baidu.com        |icode.baidu.com
代码评审平台        |cooder.baidu.com     |icode.baidu.com
代码评审工具        |coode                |git-review
Studio用GUI插件    |cooder插件            |gerrit插件
代码仓库所在位置    |仅在中央服务器上       |分布在中央服务器和本地机器上

[slide]
### 1.3 基本操作对照表
 操作              | SVN                     | GIT 
-------------------|-------------------------|---------------------
取出版本库          |无                       |git clone
取出分支            |svn switch               |git checkout
取出版本            |svn checkout             |git checkout
本地提交修改        |无                       |git commit
向服务器提交拉取代码 |svn commit 和 svn update |git push 和 git pull
提交代码审核        |cooder                    |git review 或 gerrit 或 git push
创建分支            |svn copy                 |git branch 或 git checkout -b
合并分支           |svn merge                 |git merge
重设分支基点        |无                        |git rebase

[slide]
## 2 环境配置
### 2.1 git下开发所需工具一览
* `git` - git功能主程序   {:&.rollIn}
* `git-review` - git review子命令，帮助完成更便捷的代码评审功能 
* `gerrit`插件 - Android Studio代码评审插件，一键提交代码评审
* `python` - git-review工具需要python运行时
* `tig`、`gitk`等 - 非必需，提供更多Git相关的操作和功能

[slide]
### 2.2 安装git和git-review子命令,配置git环境
``` sh
#安装git
$ sudo apt-get install git

#安装git-review自命令，请参考https://www.mediawiki.org/wiki/Gerrit/git-review
#windows和OSX下要先安装python，然后在git bash下用python包管理器pip安装git-review
$ pip install git-review
#linux包管理器安装git-review
$ sudo apt-get install git-review

#配置全局用户信息（可选）
#一次性为全局用户统一配置用户信息
git config --global user.name wangjia20
git config --global user.email wangjia20@baidu.com
```

[slide]
### 2.3 Android Studio Gerrit插件
在Android Studio直接搜索`Gerrit`插件安装即可

[slide]
### 2.3 为icode配置ssh key
一台机器首次使用icode需要配置ssh key信息；

1. 查看本地公钥 `cat ~/.ssh/id_rsa.pub`,如果本地已有公钥，可跳过第2步直接执行第3步；  {:&.rollIn}
2. 生成密钥对：`ssh-keygen -t rsa`
    * 填写SSH密钥存放目录；或直接回车存放在默认位置：~/.ssh/ {:&.rollIn}
    * 输入SSH密钥的使用密码并记住；或直接回车不设置密码
    * `cat ~/.ssh/id_rsa.pub`，复制公钥信息。
3. 浏览器打开icode在个人设定页面添加公钥处粘贴公钥信息http://icode.baidu.com/git/account/profile。

[slide]
## 3 迁出并配置icode git工程
### 3.1 找到clone命令，clone 代码仓库,配置评审钩子

1. 打开icode上的工程主页，http://icode.baidu.com/files/view/baidu/appsearch/main/@tree/master   {:&.rollIn}
2. 在页面右上角找到完整的ssh git clone命令
3. 执行命令即可完成克隆工程

```sh
#完整的icode clone命令包含了clone代码仓库和配置评审钩子两个子命令，中间以&&符号分割。
$ git clone ssh://wangjia20@icode.baidu.com:8235/baidu/appsearch/main 
    && 
    scp -p -P 8235 wangjia20@icode.baidu.com:hooks/commit-msg main/.git/hooks/

#分解动作1 克隆公司仓库到本地
$ git clone ssh://wangjia20@icode.baidu.com:8235/baidu/appsearch/playground 
#分解动作2 为gerrit设置提交评审的钩子，目的是为了让提交信息自动生成用于审核的id
$ scp -p -P 8235 wangjia20@icode.baidu.com:hooks/commit-msg playground/.git/hooks/
```

[slide]
### 3.2 第三步 配置用户信息
``` sh
#仅为本仓库配置用户信息
$ git config user.name wangjia20
$ git config user.email wangjia20@baidu.com
```
或者
``` sh
#一次性为全局用户统一配置用户信息
$ git config --global user.name wangjia20
$ git config --global user.email wangjia20@baidu.com
```

[slide]
## 4 git仓库基本操作
### 4.1 常用命令
* **更新远程状态** - `git fetch`  {:&.rollIn}
* **查看当前状态** - `git status`  
* **查看日志信息** - `git log`
* **分支相关操作** - `git branch` `git checkout`
* **本地提交操作** - `git commit`
* **远程推送提交** - `git push`
* **远程推送评审** - `git review`
* **获取远程提交** - `git pull`
* **获取远程评审** - `git review -d`
* **合并分支代码** - `git merge`

[slide]
### 4.2 分支操作
#### 4.2.1 查看分支信息
* **选项 -a**  表示查看全部分支，包括本地分支和远程分支  {:&.rollIn}
* **选项 -vv** 表示显示分支详情，包括分支跟踪信息和最后一次提交
* **输出** |当前分支标记|分支名|HEAD提交|分支跟踪信息|HEAD提交日志|

```sh
$ git branch -a -vv  
* master                                   9735509 [origin/master] v7.3.0封版发布
  sptrint133                               f3fcfb5 [origin/sptrint133：落后 2] 实现story功能
  sptrint136                               9735502 [origin/sptrint136：领先 3] story23功能提测
  topic34                                  150e1ec 技术小组项目xxx开发
  hotfix56                                 546ab8a [origin/hotfix56: 丢失] 紧急修复线上bug
  experiment/study                         fdffe0e xxx技术调研

  remotes/origin/master                    9735509 v7.3.0封版发布
  remotes/origin/sptrint133                7e3eed0 story28功能提测
  remotes/origin/sptrint136                7e3eed0 第二次全功能提测
  remotes/origin/sptrint136                7e3eed0 第二次全功能提测
  remotes/origin/topic56                   7e3eed0 技术小组项目xx开发
  remotes/origin/hotfix56                  7e3eed0 紧急修复线上bug
  remotes/origin/experiment/review-default 9735509 评审默认分支（不小心提错了会提到这里）
  remotes/origin/svn-rebase                9735509 配置Git必要的信息
```

[slide]
#### 4.2.2 创建分支
* **选项 -t** 起点是一个分支时，表示将起点分支设置为新分支的跟踪分支  {:&.rollIn}
* **参数 <新分支>** 必要，待创建的分支名
* **参数 <起点>** 非必要，迁出新分支时指定起点，默认当前分支作为起点，这个起点可以是任何一个提交
* **输出** 创建的结果

```sh
$ git branch <目标分支> -t <起点>
```

[slide]
#### 4.2.3 迁出和创建分支
* **选项 -b** 表示创建切换新分支上，包括本地分支和远程分支  {:&.rollIn}
* **选项 -t** 迁出新分支时，表示将起点分支设置为新分支的跟踪分支
* **参数 <目标分支>** 必要，待迁出的分支名
* **参数 <起点分支>** 非必要，迁出新分支时指定起点，默认当前分支作为起点
* **输出** 迁出的结果

```sh
$ git checkout -b <目标分支> -t <起点分支>
```

[slide]
#### 4.2.4 删除本地分支
* **选项 -d** 表示删除目标分支  {:&.rollIn}
* **参数 <目标分支>** 必要，待删除的分支名
* **输出** 操作的结果

```sh
$ git branch -d <目标分支>
```

[slide]
#### 4.2.5 删除远程分支
* **选项 --delete** 表示删除目标分支  {:&.rollIn}
* **参数 <remove>** 必要，远端仓库别名
* **参数 <目标分支>** 必要，待删除的分支名
* **输出** 操作的结果

```sh
$ git push --delete <remote> <目标分支>
```

[slide]
### 4.3 协同开发
#### 4.3.1 本地提交
* **选项 -m** 输入提交日志  {:&.rollIn}
* **输出** 提交的结果

```sh
$ git add .
$ git commit -m "提交日志"
```

[slide]
#### 4.3.2 拉取其他分支上的更新
* **参数 <repo> <refs>** 可选，指定拉取更新的目标仓库和目标分支  {:&.rollIn}
* **无参数** 拉取当前分支跟踪的分支，如果没有设置跟踪分支，就会失败
* **输出** 拉取的结果
```sh
$ git pull [<repo> [<refs>]]
```

[slide]
#### 4.3.3 推送更新到其他分支上
* 由于icode的限制，本地修改不能直接推送到服务器，需要代码审核才可以
* **选项 -u** 表示当推送新分支到远端时，自动将新分支设置为当前分支的跟踪分支   {:&.rollIn}
* **参数 <repo> <refs>** 可选，指定推送提交的目标仓库和目标分支 
* **无参数** 推送当前分支到跟踪的分支，如果没有设置跟踪分支，就会失败
* **输出** 推送的结果
```sh
$ git push -u [<repo> [<refs>]]
```

[slide]
#### 4.3.3 推送推送本地提交到icode代码审核
* 代码审核通过后将自动合入远程库，无需在写issus号了  {:&.rollIn}
* **选项 --track** 表示将提交推送到远程跟踪分支的审核区   {:&.rollIn}
* **参数 BRANCH** 可选，指定推送提交的目标仓库和目标分支 
* **无参数** 推送当前分支到跟踪的分支，如果没有设置跟踪分支，就会失败
* **输出** 推送的结果和代码审核ID
```sh
$ git review --track [BRANCH]
```

[slide]
#### 4.3.4 拉取icode审核中的代码到本地查看
* 在icode找到代码审核对应的ID，就可以将这个审核拉到本地查看  {:&.rollIn}
* **选项 -d CHANGE** 下载并应用制定的审核中代码到本地暂存区   
* **参数 CHANGE** 可选，指定推送提交的目标仓库和目标分支 
* **无参数** 推送当前分支到跟踪的分支，如果没有设置跟踪分支，就会失败
* **输出** 拉取的结果
```sh
$ git review -d [CHANGE]
```

[slide]
#### 4.3.5 合并分支代码
* 在icode上可以合并无冲突的分支，如果代码有冲突的话就需要在本地合并解决冲突  {:&.rollIn}
* **参数 目标分支** 待合并的分支 
* **输出** 合并的结果
```sh
$ git merge 目标分支
```

[slide] 
### 4.4 快速入门学习资料
* 图解Git - https://marklodato.github.io/visual-git-guide/index-zh-cn.html   {:&.rollIn}
* git book - http://gitbook.liuhui998.com/
* pro git - https://progit.org/

[slide]
## 公司仓库分支结构

* `master` - 充当主干分支，master分支的HEAD永远是线上最新release版本的tag  {:&.rollIn}
* `sprint1` `sprint2` `...` - 迭代小组开发分支，每次迭代开始前基于master HEAD，也就是最新的发版tag的创建分支，每次迭代封版后，rebase主干分支，打tag并merge回到主干后删除；
* `story1` `story2` `feature/*` `...` - 使用icafe创建story卡片对应的分支，基于迭代分支的HEAD，提测或单元测试完成后rebase迭代分支，merge回迭代分支后删除分支；
* `topic1` `topic2` `...` - topic认领或技术组同学独立开发分支，从master分支拉出，开发完毕后rebase方式合回任意迭代的开发分支，并删除分支；
* `hotfix` - 基于某次release版本的tag临时创建的分支，用于修复线上bug，重新封版发布后，打tag并rebase合回任一迭代分支，并删除分支；
* `experiment/*` - 实验性质的分支，无特殊限制；

[slide]
## 本地仓库分支结构

* `sprint1` track origin/sprint1 - 迭代团队的开发分支，对应上游分支是sprint1；用于story提测后修复bug，如果story较小，也可直接用于开发story； {:&.rollIn}
* `story1` `story2` `feature/*` `...` track origin/sprint1 - 用于开发迭代story1的本地分支，对应上游分支是sprint1；功能开发完成后rebase并push到远端的sprint分支，通过审核后删除分支； 
* `story3` track origin/story3 - 用于开发迭代story1的本地分支，对应上游分支是story3；功能开发完成后rebase并push到远端的sprint分支，合并回迭代后删除分支；
* `topic1` track origin/topic1 - topic认领或技术组同学独立开发分支，可以没有对应的上游分支，也可以对应上游分支topic1，开发完成后提交给上游的topic1分支或某迭代的开发分支；
* `hotfix` track origin/hotfix - 用于修复bug，对应上游分支是hotfix，重新封版后打tag，任一迭代分支。
* `experiment/*` - 实验性质的分支，追踪上游同名分支；
* `study` no track - RD自研项目分支，没有追踪上游分支。

[slide]
## 普通开发工作流
``` sh
#创建并切到本地分支，并跟踪上游迭代开发分支
$ git checkout -b dev
$ git push -u

#开发功能，可能发生了若干次本地提交
$ git commit -m "功能开发"
...
$ git pull
$ git push

#合回代码并封版
$ git rebase master
$ git tag v7.2.0
$ git checkout master
$ git merge dev
$ git push
$ git push tags

#删除dev分支
$ git branch -d dev
```

[slide]
## 百度icode普通story开发工作流
``` sh
#迭代负责人: 基于master创建迭代分支sprint1并推送到服务器
$ git checkout -b sprint1 master
$ git push -u

#RD: 基于迭代开发分支新建分支story1
$ git checkout -b story1 -t origin/sprint1

#RD: 开发功能，可能发生了若干次本地提交
$ git commit -m "功能开发"
...
$ git commit -m "功能提测"

#RD: 功能提测，rebase最新sprint代码并提交审核
$ git rebase
$ git review sprint1 #git push origin sprint1:refs/for/sprint1
#RD: 继续跟进QA修改bug，或者开新的story分支开发新功能

#迭代负责人: 全功能集成提测并封版
$ git rebase master
$ git tag v7.2.0
$ git review sprint1  #git push origin sprint1:refs/for/sprint1
$ git push tags
#迭代负责人: 最后在icode上将sprint1分支合并回归到master主干后删除分支
```

[slide]
## 关联icafe卡片的story开发工作流
``` sh
#迭代负责人: 基于master创建迭代分支sprint1并推送到服务器
$ git checkout -b sprint1 master
$ git push -u

#RD: 在icafe卡片上，基于sprint1新建story分支，并检出本地分支
$ git checkout -b story1 -t origin/story1

#RD: 开发功能，可能发生了若干次本地提交
$ git commit -m "功能开发"
...
$ git commit -m "功能提测"

#RD: 功能提测，rebase并在icode上merge回sprint
$ git pull
$ git rebase sprint1
$ git review story1 #git push origin story1:refs/for/story1
#RD: 继续跟进QA修改bug，或者开新的story分支开发新功能

#迭代负责人: 全功能集成提测并封版
$ git rebase master
$ git tag v7.2.0
$ git review sprint1 #git push origin sprint1:refs/for/sprint1
$ git push tags
#迭代负责人: 最后在icode上将sprint1分支合并回归到master主干后删除分支
```

[slide]
## 技术小组或topic独立开发工作流
``` sh
#RD: 基于master创建迭代分支sprint1并推送到服务器
$ git checkout -b topic1 master
$ git push -u

#RD: 开发功能，可能发生了若干次本地提交
$ git commit -m "功能开发"
...
$ git commit -m "功能提测"

#RD: 功能提测，rebase并在icode上merge回sprint
$ git pull
$ git rebase sprint1
$ git review story1 #git push origin story1:refs/for/story1
#RD: 继续跟进QA修改bug，或者开新的story分支开发新功能

#迭代负责人: 全功能集成提测并封版
$ git rebase master
$ git tag v7.2.0
$ git review sprint1 #git push origin sprint1:refs/for/sprint1
$ git push tags

#迭代负责人: 最后在icode上将sprint1分支合并回归到master主干后删除分支
```

[slide]
## 线上bug hot fix开发工作流
``` sh
#RD: 基于master创建迭代分支sprint1并推送到服务器
$ git checkout -b hotfix master
$ git push -u

#RD: 开发功能，可能发生了若干次本地提交
$ git commit -m "功能开发"
...
$ git commit -m "功能提测"

#RD: 功能提测
$ git pull
$ git review hotfix #git push origin hotfix:refs/for/hotfix

#RD: 封版发布
$ git tag v7.2.1-hotfix

#RD: 将更改合回迭代前rebase迭代分支的最新代码
$ git rebase sprint1

#迭代负责人: 在icode上合并hotfix代码后，全功能集成提测并封版
$ git rebase master
$ git review sprint1  #git push origin sprint1:refs/for/sprint1
$ git push tags
#迭代负责人: 最后在icode上将sprint1分支合并回归到master主干后删除分支
```

[slide]
## android studio gerrit plugin
android studio环境下与命令行环境下的操作并无二异；安装好gerrit插件后，只需在push对话框里勾选`push to gerrit`选项就好了；

[slide]
## 本地代码审核

1. 打开icode找到带审核提交的CHANGE_ID {:&.rollIn}
1. 切到与带审核提交同源的一个干净分支上
1. git review -d CHANGE_ID
1. 打开Android Studio查看审核代码

[slide]
# 谢谢