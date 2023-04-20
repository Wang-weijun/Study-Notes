# Docker

Docker通过隔离机制，可以将服务器利用到了极致。



> 容器化技术

==容器化技术不是模拟的一个完整的操作系统==





# Docker基本组成

**镜像（image）：**

docker镜像好比一个模板，可以通过这个模板来创建容器服务，tomcat镜像==>》run ==》 tomcat01容器（提供服务器），通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中）。

**容器（container）：**

Docker利用容器技术，独立运行一个或者一个组应用，通过镜像来创建的。

启动，停止，删除，基本命令！

**仓库（repository）：**

仓库就是存放镜像的地方！





# Docker安装

> 环境准备 CentOS 7



> 环境查看

~~~shell
# 系统内核是 3.10 以上的
[root@Wang /]# uname -r
3.10.0-957.21.3.el7.x86_64

~~~

~~~shell
# 系统环境
[root@Wang /]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

~~~



## 安装

帮助文档：

~~~shell
# 1、卸载旧的版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 2、需要的安装包
yum install -y yum-utils

# 3、设置镜像仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo  # 默认是从国外的！

yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 推荐使用阿里云的

# 更新yum软件包索引
yum makecache fast

# 4、安装docker相关的内容 docker-ce 社区 
yum install docker-ce docker-ce-cli containerd.io

# 5、启动docker
systemctl start docker

# 6、使用 docker version 是否安装成功！
~~~

![image-20230130172339188](C:\Users\Admin\Desktop\笔记\img\image-20230130172339188.png)

~~~shell
# 7、hello-world
docker run hello-world
~~~

![image-20230130172623721](C:\Users\Admin\Desktop\笔记\img\image-20230130172623721.png)

