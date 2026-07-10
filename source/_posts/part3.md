---
title: 第三部分 Linux 基础使用和配置
date: 2026-07-10 16:49:59
tags:
---
## 一、Linux 的用户和权限管理
Linux 自诞生之初就是为多用户服务的。root 作为最高权限用户有着“至高无上的权力”。然而“权利越大，责任越大”。root 权限不应该由所有用户都轻易取得，一些底层的配置文件也不能由一般用户修改，因此，做好用户和权限管理是至关重要的。而 Linux 就有着极其优秀的用户和权限管理原则。
### 1、用户管理与添加
为了测试方便，我们添加一个用户，并为其分配密码：
``` sh
adduser tomori
```
![Pasted image 20260708194645](/images/Pasted%20image%2020260708194645.png)
我们新建了一个名为 `tomori` 密码为 `Tomori_233` 的用户。
另外再新建一个名为 `rana` 密码为 `Rana_233` 的用户备用。
分别查看两者的所属组。
![Pasted image 20260708195112](/images/Pasted%20image%2020260708195112.png)
我们尝试在刚刚的 `clinet` 机中使用用户 `tomori` 连接 ssh。
![Pasted image 20260709000743](/images/Pasted%20image%2020260709000743.png)
同样的，我们新建一个终端，ssh 连接用户 `rana`。
### 2、用户与用户组权限实验
现在，我们有三个终端分别连接到了三个用户，我们尝试使用 `tomori` 用户在 `/home/tomori/` 下创建一个文件。并用 `rana` 和 `root` 分别访问之，发现 `rana` 没有权限，`root` 则有权限访问。
运行 `ls -l` 命令可以看到文件的信息：
```
root@0de9293d3c6b:/# ls -l /home/tomori
total 4
-rw-rw-r-- 1 tomori tomori 9 Jul  8 16:32 tomori.md
```
发现权限是 `664`。`rana` 应当可以读取文件，但是事实是无法访问，我们不妨再看看 `/home/tomori` 目录的权限：
```
root@0de9293d3c6b:/# ls -l /home/
total 8
drwx------ 2 rana   rana   4096 Jul  8 11:48 rana
drwx------ 2 tomori tomori 4096 Jul  8 16:32 tomori
```
发现权限是 `700`。
`rana` 不是 `/home/tomori` 的所有者，故无权限。
![Pasted image 20260709004914](/images/Pasted%20image%2020260709004914.png)
下面我们尝试把两者加入到新建的组 `mygo` 中：
``` sh
groupadd mygo
usermod -g mygo rana
usermod -g mygo tomori
```
我们不妨看看 `/etc/passwd` 里面是否有修改的信息：
``` sh
cat /etc/passwd
```
发现两人都被添加进了组 `mygo` 中。
![Pasted image 20260709005837](/images/Pasted%20image%2020260709005837.png)
需要注意的是：此时 `tomori` 和 `rana` 都属于多个用户组中。**如果一个用户同时属于多个用户组，那么用户可以在用户组之间切换，以便具有其他用户组的权限**。用户组的切换使用 `newgrp` 命令。
下面我们实验，使用 `root` 切换到 `mygo` 用户组，并且创建文件供 `mygo` 组内用户读写。
我们先使用 `root` 用户运行如下命令：
``` sh
newgrp mygo
mkdir /tmp/mygo_secrets
chmod 770 /tmp/mygo_secrets
```
在以上命令中，我们创建了一个属于用户 `root`，所在组为 `mygo` 的文件夹。
我们再创建一个文件：
``` sh
echo "BanG Dream! It's MyGO!!!!!" >> /tmp/mygo_secrets/qaq.md
chmod 770 /tmp/mygo_secrets/qaq.md
```
我们可以通过 `ls -l` 来查看权限。
![Pasted image 20260709012119](/images/Pasted%20image%2020260709012119.png)
现在我们尝试用其他两个用户来更改与读取该文件。
![Pasted image 20260709012323](/images/Pasted%20image%2020260709012323.png)
发现可以读取与修改。若此时我们再创建一个尝试访问的第三者，则会发现无权限。
## 二、环境变量与配置文件
环境变量是 Linux 系统中用于指定操作系统运行环境的参数，它们对系统的行为和程序的运行有着重要影响。
特别的，对于 Docker 容器，我们还能通过宿主机为容器添加环境变量，可以在运行时、构建镜像时或工具实现，**尤其方便配置应用和传递敏感信息**。
当我们需要固化用户环境变量时，我们可以将环境变量写入 `.bashrc`、`.bash_profile`、`.profile` 这些文件之中，一般我们使用 `.bashrc`（针对 `bash`），`.zshrc` （针对 `zsh`）等。
下面我们进行一些简单的实验。
### 1、查看环境变量
``` sh
env
```
![Pasted image 20260709160704](/images/Pasted%20image%2020260709160704.png)
### 2、进行简单的环境变量配置实验
我们先下载一个 `gcc`。
``` sh
apt install gcc
```
然后我们创建一个示例程序。
``` sh
mkdir mygo mygo/bin
touch mygo/mygo.c
vim mygo/mygo.c
```
输入代码：
``` c
#include <stdio.h>
int main() {
	printf("███╗   ███╗██╗   ██╗ ██████╗  ██████╗ ██╗██╗██╗██╗██╗\n");
    printf("████╗ ████║╚██╗ ██╔╝██╔════╝ ██╔═══██╗██║██║██║██║██║\n");
    printf("██╔████╔██║ ╚████╔╝ ██║  ███╗██║   ██║██║██║██║██║██║\n");
    printf("██║╚██╔╝██║  ╚██╔╝  ██║   ██║██║   ██║╚═╝╚═╝╚═╝╚═╝╚═╝\n");
    printf("██║ ╚═╝ ██║   ██║   ╚██████╔╝╚██████╔╝██╗██╗██╗██╗██╗\n");
    printf("╚═╝     ╚═╝   ╚═╝    ╚═════╝  ╚═════╝ ╚═╝╚═╝╚═╝╚═╝╚═╝\n");
	return 0;
}
```
![Pasted image 20260709163606](/images/Pasted%20image%2020260709163606.png)
接着我们编译运行，发现可以成功运行。
![Pasted image 20260709163734](/images/Pasted%20image%2020260709163734.png)
接下来我们尝试添加环境变量。
``` sh
cp mygo/mygo mygo/bin/mygo
export PATH=$PATH:/mygo/bin
mygo
```
![Pasted image 20260709163909](/images/Pasted%20image%2020260709163909.png)
可以看到我们能直接运行 `mygo` 命令，而不需要输入可执行文件的路径。
接下来我们尝试把环境变量写入 `.bashrc` 文件。
``` sh
echo "export PATH=$PATH:/mygo/bin" >> ~/.bashrc
```
可以看到原本没有的环境变量在命令写入 `.bashrc` 后可以运行。
![Pasted image 20260709165033](/images/Pasted%20image%2020260709165033.png)
环境变量和用户配置的实验完成。