---
layout:		post
title:		"Ansible 快速入门"
description: ""
date:		2020-12-28
author:		"Yawei"
categories: "ansible"
keywords:
    - ansible
---
# 是什么
Python的一套自动化工具库

# 组成

* Ansible
核心命令执行工具
* Ansible Playbook
任务剧本（又称任务集），具有编排能力的配置文件集合，yaml
* Inventory
管理的主机清单，默认/etc/ansible/hosts文件
* Modules
功能模块，支持自定义
* Plugins
模块功能的补充，常有连接类型插件，循环插件，变量插件，过滤插件等
* API
提供给第三方调用的应用程序接口
* custom modules
自定义模块

# 安装与配置

```shell
apt-get install ansible -y
ansible --version
```
## ansible配置文件优先级
```cpp
// 由上到下, 优先级逐渐降低
ANSIBLE_CONFIG  #环境变量
ansible.cfg #项目目录
.ansible.cfg #当前用户的家目录
/etc/ansible/ansible.cfg #默认配置文件
```

## 配置文件详解

```
#inventory      = /etc/ansible/hosts      #主机列表配置文件
#library        = /usr/share/my_modules/  #库文件存放目录
#remote_tmp     = ~/.ansible/tmp          #临时py文件存放在远程主机目录
#local_tmp      = ~/.ansible/tmp          #本机的临时执行目录
#forks          = 5                       #默认并发数
#sudo_user      = root                    #默认sudo用户
#ask_sudo_pass = True                     #每次执行是否询问sudo的ssh密码
#ask_pass      = True                     #每次执行是否询问ssh密码
#remote_port    = 22                      #远程主机端口
host_key_checking = False                 #跳过检查主机指纹
log_path = /var/log/ansible.log           #ansible日志
[privilege_escalation]   #如果是普通用户则需要配置提权
#become=True
#become_method=sudo
#become_user=root
#become_ask_pass=False
```

## Ansible inventory

/etc/ansible/hosts是默认主机资产清单文件，用于定义被管理主机的认证信息， 例如ssh登录用户名、密码以及key相关信息。
我们可以在执行ansible命令时，动态指定inventory文件
```
ansible web1 -m ping -i inventory.ini
```
### 如何配置Inventory文件
1. 主机支持主机名通配以及正则表达式，例如web[1:3].test.com代表三台主机
2. 主机支持基于非标准的ssh端口，例如web1.test.com:6666
3. 主机支持指定变量，可对个别主机的特殊配置，如登陆用户，密码
4. 主机组支持指定变量[group_name:vars]，同时支持嵌套组[game:children]
#### 场景一、基于密码连接
```bash
cat /etc/ansible/hosts

#方式一、主机+端口+密码
[webservers]
10.0.0.31 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'
10.0.0.41 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'

#方式二、主机+端口+密码
[webservers]
web[1:2].yuntaoshu.com ansible_ssh_pass='123456'

#方式三、主机+端口+密码
[webservers]
web[1:2].yutaoshu.com
[webservers:vars]
ansible_ssh_pass='123456'
```

#### 场景二、基于密钥连接，需要先创建公钥和私钥，并下发公钥至被控端
```bash
#利用非交换式工具实现批量分发公钥与批量管理服务器
ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.41
ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.31

#方式一、主机+端口+密钥
[group_name]
10.0.0.31:22
10.0.0.41

#方式二、别名+主机+端口+密钥
[group_name]
nfs-node1 ansible_ssh_host=10.0.0.31 ansible_ssh_port=22
```

#### 场景三、主机组使用方式
```bash
#方式一、主机组变量+主机+密码
[group_name1]
10.0.0.31
10.0.0.41
[group_name1:vars]
ansible_ssh_pass='123456'

#方式二、主机组变量+主机+密钥
[group_name2]
10.0.0.7
10.0.0.8

#定义多组，多组汇总整合
#webservers组包括两个子组[apapche,nginx]
[webservers:children]
[group_name1]
[group_name2]
```

#### 查看主机清单
```
#查看所有
[root@m01 ~]# ansible all --list-hosts
  hosts (1):
    172.16.1.7
#查看某个组内的主机数
[root@client ~]# ansible web --list-hosts
  hosts (1):
    172.16.1.7
#查看非默认配置主机清单的
[root@m01 /project1]# ansible web -i hosts --list-hosts
  hosts (1):
    172.16.1.7
```

#### inventory内置参数
![inventory](/img/post/2020-12-28/inventory.jpg)

#### 被控端首次连接容易出现问题
* 解决方案1
如果控制端和被控制端第一次通讯，需要确认指纹信息，如果机器特别多少的情况下怎么办？
将 Ansible 配置文件中的 host_key_checking = False 参数注释打开即可。
但要注意ansible.cfg文件的读取顺序。

