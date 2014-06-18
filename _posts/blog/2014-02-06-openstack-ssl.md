---
layout: post
title: OpenStack中使用SSL（未完）
description: SSL是一种安全协议，在Netscape推出首版Web浏览器的同时提出，目的是为网络通信提供安全及数据完整性保障，SSL在传输层中对网络通信进行加密。
category: blog
---

声明：  
本博客欢迎转发，但请保留原作者信息!  
新浪微博：[@孔令贤HW](http://weibo.com/lingxiankong)；   
博客地址：<http://lingxiankong.github.io/>  
内容系本人及本人团队学习、研究和总结，如有雷同，实属荣幸！

## 什么是SSL
安全套接层（Secure Sockets Layer，SSL）是一种安全协议，在网景公司（Netscape）推出首版Web浏览器的同时提出，目的是为网络通信提供安全及数据完整性保障，SSL在传输层中对网络通信进行加密。

SSL采用公开密钥技术，保证两个应用间通信的保密性和可靠性，使客户端与服务器应用之间的通信不被攻击者窃听和伪造。它在服务器和客户机两端可同时被支持，目前已成为互联网上保密通讯的工业标准。现行的Web浏览器亦普遍将HTTP和SSL相结合，从而实现安全通信。此协议其继任者是TLS。

SSL协议的优势在于它是与应用层协议独立无关的。高层的应用层协议（例如：HTTP、FTP、Telnet等等）能透明的建立于SSL协议之上。SSL协议在应用层协议通信之前就已经完成加密算法、通信密钥的协商以及服务器认证工作。在此之后应用层协议所传送的数据都会被加密，从而保证通信的私密性。

在SSL中会使用密钥交换算法交换密钥；使用密钥对数据进行加密；使用散列算法对数据的完整性进行验证，使用数字证书证明自己的身份。SSl记录协议对数据的处理过程简单图示如下：  
![](/images/2014-02-06-openstack-ssl/1.png)

OpenStack对外提供API服务，同时内部组件之间也会相互调用，为了保证通讯安全，面向公网和内部管理网络，都需要配置使用SSL。使用SSL时，你需要一个CA发布服务端的证书，包含CA证书本身。安全起见，最好不要为不同的服务配置相同的证书。

## 证书
什么是证书不就不多说了，Google一把一大堆，简单说说自签名的证书。

通常要配置https服务器，都需要一个由正式的CA机构认证的X509证书。当客户端链接https服务器时，会通过CA的公钥来检查这个证书的正确性。但要获得CA的证书是一件很麻烦的事情，而且还要花费一定的费用。因此通常一些小的机构会是使用自签名的证书。也是自己做CA，给自己的服务器证书签名。简单说一下步骤：  
第一步制作CA证书：

	openssl genrsa -des3 -out my-ca.key 2048
	openssl req -new -x509 -days 3650 -key my-ca.key -out my-ca.crt
	
这会生成 my-ca.key 和 my-ca.crt 文件，前者存放着使用 my-ca.crt 制作签名时必须的密钥，应当妥善保管。而后者是可以公开的。上面的命令为 my-ca.key 设定的有效期为 10 年。用命令openssl x509 -in my-ca.crt -text -noout可以查看 my-ca.crt 文件的内容。有了 CA 证书之后，就可以为自己的服务器生成证书了。  

	openssl genrsa -des3 -out mars-server.key 1024
	openssl req -new -key mars-server.key -out mars-server.csr
	openssl x509 -req -in mars-server.csr -out mars-server.crt -sha1 -CA my-ca.crt -CAkey my-ca.key -CAcreateserial -days 3650
	
前两个命令会生成 key、csr 文件，最后一个命令则通过 my-ca.crt 为 mars-server.csr 制作了 x509 的签名证书。需要注意的是，在执行上述第二个命令时，Common Name 选项应当输入的是服务器的域名，否则在用户通过 https 协议访问时每次都会有额外的提示信息。

## Keystone
安装Keystone后，在服务启动前，就涉及到管理员token的配置。方法如下：

    # export SERVICE_TOKEN=$(openssl rand -hex 10)
    # echo $SERVICE_TOKEN > ~/ks_admin_token
    # openstack-config --set /etc/keystone/keystone.conf DEFAULT admin_token $SERVICE_TOKEN

> Keystone中token的定期清理：keystone-manage token-flush

在Keystone配置文件中`[signing]`段进行证书相关的配置。  
`ca_certs`: 发行certfile的证书（CA证书），默认值/etc/keystone/ssl/certs/ca.pem  
`ca_key`:  CA证书的密钥，默认是/etc/keystone/ssl/certs/cakey.pem  
`ca_password`: 读取CA证书的密码  
`certfile`: 校验token的证书，默认是/etc/keystone/ssl/certs/signing_cert.pem  
`keyfile`: 对token进行签名的密钥  
`token_format`: PKI或者UUID，默认是PKI  
生成相关的证书（使用root）：

    # keystone-manage pki_setup --keystone-user keystone --keystone-group keystone
    # chown -R keystone:keystone /var/log/keystone /etc/keystone/ssl/  

我们的主题是SSL，那么如何配置Keystone支持SSL访问呢？在`[ssl]`段：  

    [ssl]
    enable = True
    certfile = /etc/keystone/ssl/certs/ssl_cert.pem
    keyfile = /etc/keystone/ssl/private/ssl_key.pem
    ca_certs = /etc/keystone/ssl/certs/cacert.pem
    cert_required = False #要求客户端证书
    key_size = 1024
    valid_days = 3650
    cert_required = False
    cert_subject = /C=US/ST=Unset/L=Unset/O=Unset/CN=[YOUR_IP_ADDRESS]

Keystone支持使用`keystone-manage ssl_setup`命令生成证书，但在生产环境中尽量使用外部CA签发证书。  
配置完SSL后，所有其他组件中连接Keystone的`auth_protocol`配置都需要修改为https，并加上证书配置，同时client使用的环境变量也需要修改，一个例子：

    export OS_AUTH_URL=https://10.233.53.117:5000/v2.0/
    export OS_CERT=/etc/keystone/ssl/certs/ssl_cert.pem
    export OS_CACERT=/etc/keystone/ssl/certs/cacert.pem
    export OS_SERVICE_ENDPOINT=https://10.233.53.117:35357/v2.0/

使用curl验证一下，查询用户：

    $ sudo curl -i -X GET https://[YOUR_IP_ADDRESS]:35357/v2.0/users -H "User-Agent: python-keystoneclient" -H "X-Auth-Token: ADMIN" -v --cacert /etc/keystone/ssl/certs/cacert.pem
    
    * About to connect() to 10.5.52.242 port 35357 (#0)
    *   Trying 10.5.52.242...
    * Adding handle: conn: 0x2556c30
    * Adding handle: send: 0
    * Adding handle: recv: 0
    * Curl_addHandleToPipeline: length: 1
    * - Conn 0 (0x2556c30) send_pipe: 1, recv_pipe: 0
    * Connected to 10.5.52.242 (10.5.52.242) port 35357 (#0)
    * successfully set certificate verify locations:
    *   CAfile: /etc/keystone/ssl/certs/cacert.pem
      CApath: /etc/ssl/certs
    * SSLv3, TLS handshake, Client hello (1):
    * SSLv3, TLS handshake, Server hello (2):
    * SSLv3, TLS handshake, CERT (11):
    * SSLv3, TLS handshake, Server finished (14):
    * SSLv3, TLS handshake, Client key exchange (16):
    * SSLv3, TLS change cipher, Client hello (1):
    * SSLv3, TLS handshake, Finished (20):
    * SSLv3, TLS change cipher, Client hello (1):
    * SSLv3, TLS handshake, Finished (20):
    * SSL connection using AES256-SHA
    * Server certificate:
    *  subject: C=US; ST=Unset; O=Unset; CN=10.5.52.242
    *  start date: 2013-10-30 22:00:47 GMT
    *  expire date: 2023-10-28 22:00:47 GMT
    *  common name: 10.5.52.242 (matched)
    *  issuer: C=US; ST=Unset; L=Unset; O=Unset; CN=10.5.52.242
    *  SSL certificate verify ok.
    > GET /v2.0/users HTTP/1.1
    > Host: 10.5.52.242:35357
    > Accept: */*
    > User-Agent: python-keystoneclient
    > X-Auth-Token: ADMIN
    > 
    < HTTP/1.1 200 OK
    HTTP/1.1 200 OK
    < Vary: X-Auth-Token
    Vary: X-Auth-Token
    < Content-Type: application/json
    Content-Type: application/json
    < Content-Length: 152
    Content-Length: 152
    < Date: Wed, 30 Oct 2013 22:13:01 GMT
    Date: Wed, 30 Oct 2013 22:13:01 GMT
    
    < 
    * Connection #0 to host 10.5.52.242 left intact
    
    {"users": [{"name": "admin", "id": "19ae15e12f1c4c0fb02ee21afe121088", "enabled": true, "email": null, "tenantId": "3e8d46120c4e4233be3cc323d8547743"}]}

## Nova
Nova中与SSL相关的配置项：  
enabled_ssl_apis：哪些服务使用SSL  
ssl_ca_file：CA证书  
ssl_cert_file：API server的证书  
ssl_key_file：API server的私钥  
tcp_keepidle：每个server socket的TCP_KEEPIDLE值   

Nova中配置SSL访问Glance：  
glance_api_insecure=false  
glance_api_ssl_compression=false #是否协商压缩算法  
（review链接[在此](https://review.openstack.org/#/c/32562/)）

## Cinder
Cinder中与SSL相关的配置项：ssl_ca_file，ssl_cert_file，ssl_key_file  
（review链接[在此](https://review.openstack.org/#/c/19560/)）  

Cinder中配置SSL访问Glance：  
glance_api_insecure=false  
glance_api_ssl_compression=false #是否协商压缩算法  

Cinder中配置SSL访问Nova：  
nova_api_insecure=false

## Glance
review链接[在此](https://review.openstack.org/#/c/190/)

## Neutron
review链接[在此](https://review.openstack.org/#/c/20459/)  
client的配置：<https://review.openstack.org/#/c/20922/>

## Heat
Heat中要调用不同组件的client，必须允许SSL配置，因为不同的服务可能已经配置了SSL，相关链接如下：  
<https://bugs.launchpad.net/heat/+bug/1213122>  

## services behind web server
虽然OpenStack支持为不同的服务配置SSL，但为了消息处理的一致性以及OpenStack服务处理消息的效率，最好的实践还是在所有SSL服务的前端统一处理SSL。OpenStack所有的服务都遵循WSGI标准，使得可以使用web server(比如Apache或是Nginx)作为OpenStack服务运行的容器，而由web server提供SSL服务。这里，我们使用Apache2 + mod_wsgi。

**Keystone**  
Keystone已经支持运行在HTTPD下：<https://github.com/openstack/keystone/blob/master/doc/source/apache-httpd.rst>，需要的文件在[这里](https://github.com/openstack/keystone/tree/master/httpd)

**Nova**  
<http://andymc-stack.co.uk/2013/07/apache2-mod_wsgi-openstack-pt-2-nova-api-os-compute-nova-api-ec2/>  
<http://www.rackspace.com/blog/enabling-ssl-for-the-openstack-api/>

**Cinder**  
<http://andymc-stack.co.uk/2013/07/apache2-mod_wsgi-openstack-pt-4-cinder-api/>

**Neutron**  
<http://andymc-stack.co.uk/2013/07/apache2-mod_wsgi-openstack-pt-6-quantum-server/>

**Glance**  
<http://andymc-stack.co.uk/2013/07/apache2-mod_wsgi-openstack-pt-3-glance-api-glance-registry/>

**Ceilometer**  
<http://andymc-stack.co.uk/2013/07/apache2-mod_wsgi-openstack-pt-5-ceilometer-api/>

### Heat
相关的[bug](https://bugs.launchpad.net/heat/+bug/1235555)：Heat API cannot cope with being behind an SSL terminator

## AMQP
基于AMQP的组件（Qpid 和 RabbitMQ）支持SSL，但ZeroMQ本身并不支持SSL。以RabbitMQ为例，配置使用SSL能够防止消息被篡改或被窃听，最好使用内部CA进行证书的签发。虽然RabbitMQ支持Simple Authentication and Security Layer (SASL)，但目前在OpenStack的使用中并不支持。RabbitMQ server的相关配置，/etc/rabbitmq/rabbitmq.config：

    [
      {rabbit, [
         {tcp_listeners, [] },
         {ssl_listeners, [{"<ip address or hostname of management network interface", 5671}] },
         {ssl_options, [{cacertfile,"/etc/ssl/cacert.pem"},
                        {certfile,"/etc/ssl/rabbit-server-cert.pem"},
                        {keyfile,"/etc/ssl/rabbit-server-key.pem"},
                        {verify,verify_peer},
                        {fail_if_no_peer_cert,true}]}
       ]}
    ].

OpenStack服务端的配置：

    [DEFAULT]
    rpc_backend=nova.openstack.common.rpc.impl_kombu
    rabbit_use_ssl=True
    rabbit_host=
    rabbit_port=5671
    rabbit_user=compute01
    rabbit_password=password
    kombu_ssl_keyfile=/etc/ssl/node-key.pem
    kombu_ssl_certfile=/etc/ssl/node-cert.pem
    kombu_ssl_ca_certs=/etc/ssl/cacert.pem
    kombu_ssl_version=SSLv3 #valid values are TLSv1, SSLv23 and SSLv3

更详细的SSL配置请参考[这里](http://www.rabbitmq.com/ssl.html)

附，最好对每个OpenStack服务创建相应的RabbitMQ用户：

    rabbitmqctl delete_user quest
    rabbitmqctl add_user compute01 password
    rabbitmqctl set_permissions compute01 ".*"".*"".*"

## Database
使用DB时，也可以配置仅接受SSL连接，如果是MySQL：

    GRANT ALL ON dbname.* to 'compute01'@'hostname' IDENTIFIED BY  'password' REQUIRE SSL;

在my.cnf:

    [[mysqld]]
    ...
    ssl-ca=/path/to/ssl/cacert.pem
    ssl-cert=/path/to/ssl/server-cert.pem
    ssl-key=/path/to/ssl/server-key.pem

使用MySQL的客户端需要添加证书和密钥来连接数据库

如果是PostgresSQL，需要在pg_hba.conf文件中：

    hostssl dbname compute01 hostname md5