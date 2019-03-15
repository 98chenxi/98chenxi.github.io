---
layout: post
title:  "rhcsa-note"
date:   2019-3-15 9:459:00
categories: [rhcsa, note]
---

#### 首先破解密码进入系统


`nmtui，在里面配置一些基本信息。`

##### 第一题、修改root密码

1. ~~~ shell
      echo 'ooxx9527' | passwd --stdin root
      #打印密码，通过通道符 执行passwd 获取打印的密码，为root用户设置密码。
     ~~~

##### 第二题、设定selinux

1. ~~~ shell
     getenforce			#先查看一下当前selinux状态
     vi /etc/selinux/config		#用vi编辑SELINUX字段为enforcing
     ~~~

##### 第三题、设定yum仓库

1. ~~~ shell
     cd /etc/yum.repos.d/ && vi xxx.repo
     #则是两条命令，先进入目录，然后vi新建xxx.repo文件，写上如下内容
     [xxx]
     name=xxx
     baseurl=http://classroom.example.com/content/rhel7.0/x86_64/dvd
     gpgcheck=0
     enabled=1
     #xxx随便填写，baseurl一定要写题目要求的，enabled一定要是1，不然不生效。
     ~~~

##### 第四题、调整逻辑卷

1. ~~~ shell
     lvdisplay		#查看lv size有没有分配的空间
     vgdisplay 		#查看vg size有没有可以分配空间
     以上都没有，我先新建一个符合要求的pv
     ~~~

2. 新建pv 大小1G，能满足题目要求就行

     ~~~ shell
     fdisk /dev/sdb      		#看你的vg里面是那块硬盘
     # 	p		打印所有分区
     #	n		新建分区
     #			--p		主分区
     #			--e		扩展分区
     #	d		删除分区
     #	q		退出
     #	w		保存修改配置
     #	t		更改文件系统
     #		新建好的分区叫/dev/sdb2
     ~~~

3. 对新分区分配组并且调整lvm大小。

     `tips：新建的分区不能让系统识别，重启，或者用partprobe命令让系统重新识别。`

     ~~~ shell
     pvcreate /dev/sdb2		创建pv
     vgextend vg1 /dev/sdb2		把新分区加入到vg1
     lvresize -L 770M /dev/vg1/lvm1		调整lvm1的大小
     #	lvdisplay查看一下
     ~~~

     

##### 第五题、创建用户和用户组

~~~ shell
#创建组 -g  指定组id 
groupadd -g 40000 adminuser	
#创建用户 -G 加入附属组
useradd -G adminuser natasha
#useradd -s  指定shell
useradd -s /sbin/nologin sarah
#例：passwd natasha
passwd 用户名			设置密码
~~~

##### 第六题、文件权限设定

~~~ shell
cp /etc/fstab /var/tmp		#复制文件
~~~

1. 任何人没有执行权限

     ~~~ shell
     chmod  a-x 文件名           #所有人都没有执行权限
     ~~~

2. 针对用户的权限

     ~~~ shell
     setfacl -m u:natasha:rw /var/tmp/fstab		
     setfacl -m u:harry:--- /var/tmp/fstab
     getfacl /var/tmp/fstab		#查看文件acl
     #所有用户都有读的权限
     chmod a+x /var/tmp/fstab
     ~~~

##### 第七题、建立计划任务

~~~ shell
crontab -e -u natasha			#为natasha用户添加一个计划任务

23 14 * * * /bin/echo "rhcsa"  	#任务格式
~~~



