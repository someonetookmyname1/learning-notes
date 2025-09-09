# Git


`Git`是一个分布式版本控制系统


## Git的核心作用


1. **版本控制**：记录每次代码修改（提交），包括内容、时间、作者；可回溯到任意历史版本
   
2. **分支管理**：支持并行开发；分支轻量且独立，合并时可智能处理冲突

3. **协作开发​**：在`GitHub`等平台上建立中央远程仓库；开发者完成本地工作后，可以把自己的更改“推送”到远程仓库；也可以从远程仓库“拉取”别人的最新更改，保持同步
   
4. **​代码保存**：代码保存并备份在远程仓库，且开发者的本地仓库都是一个完整的备份，极大地降低了项目数据丢失的风险


## 存储本质


### 结构


`Git`将所有版本控制数据存储为​​四种类型（`​​Blob`、`Tree`、`Commit`、`Tag`）的对象​​，所有对象：

- 通过`​​SHA-1`哈希值​​唯一标识（40位十六进制字符串，如`f24f6c8d`...）

- 以​​压缩格式​​存储（节省磁盘空间）

- 采用​​内容寻址存储​​：内容相同则哈希值相同，重复文件只存一次

所有使用`Git`的项目目录下，都有一个名为`.git`的隐藏目录（`Git`的​​核心数据库​），其存储所有版本控制所需的元数据和对象

`.git`目录其结构示例：
```bash
.git/
├── HEAD          # 指向当前所在的分支（默认初始化为 `refs/heads/main`）
├── config        # 本地仓库的配置文件（如远程仓库地址、用户信息等）
├── objects/      # 存储所有 Git 对象（提交、树、blob）
├── refs/         # 存储分支和标签的指针
│   ├── heads/    # 本地分支指针
│   └── tags/     # 标签指针
└── hooks/        # 客户端或服务端的钩子脚本（默认有示例脚本）

# 其中：
.git/objects
├── f2/        # 哈希值的前两位作为目录名
│   └── 4f6c8d... # 完整哈希值 = f2 + 4f6c8d... → f24f6c8d...
├── info/     # 存储对象额外信息
└── pack/     # 打包压缩后的对象（优化存储）
```


### 四种核心Git对象


一、**`​​Blob`对象​（`Binary Large Object`）**

- **​​存储内容**​​：​文件快照，即​​文件的实际数据​​（不含文件名、权限等元数据）

- ​**​创建时机​**​：`git add`时生成

二、**`Tree`对象**
   
- **​​存储内容​**​：​目录结构的快照​
  
  - 包含文件列表：文件名、权限、对应`​​Blob`的`SHA-1`

  - 或子目录的`Tree`对象引用

- **​​创建时机​**​：执行`git commit`时生成

三、**`Commit`对象**
   
- ​**​存储内容​**​：​​​提交的元数据​
  
  - 顶层`Tree`对象的`SHA-1`（代表项目快照）

  - 父提交的`SHA-1`（首次提交无此项）

  - 作者/提交者信息 + 时间戳 + 提交消息

- ​**​创建时机​**​：执行`git commit`时生成

四、**`Tag`对象（可选）**
   
- **​​存储内容​**​：标签的引用（如版本发布标记）

- ​​**创建时机​**​：`git tag -a v1.0`


### 实例说明：Git对象的创建和工作原理


#### ​步骤1：项目初始化​


- **命令**：`git init`

- **作用**: 初始化，将​​当前目录变成一个`Git`仓库

- **实例**：
```bash
# 创建项目目录
mkdir myproject
cd myproject

# 初始化 Git 仓库
git init
```

- **效果**：在项目根目录下生成`.git`目录，但`objects/`目录还是空的


#### ​步骤2：创建并添加文件​​


- **命令**：`git add`

- **作用**: 将工作目录的变更添加到暂存区（`Staging Area`）​​，为后续提交做准备

- **实例**：
```bash
# 创建第一个文件
echo "Hello, World!" > hello.txt

# 添加到暂存区
git add hello.txt
```

- **效果**：`Git`读取`hello.txt`的内容："Hello, World!"，并计算内容的`SHA-1`哈希（假设为`60f1d2d...`），然后创建`​​Blob`对象，存储在`.git/objects/60/f1d2d`​：
```bash
echo -n "Hello, World!" | git hash-object --stdin
# 输出：60f1d2d
```

- **`.git/objects`目录结构**：
```bash
.git/objects
├── 60/  # ​​Blob对象1       
│   └── f1d2d...
└── ...
```

#### ​步骤3：添加第二个文件​


- **实例**：
```bash
# 创建第二个文件
echo "# My Project" > README.md

# 添加到暂存区
git add README.md
```

- **`.git/objects`目录结构**：
```bash
.git/objects
├── 60/  # ​​Blob对象1       
│   └── f1d2d...
├── 8c/  # ​​Blob对象2       
│   └── 9d07...
└── ...
```


