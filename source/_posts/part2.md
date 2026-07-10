---
title: 第二部分 虚拟机与 ssh
date: 2026-07-10 16:49:57
tags:
---
## 一、知识储备
### 1、虚拟机与 Docker
虚拟机（VM）和 `Docker`（容器化技术）是目前最主流的两种架构隔离技术。它们最核心的区别在于隔离的层次不同：虚拟机是在硬件层面进行虚拟化，而 `Docker` 是在操作系统层面进行隔离，其优缺点如下。

|          | **虚拟机**                                     | **`Docker`**                                       |
| -------- | ------------------------------------------- | -------------------------------------------------- |
| **核心原理** | 在物理硬件上虚拟出完整的硬件资源，并在其上运行独立的 Guest OS。        | 共享宿主机的操作系统内核，实现进程级别的隔离。                            |
| **启动速度** | 慢，需要像物理机一样经历系统引导和加载过程。                      | 快。                                                 |
| **资源占用** | 高。每个虚拟机都需要单独分配内存、磁盘空间和 CPU 核心。              | 低。镜像体积小，共享宿主机内存，按需分配。                              |
| **应用场景** | 彻底隔离不同的业务<br>需要运行不同操作系统内核的任务<br>传统的重型单体应用部署 | 微服务架构部署<br>CI / CD 持续集成与持续交付<br>开发、测试、生产环境的快速同步与打包 |

虽然 Docker 并不是传统意义上的虚拟机，但是 `Docker` 的高并发、快速部署、轻量化和节省服务器成本等特点使其在软件开发中应用十分普遍，在一线大厂等也非常流行使用 `Docker` 容器。

