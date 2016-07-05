title: git work flow
speaker: 王佳
transition: slide3
theme: moon
usemathjax: yes

[slide]
# 百度手机助手项目 git work flow
## 环境配置
### 安装git和git-review子命令,配置git环境
``` sh
#安装git
$ sudo apt-get install git

#安装git-review自命令，win和osx用户请参考
#https://www.mediawiki.org/wiki/Gerrit/git-review
$ sudo apt-get install git-review

#配置全局用户信息（可选）
#一次性为全局用户统一配置用户信息
git config --global user.name wangjia20
git config --global user.email wangjia20@baidu.com
```

### Android Studio可在插件中心搜索安装`Gerrit`插件

[slide]
## clone 代码仓库,配置评审钩子
### 第一步 克隆公司仓库到本地
``` sh
$ git clone ssh://wangjia20@icode.baidu.com:8235/baidu/appsearch/playground 
```

### 第二步 配置审核信息
``` sh
#为gerrit设置提交评审的钩子，目的是为了让提交信息自动生成用于审核的id
scp -p -P 8235 wangjia20@icode.baidu.com:hooks/commit-msg playground/.git/hooks/
```

[slide]
## clone 代码仓库,配置评审钩子
### 第三步 配置用户信息

如果或者全局默认用户信息不是公司帐号，请设置个人信息

``` sh
#仅为本仓库配置用户信息
git config user.name wangjia20
git config user.email wangjia20@baidu.com
```
或者
``` sh
#一次性为全局用户统一配置用户信息
git config --global user.name wangjia20
git config --global user.email wangjia20@baidu.com
```

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