---
layout: post
title: mysqldump定时备份
categories: [MySql]
description: mysqldump定时备份
keywords: MySql
---

### 参考


www.cnblogs.com/eternal1025/p/8554225.html
解决shell中拼接字符串，后面的字符串覆盖前面的字符串 blog.csdn.net/hefrankeleyn/article/details/85287391
解决/tmp/crontab bad minute问题 blog.csdn.net/zwj695535100/article/details/50770996
mysqldump定时任务生成备份文件内容为空解决方法 blog.csdn.net/qq_34457768/article/details/80447010



### 脚本

```shell
#!/bin/bash
#保留备份个数，31天数据
number=15
#备份保存路径
backup_dir=/home/mysql/back
#日期
dd=`date +%Y-%m-%d-%H-%M-%S`
#备份工具
tool=/usr/local/mysql/bin/mysqldump
#用户名
username=root
#密码
password=elcncsyyh2019*
#将要备份的数据库
database_name=elunchun

#如果文件夹不存在则创建
if [ ! -d $backup_dir ]; 
then     
    mkdir -p $backup_dir; 
fi

#简单写法  mysqldump -u root -p123456 users > /root/mysqlbackup/users-$filename.sql
$tool -u $username -p$password $database_name > $backup_dir/$database_name-$dd.sql

#写创建备份日志
echo "create $backup_dir/$database_name-$dd.dupm" >> $backup_dir/log.txt

#找出需要删除的备份
delfile=`ls -l -crt  $backup_dir/*.sql | awk '{print $9 }' | head -1`

#判断现在的备份数量是否大于$number
count=`ls -l -crt  $backup_dir/*.sql | awk '{print $9 }' | wc -l`

if [ $count -gt $number ]
then
  #删除最早生成的备份，只保留number数量的备份
  rm $delfile
  #写删除文件日志
  echo "delete $delfile" >> $backup_dir/log.txt
fi
```
