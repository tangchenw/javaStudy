# Spring Cache

|    注解     |                             描述                             |
| :---------: | :----------------------------------------------------------: |
| @Cacheable  | 表明 Spring 在调用方法之前，首先应该在缓存中查找方法的返回值。如果这个值能够找到，就会返回缓存的值。否则的话，这个方法就会被调用，返回值会放到缓存之中 |
|  @CachePut  | 表明 Spring 应该将方法的返回值放到缓存中。在方法的调用前并不会检查缓存，方法始终都会被调用 |
| @CacheEvict |          表明 Spring 应该在缓存中清除一个或多个条目          |
|  @Caching   |      这是一个分组的注解，能够同时应用多个其他的缓存注解      |



## @Cacheable 与 @CachePut的区别

@Cacheable 首先在缓存中查找条目，如果找到了匹配的条目，那么就不会对方法进行调用了。如果没有找到匹配的条目，方法会被调用并且返回值要放到缓存之中。而 @CachePut 并不会在缓存中检查匹配的值，目标方法总是会被调用，并将返回值添加到缓存之中。

两者共有的属性：

| 属性      | 类型     | 描述                                                         |
| --------- | -------- | ------------------------------------------------------------ |
| value     | String[] | 要使用的缓存名称                                             |
| condition | String   | SpEL 表达式，如果得到的值是 false 的话，不会将缓存应用到方法调用上 |
| key       | String   | SpEL 表达式，用来计算自定义的缓存key                         |
| unless    | String   | SpEL 表达式，如果得到的值是 true 的话，返回值不会放到缓存之中 |

对于 save() 方法来说，我们需要的键是所返回 Spittle 对象的 id 属性。表达式 #result 能够得到返回的 Spittle。借助这个对象，我们可以通过将 key 属性设置为 #result.id 来引用 id 属性：

```java
@CachePut(value="spittleCache", key="#result.id")
Spittle save(Spittle spittle)
```

按照这种方式配置 @CachePut，缓存不会去干涉 save() 方法的执行，但是返回的 Spittle 将会保存在缓存中，并且缓存的 key 与 Spittle 的 id 属性相同。

**条件化缓存**

@Cacheable 和 @CachePut 提供了两个属性用以实现条件化缓存：unless 和 condition，这两个属性都接受一个 SpEL 表达式。 **如果 unless 属性的 SpEL 表达式计算结果为 true，那么缓存方法返回的数据就不会放到缓存中**。与之类似，**如果 condition 属性的 SpEL 表达式计算结果为 false，那么对于这个方法缓存就会被禁用掉**。

**unless 属性只能阻止将对象放进缓存，但是在这个方法调用的时候，依然会去缓存中进行查找，如果找到了匹配的值，就会返回找到的值**。与之不同，**如果 condition 的表达式计算结果为 false，那么在这个方法调用的过程中，缓存是被禁用的**。就是说，不会去缓存进行查找，同时返回值也不会放进缓存中。

样例：

作为样例（尽管有些牵强），假设对于 message 属性包含 “NoCache” 的 Spittle 对象，我们不想对其进行缓存。为了阻止这样的 Spittle 对象被缓存起来，可以这样设置 unless 属性：

```java
@Cacheable(value="spittleCache",
  unless="#result.message.contain('NoCache')")
Spittle findOne(long id);
```

**condition**:在一定的条件下，我们既不希望将值添加到缓存中，也不希望从缓存中获取数据。

例如，对于 ID 值小于 10 的 Spittle 对象，我们不希望对其使用缓存。在这种场景下，这些 Spittle 是用来进行调试的测试条目，对其进行缓存并没有实际的价值。为了要对 ID 小于 10 的 Spittle 关闭缓存，可以在 @Cacheable 上使用 condition 属性，如下所示：

```java
@Cacheable(value="spittleCache",
  unless="#result.message.contain('NoCache')",
  condition="#id >= 10")
Spittle findOne(long id);
```

