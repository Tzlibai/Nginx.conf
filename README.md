# 单域名、二级域名配置SSL证书升级HTTPS的nginx.conf文件配置
随着互联网的飞速发展，我们的工作生活已经离不开互联网，HTTP虽然使用极为广泛, 但是存在不小的安全缺陷, 主要是其数据的明文传送和消息完整性检测的缺乏, 而这两点恰好是网络支付,网络交易等网站应用中安全方面最需要关注的。

谷歌、火狐、360等浏览器开始要求强制使用HTTPS网站，小程序开发也必须是HTTPS网站，企业、网站网页部署HTTPS加密似乎已经势不可挡，今天来讲解一下怎么通过Nginx快速把我们的http网站升级成https。



## 需要准备的工具
### 一台可用的服务器

首先需要有一台可用的服务器，后续用作域名解析。如果没有服务器可以购买[腾讯云服务器](https://curl.qcloud.com/4tVsvMPS)或者[阿里云服务器](https://www.aliyun.com/minisite/goods?userCode=wqn73bdw&share_source=copy_link)

### 一个备案过的域名

需要一个备过案的域名，用于解析服务器的IP地址。在对应购买的域名的服务商网站控制台可以完成备案。
    
### SSL证书
SSL证书有免费和收费两个渠道。

我们这边主要介绍腾讯云和阿里云(排名不分先后)两个免费申请SSL证书的方式，当然资金允许情况下也可以直接购买。

#### 阿里云申请免费SSL证书
**阿里云免费证书规则**：自2021年起，免费证书申请将切换到证书资源包下每个实名个人/企业主体在一个自然年内可以一次性领取20张免费证书，免费证书每张证书有效期一年。免费证书仅支持绑定一个单域名，不支持绑定通配符域名或者IP。

阿里云申请免费SSL证书文档：[官方文档地址](https://help.aliyun.com/document_detail/156645.htm?spm=a2c4g.11186623.2.7.2db71a63TXmPoB#task-2436672)

#### 腾讯云申请免费SSL证书
**腾讯云免费证书规则**：只支持绑定1个域名，可以支持绑定二级域名 abc.com、或是三级域名 example.abc.com。同一主域最多只能申请20张免费证书,每张有效期一年，免费证书到期后如需继续使用证书，需要重新申请并安装。

腾讯云申请免费SSL证书文档：[官方文档地址](https://cloud.tencent.com/document/product/400/6814)

## 域名升级成HTTPS - 实战
我们如果直接用域名解析IP地址，域名也是可以访问的，但是是HTTP环境的。
我们使用的**Nginx**来配置升级HTTPS
### 登录服务器
 市场上有很多终端登录工具，比如：xshell、Putty、FinalShell等等，大家可以根据自己的喜好自行安装SSH工具进入服务器(下文以FinalShell工具为示例)。
#### 下载并安装FinalShell
大家也可自行搜索下载SSH客户端，登录页面大同小异，下面以FinalShell工具为例，安装完成后开始登录我们的服务器。


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae5594b200ec4cff94527cc22d8f5c79~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b003d463ed404320999211213867bfd7~tplv-k3u1fbpfcp-watermark.image)


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32e1aac500f749888a5fd35e897a0cbf~tplv-k3u1fbpfcp-watermark.image)

### 服务器安装Nginx
　　Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强,在SSH连接工具连接成功后，输入命令安装Nginx；

 ```linux
// 安装nginx
yum install -y nginx
 
// 设置nginx开机启动
systemctl start nginx.service
systemctl enable nginx.service
 ```
但是nginx默认使用端口 80， 我们购买的服务器实例一般默认不开启端口80，默认只有 22 和 3389端口，我们可以通过设置，打开80和443端口；登录相应服务器管理网站打开端口服务，我们这边以阿里云为例开始设置。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11ceee85788c494bacfb7c456ef29b1b~tplv-k3u1fbpfcp-watermark.image)

设置好之后，我们直接访问我们服务器的公网IP地址，就可以看到如下画面，就代表Nginx设置好了；

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8fcfdcf9913410f9d33fac8ece7e9ea~tplv-k3u1fbpfcp-watermark.image)

### Nginx服务器安装SSL证书
#### 单域名升级HTTPS

　　当我们在上面申请成功免费的SSL证书之后，点击下载证书。
    
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97576f96dcaa4e0c93a00bac763c8fa8~tplv-k3u1fbpfcp-watermark.image)

下载完成会得到一个压缩包，我们解压之后选择Nginx文件夹，里面的两个文件就是我们后续需要配置的文件。
下载到本地的压缩文件包解压后Nginx文件夹包含：
- .crt文件：是证书文件，crt是pem文件的扩展名。
- .key文件：证书的私钥文件。


这时候我们打开我们的SSH工具，进入Nginx的配置：
```
// 进入nginx目录,默认安装在/etc/nginx，这个目录如果未找到，可以根据nginx安装的位置进入
cd /etc/nginx
```
1. 在Nginx的安装目录下创建cert目录，并且将下载的全部文件拷贝到` /etc/nginx/cert `目录中（**使用SSH工具附带的本地文件上传功能，将本地证书文件和密钥文件上传到Nginx服务器的证书目录[示例中为/etc/nginx/cert]**）。如果申请证书时是自己创建的CSR文件，请将对应的私钥文件放到cert目录下并且命名为a.key；

    ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5eb73f7c43141d3ba092874b76a249e~tplv-k3u1fbpfcp-watermark.image)
    
    ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb4819eb932f4c7581fa9de7acfc7c4d~tplv-k3u1fbpfcp-watermark.image)

