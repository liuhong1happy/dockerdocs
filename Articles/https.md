# Docker with HTTPS #

默认情况下，Docker运行基于未联网的Unix Socket。它可能也仅仅只需要使用HTTP Socket进行通信。

如果你需要Docker接受在安全人员下链接网络，你可以启用TLS通过指定tlsverify标志和指定Docker的tlscacert标志来得到一个信任的CA证书。

在守护进程模式下，它仅允许来自客户端具有CA认证的链接。在客户端模式下，它仅通过CA签证和服务器联络。

> 警告：使用TSL和管理一个CA是高级话题。请熟悉OpenSSL，x509和TLS在你使用它作为部署之前。

> 警告：这些TLS命令，仅仅生成一个系列有用的证书在Linux上。Mac OS X本身携带一个版本的OpenSSL，这不能和Docker 要求的证书相同并论。

## 创建一个CA，服务端和客户端的OpenSSL key ##

首先，初始化CA系列文件，生产CA私有和共有key：

    $ echo 01> ca.srl
    
    $ openssl genrsa -des3 -out ca-key.pem 2048
    
    Generating RSA private key,2048 bit long modulus
    
    ......+++
    
    ...............+++
    
    e is65537(0x10001)
    
    Enterpass phrase for ca-key.pem:
    
    Verifying-Enterpass phrase for ca-key.pem:
    
     
    
    $ openssl req -new-x509 -days 365-key ca-key.pem -out ca.pem
    
    Enterpass phrase for ca-key.pem:
    
    You are about to be asked to enter information that will be incorporated into your certificate request.
    
    What you are about to enter is what is called a DistinguishedNameor a DN.
    
    There are quite a few fields but you can leave some blank
    
    For some fields there will be a default value,
    
    If you enter '.', the field will be left blank.
    
    -----
    
    CountryName(2 letter code)[AU]:
    
    StateorProvinceName(full name)[Some-State]:Queensland
    
    LocalityName(eg, city)[]:Brisbane
    
    OrganizationName(eg, company)[InternetWidgitsPtyLtd]:DockerInc
    
    OrganizationalUnitName(eg, section)[]:Boot2Docker
    
    CommonName(e.g. server FQDN or YOUR name)[]:your.host.com
    
    EmailAddress[]:Sven@home.org.au

现在，我们有一个CA，你可以创建一个服务端key和认证。确保“Common Name” (i.e. server FQDN or YOUR name)匹配主机的名称，你将使用它来链接Docker：

    $ openssl genrsa -des3 -out server-key.pem 2048
    
    Generating RSA private key,2048 bit long modulus
    
    ......................................................+++
    
    ............................................+++
    
    e is65537(0x10001)
    
    Enterpass phrase for server-key.pem:
    
    Verifying-Enterpass phrase for server-key.pem:
    
     
    
    $ openssl req -subj '/CN=<Your Hostname Here>'-new-key server-key.pem -out server.csr
    
    Enter pass phrase for server-key.pem:

接下来，我们将用我们的CA来生产一个key：

    $ openssl x509 -req -days 365-in server.csr -CA ca.pem -CAkey ca-key.pem \  -out server-cert.pem
    
    Signature ok
    
    subject=/CN=your.host.com
    
    Getting CA PrivateKey
    
    Enterpass phrase for ca-key.pem:

为了客户端认证，创建一个客户端key和认证签订要求：

    $ openssl genrsa -des3 -out key.pem 2048
    
    Generating RSA private key,2048 bit long modulus
    
    ...............................................+++
    
    ...............................................................+++
    
    e is65537(0x10001)
    
    Enterpass phrase for key.pem:
    
    Verifying-Enterpass phrase for key.pem:
    
    $ openssl req -subj '/CN=client'-new-key key.pem -out client.csr
    
    Enterpass phrase for key.pem:

 

为了确保key适配客户端认账，创建一个扩展配置文件：


    $ echo extendedKeyUsage = clientAuth > extfile.cnf

现在生产key：

    $ openssl x509 -req -days 365-in client.csr -CA ca.pem -CAkey ca-key.pem \  -out cert.pem -extfile extfile.cnf
    
    Signature ok
    
    subject=/CN=clientGetting CA PrivateKey
    
    Enter pass phrase for ca-key.pem:

最后，你需要移除passphrase 来自客户端和服务端key：

    $ openssl rsa -in server-key.pem -out server-key.pem
    
    Enterpass phrase for server-key.pem:
    
    writing RSA key
    
    $ openssl rsa -in key.pem -out key.pem
    
    Enter pass phrase for key.pem:
    
    writing RSA key

现在，你可以确保Docker守护进程仅仅允许通过我们提供的CA认证来链接。

    $ docker -d --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem \  -H=0.0.0.0:2376

为了能够链接到Docker，证实它的认证，你现在需要提供你的客户端key，证书和信任的CA：

    $ docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem \  -H=dns-name-of-docker-host:2376 version

> 说明：Docker over TLS需要运行在TCP端口2376。

> 警告：就像示例中显示的那样，你不必运行docker 客户端带有sudo，甚至docker group，当你使用证书认证的时候。这就意味着，每个人只有有key，就可以得到任何你docker 守护进程的信息，让他们最高权限的操纵运行在主机上的守护进程。守护好这些key，正如一个root账户的密码。

## 保护 ##

如果你先想保护你的Docker客户端，你可以移除docker目录下的文件，在你的home目录下，并设置DOCKER_HOST和DOCKER_TLS_VERIFY变量。以此来替换通过-H=tcp://:2376和--tlsverify 每次。

    $ cp ca.pem ~/.docker/ca.pem
    
    $ cp cert.pem ~/.docker/cert.pem
    
    $ cp key.pem ~/.docker/key.pem
    
    $ export DOCKER_HOST=tcp://:2376
    
    $ export DOCKER_TLS_VERIFY=1

Docker默认情况将被认为是安全链接。

    $ docker ps

 

## 其它模式 ##

如果你不想有完整的两套认证，你可以运行Docker在各种其它模式下，通过最小化标志。

* Daemon modes
    
    tlsverify, tlscacert, tlscert, tlskey：认证的客户端
    
    tls, tlscert, tlskey:不需要认证的客户端

* Client modes

    tls：认证基于共有/默认 CA池的服务端。
    
    tlsverify, tlscacert：认证服务基于给定CA的服务端。
    
    tls, tlscert, tlskey：有客户端认证，不需要认证基于给定CA的服务端。
    
    tlsverify, tlscacert, tlscert, tlskey：认证客户端证书和给定CA的服务端。

如果被发现，可断将发送他的苦短端认证请求，你就需要把你的key放到~/.docker/<ca, cert or key>.pem。作为选择，如果你想存储你的key在另一个位置，你可以指定那个位置，使用环境变量DOCKER_CERT_PATH。

    $ export DOCKER_CERT_PATH=${HOME}/.docker/zone1/
    
    $ docker --tlsverify ps

链接到安全Docker端口，使用curl

为了使用curl让测试API通过，你需要使用三个额外的命令行标志：
    
    $ curl --insecure --cert ~/.docker/cert.pem --key ~/.docker/key.pem https://boot2docker:2376/images/json`