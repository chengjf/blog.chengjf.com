+++
title = 'MapStruct使用之基础'
date = 2023-08-10T13:38:36+08:00
draft = false
comment = true
tags = ["Java", "MapStruct"]
categories = "编程技术"
+++

## 基础映射

### 创建需要映射的POJO
咱们使用两个基础对象做示范。

注意：代码中使用了lombok
```java
@Data  
@Builder  
public class X {  
    private Long id;  
    private String name;  
    private String description;  
}
@Data  
@Builder  
public class Y {  
    private Long id;  
    private String name;  
    private String description;  
}
```
### 编写映射接口
```java
@Mapper  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    Y source2Destination(X source);  
  
}
```
### 生成的接口实现
```java
public class XYMapperImpl implements XYMapper {  
  
    @Override  
    public Y source2Destination(X source) {  
        if ( source == null ) {  
            return null;  
        }  
  
        Y.YBuilder y = Y.builder();  
  
        y.id( source.getId() );  
        y.name( source.getName() );  
        y.description( source.getDescription() );  
  
        return y.build();  
    }  
}
```
### 测试验证
```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().id(1L).name("哈利路亚").description("哈哈").build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(id=1, name=哈利路亚, description=哈哈)
Y(id=1, name=哈利路亚, description=哈哈)
```

## 字段名不同的映射

### 创建需要映射的POJO
```java
@Data  
@Builder  
public class X {  
    private Long id;  
    private String name;  
    private String description;  
}
@Data  
@Builder  
public class Y {  
    private Long targetId;  
    private String placeName;  
    private String desc;  
}
```
### 编写映射接口
使用 @Mapping 注解配置映射的字段名称。

```java
@Mapper  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(source = "id", target = "targetId")  
    @Mapping(source = "name", target = "placeName")  
    @Mapping(source = "description", target = "desc")  
    Y source2Destination(X source);  
  
}
```
### 生成的接口实现
```java
public class XYMapperImpl implements XYMapper {

    @Override
    public Y source2Destination(X source) {
        if ( source == null ) {
            return null;
        }

        Y.YBuilder y = Y.builder();

        y.targetId( source.getId() );
        y.placeName( source.getName() );
        y.desc( source.getDescription() );

        return y.build();
    }
}
```
### 测试验证
```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().id(1L).name("哈利路亚").description("哈哈").build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(id=1, name=哈利路亚, description=哈哈)
Y(targetId=1, placeName=哈利路亚, desc=哈哈)
```

## 子对象的映射
### 创建需要映射的POJO
```java
@Data  
@Builder  
public class Foo {  
    private String fooName;  
}
@Data  
@Builder  
public class Bar {  
    private String barName;  
}
@Data  
@Builder  
public class X {  
    private Long id;  
    private String name;  
    private String description;  
  
    private Foo foo;  
}
@Data  
@Builder  
public class Y {  
    private Long targetId;  
    private String placeName;  
    private String desc;  
  
    private Bar bar;  
}
```
### 编写映射接口
```java
@Mapper  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(source = "id", target = "targetId")  
    @Mapping(source = "name", target = "placeName")  
    @Mapping(source = "description", target = "desc")  
    @Mapping(source = "foo", target = "bar")  
    Y source2Destination(X source);  
  
    @Mapping(source = "fooName", target = "barName")  
    Bar foo2Bar(Foo foo);  
}
```
### 生成的接口实现
```java
public class XYMapperImpl implements XYMapper {  
  
    @Override  
    public Y source2Destination(X source) {  
        if ( source == null ) {  
            return null;  
        }  
  
        Y.YBuilder y = Y.builder();  
  
        y.targetId( source.getId() );  
        y.placeName( source.getName() );  
        y.desc( source.getDescription() );  
        y.bar( foo2Bar( source.getFoo() ) );  
  
        return y.build();  
    }  
  
    @Override  
    public Bar foo2Bar(Foo foo) {  
        if ( foo == null ) {  
            return null;  
        }  
  
        Bar.BarBuilder bar = Bar.builder();  
  
        bar.barName( foo.getFooName() );  
  
        return bar.build();  
    }  
}
```
### 验证测试
```java
public class Test {  
  
    public static void main(String[] args) {  
        Foo foo = Foo.builder().fooName("FOO").build();  
        X x = X.builder().id(1L).name("哈利路亚").description("哈哈").foo(foo).build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(id=1, name=哈利路亚, description=哈哈, foo=Foo(fooName=FOO))
Y(targetId=1, placeName=哈利路亚, desc=哈哈, bar=Bar(barName=FOO))
```

