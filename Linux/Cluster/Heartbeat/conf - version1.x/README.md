#### authkeys 权限设置（必须）
`chmod 600 authkeys`
#### 开启网卡多播
`ip link set <interface> multicast on`
#### 在两个HA节点上的haresources文件必须一致...
#### 由于服务资源是由集群服务管理的，因此不能够将服务设为开机启动...