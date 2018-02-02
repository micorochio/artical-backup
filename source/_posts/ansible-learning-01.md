---
title:
- 自动化部署工具 Ansible填坑记录
date:
- 2017-05-30 17:48:58
tags:
- Linux
- ansible
- 运维
- 自动化持续部署
---
因为公司想对项目逐步转向为自动化部署，所以安排我和一位大佬做起了运维。目前是想先用ansible实现从git上获取code，在ansible主机上编译，配置，打包，发布。所以就有了这篇文章。

![](http://upload-images.jianshu.io/upload_images/1112615-fea8ebad97cb96d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->

# 0x00 安装

Ubuntu 16.04
```bash
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```
Pip安装
pip是Python的包管理工具，先装好pip，再安装ansible。目前建议Python版本：2.7，pip版本：9
```bash
sudo pip install ansible
```
源码编译安装
```bash
git clone git://github.com/ansible/ansible.git --recursive
cd ./ansible
sudo make install
```
CentOS安装
```
sudo yum install ansible
```
> 
+ 强烈建议使用CentOS安装。且安装ansible用户最好能保证拥有其他需要使用软件的权限，如Maven，Java，Python，Mysql等
否则ansible使用到这些软件时，容易出现权限不足，或者找不到环境变量等问题
+ Windows server 和OS X最好放在虚拟机里学习ansible，因为这是运维专用的，最好运行在Linux系统上，有实战意义。

检测安装成功:
```bash
ansible --version
```

![](http://upload-images.jianshu.io/upload_images/1112615-deb275ca4824f5ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 0x01 配置
安装之后是需要做一些配置，如果你是从源码`make install`安装的，可能没有配置文件。可以直接从example文件夹中，复制一份到`/etc/ansible/`下就可以了。

如果没有这个文件夹，创建一下就行，保证ansible的操作用户有这个文件夹的操作权限。

如果是其他方式安装的，执行
```
ansible --version
```
可以看到配置文件的位置，找不到就去github上看看，example文件夹里有
这两个文件，很重要！
+ ansible.cfg：是ansible 全局的配置
+ hosts：是ansible管理的主机列表，和部分参数。

anbile配置文件是有优先级的
下面是ansible1.5只后版本查找ansible.cfg的优先级。
1. ANSIBLE_CONFIG (an environment variable)
2. ansible.cfg (in the current directory)
3. .ansible.cfg (in the home directory)
4. /etc/ansible/ansible.cfg

下面是配置参数大全，参考用。
刚开始使用的时候，主要配置hosts文件位置
```bash
# (扩展插件存放目录)
action_plugins = /usr/share/ansible_plugins/action_plugins 
# (插入Ansible模板的字符串)
ansible_managed = Ansible managed: {file} modified on %Y-%m-%d %H:%M:%S by {uid} on {host}
# （PlayBook是否需要提供密码，默认为No）
# ask_pass=True
# （PlayBook是否需要提供sudo 密码）[](http://www.ansible.cn/docs/intro_configuration.html#ask-sudo-pass)
# ask_sudo_pass=True
# （回调函数插件存放路径）
action_plugins = /usr/share/ansible_plugins/action_plugins
# （连接插件存放路径）
action_plugins = /usr/share/ansible_plugins/action_plugins
# （是否展示警告信息）
deprecation_warnings = True
# （是否展示跳过的主机的信息）
# display_skipped_hosts=True
# （执行错误时候赋予的变量）
# error_on_undefined_vars=True
# （默认的Shell）
# executable = /bin/bash
# （拦截器插件）
action_plugins = /usr/share/ansible_plugins/action_plugins
# （最大进程数）
forks=5
# （哈希特性，没事不用去动它）
# hash_behavior=replace
# （资产文件存放位置）
hostfile = /etc/ansible/hosts
# （是否检查SSH key）
host_key_checking=True
# （JinJa扩展）
jinja2_extensions = jinja2.ext.do,jinja2.ext.i18n
# （PlayBook变量）
legacy_playbook_variables = no
# （Ansible默认库）
library = /usr/share/ansible
# （日志路径）
log_path=/var/log/ansible.log
# （插件路径）
action_plugins = /usr/share/ansible_plugins/action_plugins
# （默认模块名称）
module_name = command
# (输出样式)
nocolor=0
# (是否使用cowsay打印)
nocows=0
# （主机）
hosts=*
# （pool间隔）
poll_interval=15
# （私钥的存放路径）
private_key_file=/path/to/file.pem
# （远程连接端口号）
remote_port = 22
# (远程目录临时文件夹)
remote_temp = $HOME/.ansible/tmp
# （远程用户）
remote_user = root
# （角色路径）
roles_path = /opt/mysite/roles
# （SUDO执行）
sudo_exe=sudo
# （SUDO标记）
sudo_flags=-H
# （sudo用户）
sudo_user=root
# （重连次数）
timeout = 10
# （传输模式） 默认用的smart
transport
# （变量插件存放路径）
action_plugins = /usr/share/ansible_plugins/action_plugins
# SSH变量
# (SSH连接参数)
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
# （采用SCP还是SFTP进行文件传输）
scp_if_ssh=False
```

# 0x02 host文件
hosts文件是ansible管理主机的Inventory文件，里面存放的是主机组和部分参数
例如
```bash
[local]
127.0.0.2 ansible_ssh_port=22 ansible_user=zing ansible_ssh_pass=别傻了，我不会写的

[webserver]
xxx1.example.com ansible_ssh_port=33 ansible_user=root
xxx2.example.com

```
port：ssh到目标主机的端口
user：目标主机将会以这个身份登录
pass：目标主机该用户的密码

中括号内是主机的分组名
+ 下面的是主机地址，可以是ip，可以是域名。ip（域名）之后跟的是链接参数。
+ 如果啥都不写，安装咱们之前写的ansible.cfg中定义的默认配置来进行ssh链接
+ 其中如果两台主机进行了ssh 互信。那么ansible_ssh_pass参数可以不写。
http://ansible-tran.readthedocs.io/en/latest/docs/intro_inventory.html
在上面有详细讲解hosts文件定义的规范和技巧。

遇到几个坑：

错误：用户没有ssh的权限。首先保证user能被ssh连上。
```bash
Failed to connect to the host via ssh: Permission denied 
```


错误：密码不对
```bash
Authentication failure. 
```

错误：主机没有sshpass模块，这个想办法自己装上。
```bash
to use the 'ssh' connection type with passwords, you must install the sshpass program
```

# 0x03 爱因斯坦的小板凳 hello world
接下来介绍使用ansible在目标主机上打印出：hello world
先不管命令的含义
```bash
ansible 127.0.0.1 -m command -a 'echo "hello world"'
```

![成功！](http://upload-images.jianshu.io/upload_images/1112615-168b6a19d6b502f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 0x04 参考
+ http://ansible-tran.readthedocs.io/en/latest/index.html
+ http://docs.ansible.com/ansible/
+ http://docs.ansible.com/ansible/intro_configuration.html
+ 已失效：http://www.kiratechblog.com/?p=420

