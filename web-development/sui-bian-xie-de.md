# 随便写的

### 1.整体思路

1. 首先引入依赖
2. 配置yml
3. 配置方言类
4. 配置实体
5. 自定义查询语句
6. 测试

### 2.差不多介绍

主要就是以下的依赖，其他的就是jpa什么的

```markup
<dependency>
   <groupId>org.postgresql</groupId>
   <artifactId>postgresql</artifactId>
</dependency>
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>fastjson</artifactId>
   <version>1.2.47</version>
</dependency>
```

yml文件数据源的配置和mysql差不多，区别如下，第一条指定方言，第二条兼容SpringBoot2.X

```yaml
database-platform: cn.luutqf.springboot.dialect.JsonbPostgresDialect
use_jdbc_metadata_defaults: false
```

JsonbPostgresDialect类主要就是下面这条，添加jsonb类型：

```java
registerColumnType(Types.JAVA_OBJECT, "jsonb");
```

JsonbType类用于Object和json的转换，详见代码

另有ListJsonConverter转换器，也可自定义类型

entity包中的实体详见代码，配置之后即可使用相关类型

PersonRepository中使用了原生SQL语句，用于jsonb查询

PersonController用于测试。

没写太多，基本操作就是这样。。。希望能给大家提供方便，少走弯路

