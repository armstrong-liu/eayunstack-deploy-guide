# 部署步骤

## Fuel 节点安装 LMA 插件

```
# fuel plugins --install influxdb_grafana-0.7.2.fp
# fuel plugins --install lma_collector-0.7.3.fp
```

>  需要将上面两个插件的最新版本下载到当前目录下

## Fuel 节点创建虚拟机

* 在 Fuel 节点安装软件包

```
# yum -y install libvirt qemu
```

* 启动 libvirtd 服务

```
# systemctl start libvirtd
# systemctl enable libvirtd
```

* 创建 monitor 虚拟网络

```
# cat >> /tmp/monitor.xml << EOF
<network>
  <name>monitor</name>
  <ip address='192.168.254.1' netmask='255.255.255.0'>
  </ip>
</network>
EOF
# virsh net-define /tmp/monitor.xml
# virsh net-autostart monitor
# virsh net-start monitor
```

* 使用 virt-manager 或其他虚拟机管理工具连接 Fuel 节点，新建虚拟机

>  **注意**
>
>  虚拟机需要创建三块虚拟网卡：
>   第一块虚拟网卡桥接到 Fuel 节点连接 PXE 网络的物理网卡上；
>   第二块虚拟网卡桥接到 Fuel 节点连接外部网络的网卡上：
>   第三块虚拟网卡桥接到 monitor 网络上；

## 安装操作系统

在虚拟机上安装 CentOS 7 操作系统。

## 系统基本配置

系统安装完成后，需要对系统进行以下基本配置。

* 网络配置
    * 通过修改网卡配置文件（/etc/sysconfig/network-scripts/ifcfg-***）对网络进行配置
    * 第一块网卡 IP 地址配置为 PXE 网络中的预留 IP 地址（Fuel 节点配置 PXE 服务时所预留的 IP 地址），如：10.10.0.254/24
    * 第二块网卡 IP 地址配置为外部网络中的任意网络地址，如：25.0.2.254/24
    * 第三块网卡 IP 地址配置为 monitor 网络里面的任意一个可用 IP 地址，如：192.168.254.2/24（需要跟前面创建的 monitor 网络的 IP 地址在同一网段） 

* 防火墙配置
    * 安装 iptables-services 软件包

    ```
    # yum -y install iptables-services
    ```

    * 关闭并禁用 firewalld 服务

    ```
    # systemctl stop firewalld
    # systemctl disable firewalld
    ```

* 添加一条到 Fuel 节点的连接 PXE 网络的网卡的 IP 的路由

```
# ip route add 10.10.0.2/32 via 192.168.254.1
```

>  "10.10.0.2"为 Fuel 节点连接 PXE 网络的网卡的 IP 地址
>
>  "192.168.254.1"为前面步骤中在 Fuel 节点上创建的 monitor 网络的 IP 地址

## 在 Fuel 节点添加一条路由

```
# ip route add 10.10.0.254/32 via 192.168.254.2
```
>  "10.10.0.254"为虚拟机的第一块网卡的 IP 地址；
>
>  "192.168.254.2"为虚拟机第三块网卡的 IP 地址

## 在 Fuel 节点添加一条防火墙规则

```
# iptables -t nat -I PREROUTING -s 192.168.254.2 -p tcp --dport 873 -j DNAT --to-destination 172.16.100.2:873
# cp /etc/sysconfig/iptables{,.bak-`date +%F-%H-%M-%S`}
# iptables-save > /etc/sysconfig/iptables
```

>  "192.168.254.2"为前面步骤中创建的虚拟机的第三块网卡的 IP 地址
>
>  "172.16.100.2"为 Fuel 节点连接 PXE 网络的网卡的 IP 地址

## 部署 influxdb/grafana 服务

influxdb 数据库服务用于存放所有监控数据，grafana 为监控平台的 WEB UI。

* 登陆 Fuel 节点，确认要部署监控平台的环境的环境 ID