此外 [WSL(Windows SubSystem for Linux)](https://learn.microsoft.com/zh-cn/windows/wsl/) 让用户可以在 Windows 计算机上直接运行 Linux 环境，而无需单独的虚拟机或双重启动，提供了非常可用的 Linux 环境，也是非常不错的非传统虚拟机方案。

## 二、虚拟机实验
我们选用 `Docker` 运行 `Debain` 容器来进行实验。
### 1、搭建实验环境
实验系统环境在前文有提及，不再赘述。
#### （1）安装 `Docker` 环境
我们使用 `Docker` 提供的图形化应用 [Docker Desktop](https://www.docker.com/products/docker-desktop/)。
点击蓝色按钮下载安装即可。
![Pasted image 20260708151857](/images/Pasted%20image%2020260708151857.png)
由于笔者已经安装完成，安装过程于此不再赘述，安装完毕以后，我们就可以使用 `Docker` 了。
![Pasted image 20260708152033](/images/Pasted%20image%2020260708152033.png)
`Docker` 本身十分具有可玩性，例如我们可以自己创建一个实验报告文档的镜像，便于分发与部署。在此我们先完成基本的 Linux 实验。
#### （2）安装容器镜像
打开终端，查看 `Docker` 版本（验证是否安装成功），并且拉取和查看 `Debian` 的镜像。
``` sh
docker --version
docker pull debian:latest
docker images
```
![Pasted image 20260708152858](/images/Pasted%20image%2020260708152858.png)
可以看到已经拉取完毕。
我们下面开始运行这个镜像，并授予管理员权限。
``` bash
docker run -d -t --name hdu-sd-tools --privileged=true debian:latest
```
`-d -t` 意为后台运行，并且分配一个伪终端，不会占用前台终端，`--privileged=true` 则表示我们需要允许在容器内运行诸如 `apt` 等需要管理员权限的命令。
我们可以使用 `docker ps` 命令来查看容器的状态。
![Pasted image 20260708153845](/images/Pasted%20image%2020260708153845.png)
如果我们需要进入到该容器中执行命令，我们可以运行：
``` sh
docker exec -it hdu-sd-tools /bin/bash
```
我们可以看到我们已经进入到了 Debian 容器的 `bash` 环境中：
尝试更新软件包 `apt update`：
![Pasted image 20260708154335](/images/Pasted%20image%2020260708154335.png)
运行 `apt install fastfetch` 安装系统信息查看软件。我们可以看到可以正常展示相应的系统信息。
![Pasted image 20260708154454](/images/Pasted%20image%2020260708154454.png)
### 2、`Docker` 容器的网络
我们首先安装一些网络相关的软件。
``` sh
apt install net-tools iputils-ping
```
测试一下是否可以联通外部网络：
``` sh
ifconfig
ping 1.1.1.1
```
![Pasted image 20260708155040](/images/Pasted%20image%2020260708155040.png)
## 三、SSH 实验
### 1、配置 ssh 服务器
我们首先安装 ssh 服务器并启动之。
``` sh
apt install openssh-server
service ssh start
```
![Pasted image 20260708184525](/images/Pasted%20image%2020260708184525.png)
接着，我们设置 root 用户的密码。密码输入过程中不会显示。
``` sh
passwd root
```
由于是测试环境，且未暴露端口，笔者直接使用 `Test_123456` 作为密码，**在生产环境中切记需要使用强密码或者直接采用 ssh 密钥认证的形式登录**。
配置 ssh 密钥的方法与前文中配置 `git` 的 ssh 密钥形式相同，需要将本地系统的 ssh 公钥添加到远程系统的 `~/.ssh/authorized_keys` 中，使远端系统信任客户端。
**同样的，ssh 默认监听的 22 端口也有被扫描爆破的可能，在生产环境中也需要更换 ssh 端口来方式爆破。**
因为我们直接使用 root 登录，我们还需要修改 `/etc/ssh/sshd_config` 文件来允许登录。
首先安装 `vim` 编辑器。
``` sh
apt install vim
```
接着编辑文件。
``` sh
vim /etc/ssh/sshd_config
```
我们需要做如下修改。
![Pasted image 20260708190835](/images/Pasted%20image%2020260708190835.png)
修改完成以后，我们需要重启 ssh 服务。
``` sh
service ssh restart
```
至此，我们的服务端配置完成。
### 2、建立 ssh 连接
需要注意的是，Docker 容器的网络与宿主机的网络在默认的 Bridge 模式下做了严格的隔离措施，不经过端口映射的容器无法被宿主机直接访问。只有容器之间可以互相访问，我们再新建一个相同的容器。
安装软件：
``` sh
apt update
apt install net-tools openssh-client
```
我们不妨对两个容器的网络环境进行对比：
**这是运行了 `openssh-server` 的服务器**
![Pasted image 20260708185518](/images/Pasted%20image%2020260708185518.png)
**这是运行了 `openssh-client` 的客户端**
![Pasted image 20260708185552](/images/Pasted%20image%2020260708185552.png)
我们可以看到服务器的 IP 为 `172.17.0.2` 客户端的 IP 为 `172.17.0.3`。
下面我们尝试连接。
``` sh
ssh root@172.17.0.2
```
首次连接需要验证指纹。
![Pasted image 20260708190129](/images/Pasted%20image%2020260708190129.png)
输入密码，连接成功。
![Pasted image 20260708191117](/images/Pasted%20image%2020260708191117.png)
ssh的配置与测试完成。
### 3、文件传输
#### （1）使用 SCP 传输文件
创建一个目录和文档：
``` sh
mkdir secrets
echo "Will you be in a band with me, forever?" >> secrets/forever
```
我们使用 `-r` 参数传输目录。
``` sh
scp -r secrets root@172.17.0.3:/
```
![Pasted image 20260709172534](/images/Pasted%20image%2020260709172534.png)
#### （2）使用 rsync 同步文件
我们先在两台虚拟机中安装 `rsync`。
``` sh
apt install rsync
```
同样的，我们创建一个目录用于同步。
``` sh
mkdir archive
echo "Tomori" >> archive/tomori && echo "Rana" >> archive/rana
```
可以看到被成功同步。
![Pasted image 20260709173929](/images/Pasted%20image%2020260709173929.png)