# SSL证书

HTTP协议无法加密数据，导致网站数据可能产生泄露、篡改或钓鱼攻击等问题。安装SSL证书后，网站使用HTTPS协议对网站数据的传输进行加密，包括您网站中的企业应用数据、政务信息、支付环节的数据都能实现加密传输，有效保护敏感数据的传输。

## 证书选型

| 行业               | 常规推荐的证书类型 | 案例                                                         | 业务特征                                                     |
| :----------------- | :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 金融、银行         | EV                 | 中国银行                                                     | 希望企业身份信息展示在网站地址栏对数据传输保密性有很高要求   |
| 教育、政府、互联网 | OV通配符证书       | 外交部淘宝、天猫新浪、今日头条上海黄金交易所国家电网用友软件浪潮阿里云 | 网站后期有多个新增站点的需求无需政府/公司名称展示在网站地址栏 |
| 个人业务           | DV                 | 个人博客等                                                   | 无数据传输业务纯信息或内容展示的网站                         |

## 购买证书

## 申请证书

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210414135820.png)

DV证书申请:

| 参数             | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| **证书绑定域名** | 填写证书用于保护的网站域名。<br />注意事项：<br />域名类型必须与您购买证书时选择的**域名类型**（单域名、通配符域名）相同。<br />填写通配符域名时，一定要加上通配符号（`*`）。例如，`*.yourwebsite.com`。<br />不支持填写`.jp`域名。<br />申请DV证书时，不支持填写`.gov`（政府机构专用）、`.edu`（教育机构专用）域名。建议您购买OV、EV证书，用于保护`.gov`和`.edu`域名。 |
| **域名验证方式** | 选择验证域名持有者身份的方式。<br />如果您填写的域名在阿里云域名列表中，则此处自动选择**自动DNS验证方式，且不支持修改。该方式由阿里云自动为您完成域名验证，无需您手动操作。<br />如果您填写的域名不在阿里云域名列表中，则此处可以选择以下两种方式中的一种：<br />手工DNS验证：该方式需要您登录域名的管理控制台，将域名验证信息配置到域名解析列表中（添加一条TXT类型的DNS解析记录）。您需要域名解析的管理权限，才可以完成验证。<br />文件验证：通过在您域名服务器上创建指定文件来验证域名的所有权。您需要域名服务器的管理员权限，才可以完成验证。 |
| **联系人**       | 从下拉列表中选择本次证书申请的联系人（包含邮箱、手机号码等联系信息）。**注意** 收到证书申请请求后，CA中心会向联系人邮箱中发送证书申请验证邮件或者通过联系人手机号码沟通审核相关事宜。因此，请务必确保联系人信息准确且有效。如果您未创建过联系人，可以单击**新建联系人**，新建一个联系人。SSL证书服务会保存新建的联系人信息，方便您下次使用。 |
| **所在地**       | 选择申请人所在城市或地区。                                   |
| **CSR生成方式**  | CSR（Certificate Signing Request）文件是您的公钥证书原始文件，包含了您的服务器信息和单位信息，需要提交给CA认证中心审核。**手动填写**开关默认关闭，表示由SSL证书服务自动为您生成CSR文件（您可以在证书签发后下载证书和私钥）。使用自动生成方式时，此处无需修改。如果打开**手动填写**开关，表示由您使用OpenSSL或Keytool工具手动生成CRS文件，并将CSR文件的内容复制粘贴到**CSR文件内容**。**注意**您的CSR文件格式正确与否直接关系到您证书申请流程是否能顺利完成，建议您使用SSL证书服务自动生成的CSR，避免内容不正确而导致审核失败。手动填写CSR的证书不支持部署到阿里云产品。在制作CSR文件时请务必保存好您的私钥文件。私钥和SSL证书一一对应，一旦丢失了私钥，您的数字证书也将不可使用。阿里云不负责保管您的私钥，如果您的私钥丢失，您需要重新购买并替换您的数字证书。 |
| **CSR文件内容**  | 仅在打开**CSR生成方式**的**手动填写**开关后，需要配置该参数。在此处填写您的CSR文件内容。 |

## 证书安装

### 下载证书

