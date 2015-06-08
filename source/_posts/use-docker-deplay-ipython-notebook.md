title: 使用Docker部署IPython
date: 2015-06-08 16:21:56
tags: [Docker,IPyhton,nginx,Python]
categories:  
- 系统运维
---

![docker](http://zhiyue.qiniudn.com/15-6-8/63734849.jpg)
<!--more-->
> 本文的部署环境是Ubuntu 14.04 

- Docker
>Docker 详细概念可以去search，简单来说就是把应用打包到一个容器里的轻量级系统虚拟化服务

- IPython Notebook
>IPython Notebook 既是一个交互计算平台，又是一个记录计算过程的「笔记本」。它由服务端和客户端两部分组成，其中服务端负责代码的解释与计算，而客户端负责与用户进行交互。 服务端可以运行在本机也可以运行在远程服务器，包含负责运算的 IPython kernel (与 QT Console 的 kernel 相同) 以及一个 HTTP/S 服务器 (Tornado)。 而客户端则是一个指向服务端地址的浏览器页面，负责接受用户的输入并负责渲染输出。

本文主要记录使用Docker 在服务器部署IPython Note 应用的过程。比传统的部署方案果然简单轻松不少。



#### 知识点

1. Docker 的基本概念
2. 部署IPython 容器
2. 使用Nginx 容器反向代理IPython 


### Docker 的基本概念
下文的操作主要涉及Docker的一下几个知识点：
- Docker 的镜像(image)、容器(container)、仓库(registerie)
- Docker 的安装
- Docker 的基础用法
- Docker的端口映射
- Docker 数据卷
- 链接容器

详细的信息可以自行搜索

### 部署IPython 

#### 下载IPython 的镜像

官方Docker镜像[地址](https://registry.hub.docker.com/repos/ipython/)，里面包含5个镜像:
- notebook
- ipython
- scipyserver
- scipystack
- nbvierer

简单说一下这几个镜像的区别 `ipython` 是以上几个镜像的共同的根镜像。`notebook` 提供了一个web的前端。`scipystack`在`ipython`的基础上安装了许多科学计算的包(cython,h5py,matplotlib,numpy,pandas,patsy,scikit-learn,scipy,seaborn,sympy,yt)，而`scipyserver`则在`scipystack`的基础上提供了web的前端。`nbviewer`则是[nbviewer.ipython.org](nbviewer.ipython.org)的实现
更多详细信息请到GitHub上的[docker-notebook](https://github.com/ipython/docker-notebook) 查看
选择`scipystack`镜像最省事，但是需要下载的东西也越多。下文选择的也是`scipystack`
```
$ sudo docker pull ipython/scipyserver
```
#### 运行IPython 容器
```
sudo docker run -d --name IPythonApp -p 8888:8888 -e "PASSWORD=your password" -e "USE_HTTP=1" -v /home/zhiyue/repos/ipython-notebook:/notebooks ipython/scipyserver
```
解析一下参数`-d` 是以后台的方式运行，`--name` 是容器的别名，`-p` 是端口映射，`-e` 是设置环境变量，这里的环境变量`PASSWORD`设置成你自己的密码就可以了，`USE_HTTP=1` 意思是使用http，`-v` 是设置数据卷，把宿主机的目录挂载到容器里，即使容器被删除，数据也可以保留下来
### Nginx 反向代理 IPython
有两种方式，一种是使用官方的Nginx，另一种是使用[jwilder / nginx-proxy](https://registry.hub.docker.com/u/jwilder/nginx-proxy/)，后一种方法更加简单和方便
#### 方式1：nginx
- 使用Nginx的官方镜像[nginx](https://registry.hub.docker.com/_/nginx/)

- Nginx 配置文件
`ipython-server.conf`

```

 map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
 }

server{
        listen 80;
        server_name note.everforget.com;
        location / {
                proxy_pass http://ipython:8888;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
	            proxy_set_header X-Real-IP $remote_addr;
	            proxy_set_header Host $host;
	            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

这里面要注意一点是，ipython要使用websock因此要配置websocket，之前在这个问题查了好久。
` proxy_pass http://ipython:8888;` `ipython`和后面容器连接时的别名有关。
- 运行Nginx容器
```
$ sudo docker run -d -p 80:80 --name nginx --link IPythonApp:ipython  -v `pwd`/config:/etc/nginx/conf.d  -v `pwd`/logs:/var/log/nginx nginx
```

#### 方式2：nginx-proxy
- [jwilder / nginx-proxy](https://registry.hub.docker.com/u/jwilder/nginx-proxy/)

- 运行nginx-proxy 容器
```
docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy
```

- 运行ipython 容器

```
sudo docker run -d -e "VIRTUAL_HOST=ipython.everforget.com" --name IPython -p 8888:8888 -e "PASSWORD=xxxxxx" -e "USE_HTTP=1" -v /home/zhiyue/repos/ipython-notebook:/notebooks ipython/scipyserver
```


最后放一张图：

![](http://zhiyue.qiniudn.com/15-6-8/23580903.jpg)

这样就可以随时通过web来使用python了。


### 参考

- [Docker 笔记 By 枯木](http://blog.opskumu.com/docker.html)

- [[Docker] 快速建立 IPython Notebook 環境](http://godleon.github.io/blog/2014/11/23/use-docker-to-rapidly-create-ipython-notebook-environments/)
- [在Docker下部署Nginx](http://blog.shiqichan.com/Deploying-Nginx-with-Docker/)
- [IPython Notebook: 交互计算新时代](http://mindonmind.github.io/2013/02/08/ipython-notebook-interactive-computing-new-era/)



---
更新日志：
- 2015-6-8 添加配图
- 2015-6-6 第一次撰写