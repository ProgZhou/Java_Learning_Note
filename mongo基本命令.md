# mongo基本命令

**mongo数据库导出**

mongoexport --port 端口号 -d 数据库名称 -c 数据表名称 -o 备份文件的路径

```shell
mongoexport --port 27017 -d ai_platform_dev -c intention -o /data/zhezhou/ai_platform_dev_intention.json
```

**mongo数据库导入语句**

mongoimport --port 端口号 -d 数据库名称 -c 数据表名称 导入文件路径

```shell
mongoimport --port 27017 -d ai_platform_dev -c import_test /data/zhezhou/ai_platform_dev_intention.json
```

> 需要在mongo安装目录下的bin目录执行

**dump备份命令**

mongodump --port 端口号 -d 数据库名称 -c 集合名称 -o 备份文件路径

```shell
mongodump --host ip --port 27017 -d ai_platform_dev -c import_test /data/dump/mongodb/
```

**mongorestore恢复**

mongorestore -d 数据库名称 -c 集合名称 备份文件路径+集合名称.bson

```shell
./mongorestore -d ai_platform -c intention ../backup/ai_platform/intention.bson
```



现网mongo数据备份命令：

```
mongodump --host  "replica/10.247.15.173:8635,10.247.15.132:8635" -u rwuser -p 'M2+BY7r3BW9d' --authenticationDatabase=admin -d ai_llm_operation -c engine_resource_prewriting -o /data/backup/mongo/
```

