---
title: 第一部分 Web 运行环境、项目与代码库配置
date: 2026-07-10 16:49:55
tags:
---
## 一、知识储备
### 1、一些静态站点构建工具的对比
目前常用的静态站点构建工具主要有 `Hexo`，`Hugo`，`Astro`，`VuePress`，`MkDocs` 等，在构建简单的静态站点时，以下三种最为广泛运用：

|         | 语言                     | 特点                 | 生态  |
| ------- | ---------------------- | ------------------ | --- |
| `Hexo`  | `Node.js`              | 插件丰富               | 丰富  |
| `Hugo`  | `Go`                   | 构建速度极快             | 尚可  |
| `Astro` | `Node.js & TypeScript` | 核心 Web 指标高，SEO 权重高 | 一般  |

笔者选用了 `Hugo` 作为本次实验的 Web 框架。

值得一提的是，笔者虽然在这次实验的所有环境中使用了 `Hugo`，但是提交的静态页面是使用 `Hexo` 构建的（也就是说本文使用 `Hexo` 构建），各种构建工具没有绝对的优劣之分，重要的是广泛涉猎，各取所长。
### 2、静态站点和动态站点的对比
|            | 静态站点                            | 动态站点                                                                  |
| ---------- | ------------------------------- | --------------------------------------------------------------------- |
| **核心定义**   | 页面在构建时已生成好，用户访问时服务器只做文件分发。      | 页面在运行时根据用户请求，由后端实时组装生成                                                |
| **服务器端**   | 仅充当文件服务器                        | 需要运行应用服务器                                                             |
| **数据库**    | 无需依赖（纯前端文件存储）                   | 强依赖                                                                   |
| **数据实时性**  | 较低。内容更新通常需要重新编译和部署              | 极高。数据库数据一旦变动，刷新页面即可实时渲染                                               |
| **加载速度**   | 极快。直接读取静态文件，适配 CDN 加速           | 较慢。需要经历“接受请求、查库、逻辑处理、渲染”的过程 |
| **高并发承载力** | 极高。由于不经过数据库和运算，抗并发能力极强          | 较低。高并发时压力集中在数据库和服务器，需做复杂的缓存                                           |
| **安全性**    | 极高。没有后端执行环境，彻底数据库等的漏洞           | 中等。攻击面较广，需防范越权、注入、跨站脚本等安全风险                                           |
| **内容管理**   | 繁琐。通常需要修改源文件或通过 Git 提交，并对接自动化工具 | 方便。一般拥有成熟的后台管理系统                                                      |

