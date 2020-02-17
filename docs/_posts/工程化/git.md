> 版本控制系统（VCS：version control system）

## 1. 版本演变历史

### 1.1 集中式VCS
* 有集中的版本管理服务器
* 具备文件版本管理和分支管理能力
* 集成效率有明显的提高
* 客户端必须时刻和服务器相连

### 1.2 分布式VCS

* 服务端和客户端都有完整的版本库
* 脱离服务端，客户端照样可以管理版本
* 查看历史和版本比较等操作，都不需要访问服务器，比集中式更能提高版本管理效率

## 2. git的优点

* 最优的存储能力
* 非凡的性能
* 开源
* 容易做备份
* 支持离线操作
* 很容易定制工作流程

## 3. 基本概念

### commit，tree，blob的关系

> blob是文件单元，但是它不认文件名，只认文件内容。即文件内容一样，git将之视为一个blob。commit可以看做根节点，blob相当于叶子节点来存储文件值，其他的都是tree节点

![](http://ww1.sinaimg.cn/large/006tNc79ly1g541ilg25fj30hz096tcc.jpg)

## remote，repo，stage，workspace

* remote远程仓库
* respo 本地仓库
* stage git追踪树/暂存区
* workspace 本地工作区

一般的工作流程是，首先`git status 查看变动的地方`，然后`git add .`将所有的修改提交至暂存区，然后是`git commit -m "msg"`，将暂存区的文件提交至仓库，最后`git push将本地仓库推送至远程仓库`

## 4. git操作

### 初始化

#### 初始化仓库

```
cd your_path
git init

or 

git init repo_name
cs repo_name
```

#### 初始化设置

```
git config --global user.name ‘your_name’
git config --global user.email ‘your_email@domain.com’
```

* --local 只针对某个仓库有效
* --global 当前用户的全部仓库有效
* --system  对系统所有登陆的用户有效

显示config的配置，加--list

* git config --list --local
* git config --list --global
* git config --list --system


### 暂存区

#### 添加至暂存区

```
git add file_path 

进阶

git add -u: 将文件的修改、文件的删除，添加到暂存区。(update)
git add . :  将文件的修改，文件的新建，添加到暂存区。
git add -A: 将文件的修改，文件的删除，文件的新建，添加到暂存区 (= git add -all)
```

> 同理，删除则用`rm`

#### 查看暂存区状态变更

```
git status （仅显示哪些文件发生变更）

or 

git diff (查看更多的详细信息，可以具体到文件内部)
```

#### 重命名暂存区文件

```
git mv ori_name tar_name
```

### 版本库

#### 添加至版本库

```
git commit -m "desc"
```

#### 更正上次提交信息

```
git commit --amend -m "new desc"
```

#### 漏提交

可以选择再提交一次，或者git add。然后再git commit

#### 查看提交历史

```
git log

git log --oneline (以一种非常简洁的方式呈现历史版本)
git log --graph (图形的形式)
git log -n4/-4 (查看最近4条）
git log --reverse (逆序的形式)
git log --autho=“autho_name”
```

### 回滚版本号

#### 针对暂存区stage

##### git checkout

这个是针对暂存区的（暂存区就是一个画布，它没有版本号，版本号是仓库才有的），比如说我们工作区的内容有问题，我们想暂时废弃，直接采用暂存区的内容

`git checkout -- <filename>`

##### git reset file 

如果我们已经将修改内容git add至了暂存区，我们需要将暂存区内容废弃，我们就可以先reset，然后再checkout

`git reset <filename>
 git checkout -- <filename>
`

#### 针对本地仓库

##### git reset

> 比如说有版本1，版本2，版本3，那么我们reset到了版本1，版本23都会消失

```
git reset --hard 版本号
```

!> 注意，这个时候直接Git push会出错，因为我们本地库指向的版本可能比远程库的旧，我们使用git push -f 强制推上去即可

##### git revert

> 比如说有版本1，版本2，版本3，那么我们revert了版本2，就会生成一个没有版本2但是有版本3的版本4

![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTgwNDE0MjA1ODE2MTg4)

> 总结，即`reset`为回到某一特定版本，而`revert`为回滚某一次特定操作

##### git revert vs git reset

* git revert是新增一个commit，git reset是删除指定版本和之后的任何版本
* git reset是版本号HEAD向后挪动，而git revert是向前滚，我们可以理解成commit了一个完全和指定版本逆转的版本，所以它叫做revert

### 分支

#### 列出分支

```
git branch
```

#### 创建分支

```
git branch <branch_name>
```

#### 切换分支

```
git checkout <branch_name>

git branch -b <branch_name> 可以创建新分支并且立刻切换到该分支之下
```

#### 删除分支

```
git branch -d <branch_name>
```

#### 合并分支

```
git merge <branch_name>
```

### 远程仓库

#### 查看关联的远程仓库

```
git remote // 查看远程仓库的名称

git remote -v // 查看远程仓库的详细信息
```

#### 添加远程仓库

> 注意，这里的url需要是仓库的名称

```
git remote add origin <url>
```
#### 推送至远程仓库

```
git push <remote> <local>:<remote>
```

#### git fecth 

将某个远程主机的更新，全部/分治取回本地，注意它是存在repo中，对workspace无影响，如需彻底更新需要合并或使用git pull

#### git pull

拉取远程主机的更新，并且合并至本地分支

#### git pull --rebase

```
git pull --rebase
```

这个操作只要是解决git pull在git log里线路混乱赘余的问题

> 大多数时候，这个操作是为了让提交线图更好看，从而方便code review。同时这样的操作可能更容易导致产生冲突，如果预期冲突比较多，建议直接pull

## github

### 搜索技巧

可以参考这篇[官方解释](https://help.github.com/en/articles/searching-for-repositories)

#### in:readme

搜索关键字之后加入这个字段，即可

eg. `vue.js in:readme`

### filename:.gitlab-ci.yml

即可搜索

## GitLab

* 提供免费的社区版本
* 包含全套的DevOps





