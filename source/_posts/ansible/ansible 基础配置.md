---
title: ansible 基础配置
date: 2019-08-29 08:39:00
tags: [ansible]
---


### mac 下安装 ansible

```bash
brew install ansible
```

添加配置文件

```bash
sudo mkdir /etc/ansible
touch /etc/ansible/hosts
```


### ssh 公钥加到服务器

```bash
ssh-keygen -t RSA
```

把 `cat ~/.ssh/id_rsa.pub` 的内容复制到服务器对应 user home 的 authorized_keys 里面


### 修改  ~/.ssh/config

添加下面的内容

```
Host vagrant
     HostName vagrant
     User vagrant
     Port 22
     IdentityFile /Users/ruby/.ssh/id_rsa
```

Host 自定义的一个名称

HostName 服务器域名或者 ip

vagrant 是本机 /etc/hosts 已经配置了的 hosts

IdentityFile 是私钥的路径，和加到服务器的公钥是同一对


### 修改 /etc/ansible/hosts 配置

添加

```
[test-vagrant]
vagrant
```

test-vagrant 是分组名 (我们可以对某一组服务器执行某个命令)

vagrant 是 Host 

不使用 .ssh/config 的 hosts 文件配置写法：在 hosts 里面每一行指定 hostname、user 等信息，如

```
testserver ansible_host=127.0.0.1 ansible_port=2222\
  ansible_user=vagrant\
  ansible_private_key_file=.vagrant/machines/default/virtualbox/private_key
```


### 测试 

```bash
ansible vagrant -m command -a "pwd"
```

> ➜  ~ ansible vagrant -m command -a "pwd"  
vagrant | CHANGED | rc=0 >>  
/home/vagrant  

