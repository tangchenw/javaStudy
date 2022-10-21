# ELK协议栈简介

## ELK简介

ELK是三个软件的统称，即Elasticsearch、Logstash和Kibana三个开源软件的缩写。这三款软件都是开源软件，通常配合使用，并且都先后归于Elastic.co企业名下，故被简称为ELK协议栈。ELK主要用于部署在企业架构中，收集多台设备上多个服务的日志信息，并将其统一整合后提供给用户。

## ELK架构

在ELK架构中，Elasticsearch、Logstash和Kibana三款软件作用如下：
**1、Elasticsearch**
Elasticsearch是一个高度可扩展的全文搜索和分析引擎，基于Apache Lucence（事实上，Lucence也是百度所采用的搜索引擎）构建，能够对大容量的数据进行接近实时的存储、搜索和分析操作。
**2、Logstash**
Logstash是一个数据收集引擎，它可以动态的从各种数据源搜集数据，并对数据进行过滤、分析和统一格式等操作，并将输出结果存储到指定位置上。Logstash支持普通的日志文件和自定义Json格式的日志解析。
**3、Kibana**
Kibana是一个数据分析和可视化平台，通常与Elasticsearch配合使用，用于对其中的数据进行搜索、分析，并且以统计图标的形式展示。
ELK的架构如下所示：

![elk执行过程图](C:\Users\汤琛\Desktop\学习资料\常用中间件\ElasticSearch\images\elk执行过程图.png)

如上图所示，Logstash安装在各个设备上，用于收集日志信息，收集到的日志信息统一汇总到Elasticsearch上，然后由Kibana负责web端的展示。其中，如果终端设备过多，会导致Elasticsearch过载的现象，此时，我们可以采用一台Redis设备作为消息队列，以暂时缓存数据，避免Elasticsearch压力突发。

## 三、ELK优点

ELK架构优点如下：
**1、处理方式灵活。** Elasticsearch是全文索引，具有强大的搜索能力。
**2、配置相对简单。** Kibana的配置非常简单，Elasticsearch则全部使用Json接口，配置也不复杂，Logstash的配置使用模块的方式，配置也相对简单。
**3、检索性能高。** ELK架构通常可以达到百亿级数据的查询秒级响应。
**4、集群线性扩展。** Elasticsearch本身没有单点的概念，自动默认集群模式，Elasticsearch和Logstash都可以灵活扩展。
**5、页面美观。** Kibana的前端设计美观，且操作简单。