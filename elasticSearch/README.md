1. 执行 `docker-compose up` 启动 docker-compose.yml 文件
2. 稍微等一会, 检测健康程度:
```
curl http://127.0.0.1:9200/_cat/health
1551949594 09:06:34 docker-cluster green 2 2 0 0 0 0 0 0 - 100.0%
```


[文档原文](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)