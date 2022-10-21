# Mapper接口动态代理实现原理

假设有一个如下的Mapper接口，

```java
public interface UserMapper{
List<SysUser> selectAll();
}
```

这里使用Java动态代理方式创建一个代理类，代码如下：

```java
public class MyMapperProxy<T> implements InvocationHandler{
    private class<T> mapperInterface;
    private SqlSession sqlSession;
    
    public MyMapperProxy(Class<T> mapperInterface,SqlSession sqlSession){
        this.mapperInterface =  mapperInterface;
        this.sqlSession = sqlSession;
    }
    
    @Override
    public Object invoke(Object proxy,Method method,Object[] args) throws Throwable{
        //获取接口的全限定类名，以及方法的名称组合得到方法的id
        List<T> list = sqlSession.selectList(mapperInterface.getCanonicalName()+"."+method.getName());
        return list;
    }
}
```

测试代码如下：

```java
SqlSession sqlSession = getSqlSession();
MyMapperProxy userMapperProxy = new MyMapperProxy(UserMapper.class,sqlSession);
UserMapper userMapper = (UserMapper) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
new class[]{UserMapper.class},userMapperProxy);
List<SysUser> user = userMapper.selectAll();
```

