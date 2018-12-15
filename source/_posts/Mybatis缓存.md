---
title: Mybatis缓存
date: 2018-12-15 22:05:03
tags: Mybatis
---

### 一级缓存

Mybatis中一级缓存是`SqlSession`级别的缓存。即同一个`SqlSession`执行2次相同查询（条件也相同），Mybatis只会在第一发送sql从数据库中查询数据，然后将数据缓存起来，第二次就会从`SqlSession`中返回数据。Mybatis默认开启了一级缓存。

```java
@Test
public void testOneLevelCache() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    User user1 = userMapper.findUserById(1);
    User user2 = userMapper.findUserById(1);
    System.out.println(user1);
    System.out.println(user2);
}
```

![1536655016308](Mybatis缓存1.png)



上述代码中虽然对`User`对象执行了2次查询，但是Mybatis只发送了一次sql从数据库中进行查询。

### 二级缓存

二级缓存是Mapper（namespace）级别的缓存。多个`SqlSession`去操作同一个Mapper的sql语句，多个`SqlSession`可以共用二级缓存，二级缓存是跨`SqlSession`的。一个`SqlSession`在提交时会把数据刷入2级缓存中，另一个`SqlSession`执行相同语句时会从2级缓存中获取数据。

Mybatis默认是关闭二级缓存的，要使用需要在Mybatis的总配置文件中开启（全局），然后再在需要开启二级缓存的映射文件中使用`<cache/>`标签进行开启（单个映射文件）



```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

```java
@Test
public void testTwoLevelCache() {
    SqlSession sqlSession1 = sqlSessionFactory.openSession();
    UserMapper userMapper = sqlSession1.getMapper(UserMapper.class);
    User user1 = userMapper.findUserById(1);
    sqlSession1.close();//当sqlSession1提交时会将数据放入二级缓存

    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
    User user2 = userMapper2.findUserById(1);
    System.out.println(user1);
    System.out.println(user2);
}
```

![1536656135757](Mybatis缓存2.png)

上述代码虽然属于不同`SqlSession`但是也只发送了一次sql到数据库进行查询。

Mybatis在执行查询时会先查询二级缓存，然后再查询一级缓存。

二级缓存：

```java
@Override
  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
      throws SQLException {
    Cache cache = ms.getCache();
    if (cache != null) {
      flushCacheIfRequired(ms);
      if (ms.isUseCache() && resultHandler == null) {
        ensureNoOutParams(ms, boundSql);
        @SuppressWarnings("unchecked")
        List<E> list = (List<E>) tcm.getObject(cache, key);
        if (list == null) {
          list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
          tcm.putObject(cache, key, list); // issue #578 and #116
        }
        return list;
      }
    }
    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  }
```

一级缓存：

```java
@SuppressWarnings("unchecked")
  @Override
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
      clearLocalCache();
    }
    List<E> list;
    try {
      queryStack++;
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
    } finally {
      queryStack--;
    }
    if (queryStack == 0) {
      for (DeferredLoad deferredLoad : deferredLoads) {
        deferredLoad.load();
      }
      // issue #601
      deferredLoads.clear();
      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
        // issue #482
        clearLocalCache();
      }
    }
    return list;
  }
```



