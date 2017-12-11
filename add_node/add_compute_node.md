# 添加 Compute(计算) 节点

> 注意
>
> 添加计算节点前，需要对 Fuel 节点、OpenStack 数据库及所有节点配置文件进行备份。

## 节点添加流程

### 确认环境运行正常

在 Fuel 节点执行 ```# eayunstack doctor all``` 命令，确认环境运行正常。

### 服务器上架

### 新节点网线连接

* 千兆网卡 1、2 连接到 PXE + Management 网络
* 万兆网卡 1、2 连接到 Storage 网络
* 万兆网卡 3、4 连接到 Private 网络

### 设置新节点从第一块千兆网卡引导

### 启动新节点

### 在 Fuel WEB 管理界面增加新节点

> 注意
>
> 增加节点时，注意确认新增节点的 MAC 地址

登陆 Fuel WEB 管理界面，等待自动发现新节点后，点击 **增加节点** 按钮，将新节点添加到环境中，注意节点角色选择 **Compute**。

### 新节点网络配置

> 注意
> 
> 以下配置在 Fuel WEB 管理界面新节点的 **网络配置** 界面完成。
> 
> 确认网卡顺序时 **以 MAC 地址为准**

* 千兆网卡 1、2 绑定，模式为 Active Backup，网卡角色 PXE + Management
* 千兆网卡 3，网卡角色 Public
* 千兆网卡 4，网卡角色 Ceph Cluster
* 万兆网卡 1、2 绑定，模式为 LACP Balance TCP，网卡角色 Storage
* 万兆网卡 3、4 绑定，模式为 LACP Balance TCP，网卡角色 Private

> 注意
>
> 千兆网卡 3、4 不需要连接网线！此处只是为了放置未使用的网络角色标签。

### 部署变更

> 警告
>
> 部署变更过程中，如果出现错误，**不要点击 stop 按钮**！等待自动完成后进行错误处理！

以上步骤确认无误后，点击 Fuel WEB 管理界面 **部署变更** 按钮。

### 确认新节点添加成功

* 登陆 Fuel WEB 管理界面，确认新添加节点状态为“已就绪”
* 登陆任意 Controller 节点命令行，执行 ```nova service-list``` 命令，确认新添加节点 ```Status``` 为 ```enable``` 、 ```State``` 为 ```up```。

### EayunStack 环境重新初始化

新节点添加完成后，需要在 Fuel 节点执行 ```# eayunstack init``` 命令对环境进行初始化。

## 节点升级

新 Compute 节点添加完成后，需要升级到最新版本。

### 执行 eayunstack upgrade

通过 eayunstack tools 对环境进行升级，具体操作方法参考《EayunStack运维手册》“环境升级”章节。

### 升级所有节点上的 eayunstack tools 到最新版本

```
(fuel)# eayunstack init -u
```

### 配置新节点 novncproxy_base_url

修改新添加的 Compute 节点的 ```/etc/nova/nova.conf``` 配置文件，将 ```novncproxy_base_url``` 配置为与其他节点相同的值。

### 确认环境运行正常

在 Fuel 节点执行 ```# eayunstack doctor all``` 命令，确认环境运行正常。