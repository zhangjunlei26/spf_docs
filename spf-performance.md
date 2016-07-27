# spf性能
## 性能对比
使用webbench工具对nginx/php-fpm、swoole 原生http、spf http、spf-http-mvc-demo进行压力测试。


## 测试环境

### 硬件配置

* CPU: Intel(R) Xeon(R) CPU X3440  @ 2.53GHz x8
* 内存：8G
* 磁盘：Seagate Constellation ES ST3500514NS 500G ATA
* 网卡： Intel Corporation 82576 Gigabit Network Connection x2
* 操作系统：Tencent tlinux release 1.2 (Final) 2.6.32.43-tlinux-1.0.10-default (kbuild@tlinux12) (gcc version 4.4.6 20110731 (Red Hat 4.4.6-3) (GCC) )


### 压测工具

```shell
 webbench -c 50 -t 100 -k http://127.0.0.1:8080/
```
### 软件信息

#### nginx+php

版本 nginx/1.8.1、php7.0.8+opcache

VHOST配置
```javascript
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

#### swoole原生
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
```
#### spf http
代码如下：
```php
<?php
namespace demo\spf;
use spf\Swoole\Worker\Base;
use spf\Swoole\Worker\IWorker;
class DemoWorker extends Base implements IWorker 
{
   public function onRequest(\swoole_http_request $request = null, \swoole_http_response $response = null) {
      $response->header('Last-Modified', 'Tue, 26 Jul 2016 10:24:27 GMT');
      $response->header('E-Tag', '55829c5b-17');
      $response->header('Accept-Ranges', 'bytes');    
      $response->end("<h1>\nHello World.\n</h1>");    
   }
}
```
复制spf/conf/demo.php为foo.php，修改其中的`worker_class`为上面创建的类名：`\demo\spf\DemoWorker`，启用服务使用spf命令`spf start foo`。

#### spf mvc demo
使用spf源码中提供的简化版mvc demo代码进行压测，代码省略。启用服务使用spf命令`spf start demo`。
```php
<?php
namespace demo\controller;
use syb\oss\Controller;
class index extends Controller {
    function actionIndex() {
        $response = $this->response;
        $response->header('Last-Modified', 'Tue, 26 Jul 2016 10:24:27 GMT');
        $response->header('E-Tag', '55829c5b-17');
        $response->header('Accept-Ranges', 'bytes');
        $response->end("<h1>\nHello World.\n</h1>");
    }
}
```
## 测试结果
###nginx+php-fpm


```
webbench -c 100 -t 100 http://test.bao.qq.com/spf/test.php

Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Benchmarking: GET http://test.bao.qq.com/spf/test.php
100 clients, running 100 sec.

Speed=1226627 pages/min, 3414113 bytes/sec.
Requests: 2044379 susceed, 0 failed.
Requests per second:    20443.79 [#/sec] (mean)
```
![php-fpm](1-php-fpm.png)

###swoole原生
```
webbench -c 100 -t 100 http://127.0.0.1:8080/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Benchmarking: GET http://127.0.0.1:8080/
100 clients, running 100 sec.

Speed=3737880 pages/min, 16135182 bytes/sec.
Requests: 6229801 susceed, 0 failed.
Requests per second:    62298.01 [#/sec] (mean)
```
![swoole](2-swoole.png)
###spf http

```
webbench -c 100 -t 100 http://127.0.0.1:8080/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Benchmarking: GET http://127.0.0.1:8080/
100 clients, running 100 sec.

Speed=3691755 pages/min, 15936083 bytes/sec.
Requests: 6152925 susceed, 0 failed.
Requests per second:    61529.25 [#/sec] (mean)
```
![](spf-ori.png)



###spf mvc demo
```
webbench -c 100 -t 100 http://127.0.0.1:8080/demo/index/index
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Benchmarking: GET http://127.0.0.1:8080/demo/index/index
100 clients, running 100 sec.

Speed=2898611 pages/min, 12512344 bytes/sec.
Requests: 4831019 susceed, 0 failed.
Requests per second:    48310.19 [#/sec] (mean)
```
![spf-demo](spf-demo.png)

## 总结
| 压力测试用例 | 性能QPS |
| -- | -- |
| nginx/php-fpm  | 20443.79 |
| swoole 原生    | 62298.01 |
| spf http       | 61529.25 |
| spf mvc demo   | 48310.19 |

1. spf与原生swoole性能相近。
2. 在hello world场景中，spf与swoole的性能是php-fpm的3倍。
3. swoole原生测试未能达到swoole官方测试中的287104.12r/s。考虑了一下原因，可能跟linux内核版本低于3.9.0，不能开启REUSEPORT选项有关。因没有新版本linux环境可测试，只能推测原因。
4. 在增加spf mvc框架的压力测试场景中，因php执行的代码量增大，cpu使用率上升10%左右，性能下降到原生swoole的77.55%。php-fpm中如果使用框架也存在同样的性能下降问题，已得到业内测试证实，这里不再浪费时间去对比与spf的差异。在业务开发中，尽量不要使用太重量的框架，以免对性能影响太明显。

