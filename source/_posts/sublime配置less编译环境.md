---
title: sublime配置less编译环境
categories:
  - 工具
tags:
  - sublime text3
  - less
abbrlink: 725c7ed5
date: 2017-08-11 19:18:32
---

> Less 是一门 CSS 预处理语言，它扩展了 CSS 语言，增加了变量、Mixin、函数等特性，使 CSS 更易维护和扩展。
> Less 可以运行在 Node 或浏览器端。

<!-- more -->

### **一、确认是否已安装 node.js**

### **二、安装 npm 或 cnpm(此步可略过)**
如果 npm 安装失败，可使用 cnpm。命令行输入
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
等待安装完成。

### **三、全局安装 less**
(提示：如果npm安装失败，请尝试使用cnpm安装，方法与npm相同，将那npm替换为cnpm即可)
命令行输入：
```
npm install less -g
```

检查less环境是否配置完成，命令行输入：
```
lessc -v
```
看是否能正确打印版本号。

### 四、**继续安装**
命令行输入：
```
npm install less-plugin-clean-css -g
```

### **五、在 sublime 安装插件 less2Css**
通过 package install 安装即可。

此时，创建的 less 文件在保存时可自动编译为相同名称的 css 文件。    

---

*另外有 koala app 可以直接编译 less 文件，而不必在 sublime 中安装插件*
*[去官网下载 koala app](http://koala-app.com/)*
