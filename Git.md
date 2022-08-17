## Git

#### 基本命令

```powershell
把文件从工作区添加到暂存区
  git add 文件（夹）路径
  git add .
删除文件
  git rm 文件路径
  把文件从暂存区中删除
  git rm --cached 文件路径
  git rm -r --cached 文件（夹）路径
把文件从暂存区提交到本地库
  git commit -m '提交说明' 文件（夹）路径
  git commit -m '提交说明'
撤销提交到本地库的commit
  git reset --soft HEAD^
查看版本信息
  git reflog
  git log #详细版本信息
穿梭版本
  git reset --hard 版本号
查看分支
  git branch
  git branch -v
  查看本地和远程的分支
  git branch -a 
创建分支
  git branch 分支名
切换分支
  git checkout 分支名
  创建一个新分支并切换过去
  git checkout -b 分支名
  游离状态的HEADS？
  git checkout 远程分支 #此时本地并没有这个分支
  如何解决？
  1. git fetch #获取所有远程分支
  2. git branch -v #查看所有分支
  3. git checkout -b 本地分支名 远程分支名 #拉取远程到本地
  4. git branch tmp commitId #根据远程分支的最后一次 commitId 创建临时分支
  5. git merge 本地分支名 tmp #合并临时分支
  6. git push
  7. git branch -D tmp #删除临时分支
删除分支
  git branch -D 分支名
合并分支（先切换到合并操作的分支）
  git merge 需要合并的分支名
合并后发现有冲突，手动处理后
  git add .
  git commit -m '提交说明' #不能带文件路径
推送本地库代码到远程库
  git push
  如果是新建的分支，在推送时需要设置上游分支
  git push --set-upstream origin（别名） 远程分支名
更新远程库代码到本地库
  git pull
克隆远程库到本地库
  git clone 远程库地址
  先初始化
  git init
  添加本地库与远程库的关联
  git remote add origin（别名） 远程库地址
```

#### 团队内协作

```
1. A 初始化一个本地库，并推送到远程库（代码托管中心）
2. B 从远程库克隆一份代码到本地库
3. B 对本地库的代码进行修改，然后再推送到远程库
4. A 拉取远程库的代码，就可以看到 B 修改的代码
```

#### 跨团队协作

```
1. C 需要对 A 的代码库进行一些修改，但 C 不是 A 团队的成员，没法直接推送代码
2. C 从 A 的远程库中 fork 一份新的远程库，然后克隆到本地库
3. C 修改本地库代码，并推送到自己的远程库
4. C 发送一个 Pull Request 到 A 的远程库
5. A 审核 C 的代码后，再进行代码合并
```

