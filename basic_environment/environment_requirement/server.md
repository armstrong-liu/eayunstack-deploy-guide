# 服务器

搭建一个最小化的EayunStack环境需要的服务器数量及配置如下：

|数量|节点角色|MEM|NIC|DISK|
|----|----|----|----|----|
|1|fuel|64G|1G\*4|1T\*1|
|3|controller|64G|1G\*4 10G\*4|1T*1|
|2|compute|64G|1G\*2 10G\*4|1T*1|
|3|ceph-osd|64G|1G\*2 10G\*4|1T*8|
|1|mongo|64G|1G\*2|1T\*4|