# OpenStack配置文件备份

需要备份的配置文件目录如下表所示：

|组件|配置文件目录|节点|
|----|----|----|
|nova|/etc/nova|controller</br>compute|
|glance|/etc/glance|controller|
|keystone|/etc/keystone|controller|
|cinder|/etc/cinder|controller|
|neutron|/etc/neutron|controller</br>compute|
|ceilometer|/etc/ceilometer|controller</br>compute|

## 备份配置文件

管理员可在各节点通过cron服务对相应配置文件进行周期性自动化备份。备份脚本示例：

```
#!/bin/bash
backup_dir="/var/lib/backups/profile"
filename="${backup_dir}/nova-`hostname`-`eval date +%Y%m%d`.tgz"
tar -czf $filename /etc/nova
```
