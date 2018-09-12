# 1. NFS简介

NFS，是Network File System的简写，即网络文件系统。网络文件系统是FreeBSD支持的文件系统中的一种，也被称为NFS. NFS允许一个系统在网络上与他人共享目录和文件。
通过使用NFS，用户和程序可以像访问本地文件一样访问远端系统上的文件。

# 2. NFS的安装与配置

## 2.1 服务端

NFS需要安装nfs-utils、rpcbind两个包。
```bash
#可以先检查下本地是否已经安装，如果安装则无需重复安装包
[root@k8s-dbg-master-1 build]# rpm -qa|grep rpcbind
rpcbind-0.2.0-42.el7.x86_64

[root@k8s-dbg-master-1 build]# rpm -qa|grep nfs
libnfsidmap-0.25-17.el7.x86_64
nfs-utils-1.3.0-0.48.el7_4.x86_64
```

### 2.1.1. 安装nfs-utils、rpcbind两个包

```bash
#centos系统
yum -y install nfs-utils rpcbind

#Ubuntu系统
#服务端
apt-get install nfs-kernel-server
#客户端
apt-get install nfs-common
```

### 2.1.2. 创建共享目录

服务端共享目录：`/data/nfs-storage/`

```bash
mkdir /data/nfs-storage/
```

### 2.1.3. NFS共享目录文件配置

```bash
vi /etc/exports 
#添加以下信息
/data/nfs-storage *(rw,insecure,sync,no_subtree_check,no_root_squash)
```

以上配置分为三个部分：

- 第一部分就是本地要共享出去的目录。
- 第二部分为允许访问的主机（可以是一个IP也可以是一个IP段），`*`代表允许所有的网段访问。
- 第三部分小括号里面的，为一些权限选项。

**权限说明**

- rw ：读写；
- ro ：只读；
- sync ：同步模式，内存中数据时时写入磁盘；
- async ：不同步，把内存中数据定期写入磁盘中；
- secure ：nfs通过1024以下的安全TCP/IP端口发送
- insecure ：nfs通过1024以上的端口发送
- no_root_squash ：加上这个选项后，root用户就会对共享的目录拥有至高的权限控制，就像是对本机的目录操作一样。不安全，不建议使用；
- root_squash ：和上面的选项对应，root用户对共享目录的权限不高，只有普通用户的权限，即限制了root；
- subtree_check ：如果共享/usr/bin之类的子目录时，强制nfs检查父目录的权限（默认）
- no_subtree_check ：和上面相对，不检查父目录权限
- all_squash ：不管使用NFS的用户是谁，他的身份都会被限定成为一个指定的普通用户身份；
- anonuid/anongid ：要和root_squash 以及 all_squash一同使用，用于指定使用NFS的用户限定后的uid和gid，前提是本机的/etc/passwd中存在这个uid和gid。

### 2.1.4. 启动NFS服务

```bash
#先启动rpcbind
service rpcbind start

#后启动nfs
service nfs start

#可以设置开机启动
chkconfig rpcbind on
chkconfig nfs on
```

### 2.1.5. 服务端验证

通过`showmount -e`命令如果正常显示共享目录，表示安装正常。

```bash
[root@k8s-dbg-master-1 build]# showmount -e
Export list for k8s-dbg-master-1:
/data/nfs-storage *
```

## 2.2 客户端

### 2.2.1. 安装nfs-utils的包

```bash
yum install nfs-utils.x86_64  -y
```

### 2.2.2. 创建挂载点

客户端挂载目录：`/mnt/store`

```bash
mkdir /mnt/store
```

### 2.2.3. 查看NFS服务器的共享

```bash
root@k8s-dbg-node-5:~# showmount -e 172.16.5.4
Export list for 172.16.5.4:
/data/nfs-storage *
```

### 2.2.4. 挂载

```bash
mount -t nfs <NFS_SERVER_IP>:<NFS_SERVER_SHARED_DIR> <NFS_CLIENT_MOUNT_DIR>

#例如：
mount -t nfs 172.16.5.4:/data/nfs-storage /mnt/store
```

### 2.2.5. 验证挂载信息

使用`mount`命令

```bash
root@k8s-dbg-node-5:~# mount |grep /mnt/store
172.16.5.4:/data/nfs-storage/k8s-storage/ssd on /mnt/store type nfs4 (rw,relatime,vers=4.0,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=172.16.200.24,local_lock=none,addr=172.16.5.4)
```

使用`df -h`命令

```bash
root@k8s-dbg-node-5:~# df -h|grep nfs
172.16.5.4:/data/nfs-storage                                                                                             40G   25G   13G  67% /mnt/store
```

创建文件测试

```bash
#进入客户端的挂载目录，创建文件
cd /mnt/store
touch test.txt

#进入服务端的共享目录，查看客户端创建的文件是否同步
cd /data/nfs-storage 
ls
```
