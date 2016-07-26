# spf性能
## 性能对比
使用apache bench工具对nginx/php-fpm、swoole http、spf http、spf-http-mvc-demo进行压力测试。

| 软件 | QPS |
| -- | -- |
| nginx/php-fpm  | 1:2 |
| Swoole Http    | 1:2 |
| Spf Http       | 1:3 |
| Spf MVC demo   | 1:3 |

## 测试环境
### 硬件配置
CPU：Intel® Core™ i5-4590 CPU @ 3.30GHz × 4
内存：16G
磁盘：128G SSD
操作系统：Ubuntu14.04 (Linux 3.16.0-55-generic)

### 压测工具

```shell
 ab -c 100 -n 1000000 -k http://127.0.0.1:8080/
```