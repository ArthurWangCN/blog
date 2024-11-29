---
title: 使用 ZeroSSL 获得免费SSL
date: 2024-08-12 22:25:36
tags:
categories: 运维
description: ZeroSSL是一个免费的、自动化的、开放的证书颁发机构，提供SSL证书。 它以其用户友好的SSL认证方法而闻名，即使是技术知识有限的人也可以访问它。 与Let's Encrypt 类似，ZeroSSL 提供90 天的免费证书，需要在到期前续订。
---
# 使用 ZeroSSL 获得免费SSL

首先打开 https://zerossl.com/ ，按照网站提示注册输入域名，一步步往下进行。顺利的话会生成一个验证的txt文件：

你需要在Nginx上配置`.well-known/pki-validation/`，以下是配置步骤：

1. **创建目录结构**：在根目录创建 `.well-known`  文件夹， 然后在这个文件夹下创建`pki-validation` 文件夹；

2. **上传验证文件**：将由证书颁发机构（CA）提供的验证文件（通常是一个TXT文件）上传到`pki-validation`目录中。这个文件的内容是由CA生成的，用于验证你拥有该域名。

3. **Nginx配置**： 修改你的Nginx配置文件，以便正确处理验证请求。以下是一个示例的Nginx配置，用于处理`.well-known/pki-validation/`路径：

   ```conf
   server {
       listen 80;
       server_name yourdomain.com;
    
       location ^~ /.well-known/pki-validation/ {
           default_type "text/plain";
           root /path/to/your/website/root;
       }
    
       # 其他配置...
   }
   ```

   将`yourdomain.com`替换为你的实际域名，将`/path/to/your/website/root`替换为你网站的根目录路径。这个配置将确保Nginx正确地返回验证文件。

4. **重新加载Nginx配置**： 保存并退出Nginx配置文件后，运行以下命令重新加载Nginx配置：

   ```shell
   sudo nginx -t  # 验证配置是否正确
   sudo systemctl reload nginx  # 重新加载Nginx配置
   ```

这样，Nginx就会正确处理`.well-known/pki-validation/`路径下的验证文件。



如果一切顺利，zerossl就会生成证书文件，下载解压，然后上传到 /ssl 文件夹，接着合并 .crt 文件：

> NGINX requires all .crt files to be merged in order to allow SSL installation. You will need to run the following command in order to merge your certificate.crt and ca_bundle.crt files.

```shell
cat certificate.crt ca_bundle.crt >> certificate.crt
```

接下来添加 nginx ssl 相关配置：

```conf
server {

    listen               443 ssl;
    
    
    ssl                  on;
    ssl_certificate      /etc/ssl/certificate.crt; 
    ssl_certificate_key  /etc/ssl/private.key;
    
    
    server_name  your.domain.com;
    access_log   /var/log/nginx/nginx.vhost.access.log;
    error_log    /var/log/nginx/nginx.vhost.error.log;
    location     / {
    root         /home/www/public_html/your.domain.com/public/;
    index        index.html;
    }

}
```

重新加载nginx配置，完成SSL证书安装。

此方案有效期为90天。