~~~shell
# 8、查看一下下载的这个hello-world镜像
[root@Wang ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   16 months ago   13.3kB
~~~

了解：卸载docker

~~~shell
# 1、卸载依赖
yum remove docker-ce docker-ce-cli containerd.io

# 2、删除资源
rm -rf /var/lib/docker	# docker默认工作路径
rm -rf /var/lib/containerd
~~~



## 阿里云镜像加速

1、登录阿里云找到容器镜像服务

![image-20230130173819786](C:\Users\Admin\Desktop\笔记\img\image-20230130173819786.png)

2、配置使用

~~~shell
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://qfjygl3u.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
~~~



# 底层原理

**Docker是怎么工作的？**

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问！

DockerServer接受到Docker-Client的指令，就会执行这个命令！



# Docker的常用命令

## 帮助命令

~~~shell
docker version		# 显示docker版本信息
docker info			# 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help  # 帮助命令
~~~

帮助文档的地址：https://docs.docker.com/reference/



## 镜像命令

**docker images ** 查看所有本地的主机上的镜像

~~~shell
[root@Wang ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   16 months ago   13.3kB

# 解释
REPOSITORY	镜像的仓库源
TAG			镜像的标签
IMAGE ID	镜像的ID
CREATED		镜像的创建时间
SIZE		镜像的大小
~~~

**docker search** 搜索镜像

~~~shell
[root@Wang ~]# docker search mysql
NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                           MySQL is a widely used, open-source relation…   13741     [OK]    
mariadb                         MariaDB Server is a high performing open sou…   5242      [OK]       
phpmyadmin                      phpMyAdmin - A web interface for MySQL and M…   727       [OK]       
percona                         Percona Server is a fork of the MySQL relati…   599       [OK]
~~~

**docker pull** 下载镜像

~~~shell
# 下载镜像 docker pull 镜像名:tag
[root@Wang ~]# docker pull mysql

# 指定版本下载
[root@Wang ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
72a69066d2fe: Pull complete 
93619dbc5b36: Pull complete 
99da31dd6142: Pull complete 
626033c43d70: Pull complete 
37d5d7efb64e: Pull complete 
ac563158d721: Pull complete 
d2ba16033dad: Pull complete 
0ceb82207cd7: Pull complete 
37f2405cae96: Pull complete 
e2482e017e53: Pull complete 
70deed891d42: Pull complete 
Digest: sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f94
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

~~~

**docker rmi** 删除镜像

~~~shell
docker rmi -f 镜像id		# 删除指定的镜像
docker rmi -f 镜像id 镜像id 镜像id 镜像id	# 删除多个镜像

[root@Wang ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mysql         5.7       c20987f18b13   13 months ago   448MB
hello-world   latest    feb5d9fea6a5   16 months ago   13.3kB
# 删除hello-world 通过id删除
[root@Wang ~]# docker rmi -f feb5d9fea6a5
Untagged: hello-world:latest
Untagged: hello-world@sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
~~~



## 容器命令

说明：我们有了镜像才可以创建容器

~~~shell
docker pull centos
~~~

新建镜像并启动

~~~shell
docker run [可选参数] image

# 参数说明
--name="Name"	容器名字 tomcat01 tomcat02，用来区分
-d				后台方式运行
-it				使用交互方式运行，进入容器查看内容
-p				指定容器的端口	-p 8080:8080
	-p 主机端口:容器端口（常用）
	-p 容器端口
-P				随机指定端口

# 测试，启动并进入容器
[root@Wang ~]# docker run -it centos /bin/bash
[root@a1afe9f99dd6 /]# ls	# 查看容器内的centos，基本版本，很多命令不完善
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 从容器中退回主机
[root@a1afe9f99dd6 /]# exit
exit
[root@Wang /]# ls
bin   data  etc   lib    lost+found  mnt  patch  root  sbin  sys  usr  www
boot  dev   home  lib64  media       opt  proc   run   srv   tmp  var


docker run -p 1234:1234 -it fed440038981 /bin/bash
docker exec -it 354e4cf9e32f /bin/bash

~~~

**列出所有的运行的容器**

~~~shell
# docker ps 命令
-a	# 列出当前正在运行的容器+带出历史运行过的容器

[root@Wang ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED             STATUS                         PORTS     NAMES
a1afe9f99dd6   centos         "/bin/bash"   About an hour ago   Exited (0) About an hour ago             flamboyant_booth
904a45d59a92   feb5d9fea6a5   "/hello"      5 hours ago         Exited (0) 5 hours ago                   focused_lalande
~~~

**退出容器**

~~~shell
exit	# 容器退出并停止
Ctrl + P + Q	# 容器不停止退出
~~~



**删除容器**

~~~shell
docker rm 容器id	# 删除指定的容器，不能删除正在运行的容器，如果强制删除 rm -f
docker rm -f $(docker ps -aq)	# 删除所有的容器
~~~

**启动和停止容器的操作**

~~~shell
docker start 容器id		# 启动容器
docker restart 容器id		# 重启容器
docker stop 容器id		# 停止容器
docker kill 容器id		# 强制停止当前容器
~~~



## 常用其他命令

**后台启动容器**

~~~shell
# 命令 docker run -d 镜像名
[root@Wang ~]# docker run -d centos

# 问题docker ps，发现centos停止了

# 常见的坑，容器使用后台运行后，就必须要有一个前台进程，docker发现没有应用，就会自动停止
# nginx，容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
~~~

**查看日志**

~~~shell
docker logs -f -t --tail 容器
~~~

**查看容器中进程信息**ps

~~~shell
docker top 
[root@Wang ~]# docker top 5cac1fea830f
'UID	PID		PPID	C	STIME	TTY		TIME	CMD
root	30121         30100         0          22:42               pts/0         00:00:00            /bin/bash

~~~

**查看镜像的元数据**

~~~shell
# 命令docker inspect 容器id
[
    {
        "Id": "88d23bcbe1f28e1f8ae5d2b63fa8d57d2abcbdacf193db05852f4f74a95b9ffe",
        "Created": "2022-07-29T08:23:56.862239223Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo kuangshen;sleep 1;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 21212,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2022-07-29T08:23:57.109766809Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6",
        "ResolvConfPath": "/var/lib/docker/containers/88d23bcbe1f28e1f8ae5d2b63fa8d57d2abcbdacf193db05852f4f74a95b9ffe/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/88d23bcbe1f28e1f8ae5d2b63fa8d57d2abcbdacf193db05852f4f74a95b9ffe/hostname",
        "HostsPath": "/var/lib/docker/containers/88d23bcbe1f28e1f8ae5d2b63fa8d57d2abcbdacf193db05852f4f74a95b9ffe/hosts",
        "LogPath": "/var/lib/docker/containers/88d23bcbe1f28e1f8ae5d2b63fa8d57d2abcbdacf193db05852f4f74a95b9ffe/88d23bcbe1f28e1f8ae5d2b63fa8d57d2abcbdacf193db05852f4f74a95b9ffe-json.log",
        "Name": "/silly_lichterman",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/59b088d44f6e67f4ed336de44d19a5784c1a13fa856760bd2b166c4a4d421e2b-init/diff:/var/lib/docker/overlay2/7fc43e24b63e4656ab7e7718d3e4ef5297fe82509452be305a01605a7cdc3b97/diff",
                "MergedDir": "/var/lib/docker/overlay2/59b088d44f6e67f4ed336de44d19a5784c1a13fa856760bd2b166c4a4d421e2b/merged",
                "UpperDir": "/var/lib/docker/overlay2/59b088d44f6e67f4ed336de44d19a5784c1a13fa856760bd2b166c4a4d421e2b/diff",
                "WorkDir": "/var/lib/docker/overlay2/59b088d44f6e67f4ed336de44d19a5784c1a13fa856760bd2b166c4a4d421e2b/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "88d23bcbe1f2",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo kuangshen;sleep 1;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20210915",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "bfc28d2ba671adfe5c93325173b93c335d50d8b017eed4fdce2ab75410d6ac2e",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/bfc28d2ba671",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "ad0252d454e751e2c5ecef24e64c32b0884c53242925bd66e9e5dbf5542af179",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "7254ffccbdf53e0c72c1d19252980f04fa65153ea3c7ca72af55cfac504cbe3f",
                    "EndpointID": "ad0252d454e751e2c5ecef24e64c32b0884c53242925bd66e9e5dbf5542af179",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
~~~

**进入当前正在运行的容器**

~~~shell
# 我们通常容器都是使用后台方式运行的，需要进入容器，修改一些配置
# 命令
docker exec -it 容器id /bin/bash
# 测试
[root@iZbp13qr3mm4ucsjumrlgqZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
88d23bcbe1f2   centos    "/bin/sh -c 'while t…"   13 minutes ago   Up 13 minutes             silly_lichterman
[root@iZbp13qr3mm4ucsjumrlgqZ ~]# docker exec -it 88d23bcbe1f2 /bin/bash
[root@88d23bcbe1f2 /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:23 ?        00:00:00 /bin/sh -c while true;do echo kuangshen;sleep 1;done
root       841     0  0 08:37 pts/0    00:00:00 /bin/bash
root       858     1  0 08:37 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
root       859   841  0 08:37 pts/0    00:00:00 ps -ef
# 方式二
docker attach 容器id
# 测试
[root@iZbp13qr3mm4ucsjumrlgqZ ~]# docker attach 88d23bcbe1f2
正在执行当前的代码...

# docker exec        # 进入容器后开启一个新的终端，可以再里面操作（常用）
# docker attach        # 进入容器正在执行的终端，不会启动新的进程。
~~~

从容器内拷文件到主机上

~~~shell
docker cp 容器id:容器内路径 目的主机的路径

# 进入到容器内部
[root@iZbp13qr3mm4ucsjumrlgqZ home]# docker attach 6eda31ad7987
[root@6eda31ad7987 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@6eda31ad7987 /]# cd /home/
[root@6eda31ad7987 home]# ls
# 在容器的/home路径下创建test.java文件
[root@6eda31ad7987 home]# touch test.java
[root@6eda31ad7987 home]# ls
test.java
[root@6eda31ad7987 home]# exit
exit
[root@iZbp13qr3mm4ucsjumrlgqZ home]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@iZbp13qr3mm4ucsjumrlgqZ home]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS                      PORTS     NAMES
6eda31ad7987   centos    "/bin/bash"   About a minute ago   Exited (0) 28 seconds ago             stoic_kepler
# 将文件拷贝出来到主机上（在主机上执行该命令）
[root@iZbp13qr3mm4ucsjumrlgqZ home]# docker cp 6eda31ad7987:/home/test.java /home
[root@iZbp13qr3mm4ucsjumrlgqZ home]# ls
test.java
# 拷贝是一个手动过程，未来我们使用 -v 卷的技术，可以实现，自动同步（容器内的/home路径和主机上的/home路径打通）
~~~



# 安装软件

## 安装nginx

**下载镜像**：pull

~~~shell
 docker pull ngin
~~~

**启动nginx**

~~~shell
 # -d 后台运行
 # --name="ngin"    给容器命名
 # -p 宿主机端口:容器内部端口
 [root@iZbp13qr3mm4ucsjumrlgqZ ~]# docker run -d --name nginx -p 3344:80 nginx
 6e02190a50bc8d79653ffa88f6b5c143d79c5ac3257d5d5ed6a01247980fb48a
 [root@iZbp13qr3mm4ucsjumrlgqZ ~]# docker ps
 CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                   NAMES
 6e02190a50bc   nginx     "/docker-entrypoint.…"   18 seconds ago   Up 17 seconds   0.0.0.0:3344->80/tcp, :::3344->80/tcp   nginx-1
 # 进入容器
 [root@iZbp13qr3mm4ucsjumrlgqZ ~]# docker exec -it a1e130aa184d /bin/bash
 root@a1e130aa184d:/# whereis nginx
 nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
 root@a1e130aa184d:/# cd /etc/nginx/
 root@a1e130aa184d:/etc/nginx# ls
 conf.d  fastcgi_params  mime.types  modules  nginx.conf  scgi_params  uwsgi_params
~~~

**本机测试**

~~~shell
[root@Wang ~]# curl localhost:3344
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~































































