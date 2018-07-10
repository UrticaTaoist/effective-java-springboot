# PostgreSQL

PostgreSQL的话主要还是整理一下使用方法吧，毕竟我现在也只会使用。

PostgreSQL的使用方法与MySQL类似，  但更全面，实现了[SQL-2/SQL-92 和 SQL-3/SQL-99](https://baike.baidu.com/item/PostgreSQL/530240?fr=aladdin)标准。  
优点缺点就不提了，直接说说怎么用吧。

咱使用pg当然不会只把他当做MySQL来用，而我用它的目的就是jsonb。

0. 导入有用的包

```markup
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.0</version>
</dependency>
```

### 1.基本配置

```yaml
spring:
  datasource:
    url: jdbc:postgresql://118.89.146.227:5432/postgres
    username: postgres
    password: 123456
    driverClassName: org.postgresql.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    initialization-mode: always
  jpa:
    show-sql: true
    database-platform: cn...ledger.dialect.JsonbPostgresDialect
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        temp:
          use_jdbc_metadata_defaults: false
```

ps： use\_jdbc\_metadata\_defaults这个配置很重要，可以解决SpringBoot2.0版本的一个错误，具体机理我还没有研究，只是参考了一个博客做出的修改。

### 2. 配置方言

以上的配置与MySQL没什么差别，需要注意是方言的指定，我写的这个需要自己配置方言类：

```java
import org.hibernate.dialect.PostgreSQL94Dialect;
import org.hibernate.dialect.PostgreSQL95Dialect;
import org.hibernate.type.StringType;

import java.sql.Types;

/**
 * @Author ZhenYang
 * @Date Created in 2018/3/14 13:45
 * @Description
 */
public class JsonbPostgresDialect extends PostgreSQL95Dialect {


	public JsonbPostgresDialect() {
		super();
        registerColumnType(Types.JAVA_OBJECT, "jsonb");
        registerHibernateType( Types.ARRAY, StringType.class.getName());
	}
}
```

这里我指定了jsonb的类型，下面那个HibernateType是当时测试玩的。

### 3.配置jsonb类型

```java
import com.alibaba.fastjson.JSON;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.hibernate.HibernateException;
import org.hibernate.boot.registry.classloading.internal.ClassLoaderServiceImpl;
import org.hibernate.boot.registry.classloading.spi.ClassLoaderService;
import org.hibernate.engine.spi.SharedSessionContractImplementor;
import org.hibernate.type.SerializationException;
import org.hibernate.usertype.ParameterizedType;
import org.hibernate.usertype.UserType;
import org.postgresql.util.PGobject;
import org.springframework.util.ObjectUtils;

import java.io.IOException;
import java.io.Serializable;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Types;
import java.util.Map;
import java.util.Properties;

/**
 * @Author ZhenYang
 * @Date Created in 2018/3/14 13:46
 * @Description
 */
public class JsonbType implements UserType, ParameterizedType {

    private final ObjectMapper mapper = new ObjectMapper();
    ;

    private static final ClassLoaderService classLoaderService = new ClassLoaderServiceImpl();

    public static final String CLASS = "CLASS";

    private Class<?> jsonClassType;

    @Override
    public Object nullSafeGet(ResultSet resultSet, String[] strings, SharedSessionContractImplementor sharedSessionContractImplementor, Object o) throws HibernateException, SQLException {

        PGobject oo = (PGobject) resultSet.getObject(strings[0]);
        if (oo.getValue() != null) {
            System.out.println(jsonClassType.getClass());
            return JSON.parseObject(oo.getValue(), jsonClassType);
        }
        return null;
    }

    @Override
    public void nullSafeSet(PreparedStatement preparedStatement, Object o, int i, SharedSessionContractImplementor sharedSessionContractImplementor) throws HibernateException, SQLException {
        if (o == null) {
            preparedStatement.setNull(i, Types.OTHER);
        } else {
            try {
                preparedStatement.setObject(i, mapper.writeValueAsString(o), Types.OTHER);
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
//    @Override
//    public void nullSafeSet(PreparedStatement st, Object value, int index, SessionImplementor session)
//            throws HibernateException, SQLException {
//        if (value == null) {
//            st.setNull(index, Types.OTHER);
//        } else {
//            try {
//                st.setObject(index, mapper.writeValueAsString(value), Types.OTHER);
//            } catch (IOException e) {
//                e.printStackTrace();
//            }
//
//        }
//    }
//
//    /**
//     * 重写自定义类型，Jsonb
//     * @param rs
//     * @param names
//     * @param session
//     * @param owner
//     * @return
//     * @throws HibernateException
//     * @throws SQLException
//     */
//    @Override
//    public Object nullSafeGet(ResultSet rs, String[] names, SessionImplementor session, Object owner)
//            throws HibernateException, SQLException {
//        PGobject o = (PGobject) rs.getObject(names[0]);
//        if (o.getValue() != null) {
//            System.out.println(jsonClassType.getClass());
//            return JSON.parseObject(o.getValue(),jsonClassType);
//        }
//        return null;
//    }

    @Override
    public Object deepCopy(Object originalValue) throws HibernateException {
        if (originalValue != null) {
            try {
                System.out.println(mapper.readValue(mapper.writeValueAsString(originalValue), returnedClass()));
                return mapper.readValue(mapper.writeValueAsString(originalValue), returnedClass());
            } catch (IOException e) {
                throw new HibernateException("Failed to deep copy object", e);
            }
        }
        return null;
    }


    @Override
    public Serializable disassemble(Object value) throws HibernateException {
        Object copy = deepCopy(value);

        if (copy instanceof Serializable) {
            return (Serializable) copy;
        }

        throw new SerializationException(
                String.format("Cannot serialize '%s', %s is not Serializable.", value, value.getClass()), null);
    }

    @Override
    public Object assemble(Serializable cached, Object owner) throws HibernateException {
        return deepCopy(cached);
    }

    @Override
    public Object replace(Object original, Object target, Object owner) throws HibernateException {
        return deepCopy(original);
    }

    @Override
    public boolean isMutable() {
        return true;
    }

    @Override
    public int hashCode(Object x) throws HibernateException {
        if (x == null) {
            return 0;
        }

        return x.hashCode();
    }


    @Override
    public boolean equals(Object x, Object y) throws HibernateException {
        return ObjectUtils.nullSafeEquals(x, y);
    }

    @Override
    public Class<?> returnedClass() {
        return Map.class;
    }

    @Override
    public int[] sqlTypes() {
        return new int[]{Types.JAVA_OBJECT};
    }

    @Override
    public void setParameterValues(Properties parameters) {
        final String clazz = (String) parameters.get(CLASS);
        jsonClassType = classLoaderService.classForName(clazz);

    }

}
```

这一段是为了自定义了JsonB的处理类，基本的逻辑就是将从数据库中返回的Json串转化为对象。  
注释部分是因为SpringBoot1.X和2.X不兼容，导致部分函数需要修改。

### 4.配置list类型

```java
import com.fasterxml.jackson.core.JsonParseException;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.ObjectMapper;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;
import java.io.IOException;
import java.util.List;

/**
 * @Author : ZhenYang
 * @Despriction :
 * @Date: Created in 2018/6/30 13:47
 * @Modify By:
 */
@Converter
public class ListString2JsonConverter implements AttributeConverter<List<String>, String> {
    @Override
    public String convertToDatabaseColumn(List<String> attribute) {
        if(attribute == null || attribute.size() == 0) return "[]";
        ObjectMapper mapper = new ObjectMapper();
        String json = null;
        try {
            json = mapper.writeValueAsString(attribute);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return json;
    }

    @SuppressWarnings("unchecked")
    @Override
    public List<String> convertToEntityAttribute(String dbData) {
        ObjectMapper mapper = new ObjectMapper();
        List<String> list = null;
        try {
            list = (List<String>) mapper.readValue(dbData, List.class);
        } catch (JsonParseException e) {
            e.printStackTrace();
        } catch (JsonMappingException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return list;
    }
}
```

这个类自定义了list类型。

### 5.jsonb在实体类中的应用



```java
import cn.videon.diconde.common.constant.GeneralConstants;
import cn.videon.diconde.ledger.dialect.JsonbType;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.Type;
import org.hibernate.annotations.TypeDef;
import org.hibernate.annotations.TypeDefs;

import javax.persistence.*;

/**
 * @Author : ZhenYang
 * @Despriction : 元件
 * @Date: Created in 2018/6/25 13:48
 * @Modify By:
 */
@Entity
@Data
@EqualsAndHashCode(callSuper=false)
@AllArgsConstructor
@NoArgsConstructor
@Table(name = "component", schema = GeneralConstants.ledgerSchema)
@TypeDefs({
        @TypeDef(name = "JsonbTypeGeometry", typeClass = JsonbType.class, parameters = {
                @org.hibernate.annotations.Parameter(name = JsonbType.CLASS, value = "cn...ledger.entity.pg.NDEGeometry")}),
})
public class Component extends BaseEntity {

    //略

    
    @Column(columnDefinition = "jsonb")
    @Type(type = "JsonbTypeGeometry")
    private NDEGeometry geometry;

    @ManyToOne(targetEntity = NDEMaterial.class,cascade=CascadeType.ALL)
    @JoinColumn(name = "material_id", referencedColumnName = "id")
    private NDEMaterial material;
}
```

```java
import cn.videon.diconde.common.constant.GeneralConstants;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Table;

/**
 * @Author : ZhenYang
 * @Despriction : 几何结构
 * @Date: Created in 2018/6/25 14:05
 * @Modify By:
 */
//@Entity
//@Table(name = "geometry", schema = GeneralConstants.ledgerSchema)
@Data
@NoArgsConstructor
@AllArgsConstructor
public class NDEGeometry{

    //直径
    private Double diameter;

    //厚度
    private Double thickness;

    //曲率类型
//    @Column(name = "curvature_type")
    private String curvatureType;

    //形状概述
//    @Column(name = "shape_desc")
    private String shapeDesc;

}
```

我拿出来一个相对典型的实体来看看

1. 使用@TypeDefs声明JsonBType并指定具体的model。
2. NDEGeometry字段使用@Column和 @Type指定自己的类型。
3. 相关的sql操作就自行搜索吧，具体的操作也有一些坑，要多查查。

### 6.list在实体类中的应用

```java
@Convert(converter = ListString2JsonConverter.class)
private List<String> cite;
```

这里使用了@Convert来转化list，配合上面自定义的ListString2JsonConverter，就可以用了，至于为什么，暂时还不知道，网上查的。

### 7.一些SQL操作

刚才说自行搜索是因为这个项目里没写这些，但想了想还是找一下吧，当做记录了，随便找来一个充个数。

```java
public interface BookRepository extends JpaRepository<BookEntity, UUID> {

    @Query("SELECT DISTINCT b.publisher FROM BookEntity b WHERE UPPER(b.publisher) LIKE UPPER(CONCAT('%', ?1, '%'))")
    List<String> findDistinctPublisher(String publisher);

    @Query(value = "SELECT * FROM example_jsonb.books b WHERE b.author->>'firstName' ILIKE CONCAT('%', ?1, '%')", nativeQuery = true)
    List<BookEntity> findByAuthorFirstName(String firstName);

    @Query(value = "SELECT * FROM example_jsonb.books b WHERE b.author->>'lastName' ILIKE CONCAT('%', ?1, '%')", nativeQuery = true)
    List<BookEntity> findByAuthorLastName(String lastName);

    @Query(value = "SELECT * FROM example_jsonb.books b WHERE b.author->>'firstName' ILIKE CONCAT('%', ?1, '%') AND  b.author->>'lastName' ILIKE CONCAT('%', ?2, '%')", nativeQuery = true)
    List<BookEntity> findByAuthorFirstNameAndLastName(String firstName,
            String lastName);
}
```

以上这个是针对jsonb的查询操作，和mysql是很不一样的，所以要使用@Query，也不清楚有没有更方便的方法，至少是能用。

之前也试过查询数据库中定义的枚举，但失败了，一直也没再试试。

