创建网络：
```
docker network create --driver bridge --subnet=172.19.0.0/16 --gateway=172.19.0.1 zoonet
```
创建挂载目录：
```
mkdir -p /Users/apple/dockerdata/zookeeper_home/node1
mkdir -p /Users/apple/dockerdata/zookeeper_home/node2
mkdir -p /Users/apple/dockerdata/zookeeper_home/node3
```
创建编排配置： `cat docker-compose.yml`
```
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
      name: zoonet
```
启动：
```
➜> docker-compose -f docker-compose.yml up -d
Creating zookeeper_zoo3_1 ... done
Creating zookeeper_zoo1_1 ... done
Creating zookeeper_zoo2_1 ... done
```
创一个zkClient来连接这个zookeeper集群：
```
host> docker run -it -d --name  zkclient zookeeper
host>docker exec -it 2d523db953ea bash

docker> ./zkCli.sh -server 172.19.0.11:2181,172.19.0.12:2181,172.19.0.13:2183
2020-10-26 03:03:42,941 [myid:] - INFO  [main:ClientCnxn@1716] - zookeeper.request.timeout value is 0. feature enabled=false
Welcome to ZooKeeper!
JLine support is enabled
2020-10-26 03:03:43,042 [myid:172.19.0.11:2181] - INFO  [main-SendThread(172.19.0.11:2181):ClientCnxn$SendThread@1167] - Opening socket connection to server 172.19.0.11/172.19.0.11:2181.
2020-10-26 03:03:43,045 [myid:172.19.0.11:2181] - INFO  [main-SendThread(172.19.0.11:2181):ClientCnxn$SendThread@1169] - SASL config status: Will not attempt to authenticate using SASL (unknown error)
docker> [zk: 172.19.0.11:2181,172.19.0.12:2181,172.19.0.13:2181(CONNECTING) 0]
```
看看集群里面每个节点服务状态：
节点1：follower
```
➜  ~ docker exec -it zookeeper_zoo1_1 bash
root@zoo1:/apache-zookeeper-3.6.2-bin/bin# ./zkServer.sh status
root@zoo1:/apache-zookeeper-3.6.2-bin# cd bin
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
root@zoo1:/apache-zoo
```
节点2：follower
```
➜  ~ docker exec -it zookeeper_zoo2_1 bash
root@zoo2:/apache-zookeeper-3.6.2-bin# cd bin
root@zoo2:/apache-zookeeper-3.6.2-bin/bin# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
```
节点3: leader
```
➜  ~ docker exec -it zookeeper_zoo3_1 bash
root@zoo3:/apache-zookeeper-3.6.2-bin# cd bin
root@zoo3:/apache-zookeeper-3.6.2-bin/bin# ./zkServer.sh  status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader
```

暂停所有容器：
```
➜  zookeeper docker-compose -f docker-compose.yml stop
Stopping zookeeper_zoo1_1 ... done
Stopping zookeeper_zoo3_1 ... done
Stopping zookeeper_zoo2_1 ... done
```
删除所有容器：
```
➜  zookeeper docker-compose -f docker-compose.yml rm
Going to remove zookeeper_zoo1_1, zookeeper_zoo3_1, zookeeper_zoo2_1
Are you sure? [yN] y
Removing zookeeper_zoo1_1 ... done
Removing zookeeper_zoo3_1 ... done
Removing zookeeper_zoo2_1 ... done
```