```
[root@fuel ~](fuel)# fuel node list
id | status | name             | cluster | ip         | mac               | roles      | pending_roles | online | group_id
---|--------|------------------|---------|------------|-------------------|------------|---------------|--------|---------
8  | ready  | Untitled (db:a1) | 1       | 10.20.0.13 | 26:6f:ac:f7:cf:4d | mongo      |               | True   | 1       
2  | ready  | Untitled (00:30) | 1       | 10.20.0.12 | f2:42:eb:5d:71:42 | controller |               | True   | 1       
7  | ready  | Untitled (35:d7) | 1       | 10.20.0.3  | 72:80:a7:6d:8e:4b | ceph-osd   |               | True   | 1       
5  | ready  | Untitled (b2:4b) | 1       | 10.20.0.5  | ee:32:5a:7b:f3:4d | ceph-osd   |               | True   | 1       
4  | ready  | Untitled (42:3e) | 1       | 10.20.0.8  | 0e:d7:90:ee:c0:43 | compute    |               | True   | 1       
1  | ready  | Untitled (3c:2a) | 1       | 10.20.0.10 | 9e:a9:62:d2:8f:40 | controller |               | True   | 1       
6  | ready  | Untitled (70:0b) | 1       | 10.20.0.11 | 52:fd:fe:ab:bb:4a | ceph-osd   |               | True   | 1       
3  | ready  | Untitled (cf:d3) | 1       | 10.20.0.7  | ae:32:74:02:3b:47 | controller |               | True   | 1       
```

通过以上信息可以确认环境 ID 为 "1"（ cluster 列所标示 ID）

* 执行命令，部署 influxdb/grafana 服务

```
[root@fuel ~](fuel)# eayunstack fuel deployment_monitor_plugins --env 1 --influxdb
Please entry the ip address of influxdb node: 10.20.0.250
Please entry the root\'s password of influxdb node: 
[ INFO  ] (fuel) (fuel.domain.tld): Generate conf file .
Please entry influxdb_dbname [lma]: lma
Please entry influxdb_rootpass [admin]: admin
Please entry influxdb_username [lma]: lma
Please entry influxdb_userpass [lmapass]: lmapass
[ INFO  ] (fuel) (fuel.domain.tld): Push conf file to influxdb node.
[ INFO  ] (fuel) (fuel.domain.tld): Install rpm packages "puppet rsync" on node 10.20.0.250.
[ INFO  ] (fuel) (fuel.domain.tld): Deploy influxdb/grafana on node 10.20.0.250 .
[ INFO  ] (fuel) (fuel.domain.tld): Apply manifest check_environment_configuration.pp on node 10.20.0.250 .
[ INFO  ] (fuel) (fuel.domain.tld): Apply manifest eayunstack_netconfig.pp on node 10.20.0.250 .
[ INFO  ] (fuel) (fuel.domain.tld): Apply manifest firewall.pp on node 10.20.0.250 .
[ INFO  ] (fuel) (fuel.domain.tld): Apply manifest setup_influxdir.pp on node 10.20.0.250 .
[ INFO  ] (fuel) (fuel.domain.tld): Apply manifest influxdb.pp on node 10.20.0.250 .
[ INFO  ] (fuel) (fuel.domain.tld): Apply manifest grafana.pp on node 10.20.0.250 .
```

>  **提示**
>
>  部署成功后，可以通过浏览器访问监控平台服务器外网 IP 地址（如：25.0.2.254）访问监控平台 WEB UI。

## 部署 lma_collector 服务

lma_collector 服务运行在所有 OpenStack 节点，用于实时收集节点信息，并将信息发送到监控节点。

* 执行命令，部署 lma_collector 服务

```
[root@fuel ~](fuel)# eayunstack fuel deployment_monitor_plugins --env 1 --lma_collector
[ INFO  ] (fuel) (fuel.domain.tld): Checking all openstack node is online ...
[ INFO  ] (fuel) (fuel.domain.tld): Generate openstack node conf file.
[ INFO  ] (fuel) (fuel.domain.tld): Push conf file to openstack node.
[ INFO  ] (fuel) (fuel.domain.tld): Create symbolic links on openstack node.
[ INFO  ] (fuel) (fuel.domain.tld): Rsync plugin modules on openstack node.
[ INFO  ] (fuel) (fuel.domain.tld): Deployment lma_collector on openstack nodes.
```

到此，监控平台部署完成。