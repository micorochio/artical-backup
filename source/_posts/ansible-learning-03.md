---
title:
- Ansible自动化部署从入门到弃坑
date:
- 2017-06-18 17:48:58
tags:
- Linux
- ansible
- 运维
- 自动化持续部署
---

![](http://upload-images.jianshu.io/upload_images/1112615-d08c2ab7f6cf7012.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 0x00 @Before
上次Ansible自动部署入门，最后写了点playbook的项目，了解了一些如task，template，vars等简单的用法。但是真正的Ansible项目并不是通过主机来分Roles的，而是一个Ansible管理多台主机，用Roles来区分项目。
所以这一次，我带来了不一样的使用姿势。
<!--more-->
## 1x00 Playbook再介绍
playbook 是剧本，我在上片已经介绍了点，总之入口是一个yml文件
```yml
- hosts: local
  remote_user: zing
  environment: 
    JAVA_HOME:  /application/jdk8/
    JRE_HOME: /application/jdk8/jre/
    M2_HOME: /application/maven/bin/
  tasks:
    - name: package project
      shell: mvn clean package
      remote_user: zing
    - name: deploy to maven service
      shell: mvn deploy
      sudo: yes
      ignore_errors: yes
  
```

如果项目简单, 一个playbook.yml文件就可以搞定了


### 1x01 Handler
handler，用来执行某些条件下的任务，比如当配置文件发生变化的时候，通过notify触发handler去重启服务器。如下面playbook。
```yml
- hosts: local
  remote_user: zing
  tasks:
    - name: copy properties
      copy: src=/home/zing/project.properties dest=/home/java/pro/project.properties
    - name: 
      file: path=/home/java/pro/project.properties mode=600
      notify:
        - restart server
  handlers:
    - name: restart server
      service: name=tomcat state=restarted
```
还有一种条件判断，可以直接写在task里面，如下
```yml
- hosts: local
  remote_user: zing
  tasks:
    - name: copy properties
      copy: src=/home/zing/project.properties dest=/home/java/pro/project.properties
    - name: modify project property
      file: path=/home/java/pro/project.properties mode=600
    - name: restart server when server is zing
      service: name=tomcat state=restarted
      when: result|changed
```
when的用法还有很多，可以自己探索。不过一般用Handler会更加灵活一些。when的条件判断需要了解很多Ansible变量，想知道的话，可以自己参考官方文档

### 1x02 循环迭代
上一篇文章已经写过迭代了，我怕写的笼统，这里再拿出来讲一下

```yml
- hosts: local
  remote_user: zing
  tasks:
    - name: transfom template
      template: src="{ {item.file_src} }" dest="{ {item.target_file_path} }"
      with_items:
        - {file_src: '/home/zing/template/application.j2', target_file_path: '/home/service/resources/application.properties'}
        - {file_src: '/home/zing/template/config.j2', target_file_path: '/home/service/resources/config.properties'}
```
使用with_items，将下面的参数迭代进tasks里面，这样每一个参数都按照变量名称会放入`{ {  } }`对应的变量名称中。直到item循环迭代介绍，才会执行下一个task


## 1x03 Tags
通过tag可以指定运行的task，然而简单的部署时tag并不常用，需要tag的时候一般可以直接使用ansible命令处理，或者再写新的playbook。只有在大型项目部署的时候，偶尔才会使用。所以只简单介绍一下tag的用法。

+ 首先，在tasks内的yml文件中，对需要的task打上tag：
```yml
- name: yun install package 
  yum: name="{ { item } }" state=installed 
  with_items: 
     - httpd 
     - memcached 
  tags: 
     - packages 

 - name: configuration modity 
  template: src=templates/src.j2 dest=/etc/foo.conf 
  tags: 
      - configuration

```

+ 调用某个tag：

```shell
 ansible-playbook example.yml – tags “configuration,packages”
```
task可以打上任意多个tag。
 
### 1x04 变量
变量分好几种，可以定义在playbook中，也可以定义在hosts文件上，后面也可以写在roles的vars文件夹中。
上面介绍循环迭代，`{ {item.file_src} }`就是变量引用，`with_items`下的就是变量值。变量可以放置在几乎所有地方，除了关键字外，其他地方都可以引用变量。Ansible自己也定义了好多自带的变量，有兴趣的可以自己看看

hosts主机变量如下
```shell
# 针对主机的主机变量
127.0.0.1 my_name_is=zing

# 针对组的变量
[webServer]
domain1.example.com
domain2.example.com
[webServer:vars]
server_user_name_is="zing"
server_user_is="java_programer"

# 针对所有主机的所有变量
[all:vars]
user_name_is="zing"
user_is="java_programer"

```

使用变量只要在正确的位置加上变量引用`{ { varable_name } }`就好
```yml
- hosts: local
  remote_user: "{ { user_name_is } }”
  tasks:
    - name: copy properties
      copy: src=/home/zing/project.properties 
```
> Ansible支持复杂变量，我们的`{ { item.src } }`就是一个复杂型的，通过`.`来引用item下的src值。所以，变量名称不要带`.`。


复杂变量也很简单。不过一般定义在yaml文件里面，playbook中可以这么定义
```yml
- hosts: local
  remote_user: zing
  tasks:
    - name: start server { { server1.name } }
      service: name=tomcat state=restarted
  vars:
    server1:
        name: "zing_service"
        type: "tomcat"
```

> 注意：
某些时候YAML冒号后面的值不能以{开头，如果有要以{开头，必须加上引号。解决方式如下。
```yml
- hosts: zing_servers
  vars:
       server_path: "{ { base_path } }/zing"
```


### 1x05 模板和变量搭配使用
所谓的模板就是以一个现成的文件为样板，向其中填充参数，来生成我们需要的真实文件，这个无须多介绍，参考
https://micorochio.github.io/2017/06/05/ansible-learning-02/#0x03-yaml和playbook
下的 application.properties.j2 文件写法，双大括号里的参数会被定义的变量所替换，文件替换流程参考下面的写法
```yml
- name: transfom template
  template: src={ {item.template_file} } dest={ {item.target_file} }
  with_items:
    - {template_file: 'template/application.j2', target_file: 'resources/application.properties'}
    - {template_file: 'template/config.j2', target_file: 'resources/config.properties'}
```
这段task的意思是：将item的模板template_file转换成target_file。定义多个item 自动迭代，将参数替换到task变量中

## 2x00 Roles
这是个新的概念，playbook只能管理一个项目的话，通过Roles可以用一个ansible工程，管理公司所有的工程。上一篇文章，我介绍的Roles是根据主机来分Roles（角色）的，实际开发中，大多是根据项目名称来分角色。这样一套ansible，就能hold住全部工程。

###2x01 正确的ansible工程目录
```java
inventory/                   //hosts文件夹
    project_a_host           //工程a的hosts
    project_a_hosts          //工程b的hosts
project_a_playbook.yml       //参考上篇文章的side.yml
project_b_playbook.yml       //参考上篇文章的side.yml
roles/                       //roles文件夹，第一级子文件夹就是就是role的名称
   project_a/                //role,表示工程A
     files/                  //一般用来存放脚本，或者其他部署时需要使用的文件
     templates/              //存放模板
     tasks/                  //存放任务tasks
     handlers/               //存放Handler
     vars/                   //存放本角色可以使用的变量
     defaults/               //用来存放默认变量的，如果其他地方不定义，会在这里找，否则会使用其他地方定义的变量
     meta/                   //用于定义此角色的特殊设定及其依赖关系,我还没用到这个
   project_b/
     files/
     templates/
     tasks/
     handlers/
     vars/
     defaults/
     meta/
```
### 2x02 一个开源的 tomcat 工程实例
这个例子的开源地址：https://github.com/ansible/ansible-examples/tree/master/tomcat-memcached-failover
这个例子很好的展示了Tomcat服务的自动化部署的Ansible工程写法

![](http://upload-images.jianshu.io/upload_images/1112615-93ef05cd84dfacb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 3x00部署一个Maven版的java项目
只是简单介绍一下从0开始到部署的一个流程，具体的例子，可以到https://github.com/ansible/ansible-examples 上随意查找，里面应有尽有。

流程如下：
+ 所有主机安装Java （首次执行）
+ Ansible主机安装Maven（首次执行）
+ Ansible主机安装git （首次执行）
+ 使用Ansible shell模块清理残余代码（首次无须执行）
+ Ansible主机使用git 获取Java项目源码
+ Ansible Template替换新源码
+ 使用shell模块，执行mvn package打出jar包或者war包
+ 服务主机安装服务软件Tomcat Nginx Mysql等（首次执行）
+ 关闭服务（如果是热更新，无须关闭）
+ 将jar包或者war包使用copy模块，传输到服务主机
+ 修改服务软件配置等主机环境
+ 启动服务

如果是多项目部署，最好是：
+ 每个项目都有自己的role name，
+ 运维负责在当前role使用的inventory文件中修改配置，
+ 将这些配置通过模板的方式，覆盖到代码的各个配置文件中，
+ 最后打成运行包，传输到承载软件的服务器上，启动即可。

这样不会出现不同环境切换，程序员自己手动改配置，出现：在生产上使用了测试的数据库。生产服务连接测试的 redis，打爆了测试用的redis服务器还造成了严重的数据丢失。

## 4x00Ansible和Jenkins一键部署
其实很简单，安装Jinkens，用Jinkens 执行
```shell
ansible-playbook project-playbool.yml -i inventory-file
```
下面的东西就交给Ansible了，不再需要用繁琐的Shell脚本来写Jenkins部署脚本了。并且，每个项目都是一键发布，而且不用维护用于项目发布的部署shell，十分轻量，尤其是对微服务，批量扩展和修改很方便。

写到这里就出坑了，毕竟不是专业的运维，多谢观看

##5x00 @After
参考：
极力推荐=》https://github.com/ansible/ansible-examples

> 文章内代码部分，双花括号之间没有空格，为博客软件bug，无法解决。ansible项目中使用&#123;&#123; &#125;&#125;

转载请注明出处：https://micorochio.github.io/2017/06/19/ansible-learning-03/
