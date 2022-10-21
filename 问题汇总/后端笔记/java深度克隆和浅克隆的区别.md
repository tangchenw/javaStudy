# 深克隆与浅克隆的区别

## 浅克隆

克隆对象实现Cloneable接口，重写其中的clone()方法，克隆的引用指向原型对象的新克隆体。



## 深克隆实现步骤

1. 实现Cloneable和Serializable接口
2. 在类中重写Object类中的clone()方法，通过对象的序列化和反序列化实现深克隆；
3. 把克隆的引用指向原型对象新的克隆体；

```java
import java.io.*;

class Person implements Cloneable,Serializable{
    private String name;
    private Integer age;
    private Address address;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return this.name+"  "+this.age+"  "+this.address+"  "+super.toString();
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Object o = null;
        try{
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
            objectOutputStream.writeObject(this);

            ByteArrayInputStream inputStream = new ByteArrayInputStream(outputStream.toByteArray());
            ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);

            o = objectInputStream.readObject();

        }catch (Exception e){
            e.printStackTrace();
        }
        return o;
    }
}

```

