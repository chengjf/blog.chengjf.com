# 定制MybatisGenerator的查询条件

公司使用Mybatis做ORM，同时使用了mybatis-generator这个插件生成基础的增删改查语句，只要***mybatis-generator:generate
***帅气的回车即可。但是提供的语句毕竟是基础的，虽然可能适合80%的场景，毕竟还有一部分需要自己处理。

通常，我们会自己修改**某Mapper.xml**文件，然后在**某Mapper.java**文件中，添加一个方法，这样就行了。

但是，很多时候并不需要这么麻烦，特别是查询的时候，可以直接修改mybatis-generator生成的**某Example.java**文件完成。

这个修改基于生成代码中的Criteria，有如下四种：

1. noValue，即不需要参数，自己本身就是查询条件
2. singleValue，即单值参数，比如常见的比较（等于，小于，大于，不等于）
3. betweenValue，即between参数，给出下限和上限，比如常见的日期区间查询
4. listValue，即列表参数，同时使用**in**和**not in**实现

<!--more-->

上一段生成的xml文件中的代码就非常清楚了：

```xml
<where>
  <foreach collection="oredCriteria" item="criteria" separator="or">
    <if test="criteria.valid">
      <trim prefix="(" prefixOverrides="and" suffix=")">
        <foreach collection="criteria.criteria" item="criterion">
          <choose>
            <when test="criterion.noValue">
              and ${criterion.condition}
            </when>
            <when test="criterion.singleValue">
              and ${criterion.condition} #{criterion.value}
            </when>
            <when test="criterion.betweenValue">
              and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
            </when>
            <when test="criterion.listValue">
              and ${criterion.condition}
              <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                #{listItem}
              </foreach>
            </when>
          </choose>
        </foreach>
      </trim>
    </if>
  </foreach>
</where>
```

现在举个例子，有这么一张工资表，有两个字段，医疗保险和公积金，现在需要查询出医疗保险和公积金之和大于某个值的数据

```java
工资Example 某个工资Example = new 工资Example();
工资Example.Criteria criteria = 某个工资Example.createCriteria();
String condition = "医疗保险+公积金>" + 某个值;
criteria.addCriterion(condition);
List<工资> rs = 工资Mapper.selectByExample(某个工资Example);
```

上面的代码中最重要的就是***criteria.addCriterion(condition);***一行，***addCriterion***方法在生成的**某Example
.java**文件中是***protected***的，只需要就其修改为***public***即可。

当然上面的这种也可以使用***criteria.addCriterion(String condition, Object value, String property);
***方法，因为医疗保险和公积金要大于某个值，这个某个值是在最后，如果是在中间的话，就只能用noValue的方式来写了。