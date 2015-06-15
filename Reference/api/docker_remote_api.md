# Docker Remote API

* 默认情况下，Docker daemon监听unix:///var/run/docker.sock，客户端必须有权限和这个daemon进行交互。

* 如果在Boot2Docker 1.3.0上docker deamon 被设置为使用加密的TCP socket链接（--tls或者--tlsverify），那么你需要添加额外的参数给curl或者wget，测试API要求：

    curl --insecure --cert ~/.docker/cert.pem --key ~/.docker/key.pem https://boot2docker:2376/images/json or wget --no-check-certificate --certificate=$DOCKER_CERT_PATH/cert.pem --private-key=$DOCKER_CERT_PATH/key.pem https://boot2docker:2376/images/json -O - -q

* 如果名为docker的group存在你的系统里，docker将应用socket的所有权来链接这个group。

* API扩展为REST，但是某些复杂的指令，例如attach，pull；HTTP链接是被抹去了的，不能执行STDOUT,STDIN和STDERR。

* 自从API 1.2，权限配置是可以在客户端管控的，因此客户端不得不发送authConfig作为POST对象到/images/(name)/push里。

* authConfig，设置了X-Registry-Auth头，当前是一个Base64编码的JSON字符串，使用如下的结构：

        {"username": "string", "password": "string", "email": "string", "serveraddress" : "string", "auth": ""}

注意auth左边的值不能为空，serveraddress一个不带协议的domain/ip，这两个引用是被要求的。

* Remote API使用一个开源的 schema model，在这个model，不知道属性会被忽略。客户端应用需要把这个考虑进账户，确保他们不能被挡，在新的Docker daemon通话的时候。

当前API的版本是1.18

调用`/info`等同于调用`/v1.18/info`

你可以一直调用一个老版本的API，使用`/v1.17/info`。

如果想了解最新API，请导航到：

[Docker Remote API 1.18](docker_remote_api_v1.18.md)

或者你想导航到较老的API：

[Docker Remote API 1.17](docker_remote_api_v1.17.md)

[Docker Remote API 1.16](docker_remote_api_v1.16.md)