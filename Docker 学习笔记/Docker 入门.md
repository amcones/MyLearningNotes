一直以来都对docker有一个基础的概念，知道是一个包含有程序及其运行环境的sandbox，但对于原理，具体怎么用还不是很了解。这次找到个课程来学习一下。

## Getting Start

### Get the app

1. 首先clone官方的库：
```Shell
git clone https://github.com/docker/getting-started.git
```
接下来就要建立app的容器镜像。
```Shell
cd /path/to/app
```
建立一个空的文件叫做 `Dockerfile` 。
```Shell
touch Dockerfile
```

### Build the app's container image

在 `Dockerfile` 里写入：
```Shell
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

建立容器镜像
```Shell
docker build -t getting-started .
```
在我的mbp上需要先启动docker desktop才能正常运行，目前还不清楚关系

`docker build` 命令使用Dockerfile来构建一个容器镜像。在这个例子里，镜像是在 `node:12-alpine` 镜像的基础上构建的，所以需要先下载这个镜像。

在下载完成之后，docker会根据Dockerfile的引导使用 `yarn` 下载应用依赖。 `CMD` 命令直接指定了在启动镜像的容器时使用的默认命令。

最后， `-t` 标志标记了你的镜像。

`docker build` 末尾的 `.` 告诉Docker它应当在当前目录下找 `Dockerfile` 。

### Start an app container

现在有了一个镜像，你可以在一个容器里运行这个应用。你可以使用 `docker run` 命令并且指定你刚建立的镜像名：
```Shell
docker run -dp 3000:3000 getting-started
```
使用 `-d`标志在后台运行一个新的容器，并且使用 `-p`标志来创建一个主机3000端口和容器3000端口的映射。运行后可以在 `http://localhost:3000`访问刚刚运行的应用。在docker dashboard上也可以看到刚刚运行的容器。

