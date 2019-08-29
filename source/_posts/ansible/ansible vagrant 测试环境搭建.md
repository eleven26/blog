---
title: ansible vagrant 测试环境搭建
date: 2019-08-30 06:39:00
tags: [ansible]
---

### 初始化 vagrant box

```bash
mkdir playbooks
cd playbooks
vagrant init ubuntu/trusty14.04
vagrant up
```

### 连接到 vagrant 虚拟机

使用 `vagrant ssh` 命令可以登录到刚刚创建的 ubuntu14.04 虚拟机中。

这种方法可以让我们直接与 shell 交互，但是 Ansible 使用标准 SSH 客户端连接到虚拟机，而不是使用 vagrant ssh 命令。

如下操作告诉 Vagrant ssh 输出 SSH 连接的详细信息：

```bash
vagrant ssh-config
```

输出如下：

> Host default
    HostName 127.0.0.1
    User vagrant
    Port 2201
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentityFile /Users/ruby/Documents/ubuntu14.04/.vagrant/machines/default/virtualbox/private_key
    IdentitiesOnly yes
    LogLevel FATAL

最重要的部分是：

```
HostName 127.0.0.1
User vagrant
Port 2201
IdentityFile /Users/ruby/Documents/ubuntu14.04/.vagrant/machines/default/virtualbox/private_key
```

Vagrant 1.7 版本改变了它处理 SSH 私钥的行为。从 1.7 版本开始，Vagrant 为每台机器都创建了以一个私钥。之前的版本都使用相同的私钥，该私钥存放在 `~/.vagrant.d/insecure_private_key`。

基于这些信息，下面确认一下你是否可以从命令行发起到虚拟机的 SSH 会话：

```bash
ssh vagrant@127.0.0.1 -p 2201 -i /Users/ruby/Documents/ubuntu14.04/.vagrant/machines/default/virtualbox/private_key
```

注意：端口必须和上面的一致


### 将测试服务器的信息配置在 Ansible 中

Ansible 只能管理那些它明确了解的服务器。你只需要在 inventory 文件中指定服务器的信息就可以将这些信息提供给 Ansible。

每台服务器都需要一个名字以便 Ansible 来识别。可以使用服务器的 hostname，或者给服务器起一个别名并传递一些参数。这些参数告诉 Ansible 如何连接到服务器。

我们将给刚刚创建的 vagrant 服务器起一个别名：testserver。

在 playbooks 目录下创建一个 hosts 文件。这个文件将充当 inventory 文件。

如果你正在使用 Vagrant 机器作为你的测试服务器，hosts 文件应该和下面的内容很像：

```bash
testserver ansible_host=127.0.0.1 ansible_port=2201 ansible_user=vagrant ansible_private_key_file=/Users/ruby/Documents/ubuntu14.04/.vagrant/machines/default/virtualbox/private_key
```

这里我们看到一个使用 Vagrant 的缺点：不得不明确地传入额外参数来告诉 Ansible 如何连接。在一般情况下，我们不需要这些补充信息。

```bash
ansible testserver -i hosts -m ping
```

> testserver | SUCCESS => {  
      "changed": false,  
      "ping": "pong"  
  }

如果命令没有执行成功，可以添加 -vvvv 参数来查看这个错误的更多信息。

我们可以看到模块执行成功。输出中的 `changed:false` 部分告诉我们模块的执行没有改变服务器的状态。输出中的 `"ping": "pong"` 是正确的模块定义的输出。

ping 模块除了检查 Ansible 是否可以打开到服务器的 SSH 会话外并不做任何其他的事情。它就是用来检测你是否能连接到服务器的实用工具。


### 使用 ansible.cfg 文件来简化配置

在上例中，我们不得不在 inventory 文件中输入很多内容来告诉 Ansible 关于我们测试服务器的信息。幸运的是，Ansible 有许多方法来让你定义各种变量。

这样，就不需要把那些信息都堆在一个地方了。<em>ansible.cfg</em> 文件可以设定一些默认值，这样就不需要将同样的内容键入很多遍了。


### 我们应该把 ansible.cfg 放到哪里  

Ansible 按照如下位置和顺序来查找 <em>ansible.cfg</em> 文件：

* ANSIBLE_CONFIG 环境变量所指定的文件

* ./ansible.cfg （当前目录下的 ansible.cfg）

* ~/.ansible.cfg （主目录下的 .ansible.cfg）

* /etc/ansible/ansible.cfg


ansible.cfg

```
[defaults]
inventory = hosts
remote_user = vagrant
private_key_file = .vagrant/machines/default/virtualbox/private_key
host_key_checking = False
```

上面的范例配置关闭了主机密钥检查，这样在处理 Vagrant 机器时会很方便。

否则，每次销毁并重新创建一台 Vagrant 机器之后，都需要编辑 `~/.ssh/known_hosts` 文件。但是，关闭主机密钥检查在通过网络连接其他主机时会成为安全风险。

有了我们设定的默认值，你就不需要在 inventory 文件中明确配置 SSH 文件密钥了。它将简化为：

```bash
testserver ansible_host=127.0.0.1 ansible_port=2201
```

还可以在执行 ansible 命令时去掉 -i 参数：

```bash
ansible testserver -m ping 
```

查看服务器上的运行时间：

```bash
ansible testserver -m command -a uptime
```

command 模块非常有用，ansible 将它设为默认模块，所以我们可以这样简化操作：

```bash
ansible testserver -a uptime
```

如果命令中包含空格，需要使用引号将命令括起来，这样 shell 才会将整个字符串作为一个参数传递给 ansible。如：

```bash
ansible testserver -a "tail -f /var/log/dmesg"
```


