# presto on spark测试

spark3.2.0

presto-0.266

```sh
spark-submit \
--master spark://zgg:7077 \
--class com.facebook.presto.spark.launcher.PrestoSparkLauncher \
  /opt/spark-3.2.0/examples/presto-spark-launcher-0.266.jar \
--package /opt/spark-3.2.0/examples/presto-spark-package-0.266.tar.gz \
--config etc/config.properties \
--catalogs etc/catalog \
--catalog hive \
--schema default \
--file /root/query.sql
```

报如下错误：

```
21/12/23 10:55:28 WARN Bootstrap: UNUSED PROPERTIES
21/12/23 10:55:28 WARN Bootstrap: discovery-server.enabled
21/12/23 10:55:28 WARN Bootstrap: discovery.uri
21/12/23 10:55:28 WARN Bootstrap: http-server.http.port
21/12/23 10:55:28 WARN Bootstrap: 
com.google.inject.CreationException: Unable to create injector, see the following errors:

1) Configuration property 'discovery-server.enabled' was not used
  at com.facebook.airlift.bootstrap.Bootstrap.lambda$initialize$2(Bootstrap.java:244)

2) Configuration property 'discovery.uri' was not used
  at com.facebook.airlift.bootstrap.Bootstrap.lambda$initialize$2(Bootstrap.java:244)

3) Configuration property 'http-server.http.port' was not used
  at com.facebook.airlift.bootstrap.Bootstrap.lambda$initialize$2(Bootstrap.java:244)

3 errors
```

版本兼容问题？待测试？