> 

2. 然后开始Nginx配置,编辑Nginx配置文件（nginx.conf）

   ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00d48e2154b04d65b3e1c7d7376cec6b~tplv-k3u1fbpfcp-watermark.image)
    ```
    // 进入nginx目录,默认安装在/etc/nginx，这个目录如果未找到，可以根据nginx安装的位置进入
    cd /etc/nginx

    // 编辑nginx的配置文件
    vi nginx.conf
    ```
>

3. 修改与证书相关的配置内容`按i键进入编辑模式`,增加代码，监听443端口，如下
    ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c4aad9ad4534bf4b80dd906a4d5f48f~tplv-k3u1fbpfcp-watermark.image)

    ```
      server {
            listen       443 ssl;
            listen       [::]:443;
            server_name  localhost;
             ssl on;

                    root /usr/share/nginx/html;
                    index index.html index.htm;
            #证书文件名称
            ssl_certificate cert/a.crt;
            #私钥文件名称
            ssl_certificate_key cert/a.key;

            ssl_session_cache shared:SSL:1m;
            ssl_session_timeout  5m;
            ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_prefer_server_ciphers on;
            # Load configuration files for the default server block.
            include /etc/nginx/default.d/*.conf;

            location / {
            }

            error_page 404 /404.html;
                location = /40x.html {
            }
            error_page 500 502 503 504 /50x.html;
                location = /50x.html {
            }
        }
    ```
    
    修改完成后，按Esc键、输入`:wq！`并按Enter键，保存修改后的配置文件并退出编辑模式。
>
>4.  执行命令`/sbin/nginx -s reload`  重启服务器即可！（如果重启未成功，说明配置文件nginx.conf错误，检查是否有错误，后执行重启）
>
5. 验证是否安装成功,证书安装完成后，可通过访问证书的绑定域名验证该证书是否安装成功。例如：https://www.域名.com



#### 二级域名配置HTTPS
上述步骤成功之后，我们就可以通过`https://www.域名.com` 来访问我们网站了，但是问题来了，因为我们申请的免费SSL证书是单域名绑定，也就是只能让`https://www.域名.com` 下的内容实现HTTPS访问，我们的二级域名`api.域名.com`还是只能HTTP访问。

假设我们现在希望`api.域名.com`以HTTPS访问应该怎么办呢？
##### 添加二级域名解析
首先我们需要添加二级域名的解析，首先保证`httpL//api.域名.com`可以正常访问,IP地址处就填写我们的公网IP地址。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a6fda142c4440248af86e76d5aeae3c~tplv-k3u1fbpfcp-watermark.image)


##### 二级域名申请SSL证书
申请SSL证书的步骤与上面一致，这里就不在赘述了。最多可以申请20张免费证书，只是申请域名SSL证书的时候，记得填写我们的`api.域名.com`,申请成功后我们下载到本地。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8aa5ffe9994f471d8d2b2a9288249dfa~tplv-k3u1fbpfcp-watermark.image)

#### 把下载的证书文件放入服务器
这里步骤与上述一致，这里简单说下：
这时候我们打开我们的SSH工具，进入Nginx的配置：
```
// 进入nginx目录
cd /etc/nginx
```
在Nginx的安装目录下创建cert目录，并且将下载的全部文件拷贝到` /etc/nginx/cert `目录中（**使用SSH工具附带的本地文件上传功能，将本地证书文件和密钥文件上传到Nginx服务器的证书目录[示例中为/etc/nginx/cert]**）。

**为了避免与其他证书文件呼啸，我们可以将对应的私钥文件放到cert目录下并且命名为api.key、api.crt；**

#### 编辑nginx.conf文件
```
    // 进入nginx目录,默认安装在/etc/nginx，这个目录如果未找到，可以根据nginx安装的位置进入
    cd /etc/nginx

    // 编辑nginx的配置文件
    vi nginx.conf
    
    
    // 我们也可以把**nginx.conf文件**保存至本地然后打开编辑修改，
```

# 我们也可以把nginx.conf文件保存至本地然后修改
```

##### 开启监听8100端口用于二级域名使用
在**nginx.conf**文件中添加监听8100端口(别忘记检查服务器是否开启了8100端口)：
```
    server {
        listen       8100 default_server;
        listen       [::]:8100 default_server;
        server_name  _;
        # 这里root表示指向的目录,可以根据自身配置路径
        root         /usr/share/nginx/html;
    
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
    
        location / {
        }
    
        error_page 404 /404.html;
            location = /40x.html {
        }
    
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

##### 二级域名指向8100端口并配置SSL证书

在**nginx.conf**文件中再次添加监听443接口：

```
    server {
        listen       443 ssl;
        # 注：域名处修改成自己的域名地址
        server_name  api.域名.com;
     ssl on;


​		
        #证书文件名称
        ssl_certificate cert/api.crt;
        #私钥文件名称
        ssl_certificate_key cert/api.key;
    
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;


​		

		location / {
	                # 注：域名处修改成自己的域名地址
		   proxy_pass http://www.域名.com:8100;
		   proxy_set_header Host $host;
	    # 获取请求的ip地址
	    proxy_set_header X-real-ip $remote_addr;
	    # 获取请求的多级ip地址，当请求经过多个反向代理时，会获取多个ip，英文逗号隔开
	
	    }
	    
		root /usr/share/nginx/html;
		index index.html index.htm;
		
	    error_page 404 /404.html;
	        location = /40x.html {
	    }
	    error_page 500 502 503 504 /50x.html;
	        location = /50x.html {
	    }
	}
