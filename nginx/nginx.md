<center><h1>nginx</h1></center>

## 1、nginx的配置文件

> - 全局块
> - events
> - http块  
>   - http全局块
>   - server块
>   - localtion块





## 2、桃子私塾项目的运用

这个是一个vue的项目，使用docker + nginx就行前端的部署。



2.1、首先是通过npm run build的命令就行打包

此时当前目录会有一个dist的发布目录，里面就是打包的前端代码



2.2、然后编写dockerfile文件

在编写dockerfile之前，需要配置好的nginx.conf文件

vim nginx.conf

```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

    server {
      listen  8080;
      server_name 45.156.23.186;

      # static source
      location ~.*\.(html|js|css)$ {
         root /home/web;
      }

      location / {
         root /home/web;
      }

      location /api/ {

         proxy_pass http://45.156.23.186:8889;
         proxy_set_header X-Real_IP  $remote_addr;
      }
    }
     
}
```



vim dockerfile

```
FROM nginx
ENV TZ=Asia/Shanghai
COPY ./dist  /home/vue
COPY ./nginx.conf /etc/nginx
```

docker   build  .   -t  oddworldjeffchan/caraliu-cms-cli



推送镜像到dockerhub:

docker push oddworldjeffchan/caraliu-cms-cli



在docker跑的nginx中安装vim:

```
首先是进入到容器内部：
docker exec -it containerID  /bin/bash

然后执行以下命令：
apt-get update
apt-get install -y vim
```



修改nginx.conf：重新启动nginx

```
nginx -s reload   # 
```

