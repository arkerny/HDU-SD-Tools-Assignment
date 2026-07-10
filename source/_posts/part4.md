---
title: 第四部分 Web 站点部署
date: 2026-07-10 16:50:02
tags:
---
## 一、静态站点生成器
让我们回到前面的 `Hugo`。
当我们进入到站点目录下，直接运行命令 `hugo` 就可以构建静态站点。
![Pasted image 20260709174637](/images/Pasted%20image%2020260709174637.png)
构建的静态站点文件发布在 `public` 目录下，我们可以直接通过浏览器打开 `index.html`。
![Pasted image 20260709175932](/images/Pasted%20image%2020260709175932.png)
然而发现我们的站点样式等都不可见，这是我们没有运行服务器导致的。
我们下面将配置部署 Web 站点并使之支持访问。
## 二、Web 服务器配置
### 1、环境配置
我们新建一个 Docker 容器，并且配置端口映射使得外部可以直接访问。
``` sh
docker run -d -t \
  --name hdu-sd-tools-web \
  -p 30022:22 \
  -p 30080:80 \
  --privileged=true \
  debian:latest
```
我们把 ssh 使用的 `22` 端口映射到了宿主机的 `30022` 端口，把 http 使用的 `80` 端口映射到了宿主机的 `30080` 端口。
我们进入到容器内部。
``` sh
docker exec -it hdu-sd-tools-web /bin/bash
```
首先安装一些必要的软件。
``` sh
apt update
apt install openssh-server rsync nginx net-tools vim
```
配置好 ssh 服务，确保可以访问。
尝试一下是否能够启动 `nginx`。
```sh
service nginx start
```
### 2、Web 服务器配置
首先我们测试一下可用性：
``` sh
service start nginx
```
浏览器打开 `http://127.0.0.1:30080/`。
![Pasted image 20260710122516](/images/Pasted%20image%2020260710122516.png)
发现可以访问。
我们创建站点目录。
``` sh
mkdir /var/www/mygo
```
然后再更改站点配置文件。
``` sh
vim /etc/nginx/sites-available/mygo
```
写入以下信息：
``` nginx
server {
	listen 80;
	server_name _;
	location /{
		root /var/www/mygo;
	}
	index index.html;
}
```
同时我们需要删除 `Nginx` 自带的示例，添加新的软链接
``` sh
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/mygo /etc/nginx/sites-enabled/
```
重启 `Nginx` 服务。
``` sh
service nginx restart
```
![Pasted image 20260710123130](/images/Pasted%20image%2020260710123130.png)
接下来我们通过 rsync 同步 `hugo` 目录下的 `public` 目录。
``` sh
rsync -avz -e "ssh -p 30022" public/ root@127.0.0.1:/var/www/mygo/
```
![Pasted image 20260710123638](/images/Pasted%20image%2020260710123638.png)
可以看到我们的站点部署成功：
![Pasted image 20260710124836](/images/Pasted%20image%2020260710124836.png)
## 三、使用 CI/CD 自动构建并部署
对于 ServerLess 的部署方案，我们可以使用 CI/CD 自动化流程构建并部署，下面以 CloudFlare Pages 为例自动拉取镜像并且部署。网址：[CloudFlare](cloudflare.com)。
我们先在本地配置好设置文件：
``` sh
vim wrangler.toml
```
写入以下内容，主要是 `Hugo` 的版本。
``` toml
name = "hdu-sd-tools"
pages_build_output_dir = "public"
[vars]
HUGO_VERSION = "0.164.0"
```
push 到远端仓库。
进入到 Workers 和 Pages 页面。
![Pasted image 20260710153852](/images/Pasted%20image%2020260710153852.png)
创建应用程序，选择使用 Pages。
![Pasted image 20260710153930](/images/Pasted%20image%2020260710153930.png)
导入现有存储库：
![Pasted image 20260710154005](/images/Pasted%20image%2020260710154005.png)
![Pasted image 20260710154023](/images/Pasted%20image%2020260710154023.png)
框架预设选择 Hugo。
![Pasted image 20260710154054](/images/Pasted%20image%2020260710154054.png)
确定即可。
![Pasted image 20260710160913](/images/Pasted%20image%2020260710160913.png)
访问链接即可。
![Pasted image 20260710160940](/images/Pasted%20image%2020260710160940.png)
Web站点部署完成。