# 项目概述

这部分主页就是为了创建通用于github以及bitbucket的ssh登陆方式 此章节只接受bitbucket github的流程类似

## 创建本地ssh key

### 1、认识ssh目录以及工作目录

ssh key在unix like 包括linux macos等系统都是保存在当前登录用户目录下,即 ~/.ssh/目录,普通用户非Root用户的工作目录位于
/home/xxxx/,root用户的工作目录位于/root/,因此对于普通用户的ssh key保存的绝对路径为/home/xxx/.ssh/,如下代码
```
bogon:.ssh chendan$ cd
bogon:~ chendan$ pwd
/Users/chendan
bogon:~ chendan$ cd ~/.ssh/
bogon:.ssh chendan$ pwd
/Users/chendan/.ssh
bogon:.ssh chendan$
```
<img src="pic/ssh_folder.png"/>

### 2、创建key

利用ssh-keygen创建ssh key 保存在~/.ssh/目录下 在创建的时候需要输入希望保存的key文件名 如果不输入则会保存为id_rsa.pub 
如此以来就会没次新创建的key会把上次创建的key覆盖 上次的key失效
其次 创建完了以后需要将key加载到 authorized_keys文件 该文件在~/.ssh/目录下 是所有公钥的保存文件 使用 >> 链接符是希望
将新创建的id_rsa4.pub公钥放置到 authorized_keys文件的后面
```
bogon:~ chendan$ ssh-keygen -t rsa -C 'ardellelee@163.com'
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/chendan/.ssh/id_rsa): /Users/chendan/.ssh/id_rsa4
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/chendan/.ssh/id_rsa4.
Your public key has been saved in /Users/chendan/.ssh/id_rsa4.pub.
The key fingerprint is:
SHA256:meTZecNOxNti7cq1JGEgY3CcrjbB5dejwywTv6Ngcp0 ardellelee@163.com
The key's randomart image is:
+---[RSA 2048]----+
|      ....       |
|       o+  .     |
|     . += ..o    |
|      o++B.=o+   |
|       oS*o.@.o  |
|      +.o.** =   |
|    ..+.Eo oo +  |
|     + .  o. = . |
|        .. .o .  |
+----[SHA256]-----+
bogon:~ chendan$ pwd
/Users/chendan
bogon:~ chendan$ cd .ssh/
bogon:.ssh chendan$ cat id_rsa4.pub >> authorized_keys 
bogon:.ssh chendan$ 

```

### 3、创建的key添加bitbucket

首先你的将本地添加到authorized_keys里的公钥拿到
```
bogon:.ssh chendan$ cat id_rsa4.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3C9kA1xBGNWxELMdVjKxRi66AOtCnA6lC+KRT/N8EDxbDBlc4Wz77IZTe418Fu99KXC5UQXFPyjNFyn3JGvv9U67jDwG0lw6sIBP2+znQUtTPujHVeiVEOJTpE5LF2oTp2CD69crQtG5rFGjmAymg/PMVyG4aQidAUoT3ILaogze3bnpvm3gj4zPCKAovKw6ZoUAvjOmc+SekyHO523G0DKJZrF/dBG6uOHveuGCzVCV+Uh+dwqf6C7sL7GY2c0pK1DR/s6ag1AaMw6CR66DW13VN4xTEz2YMmRK+Kc/nXvqfiqLQv4E4k82RwZEiBnV5sA4ZreGia2S8zqj0bgq9 ardellelee@163.com
```
拷贝cat id_rsa4.pub输出的结果

<img src="pic/create_ssh_key.png" />
打开bitbucket.com
<img src="pic/profile_key.png"/>
<img src="pic/add_key_bitbucket.png"/>
<img src="pic/key_list.png"/>

# 使用sourcetree

这个章节主要介绍如何使用sourcetree 常见的两种方式
第一 clone远端repos到本地
第二 将已有本地的代码目录创建一个新的远端repos并上传

## clone远端repos到本地

第一添加远端bitbucket账户

<img src="sourcetree_setting.png"/>
<img src="add_account.png"/>
<img src="account_detail.png"/>
<img src="account_list.png"/>

第二从账户clone repos到本地
<img src="select_repos_from_account.png"/>
<img src="clone_repos_select_local_folder.png"/>
<img src="save_repos_into_wonderful.png"/>
<img src="click_you_clone_folder.png"/>
<img src="clone_detail.png"/>

## 将已有本地的代码目录创建一个新的远端repos并上传

第一加载本地目录
<img src="create_loca_folder.png"/>
<img src="sourcetree_create_from_local_folder.png"/>
<img src="select_local_folder.png"/>
<img src="create_remote_repos_by_create_project.png"/>
<img src="select_remote_account_and_select_private_repos.png"/>
<img src="show_sourcetree_list.png"/>
<img src="remote_still_havant_anything.png"/>
<img src="add_remote_repos.png"/>
<img src="select_remote.png"/>
<img src="select_your_remote_repos.png"/>
<img src="type_repos_name.png"/>
<img src="show_remote.png"/>
<img src="vscode_open_folder_create_some_file.png"/>
<img src="sourcetree_add_changed_file.png"/>
<img src="submit.png"/>
<img src="show_repos_at_web.png"/>