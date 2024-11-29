---
title: Nginx或Tengine服务器配置SSL证书
date: 2024-03-08 12:40:22
tags: 
categories: 服务器
description: 本文将全面介绍如何在Nginx或Tengine服务器配置SSL证书，具体包括下载和上传证书文件，在Nginx上配置证书文件、证书链和证书密钥等参数，以及安装证书后结果的验证。成功配置SSL证书后，您将能够通过HTTPS加密通道安全访问Nginx服务器。
---

## 前提条件

- 已通过数字证书管理服务控制台签发证书。
- SSL证书绑定的域名已完成DNS解析，即您的域名与主机IP地址相互映射。您可以通过DNS验证证书工具，检测域名DNS解析是否生效。
- 已在Web服务器开放443端口（HTTPS通信的标准端口）。如果您使用的是阿里云ECS服务器，请确保已经在安全组规则入方向添加TCP 443端口。

（本文以阿里云为例）

## 步骤一：下载SSL证书

1. 登录[数字证书管理服务控制台](https://yundunnext.console.aliyun.com/?p=cas)。

2. 在左侧导航栏，单击**SSL 证书**。

3. 在**SSL 证书**页面，定位到目标证书，在**操作**列，单击**下载**。

4. 在**服务器类型**为Nginx的**操作**列，单击**下载**。

   ![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/1097065861/p677531.png)

5. 解压缩已下载的SSL证书压缩包。

   根据您在提交证书申请时选择的CSR生成方式，解压缩获得的文件不同，具体如下表所示。

   ![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/2050165861/p677479.png)

   | **CSR生成方式**                 | **证书压缩包包含的文件**                                     |
   | ------------------------------- | ------------------------------------------------------------ |
   | **系统生成**或**选择已有的CSR** | 证书文件（PEM格式）：Nginx支持安装PEM格式的文件，PEM格式的证书文件是采用Base64编码的文本文件，且包含完整证书链。解压后，该文件以`证书ID_证书绑定域名`命名。私钥文件（KEY格式）：默认以*证书绑定域名*命名。 |
   | **手动填写**                    | 如果您填写的是通过数字证书管理服务控制台创建的CSR，下载后包含的证书文件与**系统生成**的一致。如果您填写的不是通过数字证书管理服务控制台创建的CSR，下载后只包括证书文件（PEM格式），不包含证书密码或私钥文件。您可以通过证书工具，将证书文件和您持有的证书密码或私钥文件转换成所需格式。转换证书格式的具体操作，请参见[证书格式转换](https://help.aliyun.com/document_detail/469153.html#section-7pl-isf-owk)。 |



## 步骤二：在Nginx服务器安装证书

1. 执行以下命令，在Nginx的conf目录下创建一个用于存放证书的目录。

   ```shell
   cd /usr/local/nginx/conf  #进入Nginx默认配置文件目录。该目录为手动编译安装Nginx时的默认目录，如果您修改过默认安装目录或使用其他方式安装，请根据实际配置调整。
   mkdir cert  #创建证书目录，命名为cert。
   ```

2. 将证书文件和私钥文件上传到Nginx服务器的证书目录（/usr/local/nginx/conf/cert）。

3. 编辑Nginx配置文件nginx.conf，修改与证书相关的配置。

   1. 执行以下命令，打开配置文件。

      ```shell
      vim /usr/local/nginx/conf/nginx.conf
      ```

      > nginx.conf默认保存在/usr/local/nginx/conf目录下。如果您修改过nginx.conf的位置，可以执行`nginx -t`，查看nginx的配置文件路径，并将`/usr/local/nginx/conf/nginx.conf`进行替换。

   2. 在nginx.conf中定位到server属性配置。

      ```shell
      server {
           #HTTPS的默认访问端口443。
           #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
           listen 443 ssl;
           
           #填写证书绑定的域名
           server_name <yourdomain>;
       
           #填写证书文件绝对路径
           ssl_certificate cert/<cert-file-name>.pem;
           #填写证书私钥文件绝对路径
           ssl_certificate_key cert/<cert-file-name>.key;
       
           ssl_session_cache shared:SSL:1m;
           ssl_session_timeout 5m;
      	 
           #自定义设置使用的TLS协议的类型以及加密套件（以下为配置示例，请您自行评估是否需要配置）
           #TLS协议版本越高，HTTPS通信的安全性越高，但是相较于低版本TLS协议，高版本TLS协议对浏览器的兼容性较差。
           ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
           ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
      
           #表示优先使用服务端加密套件。默认开启
           ssl_prefer_server_ciphers on;
       
       
          location / {
                 root html;
                 index index.html index.htm;
          }
      }
      ```

   3. **可选：**设置HTTP请求自动跳转HTTPS。

      如果您希望所有的HTTP访问自动跳转到HTTPS页面，可通过rewrite指令重定向到HTTPS。

      > 以下代码片段需要放置在nginx.conf文件中`server {}`代码段后面，即设置HTTP请求自动跳转HTTPS后，nginx.conf文件中会存在两个`server {}`代码段。

      ```shell
      server {
          listen 80;
          #填写证书绑定的域名
          server_name <yourdomain>;
          #将所有HTTP请求通过rewrite指令重定向到HTTPS。
          rewrite ^(.*)$ https://$host$1;
          location / {
              index index.html index.htm;
          }
      }
      ```

   4. 执行以下命令，重启Nginx服务。

      ```shell
      cd /usr/local/nginx/sbin  #进入Nginx服务的可执行目录。
      ./nginx -s reload  #重新载入配置文件。
      ```

   >**说明**
   >
   >报错`the "ssl" parameter requires ngx_http_ssl_module`：您需要重新编译Nginx并在编译安装的时候加上`--with-http_ssl_module`配置。
   >
   >报错`"/cert/3970497_demo.aliyundoc.com.pem":BIO_new_file() failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/cert/3970497_demo.aliyundoc.com.pem','r') error:2006D080:BIO routines:BIO_new_file:no such file)`：您需要去掉证书相对路径最前面的`/`。例如，您需要去掉`/cert/cert-file-name.pem`最前面的`/`，使用正确的相对路径`cert/cert-file-name.pem`。



## 步骤三：验证SSL证书是否配置成功

证书安装完成后，您可通过访问证书的绑定域名验证该证书是否安装成功。

如果网页地址栏出现小锁标志，表示证书已经安装成功。



本文来源：https://help.aliyun.com/zh/ssl-certificate/user-guide/install-ssl-certificates-on-nginx-servers-or-tengine-servers#concept-n45-21x-yfb