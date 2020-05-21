# 数据迁移

- 安装 nodejs

```bash
[root@iz8vb1tcxx7j3u2weoxp5ez usr]# yum -y install nodejs
```

- 安装 elasticdump

```bash
[root@iz8vb1tcxx7j3u2weoxp5ez usr]# npm install elasticdump -g
```
- 导出analyzer信息

```bash
[root@iz8vb1tcxx7j3u2weoxp5ez usr]# elasticdump \
   --input=http://localhost:9200/blog \
   --output=http://182.92.103.188:9200/blog \
   --type=analyzer
```

- 导出mapping信息

```bash
[root@iz8vb1tcxx7j3u2weoxp5ez usr]# elasticdump \
   --input=http://localhost:9200/blog \
   --output=http://182.92.103.188:9200/blog \
   --type=mapping
```

- 导出data信息

```bash
[root@iz8vb1tcxx7j3u2weoxp5ez ik]# elasticdump \
   --input=http://localhost:9200/blog \
   --output=http://182.92.103.188:9200/blog \
   --type=data
```

# Docker部署

```yaml
version: "2"
services:
  ################################### elasticsearch全文搜索引擎
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.5.1
    hostname: elasticsearch
    networks:
      - default-net
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/logs:/usr/share/elasticsearch/logs
    mem_limit: 1g
networks:
  default-net:
    driver: bridge
```

