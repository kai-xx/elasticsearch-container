# elasticsearch-container

##docker镜像地址
####elasticsearch
```
docker.elastic.co/elasticsearch/elasticsearch:7.1.1@sha256:1084c64eed7d9318d028361c9aee398afdeb70d1816ce81d590b9450ec542c08
```
####elasticsearch-head
```
tobias74/elasticsearch-head:6
```
##### 注：如果镜像不是最新，请自行去hub.docker.com中查找

####目录结构
```
├── docker-compose.yml
├── es1
│   ├── data
│   └── elasticsearch.yml
├── es2
│   ├── data
│   └── elasticsearch.yml
└── head
    ├── Gruntfile.js
    └── _site
        └── app.js
```
####docker-compose.yml 配置
```dockerfile
version: '3'
services:
  es1:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.1@sha256:1084c64eed7d9318d028361c9aee398afdeb70d1816ce81d590b9450ec542c08
    container_name: es1
    environment:
      - bootstrap.memory_lock=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    cap_add:
      - IPC_LOCK

    volumes:
      - /home/elaticsearch-container/es1/data:/usr/share/elasticsearch/data
      - /home/elaticsearch-container/es1/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 9201:9201
    restart: always
    networks:
      esnet:
        ipv4_address: 192.167.0.101
  es2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.1@sha256:1084c64eed7d9318d028361c9aee398afdeb70d1816ce81d590b9450ec542c08
    container_name: es2
    environment:
      - bootstrap.memory_lock=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    cap_add:
      - IPC_LOCK
    volumes:
      - /home/elaticsearch-container/es2/data:/usr/share/elasticsearch/data
      - /home/elaticsearch-container/es2/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 9202:9202
    restart: always
    networks:
      esnet:
        ipv4_address: 192.167.0.102
  elasticsearch-head:
    image: tobias74/elasticsearch-head:6
    container_name: es-head
    volumes:
       - /home/elaticsearch-container/head/Gruntfile.js:/usr/src/app/Gruntfile.js
       - /home/elaticsearch-container/head/_site/app.js:/usr/src/app/_site/app.js
    ports:
       - "9100:9100"
    restart: always
    networks:
      esnet:
        ipv4_address: 192.167.0.103


networks:
  esnet:
    driver: bridge
    ipam:
      config:
      - subnet: 192.167.0.0/24
```
#### elasticsearch集群配置
```
es1下elasticsearch配置-- master
elasticsearch.yml
node.name: kare-1
cluster.initial_master_nodes: ["kare-1"]
xpack.ml.enabled: false
network.host: 0.0.0.0
http.port: 9201
#memory
#bootstrap.memory_lock: false
bootstrap.system_call_filter: false

http.cors.enabled: true
http.cors.allow-origin: "*"

#集群
cluster.name: kare
node.master: true


es2下elasticsearch配置-- slaver
elasticsearch.yml
node.name: kare-2
xpack.ml.enabled: false
network.host: 0.0.0.0
http.port: 9202
#memory
#bootstrap.memory_lock: false
bootstrap.system_call_filter: false

http.cors.enabled: true
http.cors.allow-origin: "*"

#集群
cluster.name: kare
node.master: false

#自动发现
discovery.zen.ping.unicast.hosts: ['192.167.0.101']
```
####elasticsearch-head中两个文件配置：
######Gruntfile.js
```
connect: {
      server: {
            options: {
                #替换为自己的端口号
                 port: 9100,
                 base: '.',
                 keepalive: true
            }
      }
}
```
######_site/app.js
```
#ip:端口号  根据自己实际情况填写
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://39.96.77.174:9201";
```