* 解决方案2
通过脚本来实现
```
[root@ansible ~]# vim /server/scripts/fenfa.sh
#!/bin/bash 

if [ -f /root/.ssh/id_rsa ];then
   echo "----------密钥对已经存在---------------"
else
   echo "----------正在生成密钥对---------------"
   ssh-keygen -f /root/.ssh/id_rsa -N '' > /dev/null 2>&1
fi

for i in {5,6,7,8,9,31,41,51,52}
do
    echo "正在操作：172.16.1.${i}"
    echo "----------正在分发--------"
    sshpass -p123456 ssh-copy-id -i /root/.ssh/id_rsa.pub 172.16.1.${i} -o StrictHostKeyChecking=no > /tmp/ssh
.log 2>&1
done

```
## Ansible Ad-Hoc
### 什么是ad-hoc模式
>ad-hoc简而言之，就是“临时命令”，不会保存
>ansible中有两种模式, 分别是ad-hoc模式和playbook模式

### ad-hoc模式的使用场景
>场景一，在多台机器上，查看某个进程是否启动
>场景二，在多台机器上，拷贝指定日志文件到本地，等等

### ad-hoc模式的命令使用
![ansible-ad-hoc](/img/post/2020-12-28/ansible-ad-hoc.jpg)

### ad-hoc模式的常用模块
* Ansible执行返回->颜色信息说明
>黄色：对远程节点进行相应修改
>绿色：对远程节点不进行相应修改，或者只是对远程节点信息进行查看
>红色：操作执行命令有异常
>紫色：表示对命令执行发出警告信息（可能存在的问题，给你一下建议）

1. command命令模块
```
#默认模块, 执行命令
[root@m01 ~]# ansible test  -a "hostname"

#如果需要一些管道操作，则使用shell
[root@m01 ~]# ansible test -m shell -a "ifconfig|grep eth0" -f 50

-f = /etc/ansible/ansible.cfg中forks配置 #结果返回的数量
```
2. script脚本模块
```
#编写脚本
[root@m01 ~]# mkdir -p /server/scripts
[root@m01 ~]# cat /server/scripts/yum.sh
#!/usr/bin/bash
yum install -y iftop

#在本地运行模块，等同于在远程执行，不需要将脚本文件进行推送目标主机执行
[root@m01 ~]# ansible test -m script -a "/server/scripts/yum.sh"
```

3. yum安装软件模块
```
[root@m01 ~]# ansible test -m yum -a "name=httpd state=installed"
name        #指定要安装的软件包名称
state       #指定使用yum的方法
    installed，present   #安装软件包
    removed，absent      #移除软件包
    latest              #安装最新软件包 
```

4. copy文件拷贝模块
```
#推送文件模块
[root@m01 ~]# ansible test -m copy -a "src=/etc/hosts dest=/tmp/test.txt"

#在推送覆盖远程端文件前，对远端已有文件进行备份，按照时间信息备份
[root@m01 ~]# ansible test -m copy -a "src=/etc/hosts dest=/tmp/test.txt backup=yes"

#直接向远端文件内写入数据信息，并且会覆盖远端文件内原有数据信息
[root@m01 ~]# ansible test -m copy -a "content='bgx' dest=/tmp/test"
src             #推送数据的源文件信息
dest            #推送数据的目标路径
backup          #对推送传输过去的文件，进行备份
content         #直接批量在被管理端文件中添加内容
group           #将本地文件推送到远端，指定文件属组信息
owner           #将本地文件推送到远端，指定文件属主信息
mode            #将本地文件推送到远端，指定文件权限信息
```

5. file文件配置模块
```
[root@m01 ~]# ansible test -m file -a "path=/tmp/oldboy state=directory"
[root@m01 ~]# ansible test -m file -a "path=/tmp/tt state=touch mode=555 owner=root group=root"
[root@m01 ~]# ansible test -m file -a "src=/tmp/tt path=/tmp/tt_link state=link"

path            #指定远程主机目录或文件信息
recurse         #递归授权
state 
    directory   #在远端创建目录
    touch       #在远端创建文件
    link        #link或hard表示创建链接文件
    absent      #表示删除文件或目录
    mode        #设置文件或目录权限
    owner       #设置文件或目录属主信息
    group       #设置文件或目录属组信息
```

6. service服务模块
```
[root@m01 ~]# ansible test -m service -a "name=crond state=stopped enabled=yes"

name        # 定义要启动服务的名称
state       # 指定服务状态
    started     #启动服务
    stopped     #停止服务
    restarted   #重启服务
    reloaded    #重载服务
enabled         #开机自启
```

7. group组模块
```
[root@m01 ~]# ansible test -m group -a "name=oldgirl gid=888"

name            #指定创建的组名
gid             #指定组的gid
state
    absent      #移除远端主机的组
    present     #创建远端主机的组（默认）
```