#### ​步骤4：创建提交​


- **命令**：`git commit`

- **作用**: 将暂存区的变更保存到版本历史中​​

- **实例**：
```bash
git commit -m "Initial commit"
```

- **效果**：

1. 创建当前目录结构快照，计算其`SHA-1`（假设为`f24f6c8d...`），然后创建`​​Tree`对象，存储在`.git/objects/f2/4f6c8d`​：
```bash
# 数据结构：
100644 blob 60f1d2d... hello.txt
100644 blob 8c9d07... README.md
```

1. 创建`Commit`内容，计算其`SHA-1`（假设为`1a2b3c4d...`），然后创建`​Commit`对象，存储在`.git/objects/1a/2b3c4d`​：
```bash
# 数据结构：
tree f24f6c8d...        # 指向刚创建的 Tree
author You <you@test.com> 1690640000 +0800
committer You <you@test.com> 1690640000 +0800

Initial commit
```

3. 更新分支指针
```bash
# 查看 .git/refs/heads/main
cat .git/refs/heads/main
# 输出：1a2b3c4d... (新创建的 Commit SHA)
```

- **`.git/objects`目录结构**：
```bash
.git/objects/
├── 8c/               # Blob 对象1 (README.md)
│   └── 9d07f4...     # 内容: "# My Project"
├── 60/               # Blob 对象2 (hello.txt v1)
│   └── f1d2d3...     # 内容: "Hello, World!"
├── f2/               # Tree 对象
│   └── 4f6c8d...     # 指向两个Blob
├── 1a/               # Commit 对象
│   └── 2b3c4d...     # 指向Tree
└── ...
```


#### ​步骤5：修改文件并提交​


- **实例**：
```bash
# 修改文件
echo "Hello, Git!" > hello.txt  # 更新内容
git add hello.txt
git commit -m "Update greeting"
```

- **效果**：

1. 创建新`Blob`：新内容"Hello, Git!"，生成新`Blob`（假设`SHA-1`为`5d308e9a...`）

2. 创建新`Tree`：创建当前目录结构快照，生成新`​​Tree`（假设`SHA-1`为`9a0b1c2d...`）
```bash
# 修改文件
100644 blob 5d308e9a... hello.txt    # 更新为新的 Blob
100644 blob 8c9d07... README.md    # 未修改，复用旧 Blob
```

3. 创建新`Commit`：生成新`Commit`（假设`SHA`为`5d6e7f8a...`）
```bash
tree 9a0b1c2d...             # 新 Tree
parent 1a2b3c4d...         # 指向父提交
author ... 
committer ...

Update greeting
```

4. 更新分支指针
```bash
cat .git/refs/heads/main
# 现在指向 5d6e7f8a...
```

- **`.git/objects`目录结构**：
```bash
.git/objects/
# ==== 第一次提交的对象====
├── 8c/               # Blob：README.md（未修改）
│   └── 9d07f4...     # 同时被新Tree和旧Tree引用
├── 60/               # Blob：hello.txt v1（旧版本）
│   └── f1d2d3...     # 不再被最新Tree引用，但被旧Tree引用
├── f2/               # Tree：旧项目结构
│   └── 4f6c8d...     # 被旧Commit引用
├── 1a/               # Commit：初始提交
│   └── 2b3c4d...     # 被新Commit作为父提交引用

# ==== 第二次提交新增的对象 ====
├── 5d/               # 新Blob：hello.txt v2
│   └── 308e9a...     # 内容: "Hello, Git!"（新内容）
├── 9a/               # 新Tree：更新后的项目结构
│   └── 0b1c2d...     # 指向：新hello.txt v2 + 旧README.md
├── 5d/               # 新Commit：更新提交
│   └── 6e7f8a...     # 指向：新Tree + 父提交(1a2b3c4d)
└── ...
```


### 注意


1. **`Git`的每次提交，存储的是​目录级别的快照（`snapshot`），而非代码行级的增量（`delta`），但底层会通过`​增量压缩算法（packfiles）​​`优化存储**

2. **若某次`commit`中，`git add`了一个新文件，则`GIT`会存储该文件的完整内容为一个`​​Blob`对象；假设后续的`commit`中，该文件内容一直不变，则生成的`SHA-1`哈希值也不变，`commit`也会一直复用初次生成的`​​Blob`对象；一旦该文件内容发生了变化，则生成的`SHA-1`哈希值也会改变，`GIT`就会创建全新的`​​Blob`对象**


















## 分支

分支的本质是指向某个提交（`commit`）的引用，仅需41字节存储（40位`SHA-1` + 换行符）

### `git branch`

`git branch`：`Git`中管理代码开发线的核心命令，用于​​创建、查看和删除分支​​。

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