### 安装证书到服务器

#### nginx服务器安装证书

1. 在ssl证书控制台选择已签发的证书
2. 下载nginx类型的证书并上传到nginx服务器
3. 配置nginx.conf

```shell
#以下属性中，以ssl开头的属性表示与证书配置有关。
server {
    listen 443 ssl;
    #配置HTTPS的默认访问端口为443。
    #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
    #如果您使用Nginx 1.15.0及以上版本，请使用listen 443 ssl代替listen 443和ssl on。
    server_name yourdomain.com; #需要将yourdomain.com替换成证书绑定的域名。
    root html;
    index index.html index.htm;
    ssl_certificate cert/cert-file-name.pem;  #需要将cert-file-name.pem替换成已上传的证书文件的名称。
    ssl_certificate_key cert/cert-file-name.key; #需要将cert-file-name.key替换成已上传的证书密钥文件的名称。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #表示使用的加密套件的类型。
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #表示使用的TLS协议的类型。
    ssl_prefer_server_ciphers on;
    location / {
        root html;  #站点目录。
        index index.html index.htm;
    }
}
```

4. 设置HTTP请求自动跳转HTTPS

```shell
server {
    listen 80;
    server_name yourdomain.com; #需要将yourdomain.com替换成证书绑定的域名。
    rewrite ^(.*)$ https://$host$1; #将所有HTTP请求通过rewrite指令重定向到HTTPS。
    location / {
        index index.html index.htm;
    }
}
```

#### nginx虚拟主机安装证书

1. 在Web目录下创建cert目录，并将下载的证书文件和密钥文件拷贝到cert目录中。
2. 配置虚拟主机配置文件

```shell
server {
    listen 80;
    server_name localhost;
    location / {
        index index.html index.htm;
    }
}
server {
    listen 443 ssl;
    server_name localhost;
    root html;
    index index.html index.htm;
    ssl_certificate cert/cert-file-name.pem;   #需要将cert-file-name.pem替换成已上传的证书文件的名称。
    ssl_certificate_key cert/cert-file-name.key;   #需要将cert-file-name.key替换已上传的证书密钥文件的名称。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        index index.html index.htm;
    }
}
```

3. 设置HTTP请求自动跳转HTTPS

在Web目录下打开.htaccess文件（如果没有，需新建该文件），并添加以下rewrite语句。

```shell
RewriteEngine On
RewriteCond %{HTTP:From-Https} !^on$ [NC]
RewriteCond %{HTTP_HOST} ^(www.)?yourdomain.com$ [NC]   #需要将yourdomain.com替换成证书绑定的域名。
RewriteRule ^(.*)$ https://www.yourdomain.com/$1 [R=301,L]   #需要将yourdomain.com替换成证书绑定的域名。
```

### 部署证书到阿里云产品

通过SSL证书服务，将已签发的证书、已上传的证书（第三方）**一键部署**到支持的阿里云产品，实现证书在阿里云产品上的快捷应用。

部署证书到CLB、ALB限制：

- 如果CLB、ALB中配置了HTTPS双向认证的监听，则不支持通过SSL证书服务部署证书到CLB、ALB。

  这种情况下，您只能通过负载均衡控制台配置对应的证书。

- 如果CLB、ALB中已经部署过证书，则只有当待部署证书的绑定域名等于或包含已部署证书的绑定域名时，才支持通过SSL证书服务部署证书到CLB、ALB（替换已部署证书）。举例说明如下：

  - 假设您已经在CLB、ALB上部署了一个单域名证书（绑定域名为`example.com`），则只有当待部署证书的绑定域名等于或包含`example.com`时（例如，待部署证书的绑定域名为`example.com`和`www.example.com`、`*.example.com`），才可以通过SSL证书服务部署证书到CLB、ALB（替换已部署证书）。
  - 假设您已经在CLB、ALB上部署了一个通配符域名证书（绑定域名为`*.example.com`），则只有当待部署证书的绑定域名等于或包含`*.example.com`时（例如，待部署证书的绑定域名为`*.example.com`和`example.com`），才可以通过SSL证书服务部署证书到CLB、ALB（替换已部署证书）。

部署已签发的证书：

