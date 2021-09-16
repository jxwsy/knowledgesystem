# MPI集群环境搭建及测试

[TOC]

## 1、准备工作

四台虚拟机：

	node1、node2、node3、node5

操作系统：

	centos7

在安装 MPICH 之前，分别执行以下命令：

	yum update（保证安装的源最新）
	yum install gcc（安装gcc编译器，用于编译C语言）
	yum install g++（安装g++编译器，用于编译C++）
	yum install gfortran（安装gfortran编译器，用于编译fortran语言）。

## 2、安装 MPICH

下载、解压：[https://www.mpich.org/static/downloads/](https://www.mpich.org/static/downloads/)

在各节点上，进入 `mpich-3.3/` 目录，分别执行如下命令：

```sh
./configure --prefix=/home/mpiuser/mpich
make 
make install
```

安装结束后，用 `which mpicc` 和 `which mpiexec` 检查安装是否成功。

安装后加入环境变量

```sh
vi /etc/profile

PATH=$PATH:/home/mpiuser/mpich/bin
export PATH      

source /home/mpiuser/.bashrc
```

## 3、设置各节点间的免密登录

## 4、建立和挂载 NFS 共享目录

前面几步我们安装了 MPICH 和实现了 SSH 免密登录，如果要执行程序的话，需要保证每个节点的相同目录中都生成可执行文件。

为解决该问题，我们可以采用建立共享目录的方案。

节点 | 角色
--|:---
node1 | 客户端
node2 | 客户端
node3 | 客户端
node5 | 服务器

在服务器：

```sh
yum -y install nfs-utils rpcbind
```

编辑 nfs 的配置文件 `/etc/export`，默认为空，地址与括号之间不能有空格。

```sh
/mnt/mpi_share 192.168.xxx.xx1/24(rw,no_root_squash,no_all_squash,sync,anonuid=1000,anongid=1000)
/mnt/mpi_share 192.168.xxx.xx2/24(rw,no_root_squash,no_all_squash,sync,anonuid=1001,anongid=1001)
/mnt/mpi_share 192.168.xxx.xx3/24(rw,no_root_squash,no_all_squash,sync,anonuid=1002,anongid=1002)

```

```sh
exportfs -r
```

顺序启动：

```sh
systemctl start rpcbind
systemctl start nfs
```

开机自启：

```sh
systemctl enable rpcbind 
systemctl enable nfs
```

在客户端：

```sh
yum -y install nfs-utils rpcbind
```

挂载服务器共享目录：

```sh
mount -t nfs 192.168.xxx.xx5:/mnt/mpi_share /mnt/mpi_share
```

开机自动挂载：

```sh
echo "mount -t nfs 192.168.xxx.xx5:/mnt/mpi_share /mnt/mpi_share -o proto=tcp -o nolock" >> /etc/rc.d/rc.local

chmod +x /etc/rc.d/rc.local
```

查看挂载盘：

```sh
[root@node1 mpi_share]# df -h
文件系统                        容量  已用  可用 已用% 挂载点
devtmpfs                        901M     0  901M    0% /dev
tmpfs                           912M     0  912M    0% /dev/shm
tmpfs                           912M  9.6M  903M    2% /run
tmpfs                           912M     0  912M    0% /sys/fs/cgroup
/dev/mapper/centos-root          22G   14G  8.7G   61% /
/dev/sda1                      1014M  149M  866M   15% /boot
tmpfs                           183M     0  183M    0% /run/user/0
192.168.253.132@tcp:/lustrefs    39G  2.5M   37G    1% /mnt/lustre
192.168.253.135:/mnt/mpi_share   17G  3.8G   14G   22% /mnt/mpi_share
```

## 5、测试

```sh
[root@node1 mpi_share]# cd /opt/mpich-3.3/examples/
[root@node1 examples]# ls
argobots  cpi.o         f77       icpi.c       Makefile.in      pmandel_service.c  spawn_merge_child1.c
child.c   cxx           f90       ircpi.c      parent.c         pmandel_spaserv.c  spawn_merge_child2.c
cpi       developers    hellow    Makefile     pmandel.c        pmandel_spawn.c    spawn_merge_parent.c
cpi.c     examples.sln  hellow.c  Makefile.am  pmandel_fence.c  README             srtest.c
```

在单节点上执行：

```sh
[root@node1 examples]# mpicc cpi.c -o cpi.out
[root@node1 examples]# mpirun -np 2 cpi.out 
Process 0 of 2 is on node1
Process 1 of 2 is on node1
pi is approximately 3.1415926544231318, Error is 0.0000000008333387
wall clock time = 0.021034
```

集群下执行：

```sh
# 新建文件 MPI_HOSTS
# 左边表示想要运行job的节点，右边表示使用的进程数
[root@node1 examples]# cat MPI_HOSTS
node1:2
node2:2
node3:2
node5:2

[root@node1 examples]# mpirun -n 8 -f MPI_HOSTS cpi
Process 1 of 8 is on node1
Process 2 of 8 is on node2
Process 4 of 8 is on node3
Process 0 of 8 is on node1
Process 3 of 8 is on node2
Process 5 of 8 is on node3
Process 7 of 8 is on node5
Process 6 of 8 is on node5
pi is approximately 3.1415926544231247, Error is 0.0000000008333316
wall clock time = 0.090022
```