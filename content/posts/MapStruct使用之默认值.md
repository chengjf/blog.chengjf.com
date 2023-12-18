+++
title = 'MapStruct使用之默认值'
date = 2023-08-10T13:39:36+08:00
draft = false
comment = true
tags = ["Java", "MapStruct"]
categories = "编程技术"
+++

## 当源对象字段值为null时给定默认值
通常的情况时这样，源对象中的字段如果不为null，那么直接给目标对象中的对应字段进行赋值。
但是如果源对象中的字段是null，那么给目标对象中的对应字段赋默认值。

默认值有两种处理，一个是用 defaultValue 给定值即可，另外一种是使用 defaultExpression 使用表达式进行处理。

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
}
```

### 编写映射接口
```java
@Mapper(builder = @Builder(disableBuilder = true))  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(source = "lastName", target = "lastName", defaultValue = "不晓得")  
    Y source2Destination(X x);  
  
    @Mapping(source = "lastName", target = "lastName", defaultExpression = "java(\"不晓得呀呀呀\")")  
    Y source2Destination2(X x);  
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
  
        String lastName = null;  
        String firstName = null;  
  
        if ( x.getLastName() != null ) {  
            lastName = x.getLastName();  
        }  
        else {  
            lastName = "不晓得";  
        }  
        firstName = x.getFirstName();  
  
        Y y = new Y( firstName, lastName );  
  
        return y;  
    }  
  
    @Override  
    public Y source2Destination2(X x) {  
        if ( x == null ) {  
            return null;  
        }  
  
        String lastName = null;  
        String firstName = null;  
  
        if ( x.getLastName() != null ) {  
            lastName = x.getLastName();  
        }  
        else {  
            lastName = "不晓得呀呀呀";  
        }  
        firstName = x.getFirstName();  
  
        Y y = new Y( firstName, lastName );  
  
        return y;  
    }  
}
```
### 测试验证
```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().firstName("gg").build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
        Y y2 = XYMapper.INSTANCE.source2Destination2(x);  
        System.out.println(y2);  
    }  
  
}
```

## 直接给目标对象中新字段给定默认值
即目标对象中有新的字段，在源对象中没有对应的字段，直接给新字段进行赋值。
可以通过 constant 属性或者 expression 来进行处理。

### 创建需要映射的POJO
```java
@Data  
@Builder  
public class X {  
    private String firstName;  
}
@Data  
@Builder  
public class Y {  
    private String firstName;  
    private String lastName;  
}
```
### 编写映射接口
```java
@Mapper(builder = @Builder(disableBuilder = true))  
public interface XYMapper {  
  
    XYMapper INSTANCE = Mappers.getMapper(XYMapper.class);  
  
    @Mapping(target = "lastName", constant = "不晓得")  
    Y source2Destination(X x);  
    
    @Mapping(target = "lastName", expression = "java(\"真的不晓得\")")  
    Y source2Destination2(X x);
  
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
  
        String lastName = "不晓得";  
  
        Y y = new Y( firstName, lastName );  
  
        return y;  
    }  
  
    @Override  
    public Y source2Destination2(X x) {  
        if ( x == null ) {  
            return null;  
        }  
  
        String firstName = null;  
  
        firstName = x.getFirstName();  
  
        String lastName = "真的不晓得";  
  
        Y y = new Y( firstName, lastName );  
  
        return y;  
    }  
}
```

### 测试验证
```java
public class Test {  
  
    public static void main(String[] args) {  
        X x = X.builder().firstName("gg").build();  
        System.out.println(x);  
        Y y = XYMapper.INSTANCE.source2Destination(x);  
        System.out.println(y);  
        Y y2 = XYMapper.INSTANCE.source2Destination2(x);  
        System.out.println(y2);  
    }  
  
}
```

```java
X(firstName=gg)
Y(firstName=gg, lastName=不晓得)
Y(firstName=gg, lastName=真的不晓得)
```