8. user模块
```
#创建用户指定uid和gid，不创建家目录也不允许登陆
[root@m01 ~]# ansible test -m user -a "name=oldgirl uid=888 group=888 shell=/sbin/nologin create_home=no"

#将明文密码进行hash加密，然后进行用户创建
[root@m01 ~]# ansible localhost -m debug -a "msg={{ 'bgx' | password_hash('sha512', 'salt') }}"
localhost | SUCCESS => {
    "msg": "$6$salt$WP.Kb1hMfqJG7dtlBltkj4Um4rVhch54R5JCi6oP73MXzGhDWqqIY.JkSOnIsBSOeXpKglY7gUhHzY4ZtySm41"
}
[root@m01 ~]# ansible test -m user -a 'name=xlw password=$6$salt$WP.Kb1hMfqJG7dtlBltkj4Um4rVhch54R5JCi6oP73MXzGhDWqqIY.JkSOnIsBSOeXpKglY7gUhHzY4ZtySm41 create_home=yes shell=/bin/bash'

uid             #指定用户的uid
group           #指定用户组名称
groups          #指定附加组名称
password        #给用户添加密码
shell           #指定用户登录shell
create_home     #是否创建家目录
```

8.crond定时任务模块
```
#正常使用crond服务
[root@m01 ~]# crontab -l
* * * * *  /bin/sh /server/scripts/yum.sh

#使用ansible添加一条定时任务
[root@m01 ~]# ansible test -m cron -a "minute=* hour=* day=* month=* weekday=*  job='/bin/sh /server/scripts/test.sh'"
[root@m01 ~]# ansible test -m cron -a "job='/bin/sh /server/scripts/test.sh'"

# 设置定时任务注释信息，防止重复，name设定
[root@m01 ~]# ansible test -m cron -a "name='cron01' job='/bin/sh /server/scripts/test.sh'"

# 删除相应定时任务
[root@m01 ~]# ansible test -m cron -a "name='ansible cron02' minute=0 hour=0 job='/bin/sh /server/scripts/test.sh' state=absent"
 
# 注释相应定时任务，使定时任务失效
[root@m01 scripts]# ansible test -m cron -a "name='ansible cron01' minute=0 hour=0 job='/bin/sh /server/scripts/test.sh' disabled=no"
```

9. mount模块
```
[root@m01 ~]# ansible test -m mount -a "src=172.16.1.31:/data path=/data fstype=nfs opts=defaults state=present"
[root@m01 ~]# ansible web -m mount -a "src=172.16.1.31:/data path=/data fstype=nfs opts=defaults state=mounted"
[root@m01 ~]# ansible web -m mount -a "src=172.16.1.31:/data path=/data fstype=nfs opts=defaults state=unmounted"
[root@m01 ~]# ansible web -m mount -a "src=172.16.1.31:/data path=/data fstype=nfs opts=defaults state=absent"

present     # 开机挂载，仅将挂载配置写入/etc/fstab
mounted     # 挂载设备，并将配置写入/etc/fstab
unmounted   # 卸载设备，不会清除/etc/fstab写入的配置
absent      # 卸载设备，会清理/etc/fstab写入的配置
```

10.setup用于获取系统信息的一个模块
```
# 查看模块参数
[root@m01 ~]# ansible-doc -s setup

# 查看系统所有信息
[root@m01 ~]# ansible 192.16 1.31-m setup

# filter 对系统信息进行过滤
[root@m01 ~]# ansible 192.168.1.31 -m setup -a 'filter=ansible_all_ipv4_addresses' # 常用的过滤选项
ansible_all_ipv4_addresses         所有的ipv4地址
ansible_all_ipv6_addresses         所有的ipv6地址
ansible_architecture               系统的架构
ansible_date_time                  系统时间
ansible_default_ipv4               系统的默认ipv4地址
ansible_distribution               系统名称
ansible_distribution_file_variety  系统的家族
ansible_distribution_major_version 系统的版本
ansible_domain                     系统所在的域
ansible_fqdn                       系统的主机名
ansible_hostname                   系统的主机名,简写
ansible_os_family                  系统的家族
ansible_processor_cores            cpu的核数
ansible_processor_count            cpu的颗数
ansible_processor_vcpus            cpu的个数
```

11. unarchive解压模块
```
01.解压远程服务器的压缩包到指定目录
创建压缩包：

cd /etc && tar zxvf /opt/sys.tar.gz etc/fstab etc/hosts  
执行命令：

[root@m01 ~]# ansible 172.16.1.31 -m unarchive -a "src=/opt/sys.tar.gz dest=/opt/ remote_src=yes"
02.把本地文件解压到目标机器指定目录
创建命令

cd / && tar zcvf /opt/log.tar.gz var/log/messages
[root@m01 ~]# ansible 172.16.1.31 -m unarchive -a "src=/opt/log.tar.gz dest=/opt/" 
```

12. archive压缩模块
```
01.压缩单个文件

[root@m01 ~]# ansible 172.16.1.31 -m archive -a "path=/var/log/message dest=/tmp/log.tar.gz format=gz force_archi
```

13. ansible查看帮助方法
```
[root@m01 ~]# ansible-doc -l    --- 查看所有模块说明信息
[root@m01 ~]# ansible-doc copy  --- 表示指定查看某个模块参数用法信息
```

