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
* `team1` `team2` `...` - 充当团队迭代开发分支，每次迭代开始前将master上的提交merge或rebase过来，每次迭代封版后，打tag并合回主干；
* `story1` `story2` `...` - 使用icafe创建story卡片对应的分支,单元测试完成后再合回迭代分支；
* `topic1` `topic2` `...` - topic认领或技术组同学独立开发分支，从master分支拉出，开发完毕后rebase方式合回任意迭代的开发分支，并删除分支；
* `hotfix` - 基于某次release版本的tag临时创建的分支，用于修复线上bug，重新封版发布后，打tag并合回主干，并删除分支；

[slide]
## 本地仓库分支结构
* `story1` `story2` `...` track origin/team1 - 用于开发迭代story1的本地分支，对应上游分支是team1；功能开发完成后rebase并push到远端的team1分支，通过审核后删除分支； {:&.rollIn}
* `team1` track origin/team1 - 迭代团队的开发开发，对应上游分支是team1；用于story提测后修复bug，如果story较小，也可用于开发story；
* `topic1` track origin/topic1 - topic认领或技术组同学独立开发分支，可以没有对应的上游分支，也可以对应上游分支topic1，开发完成后提交给上游的topic1分支或某迭代的开发分支；
* `hotfix` track origin/hotfix - 用于修复bug，对应上游分支是hotfix，重新封版后打tag回归主干。
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
```

[slide]
## 百度icode普通story开发工作流
``` sh
#基于迭代开发分支新建分支story1
$ git checkout -b story1 -t origin/team1

#开发功能，可能发生了若干次本地提交
$ git commit -m "功能开发"
...
$ git commit -m "功能提测"

#功能提测，代码合回开发分支
$ git review -r team1 #git push origin story1:refs/for/team1

#切会迭代分支修改bug跟进QA，提交修改
$ git checkout team1
$ git commit -m "修复bug"
$ git review  #git push origin team1:refs/for/team1

#合回代码并封版
$ git rebase master
$ git tag v7.2.0
$ git review  #git push origin team1:refs/for/team1
$ git push tags

#最后在icode上将team1分支合并回归到master主干
```
[slide]
## 关联icafe卡片的story开发工作流
``` sh
#基于迭代开发分支新建分支story1
$ git branch -b story1 -t origin/story1

#开发功能，可能发生了若干次本地提交
$ git commit -m "功能开发"
...
$ git commit -m "功能提测"

#功能提测，评审通过后
$ git pull
$ git review #git push origin story1:refs/for/story1
#在icode上将代码合回开发分支，并删除分支。

#切会迭代分支修改bug跟进QA，提交修改
$ git checkout team1
$ git pull
$ git commit -m "修复bug"
$ git review #git push origin team1:refs/for/team1

#合回代码并封版
$ git rebase master
$ git tag v7.2.0
$ git review #git push origin team1:refs/for/team1
$ git push tags

#最后在icode上将team1分支合并回归到master主干
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