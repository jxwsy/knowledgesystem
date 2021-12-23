# presto jdbc连接测试

版本：0.266

--------------------------------------

1. 添加依赖

```xml
<dependency>
    <groupId>com.facebook.presto</groupId>
    <artifactId>presto-jdbc</artifactId>
    <version>0.266.1</version>
</dependency>
```

2. 连接

支持以下几种格式：

	# 可通过properties.setProperty配置catalog、schema
	jdbc:presto://host:port    
	jdbc:presto://host:port/catalog
	jdbc:presto://host:port/catalog/schema

host 为 coordinator 所在服务器名称

port 为 presto 目录下 `etc/config.properties` 中 `http-server.http.port` 的配置值

3. 连接示例

```java
import java.sql.*;
import java.util.Properties;

public class ConnTest {
    public static void main(String[] args) throws SQLException {

        String url = "jdbc:presto://bigdata101:8881/hive/default";
        Connection connection = DriverManager.getConnection(url, "root", null);
        Statement statement = connection.createStatement();

        ResultSet rs = statement.executeQuery("select * from order_table_s");
        while(rs.next()){
            String product_name = rs.getString("product_name");
            int price = rs.getInt("price");
            System.out.println("product_name: " + product_name + " price:" + price);
        }

        statement.close();
        connection.close();

    }
}

```

4. 连接参数含义

官网原文含完整的解释

5. 遇到的问题

在测试官网的连接参数示例时，出现 `Unsupported or unrecognized SSL message` 异常。

[这里](https://github.com/prestodb/presto/issues/8472)有详细解释。

如果没有配置 SSL and LDAP authentication，就可以把 `properties.setProperty("password", "secret");
` 删掉。


--------------------------------------

来自官网：[https://prestodb.io/docs/0.266/installation/jdbc.html](https://prestodb.io/docs/0.266/installation/jdbc.html)