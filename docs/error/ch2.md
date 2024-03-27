# TTomcat10版本问题导致Servlet报错

## Tomcat10包名不一致问题

### 问题

对于SpingMVC 5.x 来说，DispatcherServlet是属于javax包下的

但是对于Tomcat10及以上版本`javax`就变为了`jakarta`

### 解决方案

即使将`pom.xml`以及控制器的包名改为`jakarta`，也会存在404找不到路径，最佳解决方案就是将tomcat回退至9.0版本。
