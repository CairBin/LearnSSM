# 数据库驱动报错

## 8.0驱动问题

### 问题
我文章里都是以8.0.33这个数据库驱动作为演示的，可能某些情况会报错
驱动报错如下

```
Loading class com.mysql.jdbc.Driver'. This is deprecated. The new driver class iscom.mysql.cj.jdbc.Driver’. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
```

### 解决方案

将`pom.xml`里`com.mysql.jdbc.Driver`换成`com.mysql.cj.jdbc.Driver`

另外如果数据库为8.0.x版本，连接是不需要SSL的，然后最好使用Unicode编码，并设置服务器时区。