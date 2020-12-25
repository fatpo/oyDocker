```
docker network create --subnet 172.30.1.0/16 --gateway 172.30.0.1 kafka
```
# 背景
kafka 重度依赖 zookeeper

两个集群应该在一个网络，先开一个网络：
```dtd
docker network create --subnet 172.19.1.0/16 --gateway 172.19.0.1 kafka
```
然后两个 docker-compose.yml 都指定同一个网络：
```dtd
networks:
  kafka:
    external:
      name: kafka
```

# 单独安装 zookeeper 和 kafka
安装 zookeeper：
```dtd
cd /Users/apple/dockerdata/zookeeper_home
```
cat docker-compose.yml：
```dtd
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    privileged: true
    hostname: zoo1
    ports:
      - 2181:2181
    volumes: # 挂载数据
      - /Users/apple/dockerdata/zookeeper_home/node1/data:/data
      - /Users/apple/dockerdata/zookeeper_home/node1/datalog:/datalog
      - /Users/apple/dockerdata/zookeeper_home/node1/logs:/logs
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    networks:
      default:
        ipv4_address: 172.19.0.11

  zoo2:
    image: zookeeper
    restart: always
    privileged: true
    hostname: zoo2
    ports:
      - 2182:2181
    volumes: # 挂载数据
      - /Users/apple/dockerdata/zookeeper_home/node2/data:/data
      - /Users/apple/dockerdata/zookeeper_home/node2/datalog:/datalog
      - /Users/apple/dockerdata/zookeeper_home/node2/logs:/logs
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181
    networks:
      default:
        ipv4_address: 172.19.0.12

  zoo3:
    image: zookeeper
    restart: always
    privileged: true
    hostname: zoo3
    ports:
      - 2183:2181
    volumes: # 挂载数据
      - /Users/apple/dockerdata/zookeeper_home/node3/data:/data
      - /Users/apple/dockerdata/zookeeper_home/node3/datalog:/datalog
      - /Users/apple/dockerdata/zookeeper_home/node3/logs:/logs
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
    networks:
      default:
        ipv4_address: 172.19.0.13

networks: # 自定义网络
  default:
    external:
      name: kafka
```

安装 kafka：
```dtd
cd /Users/apple/dockerdata/kafka_home
```
cat docker-compose.yml：
```dtd
cd /Users/apple/dockerdata/kafka_home
version: '3.4'
services:
  kafka1:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka1
    container_name: kafka1
    privileged: true
    ports:
      - 9092:9092
    environment:
      KAFKA_ADVERTISED_HOST_NAME: kafka1
      KAFKA_LISTENERS: PLAINTEXT://kafka1:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
      - /Users/apple/dockerdata/kafka_home/kafka1/logs:/kafka
    networks:
      kafka:
        ipv4_address: 172.19.1.11
    extra_hosts:
      zoo1: 172.19.0.11
      zoo2: 172.19.0.12
      zoo3: 172.19.0.13
  kafka2:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka2
    container_name: kafka2
    privileged: true
    ports:
      - 9093:9093
    environment:
      KAFKA_ADVERTISED_HOST_NAME: kafka2
      KAFKA_LISTENERS: PLAINTEXT://kafka2:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9093
      KAFKA_ADVERTISED_PORT: 9093
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
      - /Users/apple/dockerdata/kafka_home/kafka2/logs:/kafka
    networks:
      kafka:
        ipv4_address: 172.19.1.12
    extra_hosts:
      zoo1: 172.19.0.11
      zoo2: 172.19.0.12
      zoo3: 172.19.0.13
  kafka3:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka3
    container_name: kafka3
    privileged: true
    ports:
      - 9094:9094
    environment:
      KAFKA_ADVERTISED_HOST_NAME: kafka3
      KAFKA_LISTENERS: PLAINTEXT://kafka3:9094
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9094
      KAFKA_ADVERTISED_PORT: 9094
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
      - /Users/apple/dockerdata/kafka_home/kafka3/logs:/kafka
    networks:
      kafka:
        ipv4_address: 172.19.1.13
    extra_hosts:
      zoo1: 172.19.0.11
      zoo2: 172.19.0.12
      zoo3: 172.19.0.13
networks:
  kafka:
    external:
      name: kafka
```
