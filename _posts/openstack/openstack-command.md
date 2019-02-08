OpenStack 命令行速查表
  
UPDATED: 2017-07-18 08:53
Contents
认证 (keystone)
镜像（glance）
计算 (nova)
实例的暂停、挂起、停止、救援、调整规格、重建、重启
网络 (neutron)
块存储(cinder)
对象存储(Swift)
这是可供参考的常用命令列表

认证 (keystone)¶
列出所有的用户

$ openstack user list
列出认证服务目录

$ openstack catalog list
镜像（glance）¶
列出您可以访问的镜像

$ openstack image list
删除指定的镜像

$ openstack image delete IMAGE
描述一个指定的镜像

$ openstack image show IMAGE
更新镜像

$ openstack image set IMAGE
上传内核镜像

$ openstack image create "cirros-threepart-kernel" \
  --disk-format aki --container-format aki --public \
  --file ~/images/cirros-0.3.5-x86_64-kernel
上传RAM镜像

$ openstack image create "cirros-threepart-ramdisk" \
  --disk-format ari --container-format ari --public \
  --file ~/images/cirros-0.3.5-x86_64-initramfs
上传第三方镜像

$ openstack image create "cirros-threepart" --disk-format ami \
  --container-format ami --public \
  --property kernel_id=$KID-property ramdisk_id=$RID \
  --file ~/images/cirros-0.3.5-x86_64-rootfs.img
注册raw镜像

$ openstack image create "cirros-raw" --disk-format raw \
  --container-format bare --public \
  --file ~/images/cirros-0.3.5-x86_64-disk.img
计算 (nova)¶
列出实例，核实实例状态

$ openstack server list
列出镜像

$ openstack image list
Create a flavor named m1.tiny

$ openstack flavor create --ram 512 --disk 1 --vcpus 1 m1.tiny
列出规格类型

$ openstack flavor list
用类型和镜像名称(如果名称唯一)来启动云主机

$ openstack server create --image IMAGE --flavor FLAVOR INSTANCE_NAME
$ openstack server create --image cirros-0.3.5-x86_64-uec --flavor m1.tiny \
  MyFirstInstance
Log in to the instance (from Linux)

 注解

The ip command is available only on Linux. Using ip netns provides your environment a copy of the network stack with its own routes, firewall rules, and network devices for better troubleshooting.

# ip netns
# ip netns exec NETNS_NAME ssh USER@SERVER
# ip netns exec qdhcp-6021a3b4-8587-4f9c-8064-0103885dfba2 \
  ssh cirros@10.0.0.2
 注解

In CirrOS, the password for user cirros is cubswin:). For any other operating system, use SSH keys.

Log in to the instance with a public IP address (from Mac)

$ ssh cloud-user@128.107.37.150
显示实例详细信息

$ openstack server show NAME
$ openstack server show MyFirstInstance
查看云主机的控制台日志

$ openstack console log show MyFirstInstance
设置云主机的元数据

$ nova meta volumeTwoImage set newmeta='my meta data'
创建一个实例快照

$ openstack image create volumeTwoImage snapshotOfVolumeImage
$ openstack image show snapshotOfVolumeImage
实例的暂停、挂起、停止、救援、调整规格、重建、重启¶
暂停

$ openstack server pause NAME
$ openstack server pause volumeTwoImage
取消挂起

$ openstack server unpause NAME
挂起

$ openstack server suspend NAME
Unsuspend

$ openstack server resume NAME
关机

$ openstack server stop NAME
开始

$ openstack server start NAME
恢复

$ openstack server rescue NAME
$ openstack server rescue NAME --rescue_image_ref RESCUE_IMAGE
调整大小

$ openstack server resize NAME FLAVOR
$ openstack server resize my-pem-server m1.small
$ openstack server resize --confirm my-pem-server1
重建

$ openstack server rebuild NAME IMAGE
$ openstack server rebuild newtinny cirros-qcow2
重启

$ openstack server reboot NAME
$ openstack server reboot newtinny
将用户数据和文件注入到实例

$ openstack server create --user-data FILE INSTANCE
$ openstack server create --user-data userdata.txt --image cirros-qcow2 \
  --flavor m1.tiny MyUserdataInstance2
使用ssh连接到实例，查看``/var/lib/cloud``验证文件是否成功注入

给实例注入一个密钥对并通过密钥对来访问实例

创建秘钥对

$ openstack keypair create test > test.pem
$ chmod 600 test.pem
启动实例

$ openstack server create --image cirros-0.3.5-x86_64 --flavor m1.small \
  --key-name test MyFirstServer
使用ssh连接到实例

# ip netns exec qdhcp-98f09f1e-64c4-4301-a897-5067ee6d544f \
  ssh -i test.pem cirros@10.0.0.4
管理安全组

在默认的安全组中，添加ping和SSH规则

$ openstack security group rule create default \
    --remote-group default --protocol icmp
$ openstack security group rule create default \
    --remote-group default --dst-port 22
网络 (neutron)¶
创建网络

$ openstack network create NETWORK_NAME
创建子网

$ openstack subnet create --subnet-pool SUBNET --network NETWORK SUBNET_NAME
$ openstack subnet create --subnet-pool 10.0.0.0/29 --network net1 subnet1
块存储(cinder)¶
用于管理连接到实例的卷和卷快照。

创建一个新卷

$ openstack volume create --size SIZE_IN_GB NAME
$ openstack volume create --size 1 MyFirstVolume
启动实例并将它链接到卷上

$ openstack server create --image cirros-qcow2 --flavor m1.tiny MyVolumeInstance
列出所有卷，注意卷状态

$ openstack volume list
当实例为正常状态且卷为可用状态时，将卷连接到实例。

$ openstack server add volume INSTANCE_ID VOLUME_ID
$ openstack server add volume MyVolumeInstance 573e024d-5235-49ce-8332-be1576d323f8
 注解

在Xen Hypervisor可以指定具体的设备名，而不使用自动分配的名称，例如：

$ openstack server add volume --device /dev/vdb MyVolumeInstance 573e024d..1576d323f8

This is not currently possible when using non-Xen hypervisors with OpenStack.
登陆进实例之后管理卷组

列出存储器

# fdisk -l
在卷上建立文件系统

# mkfs.ext3 /dev/vdb
创建一个挂载点

# mkdir /myspace
在挂载点挂载卷

# mount /dev/vdb /myspace
在卷上创建一个文件

# touch /myspace/helloworld.txt
# ls /myspace
卸载卷

# umount /myspace
对象存储(Swift)¶
展示账户，容器以及对象的信息

$ swift stat
$ swift stat ACCOUNT
$ swift stat CONTAINER
$ swift stat OBJECT
列出容器

$ swift list

