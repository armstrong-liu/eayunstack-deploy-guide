# OpenStack数据库备份

需要备份的数据库包括keystone、glance、cinder、nova、neutron、ceilometer。

> #### 重要
> 该命令需要在**Controller节点**执行

## 同时备份所有数据库

```
# mysqldump --opt --all-databases > openstack.sql
```

## 自动备份脚本

管理员可以通过cron服务实现周期性自动化备份，下面是个数据库备份的示例脚本，执行该脚本可以备份所有OpenStack数据库，同时删除7天前的备份。

```
#!/bin/bash
backup_dir="/var/lib/backups/mysql"
if ! [ -e ${backup_dir} ]; then
    mkdir -p ${backup_dir}
fi
filename="${backup_dir}/mysql-`hostname`-`eval date +%Y%m%d%H%M`.sql.gz"
# Dump the entire MySQL database
/usr/bin/mysqldump --opt --all-databases | gzip > $filename
# Delete backups older than 7 days
find $backup_dir -ctime +7 -type f -delete
```
