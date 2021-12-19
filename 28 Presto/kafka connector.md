# kafka connector

版本：0.266

--------------------------------------

允许 presto 访问 kafka topic 中的数据，包括实时产生的数据（live topic data）。

topic 中的每条信息就是 presto 中的每行数据。

从 kafka topic 读取到的数据需要经过加工处理（通过使用topic description file）后，才能展示出像常规的表的一样的数据。

在 [**官网**](https://prestodb.io/docs/current/connector/kafka-tutorial.html#connector-kafka-tutorial--page-root) 提供了详细完整的演示实例。

--------------------------------------

来自官网：[https://prestodb.io/docs/current/connector/kafka.html](https://prestodb.io/docs/current/connector/kafka.html)