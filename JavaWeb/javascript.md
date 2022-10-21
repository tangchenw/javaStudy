# JavaScript

**定义：**JavaScript程序不需要程序员手动编译，javascript的目标程序以普通文本形式保存，这种语言都叫做脚本语言

### Java中数据类型的大小

| 数据类型 | 大小（byte) |
| :------: | :---------: |
|   byte   |      1      |
|  short   |      2      |
|   int    |      4      |
|   long   |      8      |
|  float   |      4      |
|  double  |      8      |
| boolean  |      1      |
|   char   |      2      |

javascript是一种弱类型编程语言。

> 在js当中，当一个变量没有手动赋值的时候，系统默认赋值undefined

## js中函数的定义

方式一：

```javascript
function 函数名（形式参数列表）{
函数体；
}
```

方式二：

```javascript
函数名 = function(形式参数列表）{
函数体；
}
```

## js中变量作用域

**全局变量**：在函数体之外声明的变量

**全局变量的生命周期**：浏览器打开时声明，浏览器关闭时销毁，尽量少用。因为全局变量会一直在浏览器的内存当中，耗费内存空间。

**局部变量**：在函数体当中声明的变量，包括一个函数的形参都属于局部变量，局部变量的生命周期时，函数开始执行时局部变量的内存空间开辟，函数执行结束之后，局部变量的内存空间释放，局部变量生命周期较短。一个变量如果声明的时候没有使用var关键字，无论是在哪里声明的都是全局变量。

## JavaScrpit中数据类型

**原始类型**：Undefined、Number、String、Boolean、Null

**引用类型**：Object以及Object的子类

ES规范：在ES6之后，又基于以上的6种类型之外添加了一种新的类型：Symbol；JS中有一个运算符叫做typeof,这个运算符可以在程序的运行阶段动态的获取变量的数据类型。

> typeof运算符运算的结果是以下6个小写的字符串之一：undefined,number,string,boolean,object,function

在JS当中比较字符串使用 ==

**特殊：**Number类型包括：NaN Infinity（不是数字，无穷大）

isNaN函数：isNaN(数据),结果是true表示不是一个数字，结果是false表示是一个数字。

parseInt():可以将字符串自动转换成数字，并且取整数位。
parseFloat():可以将字符串自动转换成数字

> alert(Math.ceil("2.1"));//3  Math.ceil为向上取整

**String类型的方法**
substr(startIndex,length)开始的位置，截取的长度。
substring()开始的位置，结束的位置，左开右闭

**JS当中有两个比较特殊的运算符**
==等同运算符，只判断值是否相当
===全等运算符，既判断值是否相等，又判断类型是否相等

**一个节点对象中只要有的属性都可以通过“.”来获取**
