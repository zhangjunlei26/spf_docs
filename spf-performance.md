# spf性能
## 性能对比
使用webbench工具对nginx/php-fpm、swoole http、spf http、spf-http-mvc-demo进行压力测试。

| 软件 | QPS |
| -- | -- |
| nginx/php-fpm  | 1:2 |
| Swoole Http    | 1:2 |
| Spf Http       | 1:3 |
| Spf MVC demo   | 1:3 |

## 测试环境

### 硬件配置

CPU: Intel(R) Xeon(R) CPU X3440  @ 2.53GHz x8
内存：8G
磁盘：500G SSD
操作系统：Ubuntu14.04 Linux TENCENT64.site 2.6.32.43-tlinux-1.0.10-default
网卡： Intel Corporation 82576 Gigabit Network Connection x2

### 压测工具

```shell
 webbench -c 50 -t 100 -k http://127.0.0.1:8080/demo/index/index
```
### 软件信息

#### Nginx+php

版本 nginx/1.4.6 (Ubuntu)

VHOST配置
```
server {
    listen 80 default_server;
    root /data/webroot;
    index index.html index.htm index.php;
    location / {
    	try_files $uri $uri/ /index.php?uri=$uri;
    }
    location ~ ^.*\.php {
        root /usr/local/baoweb;
        fastcgi_pass   127.0.0.1:9100;
        include "fastcgi.conf";
        fastcgi_index  index.php;
    }
}
```

测试页面
<h1>Hello World!</h1>
进程数量
Nginx开启了4个Worker进程

