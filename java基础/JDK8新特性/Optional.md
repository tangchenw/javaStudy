Optional 是一个对象容器，具有以下两个特点：

-    提示用户要注意该对象有可能为null
-    简化if else代码

​	1、创建：

  Optional.empty()： 创建一个空的 Optional 实例

  Optional.of(T t)：创建一个 Optional 实例，当 t为null时抛出异常    

  Optional.ofNullable(T t)：创建一个 Optional 实例，但当 t为null时不会抛出异常，而是返回一个空的实例

​	2、获取：

  get()：获取optional实例中的对象，当optional 容器为空时报错

​	3、判断：

  isPresent()：判断optional是否为空，如果空则返回false，否则返回true

  ifPresent(Consumer c)：如果optional不为空，则将optional中的对象传给Comsumer函数

  orElse(T other)：如果optional不为空，则返回optional中的对象；如果为null，则返回 other 这个默认值

  orElseGet(Supplier<T> other)：如果optional不为空，则返回optional中的对象；如果为null，则使用Supplier函数生成默认值other

  orElseThrow(Supplier<X> exception)：如果optional不为空，则返回optional中的对象；如果为null，则抛出Supplier函数生成的异常

​	4、过滤：

  filter(Predicate<T> p)：如果optional不为空，则执行断言函数p，如果p的结果为true，则返回原本的optional，否则返回空的optional  

​	5、映射：

  map(Function<T, U> mapper)：如果optional不为空，则将optional中的对象 t 映射成另外一个对象 u，并将 u 存放到一个新的optional容器中。

  flatMap(Function< T,Optional<U>> mapper)：跟上面一样，在optional不为空的情况下，将对象t映射成另外一个optional

  区别：map会自动将u放到optional中，而flatMap则需要手动给u创建一个optional

## 使用示例

拿最上面的例子来讲，如果我们知道有的人没有车，那就不应该在Personal类内部声明Car的变量，因为Car类型存在，就说明一定会有Car类型的变量，而事实上并不是这样，所以使用Car类型不是一个明智的选择。我们可以使用Optional类对其进行包裹，当存在Car类型变量的时候就返回，当不存在的时候就返回一个空的Optional对象，它就像一个盒子，修饰的对象被放了进去。
除此之外，最重要的是，这就变的非常的明确，用Optional修饰，说明这里是允许变量缺失的。

```java
public class Person {
    //有的人有车，也可能没有车，所以这里用Optional来修饰
    private Optional<Car> car;
    public Optional<Car> getCar(){
        return car;
    }

    public void setCar(Optional<Car> car) {
        this.car = car;
    }
}
class Car{

    //有的车上了保险，也有可能没有上，所以这里使用Optional修饰
    private Optional<Insurance> insurance;

    public Optional<Insurance> getInsurance() {
        return insurance;
    }

    public void setInsurance(Optional<Insurance> insurance) {
        this.insurance = insurance;
    }
}

class Insurance{

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

当我们再次声明方法的时候，可以按如下方式操作：

```java
public class OptionalTest {

    public static void main(String[] args) {
        Person person = new Person();
        Car car = new Car();
        Insurance insurance = new Insurance();
		
		//Optional.of()表示对象不能为null
        insurance.setName("insurance");
        car.setInsurance(Optional.of(insurance));
        person.setCar(Optional.of(car));

        String carInsuranceName = getCarInsuranceName(person);
    }


    public static String getCarInsuranceName(Person person) {
    	//可以通过get方法从Optional中取出值
        return person.getCar().get().getInsurance().get().getName();
    }

}
```

这样我们就不需要进行null的判断、检查，因为发生异常的时候，会直接在赋值为null的地方进行报错，而不会在调用方法的时候出现空指针。

## 创建Optional对象

创建Optional的方式有多种
1、声明一个空的Optional

```java
	//使用Optional.empty()方法创建一个空的Car类型的Optional对象。
  Optional<Car> optionalCar = Optional.empty();
```

2、创建一个非空值的Optional，如果car为null的话，直接抛出空指针异常（参考上面的图片）。

```java
 Car car = new Car();
 Optional<Car> car1 = Optional.of(car);
```

3、创建一个可以为null的Optional，该方法支持car为null，但是会在用到car的地方抛出异常，但不是空指针异常。

```java
   Car car = new Car();
   Optional<Car> car2 = Optional.ofNullable(car);
```