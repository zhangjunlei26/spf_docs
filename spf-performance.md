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
磁盘：Seagate Constellation ES ST3500514NS 500G ATA
网卡： Intel Corporation 82576 Gigabit Network Connection x2
操作系统：Linux TENCENT64.site 2.6.32.43-tlinux-1.0.10-default

### 压测工具

```shell
 webbench -c 50 -t 100 -k http://127.0.0.1:8080/
```
### 软件信息

#### nginx+php

版本 nginx/1.8.1、php7.0.8+opcache

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

测试页面index.php
```php
<?php
    echo "Hello World!";
```
进程数量
Nginx开启了4个Worker进程
php-fpm 最小50进程，最大2000进程

#### swoole+php7.0.8
测试代码
```php
<?php
$http = new swoole_http_server("0.0.0.0", 8080, SWOOLE_BASE);
$http->set([
    'worker_num'  => 8,
]);

$http->on('request', function ($request, swoole_http_response $response) {
    $response->header('Last-Modified', 'Tue, 26 Jul 2016 10:24:27 GMT');
    $response->header('E-Tag', '55829c5b-17');
    $response->header('Accept-Ranges', 'bytes');    
    $response->end("<h1>\nHello World.\n</h1>");
});

$http->start();
[root@TENCENT64 /data/home/rosenzhang/ros
```

