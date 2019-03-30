# 1. 安装ceph-fuse

```
yum install -y ceph-fuse
```

如果安装失败，先执行以下命令，再执行上述安装命令

```
yum -y install epel-release


rpm -Uhv http://download.ceph.com/rpm-jewel/el7/noarch/ceph-release-1-1.el7.noarch.rpm
```

# 2. 配置客户端访问的key

mkdir /etc/ceph/
vi /etc/ceph/ceph.client.admin.keyring

```
[client.admin]
key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==
```

# 3. ceph-fuse 挂载

```
ceph-fuse -m <mons_IP1>:6789,<mons_IP2>:6789,<mons_IP3>:6789 -r <ceph集群中的目录> <宿主机目录> -o nonempty
```

例如：

```
# ceph-fuse -m 192.168.18.3:6789,192.168.18.4:6789,192.168.18.5:6789 -r /pvc-volumes /root/cephfsdir -o nonempty
2019-03-27 17:58:04.435985 7fc61b67cec0 -1 did not load config file, using default settings.
ceph-fuse[18051]: starting ceph client
2019-03-27 17:58:04.469144 7fc61b67cec0 -1 init, newargv = 0x55cecaba81c0 newargc=13
ceph-fuse[18051]: starting fuse
```

# 4. 查看是否挂载成功

```
# df -h
Filesystem Size Used Avail Use% Mounted on
...
ceph-fuse 1.6T  8.8G  1.6T   1% /root/cephfsdir
```

# 5. ceph-fuse命令说明

```
# ceph-fuse --help
2019-03-27 18:01:16.421376 7fae11998ec0 -1 did not load config file, using default settings.
usage: ceph-fuse [-m mon-ip-addr:mon-port] <mount point> [OPTIONS]
  --client_mountpoint/-r <root_directory>
                    use root_directory as the mounted root, rather than the full Ceph tree.

usage: ceph-fuse mountpoint [options]

general options:
    -o opt,[opt...]        mount options
    -h   --help            print help
    -V   --version         print version

FUSE options:
    -d   -o debug          enable debug output (implies -f)
    -f                     foreground operation
    -s                     disable multi-threaded operation

  --conf/-c FILE    read configuration from the given configuration file
  --id/-i ID        set ID portion of my name
  --name/-n TYPE.ID set name
  --cluster NAME    set cluster name (default: ceph)
  --setuser USER    set uid to user or uid (and gid to user's gid)
  --setgroup GROUP  set gid to group or gid
  --version         show version and quit
```
