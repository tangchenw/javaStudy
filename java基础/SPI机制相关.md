## RunTime.getRunTime().addShutdownHook

RunTime.getRunTime().addShutdownHook的作用就是在JVM销毁前执行的一个线程.当然这个线程依然要自己写.

利用这个性质,如果我们之前定义了一系列的[线程池](https://so.csdn.net/so/search?q=线程池&spm=1001.2101.3001.7020)供程序本身使用,那么就可以在这个最后执行的线程中把这些线程池优雅的关闭掉.

## ServiceLoader原理

ServiceLoader是jdk6里面引进的一个特性。它用来实现SPI(Service Provider Interface)，一种**服务发现机制**，很多框架用它来做来做服务的扩展发现。
系统里抽象的各个模块一般会有很多种不同的实现，如JDBC、日志等。通常模块之间我们均是基于接口进行编程，而不是对实现类进行硬编码。这时候就需要一种动态替换发现的机制，即在运行时动态地给接口添加实现，而不需要在程序中指明。

当服务的提供者提供了服务接口的一种实现之后，在jar包的**META-INF/services/** 目录里同时创建一个以服务接口命名的该服务接口具体实现类文件。当外部程序装配该模块时，通过该jar包META-INF/services/里的配置文件找到具体的实现类名，从而完成模块的注入，而不需要在代码里定制。

ServiceLoader的**load方法**将在META-INF/services/com.study.SPIService中配置的子类都进行了加载。