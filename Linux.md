# Linux

## 常用

1.更改系统hostname<br>
> hostnamectl set-hostname `<new hostname>`

使用该命令修改`hostname` 并且重启生效<br>
![修改hostname](images/Linux/修改hostname-1.png)

修改后查看`hosts`文件<br>
![](images/Linux/修改hostname-2.png)

该命令不会修改`/etc/hosts`文件中的`hostname`需要手动修改文件

---

2.搭建Git环境
> yum install -y git

使用 `yum` 命令安装<br>
![](images/Linux/yum安装git.png)

安装后使用 `git --version` 查看版本信息<br>
![](images/Linux/查看git版本-success.png)<br>
则安装成功
![](images/Linux/查看git版本-fail.png)<br>
则安装失败

查询是否存在用户,不存在则创建用户git用于仓库管理(UserName:git)<br>
![](/images/Linux/新建git用户并设置密码.png)

创建文件夹,作为git的仓库![](images/Linux/创建git仓库.png)

在git仓库目录下,再创建项目仓库目录<br>
![](images/Linux/创建项目仓库.png)

> git init --bare `<fileName>`

初始化项目仓库并查看该文件权限<br>
![](images/Linux/初始化仓库并查看权限.png)

目前该目录的权限是>root 用户 现在更改为>git 用户<br>
![](images/Linux/更改目录权限.png)

更改完毕后,本地进行`clone` 测试<br>
![](images/Linux/本地clone测试.png)

生成公钥以及密钥<br>
![](images/Linux/本地bash生成ssh文件.png)

查看生成的`ssh`文件<br>
![](images/Linux/ssh文件.png)

进入`ssh`目录更改服务端的`ssh`配置<br>
> cd /etc/ssh

![](images/Linux/进入ssh路径.png)

> vim sshd_config

修改`ssh`配置文件,注释其中三条配置<br>
![](images/Linux/注释ssh配置文件.png)

查看`sshd`服务的状态,并重启.此时,查看`~`目录下的`.ssh`目录

![](images/Linux/查看.ssh目录.png)

如果你需要把项目中的某一个文件夹作为项目目录，你需要把服务器上的公钥配置到git用户的权限之下，也就是我们创建的`git`用户的`.ssh`中的`authorized_keys`

创建git用户下的`.ssh`文件,并且修改权限组<br>
![](images/Linux/创建git用户下的.ssh文件.png)

将本地之前生成的密钥上传至服务器<br>
![](images/Linux/本地密钥写入服务器.png)

服务器上查看`.ssh`目录<br>
![](images/Linux/本地上传的密钥.png)

修改权限很重要,使用`chmod 700 .ssh`命令修改目录权限,进入`.ssh`使用`chmod 600 authorized_keys`修改刚刚上传的认证密钥文件的权限<br>
修改后的:
![](images/Linux/修改.ssh文件权限.png)<br>
![](images/Linux/修改.ssh目录下的认证秘钥文件权限.png)

现在从服务器上面`clone`就不需要密码了.
