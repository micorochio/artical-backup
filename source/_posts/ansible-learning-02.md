---
title:
- Ansible自动化部署入门
date:
- 2017-06-04 17:48:58
tags:
- Linux
- ansible
- 运维
- 自动化持续部署
---
![](http://upload-images.jianshu.io/upload_images/1112615-1a9c0f3807270562.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ Ansible的特性：
  + ansible基于Python实现，有Paramiko、PyYAML、JinJia2主要模块
  + 使用SSH链接主机，部署简单
  + 可以使用自定义模块，也可以使用其他语言编写的模块，基于模块可以完成各种任务。

<!--more-->
# 0x00 Ansible 主机库文件： hosts inventory

host inventory是主机库文件，里面存放的是要管理的主机列表，和一些主机参数，另外也可以加入变量等自定义的参数。

一般来说，常见的主机库文件如下
```shell
# 单独的主机域名
mail.example.com

#单独的主机ip
8.8.8.8

[webservers] #主机组的名称
127.0.0.1 #主机1
bar.example.com #主机2

[dbservers] #新的主机组
one.example.com #主机1
two.example.com #主机2
three.example.com

```

https://micorochio.github.io/2017/05/31/ansible-learning-01/#0x02-host文件
上一次介绍了默认的主机库，其实在运行ansible 命令时，是可以指定inventory文件的
这里先不做介绍。带着下面问题，继续往下看
+ hosts inventory的常用参数有哪些？
+ 如何针对不同的环境，使用不同的hosts inventory文件？
+ hosts inventory如何定义变量？

# 0x01 入门级命令
上一次介绍了一个简单的ansible命令:[ansible 打印 Hello  World](https://micorochio.github.io/2017/05/31/ansible-learning-01/#0x03-爱因斯坦的小板凳-hello-world)

+ 命令的组成

![](http://upload-images.jianshu.io/upload_images/1112615-0c5d42edf618e261.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这句话会找到指定的主机（127.0.0.1）在主机上，使用command，执行`echo hello world`

如果默认的hosts文件中，定义了主机组，也可以将ip换成主机组的名字
```shell
# /etc/ansible/hosts 文件
[local]
127.0.0.1
```
执行
```shell
aisible local -m command -a 'echo hello world'
```
local会去hosts文件中找到对应的组，组下的每一台机器都会运行指令。

如果想让所以主机全部执行
```shell
aisible all -m command -a 'echo hello world'
```

command模块意思是执行Linux主机的command，模块使用命令名称后跟一个列表空格分隔的参数。 给定的命令将全部执行选定的节点。 因为不会通过shell进行处理，所以“$ HOME”这样的变量，像“”<“”，“”>“”，“”|“`“;”“和”“＆”“将不起作用。


-那么 -m 指定模块，除了command，一定还有其他模块，那么在没有文档的情况下，
+ 怎么知道其他模块，和模块指令的用法呢？

```
# 查询所有模块
ansible-doc -l
# 查看command模块
ansible-doc command
# 查看shell 模块
ansible-doc shell
```
![模块列表](http://upload-images.jianshu.io/upload_images/1112615-9e086e810fa2a6f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![模块具体介绍](http://upload-images.jianshu.io/upload_images/1112615-d1f4efdc142e5c95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用ansible-doc，可以帮助快速了解模块的作用和用法,ansible 还有其他参数，如下：
```shell
用法：ansible-doc [options] [module ...]

选项：
   -a，--all显示所有模块的文档
   -h，--help显示此帮助消息并退出
   -l，--list列出可用的模块
   -M MODULE_PATH，--module-path = MODULE_PATH
                         指定模块库的路径（默认=无）
   -s，--snippet显示指定模块的播放列表片段
   -v，--verbose详细模式（-vvv为更多，-vvvv启用
                         连接调试）
   --version显示程序的版本号并退出
```

接下来介绍几个常用的模块

# 0x02 ansible常用模块介绍

+ command：这是个默认模块，不写-m xx模块的时候，默认会当成command模块，表示执行主机指令
```shell
ansible local -a 'whoami'
```

![默认为command模块](http://upload-images.jianshu.io/upload_images/1112615-e348ca07b9650bca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ cron：定时任务模块，可以指定一个时间去执行某个任务

有以下参数可供选择
```shell
- name: Manage cron.d and crontab entries.
  action: cron
      backup                 # 如果设置，在修改之前会创建crontab的备份。 备份的位置由此模块在`backup_file'变量中返回。
      cron_file              # 如果指定，则使用此文件而不是单个用户的crontab。 如果这是一个相对路径，它将被解释为/etc/cron.d。 （如果是绝对的，它通常是/ etc / crontab）。 要使用`cron_file'参数，你也必须指定'user'。
      day                    # 每个月中的某天 ( 1-31, *, */2, etc )
      disabled               # 当state = present的时候，可以在cron中禁用当前job。 
      env                    # 管理crontab的环境变量，新变量会从crontab顶部添加
      hour                   # 任务执行时间：时 ( 0-23, *, */2, etc )
      insertafter            # 与'state = present'和'env'一起使用。 如果指定，新的环境变量将在声明指定的环境变量后插入。
      insertbefore           # 与'state = present'和'env'一起使用。 如果指定，新的环境变量将在声明指定的环境变量前插入
      job                    # 要执行的命令，或者如果设置了env，则为环境变量的值。 state =present则为必需声明job。
      minute                 # 任务执行时间：分钟( 0-59, *, */2, etc )
      month                  # 任务执行时间：月 ( 1-12, *, */2, etc )
      name                   # crontab条目的描述，或者如果设置了env，则为环境变量的名称。 如果state=absent则必配置。 请注意，如果名称未设置且state=present，则将放弃已有条目，始终创建一个新的crontab条目。
      reboot                 # 弃用了，使用special_time更好，表示重启后执行
      special_time           # 特殊时间规格昵称。
      state                  # 是否确保工作或环境变量存在或不存在。
      user                   # crontab应该修改的具体用户。
      weekday                # 任务执行时间：周几 ( 0-6 for Sunday-Saturday, *, etc )
```
举个栗子：
每10分钟输出一次 hello
```shell
ansible local -m cron -a 'minute="*/10" job="/bin/echo hello" name="test ansible-cron"' 
```
![执行结果](http://upload-images.jianshu.io/upload_images/1112615-7dfc809a5768a865.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```shell
# 到主机中查看：
crontab -l
```
![查看](http://upload-images.jianshu.io/upload_images/1112615-71f4d73e1a56b717.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```shell
# 移除定时任务
ansible local -m cron -a 'minute="*/10" job="/bin/echo hello" name="test ansible-cron" state=absent' 
```

![移除定时任务](http://upload-images.jianshu.io/upload_images/1112615-faf3abc3f8bbec9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

cron 我是直接翻译了文档，之后可以使用`ansible-doc -s`命令自己查看.后面也不再详细翻译
+ user 操作用户 

```shell
# 创建用户
ansible local -m user -a 'name="zing1" password="自定义的密码" groups="root,system,sys" home="/home/zing1" '

# 删除用户
ansible local -m user -a 'name="zing1" state=absent fource=yes'
```

+ group 操作用户组
```shell
# 创建组
ansible local -m group -a 'name="mysql" gid=306 system=yes '
# 删除同user
```

+ copy 复制文件
```shell
# 将本机 源.txt 文件拷贝到远程主机上成为 目标.txt
ansible webserverhost -m copy -a 'src="/home/zing/Documents/源.txt" dest="/home/xxserver/目标.txt" owner="root" mode=640'
```
src=本地目标（可以是文件夹）
dest=远程目标（可以是文件夹）
src可以用content替换
content：可以取代src，表示信息直接生成文件，与src不能同时使用。

+ file 操作文件
```shell
ansible local -m file -a 'ower="mysql" mode=644 src="/home/zing/Documents/from_mysql.link"  path="/home/mysql/xxx.link" state="link" '
```

+ service 指定服务的运行状态
```shell
ansible local -m service -a 'name="httpd" enabled=true state=started '
#enable表示是否开机启动
#state 参数有started stopped restarted
```
+ shell，指定执行shell命令，与command类似，在用到pipline，等复杂命令时，使用shell更加合适,这里不举例了

+ script 在远程主机上运行本机脚本，只支持用相对路径
```shell
ansible serverhost -m script -a 'test.sh'
```

+ setup 查看主机状态信息
```shell
ansible all -m setup
```

# 0x03 yaml和playbook
是一个可读性高，用来表达数据序列的格式。YAML参考了其他多种语言，包括：C语言、Python、Perl，并从XML、电子邮件的数据格式（RFC 2822）中获得灵感。目前已经有数种编程语言或脚本语言支持（或者说解析）这种语言。

YAML的语法和其他高级语言类似，并且可以简单表达清单、散列表，标量等数据形态。[4]它使用空白符号缩进和大量依赖外观的特色，特别适合用来表达或编辑数据结构、各种配置文件、倾印除错内容、文件大纲（例如：许多电子邮件标题格式和YAML非常接近）。尽管它比较适合用来表达层次结构式（hierarchical model）的数据结构，不过也有精致的语法可以表示关系性（relational model）的数据。[5]由于YAML使用空白字符和分行来分隔数据，使得它特别适合用grep／Python／Perl／Ruby操作。其让人最容易上手的特色是巧妙避开各种封闭符号，如：引号、各种括号等，这些符号在嵌套结构时会变得复杂而难以辨认。

想细致学习YAML的可以去阮一峰大神的博客看看：
http://www.ruanyifeng.com/blog/2016/07/yaml.html

那么Ansible中的Yaml文件是干啥的，其实就是指定，什么主机，执行什么任务的一个列表
```shell
- hosts: local
  remote_user: root
  tasks:
    - name: git clone
      git: repo=git@github.com:yourproject.git
           dest=/home/yourhome/git/
           accept_hostkey=yes
           force=yes
           recursive=no
           key_file=/home/user/.ssh/id_rsa.github-{{ item.repo }}

```
上面的一串yml配置就是让local主机以root身份去git上获取代码放到本地
很简单明了。

那么playbook怎么理解呢？
playbook本身是剧本的意思，ansible playbook中，包含了：
+ Inventory 主机库文件
+ modules 调用模块
+ Ad Hoc Commands 指定主机运行的命令
+ Playbooks 剧本文件
    + tasks 任务，调用模块执行某些命令
    + varable 变量
    + Template 模板
    + Handlers 处理器 由某些事件触发某些行为
    + Roles 角色

playbook的主要文件是用来指定tasks,下面举些例子
```shell
# webservice 服务运行
tasks:
  - name: make server running
    service: name="webservice" state="ruuning"

# 执行某个命令
tasks:
  - name: kill services
    shell: kill -9 {{service_PID}}

# 执行某个命令
tasks:
  - name: kill services
    shell: kill -9 {{service_PID}}

# 忽略脚本返回值
tasks:
  - name: some shell running
    shell: xxxxx.sh || /bin/true

# 忽略错误信息
tasks:
  - name: ignore errors
    shell: /tomcatpath/bin/startup.sh
    ignore_errors: True
```

执行playbook也很简单
```shell
ansible-playbook xxxxx.yml
```

指定hosts inventory文件
```shell
ansible-playbook xxxxx.yml -i hostsInventory文件PATH
```
# 0x04 从git开始到项目发布并启动

1.定义hosts inventory
test_evn_hosts文件
```shell
[local]
127.0.0.1 ansible_ssh_port=22222 ansible_user=zing

[server]
yourside.example.com  ansible_user=root

[all:vars]
#application.prpperties
SPRING_DATASOURCE_URL="jdbc:mysql://127.0.0.1:3306/yordatabase"
SPRING_DATASOURCE_USERNAME="yourname"
SPRING_DATASOURCE_PASSWORD="your_password"
SPRING_DATASOURCE_DRIVER_CLASS_NAME="org.mysql.Driver"

```
2.编写playbook YAML 文件
side.yml文件
```shell
- hosts: local
  remote_user: zing
  environment: 
    JAVA_HOME:  /application/jdk8/
    JRE_HOME: /application/jdk8/jre/
    M2_HOME: /application/maven/bin/
  roles:
    - local

  
- hosts: server
  remote_user: root
  roles:
  - server
```
3.编写配置文件
application.properties.j2 模板文件
```shell
spring.datasource.url = {{SPRING_DATASOURCE_URL}}
spring.datasource.username = {{SPRING_DATASOURCE_USERNAME}}
spring.datasource.password = {{SPRING_DATASOURCE_PASSWORD}}
spring.datasource.driver-class-name = {{SPRING_DATASOURCE_DRIVER_CLASS_NAME}}
```
4.编写脚本
git_clone.sh文件
```shell
#!/bin/bash
cd /home/zing/Documents/ansible/xxx/src
rm -rf xxx
git clone git@gitlabhost.com:xxx/xxx.git
```
5.编写task yaml
```shell
- name: git clone your xxx project
  shell: /home/zing/Documents/ansible/xxx/roles/local/files/git_clone.sh

- name: transfom template
  template: src={{item.file_src}} dest={{item.target_file_path}}
  with_items:
    - {file_src: '/home/zing/Documents/ansible/xxx/roles/local/template/application.j2', target_file_path: '/home/zing/Documents/ansible/xxx/src/xxx/src/main/resources/application.properties'}
    - {file_src: '/home/zing/Documents/ansible/xxx/roles/local/template/config.j2', target_file_path: '/home/zing/Documents/ansible/xxx/src/xxx/src/main/resources/config.properties'}

```
6.playbook 执行
```shell
ansible-playbook side.yml -i test_evn_hosts文件路径
```

整体结构

![ansible项目结构文件](http://upload-images.jianshu.io/upload_images/1112615-acaa83da1e5a0e4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 参考
http://ansible-tran.readthedocs.io/en/latest/docs/intro_inventory.html#inventoryformat


----

转载请注明来自MaxZing : https://micorochio.github.io/2017/06/05/ansible-learning-02/