## 类型转换的映射
### 创建需要映射的POJO
```java
@Data
@Builder
public class X {
    private Long id;
    private String name;
    private String description;

    private LocalDateTime localDateTime;
    private Double price;
}
@Data  
@Builder  
public class Y {  
    private Long targetId;  
    private String placeName;  
    private String desc;  
  
    private String time;  
    private String price;  
}
```
### 编写映射接口
```java
@Mapper  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(source = "id", target = "targetId")  
    @Mapping(source = "name", target = "placeName")  
    @Mapping(source = "description", target = "desc")  
    @Mapping(source = "localDateTime", target = "time", dateFormat = "yyyy-MM-dd HH:mm:ss")  
    @Mapping(source = "price", target = "price", numberFormat = "¥#.00")  
    Y source2Destination(X source);  
  
}
```

### 生成的接口实现
```java
public class XYMapperImpl implements XYMapper {  
  
    private final DateTimeFormatter dateTimeFormatter_yyyy_MM_dd_HH_mm_ss_11333195168 = DateTimeFormatter.ofPattern( "yyyy-MM-dd HH:mm:ss" );  
  
    @Override  
    public Y source2Destination(X source) {  
        if ( source == null ) {  
            return null;  
        }  
  
        Y.YBuilder y = Y.builder();  
  
        y.targetId( source.getId() );  
        y.placeName( source.getName() );  
        y.desc( source.getDescription() );  
        if ( source.getLocalDateTime() != null ) {  
            y.time( dateTimeFormatter_yyyy_MM_dd_HH_mm_ss_11333195168.format( source.getLocalDateTime() ) );  
        }  
        if ( source.getPrice() != null ) {  
            y.price( new DecimalFormat( "¥#.00" ).format( source.getPrice() ) );  
        }  
  
        return y.build();  
    }  
}
```
### 验证测试
```java
public class Test {  
  
    public static void main(String[] args) {  
        Foo foo = Foo.builder().fooName("FOO").build();  
        X x = X.builder().id(1L).name("哈利路亚").description("哈哈")  
            .localDateTime(LocalDateTime.now())  
            .price(34531D)  
            .build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(id=1, name=哈利路亚, description=哈哈, localDateTime=2023-08-10T10:19:02.003, price=34531.0)
Y(targetId=1, placeName=哈利路亚, desc=哈哈, time=2023-08-10 10:19:02, price=¥34531.00)
```

## 前置映射和后置映射
使用 @BeforeMapping 进行前置映射，使用 @AfterMapping 进行后置映射

### 创建需要映射的POJO
```java
@Data  
@Builder  
public class X {  
    private String firstName;  
    private String lastName;  
}
@Data  
@Builder  
public class Y {  
    private String firstName;  
    private String lastName;  
    private String fullName;  
}
```
### 编写映射接口

注意：**@Mapper** 注解同lombok的 **@Builder** 使用时会出现问题，可以按照如下代码中的配置解决

```java
@Mapper(builder = @Builder(disableBuilder = true))  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    Y source2Destination(X x);  
  
    @BeforeMapping  
    default void before(X x) {  
        x.setFirstName(x.getFirstName().toUpperCase());  
        x.setLastName(x.getLastName().toUpperCase());  
    }  
  
    @AfterMapping  
    default void after(@MappingTarget Y y) {  
        y.setFullName(y.getFirstName() + " " + y.getLastName());  
    }  
  
}
```
### 生成的接口实现
```java
public class XYMapperImpl implements XYMapper {  
  
    @Override  
    public Y source2Destination(X x) {  
        before( x );  
  
        if ( x == null ) {  
            return null;  
        }  
  
        String firstName = null;  
        String lastName = null;  
  
        firstName = x.getFirstName();  
        lastName = x.getLastName();  
  
        String fullName = null;  
  
        Y y = new Y( firstName, lastName, fullName );  
  
        after( y );  
  
        return y;  
    }  
}
```
### 验证测试
```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().firstName("gg").lastName("bond").build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(firstName=gg, lastName=bond)
Y(firstName=GG, lastName=BOND, fullName=GG BOND)
```


## 表达式映射
### 创建需要映射的POJO
```java
@Data  
@Builder  
public class X {  
    private String firstName;  
    private String lastName;  
}
@Data  
@Builder  
public class Y {  
    private String firstName;  
    private String lastName;  
    private String uuid;  
}
```

### 编写映射接口
```java
@Mapper(builder = @Builder(disableBuilder = true))  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(target = "uuid", expression = "java(java.util.UUID.randomUUID().toString())")  
    Y source2Destination(X x);  
}
```
### 生成的接口实现
```java
public class XYMapperImpl implements XYMapper {  
  
    @Override  
    public Y source2Destination(X x) {  
        if ( x == null ) {  
            return null;  
        }  
  
        String firstName = null;  
        String lastName = null;  
  
        firstName = x.getFirstName();  
        lastName = x.getLastName();  
  
        String uuid = java.util.UUID.randomUUID().toString();  
  
        Y y = new Y( firstName, lastName, uuid );  
  
        return y;  
    }  
}
```
### 验证测试
```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().firstName("gg").lastName("bond").build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(firstName=gg, lastName=bond)
Y(firstName=gg, lastName=bond, uuid=3ef6875a-85ce-4d8c-807d-2c0d4193cfd4)
```