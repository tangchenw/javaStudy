

## IntStream range()方法的使用

Java中，IntStream是一个接口，继承BaseStream。里面有很多方法，如：filter()、map()、sorted()、empty()等

### Intstream.range()用法

### 说明：

- range(0,1000)方法，创建一个以1为增量步长，从startInclusive(包括)到endExclusive(不包括)的有序的数字流。
- 产生的IntStream对象，可以像数组一样遍历。(表现为链式调用.forEach(i -> todo));