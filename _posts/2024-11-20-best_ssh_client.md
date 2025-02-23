---
layout: post
title: "终于找到了完美的SSH客户端：XPipe + KeePass"
date:   2024-11-20
tags: [tool]
comments: true
author: kamust
---

程序员如何选取ssh客户端是一个老生常谈的问题，近日我关注到一款名为 XPipe 的软件，经过一段时间的使用我认为终于找到了这个问题的完美答案。XPipe是一款开源ssh连接管理器，同时具备sfpt图形界面，还可以编写自定义连接脚本，具有很高的自由度。KeePass 是一款开源的密码管理软件，同时还可用于加密其他文件，如ssh密钥。我的方案是使用 XPipe 配置连接信息，同时将密码和密钥存储在KeePass中实现更高的安全性。

## 软件地址

[XPipe 官网](https://github.com/xpipe-io/xpipe/)

[KeePass 官网](https://keepass.info/)

## 主要优点

+ 完全开源：KeePass和xpipe都是开源软件。
+ 自定义终端：大多数ssh工具的终端我都不太满意，而xpipe允许用户使用自己的终端，例如在windows上使用wt，在linux上使用kitty。
+ 自建同步服务：用户可以通过自建WebDAV和Git服务用于同步所有配置信息，还可基于Git实现团队共享配置，避免依赖特定的在线服务。
+ 密码和密钥的加密存储：KeePass提供足够安全的加密功能，确保用户的密码和密钥安全。
+ 跨平台支持：KeePass和XPipe在Windows、Linux、MacOS 上使用均有官方支持。

## 目前存在的缺点

目前，XPipe仅支持SFTP图形化文件管理，不支持SCP、FTP等协议，这可能会限制某些用户的使用场景，但是用户可以通过自定义脚本调用其他软件来实现对SCP和FTP的支持。

## 配置步骤

### 1. 安装 XPipe、KeePass

根据官网提供的安装方式自行安装两个软件。

### 2. 创建KeePass数据库

点击Keepass中的新建按钮，选择要保存数据库文件的位置，输入加密密码，可以在更多选项中提高加密难度。

![new_db](https://kamust.github.io/images/best_ssh_client/new_db.png)

### 3 使用ssh agent

#### 3.1 安装插件

访问[KeePass插件页面](https://keepass.info/plugins.html)，下载keeagent插件，用于代理ssh密钥。回到KeePass软件，点击菜单中的Tool>Plugins打开插件管理，点击下方的 Open Folder，把下载好的插件放到文件夹内，然后重启KeePass。

windows系统还需要点击Tool>Options>KeeAgent，找到 “Enable agent for windows OpenSSH”，勾选并确认。

### 3.1 在KeePass中添加密钥

在KeePass中新建一个条目，把你的ssh密钥作为附件添加

![new_db](https://kamust.github.io/images/best_ssh_client/add_key.png)

然后在KeeAgent选单中勾选允许使用此条目，在"Manage Key Files" 中选择刚刚添加的密钥。

![new_db](https://kamust.github.io/images/best_ssh_client/add_key_2.png)

添加完成后需重启KeePass

### 3.2 通过密钥对连接ssh

在 Xpipe 的主页中添加ssh连接，在密钥中选择 SSH-Agent。

![new_db](https://kamust.github.io/images/best_ssh_client/key_conn.png)

### 4 通过KeePass中的密码连接ssh，仅限windows

#### 4.1 安装插件

下载KeePassCommander 用于为Xpipe提供登录密码。找到KeePass安装目录，把 KeePassCommander.exe以及dll文件解压到安装目录，使得 KeePass.exe 和 KeePassCommand.exe 在同一目录。然后重启KeePass

#### 4.2 配置Xpipe访问密码管理器

在Xpipe设置页中设置密码管理器，先选择预设1Passwd，然后修改命令如下，其中的路径按照你的安装路径写

``` bash
&"C:\Program Files\KeePass Password Safe 2\KeePassCommand.exe" getfieldraw "$KEY" Password
```

![new_db](https://kamust.github.io/images/best_ssh_client/set_passwd_mgr.png)

可在下方测试中填写 `Sample Entry` ，应该会出现 "Password"，这是KeePass新建数据库后的示例密码。

#### 4.3 使用密码连接ssh

在 Xpipe 的主页中添加ssh连接，在密码中选择密码管理器，然后填写这个密码在KeePass中设置的条目名称。

![new_db](https://kamust.github.io/images/best_ssh_client/set_passwd_mgr.png)

## 补充

两个软件均可设置为中文界面，方便使用，用户可以自行寻找相关设置。
