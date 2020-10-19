---
title: 两个hexo博客的多端同步管理与推送问题
categories:
  - 工具
tags:
  - hexo
abbrlink: 49ec609a
---

> 在一个电脑上管理两个或多个hexo博客，在推送和多端同步时会再现一些问题，比如凭证覆盖、远程下载上传等

<!-- more -->

建立不同的hexo博客需要不同的github账号，创建过程较为简单，比较伤脑筋的首先是多端同步问题。



## 一、hexo博客的多端同步问题

在使用hexo博客时，通常不会总是在同一台电脑上发文章，可能会在不同的地方有发文章的需求，拷贝既不快又不方便，因此需要源文件同步。

#### 博客源文件同步

写完文章部署到网站

``` 
hexo clean
hexo g
hexo s // 本地查看效果，可略过
hexo d // 将本地源文件推送到github仓库
```

再进行如下操作

```
git add .             	   // 所有变化提交到暂存区
git commit -m "新增xxx文章"	// 提交文件
git push origin hexo      // 推送hexo分支
```

这就将源文件上传到github了。

github上建立一个分支hexo，建议将其设置为默认分支，用来保存环境文件，master用来保存文章相关内容。



#### 在其他电脑上同步博客

电脑上安装node和git，然后下载github博客文章与环境文件，执行如下操作

```
npm install hexo-cli -g // 先安装hexo的脚手架
git clone git@github.com:xxx/xxx.github.io.git // 下载项目，因为hexo是默认分支，所以这里直接会下载hexo分支
npm i // 安装依赖
hexo s // 启动服务器
```

此时己经完成了同步下载博客，之后在其他电脑上写完文章后，在博客根目录使用hexo推送文章，然后执行如下操作

```
git init
git remote add origin git@github.com:xxx/xxx.github.io.git  // 加远程仓库,注意这里要添加你自己的仓库,xxx即用户名
git checkout -b hexo            							// 新建hexo分支并切换到hexo分支
git add .              										// 所有变化提交到暂存区
git commit -m "解决同步问题"  								// 提交文件
git push origin hexo   										// 推送hexo分支
```

如果该电脑不止一次使用过，则只需从 git add . 开始即可。

这样就完成了多端同步，其核心即下载仓库、写文章、推送文章。

在下载仓库时，也可以使用git pull，效果与git clone类似，都是下载项目。



<mark>提示：在建立github博客时，最好先创建hexo分支，在推送文章时，即按照以上步骤，将环境文件保存于hexo分支，将文章相关推送到master</mark>



## 二、SSH公钥管理

**同一台电脑上管理两个hexo博客时，ssh绑定github账号有时会出现问题，导致无法推送文章，其原理为：**

1）SSH的公钥是GitHub作为本地仓库和远程仓库连接的唯一标识，一个公钥只能对应一个GitHub账户，如果将一个相同的公钥上传到不同的GitHub账户，GitHub则无法做出辨识，进而导致错误。

 2）一台电脑，可以生成多对公私钥，可以通过配置，将不同的公钥上传到不同的GitHub账号，那么就不存在单个公钥绑定多个GitHub账号的情况存在了。



**相关报错信息为：**

1）同一台电脑部署第二个hexo博客执行`hexo g -d`时报错：`ERROR: Permission to xxxxxx/xxxxxx.github.io.git denied to xxxxxx.`

2）添加新的 SSH 密钥 到 SSH agent 执行`ssh-add xxx`时报错：`Could not open a connection to your authentication agent.`

3）单独设置用户名/邮箱时报错：`fatal: not in a git directory`



解决这些问题，需要操作ssh公钥。

如果没有创建公钥，则操作如下：

#### 查看当前密钥

```
ls ~/.ssh/     // 打开终端，可以查看当前已存在的密钥。
```



#### 创建新密钥

进入SSH根目录

```
cd ~/.ssh/
```

**创建密钥**

方法一：

```
ssh-keygen -t rsa -f  ~/.ssh/新密钥名称 -C "github账号邮箱"  // 注意密钥名称不能相同
```

按步骤设定密码。



方法二：

```
ssh-keygen -t rsa -C "github账号邮箱"
```

回车后出现：

```
Generating public/private rsa key pair.  
Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): 
```

注意此时需要输入新密钥的名称，注意区别新密钥名称和旧密钥名称，不要相同。之后再两次回车，新密钥创建完毕。



#### 配置config

在.ssh/根目路下，查看是否存在config文件，如果没有就创建一个。（我的根目录为: C:\Users\Lenovo.ssh）

``` 
touch config    // 要么用这个命令创建，要么手动创建
```

在config文件中配置如下：

```
// 第一个账号，默认使用的账号，不用做任何更改
Host xxx.github.com
	HostName github.com
	User git
	IdentityFile ~/.ssh/新密钥的名称1
	
// 第二个新账号，"xxx"为前缀名，可以任意设置，需要用到
Host xxx.github.com
	HostName github.com
	User git
	IdentityFile ~/.ssh/新密钥的名称2
```

复制创建的公钥内容：

```
clip < ~/.ssh/创建的新密钥的名称.pub
```

或者可以用记事本打开xxx.pub并复制内容，将其分别填入对应的github账户的setting的SSH and GPG keys的New SSH key中。



#### 清空本地的 SSH 缓存，添加新的 SSH 密钥 到 SSH agent中

在.ssh/根目录下，依次执行：

```
ssh-add -D
ssh-add xxxxxx #旧密钥名称，一般是id_rsa，或之前的账号创建的
ssh-add xxxxxx #新创建的密钥名称
```

如果执行以上命令出现错误：`Could not open a connection to your authentication agent.`，那么就需要先执行`ssh-agent bash`，再执行以上命令。



#### 验证配置是否成功

```
ssh -T git@github.com    // 默认ssh_key验证
ssh -T git@xxxgithub.com // 新的ssh_key验证，其中“xxx”为config文件中的命名
```

如果显示以下信息，就说明配置成功：

```
Hi 你的用户名! You've successfully authenticated, but GitHub does not provide shell access.
```



#### 取消全局用户名/邮箱配置，单独设置用户名/邮箱

执行如下操作：

```
git config --global --unset user.name
git config --global --unset user.email
```

分别进入两个hexo博客.git目录下执行以下命令单独设置用户名/邮箱：（.git的具体路径：\Hexo\.deploy_git\.git，该目录是隐藏的）

```
git config user.name "这里是用户名"
git config user.email "这里是邮箱"
```

如果此时报错：`fatal: not in a git directory`，说明没有进入.git目录下。

执行以下命令，查看是否成功：

```
git config --list
```

到hexo配置文件中填写_config.yml文件，在repository中写入: `git@github.com:用户名/用户名.github.io.git`



#### 三、git报错ssh: connect to host github.com port 22: Connection timed out

查看SSH是否能连接成功

```
ssh -T git@xxx.github.com
```

如果还是报这个错，就先查看公钥文件是否存在，如果存在，则需要修改config内容：

```
Host xxx.github.com
    Hostname ssh.github.com
    User x@foxmail.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/xxx
    port 443

Host yyy.github.com
    HostName ssh.github.com
    User xx@foxmail.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/yyy
    port 443
```

再查看是否连接成功：

```
ssh -T git@xxx.github.com
```

出现提示直接回车yes即可。