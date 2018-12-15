---
title: Mybatis级联
date: 2018-12-15 22:04:32
tags: Mybatis
---

### 一对一级联

```java
public class Order {
    private Integer id;
    private Integer user_id;
    private String number;
    private Date createtime;
    private String note;

    private User user;
}    
```

对于需要查询Order（订单）信息及对应的User（用户）信息时我们可以采用一对一级联（一个订单对应一个用户）

```xml
<resultMap id="order" type="com.arthur.pojo.Order">
    <id column="id" property="id"/>
    <result column="createtime" property="createtime"/>
    <result column="number" property="number"/>
    <association javaType="com.arthur.pojo.User" property="user">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
    </association>
</resultMap>

<select id="getOrderList" resultMap="order">
    SELECT
        u.id,
        u.username,
        o.createtime,
        o.number
    FROM
    	orders o
    INNER JOIN user u ON u.id = o.user_id;
</select>
```

通过上述`<resultMap>`中的`<association>`标签可以进行一对一级联。除了`<resultMap>`标签以外也可以通过`<resultType>`进行一对一级联，只需要新建OrderDTO继承Order，然后将User中的属性加入OrderDTO即可。

```java
public class OrderDTO extends Order{
    private String username;
    private String userId;
}
```

### 一对多级联

一对多级联在Mybatis中只能使用`<resultMap>`和`<collection>`标签来进行映射

```java
public class UserDTO {
    private Integer id;
    private String username;
    private Date birthday_;

    private List<Order> orders;
}    
```

查询User（用户）信息及对应Order（订单）信息，一个用户对应多个订单

```xml
<resultMap id="userOrder" type="com.arthur.pojo.UserDTO">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <collection property="orders" ofType="com.arthur.pojo.Order">
        <id column="oid" property="id"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
	</collection>
</resultMap>

<select id="getUserOrderList" resultMap="userOrder">
    SELECT
        u.username,
        o.createtime,
        o.number,
        o.id oid
    FROM
    	user u
    INNER JOIN orders o ON u.id = o.user_id
</select>
```

### 延迟加载

MyBatis中的延迟加载，也称为**懒加载**，是指在进行关联查询时，按照设置延迟规则推迟对关联对象的select查询。延迟加载可以有效的减少数据库压力。

Mybatis的延迟加载，需要通过**resultMap标签中的association和collection**子标签才能演示成功。

***注意***：MyBatis的延迟加载只是对关联对象的查询有延迟设置，对于主加载对象都是直接执行查询语句的。

#### 直接加载

Mybatis默认使用的是直接加载，既执行完对主加载对象的select语句，马上执行对关联对象的select查询。

```java
public interface OrderMapper {

    User getUserById(Integer userId);

    Order findOrdersAndUser(Integer orderId);
}
```

```xml
<select id="findUserById" parameterType="int" resultType="com.arthur.pojo.User">
    SELECT * FROM user WHERE id = #{id}
</select>

<resultMap type="com.arthur.pojo.Order" id="orderUser">
    <id column="id" property="id" />
    <result column="user_id" property="user_id" />
    <result column="number" property="number" />
    <result column="note" property="note" />
    <!-- 一对一关联映射 -->
    <!-- property:Orders对象的user属性 javaType：user属性对应 的类型 -->
    <!-- select 属性：加载完主信息之后，会根据延迟加载策略，去调用select属性指定的statementID -->
    <!-- column属性：表示将主查询结果集中指定列的结果取出来，作为参数，传递给select属性的statement中 -->
    <association property="user"
                 javaType="com.arthur.pojo.User" column="user_id"
                 select="com.arthur.lazyloading.OrderMapper.findUserById"></association>
</resultMap>

<select id="findOrdersAndUser"
        resultMap="orderUser">
    SELECT
    o.id,
    o.user_id,
    o.number,
    o.note
    FROM
    orders o
    WHERE
    o.id = #{id}
</select>
```

测试用例：

```java
@Test
public void testDefault() {
    OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
    Order ordersAndUser = orderMapper.findOrdersAndUser(3);
}
```



执行结果：

​	![img](直接加载.png)

上述用例并没有用到`ordersAndUser`对象，但是主对象和关联对象的查询都发送了。



#### 侵入式延迟加载

Mybatis开启延迟加载需要在总配置文件中开启

```xml
<settings>
    <!--延迟加载总开关-->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!--侵入式延迟加载-->
    <setting name="aggressiveLazyLoading" value="true"/>
</settings>
```

```java
 @Test
public void testAggressiveLazyLoading() {
    OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
    Order ordersAndUser = orderMapper.findOrdersAndUser(3);
}
```

![img](侵入式延迟加载1.png)

上述用例中`ordersAndUser`对象没有被访问到，所以其关联对象的查询不会发送。

```java
@Test
public void testAggressiveLazyLoading() {
    OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
    Order ordersAndUser = orderMapper.findOrdersAndUser(3);
    System.out.println(ordersAndUser.getId());
}
```

![img](侵入式延迟加载2.png)

当访问到`ordersAndUser`对象或该对象中的任意属性时就会发送关联对象的查询语句



#### 深度延迟加载

深度延迟加载总配置

```xml
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

```java
@Test
public void testDeepLazyLoading() {
    OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
    Order ordersAndUser = orderMapper.findOrdersAndUser(3);
    System.out.println(ordersAndUser.getId());
}
```

![img](深度延迟加载1.png)

上述用例中`ordersAndUser`对象中的非关联对象属性（id）被访问到时不会执行关联对象的查询语句。

```java
@Test
public void testDeepLazyLoading() {
    OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
    Order ordersAndUser = orderMapper.findOrdersAndUser(3);
    System.out.println(ordersAndUser.getUser().getAddress());
}
```

![img](![img](深度延迟加载2.png))

上述用例中`ordersAndUser`的关联对象（User）被访问时就会执行其查询语句