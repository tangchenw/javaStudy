### @EqualsAndHashCode(callSuper = true/false)

首先 @EqualsAndHashCode 标在[子类](https://so.csdn.net/so/search?q=子类&spm=1001.2101.3001.7020)上

 callSuper = true，根据子类自身的字段值和从父[类继承](https://so.csdn.net/so/search?q=类继承&spm=1001.2101.3001.7020)的字段值 来生成hashcode，当两个子类对象比较时，只有子类对象的本身的字段值和继承父类的字段值都相同，[equals](https://so.csdn.net/so/search?q=equals&spm=1001.2101.3001.7020)方法的返回值是true。

 callSuper = false，根据子类自身的字段值 来生成hashcode， 当两个子类对象比较时，只有子类对象的本身的字段值相同，父类字段值可以不同，[equals](https://so.csdn.net/so/search?q=equals&spm=1001.2101.3001.7020)方法的返回值是true。

### @Builder

实体类加上@Builder注解之后，编译之后会多出一个builder()方法，和一个CardBuilder静态内部类

```java
//链式调用
User testSave3 = User.builder().name("testSave3").password(SecureUtil.md5("123456" + salt)).salt(salt)
            .email("testSave3@xkcoding.com").phoneNumber("17300000003").status(1).lastLoginTime(new DateTime()).build();
```

### @ToString(callSuper = true)

设置@ToString(callSuper = true)，callSuper属性为true  就可以实现toString方法输出父类中继承的属性。