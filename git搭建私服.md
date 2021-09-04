

# 一、Git搭建私服

搭建Git服务器需要准备一台运行Linux的机器，在此我们使用CentOS 7。以下为安装步骤：

**1. 下载git源代码**

```
yum -y install git
```

**2. 添加用户**

```
adduser -r -c 'git version control' -d /home/git -m git
此命令执行后会创建/home/git目录作为git用户的主目录。
```

**3. 设置密码**

```
passwd git
需要输入两次密码
```

**4. 切换到git用户**

```
su git
```

**5. 创建一个文件夹作为仓库**

```
mkdir repo1
```

**6. 进入仓库，创建纯版本库**

```
cd repo1
git init --bare
```



### 第二种安装方式：使用git源码安装git

**1.  安装依赖包**

```
yum -y install curl curl-devel zlib-devel openssl-devel perl cpio expat-devel gettext-devel gcc cc
```

**2. 下载git源码**

```
wget https://github.com/git/git/archive/v2.3.0.zip
```

**3.  解压并安装**

```
（1）unzip v2.3.0.zip
（2）cd git-2.3.0
（3）autoconf  注：如果没有安装autoconf，则需要执行yum -y install autoconf-2.69-11.el7.noarch命令
（4）./configure
（5）make
（6）make install
```



# 二、连接服务器

**1. 在TortoiseGit中添加远端URL，添加远端**

<img src="https://i.loli.net/2021/08/11/LYDSEtOn5dIjqlF.png" alt="image-20210810222928445" style="zoom:80%;" />

**2.push文件到私有仓库**

<img src="https://i.loli.net/2021/08/11/8yuAIlqmE6tTWcd.png" alt="image-20210810223336027" style="zoom:80%;" />



**3.点击推送，需要输入git账号密码**

<img src="https://i.loli.net/2021/08/11/n7Ud4gFuv3bwVzq.png" alt="image-20210810223455169" style="zoom:80%;" />



