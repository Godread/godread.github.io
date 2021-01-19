---
title: 一个电脑使用多个github帐号的SSH key
categories: 工具
tags:
  - github
  - SSH
toc: true
abbrlink: 7a6a6b5a
---

> 一个电脑上要使用多个github账号上传到不同的仓库时，如何建立SSH Key连接

#### **建立不同账号的SSH Key**

首先创建SSH Key，如下

```
$ ssh-keygen -t rsa -C "youremail@xxx.com"
```

如果本地没有ssh的话，一直按enter键，直到不再提示，本地/.ssh文件夹会生成id_rsa和id_rsa.pub两个文件。用编辑器或文本打开id_rsa.pub的内容，添加到github的SSH Key中。



其次创建另一个或多个SSH Key，创建命令同上但要注意，由于刚才已经创建了一个SSH Key，所以这时创建新的SSH Key时必须加以区分，需要在命令回车后取个名字，之后一直enter即可创建新的SSH Key，同时也生成两个新的文件。此时，/.ssh文件夹下面会有5个文件，known_hosts会自动出现。



注意，创建完成之后，一定要把不同的SSH Key添加到对应的github或gitlab等账号的SSH Key中去。



为了让不同的远程host不冲突，因此需要新建一个config文件，该文件只叫config，没有后缀，其内容如下：

```
Host github.com
User email1@xxx.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443

Host b.github.com
User email1@xxx.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_new
Port 443
```

Host名字可随意取。



其他SSH Key创建过程如上所示。创建完成之后，可以用以下方式测试不同的

```
$ ssh -T git@github.com    // 默认的测试方式
```

对应不同的名称，要用Host的名称来区分测试

```
$ ssh -T git@b.github.com    // 注意b.github.com要对应名称
```

<br>

#### **不同情况下SSH Key的处理方式**

##### 本地已经创建或已经clone到本地

a) 解决方式一：打开config文件

```
#更改[remote "origin"]项中的url中的
#b.github.com 对应上面配置的Hhost
[remote "origin"]
	url = git@b.github.com:itmyline/blog.git
```

b) 解决方式二： 在Git Bash中提交的时候修改remote：

```
$ git remote rm origin
$ git remote add origin git@b.github.com:itmyline/blog.git
```



##### clone仓库时对应配置host对应的账户

```
#b.github.com对应一个账号
git clone git@b.github.com:username/repo.git
```



##### 最后一个要注意的

在配置网站的连接url时，要注意使用SSH，不用HTTPS

```
// github的SSH原url
git@github.com:githubname/githubname.github.io.git

// 需要改为
git@b.github.com:githubname/githubname.github.io.git    // 注意config文件中Host的名称要对应，否则连接无法成功
```

