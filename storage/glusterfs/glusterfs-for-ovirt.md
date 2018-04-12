# 创建ovirt使用的glusterfs存储

- 参考：https://blog.csdn.net/wh211212/article/details/79412081

## 节点准备C7

- server1 server2

# server1，server2 执行
lvcreate -n glusterfs -L 50G centos
mkfs.xfs -i size=512 /dev/mapper/centos-glusterfs 
mkdir -p /data/brick1
echo '/dev/mapper/centos-glusterfs  /data/brick1 xfs defaults 1 2' >> /etc/fstab
mount -a && mount

yum install glusterfs-server -y && systemctl start glusterd && systemctl enable glusterd && systemctl status glusterd

## 配置可信池

- server1上执行

gluster peer probe server2

gluster peer status # 检查server1,server2上的对等状态

## 设置GlusterFS卷

- 在server1和server2上执行

mkdir -p /data/brick1/gv0  # data
mkdir -p /data/brick1/gv1  # iso
mkdir -p /data/brick1/gv2  # export

chown vdsm:kvm /data/brick1 -R # 为了ovirt挂载使用，不然添加glusterfs的时候报没有权限

- 任意节点上执行：

gluster volume create gv0 replica 2 server1:/data/brick1/gv0 server2:/data/brick1/gv0
gluster volume create gv1 replica 2 server1:/data/brick1/gv1 server2:/data/brick1/gv1
gluster volume create gv2 replica 2 server1:/data/brick1/gv2 server2:/data/brick1/gv2

gluster volume start gv0
gluster volume start gv1
gluster volume start gv2

- 确认volume“已启动”

gluster volume info

## ovirt engine所在机器执行

yum -y install glusterfs glusterfs-fuse

然后到控制台添加存储域