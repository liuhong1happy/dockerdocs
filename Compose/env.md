# Compose环境变量

> 注意：环境变量不在是连接关联服务（linked service）的推荐方法。而应使用链接名称（默认是关联服务的名称）作为主讲名称去进行连接。详见docker-compose.yml documentation

组合使用Docker连接（Docker links）向另一个容器暴露服务所在的容器。每个关联的容器注入一系列以容器名称大写开头的环境变量。

使用docker-compose run SERVICE env可以查看那些环境变量可用。

name_PORT

完整的URL, 如DB_PORT=tcp://172.17.0.5:5432

name_PORT_num_protocol
完整的URL, 如DB_PORT_5432_TCP=tcp://172.17.0.5:5432

name_PORT_num_protocol_ADDR
容器 IP 如 DB_PORT_5432_TCP_ADDR=172.17.0.5

name_PORT_num_protocol_PORT
容器端口，如 DB_PORT_5432_TCP_PORT=5432

name_PORT_num_protocol_PROTO
协议(tcp 或 udp),如DB_PORT_5432_TCP_PROTO=tcp

name_NAME
完全有效容器名字 DB_1_NAME=/myapp_web_1/myapp_db_1