---
description: "算上之前配的内网服务器，这应该是第三次配置HTTPS了，但每次或多或少都要搜索一些资料和配置说明，这次直接写在博客里，以供之后翻阅查找信息。"
time: 2021-05-17
heroImage: ""
tags: []
---

算上之前配的内网服务器，这应该是第三次配置HTTPS了，但每次或多或少都要搜索一些资料和配置说明，这次直接写在博客里，以供之后翻阅查找信息。  
## 简单原理
HTTPS 协议是由 **HTTP 加上 TLS/SSL 协议** 构建的可进行加密传输、身份认证的网络协议，主要通过数字证书、加密算法、非对称密钥等技术完成互联网数据传输加密，实现互联网传输安全保护。  
有HTTPS的加持，我们的网站传输的数据多了三道安全保证：  
* 数据保密性  
* 数据完整性
* 身份校验安全性  

有了HTTPS协议，浏览器与服务器之间的交互大概可以概括为一下几个过程。  

1. 客户端向服务端发起建立HTTPS请求。

2. 服务器向客户端发送数字证书。

3. 客户端验证数字证书，证书验证通过后客户端生成**会话密钥**（双向验证则此处客户端也会向服务器发送证书）。

4. 服务器生成会话密钥（双向验证 此处服务端也会对客户端的证书验证）。

5. 客户端与服务端开始进行加密会话（通过会话密钥加密）。

## SSL证书申请

这一步实际上可以通过你的域名提供商申请一个SSL证书，对于大多数网站来说，申请个人免费的证书已经绰绰有余了，CA会使用生成一对服务器的公私钥，然后用CA的私钥加密服务器的公钥得到公钥证书，最后将公钥证书颁发给你。  

经过上面的操作，我们就可以得到公钥证书以及服务器的私钥，这里我们就可以配置HTTPS协议的网站了。

> 本站使用的是腾讯云颁发的免费SSL证书，要注意免费的SSL证书不能使用通配符域名，可以是一级域名或者二级域名。  

## Nginx 配置

一般得到SSL证书后，可以根据服务器的类型不同选择不同的证书类型下载。选择Nginx服务器下载证书后，会得到 `.key` 文件和 `.pem` 文件，分别是你的私钥文件和公钥证书文件。**（私钥文件一定不要泄露出去）**  

之后，我们可以将这两个文件放在Nginx的配置文件目录下（例如`/etc/nginx`，也就是有`nginx.conf`的目录下）。  
这里我们在`nginx.conf`的同级目录下新建一个`cert`文件夹，将两个文件放进去，之后只需要更改Nginx的配置文件指定公私钥的位置即可。  

例如：
```nginx
server {
 listen 443; # 监听443端口
 server_name localhost; # 这里放对应的域名
 ssl on;
 root html;
 index index.html index.htm;
 ssl_certificate   cert/a.pem; # 这里对应公钥证书文件
 ssl_certificate_key  cert/a.key; # 这里对应私钥文件
 ssl_session_timeout 5m;
 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; # 需要根据你的 openssl 版本支持的算法套件来配置
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_prefer_server_ciphers on;
 location / {
     # 这里就是你自己的location块设置
     root html;
     index index.html index.htm;
 }
}
```

### HTTP请求转发
可以在配置中多加入一个server块，监听80端口，并将请求转发给HTTPS的URL。
```nginx
server {  
    listen 80;  
    server_name www.wenhaofan.com;  
       
    rewrite ^(.*)$  https://$host$1 permanent;  
}
```
或者还有一个方法，直接在443的server块中同时监听80端口，并加入
```nginx
if ($scheme = http) {
    return 301 https://$server_name$request_uri;
}
```
将请求转发。

## 其他
这里记录一些遇到的因为不太理解导致的比较坑的过程以及对应的解决方法。  

### 关于重定向次数过多
在刚配完HTTPS并部署重启后，出现了后端请求失败，错误代码 `ERR_TOO_MANY_REDIRECTS` 重定向过多的情况，最后解决发现是出现cookie重定向网站，清除浏览器数据后，重新访问发现问题解决。

### 关于HTTP资源请求失败
在解决了以上问题后，访问一些博客内容发现用来存放图片等资源的对象存储OSS资源全部失效，查看发现是图片内容被重定向到HTTPS请求。  

之前Chrome对于HTTPS站点的HTTP子资源是提示不安全但不阻止，查阅资料后发现，从2020年 12 月开始测试的 Chrome 79 开始，Chrome 将会逐步阻止所有混合内容。到 2020 年 1 月，Chrome 80 会将所有混合音频和视频资源自动升级为 HTTPS，如果无法通过 HTTPS 加载，则将自动被阻止。最终，在 2020 年 2 月，Chrome 81 将所有混合图像、音频与视频自动升级为 HTTPS，并且阻止那些无法通过 HTTPS 加载的图像。

这个问题可以通过设置Chrome允许不安全内容解决，但不可能要求所有人都不能用新版Chrome，并且不安全内容本身也是不安全因素。于是决定将OSS存储服务也升级HTTPS，因为用的是`img.`子域名CNAME记录的CDN加速服务，又单独申请了SSL证书，在服务提供商升级了HTTPS请求，代价就是1GB两毛八的额外流量费用，不过至少问题是解决了。
