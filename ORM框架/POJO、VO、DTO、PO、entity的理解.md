# Java中POJO、VO、DTO、PO、Entity的区别

**[POJO](https://so.csdn.net/so/search?q=POJO&spm=1001.2101.3001.7020)（Plain Ordinary Java Object无规则简单Java对象）**

一个中间对象，可以转化为VO、DTO、PO

**VO（View Object表示层对象）**

对应页面显示的数据对象，可以和表对应，也可以不对应。一般在Controller层使用

**DTO（Data Transfer Object数据传输对象）**

传递数据。如PO有100个属性，页面VO只显示10个，那么DTO就也传输10个。一般在Service层使用

**PO（Persistent Object持久化对象）**

持久化对象，它跟数据表形成一一对应的映射关系。一般在Dao层使用

**Entity**

实体，和PO的功能类似，和数据表一一对应，一个实体一张表