# Git

`Git`是一个分布式版本控制系统

## `Git`的核心作用

1. **版本控制**：记录每次代码修改（提交），包括内容、时间、作者；可回溯到任意历史版本
   
2. **分支管理**：支持并行开发；分支轻量且独立，合并时可智能处理冲突

3. **协作开发​**：在`GitHub`等平台上建立中央远程仓库；开发者完成本地工作后，可以把自己的更改“推送”到远程仓库；也可以从远程仓库“拉取”别人的最新更改，保持同步
   
4. **​代码保存**：代码保存并备份在远程仓库，且开发者的本地仓库都是一个完整的备份，极大地降低了项目数据丢失的风险

## 基础命令

###  `git init`

`git init`: 初始化，将​​当前目录变成一个`Git`仓库

效果：在项目根目录下生成一个名为`.git`的隐藏文件夹（`Git`的​​核心数据库​），存储所有版本控制所需的元数据和对象

```bash
.git/
├── HEAD          # 指向当前所在的分支（默认初始化为 `refs/heads/main`）
├── config        # 本地仓库的配置文件（如远程仓库地址、用户信息等）
├── objects/      # 存储所有 Git 对象（提交、树、blob）
├── refs/         # 存储分支和标签的指针
│   ├── heads/    # 本地分支指针
│   └── tags/     # 标签指针
└── hooks/        # 客户端或服务端的钩子脚本（默认有示例脚本）
```

### `git add`

`git add`：将工作目录的变更添加到暂存区（`Staging Area`）​​，为后续提交做准备
```bash
touch new-file.txt        # 创建新文件
git add new-file.txt      # 将新文件加入暂存区

echo "Update" >> existing-file.txt  # 修改文件
git add existing-file.txt            # 暂存修改
```


###  `git commit`

`git commit`：将暂存区的变更保存到版本历史中​​
```bash
# 1. 修改文件
echo "Hello World" > file.txt

# 2. 将变更添加到暂存区（临时存储）
git add file.txt

# 3. 提交版本
git commit -m "Add welcome message"
```


## 远程

### `git remote`

`git remote add <name> <url>`: 为已存在的本地仓库添加一个新的远程仓库关联
```bash
git remote add origin https://github.com/user/new-repo.git
```

`git remote -v`: 列出所有已配置的远程仓库名称及其对应的URL地址
```bash
$ git remote add github https://github.com/user/repo.git
$ git remote add gitee https://gitee.com/user/repo.git
$ git remote -v
github  https://github.com/user/repo.git (fetch)
github  https://github.com/user/repo.git (push)
gitee   https://gitee.com/user/repo.git (fetch)
gitee   https://gitee.com/user/repo.git (push)
```

`git remote set-url <name> <new-url>`: 修改远程URL

`git remote remove <name>`: 删除远程仓库

### `git clone`

`git clone <url>`: 克隆远程仓库到本地
```bash
git clone git@codeup.aliyun.com:5f3f374f6207a1a8b17f933f/OasisCaliber.git
```
等效于
```bash
mkdir 远程仓库名 && cd 远程仓库名

git init
git remote add origin git@codeup.aliyun.com:5f3f374f6207a1a8b17f933f/OasisCaliber.git
git fetch origin
git checkout -b main origin/main
```
















```bash
git fetch

git pull origin master
git pull --rebase origin master

git push origin master 

git checkout master

git log --oneline

git reset --hard 5896023
git rebase -i 66b63e7
```

```bash
git status # 查看git状态，包括当前所处分支、文件状态等

$ git stash # 将当前工作目录中的修改（包括已暂存和未暂存的更改）临时保存到一个「堆栈」中，让工作目录恢复到干净的状态​​。
Saved working directory and index state WIP on (no branch): 9117e0ac 实现pcap切片20s并补帧的操作 
# Saved working directory and index state 表示
# ​​on (no branch): 9117e0ac 当时所处commit的哈希值​​。Git 把这个信息也存储起来，作为上下文记录。
git stash pop # 应用并删除​​最近的一次存储
git stash apply # 应用最近的一次存储，但​​不删除​​它
git stash drop # git stash drop：删除最近的一次存储，不应用
git stash clear # 清空整个存储堆栈。

$ git branch -a # 查看所有分支
* (HEAD detached at 9117e0ac)
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/HW_RosCollect/DongFengYueXiang
  remotes/origin/HW_RosCollector
  remotes/origin/HW_RosCollector_TMP
  remotes/origin/RosCollector2.0
  remotes/origin/RosCollector2.0-czj
  remotes/origin/RosCollector2.0-dev
  remotes/origin/ZhiDi
  remotes/origin/master
  remotes/origin/xzj_dev

# 当从远程仓库拉项目时，会自动创建本地的主分支对应云上的主分支
git branch -vv
* HW_RosCollector_TMP 2511768c [origin/HW_RosCollector_TMP: ahead 1] 实现: 数据有效性验证模块(validateData)
  master              25f01e8b [origin/master] fix: pcap CPU占用bug

# 如果想要在远程仓库上的某分支进行开发，正确的做法是: 基于该远程仓库上的分支，在本地创建一个(同名的)新分支，并切换到这个新创建的本地分支上
git checkout -b HW_RosCollector_TMP origin/HW_RosCollector_TMP # -b: 创建并切换

# git checkout既可以切换到某个分支，也可以切换到某次提交。
# 切换分支是相对安全的，直接切换到提交则不是。
# 因为直接 git checkout <commit-hash> 切换到某次提交，会导致头指针分离
# 直接切换到提交的使用场景只有: 只是想临时性地、只读地查看一下项目在某个历史提交点的状态​​，看完之后你并不打算做任何修改，或者即使修改也绝不打算保存

# 最佳实践: 确保你在一个稳定的分支上​，​然后，基于这个分支创建一个新的临时分支，并立即切换到该提交​​。
​git checkout main
git checkout -b my-temp-branch <commit-hash>

```