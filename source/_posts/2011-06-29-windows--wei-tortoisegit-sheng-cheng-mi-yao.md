--- 
categories: 
  - 杂货铺
comments: true
layout: post
published: true
status: publish
tags: 
  - git
title: "windows下为TortoiseGit生成密钥"
type: post
---
使用TortoiseGit 的时候如果进行远程提交的话使用putty密钥会比较方便，记录一下生成密钥的步骤。


1. 下载 putty, putty-key-Generator
putty 是用来连接远程linux server的客户端，putty-key-Generator是用来生成公钥和密钥的。


2. 生成公钥密钥
运行  putty-key-Generator，设定生成公钥的长度，一般是1024,2048 .  然后点击Generate按钮，同时鼠标需要在生产区域不停的动，直到密钥生成。
生成好后，key-comment填写 `用户名@主机名` ，如: `git@server1`


3. Linux 端配置
确保 /etc/ssh/ssh_config文件中下面的配置打开
RSAAuthentication yes
编辑 用户目录下.ssh/authorized_keys, 把第一步生成好的公钥替换掉原来的。
修改权限：
{% codeblock %}
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
{% endcodeblock %}


4. windows端进行连接
使用putty进行连接，connection->ssh->auth 里选择第一步保存的私钥， 填写好session信息后登陆。输入login name, 这时使用了私钥就不再需要密码了。

 
