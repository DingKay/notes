# Git

## Git 基础

### 初始化仓库

使用命令初始化testGlobal仓库；`git init <仓库名>`

![初始化仓库](images\初始化testGlobal仓库.png)

查看当前初始化仓库的*gitconfig*配置（在项目目录下 *.git* 隐藏文件中的*config*）

![](images\查看testGlobal的gitconfig配置.png)

查看*git*全局配置（*window*：用户目录下的*.gitconfig*文件）

![](images\git的global配置.png)

### 修改config

`git config --global user.name`命令配置全局的*userName*

`git config --global user.email`配置全局的*email*

参数`--global`是修改全局的*.gitconfig*配置；

![](images\修改全局配置userName.png)

同理修改全局*email*配置；

### Git常见命令行操作

在空的*git*仓库中新建一个`test.txt`文件；

#### 查看状态

`git status`查看当前仓库中的文件状态

![](images\查看仓库文件状态.png)

* untracked files （新增文件且未加入到*git*管理中的文件）为红色

将新建文件加入到*git*的文件管理中

![](images\add命令.png)

* changes to be committed （修改文件后等待提交）为绿色

修改*test.txt*文件

![](images\修改test.txt文件.png)

> changes not staged for commit 提示test.txt新的变更阶段未添加

#### 新增文件

`git add test.txt`将新修改的*test.txt*文件再次添加

![](images\test.txt二次添加.png)

**git add <文件名，.. ；*>** 命令的参数可以是一个文件名，或多个文件名；

![](images\add新增多参数.png)

也可以使用`*`代表*添加所有*；

![](images\add新增所有.png)

#### 提交文件

`git commit -m "<commit message>"`提交文件并编辑提交说明信息*commit message*

![](images\第一次提交.png)

> `master`代表提交的分支是**master**分支
>
> * `root-commit`代表的是在哪吃提交的基础之上提交的，本次是第一次提交，故为**root-commit**
>
> `29f4bb1`为本次提交的哈希值，唯一标识
>
> `first message`代表本次的**commit message** 为本次提交描述信息

创建**demo.txt**文件第二次提交

![](images\第二次提交.png)

#### 查看历史

`git log`查看提交历史记录

![](images\查看提交历史.png)

> 在提交历史中可以看到完整的哈希值，以及提交者、日期等；

#### 配置作用域

查看本地的*config*配置

![](images\本地config配置.png)

本地的*config*配置中并没有**Author<email>**相关的配置，所以前几次的提交默认使用了**global**配置；

![](images\git的global配置.png)

修改本地的config配置

![](images\修改本地仓库的config文件.png)

修改*demo.txt*文件并再次提交，查看`git log`

![](images\修改本地config后提交日志.png)

> **config**文件的作用域：**local** > **global**

#### 忽略文件

新增*HelloWorld.java*文件，在本地*javac*编译后，生成了一个**.class**文件，但是我们不需要提交编译后的*class*文件

![](images\HelloWorld文件.png)

为了应对这样的情况有以下两种解决方案：

1. 修改**.git/info/exclude**文件（`*.class`表示过滤所有以**.class**后缀名结尾的文件，如果需要单独过滤一个文件则输入文件全名）

![](images\修改exclude文件.png)

修改完以后查看效果

![](images\修改exclude文件的前后对比.png)

> 该方法修改本地配置文件，不适用于多人协同开发的场景

2. 添加`.gitignore`文件

新增*ignore.bat*文件，如若我们想忽略此文件则在`.gitignore`文件中添加一行`ignore.bat`

![](images\新增ignore.bat文件.png)

将`.gitignore`文件提交至远程仓库，多人协同开发的场景下，则不需要单独修改`.git/info/exclude`文件

> `.gitignore`文件有许多的template模板；
>
> github地址： https://github.com/github/gitignore

#### 移除文件

当不小心*add*错了，或者一下子`git add *`将所有文件都添加到*git*中时，需要从待提交中提出文件

`git rm --cached <文件名>`

![](images\从stage中移除文件.png)

#### 撤销提交

