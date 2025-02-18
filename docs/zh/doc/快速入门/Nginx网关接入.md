# Nginx网关接入

* [功能简介](#功能简介)
* [快速入门示例](#快速入门示例)

---

## 功能简介

北极星的服务治理能力，支持在微服务网关层使用，下面主要介绍如何在Nginx网关上，基于北极星使用动态upstream管理、访问限流、流量监控等能力。

## 快速入门示例

### kubernetes环境使用

#### 部署polaris

如果已经部署好了polaris，可忽略这一步。

polaris支持在kubernetes环境中进行部署，注意必须保证暴露HTTP端口为8090，gRPC端口为8091。具体部署方案请参考：

- [单机版部署指南](https://polarismesh.cn/zh/doc/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E5%AE%89%E8%A3%85%E6%9C%8D%E5%8A%A1%E7%AB%AF/%E5%AE%89%E8%A3%85%E5%8D%95%E6%9C%BA%E7%89%88.html#kubernetes-%E5%AE%89%E8%A3%85)
- [集群版部署指南](https://polarismesh.cn/zh/doc/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E5%AE%89%E8%A3%85%E6%9C%8D%E5%8A%A1%E7%AB%AF/%E5%AE%89%E8%A3%85%E9%9B%86%E7%BE%A4%E7%89%88.html#%E9%83%A8%E7%BD%B2%E5%9C%A8kubernetes)

#### 访问限流

- 部署polaris-limiter：注意，如果使用的是1.12之前版本的polaris单机版，则需要额外部署polaris-limiter，作为分布式限流的集中式token-server。部署方法可参考：[polaris-limiter](https://github.com/polarismesh/polaris-limiter)

- 获取样例：下载最新版本的nginx-gateway的[release](https://github.com/polarismesh/nginx-gateway/releases)，获取对应版本的源码包：Source code (zip)。

- 部署文件说明：以1.1.0-beta.0为例，源码包名称为：nginx-gateway-1.1.0-beta.0.zip，解压后，部署文件为examples/ratelimit/nginx.yaml，里面包含有如下环境变量，用户可以按照自己的需要进行修改。

| 变量名                         | 默认值                      | 说明                                         |
| ------------------------------ | --------------------------- | -------------------------------------------- |
| polaris_address                | polaris.polaris-system:8091 | 北极星服务端地址，8091为GRPC端口             |
| polaris_nginx_namespace        | default                     | 网关服务所在的命名空间                       |
| polaris_nginx_service          | nginx-gateway               | 网关服务所在的服务名，用于查找并关联限流规则 |
| polaris_nginx_ratelimit_enable | true                        | 是否启用限流功能                             |

- 部署样例：以1.1.0-beta.0为例，源码包名称为：nginx-gateway-1.1.0-beta.0.zip

```
unzip nginx-gateway-1.1.0-beta.0.zip
cd nginx-gateway-1.1.0-beta.0
kubectl apply -f examples/ratelimit/nginx.yaml
```

- 配置限流规则

在北极星控制台新建一个限流规则，服务名和命名空间分别选择nginx-gateway以及default，设置根据http header（user=1000）进行流量限制。

![](图片/nginx接入/限流规则.png)

- 验证限流效果

通过postman，在http请求中带上头信息：user:1000，一分钟超过5次调用，会返回限流错误。

![](图片/nginx接入/限流效果.png)

### 虚拟机环境使用

#### 部署polaris

如果已经部署好了polaris，可忽略这一步。

polaris支持在linux虚拟机环境中进行部署，注意必须保证暴露HTTP端口为8090，gRPC端口为8091。具体部署方案请参考：

- [单机版部署指南](https://polarismesh.cn/zh/doc/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E5%AE%89%E8%A3%85%E6%9C%8D%E5%8A%A1%E7%AB%AF/%E5%AE%89%E8%A3%85%E5%8D%95%E6%9C%BA%E7%89%88.html#linux)
- [集群版部署指南](https://polarismesh.cn/zh/doc/%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E5%AE%89%E8%A3%85%E6%9C%8D%E5%8A%A1%E7%AB%AF/%E5%AE%89%E8%A3%85%E9%9B%86%E7%BE%A4%E7%89%88.html#%E9%83%A8%E7%BD%B2%E5%9C%A8linux%E8%99%9A%E6%8B%9F%E6%9C%BA)

#### 安装nginx网关

- 下载安装包：可以在GITHUB的版本页面下载最新的虚拟机安装包，当前只提供基于ubuntu-latest编译的虚拟机安装包，如需其他版本，可以通过源码编译的方式来进行打包。版本下载地址：[release](https://github.com/polarismesh/nginx-gateway/releases)

- 解压安装包：以nginx-gateway-release-1.1.0-beta.0.linux.x86_64.tar.gz为例。

```
tar xf nginx-gateway-release-1.1.0-alpha.6.linux.x86_64.tar.gz
cd nginx-gateway-release-1.1.0-alpha.6.linux.x86_64
```

- 修改端口号（可选）：nginx默认端口号为80，如需修改端口号，可以通过编辑conf/nginx.conf配置文件进行修改。

```
http {
  server {
    listen 80; #这里修改成希望监听的端口号
  }
}  
```

- 添加环境变量（可选）：可通过添加环境变量的方式，指定nginx-gateway对应的参数设置。环境变量说明。

| 变量名                         | 默认值                      | 说明                                         |
| ------------------------------ | --------------------------- | -------------------------------------------- |
| polaris_address                | polaris.polaris-system:8091 | 北极星服务端地址，8091为GRPC端口             |
| polaris_nginx_namespace        | default                     | 网关服务所在的命名空间                       |
| polaris_nginx_service          | nginx-gateway               | 网关服务所在的服务名，用于查找并关联限流规则 |
| polaris_nginx_ratelimit_enable | true                        | 是否启用限流功能                             |

- 重启nginx

  ```
  cd nginx-gateway-release-*/sbin
  bash stop.sh
  bash start.sh
  ```

## 其他资料

- 如果需要编译Nginx网关，可以参考：[nginx-gateway](https://github.com/polarismesh/nginx-gateway)