+++
title = 'MapStruct使用之万能自定义方法处理'
date = 2023-08-10T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "MapStruct"]
categories = "编程技术"
+++

上面的情况都是比较常见、简单的情况，还有更复杂的情况。
比如当字段是null时，可以使用上面的方法处理，但是字段类型String值为""时，那么该字段不是null就无法适用了。
亦或者，当字段为0或者某些特定值时，需要进行特殊的默认值处理。

上面这些特殊的情况，都可以进行自定义处理。

## 使用 @BeforeMapping 和 @AfterMapping 进行处理
### 创建的POJO如下：
```java
@Data  
@Builder  
public class X {  
    private String firstName;  
    private String lastName;  
    private String fullName;  
}
@Data  
@Builder  
public class Y {  
    private String firstName;  
    private String lastName;  
    private String fullName;  
}
```


### 映射接口如下：

```java
@Mapper(builder = @Builder(disableBuilder = true))  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(source = "lastName", target = "lastName", defaultValue = "不晓得")  
    Y source2Destination(X x);  
  
    @AfterMapping  
    default void after(@MappingTarget Y y) {  
        if (StringUtils.isBlank(y.getLastName())) {  
            y.setLastName("我是真的不晓得");  
        }  
        if("X".equals(y.getFullName())){  
            y.setFullName("X先生");  
        }  
    }  
  
}
```

### 生成的接口实现如下：

```java
public class XYMapperImpl implements XYMapper {  
  
    @Override  
    public Y source2Destination(X x) {  
        if ( x == null ) {  
            return null;  
        }  
  
        String lastName = null;  
        String firstName = null;  
        String fullName = null;  
  
        if ( x.getLastName() != null ) {  
            lastName = x.getLastName();  
        }  
        else {  
            lastName = "不晓得";  
        }  
        firstName = x.getFirstName();  
        fullName = x.getFullName();  
  
        Y y = new Y( firstName, lastName, fullName );  
  
        after( y );  
  
        return y;  
    }  
}
```

### 测试如下：
使用空字符串lastName和值为“X”的fullName进行测试：

```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().firstName("gg").lastName("").fullName("X").build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(firstName=gg, lastName=, fullName=X)
Y(firstName=gg, lastName=我是真的不晓得, fullName=X先生)
```
## 使用自定义方法加 @QualifiedName 进行处理
### 创建POJO
```java
@Data  
@Builder  
public class X {  
    private String firstName;  
    private Integer status;  
}
@Data  
@Builder  
public class Y {  
    private String firstName;  
    private String statusName;  
}
```
### 编写映射接口
映射接口中，创建一个默认方法 statusConvert 用于将Integer类型的status字段转换成String类型
的StatusName字段，并且给这个方法添加注解 @Nameed("statusConvert") 。
然后在 @Mapping 中，使用 qualifiedByName 属性指定上面给定的名称进行绑定。

```java
@Mapper(builder = @Builder(disableBuilder = true))  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(source = "status", target = "statusName", qualifiedByName = "statusConvert")  
    Y source2Destination(X x);  
  
    @Named("statusConvert")  
    default String statusConvert(Integer status) {  
        if (status == null) {  
            return "未知";  
        } else if (status == 1) {  
            return "成功";  
        } else if (status == 2) {  
            return "失败";  
        }  
        return "未知";  
    }  
  
}
```


### 接口实现：

```java
public class XYMapperImpl implements XYMapper {  
  
    @Override  
    public Y source2Destination(X x) {  
        if ( x == null ) {  
            return null;  
        }  
  
        String statusName = null;  
        String firstName = null;  
  
        statusName = statusConvert( x.getStatus() );  
        firstName = x.getFirstName();  
  
        Y y = new Y( firstName, statusName );  
  
        return y;  
    }  
}
```
### 测试验证

```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().firstName("gg").status(1).build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(firstName=gg, status=1)
Y(firstName=gg, statusName=成功)
```

## 使用表达式处理
前面我们已经见过 expression 和 defaultExpression 表达式了，表达式非常强大，既可以返回简单的字符串，也可以调用方法进行处理。

下面简单展示一个调用方法传递参数的例子。

### 创建POJO

```java
@Data  
@Builder  
public class X {  
    private String firstName;  
    private Integer type;  
}
@Data  
@Builder  
public class Y {  
    private String firstName;  
    private String typeName;  
}
```
### 创建外部处理类
```java
public class TypeUtils {  
  
    public static String convert(Integer type) {  
        String typeName = "未知";  
        if (type == null) {  
            return typeName;  
        }  
        switch (type) {  
            case 1:  
                typeName = "飞机";  
                break;  
            case 2:  
                typeName = "大炮";  
                break;  
            case 3:  
                typeName = "坦克";  
                break;  
            case 4:  
                typeName = "潜艇";  
                break;  
            default:  
                break;  
        }  
        return typeName;  
    }  
}
```

### 编写映射接口
```java
@Mapper(builder = @Builder(disableBuilder = true))  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(target = "typeName", expression = "java(com.tongde.digitalfabric.incident.test.TypeUtils.convert(x.getType()))")  
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
  
        firstName = x.getFirstName();  
  
        String typeName = com.tongde.digitalfabric.incident.test.TypeUtils.convert(x.getType());  
  
        Y y = new Y( firstName, typeName );  
  
        return y;  
    }  
}
```


### 验证测试

```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().firstName("gg").type(2).build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
    }  
  
}
```

```java
X(firstName=gg, type=2)
Y(firstName=gg, typeName=大炮)
```