title: Permissions 0644 for ‘/root/.ssh/id_rsa’ are too open处理
date: 2015-5-11 22:49:29
tags: Git
---
在把id_rsa复制到新系统进行git操作出现 Permissions 0644 for ‘/root/.ssh/id_rsa’ are too open. 错误，只要把权限降到0600就ok了
输入命令
chmod 0600 /root/.ssh/id_rsa
然后就可以密钥登陆了