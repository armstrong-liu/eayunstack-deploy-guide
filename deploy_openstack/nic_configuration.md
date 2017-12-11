# 配置节点网卡

发现并增加所有节点后，需要配置各节点的网卡角色。

> ###### 注意
> 为了实现网络高可用及增加网络带宽，我们为每种角色的网络分配两个物理网卡，并将这两个网卡绑定(bond)到一起。
> 在**网络配置**界面，勾选两块要绑定的网卡，点击**Bond Interfaces**按钮即可将两块物理网卡绑定到一起。

## Controller节点

* 勾选全部**Controller节点**，点击**网络配置**按钮。

  ![openstack_install_451](../images/openstack_install_451.png)

 > ###### 注意
 > Controller节点需要连接“Admin(PXE)”、“Private”、“管理”、“存储”、“公开”网络。“Ceph Cluster”标签按下图位置放置即可。

* 配置网卡角色，如下图所示。

  ![openstack_install_452](../images/openstack_install_452.png)

## Compute节点

* 勾选全部**Compute节点**，点击**网络配置**按钮。

  ![openstack_install_453](../images/openstack_install_453.png)
 > ###### 注意
 > Compute节点需要连接“Admin(PXE)”、“Private”、“管理”、“存储”网络。“Ceph Cluster”、“公开”标签按下图位置放置即可。

* 配置网卡角色，如下图所示。

  ![openstack_install_454](../images/openstack_install_454.png)

## Mongo节点

* 勾选全部**Mongo节点**，点击**网络配置**按钮。

  ![openstack_install_455](../images/openstack_install_455.png)
 > ###### 注意
 > Mongo节点需要连接“Admin(PXE)”、“管理”网络。“Ceph Cluster”、“公开”、“存储”、“Private”标签按下图位置放置即可。

* 配置网卡角色，如下图所示。

  ![openstack_install_456](../images/openstack_install_456.png)

## Ceph-osd节点

* 勾选全部**Ceph-osd节点**，点击**网络配置**按钮。

  ![openstack_install_457](../images/openstack_install_457.png)
 > ###### 注意
 > Ceph-osd节点需要连接“Admin(PXE)”、“管理”、“Ceph Cluster”、“存储”网络。“公开”、“Private”标签按下图位置放置即可。

* 配置网卡角色，如下图所示。

  ![openstack_install_458](../images/openstack_install_458.png)
