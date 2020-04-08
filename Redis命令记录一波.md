记录一下Redis常用的命令

> redis-cli -h 127.0.0.1 -a "password"

**列举所有key**

> keys *

列举出来以后，可以通过`get key-name`来获取key详细信息

```
127.0.01:6379> get DD54
"16"
127.0.01:6379> hget members_104_9808_0
(error) ERR wrong number of arguments for 'hget' command
#上面这个错误，是因为类型的原因导致的
```

**查看key的类型**

> type key-name

**Hash类型存储的数据**需要用`hgetall`来获取完整的内容，也可以通过`hget key-name field-name`来查看某个字段的名字

```
127.0.0.1:6379> hgetall members_104_9808_0
 1) "uid"
 2) "9808"
 3) "zid"
 4) "0"
 
 127.0.0.1:6379>  hget members_104_9808_0 uid
"9808"

```

**存储一个数据**

> set key value