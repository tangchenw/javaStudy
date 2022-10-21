# Redis的高阶特性

Redis的基本数据类型就只要五种，为了满足不同的业务场景，还提供了BitMaps、HyperLogLog、Geospatial、Stream这些高阶的数据类型，还有支持JSON类型的RedisJSON  

## BitMaps

Bitmaps叫位图，他不是Redis的基本数据类型，而是基于String数据类型的按位操作，高阶数据类型的一种Bitmaps支持的最大位数是2^32位，使用512M内存就可以存储多达42.9亿的字节信息。

**位图的操作**

```java
setbit key offset value  //设置key在offset处的bit值（只能是0或者1）

getbit key offset  //获取key在offset处的bit值

bitcount key  //获取key的bit位为1的个数

bitpos key value  //返回第一个被设置为bit值的索引值

bitpos and [or/xor/not] destkey key[key...]  //对多个key进行逻辑运算后存入destkey
```

|               数据量               |             描述             |
| :--------------------------------: | :--------------------------: |
|               24byte               |     一个人一天的签到数据     |
| 24B*1千万=24千万B/1024/1024 =228MB |   一千万用户一天的签到数据   |
|     228MB*30 = 6840MB/1024=6GB     |  一千万签到一个月的签到数据  |
|            6GB*12 =72GB            | 一千万用户签到一年的签到数据 |
|           72GB*5 = 360GB           | 一千万用户签到五年的签到数据 |

bitcount key   //获得key的bit位为1的个数

|           数据量            |            描述            |
| :-------------------------: | :------------------------: |
|    3千万B/1024/1024=28MB    | 一千万用户一个月的签到数据 |
|       28MB*12 = 336MB       |  一千万签到一年的签到数据  |
| 336MB*5 = 1680MB约等于1.5GB |  一千万签到五年的签到数据  |

**用户每月签到**

用户id和年为key，日期作为偏移量1表示签到

> setbit user:sign:1000:2021 615 1 #id为1000的用户20210615 签到
>
> getbit user:sign:1000:2021 615  #获取id为1000的用户20210615签到状态 0表示未签到，1表示签到
>
> bitcount user:sign:1000:2021 #获取id为1000的用户的签到次数

**统计活跃用户**

日期为key，用户Id为偏移量1表示活跃

> setbit 20210201 1000 1 #20210201的1000号用户上线
>
> setbit 20210201 1001 1 #20210201的1001号用户上线
>
> bitcount 20210201 #20210201的上线用户有2个

## HyperLogLog

HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数  

**HyperLogLog操作**

> PFADD key element [element ...] //添加指定元素到 HyperLogLog 中
> PFCOUNT key [key ...] //返回给定 HyperLogLog 的基数估算值
> PFMERGE destkey sourcekey [sourcekey ...] //将多个 HyperLogLog 合并为一个 HyperLogLog  

## Genspatial

地理信息类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作  

**Gen的操作**

> geoadd：添加地理位置的坐标
> geopos：获取地理位置的坐标
> geodist：计算两个位置之间的距离
> georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合
> georadiusbymember：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合
> geohash：返回一个或多个位置对象的 geohash 值  