## 下载安装
首先在[官网](https://redis.io/download/)下载压缩包
解压后放到任意文件夹下，到Redis目录里编译并安装
```shell
make && make install
```
这一步完成后，将会安装三个命令：`redis-server`,`redis-benchmark`,`redis-cli`。

## 配置
完成后调整Redis配置文件`redis.conf`，可以先拷贝一份。
```shell
cp redis.conf redis.conf.bck
vi redis.conf
```
修改`redis.conf`中的一些配置
```properties
# 配置允许任意ip地址访问
bind 0.0.0.0
# 守护进程，在后台运行
daemonize yes
# 密码
requirepass 123456
# 监听端口
port 6379
# 工作目录
dir .
# 数据库数量，默认16个
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 日志文件，默认为空不记录
logfile "redis.log"
```
启动Redis：
```shell
# 进入Redis安装目录
cd ~/redis-stable
redis-server redis.conf
# 查看Redis进程
ps -ef | grep redis
# 可以通过redis-cli或者kill停止Redis
```

## 开机自启
最后可以配置开机自启
```Shell
vi /etc/systemd/system/redis.service
```
内容如下：
```text
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/root/redis-stable/src/redis-server /root/redis-stable/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
重载系统服务
```shell
systemctl daemon-reload
```
操作Redis
```Shell
systemctl start redis
systemctl stop redis
systemctl restart redis
systemctl status redis

# 开机自启
systemctl enable redis
```