+++
title = 'MapStruct使用之接入Spring'
date = 2023-08-10T13:41:36+08:00
draft = false
comment = true
tags = ["Java", "MapStruct"]
categories = "编程技术"
+++

接入Spring，使用Spring的依赖注入，能使MapStruct变得更加强大。

### 创建需要映射的POJO

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
}```

### 创建Spring的service

```java
@Service  
public class TypeService {  
  
    public String convert(Integer type){  
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
在 @Mapper 注解中，增加 componentModel属性，指定为 spring 。
然后使用 @Autowired 注入TypeService。

```java
@Mapper(builder = @Builder(disableBuilder = true), componentModel = "spring")  
public abstract class XYMapper {  
  
    @Autowired  
    protected TypeService typeService;  
  
    @Mapping(target = "typeName", expression = "java(typeService.convert(x.getType()))")  
    public abstract Y source2Destination(X x);  
  
}
```

### 生成的接口实现
可以看到，生成的实现中，添加了 @Component 注解，这样就归Spring进行生命周期管理。

```java
@Component  
public class XYMapperImpl extends XYMapper {  
  
    @Override  
    public Y source2Destination(X x) {  
        if ( x == null ) {  
            return null;  
        }  
  
        String firstName = null;  
  
        firstName = x.getFirstName();  
  
        String typeName = typeService.convert(x.getType());  
  
        Y y = new Y( firstName, typeName );  
  
        return y;  
    }  
}
```

### 测试验证

```java
@RestController  
@AllArgsConstructor  
@Slf4j  
public class MessageController implements MessageClient {  
  
    private XYMapper xyMapper;  
  
    @Override  
    public SingleResponse<String> sendMessage(SendMsgCmd cmd) {  
    
        X x = X.builder().firstName("GG").type(1).build();  
        System.out.println(x);  
        Y y = xyMapper.source2Destination(x);  
        System.out.println(y);  

        return SingleResponse.of("eventId");  
    }  
  
}
```

```java
X(firstName=GG, type=1)
Y(firstName=GG, typeName=飞机)
```