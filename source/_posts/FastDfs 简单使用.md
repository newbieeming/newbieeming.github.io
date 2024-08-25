---
title: FastDfs 简单使用
date: 2022-10-12 20:32	
categories:
  - Linux
  - FastDfs
---

### FastDfs安装

### 准备
```sh
libfastcommon-1.0.48.tar.gz
fastdfs-6.07.tar.gz
libfastcommon-1.0.48.tar.gz
fastdfs-6.07.tar.gz
```

### 安装gcc环境
```sh
yum install -y gcc gcc-c++
```
### 开始
```sh
libfastcommon-1.0.48.tar.gz
fastdfs-6.07.tar.gz
# 以上解压 ./make.sh && ./make insatll
```

安装好后会在**/etc/fdfs**有配置文件

```sh
cd /etc/fdfs
cp tracker.conf.sample tracker.conf
cp storage.conf.sample storage.conf
cp client.conf.sample client.conf #客户端文件测试使用
cp /usr/local/src/java/fastdfs-6.07/conf/http.conf #供nginx访问使用 
cp /usr/local/src/java/fastdfs-6.07/conf/mime.types ./ #供nginx访问使用
```

```sh
#解压以下文件
fastdfs-nginx-module-1.22.tar.gz
nginx-1.15.4.tar.gz
```

### 配置Nginx模块

```shell
cd nginx-1.15.4
./configure --add-module=/usr/local/src/java/fastdfs-nginx-module-1.22/src #添加模块
make && make insatll #安装ninx
```

### 配置tracker

```shell
#创建存储目录
mkdir -m 777 /home/fds
vim /etc/fdfs/tracker.conf
#修改以下内容
base_path = /home/dfs # 数据和日志文件存储根目录
```

### 配置storage

```sh
vim /etc/fdfs/storage.conf
#修改以下内容
base_path=/home/dfs  # 数据和日志文件存储根目录
store_path0=/home/dfs  # 第一个存储目录
tracker_server=本机ip:22122  # 本机ip改成自己服务器的ip
http.server_port=8888  # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)
```

### 启动tracker和storage

```sh
fdfs_trackerd /etc/fdfs/tracker.conf
fdfs_storaged /etc/fdfs/storage.conf
netstat -unltp |grep fdfs
```

### 使用client测试

```sh
vim /etc/fdfs/client.conf
base_path=/home/dfs
tracker_server=本机ip:22122    #本机ip修改为服务器ip
fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/nginx-1.15.4.tar.gz
保存后测试,返回ID表示成功 如：group1/M00/00/00/wKgogmNGNpGAdE87AA-itrfn0m4.tar.gz
```

### 配置nginx访问

```sh
vim /etc/fdfs/mod_fastfds.conf
tracker_server=本机ip:22122  #本机ip修改为服务器ip
url_have_group_name=true
store_path0=/home/dfs
#修改nginx配置
vim /usr/local/src/java/nginx-1.15.4/conf/nginx.conf
server {
    listen       8888;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
#启动nginx
/usr/local/nginx/sbin/nginx
```

### 注意

```sh
如果上传成功 但是nginx报错404 先检查mod_fastdfs.conf文件中的store_path0是否一致
如果nginx无法访问 先检查防火墙 和 mod_fastdfs.conf文件tracker_server是否一致
如果不是在/usr/local/src文件夹下安装 可能会编译出错

上述ip地址可以写内网或者外网地址，切勿写127.0.0.1或者localhost
```

```sh
fdfs_trackerd /etc/fdfs/tracker.conf
fdfs_storaged /etc/fdfs/storage.conf
/usr/local/nginx/sbin/nginx
```