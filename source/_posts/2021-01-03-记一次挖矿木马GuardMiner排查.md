---
title: 记一次挖矿木马GuardMiner排查
date: 2021-01-03 01:50:19
tags:
- GuardMiner
- 木马
- linux
categories:
- tools
---

记录一次服务器被GuardMiner木马攻击后的排查与解决过程
<!-- more -->
今天一早线上的服务挂了，登录服务器一看进程停了，日志也没有什么异常，于是重启服务观察观察，结果过了一会又被停了。觉得事有蹊跷，看了下top，发现有三个进程占用很高几乎占满了所有CPU：两个phpupdate进程+一个networkmanager进程(后来才知道是挖坑木马)。

#### 问题排查

杀掉三个进程再重启服务，大约一分钟后进程再次出现并又占满了cpu。于是根据进程号pid找到文件位置：` ll /proc/pid `   cwd指向的就是文件路径，发现都在/etc目录下，/etc下这几个文件 phpupdate、phpupdates、networkmanager 都是今早同一时间段出现的，猜测是服务器被攻击。

尝试删掉文件 `rm -f phpupdate phpupdates networkmanager `  失败，提示`Operation not permitted`，大概是文件属性被修改了，执行`chattr -i phpupdate` 后再次删除成功，但一段时间后文件再次出现，继续排查下定时任务或是否有守护进程

查看定时任务`crontab -l` 发现一条任务

` */30 * * * * sh /etc/newdat.sh >/dev/null 2>&1` 

用命令 `crontab -e` 编辑 删掉这条任务，同时删除/etc/newdat.sh文件

守护进程是一个叫phpguard的（忘记当时是怎么找到的了），它脚本也在/etc/下，赶紧杀掉进程并删除文件，再回头把前面的phpupdate、phpupdates、networkmanager、newdat等杀掉进程并删除文件

过段时间发现没有再出现可疑的文件或进程

#### 后续处理

病毒清理掉后，先修改密码（其实发现被攻击时就应该第一时间改密码）放止再次被植入病毒

`last  ` 看下登陆记录，发现只找到我自己刚才的登陆记录，不知是不是被清掉了

再 `lastb` 看下登陆失败的记录，好家伙，从这个月初开始就有人一直在暴破，`lastb | wc -l `看下一共十万条登陆失败的信息，而且有数十个不同的IP，到现在也一直在进行

遂对服务器做如下处理

1. 修改ssh端口、禁止密码登陆、采用密钥/证书登录

   先创建密钥对` ssh-keygen -t rsa` ，在`/root/.ssh/`(如果是其他用户就在/home/用户/.ssh)目录下会生成id_rsa、id_rsa.pub，再看目录下是否有authorized_keys文件，没有则创建一个，再将公钥id_rsa.pub内容追加到authorized_keys中

   ``` shell
   cd .ssh
   thouch authorized_keys
   cat id_rsa.pub >> authorized_keys
   cd ..
   chmod 700 .ssh
   chmod 600 .ssh/authorized_keys
   ```

   私钥文件id_rsa可以拷贝到你的客户端保管好，登陆时要用到

   修改ssh的配置文件`/etc/ssh/sshd_config` ，主要修改以下配置项

   ```shell
   Port 12345 #修改端口,默认是22
   
   RSAAuthentication yes #启用密钥登陆
   PubkeyAuthentication yes
   
   PasswordAuthentication no #禁用密码登陆，建议在尝试密钥登陆成功后再禁掉密码登陆
   ```

   ` service sshd restart` 重启ssh服务，在客户端用新的端口号和私钥登陆

2. 对多次尝试登陆并失败的ip进行封禁

   先把始终允许的ip加到`/etc/hosts.allow` 如下

   ```shell
   sshd:192.168.1.1:allow
   sshd:192.168.1.2:allow
   ```

    写个封ip的脚本` /usr/local/bin/secure_ssh.sh` ，从 `/var/log/secure` 中统计出登陆失败超过10次的ip加到` /etc/hosts.deny` 中

   ```shell
   #! /bin/bash
   cat /var/log/secure|awk '/Failed/{print $(NF-3)}'|sort|uniq -c|awk '{print $2"="$1;}' > /usr/local/bin/black.list
   for i in `cat  /usr/local/bin/black.list`
   do
     IP=`echo $i |awk -F= '{print $1}'`
     NUM=`echo $i|awk -F= '{print $2}'`
     if [ ${#NUM} -gt 1 ]; then
       grep $IP /etc/hosts.deny > /dev/null
       if [ $? -gt 0 ];then
         echo "sshd:$IP:deny" >> /etc/hosts.deny
       fi
     fi
   done
   ```

   ` crontab -e ` 编辑定时任务，每5分钟执行一次脚本
   
   ```shell
   */5 * * * * sh /usr/local/bin/secure_ssh.sh
   ```
   