## 二、开始实验
### 1、搭建实验环境
实验系统环境如下：
![Pasted image 20260706223316](/images/Pasted%20image%2020260706223316.png)
根据 `Hugo` 官方的安装文档，我们需要安装 `Git` 版本控制软件，`Go` 语言环境，`Dart Sass` CSS 预处理器。
#### （1）`Git` 的安装
`Git` 的安装十分便利
``` sh
brew install git
```
#### （2）`Go` 的安装
根据 `Go` 语言官网的指引：[Download and install](https://go.dev/doc/install)。
我们只需要下载安装包后运行，安装程序会配置完包括环境变量在内的一切。
![Pasted image 20260706195659](/images/Pasted%20image%2020260706195659.png)
我们尝试运行一下，查看是否安装完成。
![Pasted image 20260706195803](/images/Pasted%20image%2020260706195803.png)
安装完毕。
#### （3）`Dart Sass` 的安装
`Hugo` 的文档中罗列了诸多安装方式，我们采用包管理器安装。
``` sh
brew install sass/sass/sass
```
![Pasted image 20260706200026](/images/Pasted%20image%2020260706200026.png)
### 2、安装 `hugo`
对于 macOS，`hugo` 同样提供了非常可用的包管理器安装法，我们通过 `brew` 可以很方便的安装。
``` sh
brew install hugo
```
同样测试一下。
![Pasted image 20260706214718](/images/Pasted%20image%2020260706214718.png)
此外我们检查一下 `Dart Sass` 的安装
``` sh
hugo env
```
我们可以看到可以正确显示我们安装的 `Dart Sass`。
![Pasted image 20260706220413](/images/Pasted%20image%2020260706220413.png)
### 3、创建 `hugo project`
我们来使用官方文档提供的指引创建一个简单的 example。
``` sh
hugo new project quickstart
```
![Pasted image 20260706222958](/images/Pasted%20image%2020260706222958.png)
可以看到成功的新建了 project。
我们进入到文件夹中，进行 `Git` 仓库的初始化。
``` sh
cd quickstart
git init
```
配置主题，采用了官方 example 中的 `Ananke` 主题。
下文将主题仓库注册成了整个 project 仓库下的子模块，并且配置到了 `hugo.toml` 配置文件中。
``` shell
git submodule add https://github.com/gohugo-ananke/ananke themes/ananke
echo "theme = 'ananke'" >> hugo.toml
```
启动 `hugo server`。
``` sh
hugo server
```
![Pasted image 20260706223940](/images/Pasted%20image%2020260706223940.png)
可以看到，在本地的 1313 端口上，`Hugo` 运行了一个服务器，我们用浏览器访问它。
![Pasted image 20260706224117](/images/Pasted%20image%2020260706224117.png)
按下 `Ctrl + C` 停止运行。
### 4、创建 `hugo content`
我们按官方文档的指引创建第一篇文档。
``` sh
hugo new content content/posts/my-first-post.md
```
![Pasted image 20260706225524](/images/Pasted%20image%2020260706225524.png)
尝试着往里面添加一些内容，注意到 `hugo` 默认添加了 `draft = true` 后续在运行时需要增添 `--buildDrafts` 参数来解决其无法显示的问题。
![Pasted image 20260706230439](/images/Pasted%20image%2020260706230439.png)
我们可以看到文章被显示了出来。
![Pasted image 20260706230538](/images/Pasted%20image%2020260706230538.png)
点入可以看到渲染正确。
![Pasted image 20260706230612](/images/Pasted%20image%2020260706230612.png)
## 三、代码库的配置
`Git` 是一个非常好用的版本控制软件，我们可以以这样的方式来管理我们的 `Hugo` 目录。
在前文中，我们已经配置好了仓库的初始化，并将 `Ananke` 主题配置成了仓库的 `SubModule`。我们现在要做的是将代码库保存到云端，以便多人协作与版本控制。
我们选用 `GitHub` 作为我们云代码库的提供方来进行相应的配置。
我们首先创建一个仓库。
![Pasted image 20260708003745](/images/Pasted%20image%2020260708003745.png)
我们选择采用 `ssh` 的方式来连接远端仓库。
注意，在配置远端仓库时，我们需要创建本地的 `ssh` 密钥，并添加到 `GitHub` 的受信任 `ssh` 密钥之中。笔者的试验环境已配置完成，故不在此处展示配置过程，`GitHub` 对添加密钥也有详细的文档解释 [Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)，故笔者于此不多赘述。
我们进入到 `Hugo` 目录中，首先向该仓库的配置中添加远端仓库。
``` sh
git remote add origin git@github.com:arkerny/HDU-SD-Tools.git
```
接着我们查看 `Hugo` 目录下文件被 `Git` 追踪的情况。
``` sh
git status
```
![Pasted image 20260708005213](/images/Pasted%20image%2020260708005213.png)
可以看到我们目前没有任何的提交，只有我们前面添加的 `Submodule`，我们先创建一个 `Readme` 文件，并且尝试将我们的文件添加到暂存区。
![Pasted image 20260708005911](/images/Pasted%20image%2020260708005911.png)
我们来进行第一次 `commit`。
![Pasted image 20260708010030](/images/Pasted%20image%2020260708010030.png)
我们把更改 `push` 到远端仓库。
![Pasted image 20260708010155](/images/Pasted%20image%2020260708010155.png)
可以看到我们的 `README` 文件已经被上传到 `GitHub` 上了。
![Pasted image 20260708115409](/images/Pasted%20image%2020260708115409.png)
我们使用 `VSCode` 打开我们的 `hugo` 目录。
安装 `MarkDown` 插件后可以更好的支持 `Markdown` 格式文件。
对 `README.md` 文件再次进行编辑，可以正常显示文件的情况，我们也可以发现我们之前的提交已经被正常显示在了时间线上，同时 `Submodule` 也被成功识别。
![Pasted image 20260708120330](/images/Pasted%20image%2020260708120330.png)
我们可以直接把所有文件进行第二次提交。
![Pasted image 20260708120737](/images/Pasted%20image%2020260708120737.png)
提交完成后再点击发布，就能直接同步到 `GitHub` 上了。