---
title: linux学习笔记3-实用脚本技巧
description: ""
slug: 2021-09-11-linux学习笔记3-实用脚本技巧
date: 2021-09-11
image: linux.jpg
categories:
  - 运维&云原生
tags:
  - Linux
---
**1、查找当前目录下所有以.tar结尾的文件然后移动到指定目录**  

```bash
find . -name “*.tar” -exec mv {}./backup/ ;
```

注解：find –name 主要用于查找某个文件名字，-exec 、xargs可以用来承接前面的结果，然后将要执行的动作，一般跟find在一起用的很多，find使用我们可以延伸-mtime查找修改时间、-type是指定对象类型（常见包括f代表文件、d代表目录），-size 指定大小，例如经常用到的：查找当前目录30天以前大于100M的LOG文件并删除。

```bash
find  . -name "*.log" –mtime +30 –typef –size +100M |xargs rm –rf {};
```

**2、批量解压当前目录下以.zip结尾的所有文件到指定目录**

```bash
for i  in  `find . –name “*.zip”–type f `

do

unzip –d $i /data/www/img/

done
```

注解：for I in （command）;do … done 为for循环的一个常用格式，其中I为变量，可以自己指定。

**3、sed常用命收集：test.txt做测试**

如何去掉行首的.字符:

```bash
sed -i ‘s/^.//g’ test.txt
```

在行首添加一个a字符:

```bash
sed ’s/^/a/g’    test.txt
```

在行尾添加一个a字符:

```bash
sed ’s/$/a/‘     tets.txt
```

在特定行后添加一个c字符:

```bash
sed ‘/wuguangke/ac’ test.txt
```

在行前加入一个c字符:

```bash
sed’/wuguangke/ic’ test.txt
```

**4、如何判断某个目录是否存在，不存在则新建，存在则打印信息。**

```bash
if

[! –d /data/backup/];then

Mkdir –p /data/backup/

else

echo  "The Directory alreadyexists,please exit"

fi
```

注解：if…;then …else ..fi：为if条件语句,!叹号表示反义“不存在“，-d代表目录。

**5、监控linux磁盘根分区，如果根分区空间大于等于90%，发送邮件给Linux SA**

(1)、打印根分区大小

```bash
df -h |sed -n '//$/p'|awk '{print $5}'|awk –F ”%” '{print $1}'
```

注解：awk ‘{print $5}’意思是打印第5个域，-F的意思为分隔，例如以%分隔，简单意思就是去掉百分号，awk –F. ‘{print $1}’分隔 `.`。

(2)、if条件判断该大小是否大于90，如果大于90则发送邮件报警

```bash
while sleep 5m

do

for i in `df -h |sed -n '//$/p' |awk '{print $5}' |sed 's/%//g'`

do

echo $i

if [ $i -ge 90 ];then

echo “More than 90% Linux of disk space ,Please LinuxSA Check Linux Disk !” |mail -s “Warn Linux / Parts is $i%” 

XXX@XXX.XX

fi

done

done
```

**6、统计 Nginx 访问日志，访问量排在前20 的 ip地址**

```bash
cat access.log |awk '{print $1}'|sort|uniq -c |sort -nr |head -20
```

注解：sort排序、uniq（检查及删除文本文件中重复出现的行列 ）

**7、sed另外一个用法找到当前行，然后在修改该行后面的参数**

```bash
sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
```

Sed冒号方式 sed -i ‘s:/tmp:/tmp/abc/:g’test.txt意思是将/tmp改成/tmp/abc/。

**8、打印出一个文件里面最大和最小值**

```bash
cat a.txt |sort -nr|awk ‘{}END{print} NR==1′

cat a.txt |sort -nr |awk ‘END{print} NR==1′
```

这个才是真正的打印最大最小值：sed ‘s/ / /g’ a.txt |sort -nr|sed -n ’1p;$p’

**9、使用snmpd抓取版本为v2的cacti数据方式**

```bash
snmpwalk -v2c -c public 192.168.0.241
```

**10、修改文本中以jk结尾的替换成yz**

```bash
sed -e ‘s/jk$/yz/g’ b.txt
```

**11、网络抓包：tcpdump**

```bash
tcpdump -nn host 192.168.56.7 and port 80 抓取56.7通过80请求的数据包。

tcpdump -nn host 192.168.56.7 or ! host 192.168.0.22 and port 80 排除0.22 80端口！

tcp/ip 7层协议物理层–数据链路层-网络层-传输层-会话层-表示层-应用层。
```

**12、显示最常用的20条命令**

```bash
cat .bash_history |grep -v ^# |awk ‘{print $1}’ |sort |uniq -c |sort -nr |head-20
```

**13、写一个脚本查找最后创建时间是3天前，后缀是\*.log的文件并删除。**

```bash
find . -mtime +3  -name "*.log" |xargs rm -rf {} ;
```

**14、写一个脚本将某目录下大于100k的文件移动至/tmp下。**

```bash
find . -size +100k -exec mv {} /tmp ;
```

**15、写一个防火墙配置脚本，只允许远程主机访问本机的80端口。**

```bash
iptables -F

iptables -X

iptables -A INPUT -p tcp --dport 80 -j accept

iptables -A INPUT -p tcp -j REJECT
```

或者

```bash
iptables -A INPUT -m state --state NEW-m tcp -p tcp --dport 80 -j ACCEPT
```

**16、写一个脚本进行nginx日志统计，得到访问ip最多的前10个(nginx日志路径**

```bash
/home/logs/nginx/default/access.log。

cd /home/logs.nginx/default

sort -m -k 4 -o access.logok access.1 access.2 access.3 .....

cat access.logok |awk '{print $1}'|sort -n|uniq -c|sort -nr |head -10
```

**17.替换文件中的目录**

```bash
sed 's:/user/local:/tmp:g' test.txt
```

或者

```bash
sed -i 's//usr/local//tmp/g' test.txt
```
