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

`git init --bare <fileName>` 初始化项目仓库并查看该文件权限<br>
![](images/Linux/初始化仓库并查看权限.png)

目前该目录的权限是>root 用户 现在更改为>git 用户<br>
![](images/Linux/更改目录权限.png)

更改完毕后,本地进行`clone` 测试<br>
![](images/Linux/本地clone测试.png)

生成公钥以及密钥<br>
![](images/Linux/本地bash生成ssh文件.png)

查看生成的`ssh`文件<br>
![](images/Linux/ssh文件.png)

更改服务端的`ssh`配置<br>
