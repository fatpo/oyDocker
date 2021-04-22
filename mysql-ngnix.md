cat docker-compose.yml
```
version: '3'
services:
    nginx:
        image: nginx:latest
        # 端口映射
        ports:
            - "80:80"
        # 依赖关系 先跑mysql
        depends_on:
            - "mysql"
        # 数据卷
        volumes:
            # 映射主机./conf.d目录到容器/etc/nginx/conf.d目录
            - "$PWD/conf.d:/etc/nginx/conf.d"
            - "$PWD/html:/usr/share/nginx/html"
        networks:
            - app_net
        # 容器名称
        container_name: "compose-nginx"
    mysql:
        image: mysql:5.7
        ports:
            - "3306:3306"
        # 环境变量
        environment:
            TZ: Asia/Shanghai
            # mysql密码
            MYSQL_ROOT_PASSWORD: 123456
        command:
          --character-set-server=utf8mb4
          --collation-server=utf8mb4_general_ci
          --explicit_defaults_for_timestamp=true
          --lower_case_table_names=1
          --max_allowed_packet=128M
          --sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO"
        volumes:
            - "$PWD/mysql:/var/lib/mysql"
        networks:
            app_net:
                # 固定子网ip，网段必须在子网络10.10.*.*
                ipv4_address: 10.10.10.1
        container_name: "compose-mysql"
networks:
    # 配置docker network
    app_net:
        driver: bridge
        ipam:
            config:
                # 子网络
                - subnet: 10.10.0.0/16
```