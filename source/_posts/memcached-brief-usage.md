---
title: Memcache简易说明
date: 2016-01-14 23:17:01
tags: memcache
---

![memcached](http://memcached.org/images/memcached_banner75.jpg)

<!-- more -->

### 1.安装memcached

```
yum install memcached
memcached -h
```

### 2.启动memcached

```
memcached -d -m 100 -c 1000 -u root -p 33060
```

#### 参数说明：
```
-d  启动一个守护进程

-m  分配给Memcache使用的内存数量，单位是MB，默认为64M

-u  运行Memcache的用户(only when run as root)

-l  监听的服务器IP地址

-p  Memcache监听的端口,默认为11211，最好是1024以上的端口

-c  最大运行的并发连接数，默认是1024，按照服务器的负载量来设定

-P  设置保存Memcache的pid文件
```

### 3.查看memcached状态

#### (1) telnet ip port ==> stats

```
# telnet 127.0.0.1 33060
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
stats
STAT pid 7483
STAT uptime 62616
STAT time 1484028645
STAT version 1.4.4
STAT pointer_size 64
STAT rusage_user 0.913861
STAT rusage_system 0.691894
STAT curr_connections 12
STAT total_connections 16
STAT connection_structures 13
STAT cmd_get 15
STAT cmd_set 5
STAT cmd_flush 0
STAT get_hits 9
STAT get_misses 6
STAT delete_misses 0
STAT delete_hits 0
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 5657
STAT bytes_written 21447
STAT limit_maxbytes 104857600
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT threads 4
STAT conn_yields 0
STAT bytes 5225
STAT curr_items 3
STAT total_items 5
STAT evictions 0
END
```

#### (2) 模拟top命令

```
watch "printf 'stats\r\n' | nc 127.0.0.1 33060"
```

```
watch "echo stats | nc 127.0.0.1 33060"
```

### 4.Python-memcached API


```
import memcache

mc = memcache.Client(['127.0.0.1:33060'], debug=0)

if __name__ == '__main__':
    mc.set('name', 'kallen')
    mc.set('mail', 'kallen@mail.com')
    name = mc.get('name')
    mail = mc.get('mail')
    print name, mail
```

#### 主要方法如下：

- `@set(key, val, time=0, min_compress_len=0)`

> 无条件键值对的设置，其中的time用于设置超时，单位是秒
>
> min_compress_len则用于设置zlib压缩

- `@set_multi(mapping, time=0, key_prefix='', min_compress_len=0)`

> 设置多个键值对，key_prefix是key的前缀，完整的键名是key_prefix+key, 
>
>使用方法如下：

```
mc.set_multi({'k1' : 1, 'k2' : 2}, key_prefix='pfx_') 
mc.get_multi(['k1', 'k2', 'nonexist'], key_prefix='pfx_')
{'k1' : 1, 'k2' : 2}
```

- `@add(key, val, time=0, min_compress_len=0)`

> 添加一个键值对，内部调用_set()方法

- `@replace(key, val, time=0, min_compress_len=0)`

> 替换value，内部调用_set()方法

- `@get(key)`

> 根据key去获取value，出错返回None

- `@get_multi(keys, key_prefix='')`

> 获取多个key的值，返回的是字典, keys为key的列表

- `@delete(key, time=0)`

> 删除某个key, time的单位为秒;
>
> 用于确保在特定时间内的set和update操作会失败。
>
> 如果返回非0则代表成功

- `@incr(key, delta=1)`

> 自增变量加上delta，默认加1，使用如下

```
mc.set("counter", "20")  
mc.incr("counter")
21
```

- `@decr(key,delta=1)`

> 自减变量减去delta，默认减1


```
# memcached 初始化
mc = memcache.Client(['服务器A IP:11211'], debug=0)

class test:
    def GET(self):
        sql = "select * from table"
        m = hashlib.md5(sql)
        key = m.hexdigest()

        retval = []
        if mc.get(key):
            retval = mc.get(key)
        else:
            retval = db.query(sql).list()
            mc.set(key, retval )
         
        # 断开连接
        mc.disconnect_all()
```



### [REFERENCE]

- [Memcache Examples](https://cloud.google.com/appengine/docs/python/memcache/examples)
- [Linux下安装memcached](https://my.oschina.net/flynewton/blog/9694)
- [Memcached的Client方法介绍](https://lxneng.com/posts/39)
- [Python-memcached的基本使用](https://my.oschina.net/flynewton/blog/10660)
- [python memcached 邮件列表](https://groups.google.com/forum/#!topic/python-cn/A1BG4luYj94)

